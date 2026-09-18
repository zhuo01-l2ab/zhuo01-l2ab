# Y-API — 15 Modelle, ein Schlüssel, Abrechnung pro Token

[English](README.md) · [简体中文](README.zh.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Español](README.es.md) · **Deutsch** · [Português](README.pt.md)

> **Offenlegung:** Ich arbeite an Y-API, alles unten ist also eine Aussage der Anbieterseite — prüfe sie gegen die maschinenlesbaren Quellen am Ende. Alle Links sind reine Links: kein Empfehlungscode, kein Tracking-Parameter.

**Y-API** ist ein OpenAI-kompatibles LLM-Gateway. 15 Modelle von 7 Anbietern (DeepSeek, Qwen, Z.ai, Moonshot AI, Tencent, Xiaomi, OpenAI) liegen hinter einer einzigen `base_url`, einem Schlüssel und einem Guthaben. Keine Monatsgebühr, kein Mindestumsatz, keine Plantarife — das Guthaben wird pro Token abgezogen, sonst nichts.

## In 30 Sekunden ausprobieren

```bash
curl https://api.y-api.bestvirtualgoods.com/v1/chat/completions \
  -H "Authorization: Bearer $Y_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"deepseek/deepseek-v4-flash-0731","messages":[{"role":"user","content":"Begrüße mich in einem Satz."}]}'
```

Bestehender OpenAI-Code braucht nur eine geänderte Basis-URL:

```python
from openai import OpenAI
client = OpenAI(base_url="https://api.y-api.bestvirtualgoods.com/v1", api_key=Y_API_KEY)
```

```js
import OpenAI from "openai";
const client = new OpenAI({ baseURL: "https://api.y-api.bestvirtualgoods.com/v1", apiKey: Y_API_KEY });
```

Der Zusatz `/v1` steckt bereits in der Basis-URL; ihn doppelt zu setzen ergibt einen 400. Auch Anthropic-Clients funktionieren: `POST /v1/messages` bedient denselben Katalog mit demselben Schlüssel und akzeptiert ihn zusätzlich in `x-api-key`.

## Was es kostet

- Preise sind in **Kontoguthaben** angegeben. Aktuell gilt: **$1 eingezahlt = $20 Guthaben** — ein zeitlich begrenzter Kurs, der danach auf **1:10** zurückfällt; ein Enddatum ist nicht angekündigt. **Barpreis = Guthaben ÷ 20.**
- `$1` Startguthaben bei der Anmeldung, keine Karte nötig. Ist das Guthaben aufgebraucht, antwortet die API mit 429 und funktioniert sofort nach einer Aufladung weiter; Schlüssel bleiben gültig.
- Günstigstes Bezahlmodell: `deepseek/deepseek-v4-flash-0731` für $0.15 Eingabe / $0.30 Ausgabe pro 1M Tokens in Guthaben, also **$0.0075 pro 1M Eingabe-Tokens in bar**.
- Drei Modelle kosten null: `deepseek/deepseek-v4-flash`, `tencent/hy3`, `xiaomi/mimo-v2.5`.

| Modell | In | Out | Modell | In | Out |
| --- | --- | --- | --- | --- | --- |
| `deepseek/deepseek-v4-flash` | 0 | 0 | `moonshotai/kimi-k3` | 3.00 | 15.00 |
| `deepseek/deepseek-v4-flash-0731` | 0.15 | 0.30 | `tencent/hy3` | 0 | 0 |
| `deepseek/deepseek-v4.1-flash` | 0.20 | 1.00 | `xiaomi/mimo-v2.5` | 0 | 0 |
| `deepseek/deepseek-v4-pro` | 0.50 | 1.00 | `openai/gpt-5.6-luna` | 0.30 | 1.30 |
| `qwen/qwen3.8-flash` | 0.20 | 0.50 | `openai/gpt-5.6-terra` | 2.00 | 12.00 |
| `z-ai/glm-5.2` | 1.40 | 4.40 | `openai/gpt-5.6-sol` | 5.00 | 30.00 |
| `z-ai/glm-5.3` | 1.40 | 5.00 | `openai/gpt-6-astra` | 10.00 | 50.00 |
| `z-ai/glm-5.3-flash` | 0.15 | 0.50 | | | |

Guthaben pro 1M Tokens. Für den Barpreis durch den aktuellen Aufladekurs teilen. Live-Tabelle: <https://y-api.bestvirtualgoods.com/pricing.json>

## Wann es die falsche Wahl ist

- **Für Nutzer in Festlandchina nicht verfügbar.** Das steht so auf der Seite selbst; beide Domains zeigen derzeit auf Cloudflare-IPs, ein Node auf dem Festland existiert nicht.
- **Kein Tarif für Cache-Treffer.** Eingaben werden unabhängig von Wiederholungen zum gleichen Satz berechnet. Bei 1 der 11 Modelle mit verifiziertem Anbieterpreis liegt der Cache-Eingabepreis des Anbieters unter unserem Barpreis — für cache-lastige Workloads sind wir dann teurer. Das Feld pro Modell heißt `vs_official.vendor_cheaper_on_cached_input`.
- **Kein SLA.** Die Bedingungen sagen keine Verfügbarkeitskennzahl zu, und dies ist ein Weiterverkaufs-Gateway: ein Ausfall beim Upstream schlägt direkt durch.
- **Kleine Angriffsfläche.** Drei Endpunkte. Anthropic-Clients, die vor dem Senden Token zählen, erhalten auf `/v1/messages/count_tokens` einen 404, und `POST /messages` liefert eine Antwort-`id` ohne das Präfix `msg_`.
- **Geringere Modellabdeckung als ein vollständiger Aggregator.** 15 Modelle, nicht Hunderte. Eine Modell-ID außerhalb des Katalogs ergibt 503 `model_not_found`.

## Maschinenlesbare Quellen

| Was | Wo |
| --- | --- |
| Modellkatalog und Preise, ohne Schlüssel | <https://y-api.bestvirtualgoods.com/models.json> |
| Guthabenpreis, Barpreis, Anbieterlistenpreis, Flags pro Modell | <https://y-api.bestvirtualgoods.com/pricing.json> |
| OpenAPI-3.1-Spezifikation (die 3 existierenden Endpunkte) | <https://y-api.bestvirtualgoods.com/openapi.json> |
| Alle Fehlercodes und welche die SDKs selbst wiederholen | <https://y-api.bestvirtualgoods.com/docs/errors> |
| Live-Prüfungen und 30-Tage-Verlauf | <https://y-api.bestvirtualgoods.com/status> |
| Schnellstarts für cURL / Python / Node | <https://y-api.bestvirtualgoods.com/docs> |

Von den 12 Bezahlmodellen tragen 11 einen von uns verifizierten und zitierten Anbieterpreis — der Vergleich ist nachprüfbar, nicht behauptet.

## Ebenfalls in diesem Account

**[y-api-price-tracker](https://github.com/zhuo01-l2ab/y-api-price-tracker)** — ein Skript ohne Abhängigkeiten, das die veröffentlichte Preisdatei als Barpreis-Tabelle rendert, wöchentlich von CI aktualisiert. Es rechnet `credit_price / top_up.quota_rate` nach, statt `cash_price` zu kopieren, und scheitert laut, wenn beides abweicht. Wer Preisänderungen lieber als maschinenlesbares Diff will statt als gerenderte Seite, fängt dort an.

---

*Die Angaben auf dieser Seite stammen aus `public/models.json` und `public/pricing.json` (Stand 2026-09-17, geprüft am 2026-09-18). Preise ändern sich; die JSON-Links oben sind die live-Quelle.*
