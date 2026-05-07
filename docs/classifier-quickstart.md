# Classifier API

This document describes the content classifier endpoint at `https://api.pipelet.ai/fal/classifier/tags`.

Use this endpoint to classify an image (or prompt text) into a small set of content tags. It is useful for routing requests, filtering inputs, or applying content policies before sending jobs to other endpoints.

Note: This route requires an authenticated request (middleware `requireUser`).

## Base URL

- Production: `https://api.pipelet.ai`

## Endpoint

- Classify content: `POST /fal/classifier/tags`

## Request

POST `https://api.pipelet.ai/fal/classifier/tags`

Body: JSON with at least one image field or a prompt.

Image fields (any of the following, each optional):
- `image_url`
- `image_uri`
- `data_uri`
- `second_image_url` / `second_image_uri`
- `third_image_url` / `third_image_uri`

Each image field accepts an image URL. You can supply multiple fields to classify several images in one call.

Other fields:
- `prompt` (optional, string) — text prompt to classify.

You must provide at least one image field or a `prompt` (or both).

Example:
```bash
curl -H "Authorization: Bearer $API_GATEWAY_USER_TOKEN" \
  https://api.pipelet.ai/fal/classifier/tags \
  -d '{"image_url": "https://resource.pipelet.net/images/test/two_girls_beer.jpg"}'
```

## Response

The endpoint returns a `tags` array containing exactly three labels, one from each of the following categories:

- Subject: `adult` or `children`
- Style: `realistic` or `fiction`
- Safety: `sfw` or `nsfw`

Example response:
```json
{
  "tags": [
    "adult",
    "realistic",
    "sfw"
  ]
}
```

## Errors

- `400` — no `image_url` or `prompt` was provided.
- `500` — classification failed.
- `503` — classifier service is not configured.
