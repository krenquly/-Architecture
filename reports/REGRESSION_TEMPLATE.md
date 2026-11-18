# 04. Регрессионный отчёт

## Сборка и окружение
Node: v26.13.1 · Browser: Chrome · Commit: final

## Тестирование
- `npm test`: 18 / 18 пройдено.
- `npm run test:e2e`: 6 / 6 пройдено (или ручной прогон).
- 320/375/768/1024/1440/1920 px: overflow отсутствует.
- light/dark/reduced-motion/Tab/Shift+Tab/Space/Enter/Escape: пройдено.
- Нет глобального горизонтального скролла: да.
- Сторонний виджет сохранён и не влияет на библиотеку: да.

## Доказательства
| Проверка | Команда/сценарий | Результат | Скриншот |
|---|---|---|---|
| Контраст уведомлений | dark, DevTools contrast | OK | a2.png |
| Фокус переключателя | Tab + Space | OK | s1.png |
| Модальность | `:modal` | true | d1.png |
| Контракт кнопки | primary в #workbench | var(--color-accent) | btn1.png |
| Длинный текст | 320px | без overflow | c1.png |

## Известные ограничения
1. Vendor `!important` приходится переписывать вручную.
2. Контейнерные запросы не добавлены (бонус).
3. Визуальный regression screenshot не автоматизирован.

## До/после
Кнопка `Удалить` в `#workbench`: до — `#a41e24` от `!important` в legacy; после — `var(--color-danger)` из components. Причина победы: legacy в `layer(legacy)`, `!important` удалён, ID убран.