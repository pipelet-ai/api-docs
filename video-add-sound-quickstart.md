## Video Add Sound API Quickstart

Video add sound is just like video generation, but it takes a video file, a helper prompt, and an option to specify whether ambient music is needed.
It is a huge help for the Wan22 video generation because the Wan22 does not have the capability to add sound to a video.

## Base URL

- Production: `https://batch.pipelet.net`

## Endpoints (quick reference)

There are 3 queues supported, mapping to different priority. Queues are `hunyuan-foley`, `hunyuan-foley-premium`, `hunyuan-foley-pro`.

- Create request: `POST /queue/hunyuan-foley` or `POST /queue/hunyuan-foley-premium` or `POST /queue/hunyuan-foley-pro`
- Check status: `GET /queue/{queue}/requests/:requestId/status` . For example `GET /queue/hunyuan-foley/requests/12/status`
- Cancel request: `PUT /queue/{queue}/requests/:requestId/cancel` . For example `PUT /queue/hunyuan-foley/requests/12/cancel`
- Fetch result: `GET /queue/{queue}/requests/:requestId` . For example `GET /queue/hunyuan-foley/requests/12`

## Request Format

### 1) Create a request
Request body is a JSON object, with the following fields:
- `data_uri`: base64 encoded video, or you can also specify a publicly accessible URL to the video, like the S3 Presigned URL
- `prompt`: the prompt that can help the AI to generate specific sound. Sometimes the model does not detect some sound effects like lion roars, we can add "lion **ROARS**" to the prompt to help the model.
- `background_music`: boolean, whether to add background music. The default is `false`. This feature is not 100% guaranteed but it is a big help.
- `priority`: integer, the priority of the job. The bigger the number, the higher priority we have. You can use whatever integer you have.

```
curl -XPOST `https://batch.pipelet.net/queue/hunyuan-foley-premium` \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer {your-bearer-token}' \
  -d '{"data_uri": "(base64 encoded image, or S3 Pre-signed URL)", "prompt": "The girl sings a sad song with tears and emotion", "background_music": true, "priority": 40}'
```

### 2) Poll for status, Exactly the same as video generation

```
curl https://batch.pipelet.net/queue/wan22-s2v-premium/requests/12/status -H "Authorization: Bearer {your-bearer-token}"
```

### 3) Cancel a queued request, Exactly the same as video generation

```
curl -XPUT https://batch.pipelet.net/queue/wan22-s2v-premium/requests/16/cancel -H "Authorization: Bearer {your-bearer-token}"
```

### 4) Fetch the result. The data_uri will be a S3 compatible Presigned URL starts with https://

```
curl -XGET https://batch.pipelet.net/queue/wan22-s2v-premium/requests/13 -H "Authorization: Bearer {your-bearer-token}"
```
Sample output format is like below:
```
{
  "video": {
    "data_uri": "https://staging-batch-files.ae2b0858dbcfcff39cc58bac85b7c66d.r2.cloudflarestorage.com/outputs/wan22-fast-i2v/115_AnimateDiff_00125.mp4?X-Amz-Expires=1800&X-Amz-Date=20250904T213408Z&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=e233b3b06e4169616188cfee2b20c414%2F20250904%2Fauto%2Fs3%2Faws4_request&X-Amz-SignedHeaders=host&X-Amz-Signature=d1d3d9ea4befc7a88f95bcf5ec31c8e44395062117a7dc2d8ef2732584615e89"
  }
}
```
We can parse the data_uri to get the file name too.