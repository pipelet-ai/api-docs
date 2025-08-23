## Upscale API Quickstart

Upscale is just like video generation, but it takes an image and upscale it to a higher resolution.

## Base URL

- Production: `https://batch.pipelet.net`

## Endpoints (quick reference)

- Create request: `POST /queue/seedvr2-upscale`
- Check status: `GET /queue/seedvr2-upscale/requests/:requestId/status`
- Cancel request: `PUT /queue/seedvr2-upscale/requests/:requestId/cancel`
- Fetch result: `GET /queue/seedvr2-upscale/requests/:requestId`

## Request Format

### 1) Create a request

```
curl -XPOST `https://batch.pipelet.net/queue/seedvr2-upscale` \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer {your-bearer-token}' \
  -d '{"data_uri": "(base64 encoded image)"}'
```

Or for higher resolution **This may lead to OOM and failure**
```json
{
  "data_uri": "(base64 encoded image)",
  "new_resolution": 1080
}
```
### 2) Poll for status, like video generation

```
curl https://batch.pipelet.net/queue/seedvr2-upscale/requests/12/status
```

### 3) Cancel a queued request

```
curl -XPUT https://batch.pipelet.net/queue/seedvr2-upscale/requests/16/cancel
```

### 4) Fetch the result

```
curl -XGET https://batch.pipelet.net/queue/seedvr2-upscale/requests/13
```

