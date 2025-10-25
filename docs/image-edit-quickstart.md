# Image Edit API

This document describes the image edit endpoints in `https://api.pipelet.ai/fal/` route

Use these endpoints to create image edit jobs, poll their status, cancel if still queued, and fetch the resulting image.

Note: All routes require an authenticated request (middleware `requireUser`).

## Base URL

- Production: `https://api.pipelet.ai`


## Endpoints (quick reference)

- Create request: `POST /fal/queue/qwen-image-edit`
- Check status: `GET /fal/queue/qwen-image-edit/requests/:requestId/status`
- Cancel request: `PUT /fal/queue/qwen-image-edit/requests/:requestId/cancel`
- Fetch result: `GET /fal/queue/qwen-image-edit/requests/:requestId`

---

## 1) Create a request

POST `https://api.pipelet.ai/fal/queue/qwen-image-edit`

Body: JSON inputs for the model.
- `width` and `height` for image resolution.
- `prompt` for image edit prompt.
- `image_url` or `image_uri` (required) for first/main image. `image_url` is a public URL, e.g., a S3 Pre-signed URL. `image_uri` is a base64 encoded image.
- `second_image_url` or `second_image_uri` (optional) for second image. `second_image_url` is a public URL, e.g., a S3 Pre-signed URL. `second_image_uri` is a base64 encoded image.
- `third_image_url` or `third_image_uri` (optional) for third image. `third_image_url` is a public URL, e.g., a S3 Pre-signed URL. `third_image_uri` is a base64 encoded image.
- `nsfw` (optional, boolean, true/false, default is false) for enabling NSFW content.
- `priority` (optional, integer, default is 0) for priority in the queue, the bigger the number, the higher priority we have.

Example:
```bash
curl -XPOST https://api.pipelet.ai/fal/queue/qwen-image-edit -H "Authorization: Bearer <your-api-key>" -d '{
    "image_url": "https://resource.pipelet.net/images/instagram_test.jpeg",
    "second_image_url": "r2://public-assets/images/instagram_test2.jpeg",
    "width": "1280",
    "height": "720",
    "third_image_url": "https://resource.pipelet.net/images/ancient_table.jpg",
    "prompt": "The girl from first image is playing chess with the girl from second image, on the ancient table from third image.",
    "nsfw": true
}'

```
It will return something like below:
```json
{
  "request_id": "rn5r12q9pdu6yvb3nxty",
  "response_url": "https://api.pipelet.ai/fal/queue/qwen-image-edit/requests/rn5r12q9pdu6yvb3nxty",
  "status_url": "https://api.pipelet.ai/fal/queue/qwen-image-edit/requests/rn5r12q9pdu6yvb3nxty/status",
  "cancel_url": "https://api.pipelet.ai/fal/queue/qwen-image-edit/requests/rn5r12q9pdu6yvb3nxty/cancel"
}
```
Notes:
- `request_id` is your handle for this job.
- Use `status_url` to poll for progress.
- Use `response_url` to fetch the completed result.
- Use `cancel_url` to cancel if the job has not started yet.


We will query the status of the job using the `status_url`.
```bash
curl "https://api.pipelet.ai/fal/queue/qwen-image-edit/requests/rn5r12q9pdu6yvb3nxty/status" -H "Authorization: Bearer $API_GATEWAY_USER_TOKEN"
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
  "response_url": "https://api.pipelet.ai/fal/queue/qwen-image-edit/requests/rn5r12q9pdu6yvb3nxty"
}
```
When the status changes to "COMPLETED", we can fetch the result using the `response_url`.
```bash
curl "https://api.pipelet.ai/fal/queue/qwen-image-edit/requests/rn5r12q9pdu6yvb3nxty" -H "Authorization: Bearer $API_GATEWAY_USER_TOKEN"
```
Example response for result:
```json
{
  "video": {
    "data_uri": "https://prod-batch-files.ae2b0858dbcfcff39cc58bac85b7c66d.r2.cloudflarestorage.com/outputs/qwen_image_edit_api/2DPBix_qwen_image__00008_.png?X-Amz-Expires=7200&X-Amz-Date=20251025T051931Z&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=9ca0f6abba71e6073fe4c8a9b21997d5%2F20251025%2Fauto%2Fs3%2Faws4_request&X-Amz-SignedHeaders=host&X-Amz-Signature=1031cdf308033f8cd7f9600379d16b6ac969bdb412986bc310c138ef33e1f88f",
    "file_name": "2DPBix_qwen_image__00008_.png"
  }
}
```
**Note** yes we use "video" as key because we are using the same response format as video generation.

