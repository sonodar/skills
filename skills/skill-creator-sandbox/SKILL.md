---
name: skill-creator-sandbox
description: Use alongside skill-creator whenever creating, updating, or evaluating a skill, and first check whether anyone besides the author will use it. When others will use it, review dependencies on instructions or other skills found only on the author's machine to prevent different behavior on users' machines. If the intended audience is undecided, suggest applying this workflow.
---

# Skill Creator Sandbox

Check whether a skill intended for others depends on instructions or other skills found only on the author's machine.
Passing evaluation on the author's machine does not mean it will behave the same way on users' machines.
Local instructions may fill gaps in the target skill, or another skill may do its work, hiding the fact that the target skill cannot work on its own.
To prevent this, state any dependencies that users also need and evaluate using only the target skill and the required inputs.

## First check whether anyone else will use the skill

Load this skill whenever skill-creator is used to create, update, or evaluate a skill.
Do not ask again if the intended audience is already clear from the conversation.
Otherwise, ask “Will anyone besides you use this skill?” and wait for the answer.
Sharing within a company or team counts as use by others.

- If others will use it, apply the dependency review and evaluation environment isolation below.
- If only the author will use it, skip the steps below and continue with the normal skill-creator workflow.
- If undecided, suggest applying the workflow in case others use it later. Proceed with the steps below only if the user chooses to apply it.

A skill used only by its author may legitimately rely on that machine's instructions or other skills.
When applying this workflow, also read the original skill-creator and insert the following steps before creation, after creation, and before evaluation.
Do not modify the existing skill-creator or global settings to use this companion.

## Separate requirements from local instructions before creation

Compare the behavior requested by the user with the local instructions already loaded.
Do not turn design principles, writing preferences, approval procedures, machine-specific paths, or tools into requirements for the target skill just because they exist in the environment.
Keep requirements the user explicitly requested for a particular environment and dependencies the target skill actually needs.
Ask the user only about instructions whose inclusion is unclear.

The local instructions the authoring agent must follow are separate from the instructions included in the target skill.
Isolation here does not authorize ignoring the authoring agent's instructions or permissions.

## Review dependencies after creation

Read the target skill and its bundled files, and check:

- Are any procedures or decisions impossible without local instructions filling the gaps?
- Have local principles the user did not request been presented as general requirements?
- If machine-specific paths, other skills, tools, or implicit settings are required, can the reader understand what is needed and why?
- Do expected evaluation results come from the user's requirements rather than the authoring environment's preferences?

If the reviewer receives the same instructions, do not treat this review alone as proof of independence.

## Choose the evaluation environment

Read only the reference for the agent being used:

- Claude Code: [references/claude.md](references/claude.md)
- Codex: [references/codex.md](references/codex.md)

Each reference states the CLI version on which its procedure is based.
When using a different version, check for changes to the relevant options and settings.

Use a subagent if its isolation can be verified.
Omitting conversation history is not enough: also check whether local instructions, memories, or other skills enter through another path.
If isolation cannot be verified, follow the reference's fresh-session procedure.
If that is also unavailable, continue reviewing in the normal environment and report that independent behavioral evaluation was not performed.

## What to pass to the evaluation agent

Pass only a copy of the target skill, a realistic test request derived from the user's requirements, and the minimum inputs needed to run it.
Do not pass the authoring conversation, the author's conclusions, local principles, this companion skill, or skill-creator.
If the target skill itself creates or evaluates skills, explicitly include only the dependency skills necessary for that purpose.

Preserve the target skill's bundled references, scripts, and assets.
Create a test directory outside the project; do not copy the surrounding settings and instructions wholesale.
For a baseline comparison, use a separate fresh session with the same inputs, tools, model, and permissions, omitting only the target skill.
Do not give the execution agent reference answers or the grader's conclusions. Evaluate the output against the user's requirements after receiving it.

The isolation target is the author's additional instructions, skills, settings and state that other users will not have.
Keep product-standard instructions, skills and plugins, and required administrator constraints as part of the normal evaluation environment.
Do not expand permissions or move or delete everyday configuration files for evaluation.

## Verify the environment before accepting results

Check input sources together: global and project instructions, personal and project skills, configuration, memory, previous sessions, hooks, plugins, custom agents, account-synced resources, MCP connections, CLI wrappers and inherited environment variables.
Exclude their author-specific sources through the agent's verified launch configuration; do not build a growing list of individual local files to suppress.

Use synthetic, nonsecret random markers and first confirm that the normal configuration loads them.
Then inspect the isolated configuration's actual input or loading records and check that those markers are absent.
Keep the relevant discovery mechanism enabled in the positive control; hiding the skills catalog is not proof that local skills were excluded.
Record sources checked only through configuration separately from sources exercised by a working positive control.

Also check later file access: a synthetic file inside the evaluation workspace must be readable, and an unrelated synthetic instruction file outside it must be refused.
Use actual tool results or direct sandbox process results, including a symlink case where supported.
An empty home or read-only permissions alone do not establish this boundary.
Verify the capabilities the target skill needs; a read-only test does not validate later additions of shell execution, writes or integrations.

## Report the results

Briefly state what was tried and what worked, along with the CLI or host product version, launch method, supplied materials, instructions confirmed excluded, and any remaining instructions or uncertainties.
Do not treat the agent's statement that it “did not read” something, its refusal to reveal a marker, or the target's expected output as sufficient evidence of isolation.
Consider launch settings, input inspection, and execution logs together.
State separately whether environment checks passed and whether the target skill's behavioral evaluation completed.
Distinguish behavioral evaluation that explicitly loads the skill from evaluation of automatic selection based on its description.
