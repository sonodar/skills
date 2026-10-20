# sonodar/skills

English | [日本語（原本）](README.ja.md)

A collection of agent skills for Claude Code and Codex.
Use the [skills CLI](https://github.com/vercel-labs/skills) to install the skills you need.

## Available skills

| Skill | Purpose |
| --- | --- |
| [skill-creator-sandbox](skills/skill-creator-sandbox/README.md) | Helps prevent skills used by others from relying on instructions or other skills found only on the author’s machine, so they can work on users’ machines too. |

## Install

With Node.js and npm installed, run:

```sh
npx skills add sonodar/skills
```

Follow the prompts to choose skills, agents, and installation scope.
Choose the global scope to use the skills across projects.

To list available skills without installing them:

```sh
npx skills add sonodar/skills --list
```

## Usage

Open a skill's README from the list above for its installation instructions and usage.
Mention the skill name in your request to the agent.

## Update

To update skills installed through the skills CLI:

```sh
npx skills update
```

## Add or edit a skill

Create a directory at `skills/<skill-name>/` containing a `SKILL.md` file.
Include `name` and `description` in its frontmatter.
Keep any supporting references or scripts in that directory and link them from SKILL.md.

```text
skills/
├── skill-creator-sandbox/
│   ├── SKILL.md
│   ├── SKILL.ja.md
│   ├── README.ja.md
│   ├── README.md
│   └── references/
│       ├── claude.md
│       ├── claude.ja.md
│       ├── codex.md
│       └── codex.ja.md
└── <skill-name>/
    └── SKILL.md
```

Keep each document’s English text in `.md` and its Japanese source in the corresponding `.ja.md` file.
Update both versions together when making changes.
Add new skills to the lists in both the English and Japanese READMEs.
See the [skills CLI authoring guide](https://github.com/vercel-labs/skills#creating-skills) for details on layout and frontmatter.

Run the following from the repository root to check local skill discovery without installing anything:

```sh
npx skills add . --list
```
