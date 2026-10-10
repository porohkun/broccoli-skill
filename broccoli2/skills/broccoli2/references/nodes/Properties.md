# Properties — свойства объекта

Раскрывает свойства пришедшего объекта отдельными выходами. Главный переход схемы: из документов
в элементы, из элементов в их свойства.

Входы
- Object — объект или поток объектов, принимает что угодно.
- Distinct — галочка, убирать ли повторы. По умолчанию да. Касается свойств, где повторы
  осмысленно убирать: категория, ID, заголовок; на Elements и Parameters не влияет. Габариты
  нескольких спецификаций, которые ты потом сравниваешь, — с Distinct = false: равные габариты
  иначе схлопнутся.

Выходы
- Состав зависит от типа входа и появляется только после того, как вход подключён и его тип
  известен. Если вход — поток, все выходы тоже потоки.
- У документа: Elements, Parameters, Title, ModelPath, PathName, IsLink.
- У элемента: Id, Category (системное имя OST_Walls), CategoryName, IsElementType, OwnerViewId
  (ID вида-владельца, -1 — элемент модели), BoundingBox (габариты, мм), Parameters.
- У элемента уточнённого вида — ещё свойства вида. Поток уточняет `Where` с `ElementKindFilter`:
  элемент (не тип) — TypeName, FamilyName, Pinned; элемент типа — Name, FamilyName; стена — Width,
  IsCurtainWall, IsStackedWallMember; перекрытие — IsFoundationSlab; экземпляр семейства — HostId,
  SuperComponentId, Mirrored, IsInPlace; семейство — IsInPlace, IsEditable; помещение,
  пространство, зона — Number, Area, IsPlaced, у зоны AreaSchemeName; уровень — Name, Elevation;
  ось — Name; вид — Name, IsTemplate, ViewType, Scale, ViewTemplateId; лист — ещё SheetNumber,
  IsPlaceholder; спецификация на листе — IsTitleblockRevisionSchedule; текст — Text; труба —
  Diameter, SystemName; воздуховод — SystemName; лоток — Width, Height; короб — Diameter; группа —
  MemberCount; лестница — RisersCount, TreadsCount; материал — Name, MaterialClass,
  MaterialCategory; импорт — IsLinked, IsViewSpecific; экземпляр связи — IsLoaded. Длины — мм,
  площади — м².
- У габаритов (BoundingBox): Min, Max, Center, Size — точки Vector3, Volume, IsEmpty. У Vector3 —
  X, Y, Z, Length. Так до координаты габарита добираются цепочкой `Properties`.

Ограничения
- Пока вход не подключён, выходов нет. Над Object выходов нет вовсе — так бывает внутри
  селектора, пока тот не связан с потребителем.
- Выход Parameters у элемента есть, но при исполнении падает: все параметры элемента не читаются.
  Параметр берут `ParameterValue` по имени или отбирают `ParameterFilter`.
- ID в свойствах (OwnerViewId, HostId, ViewTemplateId) — числа long. В `Comparison` их не подать:
  он берёт только строку, `double` и да/нет. По ним группируют `GroupBy` или сравнивают в
  `ExpressionSelector`.
- Смысл габаритов — geometry.md.
