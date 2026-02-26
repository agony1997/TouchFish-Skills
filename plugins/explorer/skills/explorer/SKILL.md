---
name: explorer
description: >
  專案探索者：使用 Opus Leader 指揮多個 sub-agents 並行探索專案結構，
  交叉比對後產出結構化的專案地圖文件（PROJECT_MAP.md）。
  支援單體、多模組、monorepo 等專案類型的智慧偵測與分工。
  使用時機：接手新專案、了解專案架構、產出專案地圖、實作前收集上下文。
  關鍵字：explore, 探索, 偵察, 專案地圖, project map, codebase,
  了解專案, 專案結構, 探索專案, 分析架構, architecture, 架構分析,
  onboarding, 新人導覽, 專案概覽, 技術棧, tech stack。
---

# Project Explorer

You are the explore-leader (Opus). You command sub-agents to explore project structure in parallel, cross-check results, and produce a structured PROJECT_MAP.md.

## Phase 0: Detect Project Type

**Goal:** Determine project type and sub-agent dispatch plan.

1. Read root directory (Glob + Read):
   - All files and first-level subdirectories
   - Build files (`build.gradle`, `pom.xml`, `package.json`, `Cargo.toml`, `go.mod`, etc.)
   - Config files (`docker-compose.yml`, `tsconfig.json`, `.env`, etc.)

2. Detection logic:
   ```
   if root has packages/ or modules/ or multiple build files:
       type = "multi-module" or "monorepo"
       if shared infra/ (docker, k8s, ci):
           type = "hybrid"
           agents = [infra-explorer] + [module-N-explorer per module]
       else:
           agents = [module-N-explorer per module]
   elif clear frontend/backend split (src/main + src/webapp, or server/ + client/):
       type = "monolith"
       agents = [backend-explorer, frontend-explorer]
   else:
       type = "monolith (simple)"
       agents = [single-explorer]
   ```

3. AskUserQuestion to confirm:
   - Detected project type
   - Planned sub-agent dispatch
   - Any custom scope inclusions/exclusions

## Phase 1: Dispatch Explore Sub-agents

**Goal:** Dispatch sub-agents in parallel to explore their assigned scopes.

1. Load prompt template:
   - `Glob("**/explorer/**/prompts/explore-subagent.md")` then Read
   - Fill `{scope_path}` and `{project_root}` per agent

2. Dispatch all sub-agents in parallel:
   ```
   Task(subagent_type: "Explore", model: "sonnet") x N
   All agents dispatched simultaneously, do not wait between dispatches
   ```

3. Wait for all sub-agent reports.

## Phase 2: Cross-check & Review

**Goal:** Cross-check all reports for accuracy and consistency.

**Review checklist:**
1. Tech stack descriptions consistent across reports?
2. Frontend/backend API endpoint paths aligned?
3. Shared types/interfaces match?
4. No contradictions between config files?
5. Module dependency directions reasonable (no circular)?
6. Any reports missing important directories/files?
7. Shared component lists complete and consistent?
8. Project standards found and summarized? If missing, ask user?

**Standards confirmation (required):**
- If sub-agents report "No project standards found" -> AskUserQuestion:
  - Does the project have standards docs? Where?
  - Any undocumented naming conventions or architecture rules?
  - Need to create basic standards?
- If user provides location -> dispatch sub-agent to read and summarize
- If user confirms none -> note "No explicit project standards" in PROJECT_MAP.md

**Review loop:**
- **Clear doubt** (e.g., report A says Spring Boot 3.2, report B says 3.1)
  -> Dispatch targeted sub-agent: `Task(subagent_type: "Explore", model: "sonnet")`
- **Uncertain** (e.g., unknown module purpose)
  -> AskUserQuestion; dispatch sub-agent to verify if needed
- **No issues** -> item passes

**Repeat until ALL checklist items pass, then proceed to Phase 3.**

## Phase 3: Consolidate Output

**Goal:** Merge all exploration results into PROJECT_MAP.md.

1. AskUserQuestion for output path (default: project root; alternatives: `docs/`, custom)
2. Load output template:
   - `Glob("**/explorer/**/references/project-map-template.md")` then Read
3. Write PROJECT_MAP.md using Write tool
4. Present summary to user:
   - Project type
   - Tech stack overview
   - Key issues found (if any)
   - PROJECT_MAP.md file path

> **Note:** Omit sections not applicable to the project. Do not fabricate content to fill the template.
