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

- If `data_uri` or `image_uri` is provided, the request is treated as an image-to-video task.
- If neither `data_uri` nor `image_uri` is provided, the request is treated as a text-to-video task.

Fields:

- `prompt` (string, required) text prompt describing the video.
- `data_uri` (string, optional) input image for image-to-video.
- `image_uri` (string, optional) input image for image-to-video.
- `min_height` (number, optional) output resolution selector. Only `720` and `1080` are supported.
- `duration` (number, required) video length in seconds.

Notes:

- `duration` is in seconds.
- Only `720` and `1080` are supported as output resolutions (via `min_height`).
- Mode selection: The model will automatically select the mode (image-to-video or text-to-video) based on the presence of `data_uri` or `image_uri`.

Example (image-to-video, model `ltx23-i2v-t2v-pro`):

```bash
curl 'https://batch.pipelet.net/queue/ltx23-i2v-t2v-pro' \
  -H 'Content-Type: application/json' \
  -H 'x-user-id: xy728e9er' \
  -H 'Authorization: Bearer <api-key>' \
  -d '{"data_uri":"https://resource.pipelet.net/images/step_on.jpg", "prompt":"The girl stands up, and steps on the fish and walks off saying \"Enough of this. Lets go\". The fish flaps and jumps lifelessly, blood spattered on the ground", "min_height": 720, "duration": 10}'
```

Example (text-to-video, model `ltx23-i2v-t2v-pro`):

```bash
curl 'https://batch.pipelet.net/queue/ltx23-i2v-t2v-pro' \
  -H 'Content-Type: application/json' \
  -H 'x-user-id: xy728e9er' \
  -H 'Authorization: Bearer <api-key>' \
  -d '{"prompt":"A cinematic shot of a robot chef plating ramen in a neon-lit kitchen", "min_height": 720, "duration": 10}'
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
