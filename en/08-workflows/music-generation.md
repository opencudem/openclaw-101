# Music Generation

> Generate music and audio with AI using Google Lyria, MiniMax, or ComfyUI workflows.

## What this tool solves

The `music_generate` tool creates original music and audio from text prompts. Use it for:
- Background music for content
- Sound effects and ambience
- Musical sketches and ideation
- Audio branding and jingles

## Providers

| Provider | Best For | Notes |
|----------|----------|-------|
| **Google Lyria** | High-quality instrumental music | Best for complex compositions |
| **MiniMax** | Fast generation, Chinese market | Good for simple loops |
| **ComfyUI** | Workflow-driven audio | Full control via Comfy workflows |

## Quick Start

### Via Chat

```
User: Generate a 30-second ambient synth loop
Agent: [Uses music_generate with prompt, delivers audio file when complete]
```

### Via Tool Call

```json
{
  "music_generate": {
    "prompt": "Warm ambient synth loop with soft tape texture, 30 seconds",
    "duration": 30
  }
}
```

## Configuration

### Google Lyria

```json5
{
  models: {
    providers: {
      google: {
        apiKey: "${GOOGLE_API_KEY}",
      },
    },
  },
}
```

### MiniMax

```json5
{
  models: {
    providers: {
      minimax: {
        apiKey: "${MINIMAX_API_KEY}",
      },
    },
  },
}
```

### ComfyUI Workflow

```json5
{
  models: {
    providers: {
      comfy: {
        mode: "local", // or "cloud"
        baseUrl: "http://127.0.0.1:8188",
        music: {
          workflowPath: "./workflows/music-api.json",
          promptNodeId: "3",
          outputNodeId: "18",
        },
      },
    },
  },
}
```

## Tool Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `prompt` | string | Yes | Text description of desired music |
| `duration` | number | No | Target duration in seconds (hint only) |
| `style` | string | No | Genre/style hints (jazz, electronic, ambient) |
| `instrumental` | boolean | No | Force instrumental (no vocals) |

## Provider-Specific Notes

### Google Lyria
- Highest quality for complex compositions
- May ignore unsupported hints (like exact duration) with warning
- Best for: orchestral, jazz, electronic, ambient

### MiniMax
- Faster generation times
- Optimized for Chinese market/use cases
- Best for: simple loops, background music

### ComfyUI
- Full workflow control
- Supports reference images (for image-to-audio)
- Best for: custom pipelines, batch generation

## Task Tracking

Music generation is async. The tool:
1. Submits generation request
2. Returns task ID immediately
3. Polls for completion
4. Delivers audio file when ready

**Timeout:** Configurable via `agents.defaults.musicGenerateTimeout` (default: 300 seconds)

## Examples

### Electronic beat
```json
{
  "music_generate": {
    "prompt": "Upbeat electronic dance track with strong bassline, 4/4 beat, energetic",
    "duration": 60,
    "style": "electronic"
  }
}
```

### Ambient background
```json
{
  "music_generate": {
    "prompt": "Soft ambient pads, no percussion, meditative atmosphere",
    "duration": 120,
    "instrumental": true
  }
}
```

### Jazz trio
```json
{
  "music_generate": {
    "prompt": "Piano jazz trio, walking bass, brush drums, laid-back swing",
    "style": "jazz",
    "instrumental": true
  }
}
```

## Output

The tool returns:
- **Audio file** (MP3 or WAV)
- **Metadata** (duration, provider used, generation time)
- **Task details** (if async delivery needed)

## Limitations

- Duration is a hint, not guaranteed (providers may approximate)
- Some providers don't support vocals (check instrumental flag)
- Large files may require longer timeouts

## Related

- [Video Generation](./video-generation.md) — Generate videos with matching audio
- [Image Generation](../04-skills/) — Create cover art for your music

---

**Version:** v2026.4.5+
**Providers:** Google Lyria, MiniMax, ComfyUI
**Tool:** `music_generate`