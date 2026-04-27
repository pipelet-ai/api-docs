---
title: Virtual Influencer Init Quickstart
permalink: /docs/virtual-influencer-init/
---

# Virtual Influencer Init API

The `virtual-influencer-init` queue generates a pair of AI virtual influencer characters from a text description. For each character it produces:

1. **Two base portrait images** (1440×1920) via the `z-image-human-base` GPU pipeline
2. **Four multi-angle views** per character (front-left, front-right, left side, right side) via Qwen image editing
3. **AI-generated identity** — English first name, ethnic/cultural name, physical description, and an RPG-style character backstory — via a vision LLM analyzing the base portrait

All images are stored in long-term R2 storage (pre-signed URLs auto-refresh on every status/CRUD query). Records are attributed to the calling user via the required `x-user-id` header and are accessible via the CRUD endpoints below.

**Typical end-to-end time: 8–15 minutes** (GPU queue depth dependent).

## Base URL

- Production: `https://api.pipelet.ai`

## Endpoints (quick reference)

**Queue (submit + poll):**
- Submit job: `POST /fal/queue/virtual-influencer-init`
- Check status: `GET /fal/queue/virtual-influencer-init/requests/:requestId/status`
- Cancel: `PUT /fal/queue/virtual-influencer-init/requests/:requestId/cancel`
- Fetch result: `GET /fal/queue/virtual-influencer-init/requests/:requestId`

**Workflow status (richer partial progress):**
- `GET /v1/workflow/virtual-influencer/status/:instanceId`

**CRUD (manage saved records):**
- List influencers: `GET /v1/influencers`
- Get influencer: `GET /v1/influencers/:id`
- Delete influencer: `DELETE /v1/influencers/:id`

---

## Required Headers

| Header | Required | Description |
|---|---|---|
| `Authorization` | Yes | `Bearer <your-api-key>` — requires `user:write` scope |
| `x-user-id` | Yes | Your application's user identifier. Records are scoped by `(org, user_id)`. |
| `Content-Type` | Yes | `application/json` |

---

## 1) Submit a Job

`POST /fal/queue/virtual-influencer-init`

**Request body** (JSON):

| Field | Type | Required | Description |
|---|---|---|---|
| `gender` | string | No | e.g. `"female"`, `"male"`, `"non-binary"` |
| `race` | string | No | e.g. `"Asian"`, `"Black"`, `"White"`, `"Hispanic"` |
| `age` | string or number | No | e.g. `"25"`, `"30-35"` |
| `body_type` | string | No | e.g. `"slim"`, `"athletic"`, `"curvy"` |
| `additional_traits` | string | No | Free-form traits, e.g. `"freckles, green eyes"` |
| `style` | string | No | Visual style hint, e.g. `"instagram fashion"`, `"editorial"` |

```bash
curl -XPOST "https://api.pipelet.ai/fal/queue/virtual-influencer-init" \
  -H "Authorization: Bearer <your-api-key>" \
  -H "x-user-id: user-abc123" \
  -H "Content-Type: application/json" \
  -d '{
    "gender": "female",
    "race": "Asian",
    "age": "25",
    "body_type": "slim",
    "additional_traits": "freckles, green eyes",
    "style": "instagram fashion"
  }'
```

**Response:**
```json
{
  "request_id": "bf7ba1be-ee0e-4da8-93b4-56734d1e8454",
  "response_url": "https://api.pipelet.ai/fal/queue/virtual-influencer-init/requests/bf7ba1be-ee0e-4da8-93b4-56734d1e8454",
  "status_url": "https://api.pipelet.ai/fal/queue/virtual-influencer-init/requests/bf7ba1be-ee0e-4da8-93b4-56734d1e8454/status",
  "cancel_url": "https://api.pipelet.ai/fal/queue/virtual-influencer-init/requests/bf7ba1be-ee0e-4da8-93b4-56734d1e8454/cancel"
}
```

The `request_id` is also the Cloudflare Workflow `instanceId` — you can use it with the workflow status endpoint for richer partial-progress data.

---

## 2) Poll for Status

```bash
curl "https://api.pipelet.ai/fal/queue/virtual-influencer-init/requests/bf7ba1be-ee0e-4da8-93b4-56734d1e8454/status" \
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
  "response_url": "https://api.pipelet.ai/fal/queue/virtual-influencer-init/requests/bf7ba1be-ee0e-4da8-93b4-56734d1e8454"
}
```

Poll every 30 seconds. The job moves through `IN_QUEUE` → `IN_PROGRESS` → `COMPLETED` (or `FAILED`).

---

## 3) Fetch Result

Once status is `COMPLETED`:

```bash
curl "https://api.pipelet.ai/fal/queue/virtual-influencer-init/requests/bf7ba1be-ee0e-4da8-93b4-56734d1e8454" \
  -H "Authorization: Bearer <your-api-key>" \
  -H "x-user-id: user-abc123"
```

**Response:**
```json
{
  "influencers": [
    {
      "id": "vi_bf7ba1beee0e_0",
      "name_english": "Mei",
      "name_ethnic": "美衣",
      "description": "This person has almond-shaped dark-brown eyes with full arched brows, a straight medium-height nose, soft cheekbones, and an oval face with warm ivory skin tone. Their natural hair is thick and straight with a dark-brown hue and fine texture.",
      "backstory": "Born in Osaka to a textile designer and a high school art teacher, Mei grew up sketching fashion plates before she could read. She studied communication design in Tokyo and relocated to Seoul at 22 for an internship at a K-beauty brand, where she discovered her passion for content creation. Quiet but fiercely intentional, she curates her feed like a gallery — every post a mood board. She keeps a paper journal, collects vintage film cameras, and makes matcha lattes at precisely 7 AM. Her younger sister lives in Vancouver; they FaceTime every Sunday. Mei's long-term goal is to launch a slow-fashion line rooted in Japanese wabi-sabi aesthetics.",
      "base_image_url": "https://long-term-storage.r2.cloudflarestorage.com/workflows/virtual-influencer/bf7ba1be-ee0e-4da8-93b4-56734d1e8454/base_0.png?X-Amz-...",
      "base_image_r2_uri": "r2://long-term-storage/workflows/virtual-influencer/bf7ba1be-ee0e-4da8-93b4-56734d1e8454/base_0.png",
      "angle_image_urls": [
        "https://long-term-storage.r2.cloudflarestorage.com/workflows/virtual-influencer/bf7ba1be-ee0e-4da8-93b4-56734d1e8454/angles_0_0.png?X-Amz-...",
        "https://long-term-storage.r2.cloudflarestorage.com/workflows/virtual-influencer/bf7ba1be-ee0e-4da8-93b4-56734d1e8454/angles_0_1.png?X-Amz-...",
        "https://long-term-storage.r2.cloudflarestorage.com/workflows/virtual-influencer/bf7ba1be-ee0e-4da8-93b4-56734d1e8454/angles_0_2.png?X-Amz-...",
        "https://long-term-storage.r2.cloudflarestorage.com/workflows/virtual-influencer/bf7ba1be-ee0e-4da8-93b4-56734d1e8454/angles_0_3.png?X-Amz-..."
      ],
      "angle_image_r2_uris": [
        "r2://long-term-storage/workflows/virtual-influencer/bf7ba1be-ee0e-4da8-93b4-56734d1e8454/angles_0_0.png",
        "r2://long-term-storage/workflows/virtual-influencer/bf7ba1be-ee0e-4da8-93b4-56734d1e8454/angles_0_1.png",
        "r2://long-term-storage/workflows/virtual-influencer/bf7ba1be-ee0e-4da8-93b4-56734d1e8454/angles_0_2.png",
        "r2://long-term-storage/workflows/virtual-influencer/bf7ba1be-ee0e-4da8-93b4-56734d1e8454/angles_0_3.png"
      ],
      "all_image_r2_uris": [
        "r2://long-term-storage/workflows/virtual-influencer/bf7ba1be-ee0e-4da8-93b4-56734d1e8454/base_0.png",
        "r2://long-term-storage/workflows/virtual-influencer/bf7ba1be-ee0e-4da8-93b4-56734d1e8454/angles_0_0.png",
        "r2://long-term-storage/workflows/virtual-influencer/bf7ba1be-ee0e-4da8-93b4-56734d1e8454/angles_0_1.png",
        "r2://long-term-storage/workflows/virtual-influencer/bf7ba1be-ee0e-4da8-93b4-56734d1e8454/angles_0_2.png",
        "r2://long-term-storage/workflows/virtual-influencer/bf7ba1be-ee0e-4da8-93b4-56734d1e8454/angles_0_3.png"
      ],
      "preferences": {
        "gender": "female",
        "race": "Asian",
        "age": "25",
        "body_type": "slim",
        "additional_traits": "freckles, green eyes",
        "style": "instagram fashion"
      }
    },
    {
      "id": "vi_bf7ba1beee0e_1",
      "name_english": "Sara",
      "name_ethnic": "沙羅",
      "description": "...",
      "backstory": "...",
      "base_image_url": "https://...",
      "base_image_r2_uri": "r2://...",
      "angle_image_urls": ["https://...", "..."],
      "angle_image_r2_uris": ["r2://...", "..."],
      "all_image_r2_uris": ["r2://...", "..."],
      "preferences": { "gender": "female", "..." : "..." }
    }
  ]
}
```

### Output field reference

| Field | Type | Description |
|---|---|---|
| `id` | string | Stable influencer handle. Format: `vi_<workflowInstanceId12chars>_<0\|1>`. Use this to fetch/delete via CRUD endpoints. |
| `name_english` | string | Invented English first name for the character. |
| `name_ethnic` | string | Culturally appropriate alternate name reflecting the character's background. |
| `description` | string | 1–2 sentence physical description of fixed features (eye color, nose, bone structure, skin tone, natural hair). Does not include clothing or makeup. |
| `backstory` | string | RPG-style character bio (≤200 words): career, personality, family, hobbies, favorites, and a long-term dream. Generated at temperature 0.9 for creative variety. |
| `base_image_url` | string | Pre-signed HTTPS URL for the primary portrait image (1440×1920). **Expires in 2 hours.** Re-fetch via CRUD to refresh. |
| `base_image_r2_uri` | string | Permanent `r2://` URI for the base portrait. Use this as a stable reference for downstream workflows. |
| `angle_image_urls` | string[] | Pre-signed HTTPS URLs for the 4 multi-angle views in order: front-left quarter, front-right quarter, left side, right side. **Expire in 2 hours.** |
| `angle_image_r2_uris` | string[] | Permanent `r2://` URIs for the 4 angle views. |
| `all_image_r2_uris` | string[] | Permanent `r2://` URIs for all 5 images (base + 4 angles). Convenient for passing the full set to downstream image tasks. |
| `preferences` | object | Echo of the input fields you submitted (`gender`, `race`, `age`, `body_type`, `additional_traits`, `style`). |

**Image angle order** (consistent across `angle_image_urls` and `angle_image_r2_uris`):

| Index | View |
|---|---|
| 0 | Front-left quarter view |
| 1 | Front-right quarter view |
| 2 | Left side view |
| 3 | Right side view |

---

## 4) Workflow Status (partial progress)

For richer in-flight progress, use the workflow status endpoint. It returns partial results as soon as each pipeline step saves to D1 — you can display the base portrait before the angle images are ready.

```bash
curl "https://api.pipelet.ai/v1/workflow/virtual-influencer/status/bf7ba1be-ee0e-4da8-93b4-56734d1e8454" \
  -H "Authorization: Bearer <your-api-key>" \
  -H "x-user-id: user-abc123"
```

**Response while running (with partial data):**
```json
{
  "path": "in-progress",
  "status": { "status": "running" },
  "partial_output": {
    "influencers": [
      {
        "id": "vi_bf7ba1beee0e_0",
        "status": "creating",
        "base_image_url": "https://...",
        "angle_image_urls": [],
        "name_english": null,
        "backstory": null,
        "..."
      }
    ]
  },
  "steps_completed": {
    "vi_bf7ba1beee0e_0": ["base_image"],
    "vi_bf7ba1beee0e_1": ["base_image"]
  }
}
```

**`steps_completed` values per influencer:**

| Step value | Meaning |
|---|---|
| `base_image` | Base portrait generated and saved |
| `angle_images` | All 4 multi-angle views generated and saved |
| `description` | Vision LLM ran; name + description + backstory are populated |
| `completed` | All steps done, record status = `completed` |

**Response when complete:**
```json
{
  "path": "complete",
  "status": { "status": "complete" },
  "output": {
    "influencers": [ { "..." : "..." } ]
  }
}
```

---

## 5) CRUD Endpoints

Records are persisted in D1 and scoped by `(org_id, user_id)`. Passing `x-user-id` filters results to that user; omitting it returns all records for the org.

### List influencers

```bash
curl "https://api.pipelet.ai/v1/influencers?limit=20&offset=0" \
  -H "Authorization: Bearer <your-api-key>" \
  -H "x-user-id: user-abc123"
```

**Query params:** `limit` (1–100, default 20), `offset` (default 0).

**Response:**
```json
{
  "influencers": [
    {
      "id": "vi_bf7ba1beee0e_0",
      "status": "completed",
      "user_id": "user-abc123",
      "name_english": "Mei",
      "name_ethnic": "美衣",
      "description": "...",
      "backstory": "...",
      "base_image_url": "https://...",
      "base_image_r2_uri": "r2://...",
      "angle_image_urls": ["https://...", "...", "...", "..."],
      "angle_image_r2_uris": ["r2://...", "...", "...", "..."],
      "all_image_urls": ["https://...", "...", "...", "...", "..."],
      "all_image_r2_uris": ["r2://...", "...", "...", "...", "..."],
      "preferences": { "gender": "female", "..." : "..." },
      "created_at": "2026-04-19T08:23:11Z",
      "updated_at": "2026-04-19T08:35:42Z"
    }
  ],
  "limit": 20,
  "offset": 0
}
```

### Get a single influencer

```bash
curl "https://api.pipelet.ai/v1/influencers/vi_bf7ba1beee0e_0" \
  -H "Authorization: Bearer <your-api-key>" \
  -H "x-user-id: user-abc123"
```

Returns the same object shape as a single element from the list response. Returns `404` if the record doesn't exist or belongs to a different user.

### Delete an influencer

```bash
curl -XDELETE "https://api.pipelet.ai/v1/influencers/vi_bf7ba1beee0e_0" \
  -H "Authorization: Bearer <your-api-key>" \
  -H "x-user-id: user-abc123"
```

**Response:**
```json
{ "deleted": true, "id": "vi_bf7ba1beee0e_0" }
```

Returns `404` if the record doesn't exist or belongs to a different user. Note: this deletes the D1 record only — R2 images are retained.

---

## Downstream use cases

The output fields are designed to feed directly into other Pipelet workflows:

| Use case | Fields to use |
|---|---|
| Generate content images (product shots, lifestyle scenes) | `base_image_r2_uri` or `angle_image_r2_uris[n]` as the subject image |
| Multi-angle consistency check | `all_image_r2_uris` — all 5 images share consistent lighting and identity |
| Character-driven TTS / video | `name_english`, `backstory` → derive script voice and personality |
| Image-to-video (lip sync, walking loop) | `base_image_r2_uri` → `wan22-s2v-extreme-length` queue |
| Prompt enrichment for scene generation | `description` → embed into image generation prompt to lock physical features |
| Face-swap into scenes | `base_image_r2_uri` → face-swap pipeline |

---

## Notes

- **Pre-signed URL expiry**: All `*_url` fields (not `*_r2_uri`) expire after **2 hours**. To get fresh URLs, re-call `GET /v1/influencers/:id` or the workflow status endpoint — URLs are regenerated on every response.
- **Two influencers per job**: Each submission always returns exactly 2 characters. If the GPU pipeline returns only 1 base image, both influencers use it as their starting point (rare fallback).
- **Backstory creativity**: The vision LLM runs at temperature 0.9, so names and backstories will vary significantly between runs even for similar physical inputs.
- **Idempotent IDs**: The influencer `id` is derived from the Cloudflare Workflow instance ID, so it is stable and unique per job. You can use it immediately for tracking even while the job is still running.
- **Auth scope**: Submission requires `user:write`. CRUD reads (`GET /v1/influencers`) require `user:read`. Delete requires `user:write`.
