---
tags:
  - es
  - javascript
  - ecmascript
  - browserAPI
---

Для кодирования и раскодирования стремных символов

```js
const utf8encoder = new TextEncoder();

utf8encoder.encode("€");
// 226,130,172

utf8encoder.encode("🫨");
// 240,159,171,168
```