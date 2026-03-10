# Git Workflow Guide

> **Stack:** React · Django REST Framework · PostgreSQL · Redis · Docker
> **Goal:** Maintain a structured, clean, and navigable change history.

---

## 1. Fundamental Rules

- Never work directly on `main`
- Every change goes on a **branch**
- One **branch = one clear purpose** (if you can't describe it in one sentence, it's multiple branches)
- One **commit = one complete idea** (compiles, works, can be reverted without breaking anything)
- One **PR = one feature or responsibility**
- `main` must always be **stable and deployable**
- Every significant task starts with an **issue**
- Do not merge to `main` if **CI checks fail**

---

## 2. Issues and GitHub Projects

### What is an issue

An issue is the entry point for any work. It documents WHAT needs to be done and WHY, before writing a single line of code.

### When to create an issue

- New functionality
- Reported bug
- Improvement to something existing
- Maintenance task or technical debt
- Any work that takes more than 30 minutes

### Creating an issue

```bash
# From the terminal with GitHub CLI
gh issue create \
  --title "type(scope): short description" \
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
  --label "type:feature,priority:medium,area:backend"

# List open issues
gh issue list

# View issue detail
gh issue view 42
```

### Available Labels

Always assign **one label from each relevant group** when creating an issue.

#### GitHub Default Labels

| Label            | Use                                        |
|------------------|--------------------------------------------|
| `bug`            | Something isn't working                    |
| `documentation`  | Improvements or additions to documentation |
| `duplicate`      | Issue or PR already exists                 |
| `enhancement`    | New feature or request                     |
| `good first issue` | Good for newcomers                       |
| `help wanted`    | Extra attention is needed                  |
| `invalid`        | This doesn't seem right                    |
| `question`       | Further information is requested           |
| `wontfix`        | This will not be worked on                 |

#### Custom Labels — Priority

| Label             | When to use                  |
|-------------------|------------------------------|
| `priority: high`  | High impact, do first        |
| `priority: medium`| Medium impact                |
| `priority: low`   | Low impact, can wait         |

#### Custom Labels — Type

| Label          | When to use                          |
|----------------|--------------------------------------|
| `type: feature`| New functionality                    |
| `type: fix`    | Bug correction                       |
| `type: improve`| Improvement to existing functionality|
| `type: chore`  | Maintenance task                     |
| `type: docs`   | Documentation                        |
| `type: test`   | Tests                                |

#### Custom Labels — Area

| Label            | When to use                            |
|------------------|----------------------------------------|
| `area: backend`  | Django backend                         |
| `area: frontend` | React frontend                         |
| `area: infra`    | Infrastructure, CI/CD, Docker          |
| `area: repo`     | Repository, documentation, git         |

### GitHub Projects — automatic assignment

If the repository has a **Default Project** configured under Settings, new issues are added to the project automatically with no manual step required.

What you **do** need to assign manually (or via project automation):
- **Status** (Todo / In Progress / Done)
- **Iteration** or sprint if used

### Full cycle: issue → branch → PR → auto-close

```
Issue #42 created
    ↓
Branch: feature/feature-name  (referenced in PR)
    ↓
Atomic commits
    ↓
PR with "Closes #42" in body  ← automatically closes the issue on merge
    ↓
CI green → merge → issue closed automatically
```

---

## 3. Branch Convention

### Format

```
<type>/<short-description>
```

### Types

| Type       | Use                          | Example                              |
|------------|------------------------------|--------------------------------------|
| `feature`  | New functionality            | `feature/pdf-export`                 |
| `fix`      | Bug correction               | `fix/ghost-segments`                 |
| `improve`  | Improvement to existing      | `improve/stations-view`              |
| `refactor` | Internal restructuring       | `refactor/kpis-backend`              |
| `test`     | Tests                        | `test/tracking-services`             |
| `docs`     | Documentation                | `docs/api-endpoints`                 |
| `chore`    | Minor tasks, dependencies    | `chore/update-dependencies`          |

### Scope rule

Before creating a branch, answer: "What does this branch do?"

- **Good:** "Adds the detailed driver profile view"
- **Bad:** "Improves several things in KPIs, stations, profiles and PDFs"

If your answer includes "and also...", you need more than one branch.

---

## 4. Standard Workflow

### Full cycle (with issue)

```bash
# 0. Create issue first (if it doesn't exist)
gh issue create --title "feat(drivers): add KPI history tab" ...

# 1. Always start from updated main
git checkout main
git pull

# 2. Create branch
git checkout -b feature/kpi-history-tab

# 3. Work with atomic commits
git add file1.py file2.py
git commit -m "feat(drivers): add KPI history tab with period filtering"

# 4. Push branch to remote
git push -u origin feature/kpi-history-tab

# 5. Create PR (linked to the issue)
gh pr create \
  --title "feat(drivers): add KPI history tab" \
  --body "$(cat <<'EOF'
## Summary
...

Closes #42
EOF
)"

# 6. Verify CI checks pass
gh pr checks

# 7. Merge (only if CI is green)
gh pr merge --merge

# 8. Clean up
git checkout main
git pull
git branch -d feature/kpi-history-tab
```

### Flow diagram

```
Issue #N created
    |
main ──○──────────●──────────●──────────●──── ...
       |          ↑          ↑          ↑
       |        merge      merge      merge
       |     (CI green)  (CI green) (CI green)
       |          |          |          |
       ├── feature/pdf-export (PR #1, Closes #10)
       |                     |          |
       ├───── fix/pdf-layout (PR #2, Closes #11)
       |                                |
       └──────── feature/profiles (PR #3, Closes #12)
```

---

## 5. Essential Git Tools

### git stash — Save temporary work

Saves your changes without committing. Like putting your work in a temporary box.

**When to use:** You need to switch branches but have uncommitted changes.

```bash
# Save changes in the "box"
git stash

# See what you have saved
git stash list

# Restore changes (takes from box and removes it)
git stash pop

# Restore without deleting the stash (in case something fails)
git stash apply
```

**Real example:**
```bash
# You're working on feature/kpis and notice a bug in PDFs
git stash                          # save your current work
git checkout main                  # go to main
git checkout -b fix/pdf-layout     # create branch for the fix
# ... fix, commit, push, PR, merge ...
git checkout feature/kpis          # return to your feature
git rebase main                    # incorporate the fix
git stash pop                      # recover your work in progress
```

### git rebase — Rebase your branch

Takes your commits and places them on top of the latest from another branch.

**When to use:** You want to incorporate changes that were merged to main while you were working.

```bash
# You're on feature/my-feature
git rebase main
```

**Before rebase:**
```
main    ── A ── B ── C ── D (new)
                \
feature          E ── F ── G (your commits)
```

**After rebase:**
```
main    ── A ── B ── C ── D
                            \
feature                      E' ── F' ── G' (your commits relocated)
```

**Difference from merge:**
- `merge` creates an extra merge commit and leaves branching in the history
- `rebase` rewrites history to be linear and clean
- Use `rebase` to update your branch with main
- Use `merge` (via PR) to integrate your branch into main

**Important rule:** Never rebase a branch you've already shared with others. Rebase rewrites history.

**If there are conflicts during rebase:**
```bash
git add resolved_file.py
git rebase --continue

# If everything goes wrong and you want to cancel:
git rebase --abort
```

### git cherry-pick — Copy a specific commit

Copies ONE commit from another branch to your current branch.

**When to use:** You need a specific fix that's on another branch, without bringing everything else.

```bash
git cherry-pick abc1234    # copies commit abc1234 to your current branch
```

---

## 6. What to Do When an Unrelated Change Appears

This is the most common situation: you're working on your feature and discover something that needs fixing or improving, but it's unrelated to your branch.

### Decision tree

```
You discover an unrelated change
|
├── Is it BLOCKING? (can't continue without it)
|   └── YES: Separate branch, fast
|       git stash
|       git checkout main && git pull
|       git checkout -b fix/the-problem
|       # fix, commit, push, PR, merge
|       git checkout feature/your-feature
|       git rebase main
|       git stash pop
|
├── Is it small but NOT blocking?
|   └── Log it (issue, TODO, note) and do it LATER
|       in its own branch
|
└── Is it large (new feature, redesign)?
    └── Definitely separate branch, later
        Don't mix it with your current work
```

---

## 7. Commits

### Format (Conventional Commits)

```
<type>(<scope>): short description in imperative mood
```

### Types

| Type       | When to use                                   |
|------------|-----------------------------------------------|
| `feat`     | New functionality                             |
| `fix`      | Bug correction                                |
| `refactor` | Internal change without changing behavior     |
| `test`     | Add or modify tests                           |
| `docs`     | Documentation only                            |
| `chore`    | Dependencies, config, minor tasks             |

### Rules

1. **Atomic:** each commit compiles and makes sense on its own
2. **Descriptive:** explains the WHAT, not the HOW
3. **Imperative mood:** "add", "fix", "update", not "added", "fixing"
4. **Docs go with their feature:** don't make standalone doc commits

### Examples

```bash
# Good
git commit -m "feat(drivers): add KPI history tab with period filtering"
git commit -m "fix(tracking): prevent ghost segments in battery segmentation"
git commit -m "refactor(kpis): move score calculation to backend service"

# Bad
git commit -m "update files"
git commit -m "WIP"
git commit -m "changes"
git commit -m "feat(drivers): add KPI history tab and also fix stations and update PDFs"
```

### Commits with description (for complex changes)

```bash
git commit -m "$(cat <<'EOF'
feat(drivers): add KPI history tab with period filtering

- Add KpiHistoryTab component with date range selector
- Create backend endpoint GET /api/drivers/{id}/kpi-history/
- Include summary stats (average, trend, best period)
- Filter by: last week, last month, last quarter, custom
EOF
)"
```

---

## 8. Pull Requests (PR)

### What is a PR

A Pull Request is a formal request to integrate the changes from your branch into main. It provides:

- **Review:** a space to review all changes before integrating them
- **Documentation:** a record of WHAT changed and WHY
- **Clean history:** each merge to main has an associated PR explaining context
- **Checkpoint:** CI must pass before merging

### PR flow

```bash
# 1. Work on your branch normally
git checkout -b feature/my-feature
# ... commits ...

# 2. Push your branch
git push -u origin feature/my-feature

# 3. Create PR linked to the issue
gh pr create --title "feat(module): short description" --body "$(cat <<'EOF'
## Summary
Description of what was done and why.

## Main changes
- Change 1
- Change 2

## How to test
- Step 1
- Step 2

Closes #N
EOF
)"

# 4. Verify CI checks (see section 9)
gh pr checks

# 5. Merge ONLY if CI is green
gh pr merge --merge

# 6. Clean up
git checkout main
git pull
git branch -d feature/my-feature
```

### PR structure

```markdown
## Summary
Brief description of the purpose (1-2 sentences).

## Main changes
- What was added/modified/removed (concrete list)

## How to test
- Steps to verify it works

## Notes (optional)
- Relevant technical decisions
- Things pending for future branches

Closes #N
```

### Direct merge vs PR

| Situation                              | Use           |
|----------------------------------------|---------------|
| New feature                            | PR            |
| Bug fix                                | PR            |
| Trivial 1-line change (typo)           | Direct merge  |
| Version update                         | Direct merge  |
| Any change that affects logic          | PR            |

### Useful GitHub CLI (gh) commands

```bash
gh pr create --title "title" --body "description"
gh pr list
gh pr view 123
gh pr checks 123
gh pr merge 123 --merge
```

---

## 9. CI / GitHub Actions — Checking PR status

### Fundamental rule

> **Do not merge to `main` if CI checks fail.**

Checks verify that your changes didn't break anything: tests, linting, build, etc.

### How to verify checks

```bash
# View all checks for the current PR (from the branch)
gh pr checks

# View checks for a specific PR by number
gh pr checks 42

# Wait for checks to finish and show result
gh pr checks --watch
```

### Interpreting results

```
✓  backend-tests      2m15s   pass    — OK, proceed
✓  frontend-tests     1m42s   pass    — OK, proceed
✗  lint-check         0m30s   fail    — STOP: fix before merging
-  docker-build       pending         — Waiting for result
```

### Flow when a check fails

```bash
# 1. View failure detail
gh run list                      # list recent runs
gh run view <run-id>             # run detail
gh run view <run-id> --log       # view full logs

# 2. Fix locally
# ... make the fix ...

# 3. Commit and push (CI runs automatically)
git add .
git commit -m "fix(tests): correct assertion in tracking service test"
git push

# 4. Check again
gh pr checks --watch

# 5. Merge when all checks are green
gh pr merge --merge
```

---

## 10. Branch Protection Rules

### What they are

Rules configured in GitHub that **prevent** merging to `main` without meeting certain requirements.

### Recommended configuration

In GitHub: `Settings > Branches > Add branch protection rule > main`

| Option                                        | Recommended state  |
|-----------------------------------------------|--------------------|
| Require a pull request before merging         | ✅ Enabled         |
| Require status checks to pass before merging  | ✅ Enabled         |
| Require branches to be up to date             | ✅ Enabled         |
| Do not allow bypassing the above settings     | ✅ Enabled         |
| Allow force pushes                            | ❌ Disabled        |
| Allow deletions                               | ❌ Disabled        |

### Practical effect

With these rules active, GitHub will **physically block** the merge button if:
- CI checks have not passed
- The branch is not up to date with main

---

## 11. CHANGELOG

### What it's for

Records in human language what changed in each version. Different from commit history: it's meant to be read, not scanned.

### Format (Keep a Changelog)

The `CHANGELOG.md` file lives at the root of the project.

```markdown
# Changelog

## [Unreleased]
### Added
- Description of feature in progress or merged but not yet released

## [1.2.0] - 2025-01-15
### Added
- PDF export for the 5 history views
- Endpoint POST /api/tracking/history/{type}/pdf/

### Fixed
- Ghost segments in battery segmentation

### Changed
- KPI score calculation moved to backend service

## [1.1.0] - 2024-11-20
### Added
- Circuits: cloning, templates, Excel export
```

### Available categories

| Category     | When to use                                       |
|--------------|---------------------------------------------------|
| `Added`      | New features                                      |
| `Changed`    | Changes to existing functionality                 |
| `Fixed`      | Bug fixes                                         |
| `Removed`    | Removed functionality                             |
| `Deprecated` | Functionality to be removed in the future         |
| `Security`   | Security patches                                  |

### CHANGELOG update flow

```bash
# During normal development: update [Unreleased] section
# On release:
#   1. Rename [Unreleased] to [X.Y.Z] - DATE
#   2. Add new empty [Unreleased] section above
#   3. Commit alongside version bump

git add CHANGELOG.md VERSION
git commit -m "chore: release v1.2.0"
git tag v1.2.0
git push && git push --tags
```

---

## 12. Versioning (Semantic Versioning)

### Format

```
MAJOR.MINOR.PATCH[-prerelease]
```

| Component      | When to increment                              | Example           |
|----------------|------------------------------------------------|-------------------|
| **MAJOR**      | Breaking changes                               | `1.0.0` → `2.0.0` |
| **MINOR**      | New functionality (without breaking existing)  | `1.1.0` → `1.2.0` |
| **PATCH**      | Bug fixes, minor adjustments                   | `1.2.0` → `1.2.1` |
| **prerelease** | Version in development/testing                 | `1.2.0-beta.1`    |

### When to update the version

```
DO NOT update:
  - On every commit
  - On every individual PR

DO update:
  - When a set of features/fixes is complete and tested
  - When deploying to production
  - When you need to mark a stable reference point
```

### Versioning flow

```bash
git checkout main
git pull
echo "1.2.0" > VERSION
git add VERSION CHANGELOG.md
git commit -m "chore: release v1.2.0"
git tag v1.2.0
git push && git push --tags
```

### Pre-releases (beta)

```bash
echo "1.3.0-beta.1" > VERSION
git commit -m "chore: bump version to 1.3.0-beta.1"
git tag v1.3.0-beta.1

echo "1.3.0" > VERSION
git commit -m "chore: release v1.3.0"
git tag v1.3.0
```

### Quick guide: which number to change

```
"I added 3 new features"                  → MINOR: 1.2.0 → 1.3.0
"I fixed 2 bugs"                          → PATCH: 1.3.0 → 1.3.1
"I restructured the entire API"           → MAJOR: 1.3.1 → 2.0.0
"Developing features for next version"    → Prerelease: 1.4.0-beta.1
```

---

## 13. Quick Checklist

### Before creating an issue
- [ ] Title follows `type(scope): description` format?
- [ ] Has clear acceptance criteria?
- [ ] Has type, priority and area labels assigned?

### Before creating a branch
- [ ] Is there an issue that justifies this work?
- [ ] Can I describe the purpose in one sentence?
- [ ] Am I starting from updated `main`?
- [ ] Does the name follow the `type/description` convention?

### Before each commit
- [ ] Does the commit compile and work?
- [ ] Does the message follow the `type(scope): description` format?
- [ ] Does it not include unrelated changes?

### Before creating the PR
- [ ] Are all commits clear and atomic?
- [ ] Are there no accidental files (.env, __pycache__, etc)?
- [ ] Does the PR title describe the change?
- [ ] Does the description have Summary, Changes and How to test?
- [ ] Does it include `Closes #N` to automatically close the issue?

### Before merging the PR
- [ ] Are all CI checks green?
- [ ] Is the branch up to date with main?

### When I discover something unrelated
- [ ] Is it blocking? → stash, separate branch, fix, rebase
- [ ] Is it not blocking? → New issue, do it later

### For release
- [ ] Are all features/fixes for this release merged to main?
- [ ] Is everything tested?
- [ ] Did I update CHANGELOG.md (moved [Unreleased] to [X.Y.Z])?
- [ ] Did I update VERSION?
- [ ] Did I create the git tag?

---

## 14. Summary

```
One issue   = one documented task before starting
One branch  = one purpose
One commit  = one complete idea
One PR      = one reviewable delivery with green CI
One release = one stable point with version, tag and changelog

When something unrelated appears: stash → branch → fix → rebase → pop
When in doubt whether it deserves its own branch: it does.
When CI fails: fix before merging, no exceptions.
```
