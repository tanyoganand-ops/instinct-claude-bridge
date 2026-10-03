# instinct-claude-bridge

A hand-off channel between two agents working for the same person:

- **Instinct** is the planning, research and account-operations agent. It writes task briefs.
- **Claude** (Claude Code, or Claude with the GitHub connector) is the builder. It reads briefs, does the work and reports back.

The repo is the only channel. Nothing here is private: do not put passwords, tokens, personal identifiers or private documents in any file.

## Folders

| Folder | Who writes | What it holds |
|---|---|---|
| `/inbox` | Instinct | One markdown brief per task, waiting to be done |
| `/outbox` | Claude | Results, reports, links and notes for each finished brief |
| `/done` | Claude | Briefs that have been completed (moved from `/inbox`) |

## Protocol

1. Instinct adds a brief to `/inbox`, named `YYYY-MM-DD-short-slug.md` (for example `2026-10-03-ffmpeg-assembly.md`).
2. Claude reads `/inbox`, newest first, and does the task.
3. Claude writes its result to `/outbox` using the same file name stem (`2026-10-03-ffmpeg-assembly-result.md`). Large files can sit in a subfolder with the same stem.
4. Claude moves the brief from `/inbox` to `/done`.
5. Instinct reads `/outbox`, checks the result, and sends follow-ups as new briefs.

## Brief format

Every brief has four sections:

- **Context**: why the task exists and what Claude needs to know.
- **Deliverable**: the exact thing to produce, in concrete terms.
- **Constraints**: limits on tools, style, cost, time, and what must not be touched.
- **Output location**: the path in `/outbox` (or another repo and branch) where the result goes.

See `inbox/000-example-brief.md` for a filled example.

## Rules

- One task per brief.
- If a brief is unclear or blocked, write that in `/outbox` as a short note and leave the brief in `/inbox`. Do not guess on anything load-bearing.
- Never overwrite another agent's file. Add a new one.
- Keep commit messages short and say which brief they relate to.
