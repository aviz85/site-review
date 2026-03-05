# SiteReel — Turn Any Platform Into a Video Review

A [Claude Code](https://claude.ai/code) skill that explores any website or app, documents every finding, writes the narration, and assembles a complete magazine-style video — automatically.

**Live demo:** [aviz85.github.io/site-review](https://aviz85.github.io/site-review/)

---

## What it produces

| File | Description |
|------|-------------|
| `review.md` | Live documentation built during exploration |
| `narration-script.md` | Magazine-style script with direction cues |
| `narration.mp3` | ElevenLabs voice narration |
| `bg-music.mp3` | AI-generated background music |
| `exploration.gif` | Screen recording of the walkthrough |
| `final.mp4` | Complete video: looped GIF + audio mix + burned subtitles |

---

## Usage

```
/site-review <url> [--lang en|he] [--title "Custom Title"] [--gif /path/to/existing.gif] [--output /path/to/dir]
```

### Options

| Option | Default | Description |
|--------|---------|-------------|
| `url` | (required) | Website or app URL to review |
| `--lang` | `en` | Narration language: `en` (English) or `he` (Hebrew, RTL subtitles) |
| `--title` | auto-detect | Review title shown in docs |
| `--gif` | (none) | Reuse an existing GIF — skips the recording step |
| `--output` | `/tmp/site-review-TIMESTAMP` | Output directory |

### Examples

```bash
# Basic English review
/site-review https://linear.app

# Hebrew review with RTL subtitles
/site-review https://yourapp.com --lang he

# Skip recording — use a GIF you already have
/site-review https://yourapp.com --gif /tmp/my-session.gif

# Custom output folder
/site-review https://yourapp.com --output ~/reviews/myapp

# Full control
/site-review https://yourapp.com --lang en --title "Acme CRM Review" --output ~/reviews/acme
```

---

## Pipeline

```
URL → [Browser] → exploration.gif
               ↓
           review.md  (live docs during recording)
               ↓
      narration-script.md  (magazine-style)
               ↓
      narration.mp3  (ElevenLabs voice)
      bg-music.mp3   (ElevenLabs music)
               ↓
      base.mp4 → loop.mp4  (GIF looped to narration length)
               ↓
      video-nosubtitles.mp4  (narration 100% + music 30%)
               ↓
      subtitles.srt  (Groq Whisper, word-level)
               ↓
      final.mp4  (subtitles burned in, RTL auto-detected)
```

---

## Dependencies

SiteReel orchestrates six tools. You need all of them installed and configured:

| Tool | Used for | Skill |
|------|----------|-------|
| **Claude-in-Chrome MCP** | Browser automation + GIF recording | Built-in MCP |
| **speech-generator** | ElevenLabs voice narration | `~/.claude/skills/speech-generator/` |
| **music-generator** | AI background music | `~/.claude/skills/music-generator/` |
| **transcribe** | Groq Whisper SRT generation | `~/.claude/skills/transcribe/` |
| **embed-subtitles** | FFmpeg subtitle burning | `~/.claude/skills/embed-subtitles/` |
| **FFmpeg** | Video assembly (system package) | `brew install ffmpeg` |

### API keys required

```env
ELEVENLABS_API_KEY   # voice + music generation
GROQ_API_KEY         # cheap/fast transcription
```

### Installing skill dependencies

All dependent skills (`speech-generator`, `music-generator`, `transcribe`, `embed-subtitles`) are available in the **Claude Skills Library**:

> **[github.com/aviz85/claude-skills-library](https://github.com/aviz85/claude-skills-library)**

That repo contains a curated collection of Claude Code skills with install instructions, API key setup, and usage guides. Start there if you're setting up from scratch.

---

## Installing this skill

```bash
# Clone into your Claude skills folder
git clone https://github.com/aviz85/site-review ~/.claude/skills/site-review
```

Then invoke it in any Claude Code session:

```bash
/site-review https://yourapp.com
```

---

## Key principles

- **Narration pace sets the video length** — not the GIF. The GIF loops to match.
- **Music always at 30%** — present and audible, never competes with voice.
- **Magazine style** — observations, not instructions. Insights, not tutorials.
- **No title in the narration script** — the file is read directly by ElevenLabs. Any heading becomes spoken text.
- **RTL check is automatic** — Hebrew/Arabic SRT gets Unicode directional marks applied before embedding.

---

## Output example

Watch the Linear review produced by this skill:
[youtube.com/watch?v=O8p03dbDpHA](https://www.youtube.com/watch?v=O8p03dbDpHA)

---

Built with [Claude Code](https://claude.ai/code) by [aviz85](https://github.com/aviz85)
