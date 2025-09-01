## Sound To Video API Quickstart

Sound to video is just like video generation, but it takes an audio, a prompt and generate a video.
It is very popular in the case to animate a character to lip-sync to a given audio.

## Base URL

- Production: `https://batch.pipelet.net`

## Endpoints (quick reference)

There are 3 queues supported, mapping to different priority. Queues are `wan-s2v-fast`, `wan-s2v-pro`, `wan-s2v-premium`.

- Create request: `POST /queue/wan-s2v-fast` or `POST /queue/wan-s2v-pro` or `POST /queue/wan-s2v-premium`
- Check status: `GET /queue/{queue}/requests/:requestId/status` . For example `GET /queue/wan-s2v-fast/requests/12/status`
- Cancel request: `PUT /queue/{queue}/requests/:requestId/cancel` . For example `PUT /queue/wan-s2v-fast/requests/12/cancel`
- Fetch result: `GET /queue/{queue}/requests/:requestId` . For example `GET /queue/wan-s2v-fast/requests/12`

## Request Format

### 1) Create a request
Request body is a JSON object, with the following fields:
- `audio_uri`: base64 encoded audio
- `prompt`: the prompt to generate the video
- `duration`: the **max**duration of the video in seconds
- `min_height`: integer, the minimum height of the video. Usually 480 or 720. 
- `priority`: integer, the priority of the job. The bigger the number, the higher priority we have. You can use whatever integer you have.

```
curl -XPOST `https://batch.pipelet.net/queue/wan-s2v-premium` \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer {your-bearer-token}' \
  -d '{"audio_uri": "(base64 encoded audio)", "data_uri": "(base64 encoded image)", "prompt": "The girl sings a sad song with tears and emotion", "duration": 5, "min_height": 480, "priority": 40}'
```

### 2) Poll for status, Exactly the same as video generation

```
curl https://batch.pipelet.net/queue/wan-s2v-premium/requests/12/status
```

### 3) Cancel a queued request, Exactly the same as video generation

```
curl -XPUT https://batch.pipelet.net/queue/wan-s2v-premium/requests/16/cancel
```

### 4) Fetch the result, Exactly the same as video generation

```
curl -XGET https://batch.pipelet.net/queue/wan-s2v-premium/requests/13
```
