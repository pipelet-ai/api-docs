# LTX23 Image/Text to Video API

This document describes the combined image-to-video and text-to-video generation endpoints exposed by `batch/src/routes/fal-routes.ts`.

Use these endpoints to create jobs, poll their status, cancel if still queued, and fetch the resulting video.

Note: All routes require an authenticated request (middleware `requireUser`).

## Base URL

- Production: `https://batch.pipelet.net`

## Supported models

- `ltx23-i2v-t2v-pro`
- `ltx23-i2v-t2v-premium`

These models support both text-to-video and image-to-video requests.

## Endpoints (quick reference)

- Create request: `POST /queue/:modelId`
- Check status: `GET /queue/:modelId/requests/:requestId/status`
- Cancel request: `PUT /queue/:modelId/requests/:requestId/cancel`
- Fetch result: `GET /queue/:modelId/requests/:requestId`

## Webhook notification

If you want to receive notifications when the job status changes (e.g. `acquired`, `completed`, `failed`), you can attach a webhook to this request.

See: [Webhook Quickstart](./webhook-quick-start)

---

## 1) Create a request

POST `https://batch.pipelet.net/queue/:modelId`

Body: JSON inputs for the model.

- Mode selection:
  - If any image input field is provided and non-empty (for example `data_uri`, `image_uri`, `second_image_uri`), the request is treated as an image-to-video task.
  - If all image input fields are missing or empty (for example `data_uri: ""`), the request is treated as a text-to-video task.

Notes:

- `duration` is in seconds.
- Only 720p and 1080p outputs are supported.

## 1.1) Text To Video request format {#text-to-video}

The request body fields are the same as the Text To Video format in [Video Generation API](./Video-Generation#1-1-supported-models-and-request-formats).

Fields:

- `prompt` (string, required) text prompt describing the video.
- `width` (number, required) output video width.
- `height` (number, required) output video height.
- `duration` (number, required) video length in seconds.
- `priority` (number, optional) priority in the queue, the bigger the number, the higher priority we have.
- `higher_quality_with_more_steps` (boolean, optional) generate with more steps for higher quality.

```json
{
  "prompt": "A cat wearing a superman cape playing with a dog",
  "width": 1280,
  "height": 720,
  "duration": 5,
  "priority": 30,
  "higher_quality_with_more_steps": false
}
```

Notes:

- Only `width`/`height` values corresponding to 720p or 1080p are supported.

Example (text-to-video, model `ltx23-i2v-t2v-pro`):

```bash
curl 'https://batch.pipelet.net/queue/ltx23-i2v-t2v-pro' \
  -H 'Content-Type: application/json' \
  -H 'x-user-id: xy728e9er' \
  -H 'Authorization: Bearer <api-key>' \
  -d '{"prompt":"A cinematic shot of a robot chef plating ramen in a neon-lit kitchen", "width": 1280, "height": 720, "duration": 10}'
```

## 1.2) Image To Video request format {#image-to-video}

```json
{
  "prompt": "The man does a backflip",
  "data_uri": "(base64 encoded image or URL)",
  "duration": 5,
  "min_height": 720
}
```

Example (image-to-video, model `ltx23-i2v-t2v-pro`):

```bash
curl 'https://batch.pipelet.net/queue/ltx23-i2v-t2v-pro' \
  -H 'Content-Type: application/json' \
  -H 'x-user-id: xy728e9er' \
  -H 'Authorization: Bearer <api-key>' \
  -d '{"data_uri":"https://resource.pipelet.net/images/step_on.jpg", "prompt":"The girl stands up, and steps on the fish and walks off saying \"Enough of this. Lets go\". The fish flaps and jumps lifelessly, blood spattered on the ground", "min_height": 720, "duration": 10}'
```

## 1.3) Multi-Frame Guidance (Middle Frames + Last Frame) {#multi-frame-guidance}

You can provide additional frame guidance beyond the initial `data_uri` to control the video generation at specific timestamps. The model supports up to 3 middle frames and 1 last frame.

Optional fields:

- `middle_frame1_url` (string, optional) - base64 encoded image or URL for the first middle frame.
- `middle_frame2_url` (string, optional) - base64 encoded image or URL for the second middle frame.
- `middle_frame3_url` (string, optional) - base64 encoded image or URL for the third middle frame.
- `last_frame_url` (string, optional) - base64 encoded image or URL for the last frame.

**Frame spacing:** The frames are spaced evenly across the video duration. For example, in a 10-second video with `data_uri` (first frame at 0s), `middle_frame1_url`, and `last_frame_url` (at 10s), the `middle_frame1_url` will be placed at the center (5 seconds).

Example (10-second video with first frame, one middle frame, and last frame):

```json
{
  "prompt": "A smooth transition from day to night in a city",
  "data_uri": "(base64 encoded image or URL - first frame at 0s)",
  "middle_frame1_url": "(base64 encoded image or URL - placed at 5s)",
  "last_frame_url": "(base64 encoded image or URL - last frame at 10s)",
  "duration": 10,
  "min_height": 720
}
```

```bash
curl 'https://batch.pipelet.net/queue/ltx23-i2v-t2v-pro' \
  -H 'Content-Type: application/json' \
  -H 'x-user-id: xy728e9er' \
  -H 'Authorization: Bearer <api-key>' \
  -d '{"data_uri":"https://resource.pipelet.net/images/day.jpg", "middle_frame1_url":"https://resource.pipelet.net/images/sunset.jpg", "last_frame_url":"https://resource.pipelet.net/images/night.jpg", "prompt":"A smooth transition from day to night in a city", "min_height": 720, "duration": 10}'
```

Example response:

```json
{
  "request_id": "12",
  "response_url": "https://batch.pipelet.net/queue/ltx23-i2v-t2v-pro/requests/12",
  "status_url": "https://batch.pipelet.net/queue/ltx23-i2v-t2v-pro/requests/12/status",
  "cancel_url": "https://batch.pipelet.net/queue/ltx23-i2v-t2v-pro/requests/12/cancel"
}
```

## 2) Poll for status

GET `https://batch.pipelet.net/queue/:modelId/requests/:requestId/status`

Example:

```bash
curl 'https://batch.pipelet.net/queue/ltx23-i2v-t2v-pro/requests/12/status' \
  -H 'Authorization: Bearer <api-key>'
```

Possible terminal states: `COMPLETED`, `FAILED`, `CANCELLED`.

## 2.1) Cancel a queued request

PUT `https://batch.pipelet.net/queue/:modelId/requests/:requestId/cancel`

- Right now it is only possible to cancel while status is IN_QUEUE (not started).

Example:

```bash
curl -XPUT 'https://batch.pipelet.net/queue/ltx23-i2v-t2v-pro/requests/12/cancel' \
  -H 'Authorization: Bearer <api-key>'
```

## 3) Fetch the result

GET `https://batch.pipelet.net/queue/:modelId/requests/:requestId`

When the job `status` becomes `COMPLETED`, fetch the result from the `response_url`:

```bash
curl 'https://batch.pipelet.net/queue/ltx23-i2v-t2v-pro/requests/12' \
  -H 'Authorization: Bearer <api-key>'
```

Typical response body:

```json
{
  "video": {
    "data_uri": "4AAQSkZJRgABAxxx(base64_encoded mp4 file content)"
  },
  "preview_fps": 1,
  "previews": [
    {
      "url": "https://prod-batch-files.<account>.r2.cloudflarestorage.com/outputs/premium/452675_0_preview_000.jpg?X-Amz-Expires=14400&X-Amz-...",
      "index": 0,
      "timestamp": 0,
      "width": 640,
      "height": 366,
      "content_type": "image/jpeg"
    },
    {
      "url": "https://prod-batch-files.<account>.r2.cloudflarestorage.com/outputs/premium/452675_0_preview_001.jpg?X-Amz-Expires=14400&X-Amz-...",
      "index": 1,
      "timestamp": 1,
      "width": 640,
      "height": 366,
      "content_type": "image/jpeg"
    }
  ]
}
```

## 3.1) Preview frames {#ltx23-previews}

Alongside the video, the result carries `previews`: a strip of JPEG frames sampled
from the finished video, one per second by default. Use them for a thumbnail, a
poster frame, or a scrub strip without downloading the whole MP4 first.

- `previews` (array, ordered by `timestamp`) — each entry is one frame.
  - `url` — a pre-signed HTTPS URL for the JPEG. Fetch it directly; no auth header needed.
  - `index` — 0-based position in the strip.
  - `timestamp` — seconds into the video. `previews[0]` is the first frame, at `0`.
  - `width` / `height` — pixels of the JPEG. Frames are downscaled to 640px wide at
    most, so the aspect ratio matches your video but the size does not.
  - `content_type` — always `image/jpeg` today.
- `preview_fps` (number) — the sampling rate, so you can label the strip without
  dividing timestamps. `1` means one frame per second.

**Treat `previews` as optional.** It is omitted entirely — not sent as `null` or `[]`
— when the job produced no frames, so read it defensively:

```bash
curl -s 'https://batch.pipelet.net/queue/ltx23-i2v-t2v-pro/requests/12' \
  -H 'Authorization: Bearer <api-key>' \
  | jq -r '.previews // [] | .[0].url'
```

The `url` values are re-signed every time you fetch the result, so a link you just
received is always live. Links you stored earlier will expire — re-fetch the result
rather than caching the URLs. If you need the frames long-term, download the JPEGs.
