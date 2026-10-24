# howto-video

A Claude skill that turns a short chat Q&A into a narrated, step-by-step how-to video (MP4) for any web app or tool: Google Analytics, a Shopify admin, your own app, anything.

Claude asks a few multiple-choice questions (topic, voiceover yes/no, voice ID, music, sound effects, aspect ratio, length), writes a script for you to approve, then builds, renders and checks the video.

**Status:** early version, being tested. Feedback welcome via Issues.

## What you need

- Claude with code execution (Claude Code, or claude.ai with code execution enabled)
- `node`, `ffmpeg` and Playwright Chromium (the skill installs Playwright if it can)
- Optional: an ElevenLabs connector or `ELEVENLABS_API_KEY` for voiceover. Without it, the video gets captions only.
- Optional: a browser tool (e.g. Claude in Chrome) to capture screenshots. Without it, you upload the screenshots yourself.

## Install

**Claude Code**

```bash
git clone https://github.com/RekaWeb3Design/howto-video-skill.git ~/.claude/skills/howto-video
```

**claude.ai**

Download this repo as a ZIP (Code → Download ZIP) and upload it as a custom skill in your Claude settings.

## Use

Just ask, e.g.:

> Make a how-to video: how to find the top-selling landing page in Google Analytics.

Claude takes it from there.

## Good to know

- Voice, music and sound-effect generation use your provider's credits. Nothing is generated before you approve the script.
- Real personal data is covered or replaced with fictional data by default.
- YouTube uploads are always **Unlisted**, never Public.

## Magyarul röviden

Claude-skill, ami néhány kattintós kérdés után narrált, lépésről lépésre haladó útmutató videót készít bármilyen webes felülethez. Telepítés és használat: lásd fent. Visszajelzést az Issues fülön kérünk.

## License

MIT © Réka Víg / The WOW Studio
