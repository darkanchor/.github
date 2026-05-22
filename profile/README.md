# darkanchor — nginx infrastructure for the AI era

darkanchor builds the richest nginx module ecosystem under Apache 2.0 — and the only nginx-native AI gateway.

---

## What we build

| Project | Modules | Language | License | Purpose |
|---------|---------|----------|---------|---------|
| [**nginz**](https://github.com/darkanchor/nginz) | 26 native modules | Zig | Apache 2.0 | Hot-path primitives: auth, rate limiting, WAF, health checks, Prometheus |
| [**nginz-njs**](https://github.com/darkanchor/nginz-njs) | 13 scripted modules | Gleam → njs | Apache 2.0 | Policy layer: routing, orchestration, feature flags, transforms |
| **nginz-token** | 11 modules | Zig + Gleam | BSL 1.1 → Apache 2.0 | AI Gateway: token-level rate limiting, cost tracking |

**39 modules.** All free. All open source. All plug into stock, unmodified nginx — no fork, no custom binary.

---

## Why this matters

| Capability | Stock nginx | nginz | NGINX Plus |
|-----------|-------------|-------|------------|
| Active health checks (HTTP) | ❌ | ✅ Free | ✅ $3,500/yr |
| Dynamic upstreams (no reload) | ❌ | ✅ Free | ✅ $3,500/yr |
| Sticky sessions | ❌ | ✅ Free | ✅ $3,500/yr |
| JWT / OIDC auth | ❌ | ✅ Free | ❌ |
| WAF / SQLi / XSS detection | ❌ | ✅ Free | ❌ |
| Prometheus metrics | ❌ | ✅ Free | ❌ |
| Consul service discovery | ❌ | ✅ Free | ❌ |
| PostgreSQL REST API | ❌ | ✅ Free | ❌ |

**We give you NGINX Plus features — free. Plus capabilities NGINX Plus doesn't have.** And when you need AI gateway features (token-level rate limiting, cost tracking), we have paid modules for that.

---

## Architecture

```
nginx (stock, unmodified)
├── nginz (Zig)         → Hot path: auth, counters, shared memory, WAF
└── nginz-njs (Gleam)   → Policy: routing, orchestration, transforms
```

No fork. No custom binary. Just `--add-module=nginz` and `js_import` directives. Works with nginx 1.30+.

---

## Our positioning

> **"NGINX Plus features, free. Plus Consul, Prometheus, and AI gateway"**

We're opinionated because focus ships faster than completeness.

---

## Get started

```bash
# Pull the Docker image
docker pull darkanchor/nginz:latest

# Or build from source
git clone https://github.com/darkanchor/nginz.git
cd nginz
git submodule init && git submodule update
zig build
```

→ [**Read the docs**](https://www.darkanchor.com/docs)

---

## AI Gateway — nginz-token

The first nginx-native AI gateway. Zero added latency. Your data stays local. Token-level control no SaaS proxy can match.

| Module | License | What it does |
|--------|------------|--------------|
| **llm-proxy** | BSL | Multi-provider routing (OpenAI, Anthropic, local) |
| **llm-auth** | BSL | API key validation |
| **llm-metrics** | BSL | Request counts, latency, error rates by model |
| **llm-ratelimit** | BSL | Per-user, per-key RPM + TPM rate limiting |
| **llm-cost** | BSL | Per-request cost tracking → PostgreSQL |
| **llm-cache** | BSL | Conservative cache policy surface for deciding eligibility, scope, and bypass reasons |
| **llm-security** | BSL | Prompt injection detection, PII filtering |
| **llm-fallback** | BSL | Cost-aware, latency-aware model switching |

---

## Build your own AI gateway — in 20 lines

```nginx
load_module modules/ngx_http_js_module.so;

http {
    js_import authz from /etc/nginx/njs/authz.js;
    js_import workflow from /etc/nginx/njs/workflow.js;

    upstream openai {
        server api.openai.com:443;
        nginz_healthcheck uri=/v1/models interval=5s;
    }

    upstream anthropic {
        server api.anthropic.com:443;
        nginz_healthcheck uri=/v1/messages interval=5s;
    }

    server {
        listen 443 ssl;
        
        location /v1/ {
            nginz_jwt $jwt_claims;
            js_content authz.route;
            proxy_pass $upstream;
            nginz_ratelimit zone=llm_zone rate=100r/m;
        }
    }
}
```

Basic routing, auth, health checks, and rate limiting. Free. Forever.

---

## Our principles

- **Apache 2.0 for open source.** No asterisks. No crippleware.
- **BSL for paid AI modules.** Source-visible. Converts to Apache 2.0 after 3 years.
- **No volume limits.** The open source proxy runs forever at any scale.

---

<p align="center">
  <sub>darkanchor · The infra anchor of the AI era · <a href="https://www.darkanchor.com">darkanchor.com</a></sub>
</p>
