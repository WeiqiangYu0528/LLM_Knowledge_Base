# Enhanced Search Tooling System — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the agent's basic grep search with a multi-tier search service combining lexical (ripgrep), semantic (ChromaDB + Qwen embeddings), and symbol lookup (tree-sitter) — with smart repo targeting and cheap-first escalation.

**Architecture:** A `backend/search/` module provides the search service layer with clean interfaces. Agent tools in `backend/search_tools.py` are thin wrappers. The `SearchOrchestrator` auto-targets repos and escalates from cheap lexical to semantic search only when needed.

**Tech Stack:** Python 3.14, ChromaDB (file-backed vector store), Qwen text-embedding-v3 (1024-dim, via DashScope OpenAI-compatible API), tree-sitter (Python/TypeScript/C# parsers), ripgrep (lexical search), LangChain/LangGraph (agent framework).

**Spec:** `docs/superpowers/specs/2026-04-11-enhanced-search-tooling-design.md`

---

## File Map

### New files to create

| File | Responsibility |
|------|---------------|
| `backend/search/__init__.py` | Package exports: `SearchOrchestrator`, `build_index`, `get_index_status` |
| `backend/search/registry.py` | `RepoMeta` dataclass, `RepoRegistry` class with keyword/context targeting |
| `backend/search/lexical.py` | `LexicalSearch` class wrapping ripgrep with ranking |
| `backend/search/semantic.py` | `SemanticSearch` class wrapping ChromaDB + Qwen embeddings |
| `backend/search/symbols.py` | `SymbolIndex` class using tree-sitter for function/class extraction |
| `backend/search/chunker.py` | `chunk_markdown()`, `chunk_source_file()` functions |
| `backend/search/indexer.py` | `IndexBuilder` class for startup/incremental indexing |
| `backend/search/orchestrator.py` | `SearchOrchestrator` class: repo targeting, tier escalation, ranking |
| `backend/search_tools.py` | LangChain `@tool` wrappers: `smart_search`, `find_symbol`, `read_code_section` |
| `backend/search/test_registry.py` | Tests for RepoRegistry |
| `backend/search/test_lexical.py` | Tests for LexicalSearch |
| `backend/search/test_semantic.py` | Tests for SemanticSearch |
| `backend/search/test_symbols.py` | Tests for SymbolIndex |
| `backend/search/test_chunker.py` | Tests for chunking logic |
| `backend/search/test_orchestrator.py` | Tests for SearchOrchestrator |
| `backend/test_search_tools.py` | Tests for the agent tool wrappers |
| `backend/test_search_integration.py` | End-to-end integration tests |

### Files to modify

| File | Changes |
|------|---------|
| `backend/pyproject.toml` | Add chromadb, tree-sitter, tree-sitter-python, tree-sitter-typescript, tree-sitter-c-sharp |
| `backend/security.py` | Add `qwen_embedding_model` setting |
| `backend/agent.py` | Replace `search_knowledge_base` with new tools from `search_tools.py`, update system prompt |
| `backend/main.py` | Add startup indexing hook, add `/search/status` endpoint |
| `backend/.env.example` | Add `QWEN_EMBEDDING_MODEL` config |

---

### Task 1: Dependencies & Configuration

**Files:**
- Modify: `backend/pyproject.toml`
- Modify: `backend/security.py:7-18`
- Modify: `backend/.env.example`

- [ ] **Step 1: Add new dependencies to pyproject.toml**

```toml
dependencies = [
    "fastapi[standard]",
    "uvicorn",
    "langchain",
    "langchain-openai",
    "litellm",
    "langgraph",
    "pyotp",
    "python-jose[cryptography]",
    "passlib[bcrypt]",
    "python-multipart",
    "pydantic-settings",
    "python-dotenv",
    "httpx",
    "chromadb",
    "tree-sitter",
    "tree-sitter-python",
    "tree-sitter-typescript",
    "tree-sitter-c-sharp",
]
```

- [ ] **Step 2: Add embedding config to security.py**

Add these fields to the `Settings` class after the existing publish repo fields:

```python
    # Search / Embedding config
    qwen_embedding_model: str = "text-embedding-v3"
```

- [ ] **Step 3: Update .env.example**

Append to the end of the file:

```
# --- Search / Embedding Config ---
# Qwen embedding model for semantic search (uses QWEN_API_KEY)
QWEN_EMBEDDING_MODEL="text-embedding-v3"
```

- [ ] **Step 4: Install dependencies**

Run: `cd /Users/weiqiangyu/Downloads/wiki/backend && uv sync`
Expected: Resolves and installs chromadb, tree-sitter, and language parsers.

- [ ] **Step 5: Verify imports work**

Run: `cd /Users/weiqiangyu/Downloads/wiki/backend && .venv/bin/python -c "import chromadb; import tree_sitter; import tree_sitter_python; import tree_sitter_typescript; print('OK')"`
Expected: Prints `OK`

---

### Task 2: RepoRegistry

**Files:**
- Create: `backend/search/__init__.py`
- Create: `backend/search/registry.py`
- Create: `backend/search/test_registry.py`

- [ ] **Step 1: Create search package init**

```python
"""Search service layer for the wiki agent."""
```

- [ ] **Step 2: Write test_registry.py**

```python
import sys, os
sys.path.insert(0, os.path.join(os.path.dirname(__file__), ".."))

from search.registry import RepoRegistry, RepoMeta

registry = RepoRegistry()

# --- Test 1: All 6 repos registered ---
print("=== TEST 1: all repos registered ===")
assert len(registry.repos) == 6
names = {r.namespace for r in registry.repos}
assert names == {"claude-code", "deepagents", "opencode", "openclaw", "autogen", "hermes-agent"}
print("PASS")

# --- Test 2: get_by_namespace ---
print("\n=== TEST 2: get_by_namespace ===")
repo = registry.get_by_namespace("deepagents")
assert repo is not None
assert repo.source_dir == "deepagents"
assert repo.wiki_dir == "docs/deepagents-wiki"
assert "python" in repo.languages
print("PASS")

# --- Test 3: target from page context ---
print("\n=== TEST 3: target from page context ===")
targets = registry.target(query="how does memory work", page_url="/deepagents-wiki/entities/memory-system/")
assert targets[0].namespace == "deepagents"
print(f"PASS — primary target: {targets[0].namespace}")

# --- Test 4: target from keyword ---
print("\n=== TEST 4: target from keyword ===")
targets = registry.target(query="tool system MCP permissions")
assert targets[0].namespace == "claude-code"
print(f"PASS — primary target: {targets[0].namespace}")

# --- Test 5: target ambiguous query returns all ---
print("\n=== TEST 5: target ambiguous query ===")
targets = registry.target(query="how does the agent loop work")
assert len(targets) >= 2
print(f"PASS — {len(targets)} repos targeted")

# --- Test 6: target from explicit namespace ---
print("\n=== TEST 6: explicit namespace ===")
targets = registry.target(query="anything", namespace="autogen")
assert len(targets) == 1 and targets[0].namespace == "autogen"
print("PASS")

# --- Test 7: get nonexistent namespace ---
print("\n=== TEST 7: get nonexistent ===")
assert registry.get_by_namespace("bogus") is None
print("PASS")

print("\n✅ All registry tests passed.")
```

- [ ] **Step 3: Run test to verify it fails**

Run: `cd /Users/weiqiangyu/Downloads/wiki/backend && .venv/bin/python search/test_registry.py`
Expected: FAIL with `ModuleNotFoundError: No module named 'search.registry'`

- [ ] **Step 4: Implement registry.py**

```python
"""Repository registry with metadata and query-to-repo targeting."""

import re
from dataclasses import dataclass, field


@dataclass
class RepoMeta:
    namespace: str
    source_dir: str
    wiki_dir: str
    languages: list[str]
    keywords: list[str]
    description: str
    file_count: int = 0


# Static repo metadata — update when adding new repos.
_REPO_DEFS: list[dict] = [
    {
        "namespace": "claude-code",
        "source_dir": "claude_code",
        "wiki_dir": "docs/claude-code",
        "languages": ["typescript", "python"],
        "keywords": [
            "claude code", "tool system", "mcp", "permissions", "permission model",
            "skills", "cli", "state management", "slash commands", "tengu",
        ],
        "description": "Claude Code CLI: agent system, tool system, permission model, MCP, skills, memory, state management",
    },
    {
        "namespace": "deepagents",
        "source_dir": "deepagents",
        "wiki_dir": "docs/deepagents-wiki",
        "languages": ["python", "typescript"],
        "keywords": [
            "deep agents", "deepagents", "graph factory", "subagent", "session persistence",
            "acp server", "evals", "middleware", "backend protocol",
        ],
        "description": "DeepAgents framework: graph factory, subagent system, session persistence, ACP server, evals",
    },
    {
        "namespace": "opencode",
        "source_dir": "opencode",
        "wiki_dir": "docs/opencode-wiki",
        "languages": ["typescript"],
        "keywords": [
            "opencode", "session system", "provider system", "lsp", "plugin system",
            "tui", "terminal ui", "bun",
        ],
        "description": "OpenCode AI coding assistant: session system, provider system, LSP integration, plugin system",
    },
    {
        "namespace": "openclaw",
        "source_dir": "openclaw",
        "wiki_dir": "docs/openclaw-wiki",
        "languages": ["typescript", "javascript"],
        "keywords": [
            "openclaw", "gateway", "control plane", "channel system", "routing",
            "voice", "media stack", "eliza", "pnpm",
        ],
        "description": "OpenClaw personal assistant: gateway control plane, channel system, routing, voice/media stack",
    },
    {
        "namespace": "autogen",
        "source_dir": "autogen",
        "wiki_dir": "docs/autogen-wiki",
        "languages": ["python", "csharp"],
        "keywords": [
            "autogen", "agentchat", "distributed workers", "model clients",
            "autogen studio", "microsoft", "magentic", "core runtime",
        ],
        "description": "Microsoft AutoGen: core runtime, AgentChat, distributed workers, model clients, AutoGen Studio",
    },
    {
        "namespace": "hermes-agent",
        "source_dir": "hermes-agent",
        "wiki_dir": "docs/hermes-agent-wiki",
        "languages": ["python"],
        "keywords": [
            "hermes", "hermes agent", "agent loop", "prompt assembly",
            "tool registry", "memory", "learning", "conversational",
        ],
        "description": "Hermes conversational agent: agent loop, prompt assembly, tool registry, memory/learning",
    },
]


class RepoRegistry:
    """Registry of all source code repositories with targeting logic."""

    def __init__(self) -> None:
        self.repos: list[RepoMeta] = [RepoMeta(**d) for d in _REPO_DEFS]
        self._by_namespace: dict[str, RepoMeta] = {r.namespace: r for r in self.repos}

    def get_by_namespace(self, namespace: str) -> RepoMeta | None:
        return self._by_namespace.get(namespace)

    def target(
        self,
        query: str,
        page_url: str = "",
        namespace: str = "",
    ) -> list[RepoMeta]:
        """Determine which repos are relevant for a query.

        Returns repos ordered by relevance (best match first).

        Signals used (in priority order):
        1. Explicit namespace (if provided)
        2. Page URL (extract namespace from wiki path)
        3. Keyword matching against query
        """
        if namespace:
            repo = self.get_by_namespace(namespace)
            return [repo] if repo else self.repos

        # Signal: page URL
        primary_from_url: RepoMeta | None = None
        if page_url:
            for repo in self.repos:
                wiki_suffix = repo.wiki_dir.replace("docs/", "")
                if wiki_suffix in page_url:
                    primary_from_url = repo
                    break

        # Signal: keyword scoring
        query_lower = query.lower()
        scores: list[tuple[float, RepoMeta]] = []
        for repo in self.repos:
            score = 0.0
            for kw in repo.keywords:
                if kw in query_lower:
                    score += len(kw)  # Longer keyword matches = more specific
            if repo.namespace in query_lower:
                score += 20
            scores.append((score, repo))

        scores.sort(key=lambda x: x[0], reverse=True)

        # Build result: primary from URL first, then keyword-ranked
        result: list[RepoMeta] = []
        seen: set[str] = set()

        if primary_from_url:
            result.append(primary_from_url)
            seen.add(primary_from_url.namespace)

        for score, repo in scores:
            if repo.namespace not in seen:
                if score > 0 or not result:
                    result.append(repo)
                    seen.add(repo.namespace)

        return result if result else self.repos


repo_registry = RepoRegistry()
```

- [ ] **Step 5: Run tests**

Run: `cd /Users/weiqiangyu/Downloads/wiki/backend && .venv/bin/python search/test_registry.py`
Expected: All 7 tests pass.

---

### Task 3: Chunker

**Files:**
- Create: `backend/search/chunker.py`
- Create: `backend/search/test_chunker.py`

- [ ] **Step 1: Write test_chunker.py**

```python
import sys, os
sys.path.insert(0, os.path.join(os.path.dirname(__file__), ".."))

from search.chunker import chunk_markdown, chunk_source_file

# --- Test 1: Markdown chunking by headings ---
print("=== TEST 1: chunk_markdown by headings ===")
md = """# Title

Intro paragraph.

## Section One

Content of section one with multiple sentences.
More detail here.

## Section Two

Content of section two.

### Subsection 2.1

Deep content.
"""
chunks = chunk_markdown(md, file_path="docs/test.md")
assert len(chunks) >= 3, f"Expected >=3 chunks, got {len(chunks)}"
assert chunks[0]["file_path"] == "docs/test.md"
assert "section" in chunks[0] or "heading" in chunks[0]
print(f"PASS — {len(chunks)} chunks")

# --- Test 2: Empty markdown ---
print("\n=== TEST 2: empty markdown ===")
chunks = chunk_markdown("", file_path="docs/empty.md")
assert len(chunks) == 0
print("PASS")

# --- Test 3: Markdown with no headings ---
print("\n=== TEST 3: markdown without headings ===")
chunks = chunk_markdown("Just a paragraph.\nAnother line.", file_path="docs/flat.md")
assert len(chunks) == 1
assert "Just a paragraph" in chunks[0]["text"]
print("PASS")

# --- Test 4: Source file chunking ---
print("\n=== TEST 4: chunk_source_file ===")
py_code = '''"""Module docstring."""

def hello(name: str) -> str:
    """Say hello."""
    return f"Hello, {name}"

class Greeter:
    """A greeter class."""

    def greet(self, name: str) -> str:
        """Greet someone."""
        return f"Hi, {name}"
'''
chunks = chunk_source_file(py_code, file_path="src/hello.py", language="python")
assert len(chunks) >= 2, f"Expected >=2 chunks, got {len(chunks)}"
print(f"PASS — {len(chunks)} chunks")

# --- Test 5: Source with no functions/classes ---
print("\n=== TEST 5: source with no functions ===")
chunks = chunk_source_file("x = 1\ny = 2\n", file_path="src/const.py", language="python")
assert len(chunks) >= 1  # Should still get at least the whole-file chunk
print("PASS")

# --- Test 6: Max chunk size ---
print("\n=== TEST 6: large section is split ===")
big_section = "## Big\n\n" + ("word " * 2000) + "\n"
chunks = chunk_markdown(big_section, file_path="docs/big.md", max_tokens=500)
assert len(chunks) >= 2
print(f"PASS — split into {len(chunks)} chunks")

print("\n✅ All chunker tests passed.")
```

- [ ] **Step 2: Run test to verify it fails**

Run: `cd /Users/weiqiangyu/Downloads/wiki/backend && .venv/bin/python search/test_chunker.py`
Expected: FAIL — `ModuleNotFoundError`

- [ ] **Step 3: Implement chunker.py**

```python
"""Chunking logic for wiki markdown and source code files."""

import re


def chunk_markdown(
    content: str,
    file_path: str,
    max_tokens: int = 500,
) -> list[dict]:
    """Split markdown content into chunks by headings.

    Each chunk is a dict with keys: text, file_path, section, heading, start_line.
    """
    if not content.strip():
        return []

    sections: list[dict] = []
    current_heading = ""
    current_lines: list[str] = []
    current_start = 1

    for i, line in enumerate(content.splitlines(), 1):
        heading_match = re.match(r"^(#{1,3})\s+(.+)", line)
        if heading_match and current_lines:
            sections.append({
                "heading": current_heading,
                "text": "\n".join(current_lines).strip(),
                "start_line": current_start,
            })
            current_lines = [line]
            current_heading = heading_match.group(2).strip()
            current_start = i
        else:
            if heading_match:
                current_heading = heading_match.group(2).strip()
                current_start = i
            current_lines.append(line)

    if current_lines:
        sections.append({
            "heading": current_heading,
            "text": "\n".join(current_lines).strip(),
            "start_line": current_start,
        })

    # Split oversized sections
    result: list[dict] = []
    for sec in sections:
        if not sec["text"]:
            continue
        words = sec["text"].split()
        if len(words) <= max_tokens:
            result.append({
                "text": sec["text"],
                "file_path": file_path,
                "section": sec["heading"],
                "heading": sec["heading"],
                "start_line": sec["start_line"],
            })
        else:
            # Split into max_tokens-sized pieces
            for j in range(0, len(words), max_tokens):
                chunk_words = words[j:j + max_tokens]
                result.append({
                    "text": " ".join(chunk_words),
                    "file_path": file_path,
                    "section": sec["heading"],
                    "heading": sec["heading"],
                    "start_line": sec["start_line"],
                })

    return result


def chunk_source_file(
    content: str,
    file_path: str,
    language: str = "python",
    max_lines: int = 200,
) -> list[dict]:
    """Split source code into chunks by top-level definitions.

    Uses regex-based extraction (lightweight fallback when tree-sitter
    is used separately for symbol indexing). Each chunk is a dict with
    keys: text, file_path, symbol, kind, start_line.
    """
    if not content.strip():
        return []

    lines = content.splitlines()

    # Language-specific definition patterns
    if language in ("python",):
        pattern = re.compile(r"^(class|def|async\s+def)\s+(\w+)")
    elif language in ("typescript", "javascript"):
        pattern = re.compile(
            r"^(?:export\s+)?(?:async\s+)?(?:function|class|interface|type|enum|const)\s+(\w+)"
        )
    elif language in ("csharp",):
        pattern = re.compile(
            r"^\s*(?:public|private|protected|internal|static|abstract|sealed|partial|\s)*"
            r"(?:class|struct|interface|enum|record)\s+(\w+)"
        )
    else:
        pattern = re.compile(r"^(?:def|class|function|interface|type)\s+(\w+)")

    # Find definition boundaries
    defs: list[tuple[int, str, str]] = []  # (line_idx, symbol_name, kind)
    for i, line in enumerate(lines):
        m = pattern.match(line)
        if m:
            groups = m.groups()
            symbol = groups[-1] if groups else "unknown"
            kind_match = re.match(r"\s*(?:export\s+)?(?:async\s+)?(class|def|function|interface|type|enum|struct|record)", line)
            kind = kind_match.group(1) if kind_match else "definition"
            defs.append((i, symbol, kind))

    if not defs:
        # No definitions found — return whole file as one chunk (truncated)
        text = "\n".join(lines[:max_lines])
        return [{
            "text": text,
            "file_path": file_path,
            "symbol": "",
            "kind": "module",
            "start_line": 1,
        }]

    chunks: list[dict] = []
    for idx, (start, symbol, kind) in enumerate(defs):
        end = defs[idx + 1][0] if idx + 1 < len(defs) else len(lines)
        end = min(end, start + max_lines)
        text = "\n".join(lines[start:end]).rstrip()
        if text:
            chunks.append({
                "text": text,
                "file_path": file_path,
                "symbol": symbol,
                "kind": kind,
                "start_line": start + 1,
            })

    return chunks
```

- [ ] **Step 4: Run tests**

Run: `cd /Users/weiqiangyu/Downloads/wiki/backend && .venv/bin/python search/test_chunker.py`
Expected: All 6 tests pass.

---

### Task 4: Semantic Search (ChromaDB + Qwen Embeddings)

**Files:**
- Create: `backend/search/semantic.py`
- Create: `backend/search/test_semantic.py`

- [ ] **Step 1: Write test_semantic.py**

```python
import sys, os, shutil
sys.path.insert(0, os.path.join(os.path.dirname(__file__), ".."))

import tempfile
from search.semantic import SemanticSearch

tmp_dir = tempfile.mkdtemp(prefix="test_chroma_")

try:
    ss = SemanticSearch(persist_dir=tmp_dir, api_key=os.environ.get("QWEN_API_KEY", ""), embedding_model="text-embedding-v3")

    # --- Test 1: add and query wiki docs ---
    print("=== TEST 1: add + query wiki docs ===")
    ss.add_documents("wiki_docs", [
        {"id": "doc1", "text": "The tool system in Claude Code manages permission checks and tool execution.", "file_path": "docs/claude-code/entities/tool-system.md", "section": "Overview"},
        {"id": "doc2", "text": "MemoryMiddleware loads AGENTS.md files to provide persistent context.", "file_path": "docs/deepagents-wiki/entities/memory-system.md", "section": "Overview"},
        {"id": "doc3", "text": "The gateway control plane routes messages between channels and agents.", "file_path": "docs/openclaw-wiki/entities/gateway.md", "section": "Overview"},
    ])
    results = ss.query("wiki_docs", "how does the permission system work", n_results=2)
    assert len(results) > 0
    assert any("tool-system" in r["file_path"] for r in results)
    print(f"PASS — top result: {results[0]['file_path']}")

    # --- Test 2: add code docs ---
    print("\n=== TEST 2: add + query code docs ===")
    ss.add_documents("code_docs", [
        {"id": "code1", "text": "class MemoryMiddleware: loads agent memory from AGENTS.md files", "file_path": "deepagents/libs/deepagents/deepagents/middleware/memory.py", "symbol": "MemoryMiddleware"},
        {"id": "code2", "text": "def create_react_agent: creates a ReAct-style agent with tool calling", "file_path": "langgraph/prebuilt.py", "symbol": "create_react_agent"},
    ])
    results = ss.query("code_docs", "memory middleware agent context", n_results=1)
    assert len(results) > 0
    assert "memory" in results[0]["file_path"].lower()
    print(f"PASS — top result: {results[0]['file_path']}")

    # --- Test 3: empty query returns empty ---
    print("\n=== TEST 3: empty query ===")
    results = ss.query("wiki_docs", "", n_results=5)
    assert len(results) == 0
    print("PASS")

    # --- Test 4: query nonexistent collection ---
    print("\n=== TEST 4: nonexistent collection ===")
    results = ss.query("nonexistent", "test", n_results=5)
    assert len(results) == 0
    print("PASS")

    # --- Test 5: collection_exists ---
    print("\n=== TEST 5: collection_exists ===")
    assert ss.collection_exists("wiki_docs") is True
    assert ss.collection_exists("bogus") is False
    print("PASS")

    print("\n✅ All semantic search tests passed.")
finally:
    shutil.rmtree(tmp_dir)
```

- [ ] **Step 2: Run test to verify it fails**

Run: `cd /Users/weiqiangyu/Downloads/wiki/backend && .venv/bin/python search/test_semantic.py`
Expected: FAIL — `ModuleNotFoundError`

- [ ] **Step 3: Implement semantic.py**

```python
"""Semantic search using ChromaDB and Qwen text-embedding-v3."""

import logging
from typing import Any

import chromadb
import httpx

logger = logging.getLogger(__name__)

DASHSCOPE_EMBEDDING_URL = "https://dashscope.aliyuncs.com/compatible-mode/v1/embeddings"
EMBEDDING_DIMENSION = 1024  # text-embedding-v3 output dimension
BATCH_SIZE = 20  # Documents per embedding API call


class QwenEmbeddingFunction(chromadb.EmbeddingFunction[list[str]]):
    """ChromaDB-compatible embedding function using Qwen via DashScope."""

    def __init__(self, api_key: str, model: str = "text-embedding-v3") -> None:
        self._api_key = api_key
        self._model = model

    def __call__(self, input: list[str]) -> list[list[float]]:
        if not input:
            return []
        if not self._api_key:
            raise ValueError("QWEN_API_KEY is required for semantic search")

        all_embeddings: list[list[float]] = []
        for i in range(0, len(input), BATCH_SIZE):
            batch = input[i:i + BATCH_SIZE]
            resp = httpx.post(
                DASHSCOPE_EMBEDDING_URL,
                headers={
                    "Authorization": f"Bearer {self._api_key}",
                    "Content-Type": "application/json",
                },
                json={
                    "model": self._model,
                    "input": batch,
                    "encoding_format": "float",
                },
                timeout=60,
            )
            resp.raise_for_status()
            data = resp.json()
            for item in data["data"]:
                all_embeddings.append(item["embedding"])

        return all_embeddings


class SemanticSearch:
    """Semantic search backed by ChromaDB with Qwen embeddings."""

    def __init__(
        self,
        persist_dir: str,
        api_key: str,
        embedding_model: str = "text-embedding-v3",
    ) -> None:
        self._client = chromadb.PersistentClient(path=persist_dir)
        self._embed_fn = QwenEmbeddingFunction(api_key=api_key, model=embedding_model)

    def _get_or_create_collection(self, name: str) -> chromadb.Collection:
        return self._client.get_or_create_collection(
            name=name,
            embedding_function=self._embed_fn,
            metadata={"hnsw:space": "cosine"},
        )

    def collection_exists(self, name: str) -> bool:
        try:
            self._client.get_collection(name)
            return True
        except Exception:
            return False

    def add_documents(self, collection_name: str, docs: list[dict]) -> None:
        """Add documents to a collection.

        Each doc must have: id, text. Optional: file_path, section, symbol, kind.
        """
        if not docs:
            return
        collection = self._get_or_create_collection(collection_name)

        ids = [d["id"] for d in docs]
        texts = [d["text"] for d in docs]
        metadatas = []
        for d in docs:
            meta = {}
            for key in ("file_path", "section", "symbol", "kind", "start_line", "heading"):
                if key in d and d[key]:
                    meta[key] = str(d[key])
            metadatas.append(meta)

        # Upsert in batches
        for i in range(0, len(ids), BATCH_SIZE):
            batch_ids = ids[i:i + BATCH_SIZE]
            batch_texts = texts[i:i + BATCH_SIZE]
            batch_meta = metadatas[i:i + BATCH_SIZE]
            collection.upsert(ids=batch_ids, documents=batch_texts, metadatas=batch_meta)

    def query(
        self,
        collection_name: str,
        query_text: str,
        n_results: int = 10,
        where: dict | None = None,
    ) -> list[dict]:
        """Query a collection. Returns list of {text, file_path, score, ...}."""
        if not query_text.strip():
            return []

        try:
            collection = self._client.get_collection(
                name=collection_name,
                embedding_function=self._embed_fn,
            )
        except Exception:
            return []

        kwargs: dict[str, Any] = {
            "query_texts": [query_text],
            "n_results": min(n_results, collection.count() or 1),
        }
        if where:
            kwargs["where"] = where

        try:
            results = collection.query(**kwargs)
        except Exception as e:
            logger.warning("Semantic query failed: %s", e)
            return []

        if not results or not results["documents"] or not results["documents"][0]:
            return []

        output: list[dict] = []
        for i, doc in enumerate(results["documents"][0]):
            meta = results["metadatas"][0][i] if results["metadatas"] else {}
            distance = results["distances"][0][i] if results["distances"] else 1.0
            output.append({
                "text": doc,
                "file_path": meta.get("file_path", ""),
                "section": meta.get("section", ""),
                "symbol": meta.get("symbol", ""),
                "score": round(1.0 - distance, 4),  # Convert distance to similarity
                **{k: v for k, v in meta.items() if k not in ("file_path", "section", "symbol")},
            })

        return output

    def delete_collection(self, name: str) -> None:
        try:
            self._client.delete_collection(name)
        except Exception:
            pass

    def count(self, collection_name: str) -> int:
        try:
            return self._client.get_collection(collection_name).count()
        except Exception:
            return 0
```

- [ ] **Step 4: Run tests**

Run: `cd /Users/weiqiangyu/Downloads/wiki/backend && .venv/bin/python search/test_semantic.py`
Expected: All 5 tests pass. (Requires QWEN_API_KEY in .env — tests call the real embedding API.)

---

### Task 5: Lexical Search (ripgrep)

**Files:**
- Create: `backend/search/lexical.py`
- Create: `backend/search/test_lexical.py`

- [ ] **Step 1: Write test_lexical.py**

```python
import sys, os, tempfile, shutil
sys.path.insert(0, os.path.join(os.path.dirname(__file__), ".."))

from search.lexical import LexicalSearch

# Create a temp workspace with some test files
tmp = tempfile.mkdtemp(prefix="test_lexical_")
docs_dir = os.path.join(tmp, "docs", "test-wiki")
src_dir = os.path.join(tmp, "myrepo", "src")
os.makedirs(docs_dir)
os.makedirs(src_dir)

with open(os.path.join(docs_dir, "memory.md"), "w") as f:
    f.write("# Memory System\n\nThe MemoryMiddleware loads context from AGENTS.md files.\n")
with open(os.path.join(docs_dir, "tools.md"), "w") as f:
    f.write("# Tool System\n\nTools are registered and executed by the agent.\n")
with open(os.path.join(src_dir, "memory.py"), "w") as f:
    f.write('class MemoryMiddleware:\n    """Loads agent memory."""\n    pass\n')
with open(os.path.join(src_dir, "agent.py"), "w") as f:
    f.write('def run_agent():\n    """Run the agent loop."""\n    pass\n')

ls = LexicalSearch(workspace_dir=tmp)

# --- Test 1: search wiki docs ---
print("=== TEST 1: search wiki docs ===")
results = ls.search("MemoryMiddleware", search_paths=["docs/"])
assert len(results) >= 1
assert any("memory" in r["file_path"].lower() for r in results)
print(f"PASS — {len(results)} results")

# --- Test 2: search source code ---
print("\n=== TEST 2: search source code ===")
results = ls.search("MemoryMiddleware", search_paths=["myrepo/"])
assert len(results) >= 1
assert any("memory.py" in r["file_path"] for r in results)
print(f"PASS — {len(results)} results")

# --- Test 3: no results ---
print("\n=== TEST 3: no results ===")
results = ls.search("xyznonexistent123", search_paths=["docs/"])
assert len(results) == 0
print("PASS")

# --- Test 4: case insensitive ---
print("\n=== TEST 4: case insensitive ===")
results = ls.search("memorymiddleware", search_paths=["docs/"])
assert len(results) >= 1
print("PASS")

# --- Test 5: results have context ---
print("\n=== TEST 5: results have context lines ===")
results = ls.search("MemoryMiddleware", search_paths=["docs/"])
assert results[0]["text"]  # Should have surrounding context
assert results[0]["line_number"] > 0
print("PASS")

# --- Cleanup ---
shutil.rmtree(tmp)

print("\n✅ All lexical search tests passed.")
```

- [ ] **Step 2: Run test to verify it fails**

Run: `cd /Users/weiqiangyu/Downloads/wiki/backend && .venv/bin/python search/test_lexical.py`
Expected: FAIL — `ModuleNotFoundError`

- [ ] **Step 3: Implement lexical.py**

```python
"""Lexical search using ripgrep (rg) or grep fallback."""

import json
import os
import subprocess
from typing import Optional


class LexicalSearch:
    """Fast text search using ripgrep with structured output and ranking."""

    def __init__(self, workspace_dir: str) -> None:
        self.workspace_dir = workspace_dir
        self._has_rg = self._check_rg()

    def _check_rg(self) -> bool:
        try:
            subprocess.run(["rg", "--version"], capture_output=True, check=True)
            return True
        except (FileNotFoundError, subprocess.CalledProcessError):
            return False

    def search(
        self,
        query: str,
        search_paths: Optional[list[str]] = None,
        max_results: int = 15,
        file_glob: str = "",
        context_lines: int = 2,
    ) -> list[dict]:
        """Search for a text pattern in the workspace.

        Returns list of {file_path, line_number, text, score}.
        """
        if not query.strip():
            return []

        if search_paths:
            abs_paths = [os.path.join(self.workspace_dir, p) for p in search_paths]
        else:
            abs_paths = [self.workspace_dir]

        # Filter out nonexistent paths
        abs_paths = [p for p in abs_paths if os.path.exists(p)]
        if not abs_paths:
            return []

        if self._has_rg:
            return self._search_rg(query, abs_paths, max_results, file_glob, context_lines)
        return self._search_grep(query, abs_paths, max_results, file_glob, context_lines)

    def _search_rg(
        self,
        query: str,
        paths: list[str],
        max_results: int,
        file_glob: str,
        context_lines: int,
    ) -> list[dict]:
        cmd = [
            "rg", "--json",
            "--ignore-case",
            "--max-count", "5",
            f"--context={context_lines}",
            "--max-filesize", "1M",
        ]
        if file_glob:
            cmd.extend(["--glob", file_glob])

        cmd.append(query)
        cmd.extend(paths)

        try:
            res = subprocess.run(cmd, capture_output=True, text=True, timeout=10)
        except subprocess.TimeoutExpired:
            return []

        results: list[dict] = []
        context_buffer: dict[str, list[str]] = {}

        for line in res.stdout.splitlines():
            try:
                obj = json.loads(line)
            except json.JSONDecodeError:
                continue

            if obj["type"] == "match":
                data = obj["data"]
                file_path = os.path.relpath(data["path"]["text"], self.workspace_dir)
                line_number = data["line_number"]
                text = data["lines"]["text"].rstrip()

                # Collect context
                ctx_lines = context_buffer.get(file_path, [])
                ctx_lines.append(text)

                results.append({
                    "file_path": file_path,
                    "line_number": line_number,
                    "text": text,
                    "score": self._score_match(file_path, query, text),
                })

            elif obj["type"] == "context":
                data = obj["data"]
                file_path = os.path.relpath(data["path"]["text"], self.workspace_dir)
                context_buffer.setdefault(file_path, []).append(
                    data["lines"]["text"].rstrip()
                )

        # Sort by score descending, truncate
        results.sort(key=lambda r: r["score"], reverse=True)
        return results[:max_results]

    def _search_grep(
        self,
        query: str,
        paths: list[str],
        max_results: int,
        file_glob: str,
        context_lines: int,
    ) -> list[dict]:
        """Fallback to grep when ripgrep is not available."""
        cmd = ["grep", "-r", "-i", "-n", f"-C{context_lines}"]
        if file_glob:
            cmd.extend(["--include", file_glob])
        cmd.append(query)
        cmd.extend(paths)

        try:
            res = subprocess.run(cmd, capture_output=True, text=True, timeout=10)
        except subprocess.TimeoutExpired:
            return []

        if res.returncode not in (0, 1):
            return []

        results: list[dict] = []
        for line in res.stdout.splitlines():
            if not line.strip() or line.startswith("--"):
                continue
            # Parse grep output: path:line_num:text
            parts = line.split(":", 2)
            if len(parts) >= 3:
                file_path = os.path.relpath(parts[0], self.workspace_dir)
                try:
                    line_number = int(parts[1])
                except ValueError:
                    continue
                text = parts[2].rstrip()
                results.append({
                    "file_path": file_path,
                    "line_number": line_number,
                    "text": text,
                    "score": self._score_match(file_path, query, text),
                })

        results.sort(key=lambda r: r["score"], reverse=True)
        return results[:max_results]

    @staticmethod
    def _score_match(file_path: str, query: str, text: str) -> float:
        """Simple relevance scoring: filename match > text match."""
        score = 1.0
        query_lower = query.lower()
        basename = os.path.basename(file_path).lower()

        # Boost: query appears in filename
        if query_lower.replace(" ", "-") in basename or query_lower.replace(" ", "_") in basename:
            score += 5.0

        # Boost: exact case match in text
        if query in text:
            score += 2.0

        # Boost: wiki docs over source (more curated)
        if file_path.startswith("docs/"):
            score += 1.0

        return score
```

- [ ] **Step 4: Run tests**

Run: `cd /Users/weiqiangyu/Downloads/wiki/backend && .venv/bin/python search/test_lexical.py`
Expected: All 5 tests pass. (Uses grep fallback if rg not installed.)

---

### Task 6: Symbol Index (tree-sitter)

**Files:**
- Create: `backend/search/symbols.py`
- Create: `backend/search/test_symbols.py`

- [ ] **Step 1: Write test_symbols.py**

```python
import sys, os
sys.path.insert(0, os.path.join(os.path.dirname(__file__), ".."))

from search.symbols import SymbolExtractor

se = SymbolExtractor()

# --- Test 1: Extract Python symbols ---
print("=== TEST 1: Python symbols ===")
py_code = '''
"""Module docstring."""

import os

def hello(name: str) -> str:
    """Say hello to someone."""
    return f"Hello, {name}"

class Greeter:
    """A class that greets people."""

    def greet(self, name: str) -> str:
        """Greet a specific person."""
        return f"Hi, {name}"

    async def greet_async(self, name: str) -> str:
        """Async greet."""
        return f"Hi, {name}"

CONSTANT = 42
'''
symbols = se.extract("test.py", py_code, "python")
names = {s["name"] for s in symbols}
assert "hello" in names
assert "Greeter" in names
assert "greet" in names
assert "greet_async" in names
print(f"PASS — found: {names}")

# --- Test 2: Extract TypeScript symbols ---
print("\n=== TEST 2: TypeScript symbols ===")
ts_code = '''
export interface ToolConfig {
  name: string;
  handler: Function;
}

export class ToolRegistry {
  /** Registers a tool. */
  register(config: ToolConfig): void {}
}

export function createAgent(name: string): Agent {
  return new Agent(name);
}

export type AgentState = "idle" | "running";
'''
symbols = se.extract("tools.ts", ts_code, "typescript")
names = {s["name"] for s in symbols}
assert "ToolConfig" in names
assert "ToolRegistry" in names
assert "createAgent" in names
print(f"PASS — found: {names}")

# --- Test 3: Symbol has metadata ---
print("\n=== TEST 3: symbol metadata ===")
hello_sym = next(s for s in se.extract("t.py", py_code, "python") if s["name"] == "hello")
assert hello_sym["kind"] in ("function", "def")
assert hello_sym["file_path"] == "t.py"
assert hello_sym["start_line"] > 0
assert "signature" in hello_sym
print(f"PASS — {hello_sym}")

# --- Test 4: Empty file ---
print("\n=== TEST 4: empty file ===")
symbols = se.extract("empty.py", "", "python")
assert symbols == []
print("PASS")

# --- Test 5: Unsupported language fallback ---
print("\n=== TEST 5: unsupported language ===")
symbols = se.extract("test.rb", "def hello; end", "ruby")
assert symbols == []
print("PASS")

print("\n✅ All symbol extraction tests passed.")
```

- [ ] **Step 2: Run test to verify it fails**

Run: `cd /Users/weiqiangyu/Downloads/wiki/backend && .venv/bin/python search/test_symbols.py`
Expected: FAIL

- [ ] **Step 3: Implement symbols.py**

```python
"""Symbol extraction using tree-sitter for Python, TypeScript, and C#."""

import logging
from typing import Any

import tree_sitter

logger = logging.getLogger(__name__)

# Lazy-loaded parsers
_PARSERS: dict[str, tree_sitter.Parser] = {}

_LANGUAGE_MAP = {
    "python": "tree_sitter_python",
    "typescript": "tree_sitter_typescript",
    "tsx": "tree_sitter_typescript",
    "javascript": "tree_sitter_typescript",  # TS parser handles JS too
    "csharp": "tree_sitter_c_sharp",
}

# tree-sitter node types that represent definitions
_DEFINITION_TYPES = {
    "python": {
        "function_definition", "class_definition",
    },
    "typescript": {
        "function_declaration", "class_declaration", "interface_declaration",
        "type_alias_declaration", "enum_declaration", "method_definition",
    },
    "csharp": {
        "class_declaration", "struct_declaration", "interface_declaration",
        "enum_declaration", "method_declaration", "record_declaration",
    },
}


def _get_parser(language: str) -> tree_sitter.Parser | None:
    """Get or create a tree-sitter parser for the given language."""
    if language in _PARSERS:
        return _PARSERS[language]

    module_name = _LANGUAGE_MAP.get(language)
    if not module_name:
        return None

    try:
        import importlib
        mod = importlib.import_module(module_name)
        # tree-sitter-python exposes language()
        # tree-sitter-typescript exposes language_typescript() and language_tsx()
        if hasattr(mod, "language"):
            lang = tree_sitter.Language(mod.language())
        elif language in ("typescript", "javascript") and hasattr(mod, "language_typescript"):
            lang = tree_sitter.Language(mod.language_typescript())
        elif language == "tsx" and hasattr(mod, "language_tsx"):
            lang = tree_sitter.Language(mod.language_tsx())
        else:
            return None

        parser = tree_sitter.Parser(lang)
        _PARSERS[language] = parser
        return parser
    except Exception as e:
        logger.warning("Failed to load tree-sitter parser for %s: %s", language, e)
        return None


class SymbolExtractor:
    """Extracts function, class, and type definitions from source code."""

    def extract(
        self,
        file_path: str,
        content: str,
        language: str,
    ) -> list[dict]:
        """Extract symbols from source code.

        Returns list of {name, kind, file_path, start_line, end_line, signature, docstring}.
        """
        if not content.strip():
            return []

        parser = _get_parser(language)
        if not parser:
            return []

        try:
            tree = parser.parse(content.encode("utf-8"))
        except Exception as e:
            logger.warning("tree-sitter parse failed for %s: %s", file_path, e)
            return []

        def_types = _DEFINITION_TYPES.get(language, set())
        symbols: list[dict] = []
        self._walk(tree.root_node, file_path, language, def_types, content, symbols)
        return symbols

    def _walk(
        self,
        node: Any,
        file_path: str,
        language: str,
        def_types: set[str],
        source: str,
        symbols: list[dict],
    ) -> None:
        if node.type in def_types:
            sym = self._extract_symbol(node, file_path, language, source)
            if sym:
                symbols.append(sym)

        for child in node.children:
            self._walk(child, file_path, language, def_types, source, symbols)

    def _extract_symbol(
        self,
        node: Any,
        file_path: str,
        language: str,
        source: str,
    ) -> dict | None:
        """Extract a symbol dict from a tree-sitter definition node."""
        name = ""
        kind = node.type.replace("_declaration", "").replace("_definition", "")

        # Find the name node
        for child in node.children:
            if child.type in ("identifier", "name", "type_identifier", "property_identifier"):
                name = source[child.start_byte:child.end_byte]
                break

        if not name:
            return None

        # Extract first line as signature
        start_line = node.start_point[0] + 1
        end_line = node.end_point[0] + 1
        lines = source.splitlines()
        signature = lines[start_line - 1].strip() if start_line <= len(lines) else ""

        # Extract docstring (next string literal after definition line)
        docstring = ""
        if language == "python":
            for child in node.children:
                if child.type == "block":
                    for stmt in child.children:
                        if stmt.type == "expression_statement":
                            for expr in stmt.children:
                                if expr.type == "string":
                                    docstring = source[expr.start_byte:expr.end_byte].strip("\"' \n")
                                    break
                        break
                    break

        return {
            "name": name,
            "kind": kind,
            "file_path": file_path,
            "start_line": start_line,
            "end_line": end_line,
            "signature": signature,
            "docstring": docstring[:200],
        }
```

- [ ] **Step 4: Run tests**

Run: `cd /Users/weiqiangyu/Downloads/wiki/backend && .venv/bin/python search/test_symbols.py`
Expected: All 5 tests pass.

---

### Task 7: Index Builder

**Files:**
- Create: `backend/search/indexer.py`
- Modify: `backend/.gitignore` (ensure `data/chromadb/` is ignored)

- [ ] **Step 1: Implement indexer.py**

```python
"""Index builder for semantic search and symbol extraction.

Runs at backend startup. Uses file checksums for incremental updates.
"""

import hashlib
import json
import logging
import os
import time

from search.chunker import chunk_markdown, chunk_source_file
from search.registry import RepoRegistry, repo_registry
from search.semantic import SemanticSearch
from search.symbols import SymbolExtractor

logger = logging.getLogger(__name__)

MANIFEST_FILE = "index_manifest.json"

# File extensions to index per language
_EXTENSIONS = {
    "python": [".py"],
    "typescript": [".ts", ".tsx"],
    "javascript": [".js", ".jsx"],
    "csharp": [".cs"],
}

# Directories to skip during source code indexing
_SKIP_DIRS = {
    "node_modules", ".git", "__pycache__", ".venv", "venv",
    "dist", "build", ".next", ".nuxt", "coverage", ".tox",
    "site-packages", "egg-info",
}

# Maximum file size to index (bytes)
_MAX_FILE_SIZE = 500_000  # 500 KB


def _file_hash(path: str) -> str:
    h = hashlib.md5()
    with open(path, "rb") as f:
        for chunk in iter(lambda: f.read(8192), b""):
            h.update(chunk)
    return h.hexdigest()


def _lang_for_ext(ext: str) -> str:
    for lang, exts in _EXTENSIONS.items():
        if ext in exts:
            return lang
    return ""


class IndexBuilder:
    """Builds and maintains the search index."""

    def __init__(
        self,
        workspace_dir: str,
        semantic: SemanticSearch,
        registry: RepoRegistry | None = None,
    ) -> None:
        self.workspace_dir = workspace_dir
        self.semantic = semantic
        self.registry = registry or repo_registry
        self._extractor = SymbolExtractor()
        self._manifest_path = os.path.join(
            workspace_dir, "backend", "data", MANIFEST_FILE,
        )
        self._manifest: dict[str, str] = self._load_manifest()
        self.stats: dict[str, int] = {
            "wiki_chunks": 0,
            "code_chunks": 0,
            "symbols": 0,
            "files_scanned": 0,
            "files_skipped": 0,
        }

    def _load_manifest(self) -> dict[str, str]:
        if os.path.exists(self._manifest_path):
            with open(self._manifest_path) as f:
                return json.load(f)
        return {}

    def _save_manifest(self) -> None:
        os.makedirs(os.path.dirname(self._manifest_path), exist_ok=True)
        with open(self._manifest_path, "w") as f:
            json.dump(self._manifest, f)

    def _is_changed(self, path: str) -> bool:
        current_hash = _file_hash(path)
        old_hash = self._manifest.get(path)
        if current_hash != old_hash:
            self._manifest[path] = current_hash
            return True
        return False

    def build(self) -> dict[str, int]:
        """Build or update the full index. Returns stats."""
        start = time.time()
        logger.info("Starting index build...")

        self._index_wiki_docs()
        self._index_source_code()

        self._save_manifest()
        elapsed = time.time() - start
        logger.info(
            "Index build complete in %.1fs: %d wiki chunks, %d code chunks, %d symbols",
            elapsed, self.stats["wiki_chunks"], self.stats["code_chunks"], self.stats["symbols"],
        )
        return self.stats

    def _index_wiki_docs(self) -> None:
        """Index all wiki markdown files."""
        docs_dir = os.path.join(self.workspace_dir, "docs")
        all_chunks: list[dict] = []

        for root, dirs, files in os.walk(docs_dir):
            dirs[:] = [d for d in dirs if d not in _SKIP_DIRS]
            for fname in files:
                if not fname.endswith(".md"):
                    continue
                full_path = os.path.join(root, fname)
                rel_path = os.path.relpath(full_path, self.workspace_dir)

                if not self._is_changed(full_path):
                    self.stats["files_skipped"] += 1
                    continue

                self.stats["files_scanned"] += 1
                try:
                    with open(full_path, encoding="utf-8") as f:
                        content = f.read()
                    chunks = chunk_markdown(content, file_path=rel_path)
                    for i, c in enumerate(chunks):
                        c["id"] = f"wiki:{rel_path}:{i}"
                    all_chunks.extend(chunks)
                except Exception as e:
                    logger.warning("Failed to index %s: %s", rel_path, e)

        if all_chunks:
            self.semantic.add_documents("wiki_docs", all_chunks)
            self.stats["wiki_chunks"] = len(all_chunks)

    def _index_source_code(self) -> None:
        """Index source code files: docstrings, code chunks, and symbols."""
        code_chunks: list[dict] = []
        symbol_docs: list[dict] = []

        for repo in self.registry.repos:
            repo_dir = os.path.join(self.workspace_dir, repo.source_dir)
            if not os.path.isdir(repo_dir):
                continue

            for root, dirs, files in os.walk(repo_dir):
                dirs[:] = [d for d in dirs if d not in _SKIP_DIRS]
                for fname in files:
                    ext = os.path.splitext(fname)[1]
                    lang = _lang_for_ext(ext)
                    if not lang:
                        continue

                    full_path = os.path.join(root, fname)
                    if os.path.getsize(full_path) > _MAX_FILE_SIZE:
                        continue

                    rel_path = os.path.relpath(full_path, self.workspace_dir)

                    if not self._is_changed(full_path):
                        self.stats["files_skipped"] += 1
                        continue

                    self.stats["files_scanned"] += 1
                    try:
                        with open(full_path, encoding="utf-8", errors="replace") as f:
                            content = f.read()
                    except Exception:
                        continue

                    # Code chunks
                    chunks = chunk_source_file(content, file_path=rel_path, language=lang)
                    for i, c in enumerate(chunks):
                        c["id"] = f"code:{rel_path}:{i}"
                    code_chunks.extend(chunks)

                    # Symbol extraction
                    symbols = self._extractor.extract(rel_path, content, lang)
                    for s in symbols:
                        s["id"] = f"sym:{rel_path}:{s['name']}"
                        s["text"] = f"{s['name']} ({s['kind']}) — {s['signature']}"
                        if s.get("docstring"):
                            s["text"] += f" — {s['docstring']}"
                    symbol_docs.extend(symbols)

        if code_chunks:
            self.semantic.add_documents("code_docs", code_chunks)
            self.stats["code_chunks"] = len(code_chunks)

        if symbol_docs:
            self.semantic.add_documents("symbols", symbol_docs)
            self.stats["symbols"] = len(symbol_docs)
```

- [ ] **Step 2: Create .gitignore for data directory**

```bash
mkdir -p /Users/weiqiangyu/Downloads/wiki/backend/data
echo "chromadb/" > /Users/weiqiangyu/Downloads/wiki/backend/data/.gitignore
echo "index_manifest.json" >> /Users/weiqiangyu/Downloads/wiki/backend/data/.gitignore
```

- [ ] **Step 3: Quick smoke test**

Run: `cd /Users/weiqiangyu/Downloads/wiki/backend && .venv/bin/python -c "from search.indexer import IndexBuilder; print('Import OK')"`
Expected: Prints `Import OK`

---

### Task 8: Search Orchestrator

**Files:**
- Create: `backend/search/orchestrator.py`
- Create: `backend/search/test_orchestrator.py`
- Modify: `backend/search/__init__.py`

- [ ] **Step 1: Write test_orchestrator.py**

```python
import sys, os, tempfile, shutil
sys.path.insert(0, os.path.join(os.path.dirname(__file__), ".."))

from search.orchestrator import SearchOrchestrator, classify_query

# --- Test 1: classify_query ---
print("=== TEST 1: classify_query ===")
assert classify_query("MemoryMiddleware") == "symbol"
assert classify_query("how does the tool system work") == "concept"
assert classify_query("ERROR: file not found") == "exact"
assert classify_query("create_react_agent function") == "symbol"
assert classify_query("what is the architecture of the agent loop") == "concept"
print("PASS")

# --- Test 2: orchestrator init ---
print("\n=== TEST 2: orchestrator init ===")
tmp = tempfile.mkdtemp()
try:
    orch = SearchOrchestrator(
        workspace_dir=tmp,
        chroma_dir=os.path.join(tmp, "chroma"),
        qwen_api_key="",
    )
    assert orch is not None
    print("PASS")
finally:
    shutil.rmtree(tmp)

# --- Test 3: format_results truncation ---
print("\n=== TEST 3: format_results ===")
from search.orchestrator import format_results
results = [
    {"file_path": "docs/test.md", "text": "x" * 500, "score": 0.9, "line_number": 10},
    {"file_path": "docs/other.md", "text": "y" * 500, "score": 0.5, "line_number": 20},
]
formatted = format_results(results, max_chars=600)
assert len(formatted) < 700
assert "docs/test.md" in formatted
print("PASS")

# --- Test 4: format_results empty ---
print("\n=== TEST 4: format_results empty ===")
assert format_results([], max_chars=1000) == "No results found."
print("PASS")

print("\n✅ All orchestrator tests passed.")
```

- [ ] **Step 2: Run test to verify it fails**

Run: `cd /Users/weiqiangyu/Downloads/wiki/backend && .venv/bin/python search/test_orchestrator.py`
Expected: FAIL

- [ ] **Step 3: Implement orchestrator.py**

```python
"""Search orchestrator: repo targeting, tier escalation, result ranking."""

import logging
import re
from typing import Optional

from search.lexical import LexicalSearch
from search.registry import RepoMeta, RepoRegistry, repo_registry
from search.semantic import SemanticSearch

logger = logging.getLogger(__name__)


def classify_query(query: str) -> str:
    """Classify a query as 'symbol', 'concept', or 'exact'.

    - symbol: looks like a code identifier (CamelCase, snake_case, has dots)
    - exact: looks like an error message or specific string
    - concept: natural language question
    """
    # Symbol patterns: CamelCase, snake_case with >1 segment, dotted path
    if re.match(r"^[A-Z][a-zA-Z0-9]*(?:[A-Z][a-z]+)+$", query.strip()):
        return "symbol"
    if re.match(r"^[a-z_][a-z0-9_]*(?:_[a-z0-9]+)+$", query.strip()):
        return "symbol"
    if "." in query and not " " in query:
        return "symbol"
    # Check for function-like words
    words = query.lower().split()
    if any(w in ("function", "class", "method", "def", "interface", "type") for w in words):
        # If the query also has a potential identifier
        for w in query.split():
            if re.match(r"^[A-Z][a-zA-Z0-9]+$", w) or re.match(r"^[a-z_]+[A-Z]", w):
                return "symbol"
    # Exact string patterns (error messages, paths, quoted strings)
    if query.startswith(("ERROR", "Error", "error")) or '"' in query or "'" in query:
        return "exact"
    if "/" in query and not " " in query:
        return "exact"
    return "concept"


def format_results(results: list[dict], max_chars: int = 3000) -> str:
    """Format search results into a concise string for the agent."""
    if not results:
        return "No results found."

    parts: list[str] = []
    total = 0
    for r in results:
        file_path = r.get("file_path", "unknown")
        text = r.get("text", "").strip()
        line = r.get("line_number") or r.get("start_line", "")
        score = r.get("score", 0)
        symbol = r.get("symbol", "")

        # Truncate individual result text
        if len(text) > 300:
            text = text[:300] + "…"

        if symbol:
            entry = f"**{file_path}** (L{line}) — `{symbol}`\n{text}"
        elif line:
            entry = f"**{file_path}** (L{line})\n{text}"
        else:
            entry = f"**{file_path}**\n{text}"

        if total + len(entry) > max_chars:
            parts.append(f"\n… ({len(results) - len(parts)} more results truncated)")
            break
        parts.append(entry)
        total += len(entry)

    return "\n\n".join(parts)


class SearchOrchestrator:
    """Orchestrates multi-tier search with repo targeting and escalation."""

    def __init__(
        self,
        workspace_dir: str,
        chroma_dir: str,
        qwen_api_key: str,
        embedding_model: str = "text-embedding-v3",
        registry: RepoRegistry | None = None,
    ) -> None:
        self.workspace_dir = workspace_dir
        self.registry = registry or repo_registry
        self.lexical = LexicalSearch(workspace_dir)
        self.semantic = SemanticSearch(
            persist_dir=chroma_dir,
            api_key=qwen_api_key,
            embedding_model=embedding_model,
        )
        self._ready = False

    @property
    def is_ready(self) -> bool:
        return self._ready

    def mark_ready(self) -> None:
        self._ready = True

    def search(
        self,
        query: str,
        scope: str = "auto",
        page_url: str = "",
        max_results: int = 10,
    ) -> str:
        """Run a multi-tier search and return formatted results."""
        if not query.strip():
            return "No results found."

        # Determine target repos
        namespace = scope if scope not in ("auto", "wiki", "code") else ""
        targets = self.registry.target(query, page_url=page_url, namespace=namespace)

        query_type = classify_query(query)
        all_results: list[dict] = []

        # Build search paths based on scope
        if scope == "wiki":
            search_paths = [r.wiki_dir for r in targets]
        elif scope == "code":
            search_paths = [r.source_dir for r in targets]
        elif namespace:
            t = targets[0] if targets else None
            search_paths = [t.wiki_dir, t.source_dir] if t else ["docs/"]
        else:
            search_paths = []
            for t in targets[:3]:  # Limit to top 3 repos
                search_paths.extend([t.wiki_dir, t.source_dir])

        # --- Tier 1: Lexical (always runs for symbol/exact, first attempt for concept) ---
        if query_type in ("symbol", "exact"):
            lexical_results = self.lexical.search(query, search_paths=search_paths, max_results=max_results)
            all_results.extend(lexical_results)

        elif query_type == "concept":
            lexical_results = self.lexical.search(query, search_paths=search_paths, max_results=5)
            all_results.extend(lexical_results)

        # --- Tier 2: Semantic (for concepts, or when lexical has few results) ---
        if self._ready and (query_type == "concept" or len(all_results) < 3):
            semantic_results = self._semantic_search(query, scope, targets, max_results)
            all_results.extend(semantic_results)

        # --- Tier 3: Symbol lookup (for symbol queries) ---
        if self._ready and query_type == "symbol":
            symbol_results = self.semantic.query("symbols", query, n_results=5)
            all_results.extend(symbol_results)

        # Deduplicate by file_path
        seen: set[str] = set()
        unique: list[dict] = []
        for r in all_results:
            key = f"{r.get('file_path', '')}:{r.get('line_number', r.get('start_line', ''))}"
            if key not in seen:
                seen.add(key)
                unique.append(r)

        # Sort by score
        unique.sort(key=lambda r: r.get("score", 0), reverse=True)
        return format_results(unique[:max_results])

    def find_symbol(self, name: str, namespace: str = "") -> str:
        """Look up a symbol by name in the symbol index."""
        if not self._ready:
            return "Search index is still building. Please try again in a moment."

        where = {"file_path": {"$contains": namespace}} if namespace else None
        results = self.semantic.query("symbols", name, n_results=10, where=where)

        if not results:
            # Fallback to lexical
            paths = []
            if namespace:
                repo = self.registry.get_by_namespace(namespace)
                if repo:
                    paths = [repo.source_dir]
            results = self.lexical.search(
                f"def {name}|class {name}|function {name}|interface {name}",
                search_paths=paths or None,
                max_results=5,
            )

        return format_results(results)

    def _semantic_search(
        self,
        query: str,
        scope: str,
        targets: list[RepoMeta],
        max_results: int,
    ) -> list[dict]:
        """Run semantic search across appropriate collections."""
        results: list[dict] = []

        if scope in ("auto", "wiki"):
            results.extend(
                self.semantic.query("wiki_docs", query, n_results=max_results)
            )

        if scope in ("auto", "code"):
            results.extend(
                self.semantic.query("code_docs", query, n_results=max_results)
            )

        return results
```

- [ ] **Step 4: Update `search/__init__.py`**

```python
"""Search service layer for the wiki agent."""

from search.orchestrator import SearchOrchestrator, format_results, classify_query
from search.indexer import IndexBuilder
from search.registry import repo_registry, RepoRegistry, RepoMeta
from search.semantic import SemanticSearch
from search.lexical import LexicalSearch
from search.symbols import SymbolExtractor

__all__ = [
    "SearchOrchestrator",
    "IndexBuilder",
    "SemanticSearch",
    "LexicalSearch",
    "SymbolExtractor",
    "repo_registry",
    "RepoRegistry",
    "RepoMeta",
    "format_results",
    "classify_query",
]
```

- [ ] **Step 5: Run tests**

Run: `cd /Users/weiqiangyu/Downloads/wiki/backend && .venv/bin/python search/test_orchestrator.py`
Expected: All 4 tests pass.

---

### Task 9: Agent Tools & System Prompt

**Files:**
- Create: `backend/search_tools.py`
- Modify: `backend/agent.py:128-168` (remove `search_knowledge_base`, add imports)
- Modify: `backend/agent.py:268` (update tools list)
- Modify: `backend/agent.py:292-336` (update system prompt)
- Create: `backend/test_search_tools.py`

- [ ] **Step 1: Write test_search_tools.py**

```python
import sys, os
sys.path.insert(0, os.path.dirname(__file__))

# --- Test 1: Tool imports ---
print("=== TEST 1: tool imports ===")
from search_tools import smart_search, find_symbol, read_code_section
assert callable(smart_search.invoke)
assert callable(find_symbol.invoke)
assert callable(read_code_section.invoke)
print("PASS")

# --- Test 2: read_code_section with line range ---
print("\n=== TEST 2: read_code_section line range ===")
# Read lines 1-5 of agent.py
result = read_code_section.invoke({
    "file_path": "backend/agent.py",
    "start_line": 1,
    "end_line": 5,
})
assert "import" in result
assert len(result.splitlines()) <= 10  # Shouldn't return the whole file
print("PASS")

# --- Test 3: read_code_section bad path ---
print("\n=== TEST 3: read_code_section bad path ===")
result = read_code_section.invoke({"file_path": "nonexistent.py"})
assert "Error" in result or "not found" in result.lower() or "does not exist" in result.lower()
print("PASS")

# --- Test 4: smart_search invocable ---
print("\n=== TEST 4: smart_search invocable ===")
result = smart_search.invoke({"query": "MemoryMiddleware", "scope": "wiki"})
assert isinstance(result, str)
print(f"PASS — returned {len(result)} chars")

# --- Test 5: tools list updated in agent ---
print("\n=== TEST 5: agent tools list ===")
from agent import tools
names = [t.name for t in tools]
assert "smart_search" in names
assert "find_symbol" in names
assert "read_code_section" in names
assert "search_knowledge_base" not in names
print(f"PASS — tools: {names}")

print("\n✅ All search tool tests passed.")
```

- [ ] **Step 2: Create search_tools.py**

```python
"""LangChain tool wrappers for the search service layer."""

import os

from langchain_core.tools import tool

ROOT_DIR = os.path.abspath(os.path.join(os.path.dirname(__file__), ".."))

# Lazy-init orchestrator (set by main.py at startup)
_orchestrator = None


def get_orchestrator():
    global _orchestrator
    if _orchestrator is None:
        from search.orchestrator import SearchOrchestrator
        from security import settings
        from agent import config

        chroma_dir = os.path.join(os.path.dirname(__file__), "data", "chromadb")
        _orchestrator = SearchOrchestrator(
            workspace_dir=ROOT_DIR,
            chroma_dir=chroma_dir,
            qwen_api_key=config.qwen_api_key,
            embedding_model=settings.qwen_embedding_model,
        )
    return _orchestrator


def set_orchestrator(orch) -> None:
    global _orchestrator
    _orchestrator = orch


@tool
def smart_search(query: str, scope: str = "auto") -> str:
    """Search across wiki documentation and source code repositories.

    The search automatically determines which repos to target and
    which strategy to use (lexical text search, semantic similarity,
    or symbol lookup) based on the query.

    Args:
        query: Natural language question, code identifier, or search term.
        scope: Search scope. Options:
            - "auto" (recommended) — search both wiki and source code
            - "wiki" — search only wiki documentation
            - "code" — search only source code
            - A namespace like "claude-code" — search that repo only
    """
    orch = get_orchestrator()
    if not orch:
        return "Error: Search system not initialized."
    try:
        return orch.search(query=query, scope=scope)
    except Exception as e:
        return f"Search error: {e}"


@tool
def find_symbol(name: str, namespace: str = "") -> str:
    """Find a function, class, interface, or type definition by name.

    Returns the definition location, signature, and docstring.
    Use this when you know the exact name of a code symbol.

    Args:
        name: The symbol name (e.g. 'MemoryMiddleware', 'create_react_agent').
        namespace: Optional namespace to limit search (e.g. 'deepagents').
    """
    orch = get_orchestrator()
    if not orch:
        return "Error: Search system not initialized."
    try:
        return orch.find_symbol(name=name, namespace=namespace)
    except Exception as e:
        return f"Symbol lookup error: {e}"


@tool
def read_code_section(
    file_path: str,
    symbol: str = "",
    start_line: int = 0,
    end_line: int = 0,
) -> str:
    """Read a specific section of a source file. More token-efficient
    than reading the entire file when you only need part of it.

    Specify either:
    - A line range (start_line and end_line), or
    - A symbol name (reads that function/class definition)
    If neither is specified, reads the first 100 lines.

    Args:
        file_path: Path relative to workspace root (e.g. 'deepagents/libs/deepagents/deepagents/graph.py')
        symbol: Optional symbol name to extract (e.g. 'GraphFactory')
        start_line: Start line number (1-indexed)
        end_line: End line number (1-indexed, inclusive)
    """
    target = os.path.abspath(os.path.join(ROOT_DIR, file_path))
    if not target.startswith(ROOT_DIR):
        return "Error: Access denied."
    if not os.path.exists(target):
        return f"Error: File '{file_path}' does not exist."

    try:
        with open(target, encoding="utf-8", errors="replace") as f:
            lines = f.readlines()
    except Exception as e:
        return f"Error reading file: {e}"

    total = len(lines)

    if symbol:
        # Find the symbol in the file using simple regex
        import re
        pattern = re.compile(
            rf"^(?:export\s+)?(?:async\s+)?(?:def|class|function|interface|type|enum)\s+{re.escape(symbol)}\b"
        )
        start_idx = None
        for i, line in enumerate(lines):
            if pattern.match(line.strip()):
                start_idx = i
                break

        if start_idx is None:
            return f"Symbol '{symbol}' not found in {file_path}. File has {total} lines."

        # Find the end of the definition (next top-level definition or 200 lines)
        end_idx = min(start_idx + 200, total)
        for i in range(start_idx + 1, end_idx):
            if i < total and lines[i].strip() and not lines[i][0].isspace() and pattern.match(lines[i].strip()):
                end_idx = i
                break

        content = "".join(lines[start_idx:end_idx])
        return f"# {file_path} — `{symbol}` (L{start_idx + 1}-{end_idx})\n\n```\n{content}```"

    if start_line > 0:
        s = max(0, start_line - 1)
        e = min(total, end_line if end_line > 0 else s + 100)
        content = "".join(lines[s:e])
        return f"# {file_path} (L{s + 1}-{e} of {total})\n\n```\n{content}```"

    # Default: first 100 lines
    content = "".join(lines[:100])
    suffix = f"\n… ({total - 100} more lines)" if total > 100 else ""
    return f"# {file_path} (L1-{min(100, total)} of {total})\n\n```\n{content}```{suffix}"
```

- [ ] **Step 3: Update agent.py — remove search_knowledge_base, add new tools**

Remove the `search_knowledge_base` function (lines 127-167 in current file) and update the tools list and imports.

At the top of agent.py, after the existing imports, add:

```python
from search_tools import smart_search, find_symbol, read_code_section
```

Replace the tools list:

```python
tools = [read_workspace_file, read_source_file, list_wiki_pages, smart_search, find_symbol, read_code_section, propose_doc_change]
```

- [ ] **Step 4: Update system prompt in agent.py**

Replace the "When answering:" section in `build_system_prompt()` with:

```python
    prompt += """

When answering:
1. If the user asks about a specific symbol (function, class, type), use `find_symbol` first.
2. Use `smart_search` to find relevant documentation and code. Start with scope="auto".
   Only specify a namespace scope if you are confident it is a single-repo question.
3. Use `read_code_section` to read specific functions/classes from source files.
   Prefer this over `read_source_file` when you only need part of a file.
4. Use `read_source_file` or `read_workspace_file` only when you need an entire file.
5. Do NOT call `list_wiki_pages` + `read_workspace_file` in a loop — use `smart_search` instead.
6. If your first search returns good results, STOP searching. Do not redundantly verify.
7. Provide clear, accurate answers grounded in the documentation and source code.
8. End your response with a "**Sources:**" section listing each file path you read."""
```

- [ ] **Step 5: Run tests**

Run: `cd /Users/weiqiangyu/Downloads/wiki/backend && .venv/bin/python test_search_tools.py`
Expected: All 5 tests pass.

---

### Task 10: Startup Integration & Status Endpoint

**Files:**
- Modify: `backend/main.py`
- Create: `backend/test_search_integration.py`

- [ ] **Step 1: Add startup indexing to main.py**

Add these imports at the top of main.py:

```python
import asyncio
import logging
import threading

from search_tools import get_orchestrator, set_orchestrator
from search.orchestrator import SearchOrchestrator
from search.indexer import IndexBuilder
```

Add after the `app = FastAPI(...)` line:

```python
logger = logging.getLogger("search")

_index_status = {"state": "idle", "stats": {}}


def _build_index_thread():
    """Build search index in a background thread at startup."""
    _index_status["state"] = "building"
    try:
        orch = get_orchestrator()
        builder = IndexBuilder(
            workspace_dir=ROOT_DIR,
            semantic=orch.semantic,
        )
        stats = builder.build()
        orch.mark_ready()
        _index_status["state"] = "ready"
        _index_status["stats"] = stats
        logger.info("Search index ready: %s", stats)
    except Exception as e:
        _index_status["state"] = "error"
        _index_status["stats"] = {"error": str(e)}
        logger.error("Index build failed: %s", e)


@app.on_event("startup")
async def startup_build_index():
    """Start index building in a background thread."""
    from agent import ROOT_DIR
    thread = threading.Thread(target=_build_index_thread, daemon=True)
    thread.start()
```

Add this import at the top (already imported ROOT_DIR from agent):

```python
from agent import run_agent, run_agent_stream, ROOT_DIR
```

Add the status endpoint:

```python
@app.get("/search/status")
def search_status(current_user: str = Depends(get_current_user)):
    """Check the search index build status."""
    return _index_status
```

- [ ] **Step 2: Create test_search_integration.py**

```python
"""Integration test: verify full search pipeline works end-to-end."""
import sys, os, tempfile, shutil
sys.path.insert(0, os.path.dirname(__file__))

from search.orchestrator import SearchOrchestrator, classify_query
from search.indexer import IndexBuilder
from search.registry import repo_registry
from search.semantic import SemanticSearch
from agent import tools, ROOT_DIR

print("=" * 60)
print("SEARCH INTEGRATION TEST")
print("=" * 60)

# --- Test 1: Agent tools are correct ---
print("\n--- Test 1: Agent has correct tools ---")
names = [t.name for t in tools]
assert "smart_search" in names, f"Missing smart_search! Tools: {names}"
assert "find_symbol" in names
assert "read_code_section" in names
assert "search_knowledge_base" not in names, "Old tool still present!"
print(f"PASS — tools: {names}")

# --- Test 2: Orchestrator with real workspace (lexical only) ---
print("\n--- Test 2: Lexical search on real workspace ---")
from search.lexical import LexicalSearch
ls = LexicalSearch(workspace_dir=ROOT_DIR)
results = ls.search("MemoryMiddleware", search_paths=["docs/"])
assert len(results) > 0
print(f"PASS — {len(results)} results for 'MemoryMiddleware' in docs/")

# --- Test 3: classify_query works correctly ---
print("\n--- Test 3: Query classification ---")
assert classify_query("MemoryMiddleware") == "symbol"
assert classify_query("how does routing work") == "concept"
assert classify_query("ERROR: connection refused") == "exact"
print("PASS")

# --- Test 4: Registry targeting from page context ---
print("\n--- Test 4: Registry targeting ---")
targets = repo_registry.target("tool permissions", page_url="/claude-code/entities/tool-system/")
assert targets[0].namespace == "claude-code"
print(f"PASS — targeted {targets[0].namespace}")

# --- Test 5: read_code_section works ---
print("\n--- Test 5: read_code_section ---")
from search_tools import read_code_section
result = read_code_section.invoke({"file_path": "backend/agent.py", "start_line": 1, "end_line": 10})
assert "import" in result
print("PASS")

print("\n" + "=" * 60)
print("✅ ALL SEARCH INTEGRATION TESTS PASSED")
print("=" * 60)
```

- [ ] **Step 3: Run integration tests**

Run: `cd /Users/weiqiangyu/Downloads/wiki/backend && .venv/bin/python test_search_integration.py`
Expected: All 5 tests pass.

- [ ] **Step 4: Run all existing tests to verify no regressions**

Run: `cd /Users/weiqiangyu/Downloads/wiki/backend && .venv/bin/python test_agent.py && .venv/bin/python test_proposals.py && .venv/bin/python test_git_workflow.py && .venv/bin/python test_integration.py`
Expected: All previous tests still pass.

---

## Self-Review Checklist

**Spec coverage:**
- ✅ RepoRegistry with metadata and targeting — Task 2
- ✅ Lexical search (ripgrep/grep) — Task 5
- ✅ Semantic search (ChromaDB + Qwen) — Task 4
- ✅ Symbol index (tree-sitter) — Task 6
- ✅ Chunking strategy — Task 3
- ✅ Index builder (startup + incremental) — Task 7
- ✅ SearchOrchestrator (escalation, ranking) — Task 8
- ✅ Agent tools (smart_search, find_symbol, read_code_section) — Task 9
- ✅ System prompt updates — Task 9
- ✅ Startup integration & status endpoint — Task 10
- ✅ Error handling (index not ready, API failure) — built into orchestrator

**Placeholder scan:** No TBD/TODO/placeholder text found.

**Type consistency:** Verified:
- `SearchOrchestrator` constructor params match across orchestrator.py, search_tools.py, main.py
- `SemanticSearch.query()` return format matches `format_results()` input format
- `IndexBuilder` takes `SemanticSearch` instance — matches usage in main.py startup
- `RepoMeta` fields consistent between registry.py and orchestrator.py usage
