# Release & Changelog Reference

## Semantic Versioning (SemVer)

```
MAJOR.MINOR.PATCH[-PRERELEASE][+BUILD]
Example: 1.2.3-beta.1+20240127
```

| Change Type | Version Bump | Example |
|-------------|-------------|---------|
| Breaking change (remove API, change behavior) | MAJOR | 1.5.2 -> 2.0.0 |
| New feature (backward compatible) | MINOR | 1.2.0 -> 1.3.0 |
| Bug fix / security patch | PATCH | 1.2.3 -> 1.2.4 |
| Docs only / refactor | PATCH | 1.2.3 -> 1.2.4 |
| Performance improvement | MINOR | 1.2.0 -> 1.3.0 |

**Pre-release versions:**
```
1.0.0-alpha.1 → 1.0.0-beta.1 → 1.0.0-rc.1 → 1.0.0
```

**Initial development (0.x.x):** API not yet stable, breaking changes allowed.

## Changelog Format (Keep a Changelog)

```markdown
# Changelog

All notable changes are documented here.
Format: [Keep a Changelog](https://keepachangelog.com/en/1.0.0/)
Versioning: [Semantic Versioning](https://semver.org/spec/v2.0.0.html)

## [Unreleased]

## [1.2.0] - 2024-01-27

### Added
- Employee leave request functionality
- OAuth2 login with Google/Facebook

### Changed
- Improved database query performance (2x faster)

### Deprecated
- `/api/v1/login` endpoint (use `/api/v2/auth/login`)

### Removed
- Old authentication API

### Fixed
- Token expiration validation causing premature logouts
- Memory leak in WebSocket handler

### Security
- Patched XSS vulnerability in comment system

[Unreleased]: https://github.com/user/repo/compare/v1.2.0...HEAD
[1.2.0]: https://github.com/user/repo/compare/v1.1.0...v1.2.0
```

Categories: **Added**, **Changed**, **Deprecated**, **Removed**, **Fixed**, **Security**

## Release Workflow

### Step 1: Verify Status

```bash
git checkout main && git pull origin main
git status
npm test && npm run build
```

### Step 2: Determine Version Bump

```bash
git log $(git describe --tags --abbrev=0)..HEAD --oneline
# BREAKING CHANGE → MAJOR | feat → MINOR | fix/docs/refactor → PATCH
```

### Step 3: Update Version and Changelog

```bash
npm version 1.2.0 --no-git-tag-version
# Update CHANGELOG.md (manual or via conventional-changelog)
conventional-changelog -p angular -i CHANGELOG.md -s
```

### Step 4: Commit, Tag, Push

```bash
git add package.json CHANGELOG.md
git commit -m "$(cat <<'EOF'
chore(release): bump version to 1.2.0

Update version and generate changelog

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
EOF
)"

git tag -a v1.2.0 -m "$(cat <<'EOF'
Release version 1.2.0

- feat(hrm): add employee leave request
- feat(auth): add OAuth2 authentication
- perf(database): optimize queries with indexes
- fix(api): resolve token expiration issue
EOF
)"

git push origin main --follow-tags
```

### Step 5: Create GitHub Release

```bash
gh release create v1.2.0 \
    --title "Release v1.2.0" \
    --notes "$(cat <<'EOF'
## Highlights
- Employee leave management feature
- OAuth2 authentication
- Performance improvements (2x faster queries)

## What's Changed
**Added:** Leave request workflow, OAuth2 login, Excel export
**Fixed:** Token expiration, memory leak in WebSocket
**Performance:** Optimized DB queries, reduced bundle size 30%

## Upgrade Guide
1. Run database migrations: `npm run migrate`
2. Add OAuth config to `.env`
3. Restart application

**Full Changelog**: https://github.com/user/repo/compare/v1.1.0...v1.2.0
EOF
)"
```

## Hotfix Release

```bash
git checkout main && git checkout -b hotfix/1.2.1
# Fix the critical issue
git commit -m "fix(security): patch critical vulnerability"
npm version 1.2.1 --no-git-tag-version
git add package.json CHANGELOG.md
git commit -m "chore(release): hotfix version 1.2.1"
git checkout main && git merge --no-ff hotfix/1.2.1
git tag -a v1.2.1 -m "Hotfix v1.2.1 - Security patch"
git push origin main --tags
git checkout develop && git merge main && git push origin develop
git branch -d hotfix/1.2.1
gh release create v1.2.1 --title "Hotfix v1.2.1" --notes "Critical security patch"
```

## Git Tag Quick Reference

```bash
# Create
git tag -a v1.2.0 -m "Release v1.2.0"    # Annotated tag (recommended)
git tag v1.2.0                             # Lightweight tag

# List
git tag -l "v1.*"
git tag -n                                 # With annotations

# Push
git push origin v1.2.0                     # Single tag
git push origin --tags                     # All tags
git push origin --follow-tags              # Annotated tags only

# Delete
git tag -d v1.2.0                          # Local
git push origin --delete v1.2.0            # Remote
```

## Release Automation (semantic-release)

```bash
npm install --save-dev semantic-release \
    @semantic-release/changelog \
    @semantic-release/git \
    @semantic-release/github
```

`.releaserc.json`:
```json
{
  "branches": ["main"],
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    "@semantic-release/changelog",
    "@semantic-release/npm",
    "@semantic-release/github",
    ["@semantic-release/git", {
      "assets": ["CHANGELOG.md", "package.json"],
      "message": "chore(release): ${nextRelease.version} [skip ci]\n\n${nextRelease.notes}"
    }]
  ]
}
```
