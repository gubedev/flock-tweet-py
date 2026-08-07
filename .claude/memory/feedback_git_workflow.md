---
name: feedback-git-workflow
description: Branch protection rules — one branch per phase, never commit to dev or main directly, PRs always target dev
metadata:
  type: feedback
---

Never commit directly to `dev` or `main`. Never open a PR targeting `main`. Use one branch per development phase.

**Why:** `main` is production. `dev` is the integration branch. Each phase has its own feature branch so the auditor can see a clean progression of PRs through the project history.

**How to apply:**
- Before starting any phase: `git checkout dev && git checkout -b <branch-name>`
- Branch naming per `ignore/plans/branching-strategy.md`
- `git push` and `gh pr create` always target `dev`
- If a command would modify `dev` or `main` directly, stop and inform the user
