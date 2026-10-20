# skill-creator-sandbox

English | [日本語（原本）](README.ja.md)

A companion to skill-creator for creating, updating, and evaluating skills.
It helps prevent a skill from passing evaluation on the author’s machine but failing on users’ machines because it relies on local instructions or other skills.

## Install

With Node.js and npm installed, run:

```sh
npx skills add sonodar/skills --skill skill-creator-sandbox
```

This skill supports Claude Code and Codex and works alongside skill-creator.
You can keep using your existing skill-creator installation.
If you do not have one, you can install [Anthropic's skill-creator](https://github.com/anthropics/skills/tree/main/skills/skill-creator):

```sh
npx skills add anthropics/skills --skill skill-creator
```

## Usage

For example:

> Use skill-creator and skill-creator-sandbox to create a code review skill.

It first asks whether anyone other than you will use the skill, unless that is already clear from the conversation.
This includes sharing within a company or team.
If others will use it, it applies dependency review and evaluation environment isolation; if only you will use it, it continues with the normal skill-creator workflow.
If you have not decided, it suggests applying these steps in case others use it later and proceeds with them only if you choose to.
See [SKILL.md](SKILL.md) for the workflow and the references for [Claude Code](references/claude.md) and [Codex](references/codex.md).

## Background and retirement conditions

On the author's machine, CLAUDE.md, AGENTS.md, memories, or other skills may fill gaps in the target skill or do part of its work.
Users may not have these local resources, so passing evaluation on the author's machine does not establish that the skill will work for them.
This companion reviews dependencies and guides evaluation with only the target skill and its required inputs.
Any dependencies needed by users must be stated explicitly.

The following upstream Issue and PRs track related problems in skill-creator:

- [Issue #1427](https://github.com/anthropics/skills/issues/1427): parallel command clones and existing CLAUDE.md references interfere with trigger detection.
- [PR #1755](https://github.com/anthropics/skills/pull/1755): isolates evaluation probes from the caller's project and competing installed skills.
- [PR #1298](https://github.com/anthropics/skills/pull/1298): isolates trigger evaluations and addresses runtime failures.

This companion is intended to become unnecessary when these three items are closed with the underlying problems resolved and the fixes are available in the skill-creator you use.
Closure without a fix, or an unmerged PR being closed in favor of another PR, is not sufficient.
These items focus on trigger evaluation; before retiring this companion, also verify that the upstream workflow covers local-instruction dependency review during creation and evaluation isolation for your agent.
