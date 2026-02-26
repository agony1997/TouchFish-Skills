# Conventional Commits Reference

## Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

## Type Table

| Type | When to Use | Example |
|------|-------------|---------|
| `feat` | New feature | `feat(auth): add OAuth2 login` |
| `fix` | Bug fix | `fix(api): resolve null pointer exception` |
| `refactor` | Restructure, no behavior change | `refactor(service): extract common logic` |
| `perf` | Performance improvement | `perf(query): add database index` |
| `style` | Formatting, lint fixes | `style(lint): fix eslint warnings` |
| `test` | Add or modify tests | `test(user): add unit tests` |
| `docs` | Documentation only | `docs(api): update API documentation` |
| `build` | Build system, dependencies | `build(deps): upgrade spring boot to 3.2` |
| `ci` | CI/CD configuration | `ci(jenkins): update pipeline config` |
| `chore` | Other non-source changes | `chore(config): update .gitignore` |
| `revert` | Revert previous commit | `revert: feat(auth): add OAuth2 login` |

## Decision Tree

```
New feature?           → feat
Bug fix?               → fix
Performance?           → perf
Restructure, same behavior? → refactor
Test related?          → test
Docs only?             → docs
Build/deps?            → build
CI/CD?                 → ci
Formatting?            → style
Other                  → chore
```

## Subject Line Rules

- Max 50 characters (Chinese ~25 chars)
- Imperative mood: Add, Fix, Update (not Added, Fixed)
- No trailing period
- Capitalize first letter
- Be concise and specific

## Body & Footer

- **Body**: Wrap at 72 chars, use bullet points, explain "what" and "why" (not "how")
- **Footer**: `BREAKING CHANGE: description`, `Fixes #123`, `Closes #456`
- **Required**: `Co-Authored-By: Claude <noreply@anthropic.com>`

## Commit Message Examples

**New feature:**
```
feat(HRM001): add employee leave request functionality

Implement employee leave request and approval workflow
- Add leave request form and REST API endpoints
- Add manager approval functionality with email notifications
- Add leave record query and history tracking

Co-Authored-By: Claude <noreply@anthropic.com>
```

**Bug fix:**
```
fix(auth): resolve token expiration validation issue

Fix incorrect token expiration time calculation in JWT validation
- Update token expiry check to use UTC timezone
- Add proper null checks for refresh token
- Fix race condition in token refresh logic

Fixes #1234

Co-Authored-By: Claude <noreply@anthropic.com>
```

## HEREDOC Execution Format

```bash
git commit -m "$(cat <<'EOF'
feat(HRM001): add employee leave request functionality

Implement employee leave request and approval workflow
- Add leave request form and REST API endpoints
- Add manager approval with email notifications

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

## Atomic Commits

Each commit = single logical change. Split unrelated changes:

```bash
git add src/components/UserProfile.vue
git commit -m "feat(profile): add user profile component"

git add src/api/userApi.js
git commit -m "feat(profile): add user profile API client"

git add tests/UserProfile.test.js
git commit -m "test(profile): add user profile component tests"
```

## Handling Commit Failures

**Pre-commit hook failure:**
```bash
# Fix the issue, then create a NEW commit (never use --no-verify or --amend)
npm run lint:fix
git add <fixed-files>
git commit -m "style(lint): fix linting errors"
```

**Committed wrong files (not yet pushed):**
```bash
git reset --soft HEAD~1
git add file1.java file2.java
git commit -m "feat(module): add feature X"
git add file3.java
git commit -m "fix(module): fix bug Y"
```

## Semantic Commits and Auto-Versioning

| Commit Type | Version Bump | Reason |
|-------------|-------------|--------|
| `feat:` | MINOR | New feature |
| `fix:` | PATCH | Bug fix |
| `perf:` | PATCH | Performance improvement (no new API) |
| `refactor:` | PATCH | Internal restructure |
| `docs:` | PATCH | Documentation only |
| `style:` | PATCH | Formatting, no logic change |
| `test:` | PATCH | Test additions/changes |
| `build:` | PATCH | Build system / dependency updates |
| `ci:` | — (no bump) | CI/CD configuration |
| `chore:` | — (no bump) | Non-source maintenance |
| `BREAKING CHANGE:` | MAJOR | Any breaking change (in footer or `!` suffix) |
