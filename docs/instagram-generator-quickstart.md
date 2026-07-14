# Instagram Generator API

This document describes the Instagram generator endpoints in the `https://api.pipelet.ai/fal/` route.

Instagram Generator takes an input image as the "main character" and re-generates that character into a new Instagram-style photo described by your prompt, while preserving the person's facial identity, hairstyle, and other key features.

## Example: before & after

The same person, dropped into a brand-new scene, outfit, and pose — while her face, hairstyle, and identity stay consistent.

<table>
  <tr>
    <th align="center">Input — original selfie</th>
    <th align="center">Output — new scene, same person</th>
  </tr>
  <tr>
    <td align="center"><img src="../assets/instagram_input.jpeg" alt="Input selfie used as the main character" width="360" /></td>
    <td align="center"><img src="../assets/instagram_output.png" alt="Generated Instagram photo preserving the same identity" width="360" /></td>
  </tr>
</table>

**Prompt used:** `She is now seated in a deep purple velvet armchair against a red brick wall inside a richly detailed London townhouse, one leg crossed over the other, calm confident expression, wearing a dark black wash denim jacket and tight jeans.`

Notice the model keeps the same face, hairstyle, and facial features while fully changing the background, wardrobe, and pose — this is the character-consistency problem that Instagram Generator is built to solve.

Use these endpoints to create jobs, poll their status, cancel if still queued, and fetch the resulting image.

Note: All routes require an authenticated request (middleware `requireUser`).

## Base URL

- Production: `https://api.pipelet.ai`


## Endpoints (quick reference)

- Create request: `POST /fal/queue/instagram-generator`
- Check status: `GET /fal/queue/instagram-generator/requests/:requestId/status`
- Cancel request: `PUT /fal/queue/instagram-generator/requests/:requestId/cancel`
- Fetch result: `GET /fal/queue/instagram-generator/requests/:requestId`

---

## 1) Create a request

POST `https://api.pipelet.ai/fal/queue/instagram-generator`

Body: JSON inputs for the model.
- `image_url` or `image_uri` (required) for the main character image. `image_url` is a public URL, e.g., a S3 Pre-signed URL. `image_uri` is a base64 encoded image.
- `prompt` (required) describing the scene, pose, outfit, and setting for the output photo. The character's facial identity is preserved automatically.
- `width` and `height` for the output image resolution (defaults to `1080` x `1920`, a 9:16 portrait). Values are rounded to the nearest multiple of 8.
- `batch_size` (optional, integer, default is 1) to generate multiple images at once.
- `enrich_prompt` (optional, boolean, true/false, default is false). When true, a vision model expands your prompt with additional photographic detail (lighting, environment, wardrobe, mood, framing) before generation.
- `nsfw` (optional, boolean, true/false, default is false) for enabling NSFW content.
- `priority` (optional, integer, default is 0) for priority in the queue, the bigger the number, the higher priority we have.

Note on prompts: your prompt is automatically passed through a vision model that looks at the main character image and rewrites it so the main character is the only subject. For example, a prompt like `1girl dancing in the park` is rewritten to `She is dancing in the park`, which prevents the model from blending your main character with an invented extra person. If the rewrite service is unavailable, your original prompt is used as-is.

Example:
```bash
curl -XPOST https://api.pipelet.ai/fal/queue/instagram-generator -H "Authorization: Bearer <your-api-key>" -d '{
    "image_url": "https://resource.pipelet.net/images/instagram_test.jpeg",
    "width": "1080",
    "height": "1920",
    "prompt": "She is now seated in a deep purple velvet armchair against a red brick wall inside a richly detailed London townhouse, one leg crossed over the other, calm confident expression, wearing a dark black wash denim jacket and tight jeans."
}'

```
It will return something like below:
```json
{
  "request_id": "rn5r12q9pdu6yvb3nxty",
  "response_url": "https://api.pipelet.ai/fal/queue/instagram-generator/requests/rn5r12q9pdu6yvb3nxty",
  "status_url": "https://api.pipelet.ai/fal/queue/instagram-generator/requests/rn5r12q9pdu6yvb3nxty/status",
  "cancel_url": "https://api.pipelet.ai/fal/queue/instagram-generator/requests/rn5r12q9pdu6yvb3nxty/cancel"
}
```
Notes:
- `request_id` is your handle for this job.
- Use `status_url` to poll for progress.
- Use `response_url` to fetch the completed result.
- Use `cancel_url` to cancel if the job has not started yet.


We will query the status of the job using the `status_url`.
```bash
curl "https://api.pipelet.ai/fal/queue/instagram-generator/requests/rn5r12q9pdu6yvb3nxty/status" -H "Authorization: Bearer $API_GATEWAY_USER_TOKEN"
```
Example response for status:

```json
{
  "status": "IN_PROGRESS",
  "queue_position": 0,
  "statusMessages": {
    "message": "Progress: 10/12 Tasks done",
    "estimatedTotalTimeSeconds": 40
  },
  "response_url": "https://api.pipelet.ai/fal/queue/instagram-generator/requests/rn5r12q9pdu6yvb3nxty"
}
```
When the status changes to "COMPLETED", we can fetch the result using the `response_url`.
```bash
curl "https://api.pipelet.ai/fal/queue/instagram-generator/requests/rn5r12q9pdu6yvb3nxty" -H "Authorization: Bearer $API_GATEWAY_USER_TOKEN"
```
Example response for result:
```json
{
  "image": {
    "data_uri": "https://prod-batch-files.ae2b0858dbcfcff39cc58bac85b7c66d.r2.cloudflarestorage.com/outputs/instagram_generator_api/2DPBix_ComfyUI_00008_.png?X-Amz-Expires=7200&X-Amz-Date=20251025T051931Z&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=9ca0f6abba71e6073fe4c8a9b21997d5%2F20251025%2Fauto%2Fs3%2Faws4_request&X-Amz-SignedHeaders=host&X-Amz-Signature=1031cdf308033f8cd7f9600379d16b6ac969bdb412986bc310c138ef33e1f88f",
    "file_name": "2DPBix_ComfyUI_00008_.png"
  }
}
```
