# Claude Development Guidelines

## Branch Workflow

### Branch Map
- **main**: Production branch - DO NOT commit directly to this branch
- **dev**: Working branch / Integration branch
- **feat/***: Feature branches for new features and changes

### Branch Rules

**CRITICAL: Always check current branch before starting work**

1. **Default workflow**: Always create and work in a feature branch (`feat/*`) unless explicitly told to use the `dev` branch
2. **Never commit directly to `main`** - This is the production branch
3. **Feature branches**: Create feature branches from `dev` with descriptive names:
   - `feat/setup-authentication`
   - `feat/add-user-dashboard`
   - `feat/api-endpoints`

### Pre-work Checklist

Before starting ANY work:
1. Check current branch with `git branch` or `git status`
2. If not on a feature branch, create one from `dev`:
   ```bash
   git checkout dev
   git pull origin dev
   git checkout -b feat/descriptive-name
   ```
3. If explicitly told to use `dev` branch, ensure you're on `dev`

## Commit and Push Requirements

**ALWAYS commit and push changes when:**
- A feature or task is completed
- Significant progress has been made
- User requests a commit
- End of a work session

### Commit Workflow

1. **Stage changes**: `git add .` (or specific files)
2. **Commit with clear message**: `git commit -m "descriptive message"`
3. **ALWAYS push immediately after commit**: `git push` (or `git push -u origin branch-name` for new branches)

**Never skip the push step** - commits should be backed up to remote immediately.

### Commit Message Guidelines

- Use clear, descriptive messages
- Focus on the "why" rather than the "what"
- Examples:
  - ✅ "Add user authentication with JWT tokens"
  - ✅ "Fix pagination bug in product listing"
  - ✅ "Refactor API endpoints for better error handling"
  - ❌ "update files"
  - ❌ "fixes"

## Pull Request Workflow

When a feature is complete:
1. Ensure all changes are committed and pushed
2. Create a PR from your feature branch to `dev`
3. Never create PRs directly to `main` unless explicitly instructed

## Quick Reference

```bash
# Start new feature
git checkout dev
git pull origin dev
git checkout -b feat/my-feature

# After making changes
git add .
git commit -m "Clear description of changes"
git push  # or git push -u origin feat/my-feature for first push

# Check current branch
git branch
git status
```

## Summary

- ✅ Always work in feature branches (`feat/*`)
- ✅ Always commit AND push changes
- ✅ Use clear commit messages
- ❌ Never commit directly to `main`
- ❌ Never skip pushing after committing
