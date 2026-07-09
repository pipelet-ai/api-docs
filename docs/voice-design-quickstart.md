# Voice Design (MOSS-TTS) API

This document describes the voice design endpoints in the `https://api.pipelet.ai/fal/` route.

Voice Design is a text-to-speech model that takes two text inputs — the speech text to be spoken, and a plain natural-language description of how the voice should sound — and produces an audio file. Unlike voice cloning, no reference audio is required: you simply describe the voice you want.

Use these endpoints to create jobs, poll their status, cancel if still queued, and fetch the resulting audio.

Note: All routes require an authenticated request (middleware `requireUser`).

## Base URL

- Production: `https://api.pipelet.ai`


## Endpoints (quick reference)

- Create request: `POST /fal/queue/moss-tts-voice-design`
- Check status: `GET /fal/queue/moss-tts-voice-design/requests/:requestId/status`
- Cancel request: `PUT /fal/queue/moss-tts-voice-design/requests/:requestId/cancel`
- Fetch result: `GET /fal/queue/moss-tts-voice-design/requests/:requestId`

---

## 1) Create a request

POST `https://api.pipelet.ai/fal/queue/moss-tts-voice-design`

Body: JSON inputs for the model.
- `prompt` (required) is the speech text to be spoken.
- `voice_description` (required) is a plain natural-language description of how the voice should sound, e.g. tone, accent, mood, character.
- `priority` (optional, integer, default is 0) for priority in the queue, the bigger the number, the higher priority we have.

Example:
```bash
curl -XPOST https://api.pipelet.ai/fal/queue/moss-tts-voice-design -H "Authorization: Bearer <your-api-key>" -d '{
    "prompt": "Muhahahaha! You are fired!",
    "voice_description": "A dark, menacing villain voice in a role-play RPG game"
}'

```
It will return something like below:
```json
{
  "request_id": "rn5r12q9pdu6yvb3nxty",
  "response_url": "https://api.pipelet.ai/fal/queue/moss-tts-voice-design/requests/rn5r12q9pdu6yvb3nxty",
  "status_url": "https://api.pipelet.ai/fal/queue/moss-tts-voice-design/requests/rn5r12q9pdu6yvb3nxty/status",
  "cancel_url": "https://api.pipelet.ai/fal/queue/moss-tts-voice-design/requests/rn5r12q9pdu6yvb3nxty/cancel"
}
```
Notes:
- `request_id` is your handle for this job.
- Use `status_url` to poll for progress.
- Use `response_url` to fetch the completed result.
- Use `cancel_url` to cancel if the job has not started yet.


We will query the status of the job using the `status_url`.
```bash
curl "https://api.pipelet.ai/fal/queue/moss-tts-voice-design/requests/rn5r12q9pdu6yvb3nxty/status" -H "Authorization: Bearer $API_GATEWAY_USER_TOKEN"
```
Example response for status:

```json
{
  "status": "IN_PROGRESS",
  "queue_position": 0,
  "statusMessages": {
    "message": "Generating audio",
    "estimatedTotalTimeSeconds": 20
  },
  "response_url": "https://api.pipelet.ai/fal/queue/moss-tts-voice-design/requests/rn5r12q9pdu6yvb3nxty"
}
```
When the status changes to "COMPLETED", we can fetch the result using the `response_url`.
```bash
curl "https://api.pipelet.ai/fal/queue/moss-tts-voice-design/requests/rn5r12q9pdu6yvb3nxty" -H "Authorization: Bearer $API_GATEWAY_USER_TOKEN"
```
Example response for result:
```json
{
  "audio": {
    "data_uri": "https://prod-batch-files.ae2b0858dbcfcff39cc58bac85b7c66d.r2.cloudflarestorage.com/outputs/moss_tts_voice_design_api/3E5WhW_ComfyUI_00001_.mp3?X-Amz-Expires=7200&X-Amz-Date=20251025T051931Z&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=9ca0f6abba71e6073fe4c8a9b21997d5%2F20251025%2Fauto%2Fs3%2Faws4_request&X-Amz-SignedHeaders=host&X-Amz-Signature=1031cdf308033f8cd7f9600379d16b6ac969bdb412986bc310c138ef33e1f88f",
    "content_type": "audio/mpeg",
    "file_name": "3E5WhW_ComfyUI_00001_.mp3"
  }
}
```
