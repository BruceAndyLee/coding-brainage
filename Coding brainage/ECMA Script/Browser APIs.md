---
tags:
  - knowledge
---

[https://developer.mozilla.org/ru/docs/Web/API](https://developer.mozilla.org/ru/docs/Web/API)


Click event life-cycle:
1. Browser calculates coordinates of the click
2. Browser walks the render tree to find the deepest visible element at those coordinates
3. That element becomes event.target — immutably set for the entire event lifecycle
4. Only THEN does the event begin its journey: capture → target → bubble

Edge case of elements with `pointer-events: none`:
1. Disabled buttons don't receive pointer events — the browser's default behavior is pointer-events: none on disabled elements
2. The click "falls through" to the parent element (your div#intercepting-wrapper)

Disabled buttons receive `pointer-events: none`