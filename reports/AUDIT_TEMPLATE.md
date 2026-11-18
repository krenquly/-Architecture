# 01. Аудит CSS-архитектуры

Автор / группа: Колпаков Николай Алексевич 2024-ФГиИБ-ИС-2б · Репозиторий: [GitHub] · Commit до правок: baseline

## Гипотеза о причине (до изменения кода)

`legacy.css` импортируется без `layer(...)`, поэтому его правила находятся в unlayered-пространстве и по каскаду сильнее любых `@layer`. Внутри есть `!important`, который переворачивает порядок слоёв. `@layer` решает проблему изоляции обычных правил, но не решает проблему `!important` в раннем слое — его нужно переписывать.

## Инвентаризация дефектов

| ID | Симптом и шаги воспроизведения | Файл:строка | Применённое правило и выигравший каскад | Первопричина | Минимальное исправление | Регрессионный тест |
|---|---|---|---|---|---|---|
| A-01 | Кнопка primary в `#workbench` тёмно-синяя | legacy.css | `#workbench .ui-button` (unlayered, 1-1-0) > `.ui-button--primary` (components, 0-1-0) | unlayered + ID | Импорт legacy в `layer(legacy)`; убрать ID | Визуальный тест цвета |
| A-02 | Кнопка danger не реагирует на hover | legacy.css | `#workbench .ui-button.is-danger { background: #a41e24 !important; }` | `!important` в unlayered | Убрать `!important`, ограничить `.legacy-widget` | E2E hover |
| A-03 | Метка danger выглядит как кнопка | states.css | `.is-danger { background: #c12236; }` (states) > `.ui-tag--danger` (components) | Глобальный класс состояния | Удалить `.is-danger`, использовать `.ui-tag--danger` | Визуальный тест тега |
| A-04 | Текст в карточке мелкий | legacy.css | `.ui-card p { font-size: .69rem; }` (unlayered) | unlayered-правило | Удалить или ограничить `.legacy-widget` | Computed font-size |
| A-05 | Ошибка required видна до blur | states.css | `.ui-field:has(input:invalid) input` | `:invalid` срабатывает сразу | `:has(input[aria-invalid="true"])` + JS | E2E: до blur нет ошибки |
| A-06 | Фокус не виден на кнопках | states.css | `.ui-button:focus { outline: none; }` | `:focus` без замены | `:focus-visible` с outline | Tab-навигация |
| A-07 | Переключатель недоступен с клавиатуры | components.css | `.ui-switch input { display: none; }` | `display:none` убирает из a11y-дерева | `position:absolute; opacity:0` + `:focus-visible + track` | E2E Tab/Space |
| A-08 | Диалог не модальный | js/app.js | `dialog.show()` | Неверный метод | `dialog.showModal()` | E2E `:modal` |
| A-09 | Горизонтальный скролл на 320px | components.css | `.ui-field input { min-width: 20rem; }`, `.ui-card { min-width: 22rem; }` | Жёсткие min-width | `min-width: 0; width: 100%` | E2E overflow |
| A-10 | Дубль базового `.ui-button` | components.css | Два правила `.ui-button` | Дублирование | Объединить | `npm test` #11 |
| A-11 | Уведомления нечитаемы в dark | components.css | `.ui-alert { color: #111827; }` | Хардкод цвета | `var(--color-ink)` | Контраст в dark |
| A-12 | У тегов ложный hover | states.css | `.ui-tag:hover { filter: brightness(.94); cursor: pointer; }` | Неинтерактивный элемент | Удалить правило | Визуальный тест |
| A-13 | `.is-danger` в HTML | index.html | `class="... is-danger"` | Смешение модификатора и глобального класса | Убрать `is-danger` из разметки | `npm test` #17 |
| A-14 | `:is(.ui-card, #workbench) .ui-tag` | states.css | ID внутри `:is()` повышает вес до 1-1-0 | Ловушка `:is()` | Заменить на `.ui-card .ui-tag` или `.ui-tag` | `npm test` #13 |

## Три трассировки каскада

### 1. primary button в legacy workbench
- **Элемент:** `<button class="ui-button ui-button--primary">` внутри `#workbench`.
- **Свойство:** `background-color`.
- **Кандидаты:**
  1. `#workbench .ui-button` — origin author, normal, **unlayered**, specificity 1-1-0.
  2. `.ui-button--primary` — origin author, normal, **layer components**, specificity 0-1-0.
- **Победитель:** `#workbench .ui-button`. Причина: unlayered normal сильнее любого слоя; специфичность уже не важна.
- **Скриншот:** `reports/screenshots/traces/chain1.png`.

### 2. danger tag во вложенной карточке
- **Элемент:** `<span class="ui-tag ui-tag--danger is-danger">`.
- **Свойство:** `background-color`.
- **Кандидаты:**
  1. `.is-danger` — layer states, 0-1-0.
  2. `.ui-tag--danger` — layer components, 0-1-0.
- **Победитель:** `.is-danger`. Причина: states объявлен позже components → для normal правил более поздний слой сильнее.
- **Скриншот:** `reports/screenshots/traces/chain2.png`.

### 3. input required до взаимодействия
- **Элемент:** `<input type="email" required>`.
- **Свойство:** `border-color`.
- **Кандидаты:**
  1. `.ui-field:has(input:invalid) input` — layer states, 0-2-1.
  2. `.ui-field input` — layer components, 0-1-1.
- **Победитель:** `:has(input:invalid)`. Причина: `:invalid` присутствует у пустого required сразу; более сильный селектор не лечит UX, нужно менять триггер состояния.
- **Скриншот:** `reports/screenshots/traces/chain3.png`.

## Вывод

Проблема архитектуры: unlayered legacy + глобальные `.is-danger` + `!important`. Проблема JS/семантики: `show()` вместо `showModal()`, `display:none` у checkbox, `:invalid` вместо управляемого `aria-invalid`.