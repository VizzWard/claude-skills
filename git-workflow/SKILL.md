---
name: git-workflow
description: >
  Complete Git and GitHub workflow guide for projects using modern web stacks.
  ALWAYS use this skill when the user mentions: commit, branch, PR, pull request,
  issue, merge, push, rebase, stash, cherry-pick, release, tag, versioning,
  CHANGELOG, GitHub Actions, CI, checks, labels, GitHub Projects, create task,
  create branch, open PR, merge changes, deploy, hotfix, or any git/GitHub action.
  Also trigger when the user asks how to do something in git, what type of branch
  to create, how to name a commit, whether to create an issue first, or how to
  organize work in the repository. This skill contains the exact conventions,
  labels, checklists and flows the project uses — do not assume generic defaults.
---

# Git Workflow

> Designed for: React · Django REST Framework · PostgreSQL · Redis · Docker
> Adaptable to any project — adjust stack reference and labels to fit your setup.

Read `references/GIT_WORKFLOW.md` for the full workflow. Below are the essential rules that apply to every git action.

---

## Rules That Are Never Broken

1. Never work directly on `main`
2. Every significant task starts with an **issue**
3. One branch = one purpose. One commit = one complete idea. One PR = one reviewable delivery
4. **Do not merge if CI checks fail** — no exceptions
5. `main` is always stable and deployable
6. Every PR must include `Closes #N` to automatically close the linked issue

---

## Quick Reference

### Branches — format `type/short-description`
`feature` · `fix` · `improve` · `refactor` · `test` · `docs` · `chore`

### Commits — Conventional Commits
```
<type>(<scope>): short description in imperative mood
feat · fix · refactor · test · docs · chore
```

### Labels — always assign one from each relevant group

**Type:** `type: feature` · `type: fix` · `type: improve` · `type: chore` · `type: docs` · `type: test`

**Priority:** `priority: high` · `priority: medium` · `priority: low`

**Area:** `area: backend` · `area: frontend` · `area: infra` · `area: repo`

**GitHub default:** `bug` · `enhancement` · `documentation` · `duplicate` · `help wanted` · `invalid` · `question` · `wontfix`

> **Note:** The Area and Type labels above are custom. Adapt them to your project's structure.
> You can create them via: `gh label create "type: feature" --color "#0075ca"`

---

## Standard Flow (full cycle)

```bash
# 1. Create issue first
gh issue create --title "type(scope): description" \
  --label "type:feature,priority:medium,area:backend"

# 2. Branch from updated main
git checkout main && git pull
git checkout -b feature/name

# 3. Atomic commits
git add file.py
git commit -m "feat(scope): description in imperative mood"

# 4. Push and PR with issue close reference
git push -u origin feature/name
gh pr create --title "feat(scope): description" \
  --body "...\n\nCloses #N"

# 5. Verify CI before merging
gh pr checks --watch

# 6. Merge only if CI is green
gh pr merge --merge

# 7. Clean up
git checkout main && git pull
git branch -d feature/name
```

---

## Issues — create with gh

```bash
gh issue create \
  --title "type(scope): description" \
  --body "$(cat <<'EOF'
## Description
What needs to be done and why.

## Acceptance criteria
- [ ] Criterion 1
- [ ] Criterion 2

## Technical notes (optional)
Relevant context, prior decisions, references.
EOF
)" \
  --label "type:feature,priority:medium,area:frontend"
```

**GitHub Projects note:** If the repo has a Default Project set under Settings, new issues are added to the project automatically. Status and Iteration fields must be assigned manually inside the project.

---

## CI / GitHub Actions — checking PR status

```bash
gh pr checks             # view check status
gh pr checks --watch     # wait for results in real time
gh run list              # list recent runs
gh run view <id> --log   # view logs from a failed run
```

**If a check fails:** fix → commit → push → wait for new run → verify green → merge.

---

## CHANGELOG — update on every release

- Keep an `[Unreleased]` section during active development
- On release: rename to `[X.Y.Z] - DATE`, add a new empty `[Unreleased]` section above
- Categories: `Added` · `Changed` · `Fixed` · `Removed` · `Security`
- Commit alongside the VERSION bump: `git commit -m "chore: release vX.Y.Z"`

---

## Quick Checklist

**Issue:** formatted title · acceptance criteria · labels (type + priority + area)

**Branch:** starts from updated main · `type/description` name · purpose in one sentence

**Commit:** atomic · conventional format · no unrelated changes mixed in

**PR:** clear title · body with Summary/Changes/How to test · `Closes #N`

**Merge:** CI green · branch up to date with main

**Release:** features merged · CHANGELOG updated · VERSION bumped · tag created

---

## Full Detail

For detailed flows (stash, rebase, cherry-pick, conflict resolution, versioning, branch protection rules), see:
→ `references/GIT_WORKFLOW.md`
