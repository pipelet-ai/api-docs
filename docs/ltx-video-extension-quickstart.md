# LTX23 Video Extension API Quickstart

Video extension takes an existing video (with audio) and seamlessly continues it — generating new frames and audio that follow the content and style of the input. It uses the LTX-2.3 model.

The input is a video file passed via `image_url` or `data_url`, the same field names used for image inputs elsewhere.

## Base URL

- Production: `https://batch.pipelet.net`

## Supported models

| Queue name | Description |
|---|---|
| `ltx23-video-extend` | Standard tier |
| `ltx23-video-extend-pro` | Pro tier — higher resolution outputs |
| `ltx23-video-extend-premium` | Premium tier — highest resolution outputs |

## Endpoints (quick reference)

- Create request: `POST /queue/:modelId`
- Check status: `GET /queue/:modelId/requests/:requestId/status`
- Cancel request: `PUT /queue/:modelId/requests/:requestId/cancel`
- Fetch result: `GET /queue/:modelId/requests/:requestId`

---

## 1) Create a request

POST `https://batch.pipelet.net/queue/:modelId`

### Request fields

| Field | Type | Required | Description |
|---|---|---|---|
| `image_url` | string | one of these | Public HTTPS URL or S3 pre-signed URL pointing to the input video |
| `data_url` | string | one of these | Base64-encoded input video |
| `prompt` | string | yes | Text describing what should happen in the extended portion |
| `min_height` | integer | no | Output resolution. Supported values: `480`, `720`, `1080`. Defaults to `720` |
| `duration` | number | no | How many seconds to extend the video. Maximum `20`. Defaults to `5` |
| `priority` | integer | no | Queue priority — higher numbers are processed first |

**Resolution notes:**
- `480` → 480p (e.g. 853×480 landscape, 480×853 portrait, 480×480 square)
- `720` → 720p (e.g. 1280×720 landscape, 720×1280 portrait, 720×720 square)
- `1080` → 1080p (e.g. 1920×1080 landscape, 1080×1920 portrait, 1080×1080 square) — maximum supported

The aspect ratio is automatically detected from the input video and snapped to the nearest standard ratio (16:9, 9:16, or 1:1).

This endpoint supports webhooks the same way as described in [Webhook Quickstart](./webhook-quick-start). Pass the webhook via query param `fal_webhook`.

### Example — extend by 5 seconds at 720p

```bash
curl 'https://batch.pipelet.net/queue/ltx23-video-extend' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer <api-key>' \
  -d '{
    "image_url": "https://resource.pipelet.net/demos/sample.mp4",
    "prompt": "The scene continues with smooth camera movement",
    "min_height": 720,
    "duration": 5
  }'
```

### Example — extend by 10 seconds at 1080p

```bash
curl 'https://batch.pipelet.net/queue/ltx23-video-extend-pro' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer <api-key>' \
  -d '{
    "image_url": "https://resource.pipelet.net/demos/sample.mp4",
    "prompt": "She continues speaking, gesturing naturally toward the camera",
    "min_height": 1080,
    "duration": 10
  }'
```

### Example response

```json
{
  "request_id": "42",
  "response_url": "https://batch.pipelet.net/queue/ltx23-video-extend/requests/42",
  "status_url": "https://batch.pipelet.net/queue/ltx23-video-extend/requests/42/status",
  "cancel_url": "https://batch.pipelet.net/queue/ltx23-video-extend/requests/42/cancel"
}
```

---

## 2) Poll for status

GET `https://batch.pipelet.net/queue/:modelId/requests/:requestId/status`

```bash
curl 'https://batch.pipelet.net/queue/ltx23-video-extend/requests/42/status' \
  -H 'Authorization: Bearer <api-key>'
```

Possible terminal states: `COMPLETED`, `FAILED`, `CANCELLED`.

---

## 2.1) Cancel a queued request

PUT `https://batch.pipelet.net/queue/:modelId/requests/:requestId/cancel`

Only possible while status is `IN_QUEUE` (not yet started).

```bash
curl -XPUT 'https://batch.pipelet.net/queue/ltx23-video-extend/requests/42/cancel' \
  -H 'Authorization: Bearer <api-key>'
```

---

## 3) Fetch the result

GET `https://batch.pipelet.net/queue/:modelId/requests/:requestId`

When status is `COMPLETED`, fetch the result from the `response_url`:

```bash
curl 'https://batch.pipelet.net/queue/ltx23-video-extend/requests/42' \
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
