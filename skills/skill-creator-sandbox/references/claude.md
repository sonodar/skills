# Environment isolation in Claude Code

Verified on macOS with Claude Code 2.1.311.

## Use a fresh restricted session

Exclude author-specific instructions, skills, settings and memory so they cannot fill gaps in the target skill.
Keep product-standard instructions, bundled skills and plugins, and required administrator policies.
For the tested read-only evaluation, use a fresh workspace outside the author's project and a new main-thread CLI session with these controls:

| Control | Purpose |
| --- | --- |
| `--restricted` | Excludes user/project/local settings and confines file tools to working directories. |
| `--setting-sources ""` | Explicitly excludes those settings sources. |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1` | Excludes auto memory separately. |
| `--strict-mcp-config` | Excludes MCP configurations not explicitly supplied. |
| `--tools Read,Skill` | Makes the file-reading and skill tools available, without command execution tools. |
| `--permission-mode dontAsk` | Refuses operations that require approval instead of expanding access during the test. |
| `--no-session-persistence` | Avoids retaining the test conversation; it does not itself isolate instructions. |

Copy the complete target skill and required inputs into the workspace, then explicitly ask Claude to read the copied SKILL.md and execute it.
Do not resume, continue or fork the authoring conversation, and do not add the author's home or project with `--add-dir`.
This evaluates the skill's instructions, not automatic selection from its description.

Resolve the installed CLI executable directly and pass an explicit environment allowlist.
Do not inherit wrapper variables, session context, provider overrides or shell configuration wholesale.
The authenticated test retained `HOME`, `USER`, `LOGNAME`, and `SECURITYSESSIONID` for the existing macOS login, plus a system-only `PATH`, `TMPDIR`, `DISABLE_AUTOUPDATER=1`, and the memory control above.
Keeping only `HOME` and the runtime variables lost authentication; retaining the three OS identity/session variables together restored it.
Their individual necessity was not tested, and other operating systems may need different authentication context.
Never copy another session's identifier or print credentials.

The tested authenticated procedure keeps the normal home for login state and excludes its author-specific instruction sources through the controls above.
An empty home also passed synthetic input checks, but could not use the existing login in this test.
Do not copy credentials or all of `~/.claude` to make that variant work.
`~/.claude.json` and required managed settings remain available to the CLI; this procedure does not claim the entire CLI process is unable to access the home directory.

See the official [CLI reference](https://code.claude.com/docs/en/cli-reference), [settings precedence](https://code.claude.com/docs/en/settings), [environment variables](https://code.claude.com/docs/en/env-vars), and [permissions](https://code.claude.com/docs/en/permissions).
Check installed help and relevant changes when the version differs.

## Check all author-specific input routes

Review startup instructions, rules, personal/project skills, settings, hooks, plugins, custom agents, output styles, memory, account-synced skills, MCP connections and inherited environment variables together.
Select a configuration that excludes their sources instead of maintaining a growing list of individual files to disable.
According to the [skills documentation](https://code.claude.com/docs/en/skills), excluding `user` from setting sources also prevents account-skill syncing.
Do not turn syncing off in everyday user settings for the test: that setting can move existing synced skills to trash.

In the tested release, normal startup loaded synthetic `.claude/skills` controls but did not load independently placed `~/.agents/skills` or project `.agents/skills` controls.
References to `.agents/skills` in the installed binary included import-from-other-tool code; their presence alone did not establish startup discovery.
Symlinks from a discovered skill directory can still point into `.agents`.
Recheck discovery on version changes rather than treating `.agents` as permanently irrelevant.

## Verify startup input and runtime access

Before accepting an evaluation, run positive controls with independent, nonsecret random markers in synthetic instruction sources.
Confirm those markers enter a normal session, then check that they are absent from the restricted session's actual input.
Keep the skill tool available during discovery checks: turning all tools off also removed skill descriptions in the test and would give a false sense of exclusion.

The mechanical test captured actual request bodies with a loopback-only mock API, a dummy key and synthetic homes.
The normal request contained markers from global CLAUDE.md, rules, personal skills, project CLAUDE.md, project skills, auto memory and a SessionStart hook.
The restricted request contained none of them, while standard skills and builtin-source plugins remained.
The capture-only endpoint deliberately returned an API error; those runs inspected inputs and were not counted as successful behavioral evaluations.
Never direct real credentials or private user context to a diagnostic endpoint.

For runtime checks, require an allowed read inside the workspace and denied reads outside it, including a symlink from inside to outside.
The mock API returned deterministic Read tool calls to the real CLI; the CLI returned actual `tool_result` messages with the workspace file's contents and errors for an external CLAUDE.md, an external `.agents` skill, and the symlink.
The tool results, rather than a model's summary, established the boundary.

The authenticated main-thread test then used the existing login, read a copied target skill with Read, and returned the exact random response specified only in that file.
It completed with exit code 0 and a successful result event.
Save stdout, stderr, debug logs, actual tool results, the launch configuration and marker assertions together.
Use `--output-format stream-json --verbose` for execution records and `--debug-file` for diagnostics.
If giving the user a long command, put the resolved launch command in a script and provide its short invocation.

`InstructionsLoaded` hooks and init catalogs can help, but neither alone proves all input was excluded.
Some instruction reads do not emit that hook, and a listed plugin need not be active.
Absence of a marker in the final answer, a refusal to reveal it, or the expected target output alone is not proof of isolation.

## Capabilities and limits

The verified procedure covers explicit skill loading with Read and Skill.
If a task needs writes, command execution, MCP, subagents or other integrations, provide only its declared dependencies and verify those capabilities' input and access routes before accepting the evaluation.
In particular, `--restricted` can expose command tools when explicitly requested with `--tools`; file-tool restrictions do not establish a shell sandbox.
Do not interpret a denied required operation as a defect in the target skill.

Actual API-input inspection used synthetic homes and dummy API authentication; the live OAuth run supplied actual Read/result records and startup catalogs, not a captured full OAuth request.
Custom plugins, MCP servers and account-synced skills were not each activated as separate positive controls, and administrator-policy preservation was not exhaustively tested.

Use a subagent only if the same exclusion and access checks pass for it.
No inherited conversation history or `omitClaudeMd: true` alone is sufficient: plugin hooks and other sources can add context separately.
A fresh main session preserves the normal product prompt, whereas a custom subagent prompt can replace it.
Do not substitute `--bare` or `--safe-mode` without checking their different prompt, authentication and customization behavior.
If the required evaluation cannot run with verified exclusion, continue dependency review and report independent behavioral evaluation as incomplete.
