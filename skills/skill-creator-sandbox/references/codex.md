# Environment isolation in Codex

This procedure is based on Codex CLI 0.162.1.

## Conditions for using a subagent

In environments that provide `fork_turns="none"`, a subagent can start without the parent's conversation history.
This does not guarantee exclusion of AGENTS.md, memories, skills, or developer instructions.
The [custom subagent documentation](https://learn.chatgpt.com/docs/agent-configuration/subagents) describes settings that restrict memories, plugins, and skills.
Check whether the launch tool in the current environment accepts those settings.

Use a subagent for independent behavioral evaluation only when the parent's history and unrelated local instructions can be excluded and this can be verified through actual inputs or settings.
If only `fork_turns="none"` is available, treat it as a review without inherited history, not as an isolated environment.

## Inspect a fresh session

Check `codex exec --help`, `codex debug prompt-input --help`, and the [configuration reference](https://learn.chatgpt.com/docs/config-file/config-reference).
`fork` branches from existing history, while `queue` sends a request to an existing session. Neither removes instructions.

Place only the target skill and inputs in an evaluation directory outside the project.
The following settings are candidates for reducing additional context; they do not necessarily provide complete isolation on their own.

| Setting or option | Purpose and limitations |
| --- | --- |
| `project_doc_max_bytes=0` | Stops loading project AGENTS.md files. Do not assume global instructions are also removed. |
| `memories.use_memories=false` | Stops injection of existing memories. |
| `features.memories=false` | Disables the memory feature. |
| `features.plugins=false` | Disables plugins. |
| `skills.include_instructions=false` | Omits the skills instruction block. Does not make individual skills unavailable. |
| `skills.bundled.enabled=false` | Disables bundled skills. |
| `developer_instructions=""` | Clears developer instructions added through configuration. Administrator instructions may remain separately. |
| `--ignore-user-config` | Prevents `codex exec` from reading the user's config.toml. This is separate from excluding AGENTS.md. Authentication still uses the normal CODEX_HOME. |

Inspect inputs with `codex debug prompt-input`, using the same working directory and the same settings where supported.
In local testing, adding `--strict-config` to `debug` produced an error; do not add it without checking support.
Options specific to `exec` prevent an exact match between inspection and execution conditions, so record those differences.
The full input may contain local information. Limit inspection output to the necessary items and do not send the complete input externally.

When this reference was written, the model input still contained the global AGENTS.md text even with the table's `-c` settings applied.
Removal of the skills list and memory summary was not sufficient.
Do not use this combination as a launch command assumed to provide isolation.

## If global instructions remain

Start a fresh session in a separate environment that does not bring in the user's settings, such as another OS user or an already authorized container.
Pass only the target skill and required inputs; do not mount the everyday home directory or project wholesale.
Use authentication approved for that environment and do not copy authentication files without authorization.
If a new execution environment is needed, confirm the required permissions, tools, and costs before proceeding.

Inspect settings and inputs there to verify that unrelated AGENTS.md files, memories, skills, and additional instructions are absent.
Do not remove administrator instructions to achieve isolation. Record their effects if they remain.
Only in an environment where isolation has been verified, write a request that explicitly reads the target skill into `request.txt` and run it in a fresh session.
Pass every `-c` setting used during input inspection to the evaluation command as well. Settings apply per invocation and are not inherited automatically.
The following is a basic command; add the verified settings before using it. If there are many settings, put them in an executable file and check for differences between inspection and execution.

```sh
codex exec --skip-git-repo-check --ephemeral --sandbox read-only --json - < request.txt > events.jsonl
```

For tests requiring writes, grant permissions only within the authorized evaluation directory.
`--ephemeral` prevents history from being saved; it does not isolate instructions or restrict file access.
Also inspect paths read during execution and record any unrelated instructions that were read.

If a separate environment is unavailable, review in the normal environment and report that independent behavioral evaluation was not performed.
If switching to Claude Code for evaluation, do not claim to have evaluated Codex-specific behavior.
