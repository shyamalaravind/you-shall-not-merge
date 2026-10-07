# 🧙‍♂️ you-shall-not-merge

> *"You shall not merge!"* — every good reviewer, and a certain grey wizard on a bridge

PR-style code review for the Claude Code CLI, without leaving Zed's buffers.
Select some lines, write a comment, send the review: Claude fixes the code and
replies under each comment, like a pull request conversation, with nothing
pushed anywhere.

It's an [Agent Skill](https://agentskills.io) plus a small script that four Zed
tasks run. Reviews are delivered to a Claude Code session running in
[herdr](https://herdr.dev).

## How it works

The review lives in `.claude-review.md` at the repo root. The script adds that
path to `.git/info/exclude`, so the file is never committed. Each thread is a
heading that links to the code, the code you selected, then the conversation:

````markdown
### [src/cart.ts:16-18](src/cart.ts:16)

<!-- prettier-ignore -->
```ts
if (coupon == "SAVE20") {
  t = t - t * 0.2
}
```

**me:** shouldn't this be an else if?

**Claude:** Yes, it's now a flat `else if` at src/cart.ts:16-18, and the totals are unchanged. ✅ resolved

**me:** 
````

You never type the markup. Every thread ends in an empty `**me:**` reply slot,
and the keys put your cursor in it:

| Key (suggested)   | Task                                                                  |
| ----------------- | --------------------------------------------------------------------- |
| `cmd-alt-g /`     | **Comment or reply.** On a line that already has a thread, or inside the review file, jump to that thread's reply slot; anywhere else, start a thread on the selected lines (or the cursor line) |
| `cmd-alt-g n`     | **Next for me.** Jump to the next thread where Claude replied and it's your turn |
| `cmd-alt-g x`     | **Resolve.** Delete the thread under the cursor                       |
| `cmd-alt-g enter` | **Send** the open threads to the repo's Claude                        |

- Starting a thread works in any buffer, including Zed's `git: diff branch`
  view, since Zed hands tasks the real file and line even from a diff.
- Cmd-click a thread's heading to jump to the code. `cmd-k v` opens Zed's
  Markdown preview beside the file, where threads read like comment cards.
- Sending submits `/you-shall-not-merge <file>` to a Claude in herdr for the
  repo: one started at its root, else one inside it, else one in the folder
  holding it (a Claude working across sibling repos). A pane showing a chat
  beats one on Claude's agents view, and the most recently active wins a tie. A
  notification names the herdr workspace that got it.
- Claude fixes the code, replies under each open thread, ends finished ones with
  `✅ resolved`, leaves a fresh reply slot, and keeps each heading pointing at
  the code as it moves. A question back to you stays open. Resolved threads are
  cleared on the next round.

A thread is open while its last entry is a `**me:**` comment with text in it.
The `<!-- prettier-ignore -->` line keeps format-on-save from reflowing the
quoted code, which Claude uses to find lines that moved. Don't edit the file
while Claude is replying: Zed reloads the buffer only while it has no unsaved
changes.

### Why not Zed's own diff review comments

Release builds of Zed hide them behind a staff-only feature flag, their Send
Review to Agent button has no handler, and they would reach only Zed's Agent
Panel, not a CLI session. Zed extensions can't draw in the editor, so a buffer
is the place a review can live.

## Requirements

- [Zed](https://zed.dev), for the tasks
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
    "label": "Review: next thread for me",
    "command": "~/.claude/skills/you-shall-not-merge/review next",
    "allow_concurrent_runs": false,
    "reveal": "never",
    "hide": "on_success",
    "save": "all"
  },
  {
    "label": "Review: resolve thread",
    "command": "~/.claude/skills/you-shall-not-merge/review resolve",
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
      "cmd-alt-g n": ["task::Spawn", { "task_name": "Review: next thread for me" }],
      "cmd-alt-g x": ["task::Spawn", { "task_name": "Review: resolve thread" }],
      "cmd-alt-g enter": ["task::Spawn", { "task_name": "Review: send to Claude" }]
    }
  }
]
```

## Layout

- `skill/you-shall-not-merge/SKILL.md`: what Claude does with the threads.
- `skill/you-shall-not-merge/review`: `add`, `next`, `resolve` and `send`, run
  by the Zed tasks.

## License

[MIT](LICENSE)
