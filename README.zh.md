# Y-API — 15 个模型，一把 key，按 token 付费

[English](README.md) · **简体中文** · [日本語](README.ja.md) · [한국어](README.ko.md) · [Español](README.es.md) · [Deutsch](README.de.md) · [Português](README.pt.md)

> **利益声明：** 我在做 Y-API，所以下面每一条都是第一方说法，请用文末的机器可读数据源自行核对。所有链接都是干净链接——没有邀请码，没有追踪参数。

**Y-API** 是一个 OpenAI 兼容的大模型网关。7 家厂商（DeepSeek、Qwen、Z.ai、Moonshot AI、腾讯、小米、OpenAI）的 15 个模型，共用一个 `base_url`、一把 key、一份余额。没有月费，没有最低消费，没有套餐档位——按 token 扣额度，除此之外不收任何费用。

## 30 秒跑通

```bash
curl https://api.y-api.bestvirtualgoods.com/v1/chat/completions \
  -H "Authorization: Bearer $Y_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"deepseek/deepseek-v4-flash-0731","messages":[{"role":"user","content":"用一句话打招呼。"}]}'
```

已有的 OpenAI 代码只需要改 base URL：

```python
from openai import OpenAI
client = OpenAI(base_url="https://api.y-api.bestvirtualgoods.com/v1", api_key=Y_API_KEY)
```

```js
import OpenAI from "openai";
const client = new OpenAI({ baseURL: "https://api.y-api.bestvirtualgoods.com/v1", apiKey: Y_API_KEY });
```

`/v1` 已经包含在 base URL 里了，再拼一次会返回 400。Anthropic SDK 也能用：`POST /v1/messages` 走同一份模型目录、同一把 key，并接受 `x-api-key` 头。

## 价格

- 站内标价是**账户额度**。当前**充 1 美元得 20 美元额度**，这是限时汇率，之后**恢复到 1:10**，结束日期未公布。**现金价 = 额度价 ÷ 20。**
- 注册送 1 美元额度，不需要绑卡。额度用尽时 API 返回 429，充值后立即恢复，key 不会失效。
- 最便宜的付费模型：`deepseek/deepseek-v4-flash-0731`，每百万 token 输入 $0.15 / 输出 $0.30 额度，折合**现金每百万输入 token $0.0075**。
- 三个模型定价为零：`deepseek/deepseek-v4-flash`、`tencent/hy3`、`xiaomi/mimo-v2.5`。

| 模型 | 输入 | 输出 | 模型 | 输入 | 输出 |
| --- | --- | --- | --- | --- | --- |
| `deepseek/deepseek-v4-flash` | 0 | 0 | `moonshotai/kimi-k3` | 3.00 | 15.00 |
| `deepseek/deepseek-v4-flash-0731` | 0.15 | 0.30 | `tencent/hy3` | 0 | 0 |
| `deepseek/deepseek-v4.1-flash` | 0.20 | 1.00 | `xiaomi/mimo-v2.5` | 0 | 0 |
| `deepseek/deepseek-v4-pro` | 0.50 | 1.00 | `openai/gpt-5.6-luna` | 0.30 | 1.30 |
| `qwen/qwen3.8-flash` | 0.20 | 0.50 | `openai/gpt-5.6-terra` | 2.00 | 12.00 |
| `z-ai/glm-5.2` | 1.40 | 4.40 | `openai/gpt-5.6-sol` | 5.00 | 30.00 |
| `z-ai/glm-5.3` | 1.40 | 5.00 | `openai/gpt-6-astra` | 10.00 | 50.00 |
| `z-ai/glm-5.3-flash` | 0.15 | 0.50 | | | |

单位：每百万 token 的额度价。除以当期充值汇率即为现金价。实时表：<https://y-api.bestvirtualgoods.com/pricing.json>

## 什么情况下不该选它

- **不面向中国大陆用户提供服务。** 站点自己就这么声明；两个域名目前都解析到 Cloudflare IP，大陆没有节点。
- **没有缓存命中的单独价格。** 无论 prompt 是否重复，输入都按同一个价计费。在我们核实过厂商价的 11 个模型中，有 1 个厂商自己的缓存命中价低于我们的现金价——对重缓存的负载来说，我们反而更贵。逐模型的标记是 `vs_official.vendor_cheaper_on_cached_input`。
- **没有 SLA。** 条款未承诺任何可用性指标；这是转售网关，上游故障会直接透传。
- **接口面很小。** 只有三个端点。会先数 token 再发请求的 Anthropic 客户端会在 `/v1/messages/count_tokens` 上撞到 404；`POST /messages` 返回的响应 `id` 不带 `msg_` 前缀。
- **模型覆盖不如聚合平台。** 只有 15 个模型，不是几百个。目录外的模型 ID 会返回 503 `model_not_found`。

## 机器可读的数据源

| 内容 | 地址 |
| --- | --- |
| 模型目录与价格，无需 key | <https://y-api.bestvirtualgoods.com/models.json> |
| 额度价、现金价、厂商官方价、逐模型标记 | <https://y-api.bestvirtualgoods.com/pricing.json> |
| OpenAPI 3.1 规范（现有的 3 个端点） | <https://y-api.bestvirtualgoods.com/openapi.json> |
| 全部错误状态码，以及哪些会被 SDK 自动重试 | <https://y-api.bestvirtualgoods.com/docs/errors> |
| 实时探测与 30 天历史 | <https://y-api.bestvirtualgoods.com/status> |
| cURL / Python / Node 快速上手 | <https://y-api.bestvirtualgoods.com/docs> |

12 个付费模型中有 11 个带我们核实并标注出处的厂商官方价——这个对比是可复核的，不是自说自话。

---

*本页事实读自 `public/models.json` 与 `public/pricing.json`（快照 2026-09-17，核对于 2026-09-18）。价格会变，上面的 JSON 链接是实时来源。*
