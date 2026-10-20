# sonodar/skills

English | [日本語（原本）](README.ja.md)

A collection of agent skills for Claude Code and Codex.
Use the [skills CLI](https://github.com/vercel-labs/skills) to install the skills you need.

## Available skills

| Skill | Purpose |
| --- | --- |
| [skill-creator-sandbox](skills/skill-creator-sandbox/SKILL.md) | Helps prevent skills used by others from relying on instructions or other skills found only on the author’s machine, so they can work on users’ machines too. |

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

To install a specific skill, pass its name with `--skill`:

```sh
npx skills add sonodar/skills --skill skill-creator-sandbox
```

## Usage

Mention the skill name in your request to the agent.
Each linked SKILL.md describes its workflow, required tools, and companion skills.

Use skill-creator-sandbox alongside skill-creator when creating, updating, or evaluating a skill.
It first asks whether anyone other than you will use the skill, unless that is already clear from the conversation.
This includes sharing within a company or team.
If others will use it, it applies dependency review and evaluation environment isolation; if only you will use it, it continues with the normal skill-creator workflow.
If you have not decided, it suggests applying these steps in case others use it later and proceeds with them only if you choose to.
On the author's machine, local instructions may fill gaps in the skill, or another skill may help it complete the task, allowing an evaluation to pass.
Users may not have those instructions or skills, so that result does not establish that the skill will work for them.
skill-creator-sandbox adds dependency review and evaluation with only the target skill and its required inputs.
Dependency review and evaluation isolation are skipped for skills intended only for your own use.

It supports Claude Code and Codex.
You can keep using your existing skill-creator installation.
If you do not have one, you can install [Anthropic's skill-creator](https://github.com/anthropics/skills/tree/main/skills/skill-creator):

```sh
npx skills add anthropics/skills --skill skill-creator
```

For example:

> Use skill-creator and skill-creator-sandbox to create a code review skill.

See the [skill-creator-sandbox README](skills/skill-creator-sandbox/README.md) for its background and retirement conditions.

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
