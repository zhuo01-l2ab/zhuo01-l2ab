# Y-API — 15 models, one key, pay per token

**English** · [简体中文](README.zh.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Español](README.es.md) · [Deutsch](README.de.md) · [Português](README.pt.md)

> **Disclosure:** I work on Y-API, so everything below is a first-party claim — check it against the machine-readable sources at the bottom. Every link here is a plain link: no referral code, no tracking parameter.

**Y-API** is an OpenAI-compatible LLM gateway. 15 models from 7 vendors (DeepSeek, Qwen, Z.ai, Moonshot AI, Tencent, Xiaomi, OpenAI) sit behind one `base_url`, one key and one balance. There is no monthly fee, no minimum spend, and no plan tier — credit is deducted per token and nothing else is charged.

## Try it in 30 seconds

```bash
curl https://api.y-api.bestvirtualgoods.com/v1/chat/completions \
  -H "Authorization: Bearer $Y_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"deepseek/deepseek-v4-flash-0731","messages":[{"role":"user","content":"Say hi in one sentence."}]}'
```

Existing OpenAI code only needs the base URL changed:

```python
from openai import OpenAI
client = OpenAI(base_url="https://api.y-api.bestvirtualgoods.com/v1", api_key=Y_API_KEY)
```

```js
import OpenAI from "openai";
const client = new OpenAI({ baseURL: "https://api.y-api.bestvirtualgoods.com/v1", apiKey: Y_API_KEY });
```

The `/v1` suffix is already part of the base URL — adding it twice returns a 400. Anthropic clients work too: `POST /v1/messages` runs the same catalog off the same key and accepts the key in `x-api-key`.

## What it costs

- Prices are quoted in **account credit**. Right now **$1 paid = $20 credit**; it is a limited-time rate that reverts to **1:10**, and no end date has been announced. **Cash price = credit ÷ 20.**
- `$1` of sign-up credit, no card required. When credit runs out the API returns 429 and recovers immediately after a top-up — keys stay valid.
- Cheapest paid model: `deepseek/deepseek-v4-flash-0731` at $0.15 in / $0.30 out per 1M tokens in credit, i.e. **$0.0075 per 1M input tokens in cash**.
- Three models are priced at zero: `deepseek/deepseek-v4-flash`, `tencent/hy3`, `xiaomi/mimo-v2.5`.

| Model | In | Out | Model | In | Out |
| --- | --- | --- | --- | --- | --- |
| `deepseek/deepseek-v4-flash` | 0 | 0 | `moonshotai/kimi-k3` | 3.00 | 15.00 |
| `deepseek/deepseek-v4-flash-0731` | 0.15 | 0.30 | `tencent/hy3` | 0 | 0 |
| `deepseek/deepseek-v4.1-flash` | 0.20 | 1.00 | `xiaomi/mimo-v2.5` | 0 | 0 |
| `deepseek/deepseek-v4-pro` | 0.50 | 1.00 | `openai/gpt-5.6-luna` | 0.30 | 1.30 |
| `qwen/qwen3.8-flash` | 0.20 | 0.50 | `openai/gpt-5.6-terra` | 2.00 | 12.00 |
| `z-ai/glm-5.2` | 1.40 | 4.40 | `openai/gpt-5.6-sol` | 5.00 | 30.00 |
| `z-ai/glm-5.3` | 1.40 | 5.00 | `openai/gpt-6-astra` | 10.00 | 50.00 |
| `z-ai/glm-5.3-flash` | 0.15 | 0.50 | | | |

Credit per 1M tokens. Divide by the current top-up rate for the cash price. Live table: <https://y-api.bestvirtualgoods.com/pricing.json>

## Where it is the wrong choice

- **Not available to users in mainland China.** This is stated on the site itself; both domains currently resolve to Cloudflare IPs with no mainland node.
- **No cached-input rate.** Input is billed at one rate whether or not the prompt repeats. On 1 of the 11 models where we verified a vendor price, the vendor's own cached-input rate is below our cash price — for a cache-heavy workload that makes us the more expensive option. The per-model flag is `vs_official.vendor_cheaper_on_cached_input`.
- **No SLA.** The terms commit to no availability metric, and this is a resale gateway: an upstream outage passes straight through.
- **Small surface area.** Three endpoints. Anthropic clients that count tokens before sending hit a 404 on `/v1/messages/count_tokens`, and `POST /messages` returns a response `id` without the `msg_` prefix.
- **Model coverage is narrower than a full aggregator.** 15 models, not hundreds. A model ID outside the catalog returns 503 `model_not_found`.

## Machine-readable sources

| What | Where |
| --- | --- |
| Model catalog + prices, no key needed | <https://y-api.bestvirtualgoods.com/models.json> |
| Credit price, cash price, vendor list price, per-model flags | <https://y-api.bestvirtualgoods.com/pricing.json> |
| OpenAPI 3.1 spec (the 3 endpoints that exist) | <https://y-api.bestvirtualgoods.com/openapi.json> |
| Every error status, and which ones the SDKs retry | <https://y-api.bestvirtualgoods.com/docs/errors> |
| Live checks + 30-day history | <https://y-api.bestvirtualgoods.com/status> |
| Quickstarts in cURL / Python / Node | <https://y-api.bestvirtualgoods.com/docs> |

Of the 12 paid models, 11 carry a vendor list price we verified and cited — the comparison is checkable rather than asserted.

## Also in this account

**[y-api-price-tracker](https://github.com/zhuo01-l2ab/y-api-price-tracker)** — a dependency-free script that renders the published price file as a cash-price table, refreshed weekly by CI. It recomputes `credit_price / top_up.quota_rate` rather than copying `cash_price`, and fails loudly if the two disagree. If you want price changes as a machine-readable diff instead of a rendered page, start there.

---

*Facts on this page were read from `public/models.json` and `public/pricing.json` (snapshot 2026-09-17, checked 2026-09-18). Prices move; the JSON links above are the live source.*
