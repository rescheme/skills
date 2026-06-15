# Rescheme Skills

A collection of [Agent Skills](https://agentskills.io/specification) for the open agent ecosystem.

## Skills

| Skill | Description |
|-------|-------------|
| [issue-review](skills/issue-review/SKILL.md) | Analyze and review GitHub issues for bugs or feature requests. Independently verifies code before correlating with the reporter's hypothesis. |
| [pr-review](skills/pr-review/SKILL.md) | Review code changes as a senior engineer. Assesses correctness, security, maintainability, and test coverage. Produces severity-ranked findings with merge readiness. |

## Installation

Install all skills globally using the [Skills CLI](https://skills.sh):

```bash
npx skills add rescheme/skills -g -y
```

Install a specific skill:

```bash
npx skills add rescheme/skills -g -y --skill <skill-name>
```

## Repository Structure

```
skills/
  <skill-name>/
    SKILL.md              # Required: frontmatter + instructions
    scripts/              # Optional: helper scripts
    references/           # Optional: detailed docs
    assets/               # Optional: templates, configs, etc.
```

## Creating a New Skill

```bash
npx skills init <skill-name> --path skills
```

Or manually create `skills/<skill-name>/SKILL.md` with the required frontmatter:

````markdown
---
name: <skill-name>
description: What this skill does and when to use it. Be specific.
---

# <Skill Title>

## Setup

Run once before first use:
```bash
cd /path/to/skill && npm install
```

## Usage

```bash
./scripts/process.sh <input>
```
````

## License

See [LICENSE](./LICENSE).
