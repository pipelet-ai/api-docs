# Video Info API

This document describes the video info endpoint at `https://api.pipelet.ai/video-info`.

Use this endpoint to probe a remote video and get its technical metadata (dimensions, fps, duration, codecs, audio info, etc.). The video is downloaded, inspected with `ffprobe`, then discarded — no copy is stored.

Note: This route requires an authenticated request (middleware `requireUser`).

## Base URL

- Production: `https://api.pipelet.ai`

## Endpoint

- Probe a video: `POST /video-info`

## Request

POST `https://api.pipelet.ai/video-info`

Body:
- `video_url` (required, string) — publicly accessible URL of the video to probe.

Example:
```bash
curl -XPOST https://api.pipelet.ai/video-info \
  -H "Authorization: Bearer $API_GATEWAY_USER_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"video_url": "https://example.com/clip.mp4"}'
```

## Response

`200 OK` with a JSON object describing the video and (if present) its primary audio stream.

Example response:
```json
{
  "width": 1920,
  "height": 1080,
  "fps": 29.97,
  "duration_seconds": 62.5,
  "total_frames": 1873,
  "video_codec": "h264",
  "has_audio": true,
  "audio_sample_rate": 48000,
  "audio_codec": "aac",
  "audio_channels": 2,
  "metaData": {
    "format": { "encoder": "Lavf58.29.100", "title": "..." },
    "video":  { "language": "und" },
    "audio":  { "language": "und" }
  }
}
```

| Field | Type | Notes |
|-------|------|-------|
| `width`, `height` | integer | Video dimensions in pixels. |
| `fps` | number\|null | Average frame rate (falls back to `r_frame_rate` if unavailable). |
| `duration_seconds` | number\|null | Duration of the video stream (falls back to container duration). |
| `total_frames` | integer\|null | Frame count from stream metadata, or `round(fps × duration)` if missing. |
| `video_codec` | string\|null | e.g. `h264`, `hevc`, `vp9`. |
| `has_audio` | boolean | `true` if at least one audio stream is present. |
| `audio_sample_rate` | integer\|null | Sampling rate in Hz (e.g. `44100`, `48000`). `null` if no audio. |
| `audio_codec` | string\|null | e.g. `aac`, `opus`. `null` if no audio. |
| `audio_channels` | integer\|null | Channel count. `null` if no audio. |
| `metaData` | object | Optional. Container/stream tag dictionaries (`format`, `video`, `audio`), included only when present and JSON-serializable. |

## Errors

- `401` — invalid token.
- `403` — missing `Authorization` header.
- `422` — failed to download the video, or no video stream was found.
- `500` — `ffprobe` failed.

Error body:
```json
{ "detail": "<human-readable message>" }
```
