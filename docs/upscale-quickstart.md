## Upscale API Quickstart

Upscale is just like video generation, but it takes an image and upscale it to a higher resolution.

## Base URL

- Production: `https://api.pipelet.ai`

## Endpoints (quick reference)

- Create request: `POST /queue/seedvr2-high-res`
- Check status: `GET /queue/seedvr2-high-res/requests/:requestId/status`
- Cancel request: `PUT /queue/seedvr2-high-res/requests/:requestId/cancel`
- Fetch result: `GET /queue/seedvr2-high-res/requests/:requestId`

## Request Format

### 1) Create a request

```
curl -XPOST `https://api.pipelet.ai/queue/seedvr2-high-res` \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer {your-bearer-token}' \
  -d '{"data_uri": "(base64 encoded image)"}'
```

Or for higher resolution
```json
{
  "data_uri": "(base64 encoded video)",
  "data_url": "(S3 Pre-signed URL for video)", // Only one of data_uri and data_url can be provided
  "new_resolution": 1080
}
```
### 2) Poll for status, like video generation

```
curl https://api.pipelet.ai/queue/seedvr2-high-res/requests/12/status
```

### 3) Cancel a queued request

```
curl -XPUT https://api.pipelet.ai/queue/seedvr2-high-res/requests/16/cancel
```

### 4) Fetch the result

```
curl -XGET https://api.pipelet.ai/queue/seedvr2-high-res/requests/13
```

