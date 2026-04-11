# Enhanced Backend Tooling System — Design Spec

## Goal

Replace the agent's basic grep-based search with a multi-tier search service that combines lexical search (ripgrep), semantic search (ChromaDB + Qwen embeddings), and symbol lookup (tree-sitter) — with smart repo targeting and a cheap-first escalation strategy to minimize token waste.

## Context

The wiki platform hosts 6 AI agent codebases (~22,800 source files) with 210 wiki documentation pages. The current agent uses `grep -r -i` for wiki-only search with a 50-result cap. It has no source code search, no semantic understanding, no symbol lookup, and no repo targeting. This causes wasted tokens, missed results, and inability to answer implementation-level questions.

## Architecture

```
┌──────────────────────────────────────────────────────┐
│                    Agent (agent.py)                    │
│  Tools: smart_search, find_symbol, read_code_section  │
│         read_workspace_file, propose_doc_change        │
└────────────────────┬─────────────────────────────────┘
                     │
        ┌────────────▼────────────┐
        │   SearchOrchestrator    │
        │  ─ auto-detects repo(s) │
        │  ─ cheap → expensive    │
        │  ─ ranks & compresses   │
        └──┬──────┬───────┬──────┘
           │      │       │
     ┌─────▼┐ ┌──▼────┐ ┌▼──────────┐
     │Lexical│ │Semantic│ │SymbolIndex│
     │Search │ │Search  │ │(tree-sit) │
     │(rg)   │ │(Chroma)│ │           │
     └───────┘ └───────┘ └───────────┘
                     │
        ┌────────────▼────────────┐
        │     RepoRegistry        │
        │  ─ metadata per repo    │
        │  ─ keyword→repo map     │
        │  ─ page context routing  │
        └─────────────────────────┘
```

## Components

### 1. RepoRegistry (`search/registry.py`)

Static metadata about each repo. Maps queries to likely repos using 3 signals:

1. **Page context** (strongest): user's current wiki page → namespace
2. **Keyword matching**: query terms matched against per-repo keyword lists
3. **Semantic fallback**: embedding similarity against repo descriptions

Data model:

```python
@dataclass
class RepoMeta:
    namespace: str           # "claude-code"
    source_dir: str          # "claude_code"
    wiki_dir: str            # "docs/claude-code"
    languages: list[str]     # ["typescript", "python"]
    keywords: list[str]      # ["tool system", "MCP", "permissions", "CLI"]
    description: str         # One-line summary
    file_count: int          # Cached at startup
```

### 2. Lexical Search (`search/lexical.py`)

ripgrep wrapper with smart options. ~10ms per query.

- Uses `rg --json` for structured output
- `--max-count=5` per file, language-aware `--type` filters
- Results ranked by: filename match > function name > body; match density; path proximity to wiki-referenced files
- Returns top 10 results with 3-line context windows

### 3. Semantic Search (`search/semantic.py`)

ChromaDB with Qwen embeddings. ~100ms per query.

- **Collections**: `wiki_docs`, `code_docs`, `code_blocks`, `symbols`
- **Embedding model**: Qwen `text-embedding-v3` via DashScope API
- Returns top 10 chunks ranked by cosine similarity

### 4. Symbol Index (`search/symbols.py`)

tree-sitter-based extraction. ~50ms per query, no token cost.

- Extracts function/class/type definitions with signatures and docstrings
- Supports Python, TypeScript, C# (covers all 6 repos)
- Stored in ChromaDB `symbols` collection for fast name lookup

### 5. Chunker (`search/chunker.py`)

Turns raw files into indexable chunks:

| Source | Strategy | Collection |
|--------|----------|------------|
| Wiki pages (210 .md) | By `##` headings, ~500 tokens each | `wiki_docs` |
| Source docstrings | Per function/class docstring via tree-sitter | `code_docs` |
| Source code blocks | Per function/method body, max 200 lines | `code_blocks` |
| Symbol definitions | Name + signature + file path | `symbols` |

### 6. Indexer (`search/indexer.py`)

Builds and maintains the search index.

- Runs automatically at backend startup
- Persists to `backend/data/chromadb/` (gitignored)
- Tracks file checksums in a manifest for incremental re-indexing
- First boot: ~2-5 min; subsequent boots: <10s for deltas

### 7. SearchOrchestrator (`search/orchestrator.py`)

The brain. Implements the cheap-first escalation:

```
1. Detect query type: symbol? concept? exact-string?
2. If symbol → Tier 3 (SymbolIndex) first, augment with Tier 1 (Lexical)
3. If exact-string → Tier 1 only
4. If concept → Tier 2 (Semantic), augment with Tier 1
5. If ambiguous → Tier 1 first; if <3 results, escalate to Tier 2
6. Always: deduplicate, rank, compress to fit context window
```

## Tool API

### `smart_search(query, scope="auto")` — replaces `search_knowledge_base`

Unified search across wiki docs and source code. Orchestrator handles repo targeting, tier escalation, and ranking.

- `scope`: `"auto"` (recommended), `"wiki"`, `"code"`, or a namespace like `"claude-code"`
- Returns ranked results with file paths, match snippets, and relevance scores

### `find_symbol(name, namespace="")` — new

Find function/class/type definition by name. Returns location, signature, docstring.

### `read_code_section(file_path, symbol="", start_line=0, end_line=0)` — new

Token-efficient file reading. Reads a specific symbol or line range instead of the whole file.

### Kept unchanged

- `read_workspace_file` — full wiki file read
- `read_source_file` — full source file read
- `list_wiki_pages` — page listing
- `propose_doc_change` — approval-gated doc writes

## System Prompt Updates

Replace the current "When answering" block with search-strategy instructions:

1. Symbol queries → `find_symbol` first
2. General queries → `smart_search` with `scope="auto"`
3. Targeted reads → `read_code_section` over full-file reads
4. Efficiency rule: stop searching when you have good results

## File Structure

```
backend/
├── search/
│   ├── __init__.py           # Exports: SearchOrchestrator, build_index
│   ├── registry.py           # RepoRegistry, RepoMeta
│   ├── lexical.py            # LexicalSearch (ripgrep wrapper)
│   ├── semantic.py           # SemanticSearch (ChromaDB + Qwen)
│   ├── symbols.py            # SymbolIndex (tree-sitter)
│   ├── chunker.py            # Chunking for wiki + source
│   ├── orchestrator.py       # SearchOrchestrator (escalation)
│   └── indexer.py            # IndexBuilder (startup + incremental)
├── data/
│   └── chromadb/             # Persisted index (gitignored)
├── search_tools.py           # LangChain @tool wrappers
├── agent.py                  # Updated tools + system prompt
├── main.py                   # Startup indexing hook
└── ...existing files...
```

## Error Handling

- Index still building → tools return "Search index warming up, retry shortly"
- Qwen embedding API failure → fallback to lexical-only, noted in output
- tree-sitter parse failure → skip file, log warning, don't crash

## Key Design Decisions

1. **ChromaDB** for vector store — file-backed, zero infra, Python-native
2. **Qwen embeddings** — user's preferred model, configured via existing DashScope API key
3. **tree-sitter** for symbol extraction — supports all 3 languages in the workspace
4. **Incremental indexing** — checksums prevent re-indexing unchanged files
5. **`search/` is a pure library** — no FastAPI or LangChain dependencies in the search layer
6. **`search_tools.py` bridges** search layer and LangChain tools
