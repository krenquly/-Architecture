# ATLAS UI · CSS Architecture Rescue (ARCH-06)

**Финальная версия после рефакторинга.** Все 18 структурных тестов проходят, браузерные сценарии проверены, отчёты заполнены.

## Запуск

1. Установите Node.js 18 или новее.
2. В терминале из корня проекта:
   ```bash
   npm run dev
Откройте http://127.0.0.1:5173 в современном браузере.

Не открывайте HTML через file:// — используйте локальный сервер.

Тесты
Структурные проверки (18 тестов)
bash
npm test
Ожидаемый результат: 18 / 18 pass. Тесты проверяют:

порядок @layer;

изоляцию legacy.css в слое legacy;

отсутствие !important;

модальность диалога (showModal());

focus-visible для кнопки и переключателя;

доступность переключателя (без display: none);

использование токенов вместо хардкодных HEX;

отсутствие глобального .is-danger;

валидацию по aria-invalid, а не :invalid;

отсутствие мобильных min-width;

единственный базовый .ui-button;

prefers-reduced-motion;

отсутствие опасного :is() с ID;

нативные disabled;

все 7 семейств компонентов;

доступное имя диалога;

отсутствие .is-danger в HTML;

изоляцию vendor-селекторов.

Браузерные тесты (Playwright, опционально)
bash
npm install --save-dev @playwright/test
npx playwright install chromium
npm run test:e2e
Проверяются 6 сценариев: модальность и возврат фокуса, доступность переключателя, валидация после blur, отсутствие overflow на 6 ширинах, смена темы, закрытие/восстановление уведомлений.

Если Playwright не установлен — используйте ручной чек-лист из reports/REGRESSION_TEMPLATE.md.

Архитектура
Порядок слоёв
css
@layer reset, legacy, tokens, base, components, states, utilities;
reset — нормализация браузерных стилей.

legacy — изолированные стили подрядчика (.legacy-widget).

tokens — семантические переменные и темы (light / dark).

base — типографика, каркас страницы.

components — 7 семейств компонентов.

states — hover, focus-visible, active, disabled, aria-invalid.

utilities — атомарные классы (u-visually-hidden).

Ключевые решения
legacy.css импортируется через @import url('./legacy.css') layer(legacy); — unlayered правила больше не перебивают компоненты.

Удалены все !important и ID-селекторы из компонентов.

Все цвета — через var(--color-*) из tokens.css.

Переключатель — нативный input[type=checkbox], визуально скрыт, но доступен с клавиатуры.

Диалог — нативный <dialog> с showModal().

Валидация — через aria-invalid, управляемый JS после blur/submit.

Семейства компонентов
Семейство	Классы	Состояния
Кнопка	.ui-button, .ui-button--primary|secondary|ghost|danger, .ui-button--sm|lg	hover, focus-visible, active, disabled
Поле	.ui-field, .ui-field__hint, .ui-field__actions	aria-invalid, blur/submit
Карточка	.ui-card, .ui-card--featured, .ui-card__meta	длинный текст, вложенный alert
Тег	.ui-tag, .ui-tag--neutral|success|warning|danger	статичный (неинтерактивный)
Уведомление	.ui-alert, .ui-alert--info|success|warning|danger, .ui-alert__dismiss	закрытие, восстановление
Переключатель	.ui-switch, .ui-switch__track, .ui-switch__thumb	checked, disabled, focus-visible
Диалог	.ui-dialog, .ui-dialog__head, .ui-dialog__actions	open modal, closed
Артефакты сдачи
☑ Ссылка на GitHub: https://github.com/krenquly/ARCH/tree/fix/css-architecture
☑ Коммиты: baseline: inherited CSS conflicts → fix: complete ARCH-06 css architecture rescue
☑ Заполненные отчёты в reports/:
AUDIT_TEMPLATE.md — 12+ причин, 3 трассировки каскада.

ADR_TEMPLATE.md — сравнение 4 стратегий, решение, контрпримеры.

MATRIX_TEMPLATE.csv — 40+ сценариев.

REGRESSION_TEMPLATE.md — результаты тестов, ограничения, до/после.

☑ npm test — 18/18 pass.
☑ Скриншоты: reports/screenshots/before/, after/, traces/.
☑ Готовность к неожиданной правке на защите (см. scenario-changes.md).
Известные ограничения
Vendor !important приходится переписывать вручную — @layer не отменяет !important.

Container queries для карточек не добавлены (только медиазапросы).

Визуальный regression screenshot не автоматизирован.

Принцип честной проверки
Не разрешается:

удалять сложные компоненты или разметку;

менять тесты для получения зелёного результата;

отключать стили vendor;

вставлять !important;

скрывать горизонтальное переполнение на body без устранения причины.