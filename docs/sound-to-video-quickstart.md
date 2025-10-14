## Sound To Video API Quickstart

Sound to video is just like video generation, but it takes an audio, a prompt and generate a video.
It is very popular in the case to animate a character to lip-sync to a given audio.

## Base URL

- Production: `https://api.pipelet.ai`

## Endpoints (quick reference)

There are 3 queues supported, mapping to different priority. Queue name is `wan22-s2v-extreme-length`

- Create request: `POST /queue/wan22-s2v-extreme-length`
- Check status: `GET /queue/wan22-s2v-extreme-length/requests/:requestId/status`
- Cancel request: `PUT /queue/wan22-s2v-extreme-length/requests/:requestId/cancel`
- Fetch result: `GET /queue/wan22-s2v-extreme-length/requests/:requestId`

## Request Format

### 1) Create a request
Request body is a JSON object, with the following fields:
- `audio_uri`: base64 encoded audio
- `data_uri`: base64 encoded image
- `audio_url`: S3 Pre-signed URL for audio. Only one of `audio_uri` and `audio_url` can be provided
- `data_url`: S3 Pre-signed URL for image. Only one of `data_uri` and `data_url` can be provided
- `prompt`: the prompt to generate the video
- `duration`: optional, the **max**duration of the video in seconds, default is 300 seconds
- `min_height`: integer, the minimum height of the video. We should put in 480 or 512. 
- `priority`: integer, the priority of the job. The bigger the number, the higher priority we have. You can use whatever integer you have.

```
curl -XPOST `https://api.pipelet.ai/queue/wan22-s2v-extreme-length` \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer {your-bearer-token}' \
  -d '{"audio_uri": "(base64 encoded audio)", "data_uri": "(base64 encoded image)", "prompt": "The girl sings a sad song with tears and emotion", "duration": 5, "min_height": 480, "priority": 40}'
```

### 2) Poll for status, Exactly the same as video generation

```
curl https://api.pipelet.ai/queue/wan22-s2v-premium/requests/12/status -H "Authorization: Bearer {your-bearer-token}"
```

### 3) Cancel a queued request, Exactly the same as video generation

```
curl -XPUT https://api.pipelet.ai/queue/wan22-s2v-premium/requests/16/cancel -H "Authorization: Bearer {your-bearer-token}"
```

### 4) Fetch the result, Exactly the same as video generation

```
curl -XGET https://api.pipelet.ai/queue/wan22-s2v-premium/requests/13 -H "Authorization: Bearer {your-bearer-token}"
```
