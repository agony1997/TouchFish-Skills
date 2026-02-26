# Branch Strategies Reference

## Strategy Comparison

| Aspect | Git Flow | Trunk-Based | GitHub Flow |
|--------|----------|-------------|-------------|
| Complexity | High | Low | Low |
| Release cycle | Scheduled | Continuous | Continuous |
| Merge frequency | Low | High | High |
| CI/CD requirements | Medium | High | High |
| Best for | Large teams, scheduled releases | Fast-paced, mature CI/CD | SaaS, continuous deploy |

## Git Flow

```
main (production)
├── develop (integration branch)
│   ├── feature/* (new features, branch from develop)
│   ├── release/* (release prep, branch from develop)
│   └── hotfix/* (urgent fixes, branch from main)
```

> **Optional integration** — If superpowers plugin is installed, use `superpowers:using-git-worktrees` to isolate development in separate worktrees.

### Feature Development

```bash
git checkout develop && git checkout -b feature/user-auth
# ... develop ...
git checkout develop && git merge --no-ff feature/user-auth
git branch -d feature/user-auth
```

### Release Flow

```bash
git checkout develop && git checkout -b release/1.2.0
# Version bump, final fixes
git checkout main && git merge --no-ff release/1.2.0
git tag -a v1.2.0 -m "Release version 1.2.0"
git checkout develop && git merge --no-ff release/1.2.0
git branch -d release/1.2.0
git push origin main develop --tags
```

### Hotfix Flow

```bash
git checkout main && git checkout -b hotfix/security-fix
# Fix the issue
git checkout main && git merge --no-ff hotfix/security-fix
git tag -a v1.2.1 -m "Hotfix: security vulnerability"
git checkout develop && git merge --no-ff hotfix/security-fix
git branch -d hotfix/security-fix
git push origin main develop --tags
```

## Trunk-Based Development

- Single `main` branch (trunk)
- Short-lived feature branches (max 1-2 days)
- Merge to main multiple times per day
- Use feature flags for incomplete features

```bash
git checkout main && git pull origin main
git checkout -b feature/quick-fix
# Small, focused changes
git commit -m "feat: add user validation"
git push -u origin feature/quick-fix
# Create PR, quick review, merge, delete branch
```

## GitHub Flow

1. Create branch from `main`
2. Commit and push
3. Open PR
4. Review and deploy to staging
5. Merge to `main`
6. Auto-deploy to production
7. Delete branch

```bash
git checkout main && git pull && git checkout -b feature/payment
git commit -m "feat(payment): add Stripe integration"
git push -u origin feature/payment
gh pr create --title "Add Stripe payment integration"
# After approval:
gh pr merge --squash
```

## Branch Naming Conventions

```
<type>/<description>
<type>/<ticket-id>-<description>
```

| Prefix | Purpose | Example |
|--------|---------|---------|
| `feature/` | New feature | `feature/user-authentication` |
| `fix/` | Bug fix | `fix/login-validation-error` |
| `hotfix/` | Urgent fix | `hotfix/security-vulnerability` |
| `release/` | Release prep | `release/1.2.0` |
| `refactor/` | Refactoring | `refactor/user-service-cleanup` |
| `docs/` | Documentation | `docs/update-api-docs` |

Rules: lowercase, hyphens (not underscores or spaces), descriptive, include ticket ID when available.

## Merge Strategies

| Strategy | Command | When to Use |
|----------|---------|-------------|
| Merge commit | `git merge --no-ff feature/x` | Preserve branch history (Git Flow) |
| Fast-forward | `git merge feature/x` | Linear history (Trunk-Based) |
| Squash merge | `git merge --squash feature/x` | Clean history, many small commits (GitHub Flow) |
| Rebase merge | `git rebase main` then merge | Linear history preserving individual commits |

## Branch Cleanup

```bash
# Delete merged branches (exclude main/develop)
git branch --merged main | grep -v "main\|develop" | xargs git branch -d

# Prune remote tracking branches
git fetch --prune
```

> **Optional integration** — If superpowers plugin is installed, use `superpowers:finishing-a-development-branch` to guide the branch completion process (merge, PR, or cleanup).
