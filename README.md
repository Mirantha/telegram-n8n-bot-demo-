# 🤖 Telegram Bot Automation with n8n

A no-code/low-code Telegram bot built using **n8n**, featuring command-based responses, inline keyboard buttons, and callback query handling. This project was built as a hands-on learning exercise in workflow automation, webhook integration, and conditional logic design.

![n8n](https://img.shields.io/badge/n8n-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)
![Telegram](https://img.shields.io/badge/Telegram_Bot_API-2CA5E0?style=for-the-badge&logo=telegram&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=node.js&logoColor=white)

---

## 📋 Overview

This bot demonstrates a complete automation workflow that:
- Responds to Telegram commands (`/start`, `/help`, `/menu`)
- Displays an interactive inline keyboard menu
- Handles button click events (callback queries) with dedicated responses
- Routes different message types through conditional logic

Built entirely in **n8n**, running locally and exposed to the internet via **ngrok** for webhook testing.

---

## ✨ Features

| Feature | Description |
|---|---|
| 🔤 Command Handling | Recognizes `/start`, `/help`, and `/menu` text commands |
| 🔘 Inline Keyboard | Interactive buttons (Weather, Articles, Contact) attached to the menu message |
| 🔄 Callback Query Routing | Separate logic path for button clicks vs. text messages |
| 🧠 Conditional Logic | Uses n8n's Switch node with 6 routing rules to direct traffic |
| ✅ Callback Acknowledgment | Clears Telegram's button loading state after each click |

---

## 🏗️ Architecture

```
Telegram Trigger (Message + Callback Query)
        │
        ▼
   Switch Node (Rules)
   ├── /start   → Welcome message
   ├── /help    → Help message
   ├── /menu    → Menu message with inline buttons
   ├── weather  → Weather reply + Answer Callback Query
   ├── articles → Articles reply + Answer Callback Query
   └── contact  → Contact reply + Answer Callback Query
```

The workflow uses a **single Telegram Trigger** node configured to listen for both `message` and `callback_query` update types (Telegram restricts each bot to one trigger at a time), then routes all events through one Switch node.

---

## 🛠️ Tech Stack

- **n8n** (self-hosted via `npx n8n`) — workflow automation engine
- **Telegram Bot API** — messaging platform
- **ngrok** — secure tunnel for exposing local webhooks during development
- **Node.js** — runtime for n8n

---

## 🚀 Setup Instructions

### Prerequisites
- Node.js installed
- A Telegram account
- [ngrok](https://ngrok.com/) account (free tier works)

### 1. Create a Telegram Bot
1. Open Telegram and search for **@BotFather**
2. Send `/newbot` and follow the prompts
3. Copy the **API token** you receive

### 2. Run n8n Locally
```bash
npx n8n
```
Open `http://localhost:5678` and complete the owner account setup.

### 3. Expose n8n with ngrok
```bash
ngrok http 5678
```
Copy the generated HTTPS forwarding URL.

### 4. Set the Webhook URL and Restart n8n
```bash
set WEBHOOK_URL=https://your-ngrok-url.ngrok-free.dev/
npx n8n
```

### 5. Import the Workflow
1. In n8n, create a new workflow
2. Import `workflow.json` (included in this repo)
3. Add your Telegram credential using the BotFather token
4. Click **Publish**

### 6. Test It
Message your bot on Telegram with `/start`, `/help`, or `/menu`.

---

## 🐛 Debugging Notes (Lessons Learned)

A key issue encountered during development: the Switch node routing rules for button clicks were checking for `Weather`, `Articles`, and `Contact` (capitalized), while Telegram's actual `callback_data` payload returned lowercase values (`weather`, `articles`, `contact`). Since n8n's equality check is case-sensitive, no rule matched and the workflow silently produced no output.

**Fix:** Inspected the raw JSON in the node's execution data to find the exact `callback_data` values, then corrected the routing rules to match exactly.

This highlights a useful debugging pattern for n8n: when a Switch/IF node produces no output, check the **execution's JSON input data** for the exact field values before assuming a logic error elsewhere.

---

## 📸 Demo

*(Add a screenshot or screen recording of the bot in action here)*

---

## 🔮 Future Improvements

- [ ] Integrate a live weather API via HTTP Request node
- [ ] Pull latest articles dynamically from [wikimess.com](https://wikimess.com)
- [ ] Log all user interactions to Google Sheets
- [ ] Add error-handling workflow with admin alerts
- [ ] Deploy on a persistent host (VPS) instead of ngrok for production use

---

## 👤 Author

**Mira** — Full-stack developer (PHP/WordPress) & freelancer
- Portfolio: [creativeidea.lk](https://creativeidea.lk)
- Project: [wikimess.com](https://wikimess.com)

---

## 📄 License

This project is open for learning purposes. Feel free to fork and adapt.
