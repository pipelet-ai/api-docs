## Gimm VFI Interpolate API Quickstart

Gimm VFI Interpolate is just like video generation, but it takes an video and interpolate it to a higher fps.

## Base URL

- Production: `https://api.pipelet.ai`

## Endpoints (quick reference)

- Create request: `POST /queue/gimm-vfi-interpolate`
- Check status: `GET /queue/gimm-vfi-interpolate/requests/:requestId/status`
- Cancel request: `PUT /queue/gimm-vfi-interpolate/requests/:requestId/cancel`
- Fetch result: `GET /queue/gimm-vfi-interpolate/requests/:requestId`

## Request Format

### 1) Create a request

```
curl -XPOST `https://api.pipelet.ai/fal/queue/gimm-vfi-interpolate` \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer {your-bearer-token}' \
  -d '{"data_uri": "(base64 encoded video)", "fps": 48}'
```

Or for higher resolution
```json
{
  "data_uri": "(base64 encoded video)",
  "data_url": "(S3 Pre-signed URL for video)", // Only one of data_uri and data_url can be provided
  "fps": 48 // only multiples of 16 are supported, like 16, 32, 48, 64, etc
}
```
### 2) Poll for status, like video generation

```
curl https://api.pipelet.ai/fal/queue/gimm-vfi-interpolate/requests/12/status
```

### 3) Cancel a queued request

```
curl -XPUT https://api.pipelet.ai/fal/queue/gimm-vfi-interpolate/requests/16/cancel
```

### 4) Fetch the result

```
curl -XGET https://api.pipelet.ai/fal/queue/gimm-vfi-interpolate/requests/13
```

