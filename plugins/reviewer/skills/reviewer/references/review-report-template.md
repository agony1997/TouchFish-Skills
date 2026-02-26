# Review Report Template

Use this format when producing the review report.

```markdown
## Review Report

**Project**: <project-name>
**Standards loaded**: <list of standard files with paths>
**Review scope**: <file list>
**Review date**: <date>

### Non-compliant Items

| # | Severity | File | Line | Standard Item | Issue Description | Suggested Fix |
|---|----------|------|------|---------------|-------------------|---------------|
| 1 | Critical / Major / Minor | ...  | ...  | ...           | ...               | ...           |

> **Severity levels:** Critical = architectural violation or security issue; Major = pattern/convention breach affecting maintainability; Minor = naming or style inconsistency.

### Compliant Items Summary

- [x] <passed item>

### Summary

- Items reviewed: N
- Compliant: N
- Non-compliant: N (Critical: X, Major: Y, Minor: Z)
- Compliance rate: N%
```

## Project Setup Guide

Create standards files in your project:

```
your-project/
├── .standards/
│   ├── common.md          # Company/team shared standards
│   └── project.md         # Project-specific standards
├── src/
└── ...
```

Multiple standards files are all loaded. Organize as needed (company/team/project split).

Suggested sections (pick what applies):

- Naming conventions (classes, methods, variables, file paths, constants)
- Architecture conventions (patterns, layer responsibilities, dependency direction)
- Code style (utility classes, error handling, API format, logging)
- Database conventions (entity mapping, migration naming, indexes)
- Frontend conventions (component structure, state management, type definitions)
