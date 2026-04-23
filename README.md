# Micro-Drama-Skills 🎬

An AI-powered end-to-end micro-drama production system. It uses Claude Skills to implement a complete workflow, from scriptwriting and character design to storyboard generation and video submission.

## Project Overview

This project provides a set of **Claude Skills** for automatically generating micro-drama productions. Each production contains 25 episodes (30 seconds per episode). The system automatically generates the script, character settings, 6-panel storyboard images, storyboard configurations, and can also call AI APIs to generate images and videos, then submit everything to the Seedance video generation pipeline.

### Core Capabilities

| Skill | Function | Example Trigger Command |
|------|------|-------------|
| **produce-anime** | Generates a complete micro-drama (script + characters + storyboards) | "Produce a sci-fi micro-drama" |
| **generate-media** | Calls the Gemini API to generate character images / storyboard images / videos | "Generate images for DM-002" |
| **submit-anime-project** | Batch-submits tasks to Seedance for video generation | "Submit DM-002 to Seedance" |

## Directory Structure

```
.
├── .claude/skills/                    # Claude Skill definitions
│   ├── produce-anime/SKILL.md         # Micro-drama production skill
│   ├── generate-media/SKILL.md        # Media generation skill
│   └── submit-anime-project/SKILL.md  # Task submission skill
├── .config/
│   ├── api_keys.sample.json           # Example API configuration
│   ├── api_keys.json                  # API configuration (create manually; gitignored)
│   └── visual_styles.json             # Visual style presets (10 styles)
├── projects/
│   ├── index.json                     # Global project index
│   ├── DM-001_dhgt/                   # "Lights on the Way Home"
│   └── DM-002_tjkc/                   # "Carbon Gold Frenzy"
└── README.md
```

### Single Project Directory Structure

```
DM-002_tjkc/
├── metadata.json                      # Project metadata
├── script/full_script.md              # Full script (25 episodes)
├── characters/
│   ├── character_bible.md             # Character bible
│   ├── ref_index.json                 # Character reference image index
│   ├── 林策_ref.png                    # Character reference image (gitignored)
│   └── ...
├── episodes/
│   ├── EP01/
│   │   ├── dialogue.md                # Dialogue script
│   │   ├── storyboard_config.json     # Storyboard config (6-panel × upper/lower halves)
│   │   ├── seedance_tasks.json        # Seedance submission tasks
│   │   ├── DM-002-EP01-A_storyboard.png  # Upper-half storyboard image (gitignored)
│   │   └── DM-002-EP01-B_storyboard.png  # Lower-half storyboard image (gitignored)
│   └── ... (EP01-EP25)
├── seedance_project_tasks.json        # Full-project task summary (50 items)
├── video_index.json                   # Video numbering index
└── generate_media.py                  # Media generation script
```

## Quick Start

### 1. Configure the API

Copy the sample config and fill in your API key:

```bash
cp .config/api_keys.sample.json .config/api_keys.json
```

Edit `.config/api_keys.json`:

```json
{
  "gemini_api_key": "YOUR_GEMINI_API_KEY",
  "base_url": "https://generativelanguage.googleapis.com/",
  "gemini_image_model": "gemini-2.5-flash-image-preview"
}
```

### 2. Install Dependencies

```bash
python -m venv .venv
source .venv/bin/activate
pip install google-genai Pillow requests
```

### 3. Use Claude Skills

Open this project in a tool that supports Claude Skills, such as Claude Code or OpenClaw. The skills will load automatically.

**Produce a micro-drama:**
```
> Produce a retro Hong Kong-style micro-drama
> Produce a cyberpunk-style campus micro-drama
```

**Generate media:**
```
> Generate storyboard images for DM-002
> Generate images for episodes 1 through 5
```

**Submit tasks:**
```
> Submit DM-002 to Seedance (simulation mode)
```

## Visual Style Presets

The system includes 10 built-in cinematic visual styles. During production, they can be specified by name, ID, or Chinese name.

| ID | English Name | Chinese Name | Camera / Characteristics |
|----|--------|--------|------------|
| 1 | Cinematic Film | 电影质感 | Panavision Sphero 65, Vision3 500T (**default**) |
| 2 | Anime Classic | 经典动漫 | Studio Ghibli hand-drawn style |
| 3 | Cyberpunk Neon | 赛博朋克 | RED Monstro 8K, high-contrast neon |
| 4 | Chinese Ink Painting | 水墨国风 | ARRI ALEXA Mini LF, ink-paint rendering |
| 5 | Korean Drama | 韩剧氛围 | Sony VENICE 2, warm tones and shallow depth of field |
| 6 | Dark Thriller | 暗黑悬疑 | ARRI ALEXA 65, chiaroscuro lighting |
| 7 | Vintage Hong Kong | 港风复古 | Kodak Vision3, Cooke Anamorphic |
| 8 | Wuxia Epic | 武侠大片 | Panavision DXL2, large-scale scenes with mist |
| 9 | Soft Romance | 甜蜜恋爱 | Canon C500, soft focus and warm tones |
| 10 | Documentary Real | 纪实写实 | Sony FX6, handheld natural light |

Each style’s `prompt_suffix` is automatically appended to the end of all AI-generated prompts. You can customize or add new styles in `.config/visual_styles.json`.

## Seedance Task Submission

The system maps each storyboard image (A/B, one image each) to one Seedance task. That means 2 tasks per episode and 50 tasks for the full production.

### Task Prompt Structure

```
(@DM-002-EP01-A_storyboard.png) is a 6-panel storyboard reference image,
(@林策_ref.png) is the reference appearance for the character "Lin Ce", (@沈璃_ref.png) is the reference appearance for the character "Shen Li"...

Starting from shot 1, do not display the multi-panel storyboard reference image. Turn the storyboard into a film-grade HD visual production...

DM-002-EP01-A Episode 1 "Breath Tax Era" first half. Plot summary. Atmosphere.

Shot 1 (0.0s-2.5s): Scene description. (@林策_ref.png) Lin Ce action... Lin Ce says: "dialogue" (emotion)
Shot 2 (2.5s-5.0s): ...
...
Shot 6 (12.5s-15.0s): ...
```

### Seedance API

- Service URL: `http://localhost:3456`
- Main endpoint: `POST /api/tasks/push`
- Supports batch submission (`tasks` array)
- `realSubmit: false` = simulation mode, `true` = real submission

## Technology Stack

- **AI Skills Platform**: Claude Skills (`.claude/skills/`)
- **Image Generation**: Google Gemini (`gemini-2.5-flash-image-preview` / `gemini-3-pro-image-preview`)
- **Video Generation**: Google Veo 2 (`veo-2.0-generate-001`)
- **Task Submission**: Seedance video generation pipeline (HTTP REST API)
- **Runtime Environment**: Python 3.13+, `google-genai` SDK

## Existing Projects

| Code | Name | Genre | Status |
|------|------|------|------|
| DM-001 | "Lights on the Way Home" | — | Script completed |
| DM-002 | "Carbon Gold Frenzy" | Sci-fi / finance / suspense | Script completed + storyboard images generated + tasks generated |

## License

MIT
