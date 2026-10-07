---
name: you-shall-not-merge
description: Act on the PR-style review threads the user left for you in .claude-review.md at the repo root (usually written in Zed), reply under each thread, and mark the finished ones resolved. Use whenever /you-shall-not-merge arrives (the editor's send task submits it through herdr, with the review file's path), or the user says "address my review comments", "I left review threads", "check the review file", "read my review", or "reply to my comments".
argument-hint: "[path to .claude-review.md]"
---

# You shall not merge

The user reviews code in Zed and leaves PR-style threads for you in
`.claude-review.md` at the root of a repo. The send task passes that file's
path as the argument; without one, use `.claude-review.md` in
`git rev-parse --show-toplevel`. Paths in the file are relative to the repo
holding it, which need not be your working directory.

The file is kept out of git through `.git/info/exclude`; never commit it, and if
`git status` ever lists it, add `/.claude-review.md` to that exclude file.

## The file

Each thread is a heading that links to the code, the code the user selected in
a fence, then the conversation, one entry per paragraph:

````markdown
### [src/cart.ts:16-18](src/cart.ts:16)

<!-- prettier-ignore -->
```ts
<the selected code>
```

**me:** the user's comment, possibly several lines

**Claude:** your reply

**me:** 
````

- The heading's text is `path:lines`; its link is `path:first-line`, which the
  user cmd-clicks to jump to the code. Older threads may use a plain
  `### path:lines` heading; treat it the same.
- `<!-- prettier-ignore -->` keeps the user's format-on-save from reflowing the
  quoted code. Leave it in place.
- An empty `**me:**` at the end of a thread is the user's **reply slot**: the
  place their next comment goes. Text written on that line or the lines after
  it is the user's reply.

A thread is **open** when its last entry is a `**me:**` comment with text in it.
It is **resolved** when your last reply ends with `✅ resolved`. An empty reply
slot after your reply changes neither.

`path:lines` is where the code was when the user commented. If it has moved,
find it by the quoted code.

## Steps

1. **Read the file.** If it is missing or has no open threads, say so in one
   line and stop.
2. **Clear the previous round.** Delete threads that are already resolved (your
   last reply ends in `✅ resolved` and the reply slot after it is still empty)
   and threads whose only `**me:**` entry is empty. The user had a round to
   reopen them and didn't.
3. **Work each open thread.** Read enough of the surrounding code to understand
   it, then do what the comment asks: change the code, or answer the question.
   The comment is about the quoted lines but may need changes elsewhere; make
   them. If you disagree, or the comment is ambiguous, change nothing and say
   why or ask.
4. **Reply under the thread**, after a blank line, as `**Claude:** ` plus one or
   two sentences: what you changed (with the new line numbers) or your answer.
   End it with ` ✅ resolved` when the thread needs nothing more from the user.
   A question back to the user stays unresolved, so it is still waiting for
   them. Then leave a new reply slot: a blank line and `**me:** ` (with the
   trailing space), so the user can answer without typing any markup.
5. **Keep the headings true.** If the code under a thread moved, update the
   line range in its heading text and the line in its link, so it still points
   at the code. Leave the path, the quoted code and the user's words exactly as
   they are.
6. **Edit the file in place** with Edit, never by rewriting all of it: the user
   has it open in Zed, which reloads an unmodified buffer but would conflict
   with a whole-file rewrite.
7. **Check your changes** the way you normally would (build, tests or lint for
   what you touched). Don't commit or push unless the user asked.
8. **Report in the terminal**, one line per thread: `path:lines — what you did`,
   or `path:lines — question back to you`.
