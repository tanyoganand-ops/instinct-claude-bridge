# Standing instructions for Claude

This repo is a hand-off channel from Instinct (the owner's planning and research agent) to you. Read README.md once for the full protocol. Short version:

1. Read `/inbox` newest first. Each file is one task brief (Context, Deliverable, Constraints, Output location).
2. Do one brief at a time, exactly as the Deliverable and Constraints say. Ignore `000-example-brief.md`: it is a format example, not a task.
3. Write results and reports to `/outbox` as `<brief-name-without-.md>-result.md`. Put large artifacts in a subfolder with the same stem, or in the repo and branch the brief names.
4. When a brief is finished, move it from `/inbox` to `/done` (same file name).
5. If a brief is ambiguous or blocked, write a short note to `/outbox` saying what is missing and leave the brief in `/inbox`. Do not guess on anything that matters.

Conventions:

- Plain, short markdown. No filler. Straight quotes, no emdashes.
- File names: `YYYY-MM-DD-short-slug.md`, lowercase, hyphens.
- One task per commit; commit messages short and name the brief.
- Never write passwords, tokens, API keys or personal identifiers into this repo. It is public.
- Do not edit or delete other files in `/inbox` unless a brief says to.
- Do not change repo settings or visibility, and do not touch other repositories unless a brief explicitly names them.
