# Video Face Swap API

This document describes the video face swap endpoints in the `https://api.pipelet.ai/fal/` route.

Video Face Swap takes a head reference image and a body-reference video, and swaps the head from the image onto the person in the video, producing a new video. No prompt is required.

Use these endpoints to create jobs, poll their status, cancel if still queued, and fetch the resulting video.

Note: All routes require an authenticated request (middleware `requireUser`).

## Base URL

- Production: `https://api.pipelet.ai`


## Endpoints (quick reference)

- Create request: `POST /fal/queue/video-face-swap`
- Check status: `GET /fal/queue/video-face-swap/requests/:requestId/status`
- Cancel request: `PUT /fal/queue/video-face-swap/requests/:requestId/cancel`
- Fetch result: `GET /fal/queue/video-face-swap/requests/:requestId`

---

## 1) Create a request

POST `https://api.pipelet.ai/fal/queue/video-face-swap`

Body: JSON inputs for the model.
- `image_url` (required) is the head reference image, whose face/head will be swapped onto the person in the video. It must be a public URL (e.g. an S3 pre-signed URL), not a base64 data URI.
- `video_url` (required) is the body-reference video that needs the head swap. It must be a public URL (e.g. an S3 pre-signed URL), not a base64 data URI.
- `priority` (optional, integer, default is 0) for priority in the queue, the bigger the number, the higher priority we have.

Both `image_url` and `video_url` must be URLs — base64 encoded inputs are not supported for this workflow.

Example:
```bash
curl -XPOST https://api.pipelet.ai/fal/queue/video-face-swap -H "Authorization: Bearer <your-api-key>" -d '{
    "image_url": "https://resource.pipelet.net/images/head_reference.png",
    "video_url": "https://resource.pipelet.net/videos/body_reference.mp4"
}'

```
It will return something like below:
```json
{
  "request_id": "rn5r12q9pdu6yvb3nxty",
  "response_url": "https://api.pipelet.ai/fal/queue/video-face-swap/requests/rn5r12q9pdu6yvb3nxty",
  "status_url": "https://api.pipelet.ai/fal/queue/video-face-swap/requests/rn5r12q9pdu6yvb3nxty/status",
  "cancel_url": "https://api.pipelet.ai/fal/queue/video-face-swap/requests/rn5r12q9pdu6yvb3nxty/cancel"
}
```
Notes:
- `request_id` is your handle for this job.
- Use `status_url` to poll for progress.
- Use `response_url` to fetch the completed result.
- Use `cancel_url` to cancel if the job has not started yet.


We will query the status of the job using the `status_url`.
```bash
curl "https://api.pipelet.ai/fal/queue/video-face-swap/requests/rn5r12q9pdu6yvb3nxty/status" -H "Authorization: Bearer $API_GATEWAY_USER_TOKEN"
```
Example response for status:

```json
{
  "status": "IN_PROGRESS",
  "queue_position": 0,
  "statusMessages": {
    "message": "Progress: 6/8 Tasks done",
    "estimatedTotalTimeSeconds": 180
  },
  "response_url": "https://api.pipelet.ai/fal/queue/video-face-swap/requests/rn5r12q9pdu6yvb3nxty"
}
```
When the status changes to "COMPLETED", we can fetch the result using the `response_url`.
```bash
curl "https://api.pipelet.ai/fal/queue/video-face-swap/requests/rn5r12q9pdu6yvb3nxty" -H "Authorization: Bearer $API_GATEWAY_USER_TOKEN"
```
Example response for result:
```json
{
  "video": {
    "data_uri": "https://prod-batch-files.ae2b0858dbcfcff39cc58bac85b7c66d.r2.cloudflarestorage.com/outputs/video_face_swap_api/2DPBix_ComfyUI_00008_.mp4?X-Amz-Expires=7200&X-Amz-Date=20251025T051931Z&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=9ca0f6abba71e6073fe4c8a9b21997d5%2F20251025%2Fauto%2Fs3%2Faws4_request&X-Amz-SignedHeaders=host&X-Amz-Signature=1031cdf308033f8cd7f9600379d16b6ac969bdb412986bc310c138ef33e1f88f",
    "content_type": "video/mp4",
    "file_name": "2DPBix_ComfyUI_00008_.mp4"
  },
  "preview_fps": 1,
  "previews": [
    {
      "url": "https://prod-batch-files.ae2b0858dbcfcff39cc58bac85b7c66d.r2.cloudflarestorage.com/outputs/video_face_swap_api/2DPBix_0_preview_000.jpg?X-Amz-Expires=14400&X-Amz-...",
      "index": 0,
      "timestamp": 0,
      "width": 640,
      "height": 366,
      "content_type": "image/jpeg"
    }
  ]
}
```

`previews` is a strip of JPEG frames sampled from the finished video, one per second
by default — useful for a thumbnail or scrub strip without downloading the MP4. The
key is omitted entirely when a job produced none, so read it defensively. See
[Preview frames](Video-Generation.md#previews) for the full field reference.
