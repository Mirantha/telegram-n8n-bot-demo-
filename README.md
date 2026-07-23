# Telegram Learning Bot (n8n + Telegram Bot API)

A Telegram bot built entirely with [n8n](https://n8n.io) workflow automation, featuring command handling, inline keyboard buttons, and live weather data from an external API. Built as a hands-on learning project to explore n8n's Telegram integration, webhook handling, and HTTP Request nodes.

## 🚀 Features

- **Command handling**: `/start`, `/help`, `/menu`
- **Inline keyboard buttons**: Weather, Articles, Contact
- **Live weather data**: Pulled from the [Open-Meteo API](https://open-meteo.com/) via an HTTP Request node
- **Duplicate callback protection**: Custom dedupe logic to handle Telegram's callback query retries
- **Local development**: Runs on `npx n8n` with ngrok for public webhook exposure

## 🏗️ Architecture

```
Telegram Trigger → Code (Dedupe Filter) → Switch (Rules)
                                              ├─ start   → Send Message
                                              ├─ help    → Send Message
                                              ├─ menu    → Send Message
                                              ├─ weather → Answer Callback → HTTP Request → Send Message
                                              ├─ articles→ Answer Callback → Send Message
                                              └─ contact → Answer Callback → Send Message
```

**Key design decision**: Callback queries are answered (acknowledged) *before* the reply message is sent, to avoid Telegram's "expired callback" errors during slower response times.

## 🛠️ Setup

### Prerequisites
- Node.js installed
- A Telegram account
- [ngrok](https://ngrok.com/) (free tier works)

### Steps

1. **Install and run n8n locally**
   ```bash
   npx n8n
   ```
   n8n will be available at `http://localhost:5678`

2. **Create a Telegram bot**
   - Message [@BotFather](https://t.me/BotFather) on Telegram
   - Run `/newbot` and follow the prompts
   - Save the API token you receive

3. **Expose n8n publicly with ngrok**
   ```bash
   ngrok http 5678
   ```
   Copy the generated `https://xxxx.ngrok-free.app` forwarding URL.

4. **Set the webhook URL and restart n8n**
   ```bash
   # PowerShell
   $env:WEBHOOK_URL="https://xxxx.ngrok-free.app/"
   npx n8n
   ```

5. **Add Telegram credentials in n8n**
   - Go to Credentials → New → Telegram API
   - Paste your bot token from BotFather

6. **Import/build the workflow** and set the Telegram Trigger to listen for `message` and `callback_query` updates.

7. **Publish the workflow** in n8n so the webhook goes live.

## 🐛 Debugging Notes (Real Issues Faced & Fixed)

This project involved several real-world debugging challenges — documented here for future reference:

### 1. ngrok + localhost limitation
Telegram requires a public HTTPS URL for webhooks; `localhost` doesn't work. Solved by tunneling with ngrok and setting the `WEBHOOK_URL` environment variable before starting n8n.

### 2. Case-sensitive callback_data
Switch node rules checked `Weather`/`Contact` (capitalized) but the actual `callback_data` sent by the inline keyboard was lowercase (`weather`/`contact`). Found by inspecting raw JSON in the execution data.

### 3. "Bad request — please check your parameters" on Answer Callback Query
Caused by Telegram re-sending the same `callback_query` update when the webhook response was slow (due to ngrok latency), resulting in the same `callback_query.id` being answered twice — which Telegram rejects on the second attempt.

**Fix**: Added a Code node between the Telegram Trigger and the Switch node that tracks processed `callback_query.id` values in workflow static data and silently drops duplicates within a 5-minute window.

### 4. n8n test-listener vs. production webhook conflict
n8n cannot run a Telegram Trigger in both "test" (manual execute) and "production" (published) listening modes simultaneously. Testing nodes manually while the workflow was published caused webhook requests to be silently dropped. **Fix**: avoid manually executing nodes while the bot is live; unpublish/republish after any node testing.

### 5. Expression scoping after reordering nodes
After moving "Answer Callback Query" before "Send Message" in the weather branch, `{{ $json.current_weather.temperature }}` returned empty because `$json` refers to the *immediately preceding* node's output, and node order had shifted. **Fix**: used explicit node references instead of relying on `$json`:
```
{{ $('HTTP Request').item.json.current_weather.temperature }}
```

### 6. Expression mode not enabled on text fields
Typing `{{ }}` into a field without toggling it to "Expression" mode causes n8n to treat it as literal text. Always confirm the field is switched from **Fixed** to **Expression** before entering dynamic values.

## 📌 Known Limitations

- Runs on the free ngrok tier, which has session limits and occasional disconnects — not suitable for production use as-is.
- No persistent database; all state (e.g., dedupe tracking) lives in workflow static data and resets if n8n restarts.

## 🔮 Future Improvements

- [ ] Replace ngrok with a stable tunnel (Cloudflare Tunnel) or deploy n8n on a VPS/cPanel for a permanent public URL
- [ ] Move to n8n Cloud or self-hosted production deployment
- [ ] Add more commands and richer inline keyboard menus
- [ ] Store user interactions in a database for analytics

## 🧰 Tech Stack

- [n8n](https://n8n.io) — workflow automation
- [Telegram Bot API](https://core.telegram.org/bots/api)
- [Open-Meteo API](https://open-meteo.com/) — weather data
- [ngrok](https://ngrok.com/) — local tunnel for webhook testing

---

Built as a learning project by Mira ([Creative Idea](https://creativeidea.lk)).
