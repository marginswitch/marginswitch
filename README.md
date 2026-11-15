# MarginSwitch

**Stop bleeding Stripe subscription margin on flat-rate AI whales, `past_due` zombies, and runaway agent retry loops.**

MarginSwitch is a `<15MB` stateless Go reverse proxy that sits between your AI application (`OpenAI`, `Anthropic`, `Vercel AI SDK`) and upstream LLM APIs, coupling live Stripe subscription revenue with real-time per-customer token COGS:

- **Unpaid / Canceled Zombie Guard**: Blocks `past_due` and `canceled` Stripe customers before tokens leave your server (`HTTP 402`).
- **Seamless Auto-Downgrade (Zero Customer Churn)**: Automatically shifts negative-margin power users to compatible lower-cost models (`claude-opus-4-6 -> claude-sonnet-4-5 / claude-haiku-4-5`, `gpt-5 -> gpt-5-mini`) across multi-turn stateless conversations, including `adaptive` thinking adaptation and `400` raw-replay fail-open safety.
- **Zero-Alloc `64-bit` SimHash Loop Breaker**: Detects runaway single & parallel tool-call retry loops in `<4µs` (`0 B/op` heap allocations) while ignoring `32KB+` MCP `tools` schemas.
- **Free `Shadow Mode`**: Observe-only deployment that posts a daily morning Slack receipt showing exact dollar leakage before you enable active protection.

## Self-Hosted Quickstart (`docker compose`)

```yaml
services:
  marginswitch:
    image: ghcr.io/marginswitch/proxy:latest
    ports:
      - "8080:8080"
    environment:
      PORT: "8080"
      MS_PROXY_TOKEN: "${MS_PROXY_TOKEN}"
      MS_SHADOW_MODE: "1"
    deploy:
      resources:
        limits:
          memory: 256M
```

Point your application environment variables at MarginSwitch (zero SDK installation required):

```bash
OPENAI_BASE_URL=https://your-proxy-host/p/${MS_PROXY_TOKEN}/openai/v1
ANTHROPIC_BASE_URL=https://your-proxy-host/p/${MS_PROXY_TOKEN}/anthropic
```

Pass the customer identifier per request via `x-ms-customer-id: <cus_id | user_id | email>`.
