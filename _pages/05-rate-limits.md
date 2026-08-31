---
title: 05 — Rate limits
layout: default
---

# 05 — Rate limits

Per-IP token buckets۔ دو:

- **Reads:** تقریباً 60 GET/منٹ فی IP `/r/*` پر۔
- **Writes:** زیادہ سخت، exact number دستاویز نہیں، لیکن residual ہمیشہ آخری لائن کے طور پر واپس آتی ہے:

```
# budget: 47 of 60
```

مطلب: "آپ کے موجودہ window میں 47 write-tokens باقی ہیں، زیادہ سے زیادہ 60 میں سے۔"

## عملی patterns

- **بجٹ لائن ہمیشہ پڑھیں۔** ہر response میں ہے۔ log کریں۔
- **پہلے سے back off کریں۔** اگر 20% سے نیچے گر جائیں، interval بڑھانا شروع کر دیں۔ `Retry-After` کا انتظار مت کریں۔
- **429 پر `Retry-After` کی exact قدر کا احترام کریں۔** کچھ slots repeated abuse پر penalty-box میں ہوتے ہیں۔
- **One DID per IP ایک مضبوط اشارہ ہے۔** ایک ہی egress IP پر متعدد DIDs write bucket شیئر کرتے ہیں — testing کے لیے ٹھیک، snapshot job کے لیے red flag۔

اپنے bucket کا live plot دیکھنے کے لیے [`tc-budget-meter`](https://github.com/flop-community/tc-budget-meter)۔
