---
name: git-operations-safety
description: Git operation safety standards. Use before executing any Git operation, especially destructive ones (reset, force push, clean). Must run git status before operations, destructive operations require explicit user confirmation, special restrictions on force push to main/master branches.
allowed-tools: Bash
---

# Git Operations Safety

Ensure Git operation safety and recoverability to prevent data loss.

## Destructive Operations (Require Confirmation)

Following operations cause data loss or history changes, **must** get explicit user confirmation:

| Operation | Risk | Impact |
|-----------|------|--------|
| `git reset --hard` | Lose all uncommitted changes | ⚠️ High risk - unrecoverable |
| `git push --force` | Overwrite remote history | ⚠️ High risk - affects team |
| `git clean -fd` | Delete untracked files | ⚠️ High risk - permanent deletion |
| `git branch -D` | Force delete branch | ⚠️ Medium risk - lose unmerged work |

## Detailed Explanations

### 1. git reset --hard

**Risk**: Lose all uncommitted changes and staged content

**Safe Alternatives**:
```bash
# ✅ Use stash to save changes
git stash push -m "Temporary save"
git reset --hard HEAD

# If need to recover:
git stash pop

# ✅ Create backup branch
git branch backup-$(date +%Y%m%d-%H%M%S)
git reset --hard HEAD
```

**Confirmation Process**:
1. Run `git status` to see what will be lost
2. Show user list of files that will be lost
3. Ask: "This will lose following uncommitted changes, continue?"
4. Only execute after explicit confirmation

### 2. git push --force

**Risk**: Overwrite remote history, affects other team members

**Safe Alternative**:
```bash
# ✅ Use --force-with-lease (safer)
git push --force-with-lease
```

**Special Restrictions**:
```bash
# ❌❌❌ Absolutely prohibit force push to main branch
git push --force origin main
git push --force origin master

# Must warn and require secondary confirmation
```

**Warning Template**:
```
🚨 Critical Warning:

You are attempting to force push to main branch, this is extremely dangerous!

Possible consequences:
- Overwrite other team members' commits
- Cause conflicts in team members' local repositories
- May lose important code changes

This operation should typically NOT be performed.

If you really need to do this:
1. Notify all team members
2. Confirm nobody is working based on main branch
3. Ensure complete backup exists

Do you really want to continue?
```

### 3. git clean -fd

**Risk**: Delete all untracked files and directories, unrecoverable

**Safe Practice**:
```bash
# ✅ Preview first
git clean -fd --dry-run

# Execute after confirmation
git clean -fd
```

## Safe Practices

### 1. Check Before Operations

**Rule**: Always run `git status` before any Git operation

```bash
# ✅ Standard workflow
git status                    # 1. Check status first
git add .                     # 2. Stage changes
git status                    # 3. Confirm again
git commit -m "message"       # 4. Commit
```

**git status tells us**:
- Which files were modified
- Which files are staged
- Which files are untracked
- Current branch status
- Whether ahead or behind remote branch

### 2. Preview Before Commit

```bash
# ✅ View changes to be committed
git diff                      # Working directory changes
git diff --staged             # Staged changes
git status                    # File status
```

### 3. Use stash to Save Temporary Work

```bash
# ✅ Save current work
git stash push -m "Work in progress"

# Switch branch for other work
git checkout other-branch

# Restore when back
git stash pop
```

### 4. Create Backup Branch

```bash
# ✅ Create backup before dangerous operations
git branch backup-before-rebase

# Perform operation
git rebase main

# If error, can recover
git reset --hard backup-before-rebase
```

### 5. Use reflog for Recovery

```bash
# ✅ View operation history
git reflog

# Recover to previous state
git reset --hard HEAD@{2}
```

## Safety Checklists

### Before Commit

- [ ] Run `git status`
- [ ] Run `git diff` to view changes
- [ ] Confirm all changes are intentional
- [ ] Confirm no sensitive info (passwords, keys)
- [ ] Confirm no files that shouldn't be committed
- [ ] Commit message follows conventions

### Before Push

- [ ] Passed all local quality checks
- [ ] Confirmed correct branch to push
- [ ] If force push, confirmed impact scope
- [ ] Not pushing to main/master (unless have permission)
- [ ] Notified team members (if shared branch)

### Before Destructive Operations

- [ ] Run `git status` to understand current state
- [ ] Created backup branch or used stash
- [ ] Understand exact impact of operation
- [ ] Got explicit user confirmation
- [ ] Prepared rollback plan

## Recovery Procedures

### Recover Accidentally Deleted Commits

```bash
# View operation history
git reflog

# Recover to before deletion
git reset --hard HEAD@{1}
```

### Recover Commits Overwritten by Force Push

```bash
# On affected member's machine
git reflog

# Create branch to save overwritten commits
git branch recovery <commit-hash>
```

## Examples

### Safe Commit Workflow

```bash
# 1. Check status
git status

# 2. View changes
git diff

# 3. Add files
git add src/auth.ts src/user.ts

# 4. Confirm again
git status

# 5. Commit
git commit -m "feat(auth): add user authentication"

# 6. Confirm before push
git status

# 7. Push
git push origin feature-auth
```

### Handling Dangerous Operation Requests

```
Scenario: User requests "git reset --hard"

✅ Correct handling:
1. Run git status
2. Show what will be lost:
   "⚠️ Warning: Will lose following uncommitted changes:
   - src/auth.ts
   - src/user.ts

   Continue?"
3. Wait for user confirmation
4. Suggest backup first
5. Execute after user confirms
```

## Recommended Tools

### Git Hooks

```bash
# pre-push hook: Block direct push to main
#!/bin/sh
protected_branch='main'
current_branch=$(git symbolic-ref HEAD | sed -e 's,.*/\(.*\),\1,')

if [ $current_branch = $protected_branch ]; then
    echo "⚠️  Direct push to main not allowed!"
    exit 1
fi
```

### Git Aliases

```bash
# ~/.gitconfig
[alias]
    # Safe force push
    pushf = push --force-with-lease

    # Preview before commit
    preview = diff --staged

    # Safe clean
    cleancheck = clean -fd --dry-run
```

## Summary

| Principle | Description |
|-----------|-------------|
| **Check before operations** | Always run `git status` first |
| **Preview changes** | Use `git diff` to view changes |
| **Create backups** | Backup before dangerous operations |
| **Get confirmation** | Destructive operations must confirm |
| **Protect main branch** | Special protection for main/master |
| **Recoverability** | Understand reflog recovery mechanism |
