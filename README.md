# Allen-Community — WhatsApp AI Chat Trigger

An [n8n](https://n8n.io) automation workflow that turns a WhatsApp Business number into a multimodal AI assistant. It listens for incoming WhatsApp messages — text, voice notes, or images — and replies intelligently using GPT-4.1, with live web search and short-term conversation memory.

## What it does

1. **WhatsApp Trigger** — listens for new messages on a connected WhatsApp Business account.
2. **Switch** — routes the incoming message based on its type: audio, text, or image.
3. **Audio path** — downloads the voice note via the WhatsApp Media API, then transcribes it to text using OpenAI Whisper (`Transcribe a recording`).
4. **Image path** — downloads the image and analyzes its contents with GPT-4.1 Vision (`Analyze image`), converting it into a text description.
5. **Text path** — the message text is used as-is.
6. **AI Agent** — all three paths converge into a single LangChain AI Agent (GPT-4.1) that:
   - Keeps a rolling conversation memory (last 10 messages) via `Simple Memory`.
   - Can call **Google Search (SerpApi)** as a tool to answer questions about real-time information.
7. **Send message** — the agent's reply is sent back to the user on WhatsApp.

## Architecture

```
WhatsApp Trigger → Switch ─┬─ audio → Download → Transcribe ─┐
                           ├─ text  → Edit Fields ────────────┼─→ AI Agent (GPT-4.1 + SerpAPI + Memory) → Send message (WhatsApp)
                           └─ image → Download → Analyze ─────┘
```

## Tech / Services used

| Component | Purpose |
|---|---|
| n8n | Workflow orchestration |
| WhatsApp Cloud API | Message trigger + sending replies + media download |
| OpenAI GPT-4.1 | Chat agent reasoning + image analysis |
| OpenAI Whisper | Voice note transcription |
| SerpApi | Real-time Google search tool for the agent |

## Setup

1. Import `allen-community-whatsapp-chat-trigger.json` into your n8n instance (**Workflows → Import from File**).
2. Configure credentials for each node:
   - **WhatsApp OAuth account** (trigger) and **WhatsApp account** (send/download) — from your Meta/WhatsApp Business API app.
   - **OpenAI** — API key or n8n-managed AI Gateway.
   - **SerpApi account** — API key from [serpapi.com](https://serpapi.com).
3. Replace the placeholder `YOUR_WHATSAPP_ACCESS_TOKEN_HERE` in the two `HTTP Request` nodes (used to download media) with your own WhatsApp Cloud API access token.
4. Update `phoneNumberId` and `recipientPhoneNumber` in the **Send message** node to match your setup.
5. Activate the workflow.

> **Security note:** The original export contained a live WhatsApp access token hardcoded in two HTTP Request nodes. It has been redacted to a placeholder in this published copy — set your own token via credentials/environment before running.
