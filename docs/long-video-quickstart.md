## Long Video (Image To Video) API Quickstart

Long video generation is like image-to-video, but it uses a different queue endpoint and supports prompts per 5 seconds.

The difference is that it uses two different queue endpoints, `wan22-svi-i2v-pro` and `wan22-svi-i2v-premium`, and supports prompts per 5 seconds.

## Base URL

- Production: `https://api.pipelet.ai`

## Endpoints (quick reference)

- Create request: `POST /fal/queue/wan22-svi-i2v-pro`
- Check status: `GET /fal/queue/wan22-svi-i2v-pro/requests/:requestId/status`
- Cancel request: `PUT /fal/queue/wan22-svi-i2v-pro/requests/:requestId/cancel`
- Fetch result: `GET /fal/queue/wan22-svi-i2v-pro/requests/:requestId`

## Request Format

### 1) Create a request

Request body is a JSON object, with the following fields:
- `data_uri`: base64 encoded image
- `image_url`: S3 Pre-signed URL for image, or public URL with https access.
- `prompt`: prompts for each 5 seconds of the video, separated by `|`. E.g., `"The girl starts walking towards the camera|The girl turns to the left"`
  - If the video duration requires more segments than provided prompts, the last prompt will be repeated and reused.
- `duration`: integer, supported values are 10, 15, 20, 30 (seconds)
- `min_height`: integer, supported values are 480, 720, 1080
- `priority`: integer, the priority of the job. The bigger the number, the higher priority we have. You can use whatever integer you have.

This endpoint supports webhook the same way as described in `docs/webhook-quick-start.md`. You can pass webhook via query param `fal_webhook`.

```
curl -XPOST "https://api.pipelet.ai/fal/queue/wan22-svi-i2v-pro?fal_webhook=https://echo.hetzner1.pipelet.ai/" -H "Authorization: Bearer ptk_1~kHCeA1uJgl3dPw.xxxx" -d '{
    "image_url": "https://resource.pipelet.net/demos/music_videos/source/2077-style.png",
    "prompt": "The girl starts walking towards the camera|The girl turns to the left",
    "duration": 15,
    "min_height": 1080
}'
```

### 2) Poll for status, Exactly the same as video generation

```
curl https://api.pipelet.ai/fal/queue/wan22-svi-i2v-pro/requests/5e8f8ukijqeire/status -H "Authorization: Bearer {your-bearer-token}"
```

### 3) Cancel a queued request, Exactly the same as video generation

```
curl -XPUT https://api.pipelet.ai/fal/queue/wan22-svi-i2v-pro/requests/5e8f8ukijqeire/cancel -H "Authorization: Bearer {your-bearer-token}"
```

### 4) Fetch the result, Exactly the same as video generation

```
curl -XGET https://api.pipelet.ai/fal/queue/wan22-svi-i2v-pro/requests/5e8f8ukijqeire -H "Authorization: Bearer {your-bearer-token}"
```
