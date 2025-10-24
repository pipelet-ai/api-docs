## Text To Speech API Quickstart {#tts-quickstart}

Sound to video is just like video generation, but it takes an audio, a prompt and generate a video.
It is very popular in the case to animate a character to lip-sync to a given audio.

## Base URL

- Production: `https://api.pipelet.ai`

## Endpoints (quick reference)

- Create request: `POST /fal/queue/indextts2`
- Check status: `GET /fal/queue/indextts2/requests/:requestId/status` . For example `GET /fal/queue/indextts2/requests/mw2yeeka39oxvtouc8j8z/status`
- Cancel request: `PUT /fal/queue/indextts2/requests/:requestId/cancel` . For example `PUT /fal/queue/indextts2/requests/mw2yeeka39oxvtouc8j8z/cancel`
- Fetch result: `GET /fal/queue/indextts2/requests/:requestId` . For example `GET /fal/queue/indextts2/requests/mw2yeeka39oxvtouc8j8z`

## Request Format

### 1) Create a request
Request body is a JSON object, with the following fields:
- `audio_data_uri`: optional, base64 encoded audio
- `audio_url`: optional, url to the audio
- `prompt`: the prompt to generate the video
- `priority`: integer, the priority of the job. The bigger the number, the higher priority we have. You can use whatever integer you have.
- `predefined_voice`: optional, the name of the predefined voice to use. For example `Abigail`, `Adrian`, `Alexander`, `Alice`, `Austin`, `Axel`, `Connor`, etc. 
- `emotion_text`: optional, the text to describe the emotion for the voice. For example `happy`, `sad`, `angry`, etc. We can input any text form to describe the emotion.
- `emotion_vector`: optional, the vector to describe the emotion for the voice. The vector is defined as a dictionary: `{'anger': 0.1, 'happy': 0.2, 'sad': 0.3}`. The choices are Happy, Angry, Sad, Fear, Hate, Love, Surprise, Neutral
```
curl -XPOST https://api.pipelet.ai/fal/queue/indextts2-extreme-length \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer {your-bearer-token}' \
  -d '{"audio_data_uri": "(base64 encoded audio)", "prompt": "Hey honey, do you have time for dinner tonight?"}'
```

The response will be a JSON object, with the following fields:
```
{
  "request_id": "mw2yeeka39oxvtouc8j8z",
  "response_url": "https://api.pipelet.ai/fal/queue/indextts2-extreme-length/requests/mw2yeeka39oxvtouc8j8z",
  "status_url": "https://api.pipelet.ai/fal/queue/indextts2-extreme-length/requests/mw2yeeka39oxvtouc8j8z/status",
  "cancel_url": "https://api.pipelet.ai/fal/queue/indextts2-extreme-length/requests/mw2yeeka39oxvtouc8j8z/cancel"
}
```
They are the same as the video generation API, showing you where to poll for status, cancel the request, or fetch the result.

### 2) Poll for status, nearly same as video generation, with the path having `fal` in front of it

```
curl https://api.pipelet.ai/fal/queue/indextts2-extreme-length/requests/mw2yeeka39oxvtouc8j8z/status -H "Authorization: Bearer {your-bearer-token}"
```

### 3) Cancel a queued request, nearly same as video generation, with the path having `fal` in front of it

```
curl -XPUT https://api.pipelet.ai/fal/queue/indextts2-extreme-length/requests/mw2yeeka39oxvtouc8j8z/cancel -H "Authorization: Bearer {your-bearer-token}"
```

### 4) Fetch the result, nearly same as video generation, with the path having `fal` in front of it

```
curl -XGET https://api.pipelet.ai/fal/queue/indextts2-extreme-length/requests/mw2yeeka39oxvtouc8j8z -H "Authorization: Bearer {your-bearer-token}"
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

You can get the list of voices by visiting https://api.pipelet.ai/api/chatterbox/voices
The response will be a JSON like below:
```
{
  "voices": {
    "Abigail": "https://resource.pipelet.net/voices/Abigail.wav",
    "Adrian": "https://resource.pipelet.net/voices/Adrian.wav",
    "Alexander": "https://resource.pipelet.net/voices/Alexander.wav",
    "Alice": "https://resource.pipelet.net/voices/Alice.wav",
    "Austin": "https://resource.pipelet.net/voices/Austin.wav",
    "Axel": "https://resource.pipelet.net/voices/Axel.wav",
    "Connor": "/voices/Connor.wav",
    ....
  }
}
```
For example for item `Abigail` you can download the voice by visiting https://resource.pipelet.net/voices/Abigail.wav
