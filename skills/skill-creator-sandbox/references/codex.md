# Environment isolation in Codex

Verified on macOS with Codex CLI 0.162.1.

## Use a fresh evaluation environment

Exclude the author's instructions and state so they cannot fill gaps in the target skill.
Keep product-standard instructions, bundled skills and plugins, and required administrator policies.
Do not hide the entire skills catalog merely to make an isolation check pass.

Create a fresh home, an existing empty `CODEX_HOME` beneath it, and a separate evaluation workspace outside the author's project.
Copy only the target skill and its declared inputs and dependencies into the workspace.
Start a new process with an explicit environment allowlist rather than inheriting the author's shell environment.
Set `HOME`, `CODEX_HOME`, a minimal system `PATH`, and `TMPDIR`; add only authentication and runtime variables required for this evaluation.
Do not inherit session identifiers, alternate state locations such as `CODEX_SQLITE_HOME`, provider overrides, or shell initialization variables without checking their purpose.
Resolve and invoke the installed CLI executable directly: a terminal wrapper can inject hooks or extra configuration.

Changing `CODEX_HOME` alone is insufficient: the real `~/.agents/skills` catalog remained visible in the test.
With fresh `HOME` and `CODEX_HOME`, a clean workspace, and the environment allowlist, the tested local instruction and skill sources disappeared from the model input while bundled skills remained.
See the official [skill locations](https://learn.chatgpt.com/docs/build-skills), [environment variables](https://learn.chatgpt.com/docs/config-file/environment-variables), and [configuration layers](https://learn.chatgpt.com/docs/config-file/config-basic).

The live test used API-key authentication through `CODEX_API_KEY` without copying authentication files.
Use credentials already authorized for the evaluation environment; do not print or save their values.
If that environment cannot authenticate, report the blocker rather than copying the everyday Codex home.
ChatGPT-login behavior in a fresh home was not tested.

## Restrict later file reads

An empty home prevents startup discovery but does not stop tools from reading the author's files later.
`--sandbox read-only` allowed an external synthetic AGENTS.md to be read in the test.
Use a filesystem profile that denies access by default and permits the evaluation workspace and required runtime paths.

The following configuration was tested for read-only evaluation, in the evaluation `CODEX_HOME/config.toml`:

```toml
default_permissions = "evaluation"
approval_policy = "never"
allow_login_shell = false

[permissions.evaluation.filesystem]
":root" = "deny"
":tmpdir" = "deny"
":slash_tmp" = "deny"
":minimal" = "read"

[permissions.evaluation.filesystem.":workspace_roots"]
"." = "read"
```

Inspect the effective profile instead of assuming that these keys were accepted.
Required managed policy still applies; do not bypass it.
The CLI also permitted its bundled shell runtime in the tested profile.
See [filesystem permissions](https://learn.chatgpt.com/docs/permissions).

For a task requiring writes, shell dependencies, network access, or integrations, grant only the capabilities the declared task needs and repeat the boundary checks for those capabilities.
Do not label a denied required operation as a skill failure.
The read-only test does not establish isolation for every possible tool configuration.

## Inspect and verify before evaluating

Use the same executable, environment, working directory, and configuration for inspection and execution.
Check installed CLI help before constructing commands.

- Inspect startup input with `codex debug prompt-input` and exact synthetic markers from working positive controls.
- Inspect resolved configuration and skill discovery. The tested app-server `config/read` with `includeLayers=true` and `skills/list` with `forceReload=true` exposed the loaded layers and skills; negotiate the installed app-server protocol before using these methods.
- Confirm that personal/project config, hooks, plugins, MCP definitions, memory and previous session data were not brought into the new roots. System configuration may still apply.
- Test runtime access with synthetic files inside and outside the workspace. On this version, `codex sandbox -P evaluation --include-managed-config -C <workspace> /bin/cat <fixture>` exercised the profile directly, without relying on a model's answer.

The direct sandbox test read an allowed workspace file and refused external synthetic files under both `/private/tmp` and the original home, returning exit code 1 and `Operation not permitted` for each denied read.
A model reported the same denials, but its exec JSON omitted the denied command events; that answer was not accepted as proof.
Use the actual sandbox process result or recorded tool result.

With all checks passed, run a new `codex exec --skip-git-repo-check --ephemeral --json` session from the workspace and ask it to read the copied target skill.
Keep the verified profile; do not replace it with `--sandbox read-only`.
Capture the actual argv, nonsecret environment description, configuration, input inspection, stdout, stderr and tool results.
If giving the user a long launch command, put the resolved command in a script and provide its short invocation.

## Subagents and limits

`fork_turns="none"` removes inherited conversation history but does not establish exclusion of AGENTS.md, memories, skills or developer instructions.
Use a subagent only when its actual inputs and capabilities pass the same checks.
Forking existing history or sending a message with `queue` does not create an isolated evaluation.

The fresh-home run retained system skills and fetched an official plugin catalog.
Those product defaults are not author-specific contamination and need not be removed.
The test observed empty personal and system config layers on this machine; it did not audit every MDM/cloud policy or activate a positive control for each hook, plugin and MCP mechanism.
Record the tested capabilities and any remaining author-specific inputs; do not claim all local filesystem or network routes were exhaustively tested.
If exclusion cannot be verified, continue dependency review and report independent behavioral evaluation as incomplete.
