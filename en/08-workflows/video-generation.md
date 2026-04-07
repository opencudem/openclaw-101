# AI Video Generation

> Generate videos from text prompts and images using OpenClaw's built-in video_generate tool. Supports xAI (Grok), Runway, and Alibaba Model Studio.

**Version Introduced:** v2026.4.5 (April 6, 2026)  
**Tools:** `video_generate`  
**Providers:** xAI, Runway, Model Studio (Wan)

---

## Overview

The `video_generate` tool lets agents create short-form videos through configured providers. Videos are generated asynchronously and delivered when ready.

**Use Cases:**
- Content creation (social media clips, previews)
- Visual storytelling from text descriptions
- Video variations from reference images
- Prototyping and concept visualization

**Supported Providers:**
| Provider | Models | Features |
|----------|--------|----------|
| xAI | `grok-imagine-video` | Text-to-video, fast generation |
| Runway | `gen-3-alpha` | High-quality, image-to-video support |
| Model Studio | `wan-2.1` | Open-source Wan video model |

---

## Quick Start

### 1. Configure a Video Provider

**xAI (Grok):**
```json5
{
  providers: {
    video: {
      xai: {
        apiKey: "xai-...",  // or use XAI_API_KEY env var
        model: "grok-imagine-video"
      }
    }
  }
}
```

**Runway:**
```json5
{
  providers: {
    video: {
      runway: {
        apiKey: "rw-...",  // or use RUNWAY_API_KEY env var
        model: "gen-3-alpha"
      }
    }
  }
}
```

**Alibaba Model Studio:**
```json5
{
  providers: {
    video: {
      modelstudio: {
        apiKey: "sk-...",  // or use MODELSTUDIO_API_KEY env var
        model: "wan-2.1",
        baseUrl: "https://api.modelstudio.ai/v1"
      }
    }
  }
}
```

### 2. Generate a Video

```bash
# Text-to-video
openclaw tool video_generate --prompt "A serene mountain lake at sunrise, mist rising from the water" --provider xai

# Image-to-video (Runway)
openclaw tool video_generate --prompt "Camera slowly pans right across the landscape" --image ./landscape.jpg --provider runway
```

### 3. Check Generation Status

```bash
# List pending video tasks
openclaw tasks list --type video

# Get specific task status
openclaw tasks get <task-id>
```

---

## Agent Configuration

Enable video generation for specific agents:

```json5
{
  agents: {
    contentCreator: {
      tools: {
        video_generate: {
          enabled: true,
          defaultProvider: "xai",  // fallback provider
          maxDurationSeconds: 10,
          requireConfirmation: false  // set true for costly operations
        }
      }
    }
  }
}
```

**Agent Usage:**
```
Human: Create a 5-second video of a robot dancing
Agent: [uses video_generate tool with prompt "A humanoid robot dancing energetically in a studio"]
```

---

## Tool Reference

### video_generate

Generate video from text prompt and optional reference image.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `prompt` | string | Yes | Text description of desired video |
| `provider` | string | No | Provider override (xai, runway, modelstudio) |
| `image` | string | No | Path to reference image for image-to-video |
| `durationSeconds` | number | No | Target duration (5-10 seconds, provider-dependent) |
| `aspectRatio` | string | No | "16:9", "9:16", "1:1" (provider-dependent) |

**Response:**
```json
{
  "taskId": "vid_abc123",
  "status": "pending",
  "estimatedSeconds": 45,
  "provider": "xai"
}
```

**Async Delivery:**
When complete, the video file is delivered:
- In chat: as video attachment
- Via webhook: with download URL
- Via API: status endpoint includes download URL

---

## Provider-Specific Details

### xAI (Grok Imagine Video)

**Best for:** Fast generation, general-purpose videos

**Configuration:**
```json5
{
  providers: {
    video: {
      xai: {
        apiKey: "${XAI_API_KEY}",
        model: "grok-imagine-video",
        defaultDuration: 5,
        defaultAspectRatio: "16:9"
      }
    }
  }
}
```

**Limits:**
- Max duration: 5 seconds
- Aspect ratios: 16:9, 9:16, 1:1
- No image-to-video support

### Runway Gen-3

**Best for:** High-quality output, image-to-video

**Configuration:**
```json5
{
  providers: {
    video: {
      runway: {
        apiKey: "${RUNWAY_API_KEY}",
        model: "gen-3-alpha",
        defaultDuration: 10,
        defaultAspectRatio: "16:9"
      }
    }
  }
}
```

**Limits:**
- Max duration: 10 seconds
- Supports image reference for first frame
- Higher quality, longer generation time

### Alibaba Model Studio (Wan)

**Best for:** Open-source model, cost-effective

**Configuration:**
```json5
{
  providers: {
    video: {
      modelstudio: {
        apiKey: "${MODELSTUDIO_API_KEY}",
        model: "wan-2.1",
        baseUrl: "https://api.modelstudio.ai/v1",
        defaultDuration: 5
      }
    }
  }
}
```

**Limits:**
- Max duration: 5 seconds
- Requires Model Studio account

---

## Task Tracking

Video generation is asynchronous. Track tasks:

```bash
# List all video tasks
openclaw tasks list --type video --status pending

# Watch for completion
openclaw tasks watch <task-id>

# Cancel pending generation
openclaw tasks cancel <task-id>
```

**Task States:**
- `pending` — Queued for generation
- `generating` — Active generation in progress
- `complete` — Video ready for download
- `failed` — Generation error (check logs)

---

## Advanced: ComfyUI Workflows

For custom video pipelines, use the ComfyUI plugin:

```json5
{
  plugins: {
    entries: {
      comfy: {
        config: {
          baseUrl: "http://localhost:8188",
          workflows: {
            videoGenerate: {
              path: "./workflows/video-gen.json",
              inputs: {
                prompt: "{{prompt}}",
                width: 1024,
                height: 576
              }
            }
          }
        }
      }
    }
  }
}
```

See [ComfyUI Integration](../05-integrations/comfyui.md) for details.

---

## Prompt Engineering Tips

**Effective video prompts:**
- Be specific about motion: "camera slowly zooms in", "object rotates"
- Describe lighting and atmosphere
- Include temporal elements: "over 5 seconds", "gradually transforms"
- Avoid: copyrighted characters, violent content (provider-dependent filters)

**Example Good Prompts:**
```
"A cherry blossom tree swaying gently in spring breeze, petals falling slowly, golden hour lighting"

"Abstract fluid simulation, iridescent colors shifting from blue to purple, camera orbits slowly"

"Futuristic cityscape at night, flying cars passing between neon-lit buildings, rain on lens"
```

---

## Troubleshooting

### "Provider not found"

**Cause**: Provider not configured in `providers.video.*`

**Fix**: Add provider configuration with API key.

### "Video generation timeout"

**Cause**: Generation taking longer than expected.

**Fix**: Check task status with `openclaw tasks get <id>`. Some providers (Runway) take 60-120 seconds.

### "Invalid aspect ratio"

**Cause**: Provider doesn't support requested ratio.

**Fix**: Use supported ratios:
- xAI: 16:9, 9:16, 1:1
- Runway: 16:9, 9:16, 4:3, 1:1
- Model Studio: 16:9, 1:1

### "API key invalid"

**Check**: Ensure API key is valid and has video generation permissions:
```bash
# Test with direct provider
openclaw provider test --provider xai --type video
```

---

## Pricing Considerations

Video generation uses provider APIs with associated costs:

| Provider | Cost per video | Notes |
|----------|---------------|-------|
| xAI | ~$0.10-0.30 | Varies by duration |
| Runway | ~$0.50-1.00 | Higher quality, higher cost |
| Model Studio | ~$0.05-0.15 | Most cost-effective |

**Cost Control:**
```json5
{
  agents: {
    myAgent: {
      tools: {
        video_generate: {
          enabled: true,
          requireConfirmation: true,  // Ask before generating
          dailyLimit: 10  // Max generations per day
        }
      }
    }
  }
}
```

---

## Related Documentation

- [Music Generation](./music-generation.md) — AI music generation
- [Image Generation](./image-generation.md) — AI image creation
- [ComfyUI Integration](../05-integrations/comfyui.md) — Custom media workflows
- [Task Management](../06-automation/task-flow.md) — Background task handling
- [Provider Configuration](../10-cli-and-reference/config-reference.md#providers)

---

**Last Updated**: April 7, 2026  
**Applies To**: OpenClaw v2026.4.5+
