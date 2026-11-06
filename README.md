# skills-science-career

Claude skills for career tasks in science, such as working from job postings.

## Skills

| Skill | What it does | Version |
|-------|--------------|---------|
| [`career-job-requirements-html`](career-job-requirements-html/SKILL.md) | Turn a job posting (URL, pasted text, or file) into an interactive HTML page of what it requires of the candidate, in its own wording, with a notes box under every point | 1.00 |

New career skills are added as sibling folders, each with its own `SKILL.md`.

## Layout

Each skill is a folder named after the skill:

```
<skill-name>/
├── SKILL.md      instructions and frontmatter
└── assets/       files the skill copies or fills in (here, the HTML template)
```

## Install

With [`npx`](https://docs.npmjs.com/cli/commands/npx) (Node.js):

```bash
npx skills add kenscripts/skills-science-career                     # choose skills and agent interactively
npx skills add kenscripts/skills-science-career --list              # show the skills without installing
npx skills add kenscripts/skills-science-career --skill career-job-requirements-html -a claude-code -y
```

This installs into the current project (`./.claude/skills/`). Tested with Claude Code; the tool also supports other agents.

Or manually, for all your projects:

```bash
git clone https://github.com/kenscripts/skills-science-career.git
mkdir -p ~/.claude/skills
cp -r skills-science-career/career-job-requirements-html ~/.claude/skills/    # or ln -s to follow git pull
```

For one project only, use `<project>/.claude/skills/` instead.

## Requirements

Installing a skill does not install its dependencies.

- `career-job-requirements-html`: none to run. A headless browser such as Playwright is optional, for the skill's self-check.

Skills run with full agent permissions, so read a skill before using it.

## Author

Kenneth Acosta
