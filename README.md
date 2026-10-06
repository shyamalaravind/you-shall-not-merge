# 🧙‍♂️ you-shall-not-merge

> *"You shall not merge!"* — every good reviewer, and a certain grey wizard on a bridge

PR-style code review for the Claude Code CLI, without leaving Zed's buffers.
Select some lines, write a comment, send the review: Claude fixes the code and
replies under each comment, like a pull request conversation, with nothing
pushed anywhere.

It's an [Agent Skill](https://agentskills.io) plus a small script that two Zed
tasks run. Reviews are delivered to a Claude Code session running in
[herdr](https://herdr.dev).

## How it works

The review lives in `.claude-review.md` at the repo root. The script adds that
path to `.git/info/exclude`, so the file is never committed. Each thread is a
`### path:lines` heading, the code you selected, then the conversation:

````markdown
### src/cart.ts:16-18

```ts
if (coupon == "SAVE20") {
  t = t - t * 0.2
}
```

**me:** shouldn't this be an else if?

**Claude:** Yes, it's now a flat `else if` at src/cart.ts:16-18, and the totals are unchanged. ✅ resolved
````

- **Start a thread** (`review add`) appends a thread for the selected lines, or
  the cursor line, and opens the file at the comment. It works in any buffer,
  including Zed's `git: diff branch` view, since Zed hands tasks the real file
  and line even from a diff.
- **Send** (`review send`) submits `/you-shall-not-merge <file>` to a Claude in
  herdr for the repo: one started at its root, else one inside it, else one in
  the folder holding it (a Claude working across sibling repos). A pane showing
  a chat beats one on Claude's agents view, and the most recently active wins a
  tie. A notification names the herdr workspace that got it.
- **Claude** fixes the code, replies under each open thread and ends finished
  ones with `✅ resolved`. A question back to you stays open. Reply under a
  thread with a new `**me:**` line to continue it; resolved threads are cleared
  on the next round.

A thread is open while its last entry is a `**me:**` comment. Don't edit the
file while Claude is replying: Zed reloads the buffer only while it has no
unsaved changes.

### Why not Zed's own diff review comments

Release builds of Zed hide them behind a staff-only feature flag, their Send
Review to Agent button has no handler, and they would reach only Zed's Agent
Panel, not a CLI session. Zed extensions can't draw in the editor, so a buffer
is the place a review can live.

## Requirements

- [Zed](https://zed.dev), for the two tasks
- [Claude Code](https://claude.com/claude-code) running in [herdr](https://herdr.dev)
- `git` 2.31 or later, `jq` and `bash`
- macOS notifications through `terminal-notifier` when it's installed, else
  AppleScript; elsewhere they're skipped and messages go to the task's output

Without herdr, starting threads still works: run `/you-shall-not-merge` in your
Claude session yourself.

## Install

Clone the repo and link the skill into Claude Code:

```sh
git clone https://github.com/shyamalaravind/you-shall-not-merge.git
ln -s "$PWD/you-shall-not-merge/skill/you-shall-not-merge" ~/.claude/skills/you-shall-not-merge
```

Add the tasks to `~/.config/zed/tasks.json`:

```json
[
  {
    "label": "Review: comment for Claude",
    "command": "~/.claude/skills/you-shall-not-merge/review add",
    "allow_concurrent_runs": false,
    "reveal": "never",
    "hide": "on_success",
    "save": "all"
  },
  {
    "label": "Review: send to Claude",
    "command": "~/.claude/skills/you-shall-not-merge/review send",
    "allow_concurrent_runs": false,
    "reveal": "never",
    "hide": "on_success",
    "save": "all"
  }
]
```

Keep the subcommand inside `command`. With `args` present, Zed shell-quotes
the command, a quoted `~` never expands, and the task fails without a trace.

Then bind them in `~/.config/zed/keymap.json`, for example:

```json
[
  {
    "context": "Workspace",
    "bindings": {
      "cmd-alt-g /": ["task::Spawn", { "task_name": "Review: comment for Claude" }],
      "cmd-alt-g enter": ["task::Spawn", { "task_name": "Review: send to Claude" }]
    }
  }
]
```

## Layout

- `skill/you-shall-not-merge/SKILL.md`: what Claude does with the threads.
- `skill/you-shall-not-merge/review`: `add` and `send`, run by the Zed tasks.

## License

[MIT](LICENSE)
