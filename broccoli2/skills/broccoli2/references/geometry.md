# Габариты

Геометрия в конструкторе — габариты: параллелепипед элемента вдоль осей, в миллиметрах. Других
форм нет: стена под углом — это её описанный прямоугольник, а не она сама.

## Откуда берутся и в каких координатах

Габариты — выход BoundingBox у `Properties` над элементом. Revit отдаёт их так:

| Элемент | Координаты |
|---|---|
| элемент модели | модели документа |
| элемент листа, чертёжного вида, легенды | этого вида: у листа — координаты листа |
| элемент другого вида (марка на плане) | модели документа, взятые на этом виде |
| элемент связи | основной модели или своего файла — по настройкам схемы, ниже |

Вид-владелец элемента — выход OwnerViewId (-1 — элемент модели). Элементы листа плоские: у них
Z = 0 в координатах листа.

Сравнивать можно только габариты в одной системе координат: элемент листа — с элементами того же
листа, элемент связи с собственными координатами — только с элементами той же связи.

## Когда габариты пусты

- У элемента нет геометрии: элемент типа, сам лист, вид.
- Revit отказал вычислить габариты — так бывает, например, у спецификации изменений в штампе
  листа.
- Связь вставлена в несколько мест, а документ на каждое место выключен: положения у её элементов
  нет.

В двух последних случаях вместо пустых габаритов проверку можно уронить настройкой
FailOnUndefinedBoundingBox. Пустые габариты ни с чем не пересекаются и ничего не содержат,
IsEmpty = true, углы — нули. «Нарушений не найдено» на пустых габаритах ничего не доказывает.

## Настройки схемы

Общие для всей схемы правила, `scheme_set_setting` или `settings` в описании `scheme_apply`. В
веб-редакторе их не видно: если инспекция от них зависит, напиши это текстом версии.

| Настройка | По умолчанию | Что делает |
|---|---|---|
| `BoundingBoxesInMainModel` | true | габариты элементов связей пересчитываются по месту вставки в координаты основной модели; false — остаются в координатах своего файла |
| `FailOnUndefinedBoundingBox` | false | габариты, которые не удалось вычислить, роняют проверку с текстом, какой элемент и почему; false — становятся пустыми |
| `DocumentPerLinkInstance` | false | связь, вставленная в несколько мест, даёт документ на каждое место, и габариты её элементов считаются по своему месту |

Цена `DocumentPerLinkInstance`: элемент такой связи проходит по схеме столько раз, сколько у связи
мест. В ошибках он окажется один раз — результат хранит только номер элемента, — а подсчёты и база
процента посчитают его на каждом месте, и процент выйдет выше настоящего. Документы мест вставки
ничем не различаются. На элементы листов, чертёжных видов и легенд настройки связей не действуют.

## Ноды

- `Properties` над габаритами даёт Min, Max, Center, Size — точки Vector3, — Volume и IsEmpty;
  над Vector3 — X, Y, Z, Length. Так до числа габарита добираются цепочкой `Properties`: высота —
  BoundingBox → Size → Z.
- `BoundingBoxTransform` сдвигает (Move), проецирует (Project), расширяет (Expand) и сжимает
  (Contract) габариты на вектор `Vector`. Проекция с нормалью (0, 0, 1) — положение на плане без
  высоты.
- `BoundingBoxComparison` проверяет отношение: Intersects (касание гранью — тоже пересечение),
  Contains, Inside. Допуска нет: допустимое наложение задают, сжав габарит на допуск нодой
  `BoundingBoxTransform` с операцией Contract.
- `ExpressionSelector` читает габариты свойствами: `e.BoundingBox.Size.Z > 2000`.

## Каждый с каждым

Сравнение двух наборов — селектор внутри селектора: для каждого объекта A перебираются все B.
Функция считается на каждом объекте заново, и по всей модели это N × M вызовов. Сначала разбей
объекты на группы, внутри которых сравнение осмысленно, — `GroupBy` по OwnerViewId (лист) или по
уровню, — и сравнивай внутри групп. Геометрия элементов разных листов несравнима всё равно: у
каждого листа свои координаты.

## Пример: тексты, наложенные на спецификацию

Проверка «текст на листе наложен на спецификацию того же листа». Элементы листов группируются по
листу, внутри группы каждый текст сравнивается с каждой спецификацией на плане.

Проверочная модель — два листа, на каждом спецификация и тексты, габариты в координатах листа:

```json
{"inspection": 7, "name": "два листа", "elements": [
  {"key": "sheet-1", "class": "ViewSheet", "category": "OST_Sheets"},
  {"key": "schedule-1", "class": "ScheduleSheetInstance", "category": "OST_ScheduleGraphics",
   "ownerView": "sheet-1", "boundingBox": {"min": [20, 10, 0], "max": [200, 120, 0]}},
  {"key": "text-over", "class": "TextNote", "category": "OST_TextNotes",
   "ownerView": "sheet-1", "boundingBox": {"min": [150, 100, 0], "max": [250, 110, 0]}},
  {"key": "text-free", "class": "TextNote", "category": "OST_TextNotes",
   "ownerView": "sheet-1", "boundingBox": {"min": [500, 500, 0], "max": [600, 520, 0]}},
  {"key": "sheet-2", "class": "ViewSheet", "category": "OST_Sheets"},
  {"key": "text-foreign", "class": "TextNote", "category": "OST_TextNotes",
   "ownerView": "sheet-2", "boundingBox": {"min": [50, 50, 0], "max": [100, 60, 0]}},
  {"key": "schedule-2", "class": "ScheduleSheetInstance", "category": "OST_ScheduleGraphics",
   "ownerView": "sheet-2", "boundingBox": {"min": [1000, 1000, 0], "max": [1200, 1100, 0]}}
]}
```

Эталон — ошибкой становится только `text-over`: `text-foreign` стоит там же, где спецификация
первого листа, но на другом листе.

Схема — одним `scheme_apply`:

```json
{"nodes": [
  {"name": "model", "type": "Model", "values": {"Links": "false"}},
  {"name": "doc", "type": "Properties", "links": {"Object": ["model.Result"]}},
  {"name": "isText", "type": "ElementKindFilter", "values": {"Kind": "TextNote"}},
  {"name": "isSchedule", "type": "ElementKindFilter", "values": {"Kind": "ScheduleSheetInstance"}},
  {"name": "textOrSchedule", "type": "CombineFilter", "values": {"CombineType": "Or"},
   "links": {"Input": ["isText.Result", "isSchedule.Result"]}},
  {"name": "onSheets", "type": "Where",
   "links": {"Input": ["doc.Elements"], "Selector": ["textOrSchedule.Result"]}},

  {"name": "sheetBase", "type": "CustomSelectorBase"},
  {"name": "sheetProps", "type": "Properties", "links": {"Object": ["sheetBase.Element"]}},
  {"name": "sheetKey", "type": "CustomSelector",
   "links": {"Base": ["sheetBase.Base"], "Output": ["sheetProps.OwnerViewId"]}},
  {"name": "bySheet", "type": "GroupBy",
   "links": {"Input": ["onSheets.Result"], "Selector": ["sheetKey.Result"]}},

  {"name": "normal", "type": "Vector", "values": {"Z": "1"}},

  {"name": "groupBase", "type": "CustomSelectorBase"},
  {"name": "textKind", "type": "ElementKindFilter", "values": {"Kind": "TextNote"}},
  {"name": "texts", "type": "Where",
   "links": {"Input": ["groupBase.Element"], "Selector": ["textKind.Result"]}},
  {"name": "scheduleKind", "type": "ElementKindFilter", "values": {"Kind": "ScheduleSheetInstance"}},
  {"name": "schedules", "type": "Where",
   "links": {"Input": ["groupBase.Element"], "Selector": ["scheduleKind.Result"]}},
  {"name": "scheduleProps", "type": "Properties", "values": {"Distinct": "false"},
   "links": {"Object": ["schedules.Result"]}},
  {"name": "scheduleBoxes", "type": "BoundingBoxTransform", "values": {"Operation": "Project"},
   "links": {"Input": ["scheduleProps.BoundingBox"], "Vector": ["normal.Result"]}},

  {"name": "textBase", "type": "CustomSelectorBase"},
  {"name": "textProps", "type": "Properties", "links": {"Object": ["textBase.Element"]}},
  {"name": "textBox", "type": "BoundingBoxTransform", "values": {"Operation": "Project"},
   "links": {"Input": ["textProps.BoundingBox"], "Vector": ["normal.Result"]}},

  {"name": "boxBase", "type": "CustomSelectorBase"},
  {"name": "cross", "type": "BoundingBoxComparison", "values": {"Relation": "Intersects"},
   "links": {"OperandA": ["textBox.Result"], "OperandB": ["boxBase.Element"]}},
  {"name": "crossSelector", "type": "CustomSelector",
   "links": {"Base": ["boxBase.Base"], "Output": ["cross.Result"]}},
  {"name": "crossed", "type": "Where",
   "links": {"Input": ["scheduleBoxes.Result"], "Selector": ["crossSelector.Result"]}},
  {"name": "crossCount", "type": "Length", "links": {"Input": ["crossed.Result"]}},
  {"name": "overlaps", "type": "Comparison", "values": {"Operator": "GreaterThan", "OperandB": "0"},
   "links": {"OperandA": ["crossCount.Result"]}},
  {"name": "textSelector", "type": "CustomSelector",
   "links": {"Base": ["textBase.Base"], "Output": ["overlaps.Result"]}},
  {"name": "overlapping", "type": "Where",
   "links": {"Input": ["texts.Result"], "Selector": ["textSelector.Result"]}},

  {"name": "groupSelector", "type": "CustomSelector",
   "links": {"Base": ["groupBase.Base"], "Output": ["overlapping.Result"]}},
  {"name": "flat", "type": "SelectMany",
   "links": {"Input": ["bySheet.Result"], "Selector": ["groupSelector.Result"]}},
  {"name": "error", "type": "SaveError", "values": {"Text": "Текст наложен на спецификацию"},
   "links": {"Input": ["flat.Result"]}}
]}
```

Что здесь важно:

- Отбор по листу — группировка: `GroupBy` по OwnerViewId, затем `SelectMany` с селектором над
  группой. Без групп каждый текст сравнивался бы со спецификациями всех листов — и в чужих
  координатах.
- Габариты спецификаций проецируются один раз на группу, снаружи селектора над текстом: они от
  текста не зависят. Distinct у их `Properties` выключен — иначе равные габариты двух
  спецификаций схлопнулись бы.
- Три селектора вложены: над группой, над текстом, над габаритом спецификации. Тело каждого —
  ноды от его базы.
- Выход селектора над группой — поток (отбор `Where`), поэтому потребитель — `SelectMany`, а не
  `Where`.
