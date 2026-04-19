# claude-code-viewer

**Any repo → AI-powered code wiki. One command.**

DeepWiki-style code documentation with **native Claude Code integration**. Every run is a full rebuild from scratch — no stale docs, no diff residue.

## Install

```bash
npx skills add susumutomita/claude-code-viewer
```

## Usage

In Claude Code (terminal or VSCode), just type:

```
/codewiki
```

That's it. Claude Code will:

1. Scan your entire project structure
2. Read and deeply analyze every major component
3. Generate a self-contained HTML wiki at `.codewiki/index.html`
4. Start a local server and open it in your browser

### Interactive Features

| Feature | How |
|---|---|
| **Chat** | Click the purple button (bottom-right) → ask anything |
| **Right-click** | Select any text → right-click → "Ask Claude Code" |
| **Ask button** | Click "Ask" on suggested prompts for instant deep-dive |
| **File Reader** | Browse source code in-browser, select & right-click to ask |

### Re-generate

Run `/codewiki` again at any time. It rebuilds **everything from scratch** — the wiki always matches the current state of your code.

## Why full rebuild?

Incremental doc updates don't work well with AI. Sections get stale, renamed files leave ghost references, and context drift accumulates. Full rebuild means:

- **Zero stale content** — every section is freshly analyzed
- **Consistent quality** — no "this section was updated but that one wasn't"
- **Simple mental model** — run the command, get current docs

## vs DeepWiki

| | DeepWiki | /codewiki |
|---|---|---|
| AI analysis | Static generation | Static + **live AI chat** |
| Right-click → ask | No | Yes |
| Chat with codebase | No | Yes (Claude Code native) |
| File browsing | Limited | Full reader with AI context |
| Update strategy | Unknown | Full rebuild (no stale docs) |
| Integration | Standalone web app | **Claude Code skill** (seamless) |
| Self-hosted | No | Yes (localhost, zero deps) |

## Requirements

- [Claude Code](https://claude.ai/code) CLI
- Python 3.8+ (for the local server)

## Output

Generated docs are written to `.codewiki/index.html`. Add `.codewiki/` to your `.gitignore`.

## License

MIT
