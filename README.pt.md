# Y-API — 15 modelos, uma chave, cobrança por token

[English](README.md) · [简体中文](README.zh.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Español](README.es.md) · [Deutsch](README.de.md) · **Português**

> **Divulgação:** eu trabalho na Y-API, portanto tudo abaixo é uma afirmação de primeira parte; confira nas fontes legíveis por máquina no final. Todos os links são links limpos: sem código de indicação e sem parâmetro de rastreamento.

**Y-API** é um gateway de LLM compatível com OpenAI. 15 modelos de 7 fornecedores (DeepSeek, Qwen, Z.ai, Moonshot AI, Tencent, Xiaomi, OpenAI) atrás de um único `base_url`, uma chave e um saldo. Não há mensalidade, nem consumo mínimo, nem níveis de plano: o crédito é descontado por token e nada mais é cobrado.

## Teste em 30 segundos

```bash
curl https://api.y-api.bestvirtualgoods.com/v1/chat/completions \
  -H "Authorization: Bearer $Y_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"deepseek/deepseek-v4-flash-0731","messages":[{"role":"user","content":"Cumprimente em uma frase."}]}'
```

O código OpenAI existente só precisa trocar a URL base:

```python
from openai import OpenAI
client = OpenAI(base_url="https://api.y-api.bestvirtualgoods.com/v1", api_key=Y_API_KEY)
```

```js
import OpenAI from "openai";
const client = new OpenAI({ baseURL: "https://api.y-api.bestvirtualgoods.com/v1", apiKey: Y_API_KEY });
```

O sufixo `/v1` já faz parte da URL base; repeti-lo devolve um 400. Clientes da Anthropic também funcionam: `POST /v1/messages` atende ao mesmo catálogo com a mesma chave e aceita a chave em `x-api-key`.

## Quanto custa

- Os preços são expressos em **crédito da conta**. Hoje, **$1 pago = $20 em crédito**; é uma taxa por tempo limitado que volta a **1:10**, e nenhuma data de término foi anunciada. **Preço em dinheiro = crédito ÷ 20.**
- `$1` de crédito no cadastro, sem cartão. Quando o crédito acaba, a API devolve 429 e volta a funcionar imediatamente após uma recarga; as chaves continuam válidas.
- Modelo pago mais barato: `deepseek/deepseek-v4-flash-0731`, a $0,15 de entrada / $0,30 de saída por 1M de tokens em crédito, ou **$0,0075 por 1M de tokens de entrada em dinheiro**.
- Três modelos têm preço zero: `deepseek/deepseek-v4-flash`, `tencent/hy3`, `xiaomi/mimo-v2.5`.

| Modelo | Entrada | Saída | Modelo | Entrada | Saída |
| --- | --- | --- | --- | --- | --- |
| `deepseek/deepseek-v4-flash` | 0 | 0 | `moonshotai/kimi-k3` | 3.00 | 15.00 |
| `deepseek/deepseek-v4-flash-0731` | 0.15 | 0.30 | `tencent/hy3` | 0 | 0 |
| `deepseek/deepseek-v4.1-flash` | 0.20 | 1.00 | `xiaomi/mimo-v2.5` | 0 | 0 |
| `deepseek/deepseek-v4-pro` | 0.50 | 1.00 | `openai/gpt-5.6-luna` | 0.30 | 1.30 |
| `qwen/qwen3.8-flash` | 0.20 | 0.50 | `openai/gpt-5.6-terra` | 2.00 | 12.00 |
| `z-ai/glm-5.2` | 1.40 | 4.40 | `openai/gpt-5.6-sol` | 5.00 | 30.00 |
| `z-ai/glm-5.3` | 1.40 | 5.00 | `openai/gpt-6-astra` | 10.00 | 50.00 |
| `z-ai/glm-5.3-flash` | 0.15 | 0.50 | | | |

Crédito por 1M de tokens. Divida pela taxa de recarga atual para obter o preço em dinheiro. Tabela ao vivo: <https://y-api.bestvirtualgoods.com/pricing.json>

## Quando não é a escolha certa

- **Não está disponível para usuários da China continental.** O próprio site declara isso; ambos os domínios apontam hoje para IPs da Cloudflare e não há nó na China continental.
- **Não há tarifa para entrada em cache.** A entrada é cobrada a uma única taxa, o prompt se repita ou não. Em 1 dos 11 modelos com preço de fornecedor verificado, a tarifa de entrada em cache do próprio fornecedor fica abaixo do nosso preço em dinheiro; para uma carga pesada em cache, isso nos torna mais caros. A flag por modelo é `vs_official.vendor_cheaper_on_cached_input`.
- **Não há SLA.** Os termos não assumem nenhuma métrica de disponibilidade, e este é um gateway de revenda: uma falha no upstream passa direto.
- **Superfície pequena.** Três endpoints. Clientes da Anthropic que contam tokens antes de enviar recebem 404 em `/v1/messages/count_tokens`, e `POST /messages` devolve um `id` de resposta sem o prefixo `msg_`.
- **Cobertura de modelos menor que a de um agregador completo.** 15 modelos, não centenas. Um ID fora do catálogo devolve 503 `model_not_found`.

## Fontes legíveis por máquina

| O quê | Onde |
| --- | --- |
| Catálogo de modelos e preços, sem chave | <https://y-api.bestvirtualgoods.com/models.json> |
| Preço em crédito, preço em dinheiro, preço oficial do fornecedor, flags por modelo | <https://y-api.bestvirtualgoods.com/pricing.json> |
| Especificação OpenAPI 3.1 (os 3 endpoints existentes) | <https://y-api.bestvirtualgoods.com/openapi.json> |
| Todos os códigos de erro e quais os SDKs repetem sozinhos | <https://y-api.bestvirtualgoods.com/docs/errors> |
| Verificações ao vivo e histórico de 30 dias | <https://y-api.bestvirtualgoods.com/status> |
| Inícios rápidos em cURL / Python / Node | <https://y-api.bestvirtualgoods.com/docs> |

Dos 12 modelos pagos, 11 trazem um preço oficial de fornecedor que verificamos e citamos — a comparação é verificável, não afirmada.

## Também nesta conta

**[y-api-price-tracker](https://github.com/zhuo01-l2ab/y-api-price-tracker)** — um script sem dependências que transforma o arquivo de preços publicado em uma tabela de preços em dinheiro, atualizada semanalmente pelo CI. Ele recalcula `credit_price / top_up.quota_rate` em vez de copiar `cash_price`, e falha alto se os dois divergirem. Se você quer as mudanças de preço como um diff legível por máquina, e não como uma página renderizada, comece por lá.

---

*Os dados desta página foram lidos de `public/models.json` e `public/pricing.json` (instantâneo 2026-09-17, conferido em 2026-09-18). Os preços mudam; os links JSON acima são a fonte viva.*
