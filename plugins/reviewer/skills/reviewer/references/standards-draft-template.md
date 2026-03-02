# Standards Draft Template

Use this format when generating `.standards/` draft files from extraction workflow.

## DRAFT Banner

Every generated file MUST begin with this banner:

```markdown
<!-- DRAFT — Auto-extracted on {date} — Requires human review before adoption -->
<!-- Confidence: [H] = High (>80% files consistent) | [M] = Medium (50-80%) -->
```

## Convention Entry Format

Each convention entry follows this structure:

```markdown
### {Convention Name} `[H|M]` `({n}/{total} files)`

{Rule description — one or two sentences.}

**Examples:**
- `path/to/file1.ext` — `exampleCode`
- `path/to/file2.ext` — `exampleCode`
- `path/to/file3.ext` — `exampleCode`

**Rationale:** {Why this was inferred as a convention — observed pattern description.}
```

## Dimension Sections

### naming.md

- File / Directory naming
- Class / Interface naming
- Method / Function naming
- Variable / Constant naming

### architecture.md

- Layer Structure
- Dependency Direction
- Module Organization
- Design Patterns
- Error Handling Strategy

### code-style.md

- Import Organization
- Function Structure
- Comment Style
- Logging
- Configuration

### database.md

- Table Naming
- Column Naming
- Migration Naming
- Index Naming
- Entity Mapping

### frontend.md

- Component Structure
- Component Naming
- State Management
- Styling
- Type Definitions

## Appendix: Possible Conventions

Low-confidence items (< 50% consistency) go in a separate section at the end:

```markdown
## Possible Conventions (Low Confidence)

> These patterns were observed in some files but not consistently enough to be considered conventions.
> Review and promote to main sections if they reflect intended standards.

- {pattern description} — seen in {n} files (e.g., `file1`, `file2`)
```
