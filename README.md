# WhatsApp Product Graphic Generator

A WhatsApp bot that turns a voice note and a product photo into a professional marketing graphic and sends it back — automatically, in under 60 seconds, in English, Hindi, or Gujarati.

---

## ⚠️ Personal API Keys — Must Be Replaced Before Production

The following accounts used during development and trial testing are **personal accounts belonging to Mansi Sonani, not Savoir AI-owned accounts**. They must be replaced with company-owned credentials before this system is used for real traders or handed to another developer long-term:

| Service | Account type | Where to create a company one |
|---|---|---|
| **Groq** | Personal API key | https://console.groq.com/keys |
| **Stability AI** | Personal API key | https://platform.stability.ai/account/keys |
| **htmlcsstoimage.com** | Personal account (user ID + API key) | https://htmlcsstoimage.com/ |

No key values are included anywhere in this repo or workflow export — only placeholders (`YOUR_GROQ_API_KEY`, `YOUR_STABILITY_AI_KEY`, `YOUR_HCTI_USER_ID`, `YOUR_HCTI_API_KEY`). Whoever takes this over needs to generate new keys under Savoir AI's own accounts and add them through n8n's Credentials system (Settings → Credentials), not hardcoded into node parameters.

Twilio and Google Sheets access were already set up by Savoir AI and are not personal — no action needed there.

---

## ✨ What Makes This Special

- **Dual-mode rendering with automatic fallback.** Eight rotating HTML/CSS templates (Standard) and Stability AI SDXL-generated backgrounds (Premium) run as parallel branches in the same n8n workflow. A Google Sheets quota counter tracks daily Premium usage; when the limit approaches, the system routes to Standard without any manual intervention. If the Stability AI API fails mid-request, a separate Error Trigger catches the failure and re-routes to Standard. The trader always gets a graphic — never a blank, never an error.

- **Truly multilingual — not translated.** Groq Whisper transcribes voice notes in English, Hindi, and Gujarati natively. Llama 3.3 70B extracts product details and writes a promotional tagline in the same language and script the trader spoke — Hindi in Devanagari, Gujarati in Gujarati, English in Latin. No transliteration, no forced English output.

- **End-to-end automation, zero human in the loop.** A trader sends one WhatsApp message containing a voice note and a product photo. The system transcribes the audio, extracts structured product data, generates or selects a visual layout, injects the data and photo into the template, renders the final composite to a PNG, and delivers it back via WhatsApp. No design tools, no manual editing, no approval step.

- **Visually distinct every time.** Standard Mode randomly selects from eight layout styles on each request, so consecutive graphics from the same trader never look identical. Premium Mode generates a unique AI-produced background for every single product, producing genuinely one-of-a-kind compositions.

- **Built entirely on free tiers.** Groq API (Whisper + Llama), Stability AI free credits, htmlcsstoimage.com free tier, n8n self-hosted, Google Sheets free tier. The only paid component is the Twilio WhatsApp number — a standard business expense that exists regardless of this system.

---

## Architecture Overview

```
Trader sends WhatsApp (voice note + photo)
          │
          ▼
   ┌──────────────────┐
   │   Twilio Webhook   │  Receives incoming WhatsApp message
   └────────┬──────────┘
            │
            ▼
   ┌──────────────────┐
   │  Download Media    │  Fetches voice recording + product photo
   └────────┬──────────┘
            │
            ▼
   ┌──────────────────┐
   │  Groq Whisper      │  Transcribes voice → text
   │  (whisper-large-v3)│
   └────────┬──────────┘
            │
            ▼
   ┌──────────────────┐
   │  Groq Llama 3.3    │  Extracts: product_name, price,
   │  70B Versatile      │  tagline, key_feature, language
   └────────┬──────────┘
            │
            ▼
   ┌──────────────────┐
   │  Welcome Menu       │  Trader message routed: GRAPHIC →
   │  Routing (If1)      │  graphic flow below; VIDEO → forwarded
   └────────┬──────────┘  to video pipeline (see below); anything
            │              else → welcome message
            ▼
   ┌──────────────────┐
   │  Google Sheets     │  Read daily Premium quota count
   │  Quota Check       │
   └──┬─────────────┬──┘
      │             │
  count < 23    count ≥ 23
      │             │
      ▼             ▼
  ┌────────┐   ┌────────────┐
  │Premium │   │  Standard   │
  │        │   │            │
  │Stability│  │  Random     │
  │AI SDXL │   │  template   │
  │→ bg    │   │  selection  │
  │→ compos│   │  (1 of 8)  │
  └───┬────┘   └─────┬──────┘
      │              │
      └──────┬───────┘
             │
             ▼
   ┌──────────────────┐
   │  htmlcsstoimage    │  Renders HTML + CSS → PNG
   └────────┬──────────┘
            │
            ▼
   ┌──────────────────┐
   │  Google Sheets     │  Logs: mode used, product name,
   │  Append Row        │  timestamp, status
   └────────┬──────────┘
            │
            ▼
   ┌──────────────────┐
   │  Twilio Send       │  Sends PNG graphic back to trader
   │  WhatsApp Message  │  via WhatsApp
   └──────────────────┘
```

---

## Prerequisites

| Service | What you need |
|---|---|
| Twilio | WhatsApp Sandbox or approved sender number |
| Groq | API key (`YOUR_GROQ_API_KEY`) — replace personal key, see warning above |
| Stability AI | API key (`YOUR_STABILITY_AI_KEY`) — Premium Mode only, replace personal key, see warning above |
| htmlcsstoimage.com | Account with user ID + API key (`YOUR_HCTI_USER_ID`, `YOUR_HCTI_API_KEY`) — replace personal account, see warning above |
| n8n | Self-hosted instance or n8n Cloud |
| Google Cloud | Google Sheets API enabled, OAuth2 service account credentials |

> ⚠️ **All API keys and credentials must be added through n8n's built-in Credentials system** (Settings → Credentials → New). Never paste keys directly into HTTP Request node headers — they will be exposed in workflow exports.

---

## Setup Instructions

1. **Clone this repository** to your n8n server or local machine.

2. **Import the workflow.** Open n8n → menu → Import → select `workflow/graphic-generator.json`. All nodes, connections, and expressions will load automatically.

3. **Configure credentials in n8n** (Settings → Credentials → Add Credential). Do NOT hardcode keys in node parameters — use n8n's credential store for all of the following:
   - **Header Auth** (for Groq): `Authorization` header with value `Bearer YOUR_GROQ_API_KEY`.
   - **HTTP Basic Auth** (for Twilio): Account SID as username, Auth Token as password.
   - **HTTP Basic Auth** (for htmlcsstoimage.com): User ID as username, API key as password.
   - **Google Sheets OAuth2**: Connect using your Google Cloud service account with Sheets API enabled.
   - **Header Auth** (for Stability AI): `Authorization` header with value `Bearer YOUR_STABILITY_AI_KEY`.

4. **Upload font files** to a `fonts/` directory on the n8n server. The templates use Space Grotesk (weights 500, 600, 700) and Inter (weights 300, 400, 500, 600). These are referenced via `@font-face` in each template. Obtain from Google Fonts and convert to TTF format.

5. **Copy template files** from `templates/` to the path on your n8n server that the workflow's template-selection node reads from.

6. **Create a Google Sheet** with the following tabs:
   - **Sessions** — stores registered trader phone numbers and session data.
   - **Logs-Graphic** — logs every graphic generation (timestamp, phone number, product name, mode used, template number, status).
   - **Quota Tracker** (or whatever your quota tab is named) — cell A2 holds today's date, cell B2 holds the daily Premium generation count. The workflow auto-resets B2 to 0 at midnight.
   - **Mode Log** — records which mode fired on each request and why (Premium, Quota Fallback, Standard).

7. **Update the workflow's Google Sheet references** to point to your new sheet. In the workflow JSON, search for `YOUR_GOOGLE_SHEET_ID` and replace it with your actual Google Sheet ID.

8. **Configure the Twilio webhook.** In the Twilio Console, set your WhatsApp number's webhook URL to point to your n8n webhook endpoint (the Webhook node's production URL).

9. **Update the Twilio sender and recipient numbers.** In the workflow JSON, replace `YOUR_TWILIO_SENDER_NUMBER` with your Twilio WhatsApp number and `YOUR_TRADER_PHONE_NUMBER` with the destination number (or switch it to a dynamic expression if you want it to reply to whoever sent the message).

10. **Test end to end.** Send a WhatsApp message with a voice note describing a product (including the price) and a product photo. Verify the graphic is returned within ~60 seconds for Standard Mode or ~90 seconds for Premium Mode.

---

## Trader Onboarding

Traders interact with the system by sending a WhatsApp message that contains both a voice note and a product photo. The voice note should describe the product and mention the price — for example: *"Yeh hai industrial ball valve 2 inch size ka, price hai 1250 rupaye"*.

The system handles everything from there: transcription, data extraction, template selection (or AI background generation), compositing, rendering, and delivery. The trader receives a finished 1080×1920 marketing graphic back in WhatsApp, ready to forward or post as a Status.

No manual registration is required. The first time a trader sends a photo and voice note, the system logs their phone number in the Sessions sheet automatically. See `docs/trader-onboarding-message.md` for the onboarding message template.

---

## Standard vs Premium Mode

| | Standard Mode | Premium Mode |
|---|---|---|
| **Background** | Pre-built HTML/CSS template (8 styles) | AI-generated via Stability AI SDXL |
| **Visual variety** | Random template selection per request | Unique background per product |
| **Availability** | Always available, no quota | ~23 generations/day on Stability free tier |
| **Latency** | ~15–40 seconds | ~45–90 seconds (includes SDXL inference) |
| **Cost per graphic** | Zero | Zero (free tier credits) |
| **Fallback** | N/A — this IS the fallback | Auto-routes to Standard when quota exceeded or API fails |

Both modes use the same voice transcription (Groq Whisper), the same data extraction (Groq Llama 3.3), the same product photo, and the same rendering pipeline (htmlcsstoimage.com → PNG → Twilio). The only difference is the visual background layer.

---

## Loom Walkthrough

[Watch the full walkthrough →](https://www.loom.com/share/0578ccfd621341009be5c5e9ea21af6d)

---

## Welcome Menu & Connecting to Helga's Video Pipeline

The workflow now includes a welcome-menu router: when a trader messages the bot, their message text is checked and routed three ways:

- **"GRAPHIC"** → routes into the graphic-generation flow described above.
- **"VIDEO"** → looks up the trader's session data in the Sessions sheet, then forwards a request to the video pipeline's webhook (built by Helga, a colleague on a separate video-generation pipeline) with the payload structure below.
- **Anything else** (first-time "hi", unrecognized text) → sends a welcome message asking the trader to reply GRAPHIC or VIDEO.

Video request payload sent to Helga's pipeline:

```json
{
  "event": "video_request",
  "session_key": "whatsapp:+91XXXXXXXXXX",
  "mode": "4",
  "media_urls": ["https://..."],
  "voice_note_url": "https://...",
  "transcript": "...",
  "product_name": "...",
  "price": "...",
  "key_feature": "...",
  "tagline": "...",
  "language": "English",
  "music_mood": "clean"
}
```

Mode mapping:

| Mode | Type |
|---|---|
| `4` | Single photo reel |
| `5` | Slideshow |
| `6` | Video edit |

The video pipeline handles generation and delivers the final video back to the trader via WhatsApp through its own separate webhook/Twilio integration (already tested and confirmed working with a real video).

**Current status of this integration:** the routing, session lookup, and outbound request are built and tested on this side. The final live handoff to the video pipeline is currently blocked by a 404 from the video pipeline's hosting (Railway) — confirmed by Helga to be a hosting-credit issue on her end, not a bug in this workflow. Once her hosting is restored, this integration should work without further changes here.

---

## Project Status

| Feature | Status |
|---|---|
| Standard Mode (8 templates) | ✅ Fully working, tested in EN/HI/GU |
| Premium Mode (AI backgrounds) | ✅ Fully working, quota logic + fallback tested |
| Google Sheets logging | ✅ Sessions, Logs-Graphic, Mode Log all populating |
| Daily quota reset | ✅ Auto-resets at midnight |
| Multilingual support | ✅ English, Hindi, Gujarati confirmed |
| Welcome menu routing (GRAPHIC/VIDEO) | ✅ Built and tested end-to-end on WhatsApp |
| Video pipeline handoff | ⏳ Built and tested on this side; blocked on video pipeline's hosting being restored (external, not a code issue) |
