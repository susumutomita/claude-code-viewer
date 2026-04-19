# claude-code-viewer

**Any repo → AI-powered code wiki in one command.**

DeepWiki-style code documentation with native Claude Code integration. Every run is a **full rebuild from source** — no stale docs, no diff residue.

## What it does

1. **Scans** your project structure, detects tech stack, finds key files
2. **Analyzes** each major component deeply using Claude Code (`claude -p`)
3. **Generates** a complete static site with architecture docs
4. **Serves** it with interactive AI features:
   - **Chat widget** — ask anything about the codebase
   - **Right-click** → "Ask Claude Code" on any selected text
   - **Ask button** — one-click deep-dive on suggested prompts
   - **File Reader** — browse source code, select & ask

## Quick Start

```bash
# Generate wiki for current project
./generate

# Generate for a specific repo
./generate /path/to/any/repo

# Just serve previously generated docs
./generate --serve-only

# Custom port
./generate --port 3000

# Generate without starting server
./generate --no-serve
```

## Requirements

- Python 3.8+
- [Claude Code](https://claude.ai/code) CLI installed (`claude` command available)

No other dependencies.

## How it works

```
./generate /path/to/repo
    │
    ├── [1] Scan project structure
    │     → detect tech stack, key files, major directories
    │
    ├── [2] Deep analysis with Claude Code (parallel)
    │     → claude -p "analyze this directory..."  ×N
    │     → overview, per-directory, patterns, dependencies, dev guide
    │
    ├── [3] Generate static site
    │     → .codewiki/index.html (self-contained)
    │
    └── [4] Serve with API layer
          → POST /api/ask  → claude -p (chat)
          → GET /api/file  → read source files (file reader)
```

## Why full rebuild?

> "差分更新は AI はうまくない"

Incremental doc updates leave stale sections, miss renamed files, and accumulate drift. This tool regenerates **everything from scratch** every time. Run it on a schedule or after major changes — the docs are always fresh and consistent.

## vs DeepWiki

| | DeepWiki | claude-code-viewer |
|---|---|---|
| AI analysis | Static generation | Static + **live AI chat** |
| Right-click → ask | No | Yes |
| Chat with codebase | No | Yes (Claude Code native) |
| File browsing | Limited | Full reader with AI context |
| Update strategy | Unknown | Full rebuild (no stale docs) |
| Self-hosted | No | Yes (localhost, zero deps) |
| Works offline | No | Yes (with Claude Code CLI) |

## Output

Generated docs are written to `<project>/.codewiki/index.html`. Add `.codewiki` to your `.gitignore` if you don't want to commit generated docs.

## License

MIT
