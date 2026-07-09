---
name: focus-loop
description: Autonomous Focus OS task-queue engine. Picks ONE todo task from a Focus OS project, executes by lane rules ([AUTO] full execute, [ME] draft-only, no prefix = [ME], [BLOCKED] skip), verifies the task's "Done when:" line with real proof before completing, appends a result note to the task, reports caveman-style with exact file paths. Designed to run under /loop for continuous queue drain — /loop /focus-loop <project>. Use when Igor says "run the loop", "drain the queue", "/focus-loop", or wants autonomous work off a Focus OS task queue.
---

# focus-loop — Focus OS queue engine

One iteration = ONE task. Small iterations -> fresh context, no rot. Never batch tasks.

## 0. Resolve project

- Argument given -> that Focus OS project (resolve name via `mcp__focusos__list_projects`).
- No argument -> read current project's CLAUDE.md `Focus OS:` link.
- No link or `Focus OS: none` -> stop, report "no queue linked".

## 1. Pick

- `mcp__focusos__list_tasks` for the project, status `todo` only.
- Skip titles starting `[BLOCKED]` (waiting on Igor).
- Order: Urgent priority first, then High, then oldest created.
- No actionable `[AUTO]` task left -> final report (what's `[ME]`/`[BLOCKED]` awaiting Igor) and, if running under /loop, STOP the loop. Never idle-spin.

## 2. Lanes — safety rules, never violate

- Title starts `[AUTO]` -> execute fully.
- Title starts `[ME]` -> prepare/draft everything, but NEVER the final irreversible or outward step (send, publish, push, deploy, delete, pay, Lovable send_message). Leave status `todo`, append `DRAFT READY: <where>` to description.
- **No prefix -> treat as `[ME]`. Default-deny.**
- Even inside `[AUTO]`: if a step turns out irreversible or outward-facing -> stop before it, append `BLOCKED: <reason>` to description, leave `todo`, move on.
- Unsure at any point -> same BLOCKED treatment. Never guess.
- Standing hard rules regardless of lane: no email sends, no production pushes, no data deletion, no CLAUDE.md edits (those are diff-approved by Igor -> always `[ME]`), no Lovable send_message unless the task explicitly says so AND is `[AUTO]`.

## 3. Claim

- `update_task` -> status `in-progress`, then `start_task_timer`.
- Claim = lock. A task already `in-progress` belongs to another session -> skip it.

## 4. Do

- Description must contain a `Done when:` line. Missing -> BLOCKED with reason `no Done when line`, pick next task.
- Work inside that task's project folder. That project's CLAUDE.md is authoritative.

## 5. Verify — proof or no completion

- Test the `Done when:` condition with real evidence: command output, file content/mtime, browser check (preview tools) for anything UI.
- Never claim done without proof. Verify fails -> fix and re-verify, or BLOCKED with reason.

## 6. Close

- Append result note to description: `DONE <date>: <what>. Files: <exact paths>. Proof: <evidence>.`
- `complete_task`, then `stop_task_timer`.

## 7. Report — every iteration, caveman style

`[task title] -> done/blocked/draft-ready. Files: <exact paths>. Proof: <evidence>. Queue: N todo left.`

File paths always exact and clickable. Never vague ("the file", "the folder" = banned).

## Reference guide (for Igor)

Visual guide (loop cycle, lanes, pipeline, test plan, task anatomy as diagrams): `index.html` in this folder.
Live page: https://rogerceaser21.github.io/focus-loop-guide/
To update: edit `index.html`, then from this folder run `git add -A && git commit -m "update guide" && git push`. Pages rebuilds in ~1 min.
