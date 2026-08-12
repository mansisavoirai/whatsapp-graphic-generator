# Groq Whisper Transcription Setup

## Configuration

| Parameter | Value |
|-----------|-------|
| **Endpoint** | `https://api.groq.com/openai/v1/audio/transcriptions` |
| **Model** | `whisper-large-v3` |
| **Auth** | Header Auth — `Authorization: Bearer gsk_...` |
| **Method** | `POST` |

## Request

- **Content-Type:** `multipart/form-data`
- **Fields:**
  - `file` — audio file (binary)
  - `model` — `whisper-large-v3`
  - `response_format` — `json`

### Why Save to Disk First

Twilio media URLs (`MediaUrl` from incoming WhatsApp messages) are **temporary** and expire quickly. The workflow uses n8n's **Read/Write Files** node to:

1. Download the audio from the Twilio temporary URL
2. Save it to disk as `.ogg`
3. Read it back as a binary buffer
4. Send it as `multipart/form-data` to the Groq endpoint

Skipping this step causes failures — expired URLs return 404s or the binary isn't correctly formatted for multipart upload.

## Response

```json
{
  "text": "ye hai 8mm ka ply board price 45 rupaye per square foot abhi stock mein hai""
}
```

Single field — `text` — containing the full transcript in whatever language was spoken.

## Connection Header

The request includes a `Connection: close` header to prevent the HTTP connection from hanging after response. Groq's server keeps connections alive by default; n8n's HTTP Request node can stall waiting for a close if this isn't set.

## n8n Node Setup Summary

1. **Twilio** trigger receives WhatsApp message with `MediaUrl0`
2. **HTTP Request** — download audio from `MediaUrl0`, output as binary
3. **Read/Write Files** — write binary to `/tmp/{{ $json.MessageSid }}.ogg`
4. **Read/Write Files** — read file back as binary
5. **HTTP Request** — POST to Groq Whisper with file in multipart body, `Connection: close` header
6. Output: `{{ $json.text }}` forwarded to Llama extraction node
