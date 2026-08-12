# Trader Onboarding — WhatsApp Message

This message is sent as the first response when a new number interacts with the system.

---

## English

```
👋 *Welcome to Savoir AI!*

Send me a product photo + a voice note describing it, and I'll send you back a professional product graphic ready to share on WhatsApp.

How to use:
1. Send a photo of your product
2. Record a voice note saying the product name, price, and a short description
3. Get your graphic back in seconds

Example:
📸 Photo of 8mm ply board
🎙️ Voice note: "8mm ply board, ₹45 per square foot, moisture resistant, stock available"

When available, Premium AI backgrounds are used. Otherwise, a stylish template is generated.

Just send your first product photo + voice note to get started!
```

---

## Hindi (Devanagari)

```
👋 *Savoir AI में आपका स्वागत है!*

अपने प्रोडक्ट की फोटो + वॉइस नोट भेजें, और हम आपको WhatsApp पर शेयर करने के लिए एक प्रोफेशनल प्रोडक्ट ग्राफ़िक वापस भेज देंगे।

कैसे इस्तेमाल करें:
1. अपने प्रोडक्ट की फोटो भेजें
2. वॉइस नोट रिकॉर्ड करें — प्रोडक्ट का नाम, दाम, और छोटा विवरण बताएं
3. कुछ सेकंड में अपना ग्राफ़िक पाएं

उदाहरण:
📸 8mm प्लाई बोर्ड की फोटो
🎙️ वॉइस नोट: "8mm प्लाई बोर्ड, ₹45 प्रति स्क्वेयर फुट, मॉइस्चर रेजिस्टेंट, स्टॉक में उपलब्ध"

जब उपलब्ध हो, तो Premium AI बैकग्राउंड इस्तेमाल होता है। वरना एक स्टाइलिश टेम्पलेट बनाया जाता है।

शुरू करने के लिए बस अपनी पहली प्रोडक्ट फोटो + वॉइस नोट भेजें!
```

---

## Notes

- Message uses WhatsApp markdown: `*bold*`, `_italic_`
- Keep tone practical — these are hardware wholesalers, not tech enthusiasts
- Hindi version is not a word-for-word translation; it uses natural spoken Hindi
- The message is stored as a string in the n8n workflow and sent via Twilio as the first response in the `else` branch (no media attached = first-time user)
