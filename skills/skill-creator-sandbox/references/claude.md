# Environment isolation in Claude Code

This procedure is based on Claude Code 2.1.301.

## Recommended approach

For behavioral evaluation of a skill's instructions, combine `--setting-sources ""` with `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1` in a fresh CLI session.
Exclude user, project, and local settings, and disable auto memory separately.
Also add `--strict-mcp-config` when the test does not need MCP.
Because the target skill is explicitly read as a file, this does not test automatic selection based on its description.

Consult the official [settings-loading documentation](https://code.claude.com/docs/en/agent-sdk/claude-code-features), [CLI reference](https://code.claude.com/docs/en/cli-reference), and local `claude --help` before use.
Even with these settings excluded, standard product instructions, built-in skills, administrator settings, and `~/.claude.json` remain.
Do not treat this as a way to block all local information or as a sandbox that restricts file access.

## Run the target skill in a fresh session

1. Create an evaluation directory outside the project and copy only the complete target skill and required inputs. Do not bring in surrounding CLAUDE.md, AGENTS.md, rules, or configuration files.
2. In `request.txt`, ask the agent to read the copied target skill with Read and execute it. Do not supply the expected answer or authoring conversation.
3. Start a fresh session in the evaluation directory and save stdout, stderr, and debug logs. Do not use `--resume`, `--continue`, or `--fork-session`.

Use the following as a launch example for tests that require only reads.
If asking the user to run it, put the command with resolved paths in an executable file and provide only a short command to launch that file.

```sh
env -u CLAUDE_CODE_SAFE_MODE -u CLAUDE_CODE_DISABLE_CLAUDE_MDS \
  CLAUDE_CODE_DISABLE_AUTO_MEMORY=1 \
  claude -p --setting-sources "" --strict-mcp-config \
    --permission-mode dontAsk --allowedTools Read \
    --output-format stream-json --verbose --no-session-persistence \
    --debug-file ./eval.debug.log \
    < request.txt > events.jsonl 2> stderr.txt
```

The initial `env -u` removes variables inherited from a parent running in safe mode.
Do not use it to remove administrator-enforced settings.
Local experiments also compared runs with session-related environment variables used for nested CLI launches removed.
When launching from a parent, inspect the inherited environment. If nested execution is rejected, do not count the run as successful; start a fresh session from a separate terminal.
Do not display or copy credentials from environment variables or logs.

`--allowedTools Read` grants permission to Read; it does not restrict the available tool list to Read alone.
To further restrict available tools, use `--tools` while preserving the capabilities needed by the test.
For tests requiring writes or external tools, choose launch settings consistent with the authorized scope and dependencies.
Distinguish denied operations from defects in the skill.

If the target skill is outside the evaluation directory, add only its required copy location with `--add-dir`.
Do not add the everyday home directory or the entire project.
Settings added through `--settings` remain active even with `--setting-sources ""`, so include only explicitly selected settings, such as observation hooks.

## Conditions for using a subagent

Normal non-fork subagents do not inherit the parent's conversation history, but they load instruction files and other context through separate paths.
`omitClaudeMd: true` alone does not exclude other skills or plugin hooks.
In local testing, an installed plugin's `SubagentStart` hook supplied additional instructions to a subagent configured with `omitClaudeMd: true` and `tools: ["Read"]`.
The subagent did not report this itself; it was found in the debug log.

Before using a subagent, inspect the parent's settings for hook, skill, and memory loading paths as well.
Then pass only the target skill and minimum inputs, without inheriting history or specifying persistent memory.
Also check that the delegation request does not include principles from the authoring environment.

According to the [official subagent documentation](https://code.claude.com/docs/en/subagents), providing a custom prompt to a custom subagent replaces Claude Code's standard system prompt.
To evaluate behavior under normal Claude Code conditions, use the main thread of a fresh CLI session.
Using `omitClaudeMd` when starting a main session with `--agent` does not have the same effect.

## Verify isolation

Consider launch settings, logs of loaded instructions, actual tool calls, and outputs together.
Synthetic instruction files and a separate skill containing marker strings can help uncover omissions when compared with a normal launch that does load them.
The absence of a marker or the presence of expected output alone does not prove that instructions were excluded.

- When recording loads with the `InstructionsLoaded` hook, some paths, such as direct reads of AGENTS.md, may not emit an event. Cross-check tool history and debug logs.
- `additionalContext` in debug logs helps trace instructions added through hooks.
- The init event's `plugins` list included plugins disabled by safe mode. Do not infer whether a plugin is active from that list alone.
- The init event's `skills` list may differ from the skills visible to the model. To evaluate automatic selection, inspect actual `Skill` calls.

Save raw logs in the evaluation directory and analyze the saved files.
Before sharing logs, check for credentials and unnecessary local information.

## The roles of safe mode and bare mode

With `--safe-mode`, the experiment also excluded synthetic local instructions and executed the explicitly loaded target skill.
However, the user's `language` setting remained, `--agents` was ignored, and the supplied observation hooks did not run.
The official specification retains administrator policy settings but disables managed CLAUDE.md, skills, plugins, and similar resources.
For comparisons that need to preserve administrator-distributed instructions, do not make safe mode the first choice.

`--bare` also differs in authentication and the standard prompt.
Do not treat it as equivalent to a normal session using OAuth.

## Verified scope and limitations

In a small comparison experiment conducted by Claude Code itself, the recommended configuration excluded synthetic CLAUDE.md, AGENTS.md, rules, auto memory, and another skill, and the target skill produced the expected output.
However, the comparison also launched subagents, and the process was terminated when a wait operation timed out after results had been produced.
Successful completion of the main-thread-only launch example above has not been verified; check it on the next run.

Whether administrator instructions are actually preserved has not been verified on this machine.
Also identify remaining inputs such as built-in skills and `~/.claude.json`.
This experiment did not evaluate automatic selection based on descriptions, skills that write files, or instructions read later from subdirectories.
To evaluate automatic selection, use a separate configuration that includes the target skill in discovery, and distinguish that from behavioral evaluation of its instructions.
