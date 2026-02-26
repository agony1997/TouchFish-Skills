---
name: md-to-code
description: >
  根據結構化的 AI coding 實作文件（.md）逐步實作程式碼。讀取 prompt.md 大綱，
  依序參考技術規格、後端實作指示、前端實作指示，逐步確認後實作後端與前端程式碼。
  使用時機：當使用者要求根據已產出的實作文件開始寫程式、實作功能、按照文件 coding、
  依文件實作、開始開發時觸發。關鍵字：md to code、根據文件實作、開始寫程式、
  按文件 coding、依文件開發、實作程式碼、開始實作、to code、toCode、implement, implementation, 寫程式, coding, 依規格實作, 按規格開發, 從文件產生程式碼, generate code, code generation, 開始開發。
---

# MD-to-Code

You are the implementation lead. You orchestrate parallel backend + frontend development from structured spec docs.

## Workflow Overview

```
Step 1 → 1.5 → 2 → 3 → 3.5 → 4 → 5
Read     Gate   Plan  Impl  Gate  Verify  Close
(subs)          (SP)  (team)      (SP)    (report)

subs = parallel subagents | SP = superpowers | team = Agent Teams
Gates = mandatory human checkpoints (AskUserQuestion)
```

## Step 1: Read Docs via Parallel Subagents

Read `prompt.md` to get doc paths and project standards paths. Launch **3 parallel Task agents** (model: opus) in a single message. Do NOT set `run_in_background` — wait for all results.

**Agent A — Tech Spec:**
- Read `01_技術規格.md`
- Produce: API endpoint summary, DB table structure, entity mapping, business rules

**Agent B — Project Standards + Context Exploration:**
- Read project standards from prompt.md (DEVELOP.md, NAMING_CONVENTIONS.md, etc.) + `CLAUDE.md`
- Glob/Grep existing similar-feature code — learn patterns
- Search reusable components: backend (Processor, Entity, Repository, utils), frontend (Vue, Store, Types, shared), SQL (migrations)
- Produce: naming conventions, architecture patterns, code style summary, **reusable components list**

**Agent C — Implementation Docs:**
- Read `02_後端實作.md` and `03_前端實作.md`
- Produce: backend step list, frontend step list, file inventory

After all agents return, consolidate results.

## Step 1.5: Checkpoint — Confirm Understanding (mandatory gate)

**Mandatory pause.** Use AskUserQuestion to present:

1. **Understanding summary**: 3-5 sentences on feature goal and scope
2. **Reusable components found**: list file paths and intended usage
3. **Open questions**: uncertainties or ambiguities

Question: "Is the above correct? Any missing components?"
Options: Correct, proceed | Need additions | Need direction change

## Step 2: Generate Implementation Plan

Use `superpowers:writing-plans` methodology. Produce step checklist:

```
Implementation Plan: <feature name>

Backend (per 02_後端實作.md):
  [] 1. <step> — <description>
  [] 2. ...

Frontend (per 03_前端實作.md):
  [] 1. <step> — <description>
  [] 2. ...
```

Submit to user for confirmation. User may adjust order or scope.

## Step 3: Agent Teams Parallel Implementation

> **Optional integration** — if superpowers plugin is installed, teammates may use `superpowers:test-driven-development` (TDD) and `superpowers:systematic-debugging` during implementation.

### 3a. Create team and task list

TeamCreate `"impl-<feature-name>"`. Convert Step 2 plan into TaskCreate entries. Tag backend vs frontend.

### 3b. Spawn teammates (on-demand prompt loading)

**Before spawning each teammate:**
1. Glob `**/md-to-code/**/prompts/backend-dev.md` or `**/md-to-code/**/prompts/frontend-dev.md`
2. Read the matched file
3. Fill `{variables}` with consolidated context from Step 1
4. Use filled template as the Task tool prompt (with `team_name` and `name` parameters)

**backend-dev** — Task tool with `name: "backend-dev"`:
- Context: 01_技術規格 + 02_後端實作 + project standards summary + reusable components list
- Implements all backend files per 02_後端實作.md order

**frontend-dev** — Task tool with `name: "frontend-dev"`:
- Context: 01_技術規格 + 03_前端實作 + project standards summary + reusable components list
- Implements all frontend files per 03_前端實作.md order

### 3c. Cross-team communication and consolidation

- Teammates may SendMessage each other for API alignment
- User can Shift+Down to intervene in any teammate's context
- After all tasks: consolidate outputs, list all files, summarize cross-team communication, note key decisions

## Step 3.5: Checkpoint — Implementation Results (mandatory gate)

**Mandatory pause.** Use AskUserQuestion to present:

1. **Backend output**: files created/modified, key decisions, deviations from spec (if any, with reasons)
2. **Frontend output**: files created/modified, component split, state management, deviations
3. **Cross-layer coordination**: communication record summary
4. **Attention items**: unimplemented features, manual adjustments needed

Question: "Do the results meet expectations?"
Options: Yes, proceed to verification | Backend needs changes | Frontend needs changes | Both need changes

If adjustments needed: SendMessage to the relevant teammate with change requests, wait for completion, re-present results.

## Step 4: Post-Implementation Verification

Use `superpowers:verification-before-completion` principles + `superpowers:requesting-code-review`:

1. **File completeness**: compare against implementation plan — all files produced?
2. **Code review** (via code-reviewer agent): spec compliance, naming/style consistency, flag issues
3. **Prompt user**: run build (backend + frontend), run functional tests, note manual adjustments

## Step 5: Close Team + Generate Completion Report

1. **Shutdown team**: shutdown_request to all teammates → confirm closure → TeamDelete
2. **Generate report**: Glob `**/md-to-code/**/references/completion-report-template.md`, Read it, fill in content → write `04_完成報告.md`
3. Inform user of report location

> **Optional integration** — if superpowers plugin is installed, use `superpowers:finishing-a-development-branch` for branch closing workflow.

## Principles

- Follow spec docs strictly — do not add undefined features
- Prefer existing shared components and utils
- Match project naming and code style conventions
- Spec vs project standards conflict → follow project standards, notify user
- Step 1 (read): Subagents — efficient, low-cost, no inter-agent communication needed
- Step 3 (impl): Agent Teams — cross-layer communication, human intervention, shared task list
- All teammate prompts must embed full context (tech spec, impl doc, standards summary, reusable components)
- All Task agents / teammates: model opus
