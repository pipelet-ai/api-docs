## Text To Speech API Quickstart {#tts-quickstart}

Sound to video is just like video generation, but it takes an audio, a prompt and generate a video.
It is very popular in the case to animate a character to lip-sync to a given audio.

## Base URL

- Production: `https://api.pipelet.net`

## Endpoints (quick reference)

- Create request: `POST /fal/queue/vibevoice`
- Check status: `GET /fal/queue/vibevoice/requests/:requestId/status` . For example `GET /fal/queue/vibevoice/requests/mw2yeeka39oxvtouc8j8z/status`
- Cancel request: `PUT /fal/queue/vibevoice/requests/:requestId/cancel` . For example `PUT /fal/queue/vibevoice/requests/mw2yeeka39oxvtouc8j8z/cancel`
- Fetch result: `GET /fal/queue/vibevoice/requests/:requestId` . For example `GET /fal/queue/vibevoice/requests/mw2yeeka39oxvtouc8j8z`

## Request Format

### 1) Create a request
Request body is a JSON object, with the following fields:
- `audio_data_uri`: base64 encoded audio
- `prompt`: the prompt to generate the video
- `priority`: integer, the priority of the job. The bigger the number, the higher priority we have. You can use whatever integer you have.

```
curl -XPOST https://api.pipelet.net/fal/queue/vibevoice \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer {your-bearer-token}' \
  -d '{"audio_data_uri": "(base64 encoded audio)", "prompt": "Hey honey, do you have time for dinner tonight?"}'
```

The response will be a JSON object, with the following fields:
```
{
  "request_id": "mw2yeeka39oxvtouc8j8z",
  "response_url": "https://api.pipelet.net/fal/queue/vibevoice/requests/mw2yeeka39oxvtouc8j8z",
  "status_url": "https://api.pipelet.net/fal/queue/vibevoice/requests/mw2yeeka39oxvtouc8j8z/status",
  "cancel_url": "https://api.pipelet.net/fal/queue/vibevoice/requests/mw2yeeka39oxvtouc8j8z/cancel"
}
```
They are the same as the video generation API, showing you where to poll for status, cancel the request, or fetch the result.

### 2) Poll for status, nearly same as video generation, with the path having `fal` in front of it

```
curl https://api.pipelet.net/fal/queue/vibevoice/requests/mw2yeeka39oxvtouc8j8z/status -H "Authorization: Bearer {your-bearer-token}"
```

### 3) Cancel a queued request, nearly same as video generation, with the path having `fal` in front of it

```
curl -XPUT https://api.pipelet.net/fal/queue/vibevoice/requests/mw2yeeka39oxvtouc8j8z/cancel -H "Authorization: Bearer {your-bearer-token}"
```

### 4) Fetch the result, nearly same as video generation, with the path having `fal` in front of it

```
curl -XGET https://api.pipelet.net/fal/queue/vibevoice/requests/mw2yeeka39oxvtouc8j8z -H "Authorization: Bearer {your-bearer-token}"
```

### Result JSON

```
{
  "audio":{
    "data_uri":"ZkxhQwAAACIJAAkA(......)Sq6iKS+4UQSRC1OsmqmAAAAOaK",
    "content_type":"audio/flac",
    "file_name":"3E5WhW_ComfyUI_00001_.flac"
  }
}
```

## Predefined voices

The users should upload their own voice to clone from. We provide a list of predefined voices for the convenience, but their quality is not as good as the user-uploaded voices.

Disclaimer: the voices are a replication of the OSS https://github.com/devnen/Chatterbox-TTS-Server/tree/main/voices

You can get the list of voices by visiting https://api.pipelet.net/api/chatterbox/voices
The response will be a JSON like below:
```
{
  "voices": {
    "Abigail": "/voices/Abigail.wav",
    "Adrian": "/voices/Adrian.wav",
    "Alexander": "/voices/Alexander.wav",
    "Alice": "/voices/Alice.wav",
    "Austin": "/voices/Austin.wav",
    "Axel": "/voices/Axel.wav",
    "Connor": "/voices/Connor.wav",
    ....
  }
}
```
For example for item `Abigail` you can download the voice by visiting https://api.pipelet.net/voices/Abigail.wav
