# Bumi Trasa SOP Assistant

Internal SOP assistant for a civil engineering contractor, built with **n8n**, **Google Gemini 2.5 Flash**, and the **Telegram Bot API**.

> **Portfolio Project** — The same workflow as a retail customer service bot, retargeted to a different industry by changing one system prompt.

---

## Overview

PT Bumi Trasa Infrastruktur is a fictional contractor specialising in roads, bridges, and drainage. Its field teams — site supervisors, engineers, project admins — need fast answers about safety procedures, material specifications, and permit workflows.

This bot answers those questions on Telegram, where field staff already are.

**What makes it interesting is what it started as.** This workflow is a duplicate of [kahfka-ai-chatbot](https://github.com/genta-septiawan/kahfka-ai-chatbot), a customer service bot for an online fashion store. Same nodes, same wiring, same model. Only the system prompt changed.

That is the point: the architecture is not tied to an industry. Adapting it to yours means rewriting a prompt, not rebuilding a system.

---

## Scope

**Answers questions about**

- Work procedures — site preparation, concrete pouring, soil compaction
- Safety (K3) — required PPE, working at height, heavy equipment protocols
- Material specifications — concrete grades, aggregate types, dimensional tolerances
- Project administration — permit submissions, daily reports

**Refuses, by design**

- Client pricing and contract negotiation
- Anything outside construction
- Any specification or procedure it does not actually hold

That last refusal matters more here than in a retail bot. A wrong answer about a return policy costs a customer some annoyance. A wrong answer about fall-protection equipment costs something else. The system prompt states this explicitly and instructs the model to say it has no data rather than produce a plausible number.

---

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Automation platform | n8n (self-hosted) |
| LLM | Google Gemini 2.5 Flash |
| Messaging interface | Telegram Bot API |
| Memory store | n8n Simple Memory (session-based) |
| Tunnelling (dev) | ngrok |

Running cost: **$0** on free tiers.

---

## Workflow Architecture

```
Telegram Trigger → AI Agent → Send a text message
                      │
                      ├─ Google Gemini Chat Model
                      └─ Simple Memory (keyed on chat ID)
```

Identical to the retail version. The entire domain shift lives inside the AI Agent's system message.

---

## Demo

![Workflow](screenshots/bumi-trasa-canvas.png)

**Live conversation — a clean refusal, and an answer that shows the ceiling of this approach**
![Telegram demo](screenshots/bumi-trasa-demo.png)

Two things are visible in that exchange, and the second one matters more.

**The guardrail works.** Asked for the contract value of a specific toll road project, the bot declines and routes the question to the commercial department. It does not produce a number.

**The knowledge ceiling is also visible.** Asked for the concrete grade used in bridge structures, it answers with general industry practice — *"K-350 to K-500 is commonly applied"* — and then tells the user to check the project's actual technical specification. That hedge is correct behaviour for a bot that has never read the document. It is also the exact limitation that makes this approach unsuitable for real safety documentation.

The RAG version answers the same question with *"K-350, f'c 29.0 MPa, slump 12 ± 2 cm, per SOP-BTI-MTL-001"* — a specific figure with a document citation, because it retrieved the actual table.

Other tested scenarios:

- **In-scope question** — PPE requirements and work procedures, answered within its defined domain
- **Multi-turn memory** — a follow-up resolves against the previous message without restating the subject

---

## Setup

**Prerequisites**

- Node.js 18+
- n8n (`npm install -g n8n`)
- A Telegram bot token from [@BotFather](https://t.me/BotFather)
- A Google Gemini API key from [Google AI Studio](https://aistudio.google.com/apikey)

**Steps**

1. Start a tunnel: `ngrok http 5678`
2. Start n8n with the tunnel URL — Telegram rejects non-HTTPS webhooks:

   ```powershell
   $env:WEBHOOK_URL = "https://your-subdomain.ngrok-free.dev/"
   n8n start
   ```

3. Import `workflow/bumi-trasa-sop-assistant.json` (n8n → Import from File)
4. Set credentials on the Telegram and Gemini nodes
5. Click **Publish** — not *Execute workflow*, which only registers a temporary test webhook
6. Message the bot on Telegram

The workflow export contains credential *references* only — no keys or tokens. Select your own after importing.

**Note:** this bot needs its own Telegram token, separate from the retail version. Two workflows cannot share one — the webhook registration collides.

---

## Limitations

The bot's knowledge is held entirely in its system prompt — roughly a page of text covering procedures in general terms.

It has not read the company's actual SOP manual. Ask it for the contents of a specific document number and it has nothing to retrieve, because there is nothing to retrieve from. Instructing a model not to invent procedures reduces hallucination; it does not eliminate it.

For safety documentation, "reduces" is not good enough.

**That limitation is the reason the next project exists:** [bumi-trasa-rag-assistant](https://github.com/genta-septiawan/bumi-trasa-rag-assistant) grounds this same bot in a real 10-page SOP manual using a Supabase vector database. Every answer is retrieved from the document, cites its SOP number, and the bot refuses when the document genuinely does not cover the question — verified against a document number that does not exist.

Read this repository as the baseline. Read that one as the fix.

---

## Project Structure

```
bumi-trasa-sop-assistant/
├── README.md
├── LICENSE
├── workflow/
│   └── bumi-trasa-sop-assistant.json
└── screenshots/
    ├── bumi-trasa-canvas.png
    └── bumi-trasa-demo.png
```

---

## Related Projects

- **[kahfka-ai-chatbot](https://github.com/genta-septiawan/kahfka-ai-chatbot)** — the retail customer service bot this was duplicated from
- **[bumi-trasa-rag-assistant](https://github.com/genta-septiawan/bumi-trasa-rag-assistant)** — the same bot, grounded in a real document via RAG

---

## Author

**Genta Septiawan** — Civil Engineer moving into AI automation

I spent years on construction projects before building this. That background is why this variant exists: I know how site documentation, permit chains, and safety procedures actually behave, not just how to wire an API.

- [LinkedIn](https://www.linkedin.com/in/gentaseptiawan/)
- Available for freelance AI automation work

---

## License

MIT — use it as a reference for your own projects.

---

*Built as part of an AI Automation Engineer portfolio, 2026*
