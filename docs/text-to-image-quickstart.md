# Image Edit API

This document describes the image edit endpoints in `https://api.pipelet.ai/fal/` route

Use these endpoints to create image edit jobs, poll their status, cancel if still queued, and fetch the resulting image.

Note: All routes require an authenticated request (middleware `requireUser`).

## Base URL

- Production: `https://api.pipelet.ai`


## Endpoints (quick reference)

- Create request: `POST /fal/queue/z-image-text-to-image`
- Check status: `GET /fal/queue/z-image-text-to-image/requests/:requestId/status`
- Cancel request: `PUT /fal/queue/z-image-text-to-image/requests/:requestId/cancel`
- Fetch result: `GET /fal/queue/z-image-text-to-image/requests/:requestId`

---

## 1) Create a request

POST `https://api.pipelet.ai/fal/queue/z-image-text-to-image`

Body: JSON inputs for the model.
- `width` and `height` for image resolution.
- `prompt` for image edit prompt.
- `priority` (optional, integer, default is 0) for priority in the queue, the bigger the number, the higher priority we have.
- `batch_size` (optional, integer, default is 1) for batch size. Generate multiple images at once.

Example:
```bash
curl -XPOST https://api.pipelet.ai/fal/queue/z-image-text-to-image -H "Authorization: Bearer <your-api-key>" -d '{
    "width": "3200",
    "height": "2000",
    "batch_size": 4,
    "prompt": "The girl from first image is playing chess with the girl from second image, on the ancient table from third image."
}'

```
It will return something like below:
```json
{
  "request_id": "rn5r12q9pdu6yvb3nxty",
  "response_url": "https://api.pipelet.ai/fal/queue/z-image-image-edit/requests/rn5r12q9pdu6yvb3nxty",
  "status_url": "https://api.pipelet.ai/fal/queue/z-image-image-edit/requests/rn5r12q9pdu6yvb3nxty/status",
  "cancel_url": "https://api.pipelet.ai/fal/queue/z-image-image-edit/requests/rn5r12q9pdu6yvb3nxty/cancel"
}
```
Notes:
- `request_id` is your handle for this job.
- Use `status_url` to poll for progress.
- Use `response_url` to fetch the completed result.
- Use `cancel_url` to cancel if the job has not started yet.


We will query the status of the job using the `status_url`.
```bash
curl "https://api.pipelet.ai/fal/queue/z-image-text-to-image/requests/rn5r12q9pdu6yvb3nxty/status" -H "Authorization: Bearer $API_GATEWAY_USER_TOKEN"
```
Example response for status:

```json
{
  "status": "IN_PROGRESS",
  "queue_position": 0,
  "statusMessages": {
    "message": "Progress: 10/12 Tasks done",
    "estimatedTotalTimeSeconds": 120
  },
  "response_url": "https://api.pipelet.ai/fal/queue/z-image-text-to-image/requests/rn5r12q9pdu6yvb3nxty"
}
```
When the status changes to "COMPLETED", we can fetch the result using the `response_url`.
```bash
curl "https://api.pipelet.ai/fal/queue/z-image-text-to-image/requests/rn5r12q9pdu6yvb3nxty" -H "Authorization: Bearer $API_GATEWAY_USER_TOKEN"
```
Example response for result:
```json
{
  "image": {
    "data_uri": "https://prod-batch-files.ae2b0858dbcfcff39cc58bac85b7c66d.r2.cloudflarestorage.com/outputs/z_image_image_edit_api/2DPBix_z_image_image__00008_.png?X-Amz-Expires=7200&X-Amz-Date=20251025T051931Z&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=9ca0f6abba71e6073fe4c8a9b21997d5%2F20251025%2Fauto%2Fs3%2Faws4_request&X-Amz-SignedHeaders=host&X-Amz-Signature=1031cdf308033f8cd7f9600379d16b6ac969bdb412986bc310c138ef33e1f88f",
    "file_name": "2DPBix_z_image_image__00008_.png"
  },
  "results": [
    "https://prod-batch-files.ae2b0858dbcfcff39cc58bac85b7c66d.r2.cloudflarestorage.com/outputs/z_image_image_edit_api/2DPBix_z_image_image__00008_.png?X-Amz-Expires=7200&X-Amz-Date=20251025T051931Z&X-Amz-Algorithm=AWS4-HMAC-SHA256....",
    "https://prod-batch-files.ae2b0858dbcfcff39cc58bac85b7c66d.r2.cloudflarestorage.com/outputs/z_image_image_edit_api/2DPBix_z_image_image__00009_.png?X-Amz-Expires=7200&X-Amz-Date=20251025T051931Z&X-Amz-Algorithm=AWS4-HMAC-SHA256....",
    "https://prod-batch-files.ae2b0858dbcfcff39cc58bac85b7c66d.r2.cloudflarestorage.com/outputs/z_image_image_edit_api/2DPBix_z_image_image__00010_.png?X-Amz-Expires=7200&X-Amz-Date=20251025T051931Z&X-Amz-Algorithm=AWS4-HMAC-SHA256....",
    "https://prod-batch-files.ae2b0858dbcfcff39cc58bac85b7c66d.r2.cloudflarestorage.com/outputs/z_image_image_edit_api/2DPBix_qwen_image__00011_.png?X-Amz-Expires=7200&X-Amz-Date=20251025T051931Z&X-Amz-Algorithm=AWS4-HMAC-SHA256...."
  ]
}
```

