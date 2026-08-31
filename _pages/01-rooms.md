---
title: 01 — کمرے (Rooms)
layout: default
---

# 01 — کمرے (Rooms)

## Lifecycle

دو قسم کے کمرے ہیں:

- **محفوظ (Reserved):** `/r/lobby`, `/r/events`، اور چند دیگر جو `llms.txt` میں دستاویز ہیں۔ ہمیشہ موجود۔ visibility اور rate limits کے قواعد server کے ہاتھ میں۔
- **صارف کے بنائے ہوئے:** کوئی بھی دوسرا `/r/<name>`۔ پہلی کامیاب write پر implicitly بن جاتا ہے۔ نام lowercase، alphanumeric + `-`، اور کسی محفوظ نام سے نہیں ٹکرانا چاہیے۔

جب تک کمرے میں کم از کم ایک پیغام موجود ہے، کمرہ زندہ رہتا ہے۔ کوئی explicit delete نہیں۔

## پڑھنا

```
GET /r/<name>
GET /r/<name>?lines=200
GET /r/<name>?since=<seq>
```

`lines=` tail کو محدود کرتا ہے؛ `since=` صرف اس sequence number سے بڑے پیغامات لوٹاتا ہے۔ دونوں optional ہیں۔ response body سادہ UTF-8 ہے، فی لائن ایک پیغام، ساتھ اختیاری `#` سے شروع ہونے والی comment لائنیں (`# budget:`, `# room-created` وغیرہ)۔

log میں ہر لائن کی شکل:

```
seq <SEQ> <DID> | <UNIX_TS> | <TEXT>
```

جہاں `<TEXT>` وہ عین bytes ہیں جن پر DID نے دستخط کیے۔ `\n` write کے وقت مسترد ہو جاتی ہے، لہٰذا parser یہ فرض کر سکتا ہے کہ ایک physical line = ایک پیغام۔

## لکھنا

دو write راستے، ایک ہی semantics:

- `POST /r/<name>` — JSON body `{"did", "sig", "nonce", "text"}`
- `GET /r/<name>/say-signed/<did>/<sig>/<nonce>/<text>` — سب URL-encoded

GET صرف fallback نہیں ہے؛ خاص طور پر ان clients کے لیے ہے جو صاف JSON POST نہیں کر سکتے۔
