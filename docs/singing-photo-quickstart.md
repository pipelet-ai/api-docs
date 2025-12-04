## Singing Photo API Quickstart

Singing photo is just like video generation, but it takes an image, a prompt and generate a video.
It is very popular in the case to animate a character to lip-sync to a given audio.

## Base URL

- Production: `https://api.pipelet.ai`

## Endpoints (quick reference)

- Create request: `POST /fal/queue/singing-photo`
- Check status: `GET /fal/queue/singing-photo/requests/:requestId/status`
- Cancel request: `PUT /fal/queue/singing-photo/requests/:requestId/cancel`
- Fetch result: `GET /fal/queue/singing-photo/requests/:requestId`

## Request Format

### 1) Create a request
Request body is a JSON object, with the following fields:
- `data_uri`: base64 encoded image
- `image_url`: S3 Pre-signed URL for image, or public URL with https access.
- `prompt`: the prompt to generate the video. E.g., "The girl sings a sad song with tears and emotion"
- `duration`: optional, the target/maximum duration of the video in seconds, default is 10 seconds. If lyrics is too short it will be filled with sound so it will be very close to 10 seconds.
- `min_height`: integer, the minimum height of the video. Right now we recommend 480 otherwise the speed would be slow. 
- `priority`: integer, the priority of the job. The bigger the number, the higher priority we have. You can use whatever integer you have.

```
curl -XPOST `https://api.pipelet.ai/fal/queue/singing-photo` \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer {your-bearer-token}' \
  -d '{"image_url": "https://resource.pipelet.net/images/sketch_art_girl.png", "prompt": "The girl sings a sad song with tears and emotion", "duration": 10, "min_height": 480, "priority": 40}'
```

### 2) Poll for status, Exactly the same as video generation

```
curl https://api.pipelet.ai/fal/queue/singing-photo/requests/5e8f8ukijqeire/status -H "Authorization: Bearer {your-bearer-token}"
```

### 3) Cancel a queued request, Exactly the same as video generation

```
curl -XPUT https://api.pipelet.ai/fal/queue/singing-photo/requests/5e8f8ukijqeire/cancel -H "Authorization: Bearer {your-bearer-token}"
```

### 4) Fetch the result, Exactly the same as video generation

```
curl -XGET https://api.pipelet.ai/fal/queue/singing-photo/requests/5e8f8ukijqeire -H "Authorization: Bearer {your-bearer-token}"
```
