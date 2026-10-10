# Значения входов

Значения передаются текстом — тул разбирает их по типу порта. Число, «да/нет» и член перечисления
тоже текстом.

| Тип порта | Что писать | Пример |
|---|---|---|
| Строка | текст как есть | `OST_Walls` |
| Число | точка как разделитель | `0.5` |
| Да/нет | `true` или `false` | `true` |
| Перечисление | имя члена | `TextNote` |
| Тип | короткое имя типа | `IElement` |

Пустое значение очищает вход. Если порт принимает связи, значение имеет смысл только пока связи
нет: подключённая связь его вытесняет.

Текстом задаётся не всё. Вектор, габариты, поток приходят только связью — тул откажет «только
связью». Тип, который выводится из связи другого входа (операнд B у `Comparison`), задаётся после
этой связи — до неё тул откажет «тип входа ещё не выведен»; `scheme_apply` ставит значения после
связей сам.

## Перечисления

| Перечисление | Члены | Где |
|---|---|---|
| `ElementKind` | `Any`, `Element`, `ElementType`, а также классы Revit: `Wall`, `Floor`, `Ceiling`, `Roof`, `Stairs`, `Railing`, `FamilyInstance`, `FamilySymbol`, `Family`, `Room`, `Space`, `Area`, `Level`, `Grid`, `View`, `ViewSheet`, `ScheduleSheetInstance`, `TextNote`, `Pipe`, `Duct`, `CableTray`, `Conduit`, `Group`, `RevitLinkInstance`, `ImportInstance`, `Material` | `ElementKindFilter.Kind`, по умолчанию `Any` |
| `CombineType` | `And`, `Or` | `CombineFilter.CombineType` |
| `ValueType` | `String`, `Double`, `Bool` | `ParameterValue.ParameterType` |
| `TextComparison` | `Equal`, `NotEqual`, `IsMatch`, `IsNotMatch` | оператор при текстовом сравнении |
| `NumberComparison` | `Equal`, `NotEqual`, `LessThan`, `LessThanOrEqual`, `GreaterThan`, `GreaterThanOrEqual` | оператор при числовом |
| `BooleanComparison` | `Equal`, `NotEqual` | оператор при «да/нет» |
| `BoundingBoxOperation` | `Move`, `Project`, `Expand`, `Contract` | `BoundingBoxTransform.Operation` |
| `BoundingBoxRelation` | `Intersects`, `Contains`, `Inside` | `BoundingBoxComparison.Relation` |

`IsMatch` и `IsNotMatch` — сравнение по регулярному выражению, а не по точному совпадению.

Текст во всех нодах сравнивается **без учёта регистра**, в регулярных выражениях тоже: `Ст-1` и
`ст-1` равны.

Набор операторов зависит от определения входа. У `Comparison` он следует за типом операнда A сам.
У `ParameterFilter` определение переключают у входа Value (`scheme_select_definition`, ключи
`string`, `double`, `bool`), и оператор следует за ним. Оператор из чужого перечисления — ошибка
схемы, а не молчаливое «нет».

## Координаты

Длины во всех нодах — в миллиметрах, площади — в квадратных метрах. Координаты `Vector` — числа в
миллиметрах: нормаль плана — X = 0, Y = 0, Z = 1.

## Категории

Категория пишется системным именем Revit: `OST_Walls`, `OST_Doors`, `OST_Rooms`. Несколько
категорий в одном фильтре перечисляются через `|`. Это дешевле, чем несколько фильтров через
`CombineFilter`, и не ломает быстрый фильтр.

## Словари

`Dictionary` хранит таблицу «ключ — значение» строк или чисел. Вид выбирается определением входа
(`string` или `double`). Значение приходит целиком, отдельными тулами строки не правятся.

## Параметры «да/нет»

Revit хранит их целым числом, а не логическим. Конструктор приводит это сам, и в выражениях для
таких параметров есть `AsBool()`. Сравнивать параметр «да/нет» с `1` не надо — сравнивай с
`true`/`false`, выбрав определение `bool`.
