# Invite API

## Prerequisites
- Node.js 18+ recommended
- npm
- Access to the configured Gnosis Chain RPC 

## Configuration
Create a `.env` file in the project root:

```
API_KEY=            
TRUST_PRIVATE_KEY=        
SLACK_WEBHOOK_URL=
```

## Install & Run
```
npm install
npm run dev    
# or
npm start       
```

The server listens on `http://localhost:3000`.

## Rate limits & queueing
- 100 requests/sec token bucket.
- Max 500 queued jobs; additional requests receive HTTP 429.
- Jobs time out after 5 minutes; failures can be retried by resubmitting the same address.

## API

### POST /onboard
Queues an address for onboarding.

Headers:
- `Content-Type: application/json`
- `x-api-key: $API_KEY`

Body:
```
{ "address": "0xabc123... (checksum address)" }
```


Example:
```
curl -X POST http://localhost:3000/onboard \
  -H "Content-Type: application/json" \
  -H "x-api-key: $API_KEY" \
  -d '{"address":"0x742d35Cc6634C0532925a3b844Bc454e4438f44e"}'
```

Job payload shape:
```
{
  "jobId": "uuid",
  "status": "queued | processing | submitted | confirmed | failed",
  "address": "0x...",
  "result": {
    "address": "0x...",
    "isHuman": false,
    "invite": {
      "inviteId": "123",
      "claimTxHash": "0x...",
      "transferTxHash": "0x..."
    },
    "transactions": {
      "gnosisPayGroup": "0x...",
      "dublinGroup": "0x..."
    }
  },
  "createdAt": 1730000000000,
  "updatedAt": 1730000005000
}
```
`result` is `null` until the job finishes. `invite` is `null` when the address is already marked human.

### GET /status/:jobId
Polls a job created by `/onboard`.

- Headers:
  - `x-api-key: $API_KEY`

- `200 OK` with the job payload.
- `404 Not Found` if the `jobId` is unknown.

Example:
```
curl http://localhost:3000/status/5e9a8c7d-3c6f-4b5e-a9d4-123456789abc \
  -H "x-api-key: $API_KEY"
```

### GET /health
Simple readiness probe.

- `200 OK` and `{ "message": "OK" }`
