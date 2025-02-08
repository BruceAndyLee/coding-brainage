---
Status: Not started
tags:
  - chore
  - ui
  - frontend
---
# Baselines

- focus on the experience of the new user - if they can figure it out without a tutorial, then you’ve set yourself a baseline for simplicity.
- attone to the meaning implied by the UI-control elements. Remember to group and separate the elements based on their implied intention.
    - **checkboxes** are for conditions
    - **switches** are for turning things on and off
    - **buttons** are for actions
- Think about accessibility at some point
    - the standard minimum color-contrast ratio is **4.5:1**

## Кнопки
- save/send/download - синий, позитивная ассоциация
- cancel - серый или черный, аутлайн без заполнения для большего контраста с основной кнопкой. Нейтральная ассоциация
  ![[Screenshot 2025-01-12 at 01.44.54.png|300]]
Цвета, которые должны быть прописаны в дизайн системе и воспроизводиться всегда:
- `button`: позитивный основной (send/save) - filled/синий
- `button`: нейтральный побочный (cancel) - outline/черный
- `button`: негативный основной (delete/override) - filled/red
- `button`: отключенные версии позитивного, нейтрального и негативного
- `text`: цвет заголовка и основного текста
- `text`: цвет отключенного текста
- `text`: цвет побочного текста (placeholder-ы)
- `text`: цвет негативного текста (сообщение об ошибке)
- `badge`: ??????
- `tab`: позитивный основной (активная вкладка)
- `tab`: нейтральный включенный (неактивная вкладка)
Utility
- переопределять переменные
- собирать сетап, чтобы не писать одни и те же пропы
- назначать цвета из ui-кита переменным и использовать их декларативно (primary, accent, error)
