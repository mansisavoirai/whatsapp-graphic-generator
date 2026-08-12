# Groq Llama Product Extraction

## Configuration

| Parameter | Value |
|-----------|-------|
| **Endpoint** | `https://api.groq.com/openai/v1/chat/completions` |
| **Model** | `llama-3.3-70b-versatile` |
| **Auth** | Header Auth — `Authorization: Bearer gsk_...` |
| **Method** | `POST` |
| **Content-Type** | `application/json` |
| **Temperature** | 0.1 |

---

## System Prompt

```
You are a product detail extractor and copywriter for a hardware trading business in India. Extract product information from the voice note transcript and write a short promotional tagline. Always respond with valid JSON only — no extra text before or after.
```

---

## User Prompt Template

```handlebars
Extract product details from this voice transcript and return valid JSON with these exact fields:

- "product_name": The product name as spoken (keep original spelling/words)
- "price": The price mentioned, as a string including currency symbol if mentioned (e.g. "₹45/sq ft", "₹1200/piece", null if not mentioned)
- "key_feature": One main feature or selling point mentioned, or null
- "tagline": A short promotional tagline for this product — emphasise quality, durability, competitive pricing, or availability. Keep it punchy (max 8 words).
- "language": The language the transcript is in — "hindi", "english", "gujarati", "marathi", etc.

Transcript:
{{ $json.text }}

Respond with JSON only. No markdown fences, no explanation.
```

---

## Expected Output Format

```json
{
  "product_name": "8mm Ply Board",
  "price": "₹45/sq ft",
  "key_feature": "moisture resistant",
  "tagline": "Built to last. Priced to move.",
  "language": "hindi"
}
```

---

## Rules

1. **Never translate** — `product_name`, `tagline`, and `key_feature` must stay in the language spoken. Hindi stays in Devanagari script, Gujarati in Gujarati script, etc.
2. **Language detection** — the `language` field should be a lowercase language code matching what was spoken.
3. **Tagline style** — hardware traders respond to practical claims. Emphasise:
   - Quality / durability
   - Competitive price
   - Stock availability
   - Not flowery — direct and commercial
4. **JSON-only response** — if the model wraps in markdown code fences, the downstream node must strip them. Temperature at 0.1 minimises this.
5. **Graceful nulls** — if price or key_feature isn't mentioned, output `null`, don't hallucinate.

---

## n8n Node Setup

- **HTTP Request** node with `body` containing `messages` array (system + user)
- System message uses the fixed system prompt above
- User message injects `{{ $json.text }}` from the Whisper output
- Response is parsed and the `choices[0].message.content` is passed through a **JSON Parse** node (or `JSON.parse()` in a Code node) to extract the fields
- If parsing fails (fences, extra text), a fallback Code node strips markdown fences with regex before re-parsing
