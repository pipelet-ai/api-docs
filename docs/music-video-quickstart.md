# Music Video API Quickstart

Music Video is a workflow that generates a lip-synced video from an image and audio file. Right now it splits the audio into segments, generates video for each segment, and combines them into a final video. You can also patch (regenerate) specific time ranges of an existing video.

## Base URL

- Production: `https://api.pipelet.ai`

## Endpoints (quick reference)

- **Start workflow**: `POST /v1/workflow/music-video/start`
- **Check status**: `GET /v1/workflow/music-video/status/:instanceId`
- **Query status**: `GET /v1/workflow/music-video/:instanceId`
- **Patch (regenerate segments)**: `POST /v1/workflow/music-video/patch`
- **Check patch status**: `GET /v1/workflow/music-video/patch/status/:instanceId`
- **Get patch history**: `GET /v1/workflow/music-video/patch/history/:instanceId`

---

## Request Format

### 1) Start a Music Video Workflow

Creates a new music video generation job.

**Request body** (JSON):
- `prompt` (required): Text prompt describing the video style/content. E.g., "Cute and Calm Capybara"
- `audio_url` (required): URL to the audio file (MP3, WAV, etc.)
- `image_url` (required): URL to the input image

```bash
curl -XPOST "https://api.pipelet.ai/v1/workflow/music-video/start" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {your-bearer-token}" \
  -d '{
    "prompt": "Cute and Calm Capybara",
    "audio_url": "https://resource.pipelet.net/demos/music_videos/capybara/capybara_itself.mp3",
    "image_url": "https://resource.pipelet.net/demos/music_videos/capybara/capybara_itself2.png"
}'
```

**Response:**
```json
{
  "id": "76c1a2cb-2c87-4d43-a597-b8903d1d0d42",
  "details": {
    "status": "running",
    ...
  }
}
```

---

### 2) Poll for Status

Check the status of a running or completed workflow.

```bash
curl "https://api.pipelet.ai/v1/workflow/music-video/status/76c1a2cb-2c87-4d43-a597-b8903d1d0d42" \
  -H "Authorization: Bearer {your-bearer-token}"
```

**Response (complete):**
```json
{
  "status": "complete",
  "error": null,
  "output": {
    "audio_url": "https://prod-batch-files.ae2b0858dbcfcff39cc58bac85b7c66d.r2.cloudflarestorage.com/uploads/music-video/...",
    "input_image": {
      "download_url": "https://long-term-storage.ae2b0858dbcfcff39cc58bac85b7c66d.r2.cloudflarestorage.com/workflows/music-video/76xxx...",
      "r2_uri": "r2://long-term-storage/workflows/music-video/76c1a2cb-2c87-4d43-a597-b8903d1d0d42/input_image.png"
    },
    "segments": [
      {
        "index": 0,
        "start_ms": 0,
        "end_ms": 10000,
        "download_url": "https://long-term-storage.../segment_0_0_10000.mp4?...",
        "r2_uri": "r2://long-term-storage/workflows/music-video/76c1a2cb-2c87-4d43-a597-b8903d1d0d42/segment_0_0_10000.mp4",
        "input_image_r2_uri": "r2://long-term-storage/workflows/music-video/76c1a2cb-2c87-4d43-a597-b8903d1d0d42/segment_0_0_10000.png",
        "input_image_url": "https://long-term-storage.../segment_0_0_10000.png?..."
      },
      {
        "index": 1,
        "start_ms": 10000,
        "end_ms": 20000,
        ...
      }
    ],
    "combined": {
      "download_url": "https://long-term-storage.../combined.mp4?...",
      "r2_uri": "r2://long-term-storage/workflows/music-video/.../combined.mp4"
    }
  }
}
```

---

### 3) Patch (Regenerate Specific Segments)

Regenerate specific time ranges of an existing music video. This is useful when certain segments didn't turn out well and you want to retry them without regenerating the entire video.

**Request body** (JSON):
- `instance_id` (required): The instance ID of the original music video (or a previous patch)
- `ranges` (required): Array of time ranges to regenerate, each with `start_ms` and `end_ms`

```bash
curl -XPOST "https://api.pipelet.ai/v1/workflow/music-video/patch" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {your-bearer-token}" \
  -d '{
    "instance_id": "76c1a2cb-2c87-4d43-a597-b8903d1d0d42",
    "ranges": [
      { "start_ms": 0, "end_ms": 10000 },
      { "start_ms": 20000, "end_ms": 30000 }
    ]
}'
```

**Response:**
```json
{
  "id": "a1b2c3d4-5678-90ab-cdef-1234567890ab",
  "details": {
    "status": "running",
    ...
  }
}
```

---

### 4) Check Patch Status

Check the status of a patch workflow. Returns the merged segment list combining original and patched segments.

```bash
curl "https://api.pipelet.ai/v1/workflow/music-video/patch/status/a1b2c3d4-5678-90ab-cdef-1234567890ab" \
  -H "Authorization: Bearer {your-bearer-token}"
```

**Response (complete):**
```json
{
  "status": "complete",
  "output": {
    "original_instance_id": "76c1a2cb-2c87-4d43-a597-b8903d1d0d42",
    "ranges": [
      { "start_ms": 0, "end_ms": 10000 },
      { "start_ms": 20000, "end_ms": 30000 }
    ],
    "regenerated_segments": [
      {
        "index": 0,
        "audio_start_ms": 0,
        "audio_length_ms": 10000,
        "status": "succeeded",
        "r2_uri": "r2://long-term-storage/workflows/music-video/.../patch/.../segment_0_0_10000.mp4",
        "input_image_r2_uri": "r2://long-term-storage/workflows/music-video/.../patch/.../segment_0_0_10000.png"
      },
      ...
    ],
    "segments": [
      {
        "index": 0,
        "start_ms": 0,
        "end_ms": 10000,
        "r2_uri": "r2://...",
        "input_image_r2_uri": "r2://..."
      },
      ...
    ],
    "combined": {
      "download_url": "https://...",
      "r2_uri": "r2://long-term-storage/workflows/music-video/.../patch/.../combined.mp4"
    }
  }
}
```

---

### 5) Get Patch History

Retrieve the full patch history for a music video, showing all patch operations applied.

```bash
curl "https://api.pipelet.ai/v1/workflow/music-video/patch/history/76c1a2cb-2c87-4d43-a597-b8903d1d0d42" \
  -H "Authorization: Bearer {your-bearer-token}"
```

**Response:**
```json
{
  "rootInstanceId": "76c1a2cb-2c87-4d43-a597-b8903d1d0d42",
  "nodes": [
    {
      "instanceId": "a1b2c3d4-5678-90ab-cdef-1234567890ab",
      "workflowName": "music-video-patch",
      "status": "succeeded",
      "createdAt": 1706300000000,
      "updatedAt": 1706300500000,
      "parentInstanceId": "76c1a2cb-2c87-4d43-a597-b8903d1d0d42",
      "rootInstanceId": "76c1a2cb-2c87-4d43-a597-b8903d1d0d42"
    }
  ]
}
```

---

## Notes

- **Segment duration**: Audio is split into 10-second chunks. Each segment generates a corresponding video clip.
- **Patching**: You can chain patches—use a patch instance ID as the `instance_id` for subsequent patches.
- **Download URLs**: Pre-signed URLs expire after 2 hours. Query the status endpoint again to get fresh URLs.
