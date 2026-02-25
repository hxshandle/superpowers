---
name: using-git-feature-branches
description: Use when starting feature work or bug fixes - creates isolated feature branches following Git Feature Branch Workflow best practices with safety checks and clean integration
---

# Using Git Feature Branch Workflow

## Overview

Git Feature Branch Workflow is the most popular git workflow used by development teams worldwide. Each feature or fix gets its own branch, keeping the main branch stable and enabling parallel development.

**Core principle:** Isolated development + clean integration = stable main branch.

**Announce at start:** "I'm using the using-git-feature-branches skill to set up a feature branch."

## When to Use

- Starting new feature development
- Bug fixes
- Experiments or proof-of-concepts
- Any work that should be isolated from main/master

## Branch Naming Conventions

Follow these widely-adopted naming patterns:

```bash
# Features
feature/user-authentication
feature/payment-integration
feature/dark-mode

# Bug fixes
bugfix/login-error
bugfix/memory-leak
fix/broken-navigation

# Hotfixes (urgent production fixes)
hotfix/security-patch
hotfix/critical-crash

# Experiments
experiment/new-algorithm
spike/performance-test
```

**Pattern:** `<type>/<short-descriptive-name>`

**Types:** feature, bugfix, fix, hotfix, experiment, spike, docs, refactor

## Setup Process

### 1. Verify Clean State

**MUST verify working directory is clean before creating branch:**

```bash
git status
```

**If uncommitted changes exist:**

Ask user:
```
You have uncommitted changes:
<list changes>

What would you like to do?
1. Stash changes and create branch
2. Commit changes first
3. Discard changes (careful!)
```

### 2. Update Main Branch

**Ensure local main/master is up-to-date:**

```bash
# Detect main branch name
MAIN_BRANCH=$(git symbolic-ref refs/remotes/origin/HEAD | sed 's@^refs/remotes/origin/@@')

# Fetch latest
git fetch origin

# Check if behind
BEHIND=$(git rev-list --count HEAD..origin/$MAIN_BRANCH)

if [ $BEHIND -gt 0 ]; then
  echo "Local $MAIN_BRANCH is $BEHIND commits behind origin"
  git checkout $MAIN_BRANCH
  git pull origin $MAIN_BRANCH
fi
```

### 3. Create Feature Branch

```bash
# Create and checkout new branch from updated main
git checkout -b <branch-name> $MAIN_BRANCH
```

**Verify branch created:**
```bash
git branch --show-current
```

### 4. Push Branch to Remote (Optional but Recommended)

```bash
# Set up tracking and push
git push -u origin <branch-name>
```

**Why:** Enables collaboration, backup, and CI/CD integration.

### 5. Run Project Setup

Auto-detect and run appropriate setup (dependencies may have changed):

```bash
# Node.js
if [ -f package.json ]; then 
  npm install
fi

# Python
if [ -f requirements.txt ]; then 
  pip install -r requirements.txt
fi
if [ -f pyproject.toml ]; then 
  poetry install
fi

# Ruby
if [ -f Gemfile ]; then 
  bundle install
fi

# Go
if [ -f go.mod ]; then 
  go mod download
fi

# Rust
if [ -f Cargo.toml ]; then 
  cargo build
fi
```

### 6. Verify Baseline

Run tests to ensure clean starting point:

```bash
# Use project-appropriate test command
npm test
pytest
cargo test
go test ./...
bundle exec rspec
```

**If tests fail:** Report failures, investigate before proceeding.

**If tests pass:** Ready to develop.

### 7. Report Status

```
Feature branch ready: <branch-name>
Based on: <main-branch> (commit: <short-hash>)
Tests passing: <N> tests, 0 failures
Ready to implement <feature-description>
```

## Development Workflow

### Making Changes

```bash
# Make changes to files
# Stage changes
git add <files>

# Commit with descriptive message
git commit -m "feat: add user authentication

- Implement login endpoint
- Add JWT token generation
- Include password hashing"
```

### Commit Message Best Practices

Follow Conventional Commits format:

```
<type>: <short description>

<optional longer description>

<optional footer>
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, etc.)
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `chore`: Maintenance tasks

**Examples:**
```
feat: add password reset functionality

fix: resolve memory leak in image processing

docs: update API documentation for auth endpoints

test: add integration tests for payment flow
```

### Keeping Branch Updated

**Regularly sync with main branch to avoid merge conflicts:**

```bash
# Fetch latest changes
git fetch origin

# Option 1: Rebase (cleaner history, preferred)
git rebase origin/$MAIN_BRANCH

# Option 2: Merge (preserves complete history)
git merge origin/$MAIN_BRANCH
```

**If conflicts occur during rebase:**
```bash
# Fix conflicts in files
# Stage resolved files
git add <resolved-files>
# Continue rebase
git rebase --continue
```

### Pushing Changes

```bash
# Push to remote feature branch
git push origin <branch-name>

# After rebase, may need force push (careful!)
git push --force-with-lease origin <branch-name>
```

**Note:** `--force-with-lease` is safer than `--force` - it fails if remote has changes you don't have locally.

## Integration Options

### Option 1: Pull Request / Merge Request (Recommended)

**Most popular for team development:**

1. Push branch to remote
2. Create PR/MR on GitHub/GitLab/Bitbucket
3. Request code review
4. Address feedback
5. Merge when approved

**Advantages:**
- Code review
- CI/CD validation
- Discussion and documentation
- Team visibility

### Option 2: Direct Merge (Small teams/solo)

```bash
# Switch to main branch
git checkout $MAIN_BRANCH

# Pull latest changes
git pull origin $MAIN_BRANCH

# Merge feature branch
git merge --no-ff <branch-name>

# Push to remote
git push origin $MAIN_BRANCH
```

**`--no-ff`**: Creates merge commit even for fast-forward, preserving branch history.

## Cleanup

### After Successful Merge

```bash
# Delete local branch
git branch -d <branch-name>

# Delete remote branch
git push origin --delete <branch-name>

# Clean up tracking branches
git fetch --prune
```

### If Branch No Longer Needed

```bash
# Force delete local branch (even if not merged)
git branch -D <branch-name>

# Delete remote branch
git push origin --delete <branch-name>
```

## Quick Reference

| Situation | Command |
|-----------|---------|
| Create feature branch | `git checkout -b feature/my-feature` |
| Push to remote | `git push -u origin feature/my-feature` |
| Update from main | `git rebase origin/main` or `git merge origin/main` |
| Check branch status | `git status` |
| List all branches | `git branch -a` |
| Switch branches | `git checkout <branch-name>` |
| Delete local branch | `git branch -d <branch-name>` |
| Delete remote branch | `git push origin --delete <branch-name>` |
| View branch history | `git log --oneline --graph --all` |

## Common Patterns

### Short-lived Branches
Best practice: Keep branches small and short-lived (1-3 days). Merge frequently.

### Long-running Branches
If feature takes weeks:
- Break into smaller sub-features
- Merge incrementally with feature flags
- Regularly rebase/merge from main

### Emergency Hotfixes
```bash
# Create from production tag or main
git checkout -b hotfix/critical-fix main
# Fix and test
git commit -m "hotfix: resolve critical security issue"
# Fast-track merge to main
git checkout main
git merge --no-ff hotfix/critical-fix
git tag -a v1.2.1 -m "Hotfix release"
git push origin main --tags
```

## Common Mistakes

### Creating branch from wrong base

- **Problem:** Branch based on another feature branch or outdated main
- **Fix:** Always create from updated main/master

### Forgetting to push branch

- **Problem:** Work only exists locally, no backup or CI/CD
- **Fix:** Push immediately after creating branch

### Mixing unrelated changes

- **Problem:** Multiple features in one branch makes review/revert difficult
- **Fix:** One feature per branch, create separate branches for different work

### Merge conflicts accumulate

- **Problem:** Branch diverges too far from main
- **Fix:** Regularly rebase/merge from main (daily for active branches)

### Force pushing without lease

- **Problem:** Overwrites collaborator's work
- **Fix:** Use `--force-with-lease` instead of `--force`

## Advanced: Interactive Rebase

Clean up commits before merging:

```bash
# Rebase last 3 commits
git rebase -i HEAD~3

# Options:
# pick = keep commit
# reword = change commit message
# squash = combine with previous commit
# fixup = like squash but discard message
# drop = remove commit
```

## Verification Checklist

Before merging feature branch:

- [ ] All commits have clear, descriptive messages
- [ ] Branch is up-to-date with main/master
- [ ] All tests pass
- [ ] Code follows project style guidelines
- [ ] No merge conflicts
- [ ] Changes reviewed (if team workflow)
- [ ] CI/CD checks pass (if configured)
- [ ] Documentation updated if needed

## Integration with Other Skills

**Called by:**
- **brainstorming** - When design approved and ready to implement
- **executing-plans** - Before executing implementation plans
- **subagent-driven-development** - Alternative to worktrees for isolation

**Pairs with:**
- **test-driven-development** - Write tests on feature branch
- **requesting-code-review** - Request review of feature branch
- **finishing-a-development-branch** - Clean merge and deletion

**Alternative to:**
- **using-git-worktrees** - Traditional worktree-based isolation strategy

## Red Flags

**Never:**
- Create branch without checking working directory is clean
- Create branch from outdated main
- Mix unrelated features in one branch
- Let branch diverge weeks from main without syncing
- Force push without `--force-with-lease`
- Merge without testing
- Delete branch before confirming merge success

**Always:**
- Verify clean state before branching
- Update main before creating branch
- Use descriptive branch names
- Commit frequently with clear messages
- Sync regularly with main
- Run tests before merging
- Clean up branches after merge

## Troubleshooting

### "Branch already exists"
```bash
# Check if branch exists
git branch -a | grep <branch-name>
# Use different name or delete old branch
git branch -D <branch-name>
```

### "Cannot push - diverged"
```bash
# Check what diverged
git log origin/<branch-name>..HEAD
git log HEAD..origin/<branch-name>
# Pull and merge or use force-with-lease
git pull --rebase origin <branch-name>
```

### "Merge conflicts"
```bash
# See conflicting files
git status
# Edit files, remove conflict markers
# Stage resolved files
git add <files>
# Complete merge/rebase
git rebase --continue  # or git merge --continue
```

## Why This Workflow is Popular

1. **Simple**: Easy to learn, no complex branching rules
2. **Flexible**: Works for teams of any size
3. **Safe**: Main branch stays stable and deployable
4. **Collaborative**: Perfect for pull request workflows
5. **Supported**: Built-in to all major git hosting platforms
6. **Proven**: Used by thousands of successful projects

GitHub Flow, GitLab Flow, and many company workflows are variations of this pattern.