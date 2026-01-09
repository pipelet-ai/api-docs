# Image Edit API

This document describes the image edit endpoints in `https://api.pipelet.ai/fal/` route

Use these endpoints to create image edit jobs, poll their status, cancel if still queued, and fetch the resulting image.

Note: All routes require an authenticated request (middleware `requireUser`).

## Base URL

- Production: `https://api.pipelet.ai`


## Endpoints (quick reference)

- Create request: `POST /fal/queue/qwen-image-edit-multi-angle`
- Check status: `GET /fal/queue/qwen-image-edit-multi-angle/requests/:requestId/status`
- Cancel request: `PUT /fal/queue/qwen-image-edit-multi-angle/requests/:requestId/cancel`
- Fetch result: `GET /fal/queue/qwen-image-edit-multi-angle/requests/:requestId`

---

## 1) Create a request

POST `https://api.pipelet.ai/fal/queue/qwen-image-edit-multi-angle`

Body: JSON inputs for the model.
- `image_url` (required, string) for image URL.
- `priority` (optional, integer, default is 0) for priority in the queue, the bigger the number, the higher priority we have.
- `azimuth` (optional, array of strings, default is ["all"]) for azimuth of the camera, can be "front view", "front-right quarter view", "front-left quarter view", "back view", "back-right quarter view", "back-left quarter view", "left side view", "right side view"
- `elevation` (optional, array of strings, default is ["all"]) for elevation of the camera, can be "low-angle shot", "eye-level shot", "elevated shot", "high-angle shot"
- `distance` (optional, array of strings, default is ["all"]) for distance of the camera, can be "close-up", "medium shot", "wide shot"
The result will be a multiplication of the azimuth, elevation, and distance. For example if we provide azimuth as ["front view", "back view"], elevation as ["eye-level shot", "high-angle shot"], and distance as ["close-up", "medium shot"], the result will be 8 images.

If you wish for more detailed explanation, right now we are loading https://huggingface.co/fal/Qwen-Image-Edit-2511-Multiple-Angles-LoRA and it explains the model.

Example:
```bash
curl -XPOST https://api.pipelet.ai/fal/queue/qwen-image-edit-multi-angle -H "Authorization: Bearer <your-api-key>" -d '{
    "image_url": "https://resource.pipelet.net/images/instagram_test.jpeg",
    "priority": 4,
    "azimuth": ["all"],
    "elevation": ["eye-level shot", "high-angle shot"],
    "distance": ["close-up", "medium shot"]
}'

```
It will return something like below:
```json
{
  "request_id": "rn5r12q9pdu6yvb3nxty",
  "response_url": "https://api.pipelet.ai/fal/queue/qwen-image-edit-multi-angle/requests/rn5r12q9pdu6yvb3nxty",
  "status_url": "https://api.pipelet.ai/fal/queue/qwen-image-edit-multi-angle/requests/rn5r12q9pdu6yvb3nxty/status",
  "cancel_url": "https://api.pipelet.ai/fal/queue/qwen-image-edit-multi-angle/requests/rn5r12q9pdu6yvb3nxty/cancel"
}
```
Notes:
- `request_id` is your handle for this job.
- Use `status_url` to poll for progress.
- Use `response_url` to fetch the completed result.
- Use `cancel_url` to cancel if the job has not started yet.


We will query the status of the job using the `status_url`.
```bash
curl "https://api.pipelet.ai/fal/queue/qwen-image-edit-multi-angle/requests/rn5r12q9pdu6yvb3nxty/status" -H "Authorization: Bearer $API_GATEWAY_USER_TOKEN"
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
  "response_url": "https://api.pipelet.ai/fal/queue/qwen-image-edit-multi-angle/requests/rn5r12q9pdu6yvb3nxty"
}
```
When the status changes to "COMPLETED", we can fetch the result using the `response_url`.
```bash
curl "https://api.pipelet.ai/fal/queue/qwen-image-edit-multi-angle/requests/rn5r12q9pdu6yvb3nxty" -H "Authorization: Bearer $API_GATEWAY_USER_TOKEN"
```
Example response for result:
```json
{
  "image": {
    "data_uri": "https://prod-batch-files.ae2b0858dbcfcff39cc58bac85b7c66d.r2.cloudflarestorage.com/outputs/qwen_image_edit_api/2DPBix_qwen_image__00008_.png?X-Amz-Expires=7200&X-Amz-Date=20251025T051931Z&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=9ca0f6abba71e6073fe4c8a9b21997d5%2F20251025%2Fauto%2Fs3%2Faws4_request&X-Amz-SignedHeaders=host&X-Amz-Signature=1031cdf308033f8cd7f9600379d16b6ac969bdb412986bc310c138ef33e1f88f",
    "file_name": "2DPBix_qwen_image__00008_.png"
  },
  "results": [
    "https://prod-batch-files.ae2b0858dbcfcff39cc58bac85b7c66d.r2.cloudflarestorage.com/outputs/qwen_image_edit_api/2DPBix_qwen_image__00008_.png?X-Amz-Expires=7200&X-Amz-Date=20251025T051931Z&X-Amz-Algorithm=AWS4-HMAC-SHA256....",
    "https://prod-batch-files.ae2b0858dbcfcff39cc58bac85b7c66d.r2.cloudflarestorage.com/outputs/qwen_image_edit_api/2DPBix_qwen_image__00009_.png?X-Amz-Expires=7200&X-Amz-Date=20251025T051931Z&X-Amz-Algorithm=AWS4-HMAC-SHA256....",
    "https://prod-batch-files.ae2b0858dbcfcff39cc58bac85b7c66d.r2.cloudflarestorage.com/outputs/qwen_image_edit_api/2DPBix_qwen_image__00010_.png?X-Amz-Expires=7200&X-Amz-Date=20251025T051931Z&X-Amz-Algorithm=AWS4-HMAC-SHA256....",
    "https://prod-batch-files.ae2b0858dbcfcff39cc58bac85b7c66d.r2.cloudflarestorage.com/outputs/qwen_image_edit_api/2DPBix_qwen_image__00011_.png?X-Amz-Expires=7200&X-Amz-Date=20251025T051931Z&X-Amz-Algorithm=AWS4-HMAC-SHA256....",
    ....
  ]
}
```

