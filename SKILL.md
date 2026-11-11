---
name: code-video-3d
description: Make animated explainer videos and vertical Shorts entirely from code, with low-poly 3D scenes (three.js) and motion graphics, reviewed in a live browser editor before any render. Use this whenever the user wants a "3D short", "3D video", "animated video with code", "faceless explainer", "video made by Claude Code", a Short/Reel from a script or voiceover, or says things like "make a video about X" without footage to edit. Covers the full workflow - script, shot plan, voice timing, style rules, live preview, review notes, chunked render, YouTube packaging. For editing an existing recording (talking head or screen share) use youtube-video-editor instead.
---

# Code video with 3D (videos and Shorts)

Every frame is a pure function of time, written as HTML/CSS/three.js. The same page is the **live editor**
(plays instantly, nothing saved) and the **render source** (Chromium captures it frame by frame, ffmpeg joins).
The user is the director; you do the work. Nothing is rendered until they have watched the live version.

The workflow has seven steps. Do them in order and do not skip step 5.

| # | Step | What you produce |
|---|------|------------------|
| 1 | Setup | project folder copied from `template/`, server running |
| 2 | Script | `script.md`, then a shot plan: one sentence = one shot |
| 3 | Voice | `voiceover.mp3` + word timings, or estimated timings if the voice comes later |
| 4 | Style | the look written as numbers in `STYLE.md` and built into the 3D world |
| 5 | Fix | live editor review, notes applied, repeated mistakes added to `references/lessons.md` |
| 6 | Render | chunked render, joined with audio |
| 7 | Upload | title options, description, chapters, thumbnail prompt |

## 1. Setup
- Tools live in `~/.youtube-video-editor/venv` (Playwright Chromium, imageio-ffmpeg, pillow, faster-whisper).
  If it is missing, run `~/.claude/skills/youtube-video-editor/scripts/setup.sh`. Always call it as
  `~/.youtube-video-editor/venv/bin/python -I`. There is no system ffmpeg or node on the user's Mac.
- Copy `template/` to `<project>/` inside a folder the user can see (they usually work in `~/Downloads`).
  16:9 video → `index.html` + `scenes.js`. Vertical Short → `short.html` + `scenes-short.js` (stage 1080×1920).
- Serve it: `python3 -m http.server 8766 --bind 127.0.0.1` from the project folder (run it with `nohup … &`),
  then open `http://localhost:8766/index.html` in the Browser pane. A `file://` page shows as a dead snapshot there,
  and `preview_start` with a launch.json did not bring the server up on this Mac, so start it from Bash.

## 2. Script
- The script decides the video. Write it in the user's voice: short spoken sentences, plain words, no textbook tone.
  Their audience is non-technical beginners (Low Code Studio). ~150 words per minute; a Short is 60–110 words.
- Then plan shots before animating: one sentence = one scene, and for each one a line saying what the camera sees.
  Changing a plan is free, changing an animation is not.
- Inspired by another video? Keep the structure and the lesson, change every example, visual and line. Never
  reuse their footage, wording, thumbnails or characters.

## 3. Voice
- The user records their own voice (it is part of the brand) and adds a headshot later. So the first pass has no
  audio: each scene's length is estimated from its `vo` word count (`WPS`/`PAD` in engine.js).
- When `voiceover.mp3` exists: convert to 16 kHz mono wav, run
  `~/.claude/skills/youtube-video-editor/scripts/transcribe.py` for word times, set each scene's `dur` to the gap
  between its first word and the next scene's first word, and nudge `data-in` values onto the key words.
  In-scene timings stretch with the scene length automatically (authored pace → real pace), so most scenes need
  only `dur`. Pass `--audio voiceover.mp3` at render.
- On this i3 Mac whisper `small` can crawl when the machine is busy. `base`, `beam_size=1`, no word timestamps is
  about 5× faster and fine for reading a reference; use `small` + word timestamps only for final sync.

## 4. Style: numbers, not adjectives
"Cinematic" means nothing; every scene would guess differently. Write `STYLE.md` in the project with rules you can
check, then build them into the world once. Start from `references/style-rules.md` (the house 3D look: camera
angles, lens, lighting values, palette, motion, captions). If the user gives reference images, read them and turn
what you see into those same numbers. Their own references, not a copy of another creator's video.

2D motion graphics cost far fewer tokens and break less than 3D. Use 3D for shots where space, scale or a journey
matters (the hook, the "world" shots) and 2D cards for prompts, lists, comparisons and chapter cards. A good mix
is roughly one 3D shot in every three or four scenes. A Short can be 3D throughout because it is only a few shots.

## 5. Fix: the live editor (never skip)
- Open the editor for the user and stop there. They asked for this explicitly: review in the live editor **before**
  any full render. The side panel shows each scene's voiceover, lets them change its length (±0.5 s) and write a
  note; **Copy my review notes** puts everything on the clipboard for them to paste back to you.
- Before handing over, check your own work: `render.py --sheet sheet.jpg` gives one frame per scene. Look at it
  (split a tall sheet into 3 crops so it stays readable), fix what is wrong, repeat. Typical first-pass faults are
  in `references/lessons.md`. Read that file before building.
- When the same note shows up in more than one scene, fix the video **and** add a line to
  `references/lessons.md`, so the next video starts with it solved. This is the whole point of the skill.

## 6. Render
- Only after the user approves the live version. `render.py --chunks out/chunks --chunk-len 60` then
  `render.py --join out/chunks final.mp4 [--audio voiceover.mp3]`. For Shorts add `--size 1080x1920` and the
  `short.html?render` URL. One wrong scene later → delete that `chunk_NNN.mp4` and rerun; only it is re-rendered.
- Speed on the i3 (measured): about 1 s per frame for a full-screen 3D scene at 1080p, because headless WebGL
  runs in software. A 10 minute video at 30 fps is 18,000 frames, so plan for 3–5 hours and tell the user that
  before starting. Use `--fps 24` if the user is fine with it, run with `nohup` in the background, keep the Mac awake (`request_keep_awake`), and
  render at most two chunk processes at once (split the range with `--start/--end` into different folders).
- Headless Chromium needs `--use-angle=swiftshader --enable-unsafe-swiftshader` for WebGL (already in render.py).
- Re-cut without rebuilding: scenes are an array, so reordering or dropping scenes is an edit list. If the opening
  is slow, move the strongest shot to index 0.

## 7. Upload
You know the script, so write 5 title options (recommend one), a description, chapters from the scene start times,
and tags. Follow `~/.claude/skills/youtube-video-editor/references/youtube-metadata.md`. The thumbnail is the one
place an image model helps: render a strong still (`render.py --stills`), write the prompt, and let the user
generate or composite it. Every claim on the thumbnail must be true to the video.

## Scene format (engine.js)
```js
SCENES.push({
  id: 'belt', chapter: '2 · Script', title: 'Box on the belt',   // shown in the editor
  vo: `Your box rides a belt.`,                                   // spoken line; sets the length until audio exists
  dur: 4,                                                         // optional fixed length in seconds
  cls: 'z',                                                       // optional: plain | grad | paper | z | z1 (scale-up)
  html: `<div class="three abs" style="inset:0"></div>
         <div class="h1" data-in="0.4" data-a="pop">Hello</div>`,
  tick: (t, root) => K.draw3d(root.querySelector('.three'), 1920, 1080, t, 'belt'),
});
```
- Declarative: `data-in` / `data-out` (seconds), `data-a` (`up down left right pop fade drop zoom`), `data-d`,
  `data-type` (typewriter), `data-count="0,2400000"`, `data-w="0,80"` (width %), `data-draw` (SVG stroke).
- `tick(t)` is for anything custom. It must depend only on `t`: no `Date.now()`, no `Math.random()` (use
  `E.rnd(i)`), no CSS animations or transitions. Otherwise the render will not match the preview.
- Kit: `K.chat()` Claude-style prompt box, `K.track()` step tracker, `K.chapter()` full-bleed chapter card,
  `K.wave()` + `K.waveTick()`, `K.draw3d(holder, w, h, t, mode)` for one 3D view, `K.snap(canvas, t, mode)` to
  show several camera views in one scene (one shared WebGL renderer; never create a renderer per scene).
- The 3D world is `K.world()` in kit.js: built once, `update(t, mode)` moves objects and places the camera.
  For a new film, replace the objects and camera modes there; keep the pattern. `examples/workflow-video/` is a
  complete 10 minute video (45 scenes) made with this skill. Read it for patterns before writing new scenes.

## Two looks: toy diorama and story worlds
- `kit.js` `K.world()` is the simple flat-shaded diorama (fast to build, good for explainers and backgrounds).
- `template/worlds.js` is the richer **story look** the user preferred after calling the diorama "not very
  realistic": smooth toy-like characters (`person()` with animatable head, arms, legs, plus `walk()`), a gradient
  sky dome, ACES filmic tone mapping, one low warm sun plus warm practical lights, additive glow sprites for
  lamps, steam and beams, canvas-texture windows, fog for depth. Three finished 6 second worlds are in it (chai
  stall at dawn, night delivery rider, lighthouse keeper); `examples/story-samples/` shows how to put them on a
  page. Load `worlds.js` after `kit.js` and draw with `K.drawStory(holder, w, h, t, name)`.
- Code 3D cannot be photoreal. If the user asks for "ultra realistic", say so plainly and offer the choice:
  AI video (costs credits, and the video can no longer claim "all code") or the story look. Then give 2-3 short
  sample clips to pick from before building the whole film in one look.
- Give every story shot a character doing something with a beginning and an end inside the shot (walks in,
  stops, waves; lifts a lantern), a name tag, and one moving light source. That is what reads as "a story".

## Shorts checklist
1080×1920, 20–45 s, one idea. Hook visible and spoken inside 2 seconds, starting on motion. Captions of 2–5 words
on a dark pill (text straight on a bright 3D sky is unreadable), inside the safe area: nothing important in the
top 220 px or bottom 420 px. Use tighter camera modes (`belt`, `follow`, `hero`) because wide shots turn tiny in
vertical. End on the payoff plus one line asking for a follow; cut any outro.

## Bundled files
- `template/`: index.html, short.html, engine.js, kit.js, worlds.js, style.css, render.py, starter scenes, lib/three.min.js
- `examples/story-samples/`: the three character sample clips page
- `examples/workflow-video/`: the full "How I use Claude Code for video" build (scenes-1/2/3.js + index.html);
  copy these next to the template files to run it (thumbnails in its 0:21 wall are not bundled)
- `references/style-rules.md`: the 3D house look in numbers
- `references/lessons.md`: mistakes already paid for (read before building, append after every review)
