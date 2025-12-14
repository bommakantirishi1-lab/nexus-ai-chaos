# NEXUS: AI CHAOS - PIPELINE ARCHITECTURE

## EXECUTIVE SUMMARY

NEXUS_AI_CHAOS_PIPELINE is a production-grade, deterministic workflow for generating vertical short-form video content (max 60 seconds) optimized for cyber + AI audience engagement. The pipeline chains ChatGPT and Grok Imagine into an automated, repeatable content generation system with multi-platform distribution capability.

---

## STEP 1: DEPRECATION OF LEGACY WORKFLOWS

### Deprecation Strategy
Any existing video automation workflows are deprecated without deletion to preserve audit trails.

**Process:**
1. Rename legacy workflow file to `{filename}.deprecated.yml`
2. Add header comment explaining deprecation rationale
3. Strip all triggers (cron, push, pull_request, workflow_dispatch)
4. Retain file in .github/workflows for historical reference
5. No cleanup; retention for compliance

**Safety Enforcement:**
- Triggers disabled at YAML root level
- No `on:` events configured
- Cannot execute via UI, CLI, or automated events
- Read-only reference only

---

## STEP 2: NEXUS_AI_CHAOS_PIPELINE ARCHITECTURE

### Pipeline Name
`NEXUS_AI_CHAOS_PIPELINE`

### Execution Modes
1. **Manual Trigger** (workflow_dispatch)
   - On-demand video generation via GitHub Actions UI
   - Input-driven with JSON orchestration
   - Supports immediate deployment

2. **Scheduled Execution** (cron)
   - Daily trigger at 06:00 UTC
   - Automated content generation
   - Deterministic input defaults

### End-to-End Pipeline Flow

```
INPUT (JSON) 
  ↓
CHATGPT SCRIPT GENERATION 
  ↓
SCENE DECOMPOSITION & TAGGING 
  ↓
GROK IMAGINE PROMPT GENERATION 
  ↓
VIDEO GENERATION (Grok) 
  ↓
POST-PROCESSING (Captions, Trimming, Aspect Ratio) 
  ↓
ARTIFACT PACKAGING 
  ↓
MULTI-PLATFORM OUTPUT
```

---

## STEP 3: TOOLS & SERVICES SPECIFICATION

### 3.1 ChatGPT (OpenAI)

**Responsibility:** Script generation and scene orchestration

**Input Contract:**
```json
{
  "topic": "string - cyber/AI security concept",
  "tone": "dark | cinematic | educational | hype",
  "duration_seconds": "number (max 60)",
  "cta_required": "boolean"
}
```

**Output Contract:**
```json
{
  "script": "string - full video narration",
  "scenes": [
    {
      "scene_number": "integer",
      "duration_ms": "integer",
      "visual_description": "string - cinematic direction",
      "camera_motion": "string - pan, zoom, dolly, static",
      "mood": "string - dark, intense, technical",
      "on_screen_text": "string (optional)"
    }
  ],
  "hook_start_ms": "integer - when retention hook fires (0-2000ms)"
}
```

**Constraints:**
- Scene duration strictly 2-4 seconds each
- Total runtime must not exceed duration_seconds
- First scene hook within 2 seconds
- No emojis or hashtags in output
- Dark, authoritative tone mandatory

### 3.2 Grok Imagine (xAI)

**Responsibility:** Visual content generation for vertical video

**Input Contract:**
Single unified prompt synthesizing all scenes into coherent visual direction

**Output Contract:**
High-resolution image asset optimized for 9:16 aspect ratio (vertical)

**Prompt Engineering Rules:**
- Emphasize first 1.5 seconds visually (hook strength)
- Cyberpunk + dark aesthetic mandatory
- Visual consistency across temporal progression
- High-contrast, server + AI core visuals
- Cinematic lighting
- No friendly or playful visual elements

### 3.3 Post-Processing & Artifact Management

**Responsibility:** Output normalization and preparation

**Input:**
- Grok Imagine image(s)
- ChatGPT script + scene metadata
- Platform-specific requirements

**Processing Steps:**
1. Aspect ratio enforcement (9:16 for vertical)
2. Caption overlay generation (on_screen_text from scenes)
3. Duration trimming (enforce max 60s)
4. Audio synthesis (script → TTS)
5. Watermark/branding application
6. Format conversion (MP4/WebM)

**Output Artifacts:**
- `video_master.mp4` (source)
- `video_youtube_shorts.mp4` (optimized)
- `video_instagram_reels.mp4` (optimized)
- `video_x_video.mp4` (optimized)
- `metadata.json` (title, description, tags)

---

## STEP 4: CONTENT ORCHESTRATION INPUT CONTRACT

### JSON Input Schema

```json
{
  "topic": "string (required)",
  "platform": "youtube_shorts | instagram_reels | x_video (required)",
  "duration_seconds": "number (required, max 60)",
  "tone": "dark | cinematic | educational | hype (required)",
  "visual_style": "cyberpunk | cinematic | realistic (required)",
  "cta_required": "boolean (optional, default false)"
}
```

### Input Validation Rules
1. `topic` must be cyber/AI-related
2. `duration_seconds` must be integer between 15-60
3. `platform` must be one of three options
4. `tone` must be one of four options
5. `visual_style` must be one of three options
6. No null values permitted

### Output Schema

```json
{
  "pipeline_run_id": "UUID",
  "timestamp": "ISO 8601",
  "input": { ... },
  "video_assets": {
    "youtube_shorts": "s3://bucket/video_yt_shorts.mp4",
    "instagram_reels": "s3://bucket/video_ig_reels.mp4",
    "x_video": "s3://bucket/video_x.mp4"
  },
  "metadata": {
    "title": "string",
    "description": "string",
    "tags": ["string"],
    "hook_timestamp_ms": "integer"
  },
  "status": "success | failure",
  "error_log": "string (null on success)"
}
```

---

## STEP 5: CHATGPT RESPONSIBILITIES (DETAILED)

### Core Task: Script Generation

Given topic + parameters, generate a high-retention cybersecurity + AI short-form video script.

### Hook Requirements
- **First 2 seconds:** Visual/narrative hook
- **Goal:** Immediate attention capture
- **Technique:** Shocking stat, question, or visual premise

### Tone Requirements (NEXUS: AI CHAOS Brand)
- **Dark:** No light-hearted language
- **Bold:** Authoritative, confident statements
- **Futuristic:** Forward-looking, speculative
- **Unsettling:** Tension, urgency, warning undertones
- **No emojis, no hashtags**

### Scene Structure
Each scene must include:
```json
{
  "scene_number": 1,
  "duration_ms": 3000,
  "visual_description": "Dark server room, glowing red core, code cascading",
  "camera_motion": "Dolly zoom inward",
  "mood": "Intense, technical threat",
  "on_screen_text": "ZERO DAY DETECTED"
}
```

### Constraints
- 2-4 seconds per scene (mandatory)
- No scene under 2s or over 4s
- Maximum 15 scenes for 60s video
- Total duration must be <= duration_seconds
- Hook fires within first 2 seconds

---

## STEP 6: GROK IMAGINE RESPONSIBILITIES (DETAILED)

### Core Task: Unified Visual Prompt

Generate ONE single comprehensive Grok Imagine prompt that encapsulates the entire visual journey of the video.

### Prompt Characteristics
- **Vertical-first (9:16 aspect ratio)**
- **Cinematic quality**
- **Dark color palette**
- **High contrast**
- **Cyberpunk + AI core emphasis**
- **First 1.5 seconds weighted for hook impact**

### Visual Brand Elements (NEXUS: AI CHAOS)
- Servers with glowing cores
- Abstract data streams
- Circuit patterns
- Red/blue color dominance
- Tech-forward, authoritative
- Zero playful elements
- Professional, unsettling atmosphere

### Prompt Template Structure
```
Cinematic vertical video (9:16), dark cyberpunk aesthetic.
Opening 1.5 seconds: [hook visual concept].
Progressively: [scene progression].
Color: dark reds, deep blues, blacks.
Vibe: authoritative, technical, unsettling.
Style: high-contrast, professional, futuristic.
No playful elements. Emphasis on technological threat/innovation.
```

---

## STEP 7: BRANDING RULES (NON-NEGOTIABLE)

### Brand Identity
- **Channel Name:** NEXUS: AI CHAOS
- **Tone:** Dark, authoritative, futuristic, unsettling
- **Target Audience:** Cybersecurity professionals, AI researchers, threat hunters

### Visual Identity
- **Primary Colors:** Red (#FF0000), Blue (#0055FF), Black (#000000)
- **Lighting:** Low-key, high-contrast, server/data-centric
- **No Playful Visuals:** Avoid friendly characters, light colors, casual aesthetics
- **Subject Matter:** Servers, AI cores, code, threats, innovation

### Content Constraints
- Zero emojis
- Zero hashtags
- No references to happiness, positivity, fun
- Dark, forward-looking messaging
- Technical depth prioritized over simplicity
- Threat-aware language

---

## IMPLEMENTATION READINESS

This architecture is implementation-ready. No code is specified here; all outputs are:
1. **Structured** - JSON contracts defined
2. **Clear** - Responsibilities unambiguous
3. **Deterministic** - Repeatable outputs
4. **Enforceable** - Validation rules explicit

GitHub Actions workflow file (`nexus-ai-chaos.yml`) implements this architecture.
