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
  }
}
```
