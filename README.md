# claude-skills

A collection of Claude Code skills for structured development workflows.

## Skills

| Skill | Description |
|-------|-------------|
| [git-workflow](./git-workflow/) | Complete Git & GitHub workflow — branches, commits, PRs, issues, CI checks, versioning and CHANGELOG |

---

## Installation

### Option A — Global install (recommended)

Makes all skills available across every project on your machine.

```bash
git clone https://github.com/VizzWard/claude-skills ~/.claude/skills
```

Done. Claude Code picks them up automatically.

**To update later:**
```bash
cd ~/.claude/skills && git pull
```

### Option B — Per-project install

Place the skills inside your project's `.claude/skills/` directory.

```bash
cd your-project
git clone https://github.com/VizzWard/claude-skills .claude/skills
```

Or as a git submodule if you want to track the version:

```bash
git submodule add https://github.com/VizzWard/claude-skills .claude/skills
```

---

## Installing a single skill

If you only want one skill, download the `.skill` file from the [Releases](../../releases) page and install it via Claude Code:

```
/skills install git-workflow-skill.skill
```

---

## How skills work

Skills are instructions that Claude Code loads automatically when it detects a relevant task. No need to paste file paths or repeat context — the skill activates based on what you're doing.

For example, `git-workflow` activates whenever you mention `commit`, `branch`, `PR`, `issue`, `merge`, `CI`, `release`, or any git-related action.

Learn more: [Claude Code Skills documentation](https://docs.anthropic.com/claude-code/skills)

---

## Adapting the git-workflow skill

The `git-workflow` skill was designed for a specific stack (React + Django REST + PostgreSQL + Redis + Docker) and set of GitHub labels. To adapt it to your project:

1. Open `git-workflow/SKILL.md`
2. Update the stack reference at the top
3. Replace the custom labels (Type, Priority, Area) with your own
4. Adjust `references/GIT_WORKFLOW.md` with any workflow specifics

---

## License

MIT — see [LICENSE](./LICENSE)
