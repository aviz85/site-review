---
name: site-review
description: "Full pipeline: explore any website/app → record GIF → document findings in MD → write magazine-style review script → generate narration + music → assemble looped video → burn subtitles. Use for: site review, platform review, app walkthrough video, demo video, product review reel, סקירת אתר, סרטון סקירה."
---

# Site Review

Full pipeline for creating a magazine-style video review of any website or app.

**What gets produced:**
1. `review.md` — Live documentation built during exploration
2. `exploration.gif` — Screen recording of the walkthrough
3. `narration-script.md` — Magazine-style script with direction cues
4. `narration.mp3` — ElevenLabs voice narration
5. `bg-music.mp3` — Background music
6. `final.mp4` — Complete video: looped GIF + audio mix + burned subtitles

## Usage

```
/site-review <url> [--lang en|he] [--title "Custom Title"] [--gif /path/to/existing.gif] [--output /path/to/dir]
```

## Options

| Option | Default | Description |
|--------|---------|-------------|
| `url` | (required) | Website or app URL to review |
| `--lang` | `en` | Narration language: `en` (English) or `he` (Hebrew, RTL subtitles) |
| `--title` | auto-detect | Review title shown in docs |
| `--gif` | (none) | Reuse existing GIF (skips recording step) |
| `--output` | `/tmp/site-review-TIMESTAMP` | Output directory |

---

## Step-by-Step Instructions

### Step 0: Setup

Parse arguments. Create output directory:

```bash
TIMESTAMP=$(date +%s)
OUTPUT_DIR="/tmp/site-review-$TIMESTAMP"
mkdir -p "$OUTPUT_DIR"
```

Initialize `$OUTPUT_DIR/review.md` with this header:

```markdown
# Site Review: [SITE NAME]
**URL:** [URL]
**Date:** [TODAY]

---

## Initial Impressions

## Key Features Observed

## UX & Navigation

## Data & Scale

## Notable Findings

## Automation Opportunities
```

---

### Step 1: Plan the Exploration

Before opening the browser, plan what to cover:
- Landing page / homepage
- Main navigation sections
- Core feature areas (dashboards, lists, forms)
- Any workflows or key interactions
- Footer / about / pricing if relevant

Target: 60–90 seconds of content.

---

### Step 2: Record GIF + Explore

**If `--gif` was provided:** skip this step, use the provided GIF.

**Otherwise:**

Use `mcp__claude-in-chrome__gif_creator` to record:

```
Start GIF recording → output: $OUTPUT_DIR/exploration.gif, fps: 2
Navigate to the URL
Systematically explore: homepage → navigation → core features → workflows → interesting details
Stop GIF after 60–90 seconds
```

**During exploration, continuously update `review.md`:**
- Feature names and counts
- Data volumes, ticket numbers, user counts
- Navigation structure
- UI patterns (tables, cards, forms)
- Anything surprising or notable
- Automation opportunities spotted

Keep observations factual and specific. Numbers are gold.

---

### Step 3: Finalize Documentation

Complete `review.md` with:
- Full feature inventory
- Identified pain points or inefficiencies
- Data volumes observed
- Tech stack hints (Zoho? React? Custom CRM?)
- Specific automation opportunities with estimated impact

---

### Step 4: Write the Narration Script

Write to `$OUTPUT_DIR/narration-script.md`.

**CRITICAL: No title, no heading, no "Narration Script:" prefix.** The file is read directly by ElevenLabs — start with the first spoken word. Any markdown heading will be read aloud as speech.

**Magazine-style voice** — not a tutorial, not a demo. Like a journalist who just got inside access.

**Structure (for ~90 seconds / ~200 words English / ~150 words Hebrew):**

```
[direction cue] HOOK — the most striking number or observation. Open with impact.

[direction cue] DISCOVERY — what lives inside this platform. The detail that surprises.

[pause]

[direction cue] INSIGHT — what this reveals. The bigger picture. Why it matters.

[building intensity]

[direction cue] PROPOSITION — what AI / automation changes about this picture.

[deliberate, measured]

CLOSING LINE — one strong sentence.
```

**Direction cues in square brackets** (before the relevant sentence):
- `[confident, measured tone]` — for openings
- `[pause]` — beat of silence
- `[emphatic]` — stress this
- `[building intensity]` — accelerate energy
- `[slower, deliberate]` — land the point
- `[warm]` — human moment

**English example style:**
> [confident, measured tone] Twenty-eight thousand open tickets. That is not a backlog — that is a business problem sitting in a spreadsheet.
>
> [building intensity] Each row is a real person. A real claim. Waiting.
>
> [pause]
>
> [deliberate] The workflow is not broken. It is simply manual. And manual at this scale has a cost.

**Hebrew example style:**
> [בטחון שקט] עשרים ושמונה אלף כרטיסים פתוחים. זה לא פיגור — זו הזדמנות.

---

### Step 5: Generate Narration Audio

Use the `speech-generator` skill:
- Source file: `$OUTPUT_DIR/narration-script.md`
- Output: `$OUTPUT_DIR/narration.mp3`
- Language: English (default) or Hebrew if `--lang he`
- Model: `eleven_multilingual_v2`

After generating, get duration:
```bash
NARRATION_DURATION=$(ffprobe -v quiet -show_entries format=duration -of csv=p=0 "$OUTPUT_DIR/narration.mp3")
echo "Narration: ${NARRATION_DURATION}s"
```

---

### Step 6: Generate Background Music

Use the `music-generator` skill:
- Prompt: `"Subtle corporate ambient background, professional and understated, soft electronic texture, minimal melody, no drums"`
- Duration: `NARRATION_DURATION + 5` seconds (a little extra)
- Output: `$OUTPUT_DIR/bg-music.mp3`
- Instrumental: yes

---

### Step 7: Assemble the Video

**7a. Convert GIF → MP4**
```bash
ffmpeg -i "$OUTPUT_DIR/exploration.gif" \
  -vf "scale=trunc(iw/2)*2:trunc(ih/2)*2" \
  -c:v libx264 -pix_fmt yuv420p \
  "$OUTPUT_DIR/base.mp4"
```

**7b. Get base video duration**
```bash
GIF_DURATION=$(ffprobe -v quiet -show_entries format=duration -of csv=p=0 "$OUTPUT_DIR/base.mp4")
```

**7c. Loop if GIF is shorter than narration**

If `GIF_DURATION < NARRATION_DURATION`:
```bash
LOOPS=$(python3 -c "import math; print(math.ceil($NARRATION_DURATION / $GIF_DURATION) + 1)")

python3 -c "
loops = $LOOPS
with open('$OUTPUT_DIR/concat.txt', 'w') as f:
    for _ in range(loops):
        f.write(\"file '$OUTPUT_DIR/base.mp4'\\n\")
"

ffmpeg -f concat -safe 0 -i "$OUTPUT_DIR/concat.txt" \
  -t "$NARRATION_DURATION" \
  -c:v libx264 -pix_fmt yuv420p \
  "$OUTPUT_DIR/loop.mp4"

VIDEO_INPUT="$OUTPUT_DIR/loop.mp4"
```

If GIF is already long enough:
```bash
ffmpeg -i "$OUTPUT_DIR/base.mp4" -t "$NARRATION_DURATION" -c copy "$OUTPUT_DIR/loop.mp4"
VIDEO_INPUT="$OUTPUT_DIR/loop.mp4"
```

**7d. Mix audio: narration (100%) + music (30%)**
```bash
ffmpeg -i "$VIDEO_INPUT" \
  -i "$OUTPUT_DIR/narration.mp3" \
  -i "$OUTPUT_DIR/bg-music.mp3" \
  -filter_complex "[1:a]volume=1.0[narr];[2:a]volume=0.30[music];[narr][music]amix=inputs=2:duration=first[aout]" \
  -map 0:v -map "[aout]" \
  -c:v libx264 -c:a aac \
  -t "$NARRATION_DURATION" \
  "$OUTPUT_DIR/video-nosubtitles.mp4"
```

---

### Step 8: Generate Subtitles (SRT)

Use the `transcribe` skill:
- Input: `$OUTPUT_DIR/narration.mp3`
- Output: `$OUTPUT_DIR/subtitles.srt`
- Mode: `--cheap` (Groq whisper — fast, word-level, accurate)

After transcription, **review the SRT file** for:
- Numbers rendered with `$` signs (e.g., `$6650` → `6,650`)
- Mis-transcribed proper nouns (fix to match the script)
- Any duplicated or garbled entries

Fix errors by editing the SRT file before the next step.

---

### Step 9: Embed Subtitles

Use the `embed-subtitles` skill:
- Input: `$OUTPUT_DIR/video-nosubtitles.mp4`
- Subtitles: `$OUTPUT_DIR/subtitles.srt`
- Output: `$OUTPUT_DIR/final.mp4`
- Font size: 22, position: bottom, margin: 30
- **RTL auto-detection**: if narration is Hebrew, the embed-subtitles skill will detect it and apply the RTL fix automatically

---

### Step 10: Report Output

```
✓ Site Review complete: [SITE NAME]
Duration: [NARRATION_DURATION]s

Output files in $OUTPUT_DIR:
  review.md              — Full exploration notes
  narration-script.md    — Review script
  exploration.gif        — Raw GIF recording
  final.mp4              — Complete video

Next steps:
  WAME the video         — WhatsApp it to yourself
  /youtube-uploader      — Publish to YouTube
  /linkedin              — Share on LinkedIn
```

---

## Sub-Skills Called

| Step | Skill / Tool |
|------|-------------|
| GIF recording | `mcp__claude-in-chrome__gif_creator` |
| Narration | `speech-generator` |
| Background music | `music-generator` |
| Subtitles (SRT) | `transcribe --cheap` |
| Subtitle embedding | `embed-subtitles` |

## Key Principles

- **Narration pace sets the video length** — not the GIF. Loop GIF to match.
- **Music always at 30%** — present and audible, never competes with voice.
- **Magazine style** — observations, not instructions. Insights, not tutorials.
- **Numbers are the hook** — always open with the most striking data point.
- **RTL check is mandatory** — always verify Hebrew SRT gets the RTL fix.

ARGUMENTS:
