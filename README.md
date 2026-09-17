# Маркетплейс Broccoli2

Плагины Claude Code для работы с конструктором инспекций.

| Плагин | Что даёт |
|---|---|
| `broccoli2` | Скилл с документацией по конструктору Broccoli2: устройство схемы, все ноды, селекторы, два мира компиляции, быстрые фильтры и работа тулами BroccoliMcp |

## Подключение

```bash
claude plugin marketplace add porohkun/broccoli-skill
claude plugin install broccoli2@porohkun
```

Локально, из рабочей копии, — путём к папке:

```bash
claude plugin marketplace add ./Skill
```

## Что внутри

```
.claude-plugin/marketplace.json     перечень плагинов
broccoli2/
  .claude-plugin/plugin.json        манифест плагина
  skills/broccoli2/
    SKILL.md                        обзор, порядок работы, таблица нод
    references/
      scheme.md                     порты, определения, разрешение типов
      selectors.md                  селекторы и согласование их типов
      expression-world.md           мир потока и мир выражения
      fast-filters.md               что сворачивается в запрос к Revit
      values.md                     значения входов и перечисления
      mcp.md                        тулы BroccoliMcp
      troubleshooting.md            что смотреть, когда не сходится
      inspections.md                версии, статусы, результат проверки
      nodes/<Тип>.md                по файлу на каждую ноду
```

## Правка

Документация описывает поведение конструктора, а не желаемое: всё, что здесь написано, проверено
по коду конструктора. Меняется конструктор — меняется и она, иначе агент будет
уверенно делать не то.

Состав файлов в `nodes/` обязан совпадать с составом нод конструктора. Появилась нода — появился
файл и строка в таблице `SKILL.md`.
