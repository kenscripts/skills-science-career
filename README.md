# skills-science-career

![Stars](https://img.shields.io/github/stars/kenscripts/skills-science-career) ![Forks](https://img.shields.io/github/forks/kenscripts/skills-science-career) ![Last commit](https://img.shields.io/github/last-commit/kenscripts/skills-science-career) ![Clones](https://img.shields.io/endpoint?url=https://gist.githubusercontent.com/kenscripts/bc500b179397ff30ba2613d9226d9e75/raw/skills-science-career-clones.json)

Claude skills for career tasks in science.

## Skills

| Skill | What it does | Version |
|-------|--------------|---------|
| [`career-job-requirements-html`](career-job-requirements-html/SKILL.md) | Turn a job posting (URL, pasted text, or file) into an interactive HTML page of what it requires of the candidate, in its own wording, with a notes box under every point | 1.00 |

## Usage

Skill name: `career-job-requirements-html`. Example request:

```text
Run career-job-requirements-html on this job posting: <web link or attached document>
```

## Install

Skills run with full agent permissions, so read a skill before using it.

### With npx (Node.js)

Installs into the current project (`./.claude/skills/`); tested with Claude Code. Omit `--skill` and `-a` to choose interactively.

```bash
npx skills add kenscripts/skills-science-career --list
npx skills add kenscripts/skills-science-career --skill career-job-requirements-html -a claude-code -y
```

### Manually

Clone, then copy into `~/.claude/skills/` for all projects or `<project>/.claude/skills/` for one project. Use `ln -s` instead of `cp -r` to follow `git pull`.

```bash
git clone https://github.com/kenscripts/skills-science-career.git
mkdir -p ~/.claude/skills
cp -r skills-science-career/career-job-requirements-html ~/.claude/skills/
```
