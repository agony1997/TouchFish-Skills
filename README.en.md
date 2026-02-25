# touchfish-skills

> Workflow-oriented skill plugins for Claude Code — 7 plugins covering DDD, Git, code review, spec conversion, implementation, project exploration, and team collaboration.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Plugins](https://img.shields.io/badge/plugins-7-blue.svg)](.claude-plugin/marketplace.json)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skills-blueviolet.svg)](https://docs.anthropic.com/en/docs/claude-code)

[繁體中文](README.md) | **English**

A Claude Code skill plugin marketplace with 7 workflow-oriented plugins.

> **Design Principle**: Only keep skills with clear workflows. Pure domain knowledge (Spring Boot, PostgreSQL, Vue.js, etc.) is left to Claude's built-in capabilities; process methodologies (TDD, brainstorming, debugging) are handled by external plugins like superpowers.

## Quick Start

```bash
claude /plugin add ./path/to/touchfish-skills
```

Set **git-nanny** at User scope (globally available); enable others per project.

## Plugin List

| Plugin | Type | Description |
|--------|------|-------------|
| `ddd-core` | Methodology | DDD end-to-end delivery: Event Storming → SA → SD → Implementation Plan |
| `git-nanny` | Operations | Git Commit, PR, branching strategy, release & Changelog |
| `reviewer` | Review | Project standards reviewer: reads project standards files, runs compliance checks |
| `spec-to-md` | Conversion | Spec files → structured AI coding implementation docs |
| `md-to-code` | Implementation | Implementation docs → code (parallel Agent Teams) |
| `explorer` | Exploration | Project explorer: Opus Leader + sub-agents parallel exploration, outputs project map |
| `dev-team` | Collaboration | Dev team: multi-role pipeline (PM/Dev/QA), dynamic scaling, mixed agents |

## Architecture Overview

```
Requirements ──→ ddd-core ──→ spec-to-md ──→ md-to-code ──→ git-nanny
                  (DDD)       (Spec→Docs)    (Docs→Code)    (Commit/PR)

Exploration ───→ explorer ──→ PROJECT_MAP.md
                  (Parallel)

Large Feature ─→ dev-team ──→ PM → API Contract → Dev + QA Pipeline → Delivery
                  (Multi-role)
```

## Companion Plugins

| Plugin | Source | Covers |
|--------|--------|--------|
| superpowers | obra/superpowers-marketplace | brainstorming, TDD, debugging, code review, plan |
| document-skills | anthropic-agent-skills | docx, pptx, pdf, xlsx document processing |
| claude-developer-platform | anthropic-agent-skills | Claude API development guide |

## Common Development Scenarios

Below are recommended skill combinations for common scenarios. **Bold** items are touchfish-skills; others are superpowers skills (requires separate superpowers plugin installation).

| Scenario | Recommended Skills | Workflow |
|----------|--------------------|----------|
| Onboarding to a project | **`explorer`** | Parallel exploration → PROJECT_MAP.md |
| New feature (full DDD) | `brainstorming` → **`ddd-core`** → **`spec-to-md`** → **`md-to-code`** → **`git-nanny`** | Brainstorm → DDD 4 phases → Generate docs → Parallel impl → Commit/PR |
| New feature (large team) | **`dev-team`** | PM analysis → API contract → Multi-role pipeline → QA review → Delivery |
| New feature (lightweight) | `brainstorming` → `writing-plans` → **`md-to-code`** → **`git-nanny`** | Brainstorm → Write plan → Implement → Commit/PR |
| Bug fix | `systematic-debugging` → `test-driven-development` → **`git-nanny`** | Systematic diagnosis → TDD fix → Commit/PR |
| Code review | **`reviewer`** + `requesting-code-review` | Standards review + Review workflow |
| Refactoring | `brainstorming` → `writing-plans` → `test-driven-development` → **`git-nanny`** | Discuss scope → Write plan → TDD refactor → Commit/PR |

> **No superpowers installed?** All touchfish-skills work independently; superpowers integration points are optional enhancements.

## Configuration Examples

#### User scope — Global tools

`~/.claude/settings.json` (see `examples/global-settings.json`):

```jsonc
{
  "enabledPlugins": {
    "touchfish-skills": true,
    "git-nanny@touchfish-skills": true
  }
}
```

#### Project scope — Per-project

`<project>/.claude/settings.json` (see `examples/project-settings.json`):

```jsonc
{
  "enabledPlugins": {
    "touchfish-skills": true,
    "ddd-core@touchfish-skills": true,
    "reviewer@touchfish-skills": true,
    "spec-to-md@touchfish-skills": true,
    "md-to-code@touchfish-skills": true,
    "explorer@touchfish-skills": true,
    "dev-team@touchfish-skills": true
  }
}
```

## Directory Structure

```
touchfish-skills/
├── README.md                        # 繁體中文
├── README.en.md                     # English
├── LICENSE
├── plugins/
│   ├── ddd-core/                    # DDD methodology
│   ├── git-nanny/                   # Git workflow
│   ├── reviewer/                    # Standards reviewer
│   ├── spec-to-md/                  # Spec → implementation docs
│   ├── md-to-code/                  # Docs → code
│   ├── explorer/                    # Project explorer (parallel sub-agents)
│   └── dev-team/                    # Dev team (multi-role pipeline)
├── docs/plans/                      # Design & plan documents
└── examples/                        # Configuration examples
    ├── global-settings.json
    └── project-settings.json
```

## Attribution

The `git-nanny` skill references these open standards:

- [Conventional Commits](https://www.conventionalcommits.org/) (CC BY 3.0)
- [Semantic Versioning](https://semver.org/) (CC BY 3.0)
- [Keep a Changelog](https://keepachangelog.com/) (MIT)

## License

[MIT](LICENSE)
