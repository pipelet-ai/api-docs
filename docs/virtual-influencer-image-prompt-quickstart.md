---
title: Virtual Influencer Image Prompt Quickstart
permalink: /docs/virtual-influencer-image-prompt/
---

# Virtual Influencer Image Prompt API

The `virtual-influencer-image-prompt` queue generates a detailed **text-to-image prompt** for a virtual influencer character. Given an existing influencer (from `virtual-influencer-init`), it:

1. Loads the character's description, backstory, and reference images from storage
2. Fetches dynamic scene candidates from `assets.pipelet.ai/location/place` (4 random places × 2 scenes each = up to 8 candidates)
3. Calls a Vision LLM that sees the reference images and generates a photorealistic, editorial-quality prompt (~250 words)

The resulting prompt is ready to feed into text-to-image models like `z-image-turbo`, `nano-banana`, `qwen-text-to-image`, etc.

**Typical end-to-end time: 15–30 seconds.**

## Base URL

- Production: `https://api.pipelet.ai`

## Endpoints (quick reference)

**Queue (submit + poll):**
- Submit job: `POST /fal/queue/virtual-influencer-image-prompt`
- Check status: `GET /fal/queue/virtual-influencer-image-prompt/requests/:requestId/status`
- Cancel: `PUT /fal/queue/virtual-influencer-image-prompt/requests/:requestId/cancel`
- Fetch result (raw r2:// URIs): `GET /fal/queue/virtual-influencer-image-prompt/requests/:requestId`

**Workflow status (result with presigned image URLs):**
- `GET /v1/workflow/virtual-influencer-image-prompt/status/:instanceId`

The `request_id` returned on submission is the same as the Cloudflare Workflow `instanceId` — use it for both endpoints.

---

## Required Headers

| Header | Required | Description |
|---|---|---|
| `Authorization` | Yes | `Bearer <your-api-key>` — requires `user:write` scope |
| `x-user-id` | Yes | Your application's user identifier. Must match the user who owns the influencer record. |
| `Content-Type` | Yes | `application/json` |

---

## 1) Submit a Job

`POST /fal/queue/virtual-influencer-image-prompt`

**Request body** (JSON):

| Field | Type | Required | Description |
|---|---|---|---|
| `influencer_id` | string | **Yes** | The influencer handle from `virtual-influencer-init` (e.g. `vi_3534e3bfc6c6_0`) |
| `prompt` | string | No | Free-text scenario hint, e.g. `"skateboarding in the city"` or `"attending a gallery opening"`. If omitted, the LLM picks the most interesting scene from the candidates. |

```bash
curl -XPOST "https://api.pipelet.ai/fal/queue/virtual-influencer-image-prompt" \
  -H "Authorization: Bearer <your-api-key>" \
  -H "x-user-id: user-abc123" \
  -H "Content-Type: application/json" \
  -d '{
    "influencer_id": "vi_3534e3bfc6c6_0",
    "prompt": "skateboarding in the city"
  }'
```

**Response:**
```json
{
  "request_id": "934291ae-1694-4382-96ce-0a87ba50a6d6",
  "response_url": "https://api.pipelet.ai/fal/queue/virtual-influencer-image-prompt/requests/934291ae-1694-4382-96ce-0a87ba50a6d6",
  "status_url": "https://api.pipelet.ai/fal/queue/virtual-influencer-image-prompt/requests/934291ae-1694-4382-96ce-0a87ba50a6d6/status",
  "cancel_url": "https://api.pipelet.ai/fal/queue/virtual-influencer-image-prompt/requests/934291ae-1694-4382-96ce-0a87ba50a6d6/cancel"
}
```

---

## 2) Poll for Status

```bash
curl "https://api.pipelet.ai/fal/queue/virtual-influencer-image-prompt/requests/934291ae-1694-4382-96ce-0a87ba50a6d6/status" \
  -H "Authorization: Bearer <your-api-key>" \
  -H "x-user-id: user-abc123"
```

**Response while running:**
```json
{
  "status": "IN_QUEUE",
  "queue_position": 0,
  "progress": 0,
  "statusMessages": ["in-progress"],
  "etaSeconds": 0,
  "response_url": "https://api.pipelet.ai/fal/queue/virtual-influencer-image-prompt/requests/934291ae-..."
}
```

Poll every 15 seconds. Typical completion is within 30 seconds.

---

## 3) Fetch Result

### Option A — Fal queue result endpoint (r2:// URIs, no presigning)

```bash
curl "https://api.pipelet.ai/fal/queue/virtual-influencer-image-prompt/requests/934291ae-1694-4382-96ce-0a87ba50a6d6" \
  -H "Authorization: Bearer <your-api-key>" \
  -H "x-user-id: user-abc123"
```

**Response:**
```json
{
  "statusMessages": "completed",
  "result": {
    "influencer": {
      "id": "vi_3534e3bfc6c6_0",
      "status": "completed",
      "user_id": "user-abc123",
      "name_english": "Mei",
      "name_ethnic": "Lin",
      "description": "This person has an oval face with a soft jawline...",
      "backstory": "A brilliant architectural conservator based in Shanghai...",
      "base_image_r2_uri": "r2://long-term-storage/workflows/virtual-influencer/.../base_0.png",
      "angle_image_r2_uris": [
        "r2://long-term-storage/workflows/virtual-influencer/.../angles_0_0.png",
        "r2://long-term-storage/workflows/virtual-influencer/.../angles_0_1.png",
        "r2://long-term-storage/workflows/virtual-influencer/.../angles_0_2.png",
        "r2://long-term-storage/workflows/virtual-influencer/.../angles_0_3.png"
      ],
      "preferences": { "gender": "female", "race": "east-asian", "..." : "..." }
    },
    "generated_prompt": "Editorial-quality street photography of Mei, a meticulous architectural conservator, skateboarding through a juxtaposition of ancient and modern Shanghai...",
    "scene_candidates": [
      "[ancient-stone-temple] a secluded stone temple, hidden deep in the forest, ancient trees towering above...",
      "[ancient-stone-temple] an ancient stone temple, surrounded by overgrown jungle...",
      "[medieval-fair] Renaissance Fair, bustling fairground with tents and booths...",
      "..."
    ]
  }
}
```

### Option B — Workflow status endpoint (presigned HTTPS URLs included)

Use this when you want the influencer's images as ready-to-use HTTPS URLs (pre-signed, 2-hour expiry):

```bash
curl "https://api.pipelet.ai/v1/workflow/virtual-influencer-image-prompt/status/934291ae-1694-4382-96ce-0a87ba50a6d6" \
  -H "Authorization: Bearer <your-api-key>" \
  -H "x-user-id: user-abc123"
```

**Response when complete:**
```json
{
  "path": "complete",
  "status": { "status": "complete" },
  "output": {
    "influencer": {
      "id": "vi_3534e3bfc6c6_0",
      "name_english": "Mei",
      "name_ethnic": "Lin",
      "description": "...",
      "backstory": "...",
      "base_image_r2_uri": "r2://long-term-storage/...",
      "base_image_url": "https://long-term-storage.ae2b0858....r2.cloudflarestorage.com/...?X-Amz-Expires=7200&...",
      "angle_image_r2_uris": ["r2://...", "..."],
      "angle_image_urls": ["https://...", "..."],
      "preferences": { "..." : "..." }
    },
    "generated_prompt": "Editorial-quality street photography of Mei...",
    "scene_candidates": ["[ancient-stone-temple] ...", "..."]
  }
}
```

**Response while still running:**
```json
{
  "path": "in-progress",
  "status": { "status": "running" }
}
```

---

## Output field reference

### Top-level fields

| Field | Type | Description |
|---|---|---|
| `generated_prompt` | string | The text-to-image prompt. 200–300 words, photorealistic editorial style. Ready to pass directly to `z-image-turbo`, `nano-banana`, `qwen-text-to-image`, etc. |
| `scene_candidates` | string[] | The 0–8 raw scene descriptions fetched from `assets.pipelet.ai` and provided to the LLM as inspiration. Each item is prefixed with `[place-key]`. Useful for debugging or showing the user which scene was chosen. |
| `influencer` | object | The full influencer record (see below). |

### `influencer` object

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable influencer handle (e.g. `vi_3534e3bfc6c6_0`). |
| `name_english` | string | Character's English first name. |
| `name_ethnic` | string | Culturally appropriate alternate name. |
| `description` | string | Physical description of fixed features (face, eyes, nose, skin tone, hair). |
| `backstory` | string | RPG-style character bio used to inform the prompt's personality, mood, and expression. |
| `base_image_r2_uri` | string | Permanent `r2://` URI of the base portrait. |
| `base_image_url` | string \| undefined | Pre-signed HTTPS URL (only present via the workflow status endpoint). **Expires in 2 hours.** |
| `angle_image_r2_uris` | string[] | Permanent `r2://` URIs for the 4 multi-angle views. |
| `angle_image_urls` | string[] \| undefined | Pre-signed HTTPS URLs (only via the workflow status endpoint). **Expire in 2 hours.** |
| `preferences` | object | Echo of the original creation inputs (`gender`, `race`, `age`, etc.). |

---

## How the prompt is generated

1. **Visual input**: The LLM receives the influencer's base portrait + one angle image as image references.
2. **Character context**: Description and backstory are included in the user message to inform personality, mood, and expression.
3. **Scene candidates**: Up to 8 scene descriptions from `assets.pipelet.ai/location/place` (4 random locations × 2 scenes) are offered as inspiration.
4. **User scenario** (optional): If `prompt` was provided (e.g. `"skateboarding in the city"`), it is incorporated alongside the scene candidates.
5. **Composition guidance**: The system prompt instructs the LLM to allocate ~1/3 of words to environment/objects, ~1/3 to pose/action, and the rest to character appearance, lighting, and camera.
6. **Reasoning disabled**: The LLM is called with `enable_thinking: false` to avoid slow reasoning-model overhead.

---

## Prompt structure

The generated prompt follows this rough structure:

```
[Shot type + character action sentence]
[Character physical appearance: face, skin, hair, eyes]
[Clothing style reflecting character personality]

[Scene setting: location, background, objects, architecture]
[Lighting: direction, quality, color temperature]
[Camera: angle, lens, depth of field]
[Color palette and mood]
[Technical quality tags: resolution, grain, etc.]
```

**Example output** (influencer: Mei, prompt: "skateboarding in the city"):
> Editorial-quality street photography of Mei, a meticulous architectural conservator, skateboarding through a juxtaposition of ancient and modern Shanghai. Mei has a soft oval face, light porcelain skin, and dark brown almond-shaped eyes with a focused, introverted expression. Her straight, fine-textured dark brown hair is tied back in a practical low ponytail with a few loose strands framing her face.
>
> She is captured in a dynamic mid-action shot, balancing on a minimalist matte black skateboard on a weathered stone path that leads toward a secluded, ancient stone temple hidden within an urban pocket of greenery. She wears a contemporary, oversized charcoal grey linen blazer over a simple black cropped tank top and wide-leg black trousers. The lighting is the soft, diffused glow of a late overcast afternoon, casting gentle shadows that emphasize her refined nose bridge and smooth jawline. Low-angle wide shot, towering weathered grey stones of the temple and the distant blurred silhouette of futuristic skyscrapers. Slate grey, deep blacks, and muted moss-green. High-end cinematic grain, 8k resolution, 35mm lens, shallow depth of field.

---

## Downstream use cases

| Use case | How to use `generated_prompt` |
|---|---|
| Text-to-image generation | Pass `generated_prompt` directly as `prompt` to `z-image-turbo` or `nano-banana` queue |
| Iterative refinement | Re-call with a different `prompt` to get a new scene; the LLM will re-roll scene candidates each time |
| Batch variations | Submit 3–5 parallel jobs with the same `influencer_id` and different `prompt` values to get a variety of scenes |
| Prompt inspection | Check `scene_candidates` to see which locations were offered — useful for UX "scene picker" features |

---

## Notes

- **Auth**: Requires `user:write` scope and `x-user-id` header. The `influencer_id` must belong to the same `(org_id, user_id)` pair — you'll get a 500 error (influencer not found) if the ID doesn't match.
- **Pre-signed URL expiry**: `base_image_url` and `angle_image_urls` (from the workflow status endpoint) expire after **2 hours**. Re-call the status endpoint to get fresh URLs.
- **Scene candidates**: Fetched fresh on every call — different runs with the same inputs will typically get different scene candidates, producing varied prompts even without changing the `prompt` field.
- **`prompt` field is optional**: Without it, the LLM picks freely from the scene candidates. With it, the LLM blends your scenario into the chosen scene.
- **Relationship to `virtual-influencer-init`**: This endpoint is a downstream consumer of `virtual-influencer-init`. You must first create an influencer and note its `id`. The `id` is stable as long as the record isn't deleted.
