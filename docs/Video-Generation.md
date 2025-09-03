# Video Generation API

This document describes the video generation endpoints exposed by `batch/src/routes/fal-routes.ts`.

Use these endpoints to create text-to-video jobs, poll their status, cancel if still queued, and fetch the resulting video.

Note: All routes require an authenticated request (middleware `requireUser`).

## Base URL

- Production: `https://batch.pipelet.net`

## Supported models

- Text-to-Video: `wan22-fast-t2v`, `wan22-fast-t2v-pro`, `wan22-fast-t2v-premium`, `wan22-t2v-fp8` Takes a prompt as input and generates a video.
- Image-to-Video: `wan22-fast-i2v`, `wan22-fast-i2v-pro`, `wan22-fast-i2v-premium`, `wan22-i2v-fp8` Takes an image and a prompt as input and generates a video.
- Sound-Text-to-Video: `wan22-s2v-fast`, `wan22-s2v-pro`, `wan22-s2v-premium` Takes an image, a prompt and an audio as input and generates a video.

## Endpoints (quick reference)

- Create request: `POST /queue/:modelId`
- Check status: `GET /queue/:modelId/requests/:requestId/status`
- Cancel request: `PUT /queue/:modelId/requests/:requestId/cancel`
- Fetch result: `GET /queue/:modelId/requests/:requestId`

Excluded from this doc: `POST /queue/:modelId/:subpath`

---

## 1) Create a request

POST `https://batch.pipelet.net/queue/:modelId`

Body: JSON inputs for the model. For basic T2V, at minimum provide a `prompt`.
- For image-to-video, provide a `data_uri` field as the base64 encoded image.
- `width` and `height` for video resolution.
- `duration` for video length in seconds.
- `new_resolution` for upscaling an image to a higher resolution.
- `min_height` for image to video, to ensure the video is at least this height
- `priority` for priority in the queue, the bigger the number, the higher priority we have

Example (model `wan22-fast-t2v`):

```bash
curl 'https://batch.pipelet.net/queue/wan22-fast-t2v' \
  -H 'Content-Type: application/json' \
  -d '{"prompt": "A cat wearing a superman cape playing with a dog", "width": 1280, "height": 720, "duration": 5, "priority": 100}'
```

Example response:

```json
{
  "request_id": "12",
  "response_url": "https://batch.pipelet.net/queue/wan22-fast-t2v/requests/12",
  "status_url": "https://batch.pipelet.net/queue/wan22-fast-t2v/requests/12/status",
  "cancel_url": "https://batch.pipelet.net/queue/wan22-fast-t2v/requests/12/cancel"
}
```

Notes:
- `request_id` is your handle for this job.
- Use `status_url` to poll for progress.
- Use `response_url` to fetch the completed result.
- Use `cancel_url` to cancel if the job has not started yet.

## 1.1) Supported Models and Request Formats

### Text To Video Models

`wan22-fast-t2v-pro`, `wan22-fast-t2v`, `wan22-fast-t2v-premium`, `wan22-fast-t2v-fp8`

These two are the same wan22 fast text-to-video model, but `wan22-fast-t2v-pro` has a higher priority in the queue.
It usually takes 3 minutes to generate a 5 second video.

```json
{
  "prompt": "A cat wearing a superman cape playing with a dog",
  "width": 1280,
  "height": 720,
  "duration": 5,
  "priority": 30
}
```

### Image To Video Models

`wan22-fast-i2v-pro`, `wan22-fast-i2v`, `wan22-fast-i2v-premium`, `wan22-fast-i2v-fp8`

These two are the same wan22 fast image-to-video model, but `wan22-fast-i2v-pro` has a higher priority in the queue.
It usually takes 2 minutes to generate a 7 second video.

```json
{
  "prompt": "The man does a backflip",
  "data_uri": "(base64 encoded image)",
  "duration": 5,
  "min_height": 480,
  "priority": 30
}
```

### Sound and Text To Video Models

`wan22-s2v-pro`, `wan22-s2v-fast`, `wan22-s2v-premium`

These two are the same wan22 fast image-to-video model, but `wan22-s2v-pro` has a higher priority in the queue.

```json
{
  "prompt": "The girl sings a sad song with tears and emotion",
  "data_uri": "(base64 encoded image)",
  "audio_uri": "(base64 encoded audio)",
  "duration": 5,
  "min_height": 480,
  "priority": 30
}
```


## 2) Poll for status

GET `https://batch.pipelet.net/queue/:modelId/requests/:requestId/status`

Example:

```bash
curl https://batch.pipelet.net/queue/wan22-fast-i2v/requests/12/status
```

Possible responses:

- In progress
When it is IN_PROGRESS, you will also see the current `estimated_total_time_seconds`. It is for the whole job time, not just the remaining time.
```json
{
  "status": "IN_PROGRESS",
  "statusMessages": {
    "message": "PROCESSING",
    "detail_message": "Progress: 4/24 Tasks done",
    "estimated_total_time_seconds": 120
  },
  "response_url": "https://batch.pipelet.net/queue/wan22-fast-i2v/requests/12"
}
```

- In queue (not started yet). You will also see the current `queue_position`, starting at index 0. Below example, `queue_position` is 1, meaning there is one job in front of you in the queue.

```json
{
  "status": "IN_QUEUE",
  "queue_position": 1,
  "statusMessages": {},
  "response_url": "https://batch.pipelet.net/queue/wan22-fast-i2v/requests/16"
}
```

- Other terminal states: `COMPLETED`, `FAILED`, `CANCELLED`.

## 2.1) Cancel a queued request

PUT `https://batch.pipelet.net/queue/:modelId/requests/:requestId/cancel`

- Right now it is only possible to cancel while status is IN_QUEUE (not started).

Example:

```bash
curl -XPUT https://batch.pipelet.net/queue/wan22-fast-i2v/requests/16/cancel
```

If successful, a 204 No Content response is returned; otherwise a conflict (409) is returned if it has already started.

## 3) Fetch the result

GET `https://batch.pipelet.net/queue/:modelId/requests/:requestId`

When the job `status` becomes `COMPLETED`, fetch the result from the `response_url`:

```bash
curl -XGET https://batch.pipelet.net/queue/wan22-fast-i2v/requests/13
```

Typical response body:

```json
{
  "video": {
    "data_uri": "4AAQSkZJRgABAxxx(base64_encoded mp4 file content)"
  }
}
```

- `video.data_uri` is a base64-encoded string of an MP4 file encoded with H.264.
- Decode and save it locally to obtain the final `.mp4`.

Example to save locally (macOS/Linux):

```bash
curl -s https://batch.pipelet.net/queue/wan22-fast-i2v/requests/13 \
  | jq -r '.video.data_uri' | base64 --decode > output.mp4
```

## Request/Response Details

- Authentication: All endpoints require "Authentication: Bearer {your-bearer-token}" header.
- Inputs: Arbitrary JSON accepted and forwarded to the model. If `data_uri` is provided, it will be stored and transformed to an internal URL before enqueueing.
- Status mapping: Internal job states map to the following external states
  - `PENDING`/`RETRY` -> `IN_QUEUE`
  - `RUNNING` -> `IN_PROGRESS`
  - `COMPLETED` -> `COMPLETED`
  - `FAILED` -> `FAILED`
  - `CANCELLED` -> `CANCELLED`

## Error cases

- 400: Invalid `requestId` format.
- 404: Unknown request.
- 409: Cannot cancel (already started or finished).

## Notes

- This document intentionally excludes the subpath endpoint `POST /queue/:modelId/:subpath`.
- Result fetching attempts to inline the stored payload and return the uniform shape `{ "video": { "data_uri": "..." } }`.
