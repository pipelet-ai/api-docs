# Webhook is a simple way to receive notifications about events

This document describes the webhook details in `https://api.pipelet.ai/fal/queue/` and `https://batch.pipelet.net/queue/` routes.

The webhook is triggered when the job status changes.

## Creating Webhook

There are two ways to create a webhook:

1. adding the webhook URL as a query parameter, with ?fal_webhook=<your-webhook-url>
```
curl 'https://batch.pipelet.net/queue/wan22-fast-t2v?fal_webhook=https://echo.hetzner1.pipelet.ai/' \
    -H "Authorization: Bearer <your-api-key>" \
    -H "Content-Type: application/json" \
    -d '{"prompt": "A photo of an astronaut riding a horse."}'
```
2. passing in the webhook URL as a header 
```
curl 'https://batch.pipelet.net/queue/wan22-fast-t2v' \
    -H "Authorization: Bearer <your-api-key>" \
    -H "Content-Type: application/json" \
    -H "x-pipelet-webhook: https://echo.hetzner1.pipelet.ai/" \
    -d '{"prompt": "A photo of an astronaut riding a horse."}'
```

## Using the Webhook

The webhook will be triggered when the job status changes. Your endpoint will receive a POST request with the JSON body.

The JSON body schema will be like below:
```
{
  "eventType": "acquired",
  "queueName": "free",
  "jobId": 437852,
  "workerName": "k8s_pipelet-v2-batch-premium-worker-559f48bc58-lz862_203.168.252.2_251227090447_ojfl",
  "leaseExpiry": 1766867867109,
  "acquiredAt": 1766867822109,
  "updatedAt": 1766867822109,
  "webhookUrl": "https://echo.hetzner1.pipelet.ai/"
}
```
### Event Types
The eventType would be one of the following:
- `acquired`: when the job is acquired by a worker. It means the job is processing and the count down timer can start
- `completed`: when the job is completed.
- `failed`: when the job failed.
- `cancelled`: when the job is cancelled.
- `retried`: when the job is retried.

### Job Result

The job result will be passed in when the job is completed. It would be exactly like you have visited the `/result` endpoint to fetch results
```
{
  "eventType": "completed",
  "video": {
    "data_uri": "https://prod-batch-files.ae2b0858dbcfcff39cc58bac85b7c66d.r2.cloudflarestorage.com/outputs/free/437852_0_WanVideo2_2_I2V_00150.mp4?X-Amz-Expires=7200&X-Amz-Date=20251227T203935Z&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=0c9db542c165ecde3eae49725be07d03%2F20251227%2Fauto%2Fs3%2Faws4_request&X-Amz-SignedHeaders=host&X-Amz-Signature=94e8e3d507137daa9e1dc4fac8319e7c58d3a5db635faac00f35dd0dba7ebb71",
    "data_url": "https://prod-batch-files.ae2b0858dbcfcff39cc58bac85b7c66d.r2.cloudflarestorage.com/outputs/free/437852_0_WanVideo2_2_I2V_00150.mp4?X-Amz-Expires=7200&X-Amz-Date=20251227T203935Z&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=0c9db542c165ecde3eae49725be07d03%2F20251227%2Fauto%2Fs3%2Faws4_request&X-Amz-SignedHeaders=host&X-Amz-Signature=94e8e3d507137daa9e1dc4fac8319e7c58d3a5db635faac00f35dd0dba7ebb71"
  }
}
```

### Extra note on retries

Right now the retry is not implemented yet. It will be supported in the future. If at the moment the job is completed, and your endpoint is not ready to receive the result, you can try to fetch the result using the `/result` endpoint.

