# PROJECT_MAP.md Template

Use this template when producing the final PROJECT_MAP.md output.

```markdown
# Project Map: <project-name>
> Generated: YYYY-MM-DD | Scope: <root-dir>

## Project Overview
- Type: monolith / multi-module / monorepo
- Primary language: <lang + version>
- Framework: <framework + version>
- Build tool: <tool + version>

## Architecture Overview
(High-level module/layer relationships)

## Module List (for multi-module)
| Module | Path | Responsibility | Key Dependencies |
|--------|------|----------------|------------------|

## Backend Structure
- Entry point: ...
- Layering: <describe actual layering pattern, e.g., Controller -> Service -> Repository, Handler -> UseCase -> Gateway, etc.>
- Key config files: ...

## Frontend Structure
- Entry point: ...
- Component structure: <describe actual component organization>
- State management: ...
- Routing: ...

## Infrastructure
- CI/CD: ...
- Containerization: ...
- Environment config: ...

## Key File Index
| File | Purpose | Notes |
|------|---------|-------|

## Shared Components & Reusable Resources
| Component | Path | Type | Description |
|-----------|------|------|-------------|

(e.g., base classes, API clients, utility functions, shared UI components, response wrappers, etc.)

## Project Standards Summary
- **Standards sources:** (list file paths found)
- **Naming conventions:** (class, method, variable, file naming rules)
- **Architecture patterns:** (layered architecture, design patterns, etc.)
- **Code style:** (indentation, quotes, lint rules, etc.)
- **Commit conventions:** (Conventional Commits, etc.)
- **Other:** (API conventions, error handling patterns, etc.)

> If the project has no explicit standards, note "No explicit project standards" and list implicit conventions observed from the codebase.

## Known Issues & Notes
(Inconsistencies found during cross-check, potential issues, etc.)
```

Note: Omit sections not applicable to the project. Do not fabricate content to fill the template.
