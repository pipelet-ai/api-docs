
## Video Generation API

- Production: `https://batch.pipelet.net`

## Endpoints (quick reference)

- Create request: `POST /queue/:modelId`
- Check status: `GET /queue/:modelId/requests/:requestId/status`
- Cancel request: `PUT /queue/:modelId/requests/:requestId/cancel`
- Fetch result: `GET /queue/:modelId/requests/:requestId`

## Supported modelIds

- Text-to-Video: `wan22-fast-t2v`, `wan22-fast-t2v-premium`, `wan22-fast-t2v-pro`
- Image-to-Video: `wan22-fast-i2v`, `wan22-fast-i2v-vip`, `wan22-fast-i2v-pro`

## Request Format

### 1) Create a request

POST `https://batch.pipelet.net/queue/:modelId`
Headers:
- `Content-Type: application/json`
- `Authorization: Bearer {your-bearer-token}`
Body:
* For text-to-video models, provide a `prompt` field with text prompts.
* For image-to-video models, provide a `prompt` and a `data_uri` field with base64 encoded image.
* Extra parameters are optional, `duration` for video length in seconds, `width` and `height` for video resolution.

Supported Resolution:
* For text-to-video models, 1280x720, 720x1280, 480x832, 832x480, 512x512, 768x768
* For image-to-video models, only `height` is used, and we only support 720, 480 as input
* For both models, maximum `duration` for 720p video is 5 seconds. For 480p video it can be 7 seconds.

Example (model `wan22-fast-i2v`):

```bash
curl 'https://batch.pipelet.net/queue/wan22-fast-i2v' \
  -H 'Content-Type: application/json' \
  -d '{"prompt": "A cat wearing a superman cape playing with a dog", "data_uri": "(base64 encoded image)", "duration": 5, "height": 720}'
```
NOTE: to create a base64 encoded image, do something like `base64 -i image.jpg`

Example response:

```json
{
  "request_id": "12",
  "response_url": "https://batch.pipelet.net/queue/wan22-fast-i2v/requests/12",
  "status_url": "https://batch.pipelet.net/queue/wan22-fast-i2v/requests/12/status",
  "cancel_url": "https://batch.pipelet.net/queue/wan22-fast-i2v/requests/12/cancel"
}
```

Notes:
- `request_id` is your handle for this job for future requests
- Use `status_url` to poll for progress.
- Use `response_url` to fetch the completed result.
- Use `cancel_url` to cancel if the job has not started yet.

## 2) Poll for status

GET `https://batch.pipelet.net/queue/:modelId/requests/:requestId/status`

Example:

```bash
curl https://batch.pipelet.net/queue/wan22-fast-i2v/requests/12/status
```

Possible responses:

- In progress

```json
{
  "status": "IN_PROGRESS",
  "statusMessages": {
    "message": "PROCESSING",
    "detail_message": "Progress: 4/24 Tasks done"
  },
  "response_url": "https://batch.pipelet.net/queue/wan22-fast-i2v/requests/12"
}
```
NOTE: Once a video starts generating, it usually takes 90 seconds to complete for 480p video. For 720p video it takes 180 seconds.

- In queue (not started yet). You will also see the current `queue_position`, starting at index 0: It means how many jobs are in front of you in the queue.

```json
{
  "status": "IN_QUEUE",
  "queue_position": 1,
  "statusMessages": {},
  "response_url": "https://batch.pipelet.net/queue/wan22-fast-i2v/requests/16"
}
```

- Other terminal states: `COMPLETED`, `FAILED`, `CANCELLED`.
NOTE: always check statusMessages for details.

### 2) Cancel a queued request

PUT `https://batch.pipelet.net/queue/:modelId/requests/:requestId/cancel`

- Right now it is only possible to cancel while status is IN_QUEUE (not started).

Example:

```bash
curl -XPUT https://batch.pipelet.net/queue/wan22-fast-i2v/requests/16/cancel
```

If successful, a 204 No Content response is returned; otherwise a conflict (409) is returned if it has already started.

### 3) Fetch the result

GET `https://batch.pipelet.net/queue/:modelId/requests/:requestId`

When the job `status` becomes `COMPLETED`, fetch the result from the `response_url`:

```bash
curl -XGET https://batch.pipelet.net/queue/wan22-fast-i2v/requests/13
```

Typical response body:

```json
{
  "video": {
    "data_uri": "(base64_encoded mp4 file content like 4AAQkZJRgABAxxx)"
  }
}
```

-  JSON`video.data_uri` is a base64-encoded string of an MP4 file encoded with H.264.
- Decode and save it locally to obtain the final `.mp4` file

Example to save locally (macOS/Linux):

```bash
curl -s https://batch.pipelet.net/queue/wan22-fast-i2v/requests/13 \
  | jq -r '.video.data_uri' | base64 --decode > output.mp4
```
