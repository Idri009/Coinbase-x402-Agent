## Coinbase x402 + Hyperbolic Chat Completions API

Production-ready example demonstrating pay-per-request API access using the x402 open payment standard with Hyperbolic’s chat completions. Includes an Express server, payment middleware, a typed request/response schema, structured logging, and a client example that pays automatically using `x402-fetch`.

- Documentation: [x402 standard](https://x402.gitbook.io/x402)
- Models: [Hyperbolic Models](https://app.hyperbolic.ai/models)

### Architecture
- **Server**: `api/index.ts` (Express, `x402-express`, `winston`, `zod`, `helmet`, `cors`)
- **Client**: `client.ts` (`x402-fetch`, `viem` accounts)
- **Payment**: 402 flow enforced via middleware; client pays and retries automatically

---

## Prerequisites
- Node.js 20+
- PNPM (recommended) or npm
- A Base Mainnet account/private key with funds for payments
- Hyperbolic API key
- Coinbase Developer Platform API key pair (for `@coinbase/x402` facilitator)

## Environment Variables
Create a `.env` file or use the provided `.env.example` as a template:

```
# Address that receives payments (Base mainnet)
ADDRESS_MAINNET=0x...

# Hyperbolic API key (server-side only)
HYPERBOLIC_API_KEY=...

# Coinbase Developer Platform keys (used by facilitator)
CDP_API_KEY_ID=...
CDP_API_KEY_SECRET=...

# Client private key for paid requests (Base mainnet)
MAINNET_PRIVATE_KEY=0x...

# Optional
PORT=3000
ALLOWED_ORIGINS=https://yourapp.com,http://localhost:3000
LOG_LEVEL=info
```

## Setup
1) Install dependencies

```bash
pnpm install
```

2) Copy and edit environment

```bash
cp .env.example .env
# Fill in values as described above
```

## Run Locally
- Start the API locally (non-Vercel):

```bash
pnpm run dev
```

- Health and readiness:
```
GET /health   → basic heartbeat
GET /ready    → validates env and Hyperbolic availability
```

## API
- Base URL (example): `https://hyperbolic-x402.vercel.app`
- Endpoint: `POST /v1/chat/completions`

Required headers:
- `Content-Type: application/json`
- `Accept: application/json`
- `X-Request-ID: <uuid>`

Request body (OpenAI compatible):

```json
{
  "model": "meta-llama/Meta-Llama-3.1-405B-Instruct",
  "messages": [{ "role": "user", "content": "What is 1+1?" }],
  "max_tokens": 512,
  "temperature": 0.1,
  "top_p": 0.9,
  "stream": false
}
```

Response includes standard OpenAI-style payload and an `x-payment-response` header with payment confirmation metadata.

## Client Example
Run the scripted client which automatically pays via `x402-fetch`:

```bash
pnpm run client
```

The client prints the model response and logs the on-chain transaction hash after confirming via `POST /v1/transaction-log`.

## Deployment (Vercel)
This repository includes a `vercel.json` config and `@vercel/node` adapter.

```bash
pnpm run deploy        # production
pnpm run deploy:preview
```

Ensure environment variables are set in Vercel prior to deploying.

## Security Notes
- Never commit private keys. Use environment variables or secret managers.
- Restrict `ALLOWED_ORIGINS` in production.
- Keep `LOG_LEVEL` at `info` or higher; avoid logging sensitive data.

## Troubleshooting
- 400 Validation Error: ensure `model` and `messages` meet schema requirements.
- 402 Payment Required: expected for unpaid requests; client should auto-pay and retry.
- 500 Configuration Error: check missing envs via `GET /ready`.

## License
ISC