# Pull Request Reference

## PR Creation Workflow

### Step 1: Pre-PR Analysis

```bash
git branch --show-current
git log main..HEAD --oneline
git diff main...HEAD --stat
git diff main...HEAD
git fetch origin && git status
```

Checklist: identify all commits, overall goal, modified files, affected modules, breaking changes, test coverage, documentation updates.

### Step 2: Generate PR Title

- Max 70 characters, imperative mood, specific
- Good: `Add employee leave request and approval workflow`
- Bad: `Added some new features for HR module`

### Step 3: Generate PR Description

```markdown
## Summary
<1-3 sentences>

## Changes
- Change 1
- Change 2

## Test Plan
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing completed

## Impact
- **Affected modules:** Module1, Module2
- **Breaking changes:** Yes/No
- **Database migrations:** Yes/No
- **Config changes:** Yes/No

## Related Issues
Closes #123

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

### Step 4: Create PR via GitHub CLI

```bash
# Push branch if needed
git push -u origin feature/leave-request

# Create PR
gh pr create --title "Add employee leave request functionality" --body "$(cat <<'EOF'
## Summary
Implement employee leave request and approval for HRM001.

## Changes
- Add LeaveRequest entity and repository
- Implement REST API endpoints
- Add frontend form and approval interface
- Add email notifications and tests

## Test Plan
- [x] Unit tests pass
- [x] Integration tests pass
- [x] Manual testing completed

## Impact
- **Affected modules:** HRM001, AU001, EMAIL
- **Breaking changes:** No
- **Database migrations:** Yes

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

Options: `--base develop`, `--draft`, `--reviewer john,jane`, `--label enhancement`, `--assignee @me`

### Step 5: Verify and Return PR URL

```bash
gh pr view
gh pr view --web
```

> **Optional integration** — If superpowers plugin is installed, use `superpowers:requesting-code-review` after PR creation for structured code review.

> **Optional integration** — If superpowers plugin is installed, use `superpowers:receiving-code-review` when handling review feedback to process it carefully.

## Code Review Checklist

**Critical (must fix):** Security vulnerabilities, data corruption risk, logic errors
**Important (should fix):** Performance issues, improper error handling, missing tests
**Minor (suggested):** Code style, naming, refactoring suggestions

Key review areas:
- Architecture and separation of concerns
- Code quality and readability
- Functional correctness and edge cases
- Test coverage (>80%)
- Performance (N+1 queries, caching)
- Security (SQL injection, XSS, authentication)
- Documentation

## Submitting Reviews

```bash
gh pr review 123 --approve --body "LGTM! Great tests."
gh pr review 123 --request-changes --body "Please fix the security issue."
gh pr review 123 --comment --body "A few questions..."
```

## PR Best Practices

- Keep PRs under 400 lines; one feature or fix per PR
- All commits follow Conventional Commits
- Keep branch in sync with base branch
- Self-review before requesting reviews

## Resolving Merge Conflicts

```bash
git checkout main && git pull origin main
git checkout feature/leave-request
git rebase main  # or merge (per project preference)
# Resolve conflicts...
git add <resolved-files> && git rebase --continue
git push --force-with-lease origin feature/leave-request
```
