# Y-API — 15 モデル、1 つのキー、トークン単位の従量課金

[English](README.md) · [简体中文](README.zh.md) · **日本語** · [한국어](README.ko.md) · [Español](README.es.md) · [Deutsch](README.de.md) · [Português](README.pt.md)

> **開示：** 私は Y-API の開発に関わっています。したがって以下の内容はすべて当事者による主張です。末尾の機械可読な情報源でご自身で検証してください。記載しているリンクはすべて素のリンクで、紹介コードもトラッキングパラメータも含まれていません。

**Y-API** は OpenAI 互換の LLM ゲートウェイです。7 社（DeepSeek、Qwen、Z.ai、Moonshot AI、Tencent、Xiaomi、OpenAI）の 15 モデルが 1 つの `base_url`、1 つのキー、1 つの残高の背後にあります。月額料金も最低利用金額もプラン階梯もなく、トークンごとにクレジットが引かれるだけで、それ以外の請求はありません。

## 30 秒で試す

```bash
curl https://api.y-api.bestvirtualgoods.com/v1/chat/completions \
  -H "Authorization: Bearer $Y_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"deepseek/deepseek-v4-flash-0731","messages":[{"role":"user","content":"一言で挨拶して。"}]}'
```

既存の OpenAI のコードは base URL を書き換えるだけです：

```python
from openai import OpenAI
client = OpenAI(base_url="https://api.y-api.bestvirtualgoods.com/v1", api_key=Y_API_KEY)
```

```js
import OpenAI from "openai";
const client = new OpenAI({ baseURL: "https://api.y-api.bestvirtualgoods.com/v1", apiKey: Y_API_KEY });
```

`/v1` は base URL に最初から含まれています。二重に付けると 400 が返ります。Anthropic のクライアントも動作します。`POST /v1/messages` は同じキーで同じカタログを実行し、`x-api-key` ヘッダーでも認証できます。

## 料金

- 価格は**アカウントクレジット**建てです。現在は**$1 の入金で $20 のクレジット**ですが、これは期間限定レートで、終了後は **1:10** に戻ります。終了日は未発表です。**現金価格 = クレジット ÷ 20** です。
- 登録時に $1 分のクレジット、カード不要。クレジットが尽きると API は 429 を返し、入金後ただちに復旧します。キーは無効になりません。
- 最も安い有料モデル：`deepseek/deepseek-v4-flash-0731`、100 万トークンあたり入力 $0.15 / 出力 $0.30（クレジット）。**現金では入力 100 万トークンあたり $0.0075** です。
- 3 モデルはゼロ価格です：`deepseek/deepseek-v4-flash`、`tencent/hy3`、`xiaomi/mimo-v2.5`。

| モデル | 入力 | 出力 | モデル | 入力 | 出力 |
| --- | --- | --- | --- | --- | --- |
| `deepseek/deepseek-v4-flash` | 0 | 0 | `moonshotai/kimi-k3` | 3.00 | 15.00 |
| `deepseek/deepseek-v4-flash-0731` | 0.15 | 0.30 | `tencent/hy3` | 0 | 0 |
| `deepseek/deepseek-v4.1-flash` | 0.20 | 1.00 | `xiaomi/mimo-v2.5` | 0 | 0 |
| `deepseek/deepseek-v4-pro` | 0.50 | 1.00 | `openai/gpt-5.6-luna` | 0.30 | 1.30 |
| `qwen/qwen3.8-flash` | 0.20 | 0.50 | `openai/gpt-5.6-terra` | 2.00 | 12.00 |
| `z-ai/glm-5.2` | 1.40 | 4.40 | `openai/gpt-5.6-sol` | 5.00 | 30.00 |
| `z-ai/glm-5.3` | 1.40 | 5.00 | `openai/gpt-6-astra` | 10.00 | 50.00 |
| `z-ai/glm-5.3-flash` | 0.15 | 0.50 | | | |

単位は 100 万トークンあたりのクレジット価格。現金価格は現在の入金レートで割ってください。最新の表：<https://y-api.bestvirtualgoods.com/pricing.json>

## 選ぶべきでないケース

- **中国本土のユーザーには提供していません。** サイト自身がそのように明示しています。両ドメインは現在 Cloudflare の IP を指しており、本土側にノードはありません。
- **キャッシュ入力の料率がありません。** プロンプトが反復するかどうかに関係なく、入力は同一料率で課金されます。ベンダー価格を確認した 11 モデルのうち 1 つでは、ベンダー自身のキャッシュ入力料率が当方の現金価格を下回ります。キャッシュ多用のワークロードでは当方のほうが高くつきます。モデルごとのフラグは `vs_official.vendor_cheaper_on_cached_input` です。
- **SLA がありません。** 利用規約は可用性指標を一切約束していません。ここは転売ゲートウェイなので、上流の障害はそのまま透過します。
- **表面積が小さい。** エンドポイントは 3 つだけです。送信前にトークン数を数える Anthropic クライアントは `/v1/messages/count_tokens` で 404 になります。また `POST /messages` の応答 `id` には `msg_` プレフィックスが付きません。
- **モデル網羅は大手アグリゲーターに劣ります。** 数百ではなく 15 モデルです。カタログ外のモデル ID は 503 `model_not_found` になります。

## 機械可読な情報源

| 内容 | URL |
| --- | --- |
| モデルカタログと価格（キー不要） | <https://y-api.bestvirtualgoods.com/models.json> |
| クレジット価格、現金価格、ベンダー公式価格、モデル別フラグ | <https://y-api.bestvirtualgoods.com/pricing.json> |
| OpenAPI 3.1 仕様（存在する 3 エンドポイント） | <https://y-api.bestvirtualgoods.com/openapi.json> |
| 全エラーコードと、SDK が自動リトライするもの | <https://y-api.bestvirtualgoods.com/docs/errors> |
| リアルタイム監視と 30 日履歴 | <https://y-api.bestvirtualgoods.com/status> |
| cURL / Python / Node のクイックスタート | <https://y-api.bestvirtualgoods.com/docs> |

有料 12 モデルのうち 11 モデルには、検証して出典を明記したベンダー公式価格が付いています。この比較は主張ではなく検証可能なものです。

---

*本ページの記述は `public/models.json` と `public/pricing.json`（スナップショット 2026-09-17、確認日 2026-09-18）から読み出したものです。価格は変動します。上記の JSON リンクが最新の情報源です。*
