# pstack in Codex

pstack skills use Cursor names. In Codex, read this file once per session, then map each Cursor name in a skill with the tables below. `<pstack>` means the folder that holds `skills/` and `docs/`. If a Cursor field has no row, drop it and say so. Never pass a field or value the tool schema rejects.

## Tools

| Cursor | Codex |
| --- | --- |
| `Task` | `spawn_agent`. Give it a short `task_name` and the brief as `message`. |
| `AskQuestion` | `request_user_input`: one to three questions, each with two or three options. In `codex exec` it is absent, so ask in plain text. |
| Open a todolist | Use `update_plan` if the session offers it. Otherwise keep the list in your reply and update it at each phase. |
| Run another pstack skill (`/name` or "the **name** skill") | Read `<pstack>/skills/<name>/SKILL.md` and follow it. Codex lists pstack skills as `pstack:<name>`, so when you tell the user to type one, write `$pstack:<name>`. |
| `Grep`, `Glob` | `rg` in the shell. |

## Subagent fields

Codex has no named agent types. Every subagent is a `spawn_agent` call, and the brief carries the role.

| Cursor | Codex |
| --- | --- |
| `subagent_type: generalPurpose` | A plain `spawn_agent`. |
| `poteto-agent`, `comment-sicko` | A plain `spawn_agent` whose `message` opens with "Read `<pstack>/agents/<name>.md` and act as that agent for this task.", then the brief. Put the absolute path in, since the agent cannot resolve `<pstack>`. |
| `readonly: true` | Codex has no read-only spawn. The brief says "do not edit files", and you check `git status` afterward. |
| `readonly: false` | A plain `spawn_agent`. |
| `environment: "cloud"` | Tell the worker to run `git worktree add` on the named branch or commit in a temporary folder and work only there. Remove the worktree when you have its result. |
| `environment: "local"` | Omit. |
| `cloud_base_branch: <b>` | Tell the worker to check out `<b>` first. |
| `run_in_background: true` | Omit. `spawn_agent` returns at once and the agent runs in the background. Call `wait_agent` only when the next step needs the result. |
| A fresh subagent | `fork_turns: "none"`, so the agent starts from your brief and file pointers, not your whole thread. Always pass it. The default copies the whole thread. |
| `resume` | `followup_task` with the agent's `target`. |
| `file_attachments` | List the paths in the brief. |

## Models

The models a subagent can take are the ones the `spawn_agent` tool description lists, each with its `reasoning_effort` values. `model` and `reasoning_effort` work only with `fork_turns: "none"`.

| Cursor | Codex |
| --- | --- |
| A `claude-*`, `grok-*`, or other slug the `spawn_agent` description does not list | Omit `model` so the agent runs on the session model, and say so. Keep the slug's effort as `reasoning_effort` when the session model offers it. |
| A slug the `spawn_agent` description lists | `model` set to it. |
| `auto` or `inherit-parent` | Omit `model`. |
| The `pstack-models.mdc` rule, `~/.cursor/rules/pstack-models.mdc` | `~/.codex/pstack-models.md`. Codex does not load it on its own, so read it before your first spawn. If the file or the role's line is missing, use the skill's default slug. |
| A role value `<model> <effort>`, such as `gpt-6.1-sol high` | `model: "gpt-6.1-sol"` and `reasoning_effort: "high"`. |

A panel whose entries all fall back to the session model is no longer diverse. Run it anyway, and say in your reply that its members shared one model.

## setup-pstack

Follow its steps with these changes:

- Detect. The detected models are the ones the `spawn_agent` tool description lists, each with its `reasoning_effort` values. Write a real model as `<model> <effort>`, such as `gpt-6.1-sol xhigh`. The effort is the last token, so the budget step works as written.
- Defaults. Codex lists no Claude or Grok models, so `claude-opus-5-5-xhigh` and `grok-4.7-xhigh-fast` both map to `inherit-parent`. For a panel role, offer two different detected models so the panel stays diverse.
- Write. Write `~/.codex/pstack-models.md` with no frontmatter, then the comment lines, the `# budget` line, and one line per role from step 5. Overwrite the whole file. Never put the role lines in `AGENTS.md`.

## Paths

| Cursor | Codex |
| --- | --- |
| `.cursor/skills/`, `~/.cursor/skills/` | `.agents/skills/`, `~/.agents/skills/` |
| `~/.cursor/plugins/`, `.cursor-plugin/plugin.json` | `~/.codex/plugins/cache/<marketplace>/<plugin>/<version>/`, `.codex-plugin/plugin.json` |
| `.cursor/rules/*.mdc`, always-applied rules | `AGENTS.md`. Codex never reads `.mdc` files. The model rule is the exception. See Models. |
| `https://github.com/cursor/plugins/blob/main/pstack/<path>` | `<pstack>/<path>` when that file exists. The plugin ships the skills, `agents/`, `docs/guide/`, and `README.md`. Read the local copy and never fetch it. Give the user the URL when they need a link. |
| Transcripts in `~/.cursor/projects/<slug>/agent-transcripts/` | `~/.codex/sessions/<yyyy>/<mm>/<dd>/rollout-<timestamp>-<id>.jsonl`. Sessions from every workspace share these folders, so read only files whose `session_meta` line has this workspace as its `cwd`. The newest such file is the current session. |

## Cursor-only features

Codex has no Cursor Automations, Bugbot, Cursor cloud agents, or built-in PR tool. When a step needs one, put one line in your reply, `Skipped in Codex: <step>. <reason>.`, then go on to the next step. Do not stop, ask, or retry. When a later step needs what the skipped step made, such as a URL or a key, leave a placeholder and name it in the line.

| Cursor | Codex |
| --- | --- |
| Cursor Automations: routines, `update_state` routine calls, `api2.cursor.sh/automations` webhooks, routine wakes, and `SendToUser` secret requests | Skip with the line. |
| Bugbot | Bugbot comments on a GitHub PR are ordinary review threads, so triage them with the forge command that the step gives. Skip a step that starts, waits for, or configures Bugbot when the PR has no Bugbot check or comment. |
| Cursor cloud agents | `environment: "cloud"` maps to a worktree, as in Subagent fields. Skip a step that needs the cloud agent itself, such as its dashboard status or its own machine. |
| A built-in PR tool | Use the forge command that the step gives, such as `gh pr create`. That is not a skip, so write no line. |
