# Y-API — 15개 모델, 키 하나, 토큰 단위 과금

[English](README.md) · [简体中文](README.zh.md) · [日本語](README.ja.md) · **한국어** · [Español](README.es.md) · [Deutsch](README.de.md) · [Português](README.pt.md)

> **이해관계 공개:** 저는 Y-API를 만들고 있는 사람입니다. 따라서 아래 내용은 모두 당사자의 주장이며, 하단의 기계 판독 가능한 출처에서 직접 확인하시기 바랍니다. 여기 있는 모든 링크는 추천 코드나 추적 파라미터가 없는 순수한 링크입니다.

**Y-API**는 OpenAI 호환 LLM 게이트웨이입니다. 7개 벤더(DeepSeek, Qwen, Z.ai, Moonshot AI, Tencent, Xiaomi, OpenAI)의 15개 모델이 하나의 `base_url`, 하나의 키, 하나의 잔액 뒤에 있습니다. 월 요금도, 최소 사용 금액도, 요금제 등급도 없으며 토큰 단위로 크레딧이 차감될 뿐 그 외에 청구되는 것은 없습니다.

## 30초면 실행됩니다

```bash
curl https://api.y-api.bestvirtualgoods.com/v1/chat/completions \
  -H "Authorization: Bearer $Y_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"deepseek/deepseek-v4-flash-0731","messages":[{"role":"user","content":"한 문장으로 인사해줘."}]}'
```

기존 OpenAI 코드는 base URL만 바꾸면 됩니다:

```python
from openai import OpenAI
client = OpenAI(base_url="https://api.y-api.bestvirtualgoods.com/v1", api_key=Y_API_KEY)
```

```js
import OpenAI from "openai";
const client = new OpenAI({ baseURL: "https://api.y-api.bestvirtualgoods.com/v1", apiKey: Y_API_KEY });
```

`/v1`은 base URL에 이미 포함되어 있습니다. 두 번 붙이면 400이 반환됩니다. Anthropic 클라이언트도 동작합니다. `POST /v1/messages`는 같은 키로 같은 카탈로그를 실행하고 `x-api-key` 헤더도 받습니다.

## 가격

- 가격은 **계정 크레딧** 기준입니다. 현재 **$1 결제 시 $20 크레딧**이지만 이는 한정 레이트이며 종료 후 **1:10**으로 돌아갑니다. 종료일은 발표되지 않았습니다. **현금 가격 = 크레딧 ÷ 20**입니다.
- 가입 시 $1 크레딧, 카드 불필요. 크레딧이 소진되면 API는 429를 반환하고, 충전 후 즉시 복구됩니다. 키는 유효하게 유지됩니다.
- 가장 저렴한 유료 모델: `deepseek/deepseek-v4-flash-0731`, 100만 토큰당 입력 $0.15 / 출력 $0.30(크레딧). **현금으로는 입력 100만 토큰당 $0.0075**입니다.
- 세 모델은 가격이 0입니다: `deepseek/deepseek-v4-flash`, `tencent/hy3`, `xiaomi/mimo-v2.5`.

| 모델 | 입력 | 출력 | 모델 | 입력 | 출력 |
| --- | --- | --- | --- | --- | --- |
| `deepseek/deepseek-v4-flash` | 0 | 0 | `moonshotai/kimi-k3` | 3.00 | 15.00 |
| `deepseek/deepseek-v4-flash-0731` | 0.15 | 0.30 | `tencent/hy3` | 0 | 0 |
| `deepseek/deepseek-v4.1-flash` | 0.20 | 1.00 | `xiaomi/mimo-v2.5` | 0 | 0 |
| `deepseek/deepseek-v4-pro` | 0.50 | 1.00 | `openai/gpt-5.6-luna` | 0.30 | 1.30 |
| `qwen/qwen3.8-flash` | 0.20 | 0.50 | `openai/gpt-5.6-terra` | 2.00 | 12.00 |
| `z-ai/glm-5.2` | 1.40 | 4.40 | `openai/gpt-5.6-sol` | 5.00 | 30.00 |
| `z-ai/glm-5.3` | 1.40 | 5.00 | `openai/gpt-6-astra` | 10.00 | 50.00 |
| `z-ai/glm-5.3-flash` | 0.15 | 0.50 | | | |

단위는 100만 토큰당 크레딧 가격입니다. 현금 가격은 현재 충전 레이트로 나누면 됩니다. 실시간 표: <https://y-api.bestvirtualgoods.com/pricing.json>

## 선택하면 안 되는 경우

- **중국 본토 사용자에게는 제공되지 않습니다.** 사이트 자체가 그렇게 명시하고 있습니다. 두 도메인은 현재 모두 Cloudflare IP를 가리키며 본토 노드는 없습니다.
- **캐시 입력 요율이 없습니다.** 프롬프트가 반복되는지와 무관하게 입력은 동일 요율로 청구됩니다. 벤더 가격을 확인한 11개 모델 중 1개는 벤더 자체의 캐시 입력 요율이 저희 현금 가격보다 낮습니다. 캐시 비중이 높은 워크로드에서는 저희 쪽이 더 비쌉니다. 모델별 플래그는 `vs_official.vendor_cheaper_on_cached_input`입니다.
- **SLA가 없습니다.** 이용약관은 어떤 가용성 지표도 약속하지 않습니다. 이곳은 재판매 게이트웨이이므로 업스트림 장애가 그대로 전달됩니다.
- **표면적이 좁습니다.** 엔드포인트가 3개뿐입니다. 전송 전에 토큰 수를 세는 Anthropic 클라이언트는 `/v1/messages/count_tokens`에서 404를 만납니다. 또한 `POST /messages`의 응답 `id`에는 `msg_` 접두사가 없습니다.
- **모델 커버리지는 대형 애그리게이터에 못 미칩니다.** 수백 개가 아니라 15개입니다. 카탈로그 밖 모델 ID는 503 `model_not_found`를 반환합니다.

## 기계 판독 가능한 출처

| 내용 | 주소 |
| --- | --- |
| 모델 카탈로그와 가격(키 불필요) | <https://y-api.bestvirtualgoods.com/models.json> |
| 크레딧 가격, 현금 가격, 벤더 공식 가격, 모델별 플래그 | <https://y-api.bestvirtualgoods.com/pricing.json> |
| OpenAPI 3.1 명세(존재하는 3개 엔드포인트) | <https://y-api.bestvirtualgoods.com/openapi.json> |
| 모든 오류 코드와 SDK가 자동 재시도하는 항목 | <https://y-api.bestvirtualgoods.com/docs/errors> |
| 실시간 점검과 30일 이력 | <https://y-api.bestvirtualgoods.com/status> |
| cURL / Python / Node 퀵스타트 | <https://y-api.bestvirtualgoods.com/docs> |

유료 모델 12개 중 11개에는 검증하고 출처를 명시한 벤더 공식 가격이 붙어 있습니다. 이 비교는 주장이 아니라 검증 가능한 수치입니다.

## 이 계정의 다른 저장소

**[y-api-price-tracker](https://github.com/zhuo01-l2ab/y-api-price-tracker)** — 의존성 없는 스크립트로 공개 가격 파일을 현금 가격 표로 렌더링하고, CI가 매주 갱신합니다. `cash_price`를 복사하지 않고 `credit_price / top_up.quota_rate`를 다시 계산하며, 둘이 다르면 즉시 실패합니다. 렌더링된 페이지가 아니라 diff로 읽히는 가격 변동 기록이 필요하다면 여기서 시작하세요.

---

*이 페이지의 사실은 `public/models.json`과 `public/pricing.json`(스냅샷 2026-09-17, 확인 2026-09-18)에서 읽은 것입니다. 가격은 변동됩니다. 위 JSON 링크가 실시간 출처입니다.*
