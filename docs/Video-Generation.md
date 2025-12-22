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
  "priority": 30,
  "higher_quality_with_more_steps": false
}
```

NOTE: the `higher_quality_with_more_steps` and `use_best_resolution` are optional, and are set to false by default.
`higher_quality_with_more_steps` will take more time to generate the video, but will be higher quality.

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
  "priority": 30,
  "higher_quality_with_more_steps": false,
  "use_best_resolution": false
}
```

#### Image To Video with First-Frame and Last-Frame

```json
{
  "prompt": "The man does a backflip",
  "data_uri": "(base64 encoded image)",
  "second_image_uri": "(base64 encoded image)",
  "duration": 5,
  "min_height": 480,
  "priority": 30,
  "higher_quality_with_more_steps": false,
  "use_best_resolution": false
}
```

NOTE: `use_best_resolution` will make the video choose the shortest side of the image to match the "height" or "min_height" value.
For example, if the input image is like a selfie, 9:16, portrait style, and "height" is 480, use_best_resolution will make the video 480x832.
If use_best_resolution is false, then the video will have 480 as height, resulting a smaller video like 270x480

NOTE: We support first-frame plus last-frame to generate a video. If the `second_image_uri` is provided, the video will be generated with the `data_uri` as first frame and the `second_image_uri` as the last frame.

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
  "priority": 30,
  "higher_quality_with_more_steps": false,
  "use_best_resolution": false
}
```
### Add Sound To Video Models

`hunyuan-foley-pro`, `hunyuan-foley`, `hunyuan-foley-premium`

These are the same hunyuan foley model, but `hunyuan-foley-pro` has a higher priority in the queue.

```json
{
  "prompt": "The girl sings a sad song with tears and emotion",
  "data_uri": "(base64 encoded image or S3 Pre-signed URL)",
  "background_music": true,
  "priority": 30,
  "higher_quality_with_more_steps": false,
  "use_best_resolution": false
}
```
## 1.2) Controlling the output format of the video {#output-format}

When you create a request, you can control the output format of the video by adding an extra header "x-pipelet-output: binary" to the request.

This will force the output to be a binary file, instead of a base64 encoded string.

When this is set, the response field will be a pres-signed download URL for the binary mp4 file, like below:
```
curl https://batch.pipelet.net/queue/wan22-fast-i2v/requests/375867 -H "Authorization: Bearer ${BATCH_USER_TOKEN}"
{"video":{"data_uri":"https://prod-batch-files.ae2b0858dbcfcff39cc58bac85b7c66d.r2.cloudflarestorage.com/outputs/free/375867_0_WanVideo2_2_I2V_00781.mp4?X-Amz-Expires=3600&X-Amz-Date=20251222T045058Z&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=0c9db542c165ecde3eae49725be07d03%2F20251222%2Fauto%2Fs3%2Faws4_request&X-Amz-SignedHeaders=host&X-Amz-Signature=0c0a67d48ecb346851bd8deb8f1833af14fe4cb6023d93392e4feab3b18ed813","data_url":"https://prod-batch-files.ae2b0858dbcfcff39cc58bac85b7c66d.r2.cloudflarestorage.com/outputs/free/375867_0_WanVideo2_2_I2V_00781.mp4?X-Amz-Expires=3600&X-Amz-Date=20251222T045058Z&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=0c9db542c165ecde3eae49725be07d03%2F20251222%2Fauto%2Fs3%2Faws4_request&X-Amz-SignedHeaders=host&X-Amz-Signature=0c0a67d48ecb346851bd8deb8f1833af14fe4cb6023d93392e4feab3b18ed813"}}
```

## 1.3) Downloading the video file directly instead of from the response field. {#download-direct}

When we poll for the status, we can see the `data_uri` field with the s3://(bucket_name)/path/to/file format, like below:
```
{"status":"COMPLETED","queue_position":0,"statusMessages":{"message":"Job completed successfully","result_uri":"s3://prod-batch-files/outputs/free/375867_0_WanVideo2_2_I2V_00781.mp4","results":["s3://prod-batch-files/outputs/free/375867_0_WanVideo2_2_I2V_00781.mp4"]},"etaSeconds":0,"response_url":"https://batch.pipelet.net/queue/wan22-fast-i2v/requests/375867"}
```
We can keep the s3://(bucket_name)/path/to/file format, and download the file directly from the s3 bucket, via the below endpoint:
```
curl -H "Authorization: Bearer $API_GATEWAY_USER_TOKEN" https://api.pipelet.ai/fal/download/(bucket_name)/path/to/file
```
For example:
```
curl -H "Authorization: Bearer $API_GATEWAY_USER_TOKEN" https://api.pipelet.ai/fal/download/prod-batch-files/outputs/free/375867_0_WanVideo2_2_I2V_00781.mp4
{"url":"https://prod-batch-files.ae2b0858dbcfcff39cc58bac85b7c66d.r2.cloudflarestorage.com/outputs/free/375867_0_WanVideo2_2_I2V_00781.mp4?X-Amz-Expires=7200&X-Amz-Date=20251222T044352Z&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=0c9db542c165ecde3eae49725be07d03%2F20251222%2Fauto%2Fs3%2Faws4_request&X-Amz-SignedHeaders=host&X-Amz-Signature=3dfe21762327f1fb087b14f508bccfcc51795cbd807c2ec661ec799ced38eccc","key":"outputs/free/375867_0_WanVideo2_2_I2V_00781.mp4","r2_uri":"r2://prod-batch-files/outputs/free/375867_0_WanVideo2_2_I2V_00781.mp4"}
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

- `video.data_uri` can be two formats: one is just a base64 encoded string of an MP4 file encoded with H.264, the other is a S3 compatible Presigned URL starts with https://
- Decode and save it locally or download it from the S3 URL to obtain the final `.mp4`.

Example to save locally (macOS/Linux):

```bash
curl -s https://batch.pipelet.net/queue/wan22-fast-i2v/requests/13 \
  | jq -r '.video.data_uri' | base64 --decode > output.mp4
```

Example with https:// pre-signed URL:

```
{
  "video": {
    "data_uri": "https://staging-batch-files.ae2b0858dbcfcff39cc58bac85b7c66d.r2.cloudflarestorage.com/outputs/wan22-fast-i2v/115_AnimateDiff_00125.mp4?X-Amz-Expires=1800&X-Amz-Date=20250904T213408Z&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=e233b3b06e4169616188cfee2b20c414%2F20250904%2Fauto%2Fs3%2Faws4_request&X-Amz-SignedHeaders=host&X-Amz-Signature=d1d3d9ea4befc7a88f95bcf5ec31c8e44395062117a7dc2d8ef2732584615e89"
  }
}
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
