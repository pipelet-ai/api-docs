## Video Extension (Image To Video Extend) API Quickstart

Video extension is like image-to-video, but it extends an existing video.

The difference is that the input is a video (while still using the same input field name `image_url` or `data_url`), it uses a different queue endpoint, `wan22-svi-extend`, and it supports more flexible durations.

## Base URL

- Production: `https://api.pipelet.ai`

## Endpoints (quick reference)

- Create request: `POST /fal/queue/wan22-svi-extend`
- Check status: `GET /fal/queue/wan22-svi-extend/requests/:requestId/status`
- Cancel request: `PUT /fal/queue/wan22-svi-extend/requests/:requestId/cancel`
- Fetch result: `GET /fal/queue/wan22-svi-extend/requests/:requestId`

## Request Format

### 1) Create a request

Request body is a JSON object, with the following fields:
- `data_url`: base64 encoded video
- `image_url`: S3 Pre-signed URL for video, or public URL with https access.
- `prompt`: the prompt to guide the extension
- `duration`: integer, supported values are 5, 10, 15, 20, 30 (seconds)
- `min_height`: integer, supported values are 480, 720, 1080
- `priority`: integer, the priority of the job. The bigger the number, the higher priority we have. You can use whatever integer you have.

This endpoint supports webhook the same way as described in `docs/webhook-quick-start.md`. You can pass webhook via query param `fal_webhook`.

```
curl -XPOST "https://api.pipelet.ai/fal/queue/wan22-svi-extend?fal_webhook=https://echo.hetzner1.pipelet.ai/" -H "Authorization: Bearer ptk_1~kHCeA1uJgl3dPw.xxxx" -d '{
    "image_url": "https://resource.pipelet.net/demos/music_videos/source/input.mp4",
    "prompt": "Extend the video with a smooth continuation",
    "duration": 10,
    "min_height": 1080
}'
```

### 2) Poll for status, Exactly the same as video generation

```
curl https://api.pipelet.ai/fal/queue/wan22-svi-extend/requests/5e8f8ukijqeire/status -H "Authorization: Bearer {your-bearer-token}"
```

### 3) Cancel a queued request, Exactly the same as video generation

```
curl -XPUT https://api.pipelet.ai/fal/queue/wan22-svi-extend/requests/5e8f8ukijqeire/cancel -H "Authorization: Bearer {your-bearer-token}"
```

### 4) Fetch the result, Exactly the same as video generation

```
curl -XGET https://api.pipelet.ai/fal/queue/wan22-svi-extend/requests/5e8f8ukijqeire -H "Authorization: Bearer {your-bearer-token}"
```
