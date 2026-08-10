---
name: using-git-worktrees
description: Set up an isolated workspace via git worktree before starting a phase. Worktrees live in .claude/temp/worktrees/.
version: 1.0.0
---

# Using Git Worktrees

## Overview

Ensure each phase is implemented in an isolated workspace. This prevents in-progress changes from polluting `dev` or other branches.

**Standard location:** `<repo>/.claude/temp/worktrees/<branch-name>`

**Announce at start:** "I'm using the using-git-worktrees skill to set up an isolated workspace."

---

## Step 0: Detect Existing Isolation

Before creating anything, check if you are already in a worktree:

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
```

- If `GIT_DIR != GIT_COMMON` (and not a submodule): already in a worktree — skip to Step 3.
- If `GIT_DIR == GIT_COMMON`: proceed to Step 1.

---

## Step 1: Safety Check

Verify `.claude/temp/worktrees/` is git-ignored:

```bash
git check-ignore -q .claude/temp/worktrees 2>/dev/null
```

If NOT ignored: stop and add `.claude/temp/` to `.gitignore` before continuing.

---

## Step 2: Create the Worktree

```bash
SOURCE_ROOT=$(git rev-parse --show-toplevel)
BRANCH_NAME=<branch-for-this-phase>   # e.g. feat/auth
LOCATION="$SOURCE_ROOT/.claude/temp/worktrees"

mkdir -p "$LOCATION"

# Create the branch from dev if it doesn't exist yet
git checkout dev
git checkout -b "$BRANCH_NAME"
git checkout dev  # go back so the worktree starts on the new branch

# Create the worktree
git worktree add "$LOCATION/$BRANCH_NAME" "$BRANCH_NAME"
cd "$LOCATION/$BRANCH_NAME"
```

Copy Claude settings from main checkout:

```bash
for f in ".claude/settings.json" ".claude/settings.local.json"; do
  [ -f "$SOURCE_ROOT/$f" ] && cp -p "$SOURCE_ROOT/$f" "./$f"
done
```

---

## Step 3: Project Setup

```bash
# Backend (if working on backend phase)
cd backend && pip install -r requirements.txt

# Frontend (if working on frontend phase)
cd frontend && npm install
```

---

## Step 4: Implement, Test, Commit

Work normally inside the worktree on `$BRANCH_NAME`. When done:

```bash
git add <files>
git commit -m "feat(...): ..."
git push -u origin "$BRANCH_NAME"
gh pr create --base dev --title "..."
```

---

## Step 5: Cleanup After PR Merge

```bash
# Verify nothing unsaved
git status --porcelain   # must be empty
git log @{u}..           # must be empty

WORKTREE_PATH=$(git rev-parse --show-toplevel)

cd "$SOURCE_ROOT"
git worktree remove "$WORKTREE_PATH"
git worktree prune
git branch -d "$BRANCH_NAME"   # safe delete (refuses if unmerged)
```

Verify:
```bash
git worktree list   # WORKTREE_PATH must not appear
```

---

## Quick Reference

| Situation | Action |
|---|---|
| Already in worktree (Step 0) | Skip creation |
| `.claude/temp/` not in .gitignore | Add it before creating worktree |
| Uncommitted changes at cleanup | Stop and ask user |
| Branch not yet merged | Use `git branch -d` (safe), not `-D` |
