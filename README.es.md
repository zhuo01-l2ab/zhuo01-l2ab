# Y-API — 15 modelos, una clave, pago por token

[English](README.md) · [简体中文](README.zh.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · **Español** · [Deutsch](README.de.md) · [Português](README.pt.md)

> **Divulgación:** trabajo en Y-API, así que todo lo que sigue es una afirmación de primera mano; compruébalo con las fuentes legibles por máquina del final. Todos los enlaces son enlaces limpios: sin código de referido ni parámetros de seguimiento.

**Y-API** es una pasarela de LLM compatible con OpenAI. 15 modelos de 7 proveedores (DeepSeek, Qwen, Z.ai, Moonshot AI, Tencent, Xiaomi, OpenAI) detrás de un mismo `base_url`, una clave y un saldo. No hay cuota mensual, ni gasto mínimo, ni niveles de plan: el crédito se descuenta por token y no se cobra nada más.

## Pruébalo en 30 segundos

```bash
curl https://api.y-api.bestvirtualgoods.com/v1/chat/completions \
  -H "Authorization: Bearer $Y_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"deepseek/deepseek-v4-flash-0731","messages":[{"role":"user","content":"Saluda en una frase."}]}'
```

El código de OpenAI existente solo necesita cambiar la URL base:

```python
from openai import OpenAI
client = OpenAI(base_url="https://api.y-api.bestvirtualgoods.com/v1", api_key=Y_API_KEY)
```

```js
import OpenAI from "openai";
const client = new OpenAI({ baseURL: "https://api.y-api.bestvirtualgoods.com/v1", apiKey: Y_API_KEY });
```

El sufijo `/v1` ya forma parte de la URL base; duplicarlo devuelve un 400. Los clientes de Anthropic también funcionan: `POST /v1/messages` usa el mismo catálogo con la misma clave y acepta la clave en `x-api-key`.

## Cuánto cuesta

- Los precios se expresan en **crédito de cuenta**. Ahora mismo **$1 pagado = $20 de crédito**; es una tasa por tiempo limitado que volverá a **1:10**, y no se ha anunciado fecha de fin. **Precio en efectivo = crédito ÷ 20.**
- `$1` de crédito al registrarse, sin tarjeta. Cuando el crédito se agota la API devuelve 429 y se recupera de inmediato tras una recarga; las claves siguen válidas.
- El modelo de pago más barato: `deepseek/deepseek-v4-flash-0731`, a $0.15 de entrada / $0.30 de salida por millón de tokens en crédito, es decir **$0.0075 por millón de tokens de entrada en efectivo**.
- Tres modelos tienen precio cero: `deepseek/deepseek-v4-flash`, `tencent/hy3`, `xiaomi/mimo-v2.5`.

| Modelo | Entrada | Salida | Modelo | Entrada | Salida |
| --- | --- | --- | --- | --- | --- |
| `deepseek/deepseek-v4-flash` | 0 | 0 | `moonshotai/kimi-k3` | 3.00 | 15.00 |
| `deepseek/deepseek-v4-flash-0731` | 0.15 | 0.30 | `tencent/hy3` | 0 | 0 |
| `deepseek/deepseek-v4.1-flash` | 0.20 | 1.00 | `xiaomi/mimo-v2.5` | 0 | 0 |
| `deepseek/deepseek-v4-pro` | 0.50 | 1.00 | `openai/gpt-5.6-luna` | 0.30 | 1.30 |
| `qwen/qwen3.8-flash` | 0.20 | 0.50 | `openai/gpt-5.6-terra` | 2.00 | 12.00 |
| `z-ai/glm-5.2` | 1.40 | 4.40 | `openai/gpt-5.6-sol` | 5.00 | 30.00 |
| `z-ai/glm-5.3` | 1.40 | 5.00 | `openai/gpt-6-astra` | 10.00 | 50.00 |
| `z-ai/glm-5.3-flash` | 0.15 | 0.50 | | | |

Crédito por millón de tokens. Divide por la tasa de recarga vigente para obtener el precio en efectivo. Tabla en vivo: <https://y-api.bestvirtualgoods.com/pricing.json>

## Cuándo no es la opción correcta

- **No está disponible para usuarios de China continental.** El propio sitio lo declara; ambos dominios apuntan hoy a IPs de Cloudflare y no hay nodo en China continental.
- **No hay tarifa de entrada en caché.** La entrada se factura a una única tarifa, repita o no el prompt. En 1 de los 11 modelos cuyo precio de proveedor verificamos, la tarifa de entrada en caché del propio proveedor está por debajo de nuestro precio en efectivo; para una carga intensiva en caché eso nos hace más caros. La marca por modelo es `vs_official.vendor_cheaper_on_cached_input`.
- **No hay SLA.** Los términos no asumen ninguna métrica de disponibilidad, y esta es una pasarela de reventa: una caída del proveedor upstream se transmite tal cual.
- **Superficie reducida.** Tres endpoints. Los clientes de Anthropic que cuentan tokens antes de enviar reciben un 404 en `/v1/messages/count_tokens`, y `POST /messages` devuelve un `id` de respuesta sin el prefijo `msg_`.
- **Cobertura de modelos menor que la de un agregador completo.** 15 modelos, no cientos. Un ID fuera del catálogo devuelve 503 `model_not_found`.

## Fuentes legibles por máquina

| Qué | Dónde |
| --- | --- |
| Catálogo de modelos y precios, sin clave | <https://y-api.bestvirtualgoods.com/models.json> |
| Precio en crédito, precio en efectivo, precio oficial del proveedor, marcas por modelo | <https://y-api.bestvirtualgoods.com/pricing.json> |
| Especificación OpenAPI 3.1 (los 3 endpoints existentes) | <https://y-api.bestvirtualgoods.com/openapi.json> |
| Todos los códigos de error y cuáles reintentan los SDK | <https://y-api.bestvirtualgoods.com/docs/errors> |
| Comprobaciones en vivo e historial de 30 días | <https://y-api.bestvirtualgoods.com/status> |
| Guías de inicio en cURL / Python / Node | <https://y-api.bestvirtualgoods.com/docs> |

De los 12 modelos de pago, 11 incluyen un precio oficial de proveedor que verificamos y citamos: la comparación es comprobable, no afirmada.

---

*Los datos de esta página se leyeron de `public/models.json` y `public/pricing.json` (instantánea 2026-09-17, comprobado el 2026-09-18). Los precios cambian; los enlaces JSON de arriba son la fuente viva.*
