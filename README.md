# code-video-3d

A Claude Code skill for making animated explainer videos and vertical Shorts entirely from code:
low-poly 3D scenes (three.js) plus 2D motion graphics, reviewed in a live browser editor before anything
is rendered, then captured frame by frame into an MP4.

## What it does
1. **Setup**: copies `template/` into a project folder and serves a live editor in the browser
2. **Script**: one spoken sentence = one scene, with a shot plan
3. **Voice**: times scenes to your recorded voiceover (or estimates from word count)
4. **Style**: the look written as numbers (`references/style-rules.md`)
5. **Fix**: you review in the live editor, paste notes back, repeated fixes go into `references/lessons.md`
6. **Render**: chunked, frame-exact render with Playwright Chromium + ffmpeg (`template/render.py`)
7. **Upload**: titles, description, chapters, thumbnail prompt

Every frame is a pure function of time, so the live preview and the render always match.

## Install
Copy this folder to `~/.claude/skills/code-video-3d/`. Claude Code picks it up automatically; trigger it with
`/code-video-3d` or by asking for a "3D explainer video" or a "Short".

## Requirements
- Python venv with `playwright` (Chromium), `imageio-ffmpeg`, `pillow`, and `faster-whisper` for voice timing.
  `SKILL.md` expects it at `~/.youtube-video-editor/venv`; change those paths if yours differs.
- Setup and transcription scripts are referenced from a companion skill (`youtube-video-editor`:
  `scripts/setup.sh`, `scripts/transcribe.py`). Without it, install the packages above yourself and
  run `python -m playwright install chromium`.

## Contents
- `SKILL.md`: the workflow Claude follows
- `template/`: editor page, scene engine, 3D kit and story worlds, renderer
- `examples/`: a full 10-minute build and three 6-second story-look samples
- `references/`: house style in numbers, and lessons learned from real reviews

Timing notes in `SKILL.md` (about 1 s per 3D frame) were measured on a dual-core i3 Mac; faster machines render faster.

`template/lib/three.min.js` is three.js (MIT licence, header kept in the file).
