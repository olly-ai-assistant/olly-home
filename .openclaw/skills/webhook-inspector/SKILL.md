# Webhook Inspector

Inspect, debug, and analyze webhooks — both receiving and sending. Use when asked to "inspect a webhook", "debug webhook", "test webhook", "capture webhook requests", "analyze webhook payload", or when troubleshooting integration issues with webhooks.

## When to Use This

- Debugging a webhook endpoint that's not firing
- Inspecting incoming webhook payloads
- Testing outgoing webhook delivery
- Validating webhook signatures
- Parsing and formatting webhook payloads
- Monitoring webhook health/reliability

## Receiving Webhooks

### 1. Capture with netcat (quick)

```bash
# Listen for webhook (any method, any path)
nc -l -p 8080

# With timestamp
ncat -l -k -p 8080 --recv-only | tee webhook-$(date +%s).raw
```

### 2. Capture with socat

```bash
# Mirror to console + file
socat -u TCP-LISTEN:8080,fork,reuseaddr SYSTEM:'socat - STDIO'

# Full dump with headers
socat -u TCP-LISTEN:8080,crlf,binary,fork,reuseaddr OPEN:/tmp/wh-$(date +%s).raw,creat,trunc
```

### 3. Use a temporary tunnel (for external webhooks)

```bash
# Cloudflare tunnel (quick)
cloudflared tunnel --url http://localhost:8080

# Ngrok
ngrok http 8080

# Serveo
ssh -o StrictHostKeyChecking=no -R 80:localhost:8080 serveo.net
```

### 4. Python receiver (full control)

```python
from http.server import HTTPServer, BaseHTTPRequestHandler
import json, datetime

class Handler(BaseHTTPRequestHandler):
    def do_POST(self):
        length = int(self.headers.get('Content-Length', 0))
        body = self.rfile.read(length)
        ts = datetime.datetime.now().isoformat()
        
        with open(f'/tmp/webhook-{ts}.txt', 'w') as f:
            f.write(f"=== {ts} ===\n")
            f.write(f"Path: {self.path}\n")
            f.write(f"Headers: {dict(self.headers)}\n")
            f.write(f"Body: {body.decode()}\n")
        
        self.send_response(200)
        self.end_headers()
        self.wfile.write(b'OK')
    
    def log_message(self, fmt, *args):
        print(f"[WH] {fmt % args}")

HTTPServer(('0.0.0.0', 8080), Handler).serve_forever()
```

## Inspecting Payloads

### Parse and pretty-print

```bash
# JSON (curl example)
curl -s -X POST http://localhost:8080/webhook \
  -H "Content-Type: application/json" \
  -d '{"event":"order.created","data":{"id":123}}' | jq .

# From file
cat webhook-raw.txt | python3 -c "import sys,json; print(json.dumps(json.loads(sys.stdin.read()), indent=2))"

# Extract specific field
cat payload.json | jq '.event, .data.id, .data.customer.email'
```

### Compare two webhook payloads

```bash
diff <(jq -S . webhook1.json) <(jq -S . webhook2.json)
```

## Verifying Signatures

### GitHub Webhooks

```python
import hmac, hashlib, os

def verify_github_signature(payload_bytes, signature, secret):
    expected = 'sha256=' + hmac.new(
        secret.encode(),
        payload_bytes,
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(expected, signature)

# Usage
sig = request.headers.get('X-Hub-Signature-256')
if not verify_github_signature(body, sig, GITHUB_WEBHOOK_SECRET):
    abort(403)
```

### Stripe Webhooks

```python
import stripe

def verify_stripe_signature(payload, sig_header, secret):
    return stripe.Webhook.construct_event(payload, sig_header, secret)
```

### Slack / Discord

```bash
# Slack signing secret
echo -n "v0:timestamp:body" | openssl dgst -sha256 -hmac "SIGNING_SECRET" | cut -d' ' -f2

# Discord - check X-Signature-Ed25519 and X-Signature-Timestamp headers
```

## Sending Test Webhooks

### cURL

```bash
curl -X POST https://example.com/webhook \
  -H "Content-Type: application/json" \
  -H "X-Test-Event: true" \
  -d '{"event":"test","data":{"key":"value"}}'
```

### Python

```python
import requests, json

resp = requests.post(
    'http://localhost:8080/webhook',
    json={'event': 'test.order.created', 'data': {'order_id': 999}},
    headers={'X-Api-Key': 'test-secret'}
)
print(f"Status: {resp.status_code}, Body: {resp.text}")
```

### Repeat/loop a webhook

```bash
# 10 times with 1s delay
for i in $(seq 1 10); do
  curl -s -X POST http://localhost:8080/webhook \
    -H "Content-Type: application/json" \
    -d "{\"seq\":$i,\"ts\":\"$(date -Iseconds)\"}"
  sleep 1
done
```

## Webhook Debugging Checklist

| Check | Command/Action |
|-------|----------------|
| Endpoint reachable? | `curl -I https://your-endpoint.com/webhook` |
| Correct HTTP method? | POST vs GET — check provider docs |
| Content-Type correct? | Most expect `application/json` |
| Signature verified? | Check `X-Hub-Signature`, `X-Signature` headers |
| Timeout too short? | Increase from default (5s → 30s) |
| SSL/TLS issues? | Test with `-k` flag or check cert |
| Retries happening? | Webhook logs often show duplicate deliveries |
| Event type correct? | Check `event` field mapping in provider |

## Tools

- **Hookbin** (by RequestBin): `https://hookbin.com` — capture + inspect webhooks online
- **Webhook.site**: `https://webhook.site` — instant unique endpoint
- **ngrok**: Tunnel local server to public URL
- **cloudflared**: Cloudflare tunnel (no account needed)
- **jq**: Query/transform JSON payloads
- **mitmproxy**: Full-featured HTTP proxy for debugging

## Common Issues

| Problem | Fix |
|---------|-----|
| Webhook not firing | Check provider dashboard for delivery attempts + failures |
| 403 Forbidden | Signature verification failing — check secret |
| 200 but no action | Your app returned OK but didn't process — check logs |
| Duplicate deliveries | Webhook already succeeded but provider retried — use idempotency keys |
| Payload empty | Some providers expect raw body — don't JSON.parse twice |
| Timeout | Provider gave up — return 200 fast, process async |
