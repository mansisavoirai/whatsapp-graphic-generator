# Stability AI Image Generation — Premium Mode

## Configuration

| Parameter | Value |
|-----------|-------|
| **Endpoint** | `https://api.stability.ai/v1/generation/stable-diffusion-xl-1024-v1-0/text-to-image` |
| **Auth** | Header Auth — `Authorization: Bearer sk-...` |
| **Method** | `POST` |
| **Content-Type** | `application/json` |

---

## Positive Prompt

```handlebars
Professional product advertisement photography background, {{ $json.product_name }}, clean commercial studio lighting, high resolution, no text, no people, suitable for product overlay, white or neutral tones
```

`{{ $json.product_name }}` is injected via n8n's expression mode inside the HTTP Request body JSON.

---

## Negative Prompt

```
text, watermark, logo, people, faces, hands, low quality, blurry
```

### Why These Negatives

- **text / watermark / logo** — the generated image is a *background*. Text from the product info (name, price, tagline) is overlaid later by the HTML template. Any generated text looks bad and conflicts.
- **people / faces / hands** — hardware product graphics should focus on the product or a clean background. Hands holding products are common in SDXL outputs and don't work for this use case.
- **low quality / blurry** — standard quality enforcement.

---

## Request Body

```json
{
  "text_prompts": [
    {
      "text": "Professional product advertisement photography background, 8mm Ply Board, clean commercial studio lighting, high resolution, no text, no people, suitable for product overlay, white or neutral tones",
      "weight": 1
    },
    {
      "text": "text, watermark, logo, people, faces, hands, low quality, blurry",
      "weight": -1
    }
  ],
  "cfg_scale": 7,
  "height": 1920,
  "width": 1080,
  "samples": 1,
  "steps": 30
}
```

---

## Parameters

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| **width** | 1080 | WhatsApp Story / portrait format | 
| **height** | 1920 | 9:16 vertical — optimal for phone screens | 
| **steps** | 30 | Good detail without excessive latency (~4-6s on Stability API) | 
| **cfg_scale** | 7 | Strong adherence to prompt without oversaturation | 
| **samples** | 1 | One background is sufficient; latency matters for WhatsApp reply speed | 

---

## Response Handling

The API returns `artifacts` array. The first artifact with `finish_reason: "SUCCESS"` contains a base64-encoded PNG in `base64` field.

```json
{
  "artifacts": [
    {
      "base64": "iVBORw0KGgoAAAANSUhEUgAA...",
      "finishReason": "SUCCESS"
    }
  ]
}
```

In n8n: extract `artifacts[0].base64`, prepend `data:image/png;base64,` to create a data URI, then inject into the HTML template as the background image.

---

## Fallback

If the Stability API call fails (timeout, auth error, rate limit), the workflow falls back to a static HTML template with a CSS gradient background instead of an AI-generated image. This ensures the trader still gets a graphic back regardless.
