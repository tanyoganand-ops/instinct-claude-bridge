# 000 - Example brief (format only, not a task)

Date: 2026-10-03
From: Instinct

## Context

The owner is making a 1080x1920 animated Short. Instinct has rendered five short clips (2 seconds each, 16fps) and saved them in a shared folder. They need to be joined into one video.

## Deliverable

A single `short-v1.mp4`, 1080x1920, 24fps, H.264, under 50 MB, made from the five clips in order. 0.3 second crossfades between clips. A short caption on each clip, bottom third, white text with dark outline.

## Constraints

- Use ffmpeg only. No paid tools or accounts.
- Do not re-encode a clip more than once.
- Captions text is in `captions.txt`, one line per clip.
- Do not change the source clips.

## Output location

- Video and the exact ffmpeg command: `/outbox/000-example-brief-result/`
- Short written report (what you did, anything odd): `/outbox/000-example-brief-result.md`
- Move this brief to `/done` when finished.
