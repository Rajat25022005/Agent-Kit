# Image Generation Backends — Provider & Codename Cheat Sheet

The same model often shows up under different names across providers. This
file decodes the most common aliases so you pick the right endpoint and don't
accidentally double-bill the user or hit a stale model id.

## "Nano Banana" — Google's codename for Gemini image models

Google's internal codename leaked via LMArena and stuck. The model family:

| Codename | Real model | Endpoint(s) | Notes |
|---|---|---|---|
| Nano Banana (original) | `gemini-2.5-flash-image` | Google direct; FAL (`fal-ai/nano-banana`) | Cheap, fast, decent quality. Default choice. |
| Nano Banana Pro / 2 | `gemini-3-pro-image-preview` (a.k.a. Gemini 3 Pro Image) | Google direct; FAL (`fal-ai/nano-banana-pro`); Hedra; nanophoto.ai; nanobananapro.cloud | Supports 1K/2K/4K. 2K/4K require the Pro model. Best text-in-image and reasoning. |
| Nano Banana 2 (Gemini 3.1 Flash) | `gemini-3.1-flash-image-preview` | Hedra, various resellers | Faster Pro-tier option, multi-subject ref, ~7 credits/gen on Hedra. |

**Direct Google API** (no middleman, lowest latency, image comes back as base64
`inlineData` in a `generateContent` response):

```
POST https://generativelanguage.googleapis.com/v1beta/models/{model}:generateContent?key=KEY
{
  "contents": [{"parts": [{"text": "..."}]}],
  "generationConfig": {
    "responseModalities": ["TEXT", "IMAGE"],
    "imageConfig": {"aspectRatio": "1:1", "imageSize": "1K"}
  }
}
```

Auth: `GEMINI_API_KEY` or `GOOGLE_API_KEY` in env. Aspect ratio enum:
`1:1, 2:3, 3:2, 3:4, 4:3, 4:5, 5:4, 9:16, 16:9, 21:9`. `imageSize` ∈ `1K | 2K | 4K`
(2K/4K rejected on non-Pro models).

**OpenRouter free tier** (no card, slow, sometimes rate-limited):
- `google/gemini-2.5-flash-image-preview:free` — returns base64 in a chat-completions response.

**Third-party resellers** (Hedra, nanophoto.ai, nanobananapro.cloud, etc.):
mostly async with a `taskId`/`generationId` + poll-for-result pattern. They
add a billing markup but bundle credit packs and a web UI. Read their docs
before assuming the request shape — each has its own auth header, payload
schema, and credit cost per resolution.

**FAL.ai (already wired into Hermes via `image_generation_tool.py`):**
- `fal-ai/nano-banana` → original Flash image
- `fal-ai/nano-banana-pro` → Pro tier, supports `resolution: 1K|2K|4K`,
  `aspect_ratio`, `enable_web_search`, `limit_generations`. Pricing
  ~$0.15/image at 1K.
- Use the existing `image_generation` tool — no need to write a new one if
  FAL is acceptable. The `aspect_ratio` key in the FAL payload corresponds
  to `imageConfig.aspectRatio` on the direct Google API.

## When the user says "nano banana" / "gemini image" / "banana pro"

1. Ask which provider they have credentials for, or which billing they want.
2. If they say "official" / "Google direct" / "no middleman" → write/extend a
   tool that calls `generativelanguage.googleapis.com` directly with their
   `GEMINI_API_KEY`. No FAL markup.
3. If they say "use what you have" / "FAL is fine" / "I have a FAL key" →
   point them at the existing FAL-backed tool with `fal-ai/nano-banana-pro`.
4. If they have nothing → `image_gen` is a no-op regardless of backend;
   surface the credential ask up front. **Never** report success without a
   real response payload from the chosen provider.

## What NOT to assume

- `gemini-2.5-flash-image` ≠ `gemini-2.5-flash-image-preview`. The latter is
  the older preview alias; both still work but prefer the stable id.
- "Nano Banana 2" in marketing copy can mean either Gemini 3.1 Flash Image
  (faster) OR Gemini 3 Pro Image (higher quality) — confirm the exact
  model id with the user before paying for 4K.
- Reseller pricing drifts weekly. The numbers in this file were correct as
  of June 2026; verify before quoting a user.

## Related

- `image_generation_tool.py` (Hermes built-in, FAL-backed) — handles
  nano-banana-pro as one model in a multi-model catalog.
- `comfyui` workflows (this skill) — for local generation, none of the
  Nano Banana models are usable; use Flux / SDXL / Wan instead.
