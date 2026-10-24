---
name: howto-video
description: Guides the user through a short Q&A in chat, then produces a narrated step-by-step how-to / tutorial video (MP4) for any web app or tool, from screenshots, a URL or a rebuilt mock UI, with optional voiceover, captions, music and sound effects.
---

# How-to video maker

Produce short, clear tutorial videos ("how to do X in Y") for clients or teams. The user answers a few multiple-choice questions; you write the script, get it approved, then build, render and check the video.

Always talk to the user in **their** language (the language of their messages). The narration language is asked separately.

## Principles

- **Ask in rounds, with defaults.** Max 4 questions per round, 4 rounds total. Every question has a recommended default so the user can click through fast. Never ask what you can find out yourself.
- **Nothing paid before the script is approved.** Voice, music and SFX generation cost credits; generate them only after the user approves the script. Tell the user up front which steps use credits.
- **Never send audio alone for review.** Always deliver a finished video with sound; the user gives feedback on the video.
- **Privacy first.** No real personal data (names, emails, amounts, order numbers, IDs) may appear in the video unless the user explicitly confirms it is fine. Default: blur/cover it, or rebuild the screen with fictional data.
- **Small steps, explicit gates.** Gates: (1) answers, (2) script, (3) finished video, (4) publishing.

## Step 0 — Capability check (silent, before asking anything)

Check what this session can do and offer only options that will work:

| Capability | How to check | If missing |
|---|---|---|
| Code execution | Bash / code tool available | Stop: explain this skill needs code execution (Claude Code, or claude.ai with code execution on). |
| `node`, `ffmpeg`, Playwright Chromium | `node -v`, `ffmpeg -version`, `npx playwright --version`; install with `npm i playwright && npx playwright install chromium` (and `apt-get install -y ffmpeg` / `brew install ffmpeg` if allowed) | If install fails, report exactly what is missing. |
| Text-to-speech | An ElevenLabs connector tool (search tools for `generate_speech`), or an `ELEVENLABS_API_KEY` env var | Voice option becomes "captions only"; say so once in round 2. |
| Screenshots of a live page | A browser tool (Claude in Chrome, a built-in browser, or Playwright for public pages without login) | Ask the user to upload screenshots instead. |

## Step 1 — Questions

Use a multiple-choice question tool when available (e.g. `AskUserQuestion`); otherwise ask numbered questions in plain text with the default marked. Skip any question already answered in the user's request.

**Round 1 — Content**
1. What should the video teach? (free text: the tool + the task, e.g. "Google Analytics: find which page brings the most sales")
2. Who is it for, and how should it address them? (informal / formal; optional name in the greeting)
3. Narration/caption language? (default: the user's language)
4. What source material do you have? → *link* / *screenshots* / *both* / *I don't know*
   - "I don't know": ask for the URL. If a browser tool is available and the page needs no login (or the user is logged in in that browser), capture the screens yourself. Otherwise give the user a numbered list of exactly which screens to screenshot.

**Round 2 — Voice**
1. Do you want a voiceover? → *yes (recommended)* / *no, captions only*
2. (if yes) Do you have a voice ID? → *yes, I'll paste it* / *no, suggest a voice*. Never assume or hardcode a voice ID. If "suggest", list 3 suitable voices from the TTS provider for the narration language and let them pick.
3. Tone? → *calm and matter-of-fact (recommended)* / *friendly and upbeat* / *formal*
   Captions are always on by default (many people watch muted); the user can turn them off.

**Round 3 — Feel and format**
1. Background music? → *none (recommended for short how-tos)* / *generated soft background* / *my own file*. Never use copyrighted music.
2. Sound effects (click, soft whoosh on transitions)? → *yes* / *no (default)*
3. Aspect ratio? → *16:9 (default, desktop apps)* / *9:16 (social / mobile)* / *1:1*
4. Length? → *short ~45 s, one task* / *standard 60–120 s, full flow (default)*

**Round 4 — Visual mode and output**
1. Visual mode (recommend based on the subject):
   - **Screenshot mode** (default for third-party tools such as Google Analytics or a Shopify admin): real screenshots with zoom, pan, highlight boxes, an animated cursor and captions. Fast; works for anything.
   - **Rebuilt UI mode** (recommended for the user's own app, or when screens contain real data): the screens are recreated in HTML with fictional data, so the video is fully data-safe and fully animatable. Slower and higher quality.
2. Output → *MP4 only* / *MP4 + subtitle file (.srt)* / *MP4 + unlisted YouTube upload* / *+ embed snippet for a website*

After round 4, summarise all choices in 4–6 lines and say which later steps will use credits.

## Step 2 — Source material

- **Screenshots:** one screenshot per step, same browser width (1280–1440 px wide works best for 16:9). Check every screenshot for personal data; list what you will cover and confirm with the user if unsure.
- **Live capture:** navigate step by step, screenshot each state, and record the on-screen coordinates of each element you will highlight. Do not submit forms, purchase anything, or change settings to "get a nicer screen".
- **Rebuilt UI:** read the real screens (or the app code, if available) only to copy layout, labels and wording; fill everything with obviously fictional data (e.g. "Sample Anna", "Test Ltd", round amounts).

## Step 3 — Script (gate)

Write the script as a table: `# | on screen (what is shown/highlighted) | narration (or caption text)`.

- Structure: 1-line intro (what you will learn) → one scene per step → 1-line close (where to get help, or what comes next).
- One action per scene. Use the exact UI labels in quotes, as they appear on screen, in the screen's language.
- Pacing: about 2.2 words/second of narration in English, about 1.8 in Hungarian; check the total against the chosen length.
- Plain spoken language, short sentences, no marketing tone.

Show the script and wait for approval. Apply edits and show only the changed rows.

## Step 4 — Audio

**Voiceover** (after approval):
- ElevenLabs: use the user's voice ID and a current multilingual model (check the provider's model list rather than assuming a model name). Generate **one scene per clip** so timing is exact and a single line can be redone cheaply. Keep the tone instruction consistent across clips.
- Measure each clip: `ffprobe -v error -show_entries format=duration -of csv=p=0 clip.mp3`.
- Scene duration = clip length + 0.4 s pause (+0.8 s on the intro and the close).

**Captions only:** scene duration = max(2.5 s, characters ÷ 15) + 0.5 s.

**Music** (optional): generated or user-supplied only, mixed at about −22 dB under speech, with a 1 s fade in and a 2 s fade out.
**SFX** (optional): short click and whoosh sounds, generated or self-made, placed at the cursor-click and transition times.

Write a `timeline.json`: per scene its start time, duration, audio file, caption text and visual actions.

## Step 5 — Build the scene (`scene.html`)

One self-contained HTML page at the output size (1280×720 for 16:9, 720×1280 for 9:16, 1080×1080 for 1:1). All motion must be a **pure function of time**: expose `window.setT(t)`, which positions everything for second `t`. No CSS animations, transitions or timers, because frames are captured out of real time.

Building blocks (all driven by `t` and `timeline.json`):
- **Title card** (topic and tool name) and **end card** (close line and optional contact).
- **Screen layer:** the screenshot or rebuilt UI, with an eased zoom/pan towards the active element (ease-in-out, max ~1.8× zoom).
- **Highlight:** a rounded outline around the element plus a dimmed backdrop; it fades in 0.3 s after the scene starts.
- **Cursor:** moves along an eased path to the element, with a small click ripple at the SFX time.
- **Step badge:** "Step 2/6".
- **Captions:** a bottom bar (top bar if the highlight sits in the bottom third), at most 2 lines and 42 characters per line.
- **Redactions:** solid or blurred boxes over personal data, applied in screenshot mode before any zoom.

Style: neutral and readable (system font, high contrast, a single accent colour). Use the client's brand colour as the accent if the user gives one.

## Step 6 — Render

`render.js`:
```js
const {chromium} = require('playwright');
const fs = require('fs');
(async () => {
  const [w, h, fps, dur] = [1280, 720, 25, Number(process.argv[2])];
  const b = await chromium.launch();
  const p = await b.newPage({viewport: {width: w, height: h}});
  await p.goto('file://' + __dirname + '/scene.html');
  await p.evaluate(() => document.fonts.ready);
  fs.mkdirSync(__dirname + '/frames', {recursive: true});
  const n = Math.ceil(fps * dur);
  for (let i = 0; i < n; i++) {
    await p.evaluate((t) => window.setT(t), i / fps);
    await p.screenshot({path: `${__dirname}/frames/f${String(i).padStart(5, '0')}.jpg`, type: 'jpeg', quality: 92});
  }
  await b.close();
})();
```
Run it with a long timeout (rendering takes roughly 1–3× the video length).

Audio: concatenate the clips with silence pads into `voice.wav`, following `timeline.json` (ffmpeg `adelay` + `amix`, or the `concat` filter with `anullsrc` gaps). Mix in the music and SFX if chosen.

Encode:
```bash
ffmpeg -y -framerate 25 -i frames/f%05d.jpg -i mix.wav \
  -c:v libx264 -pix_fmt yuv420p -crf 20 -preset veryfast \
  -c:a aac -b:a 160k -shortest -movflags +faststart output.mp4
```
If the user asked for subtitles, also write `output.srt` from `timeline.json`.

## Step 7 — Check (before showing the user)

- Extract 6–9 frames at key moments into a contact sheet (`ffmpeg -ss T -frames:v 1`, then `hstack`/`vstack`) and **look at it**. Check that captions never cover the highlighted element, highlights sit on the right element, no personal data is visible, and the text is readable.
- Check sync: each highlight appears while its line is spoken.
- Check the duration against the chosen length.
Fix problems before delivering. Do not report a check you did not do.

## Step 8 — Deliver (gate)

Send the MP4, and the .srt if requested. In 3–6 lines, say what each part of the video shows, that all data is fictional or covered, and any known small issue. Ask for feedback. On revisions, change only what was asked, then re-render and re-check.

**Unlisted YouTube upload** (only if chosen, and only with a browser tool and the user's go-ahead): upload the file, set the title and description, audience "not made for kids", and visibility **Unlisted**, never Public or Scheduled. Comments off; embedding on if an embed snippet was requested. Stop and ask if YouTube requests new terms, account or channel changes, or verification. Return the link and confirm the visibility.

**Embed snippet** (if requested): use `https://www.youtube-nocookie.com/embed/<ID>?rel=0` and load the iframe only after the viewer clicks (privacy: nothing is sent to YouTube before interaction).

## Rules

- No copyrighted music, characters or footage. Third-party UIs appear only as screenshots of the real product being explained; never make the video look official or endorsed by that company.
- Never put real personal data in the video without explicit confirmation.
- Never upload anywhere as Public.
- Ask before spending credits beyond what the user approved (e.g. regenerating the whole voiceover rather than one line).
