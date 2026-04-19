<p align="center">
  <br>
  <strong style="font-size:48px">codewiki</strong>
  <br>
  <em>Democratize AI-assisted code reading.</em>
  <br><br>
  <a href="https://github.com/susumutomita/codewiki/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="License"></a>
  <a href="https://github.com/susumutomita/codewiki"><img src="https://img.shields.io/github/stars/susumutomita/codewiki?style=social" alt="Stars"></a>
</p>

<p align="center">
  A Claude Code skill that encodes senior-level investigation techniques<br>
  (<code>git log</code>, <code>grep</code>, <code>ctags</code>, <code>gh</code>, dependency graphs) into one command.
</p>

<p align="center">
  <a href="https://susumutomita.github.io/codewiki">Website</a> &middot;
  <a href="#install">Install</a> &middot;
  <a href="#why">Why</a> &middot;
  <a href="#how-it-works">How it works</a> &middot;
  <a href="#the-gap">The gap</a>
</p>

---

## Why codewiki exists <a name="why"></a>

**AI-assisted code reading is a skill — and it's not distributed evenly.**

Senior engineers run `git log --oneline -- path/`, trace imports, check related Issues, grep for error handlers. They ask AI the right questions in the right order and get deep understanding. Juniors ask "what does this file do?" and get shallow answers. Same AI, wildly different results.

codewiki encodes senior-level investigation techniques — `ctags`, `grep import`, `git log`, `gh issue view`, dependency graphs — into the skill itself. Whoever runs `/codewiki` gets the same thorough analysis a senior engineer would produce after hours of digging.

**codewiki democratizes AI-assisted code reading.**

And it does it in one command, answering upfront every question you would have asked AI one-by-one anyway:

```
"What's this directory?"        → directory map table
"How does this function work?"  → explanation with code quotes
"Why is it written this way?"   → design rationale from git log / Issues / PRs
"How does data flow?"           → Mermaid flowchart
"What's the DB schema?"         → Mermaid ER diagram
"Who depends on this module?"   → dependency graph
"How do I set it up?"           → setup guide
"Where do I look when it breaks?" → error flow map
```

All in one `/codewiki`. No more paying tokens to ask the same questions across your team.

## Install

```bash
npx skills add susumutomita/codewiki
```

Then in any project, open Claude Code and type:

```
/codewiki
```

That's it.

## What happens

1. **Scans** your project — detects structure, tech stack, key files
2. **Analyzes** every major component — Claude Code reads and understands your actual source code
3. **Generates** a self-contained HTML wiki at `.codewiki/index.html`
4. **Serves** it locally with interactive AI features

### Interactive Features

| Feature | How |
| --- | --- |
| **Chat** | Click the purple button (bottom-right) and ask anything |
| **Right-click** | Select any text, right-click, choose "Ask Claude Code" |
| **Ask button** | Click "Ask" on suggested prompts for instant deep-dive |
| **File Reader** | Browse source code in-browser, select and right-click to ask |

## How it works

```
You type: /codewiki

Claude Code:
  ├── Scans project structure (Glob)
  ├── Reads key files deeply (Read)
  ├── Analyzes each component (native AI)
  ├── Generates .codewiki/index.html (Write)
  └── Starts local server + opens browser (Bash)

You get:
  ├── Complete architecture documentation
  ├── Per-component deep analysis
  ├── Live chat widget → Claude Code
  ├── Right-click → Ask on any text
  └── Built-in source code browser
```

The key difference from other doc generators: **Claude Code itself is doing the analysis**. Not a subprocess, not an API call — the same AI that understands your codebase generates the documentation and answers your follow-up questions.

## Re-generate

Run `/codewiki` again. It rebuilds **everything from scratch**.

No incremental diffs. No stale sections. No ghost references to renamed files. The wiki always matches the current state of your code.

> *"AI is not good at incremental doc updates — sections get stale, renames leave ghost references, and context drifts. Full rebuild eliminates all of that."*

## The gap, closed <a name="the-gap"></a>

The difference between a senior and a junior asking AI about a codebase isn't the AI — it's the *investigation technique*. Here's what codewiki gives you for free:

| Question you'd ask AI | Manual (senior technique) | codewiki |
| --- | --- | --- |
| "What's in this repo?" | `find`, `tree`, skim README | directory map table |
| "How does this function work?" | `grep`, trace callers, read refs | explanation with code quotes |
| "Why is it written this way?" | `git log`, `git blame`, read PRs | rationale from git + Issues + PRs |
| "How does data flow?" | read + whiteboard | Mermaid flowchart, auto-generated |
| "What's the DB schema?" | find migrations, read models | Mermaid ER diagram |
| "Who depends on this module?" | `madge`, `grep imports` | dependency graph |
| "How do I set it up?" | read scripts + trial & error | setup guide from actual scripts |
| "Where do I look when it breaks?" | `grep catch`, find log points | error flow map (`--debug`) |
| "Can I ask follow-ups?" | new AI session, paste code | right-click → ask, inline |

*Same answers. Same depth. No senior required.*

## Works with any stack

Tested on:

- **TypeScript / Node.js** — Next.js, Angular, Express
- **Python** — Django, Flask, Lambda
- **Go** — gin, standard library projects
- **Rust** — axum, tokio ecosystem
- **AWS CDK / Terraform** — infrastructure as code
- **Monorepos** — multi-package projects

## Roadmap — from skill to orchestrator

Today codewiki runs `git log`, `grep`, `ctags` and similar. That's already senior-level investigation. But real code understanding needs more specialized tools — and each ecosystem has best-in-class analyzers. **codewiki's next evolution is as an AI orchestrator that wraps them.**

### v1.1 — integrate specialized analyzers

| Need | Tool codewiki will invoke | What you get |
| --- | --- | --- |
| Module dependencies | `madge`, `pydeps`, `go mod graph`, `cargo tree` | Enhanced Mermaid graphs from real tool output |
| Cyclomatic complexity | `radon`, `eslintcc`, `gocyclo` | Per-function complexity table + hotspot list |
| Change hotspots | `git log` frequency analysis | CodeScene-style "where the risk lives" map |
| Architecture violations | `dependency-cruiser`, `import-linter` | Layer rule violations flagged with explanations |
| Symbol extraction | `ctags`, `tree-sitter` | Accurate class/function maps for diagrams |

### v1.2 — interactive visualizations

- D3-based force-directed dependency graphs (drag, zoom, filter) embedded in the wiki
- Clickable call graphs via `pyan` / AST walks
- Heatmap overlays (complexity × change frequency)

### v1.3 — plugin system

```yaml
# .codewikirc.yml
analyzers:
  - name: madge
    run: "npx madge --json src/"
    parse: "json"
  - name: custom-tool
    run: "./my-analyzer.sh"
    parse: "json"
```

Any analyzer that outputs JSON can be plugged in. Claude Code reads the output and integrates it into the wiki narrative automatically — no per-tool plugin code needed.

### The strategic bet

Existing static analyzers (SonarQube, CodeScene, Sourcegraph) give you **numbers and graphs**. Interpretation is left to the reader. codewiki layers **AI-driven interpretation** on top: correlating tool outputs, linking findings to specific code, explaining *why* this hotspot matters in *this* codebase.

*The integration is where the defensibility lives.*

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- Python 3.8+ (for the local wiki server)

## Output

Generated docs go to `.codewiki/index.html`. Add `.codewiki/` to your `.gitignore`.

## License

MIT
