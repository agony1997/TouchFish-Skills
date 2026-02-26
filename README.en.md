# touchfish-skills

> Workflow-oriented skill plugins for Claude Code — 7 plugins covering DDD, Git, code review, spec conversion, implementation, project exploration, and team collaboration.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Plugins](https://img.shields.io/badge/plugins-7-blue.svg)](.claude-plugin/marketplace.json)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skills-blueviolet.svg)](https://docs.anthropic.com/en/docs/claude-code)

[繁體中文](README.md) | **English**

A Claude Code skill plugin marketplace with 7 workflow-oriented plugins.

> **Design Principle**: Only keep skills with clear workflows. Pure domain knowledge (Spring Boot, PostgreSQL, Vue.js, etc.) is left to Claude's built-in capabilities; process methodologies (TDD, brainstorming, debugging) are handled by external plugins like superpowers.

### v1.1.0 Unified Architecture (2026-02-26)

All 7 plugins now follow a unified structure:

- **English SKILL.md** (60–150 lines) — always loaded by AI, terse directive style
- **references/ + prompts/** (on-demand) — loaded via Glob + Read to minimize context usage
- **docs/GUIDE.zh-TW.md** — Chinese human-readable guide

| Metric | v1.0.0 | v1.1.0 |
|--------|--------|--------|
| Total SKILL.md lines (7 plugins) | ~3,055 | ~912 (-70%) |
| On-demand reference files | 1 | 18 |
| Human guides | 1 | 7 |

## Installation

```bash
# 1. Clone locally
git clone https://github.com/agony1997/TouchFish-Skills.git

# 2. Add to Claude Code (installs all plugins; enable individual skills as needed)
claude mcp add-plugin ./path/to/TouchFish-Skills
```

> **Tip**: Set `git-nanny` at User scope (globally available); enable others per project.

## Plugin List

| Plugin | Version | Type | Description | Guide |
|--------|---------|------|-------------|-------|
| `ddd-core` | 1.1.0 | Methodology | DDD end-to-end delivery: Event Storming → SA → SD → Implementation Plan | [Guide](plugins/ddd-core/docs/GUIDE.zh-TW.md) |
| `git-nanny` | 1.1.0 | Operations | Git Commit, PR, branching strategy, release & Changelog | [Guide](plugins/git-nanny/docs/GUIDE.zh-TW.md) |
| `reviewer` | 1.1.0 | Review | Project standards reviewer: reads project standards files, runs compliance checks | [Guide](plugins/reviewer/docs/GUIDE.zh-TW.md) |
| `spec-to-md` | 1.1.0 | Conversion | Spec files → structured AI coding implementation docs | [Guide](plugins/spec-to-md/docs/GUIDE.zh-TW.md) |
| `md-to-code` | 1.1.0 | Implementation | Implementation docs → code (parallel Agent Teams) | [Guide](plugins/md-to-code/docs/GUIDE.zh-TW.md) |
| `explorer` | 1.1.0 | Exploration | Project explorer: Opus Leader + sub-agents parallel exploration, outputs project map | [Guide](plugins/explorer/docs/GUIDE.zh-TW.md) |
| `dev-team` | 2.1.0 | Collaboration | Dev team: task pool architecture (TL + challenger + workers), self-assignment, Agent Metrics | [Guide](plugins/dev-team/docs/GUIDE.zh-TW.md) |

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


## Directory Structure

```
touchfish-skills/
├── README.md / README.en.md
├── LICENSE
├── plugins/
│   ├── ddd-core/
│   │   ├── .claude-plugin/plugin.json
│   │   ├── skills/ddd-core/
│   │   │   ├── SKILL.md                 # AI instructions (English, always loaded)
│   │   │   └── references/              # On-demand (theory, templates)
│   │   └── docs/GUIDE.zh-TW.md          # Chinese human guide
│   ├── git-nanny/                       # Same structure
│   ├── reviewer/                        # Same structure
│   ├── spec-to-md/
│   │   ├── skills/spec-to-md/
│   │   │   ├── SKILL.md
│   │   │   ├── prompts/                 # Teammate spawn templates (on-demand)
│   │   │   └── references/
│   │   └── docs/GUIDE.zh-TW.md
│   ├── md-to-code/                      # Same as spec-to-md
│   ├── explorer/                        # Has prompts/ + references/
│   └── dev-team/
│       ├── skills/dev-team/
│       │   ├── SKILL.md
│       │   ├── prompts/                 # challenger.md + worker.md
│       │   └── references/              # 5 tracking templates + qa-review-template
│       └── docs/GUIDE.zh-TW.md
└── LICENSE
```

## Attribution

The `git-nanny` skill references these open standards:

- [Conventional Commits](https://www.conventionalcommits.org/) (CC BY 3.0)
- [Semantic Versioning](https://semver.org/) (CC BY 3.0)
- [Keep a Changelog](https://keepachangelog.com/) (MIT)

## License

[MIT](LICENSE)
