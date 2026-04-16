# Алгоритм Вороного и архитектура проекта

Здесь объяснено КАК работает весь проект: от нажатия Play до появления карты.

---

## Аналогия: что такое диаграмма Вороного

Представь, что у тебя есть большое поле. Ты бросаешь на него N камней. Каждый камень — это "точка-сайт". Теперь для каждой точки на поле ты определяешь: *к какому камню ближе всего?* Все точки поля, ближайшие к одному камню, образуют "клетку" этого камня.

Результат — карта разбита на многоугольники (клетки Вороного). Как пиццу режут на куски — только куски неправильной формы.

**Зачем это для игры?** Такие клетки идеально подходят для биомов карты: они органично выглядят, никогда не пересекаются, и каждая имеет соседей.

---

## Алгоритм в коде: half-plane intersection (Сазерленд-Ходжман)

### Простое объяснение

Для каждой точки A:
1. Начинаем с прямоугольника всей карты (4 вершины)
2. Берём каждого соседа B:
   - Находим середину отрезка A-B (`midpoint = (A + B) / 2`)
   - Находим нормаль к отрезку A-B (перпендикуляр)
   - Отрезаем часть многоугольника, которая "ближе к B чем к A"
   - Остаток — уже меньший многоугольник
3. После всех соседей — оставшийся многоугольник = клетка A

**Проверка "с какой стороны точка":**
```
dot(point - midpoint, normal) <= 0   → точка на стороне A (оставляем)
dot(point - midpoint, normal) >  0   → точка на стороне B (отрезаем)
```

Это одна строчка кода: `Vector2.Dot(cur - mid, n) <= 0`

### Алгоритм Сазерленда-Ходжмана (для отсечения)

Для каждого ребра многоугольника (current → next):
- Оба внутри → добавь next
- Снаружи → внутри → добавь пересечение + next  
- Внутри → снаружи → добавь только пересечение
- Оба снаружи → ничего

**Пересечение ребра с линией:**
```
t = dot(midpoint - current, normal) / dot(direction, normal)
intersection = current + direction * t
```

---

## Как строится 3D меш из 2D клетки

### Веерная триангуляция (Fan Triangulation)

Клетка — это 2D многоугольник (список точек). Чтобы сделать 3D поверхность:

1. Добавляем центральную точку (среднее всех вершин)
2. Для каждого ребра создаём треугольник: центр → вершина[i] → вершина[i+1]

Как пицца: центр → кусок.

```
triangles[i * 3 + 0] = centerIndex      // вершина N (центр)
triangles[i * 3 + 1] = (i + 1) % n     // следующая вершина
triangles[i * 3 + 2] = i               // текущая вершина
```

### Боковые стенки

Чтобы клетка не была "летающей плашкой" — добавляем нижние вершины:
- Верхние вершины индексы: `0..n-1` (с высотой Perlin)
- Центральная вершина: индекс `n`
- Нижние вершины индексы: `n+1..2n` (Y=0)

Боковые треугольники (два на каждое ребро):
```
t0=i, t1=(i+1)%n, f0=n+1+i, f1=n+1+(i+1)%n
треугольник 1: t0, t1, f1
треугольник 2: t0, f1, f0
```

### Perlin noise для высот

`Mathf.PerlinNoise(x * 0.05f + offsetX, y * 0.05f + offsetY) * heightVariation`

- `x, y` — координаты вершины на карте
- `0.05f` — масштаб шума (меньше = более плавные холмы)
- `offsetX, offsetY` — случайное смещение (разное при каждой генерации!)
- `heightVariation` — максимальная высота холмов (из BiomeType)

Почему shared offset? Если у каждой клетки свой offset, на стыках будут "стены". Один общий offset = плавный переход между клетками.

---

## Архитектура проекта — все 18 файлов

### Слои архитектуры
```
UI (MonoBehaviour)          — только отображение и пользовательский ввод
   ↓
GameLauncher (Composition Root) — создаёт всё, соединяет всё
   ↓
MapGenerator (Orchestrator) — запускает шаги по очереди
   ↓
IMapGenerationStep          — контракт для каждого шага
   ↓
Steps/ (конкретные шаги)    — реализуют шаги
   ↓
Algorithms/ (чистая математика) — Вороной, меш
   ↓
Data/ (модели данных)       — хранят данные, нет логики
   ↓
Persistence/ (хранилище)    — читают/пишут JSON
```

---

## Описание каждого файла

### 1. `Data/MapSettings.cs`
**Что делает:** Хранит все настройки генерации карты + атрибуты игрока.

Это просто структура данных — никакой логики. Помечена `[Serializable]` чтобы `JsonUtility` мог её сериализовать.

Поля:
- `pointCount` — сколько клеток Вороного (50 по умолчанию)
- `mapSize` — размер карты в Unity units (100)
- `biomeZones` — сколько биомов (5)
- `objectPlacementAttempts` — попыток поставить объект в клетке (10)
- `visualizeSteps` — замедленная визуализация шагов
- `maxHealth`, `maxHunger`, `maxThirst` — максимумы атрибутов
- `hpLossOnHunger`, `hpLossOnThirst` — урон HP при 0 голода/жажды

---

### 2. `Data/VoronoiCell.cs`
**Что делает:** Модель одной клетки Вороного.

Поля:
- `Site` — 2D точка-центр клетки (Vector2)
- `Polygon` — список вершин многоугольника (List<Vector2>)
- `BiomeIndex` — индекс биома в массиве ctx.Biomes

Нет конструктора, нет методов — только данные.

---

### 3. `Data/BiomeType.cs`
**Что делает:** ScriptableObject с настройками одного биома.

`[CreateAssetMenu]` добавляет пункт в ПКМ меню для создания файла.

Поля:
- `biomeName` — название ("Grass", "Forest")
- `groundColor` — цвет земли
- `heightVariation` — высота холмов (0 = плоско, 4 = горы)
- `isStartBiome` — стартовая зона (безопасная)
- `hasWater` — в этом биоме вода (для рыбалки)
- `objectPrefabs` — массив префабов объектов (деревья, камни)

---

### 4. `Interfaces/IMapGenerationStep.cs`
**Что делает:** Контракт для каждого шага генерации.

```csharp
public interface IMapGenerationStep
{
    string Name { get; }
    IEnumerator Execute(GenerationContext ctx);
}
```

`Name` — строка для логов ("Initialize", "Voronoi", "Terrain").  
`Execute` возвращает `IEnumerator` чтобы можно было делать паузы (`yield return null`) для визуализации.

Почему интерфейс? `MapGenerator` не знает про конкретные шаги — только про контракт. Можно добавить новый шаг не меняя `MapGenerator`.

---

### 5. `Interfaces/ISettingsRepository.cs`
**Что делает:** Контракт для хранилища настроек.

```csharp
public interface ISettingsRepository
{
    MapSettings Load();
    void Save(MapSettings settings);
}
```

Всего 2 метода. Хочешь хранить в XML — создай `XmlSettingsRepository`. `GameLauncher` не изменится.

---

### 6. `Algorithms/VoronoiGenerator.cs`
**Что делает:** Чистая математика. Принимает параметры, возвращает список клеток.

**Не знает про:** Unity сцену, MonoBehaviour, Transform, ScriptableObject.  
**Знает про:** Vector2, Random, List.

Методы:
- `Generate(count, size, seed)` — создаёт N случайных точек, строит клетки
- `ClipToRect(size)` — возвращает прямоугольник карты
- `ClipByBisector(poly, a, b)` — отрезает полигон по биссектрисе A-B

Все методы `static` — не нужен экземпляр класса.

---

### 7. `Algorithms/CellMeshBuilder.cs`
**Что делает:** Строит Unity Mesh из 2D многоугольника.

Метод `Build(polygon, heightNoise, offsetX, offsetY)`:
1. Создаёт верхние вершины с Perlin высотой
2. Создаёт нижние вершины (Y=0)
3. Добавляет центральную вершину
4. Строит треугольники (веер для верха, пары для боков)
5. Вызывает `RecalculateNormals()` для правильного освещения
6. Возвращает готовый Mesh

---

### 8. `GenerationContext.cs`
**Что делает:** "Рюкзак" с данными, которые передаются между шагами.

Это просто класс с публичными свойствами. Каждый шаг может читать и писать в него.

Свойства:
- `Settings` — настройки генерации
- `Biomes` — массив биомов
- `Root` — Transform родительского объекта (GameManager)
- `Cells` — список сгенерированных клеток (заполняет VoronoiStep)
- `NoiseOffsetX/Y` — случайное смещение шума (создаётся в GameLauncher один раз)
- `PlayerPrefab` — префаб игрока

---

### 9. `MapGenerator.cs`
**Что делает:** MonoBehaviour-оркестратор. Запускает шаги по очереди.

Метод `Generate()`:
```csharp
foreach (var step in _steps)
{
    yield return StartCoroutine(step.Execute(_context));
}
```

Это всё что он делает. Не знает про Вороной, биомы, префабы. Только про список шагов и контекст.

`SetContext()` и `SetSteps()` — вызываются из `GameLauncher` в `Awake()`.

---

### 10. `GameLauncher.cs`
**Что делает:** Composition Root. Создаёт все зависимости и соединяет их.

В `Awake()`:
1. Получает `MapGenerator` (RequireComponent)
2. Создаёт `JsonSettingsRepository`
3. Загружает настройки из JSON
4. Создаёт `GenerationContext` со всеми данными
5. Создаёт список шагов (`new VoronoiStep()`, etc.)
6. Передаёт контекст и шаги в `MapGenerator`
7. Если `autoGenerateOnStart` — запускает генерацию

Публичный метод `StartWithSettings(settings)` — для UI: когда игрок нажал "Играть" в меню настроек.

---

### 11. `Steps/InitializeStep.cs`
**Что делает:** Очищает сцену перед повторной генерацией.

Уничтожает дочерние объекты `ctx.Root` с именами: "Terrain", "Walls", "Objects", "Player".  
Очищает `ctx.Cells`.

Почему нужен: если нажать "Играть" снова, без очистки карта нарисуется поверх старой.

---

### 12. `Steps/VoronoiStep.cs`
**Что делает:** Запускает алгоритм Вороного, сохраняет клетки в контекст.

1. Вызывает `VoronoiGenerator.Generate(...)` 
2. Сохраняет результат в `ctx.Cells`
3. (Опционально) Рисует контуры клеток через LineRenderer

---

### 13. `Steps/BiomeAssignStep.cs`
**Что делает:** Назначает биом каждой клетке по шуму Перлина.

```csharp
float noise = Mathf.PerlinNoise(site.x * 0.02f + ctx.NoiseOffsetX, ...)
int biomeIndex = Mathf.FloorToInt(noise * biomeCount)
```

Perlin возвращает 0..1, умножаем на количество биомов, округляем вниз.

---

### 14. `Steps/TerrainStep.cs`
**Что делает:** Создаёт 3D меш для каждой клетки.

Для каждой клетки:
1. Создаёт `GameObject` → `Terrain/Cell_N`
2. Добавляет `MeshFilter`, `MeshRenderer`, `MeshCollider`
3. Вызывает `CellMeshBuilder.Build(...)` для меша
4. Устанавливает цвет материала (из биома ± 15% вариация)

---

### 15. `Steps/WallsStep.cs`
**Что делает:** Ставит 4 куба по границам карты.

```
Северная стена: Position(mapSize/2, 5, mapSize), Scale(mapSize, 10, 1)
Южная стена:   Position(mapSize/2, 5, 0),       Scale(mapSize, 10, 1)
Восточная стена: Position(mapSize, 5, mapSize/2), Scale(1, 10, mapSize)
Западная стена:  Position(0, 5, mapSize/2),       Scale(1, 10, mapSize)
```

Использует `GameObject.CreatePrimitive(PrimitiveType.Cube)`.

---

### 16. `Steps/ObjectsStep.cs`
**Что делает:** Инстанциирует объекты биомов (деревья, камни, трава).

Для каждой клетки:
1. Берёт `ctx.Biomes[cell.BiomeIndex].objectPrefabs`
2. Делает N попыток разместить объект (N = `ctx.Settings.objectPlacementAttempts`)
3. Каждая попытка: случайная точка → проверка что внутри полигона → `Instantiate`

Проверка "внутри полигона" — алгоритм ray casting:
Пускаем луч вправо и считаем пересечения с рёбрами. Чётное число = снаружи, нечётное = внутри.

---

### 17. `Steps/PlayerSpawnStep.cs`
**Что делает:** Спавнит игрока в стартовом биоме.

1. Ищет клетку где `ctx.Biomes[cell.BiomeIndex].isStartBiome == true`
2. Вычисляет центр полигона
3. Если есть `ctx.PlayerPrefab` — `Instantiate(prefab, position, rotation)`
4. Если нет — создаёт капсулу (для тестирования)

---

### 18. `Persistence/JsonSettingsRepository.cs`
**Что делает:** Читает/пишет JSON из StreamingAssets.

`Load()`:
- Проверяет есть ли файл
- Читает текст: `File.ReadAllText(path)`
- Десериализует: `JsonUtility.FromJson<MapSettings>(text)`
- Если файла нет или ошибка — возвращает `new MapSettings()` (дефолтные значения)

`Save(settings)`:
- Сериализует: `JsonUtility.ToJson(settings, true)` (true = pretty print)
- Пишет: `File.WriteAllText(path, json)`

---

## Последовательность при нажатии Play

```
Unity запускает Awake() на GameManager
  → GameLauncher.Awake()
      → создаёт JsonSettingsRepository
      → загружает MapSettings из mapSettings.json
      → создаёт GenerationContext
      → создаёт список из 7 шагов
      → передаёт всё в MapGenerator
      → вызывает StartCoroutine(mapGenerator.Generate())
  
  → MapGenerator.Generate() (корутина)
      → yield InitializeStep.Execute()     // очищает сцену
      → yield VoronoiStep.Execute()        // строит клетки
      → yield BiomeAssignStep.Execute()    // назначает биомы
      → yield TerrainStep.Execute()        // строит меши
      → yield WallsStep.Execute()          // ставит стены
      → yield ObjectsStep.Execute()        // ставит объекты
      → yield PlayerSpawnStep.Execute()    // спавнит игрока
```

Каждый шаг завершается прежде чем начнётся следующий (`yield return StartCoroutine`).

---

## SOLID на примерах проекта

### S — Single Responsibility (Единственная ответственность)
| Класс | Одна ответственность |
|-------|---------------------|
| `VoronoiGenerator` | Только математика Вороного |
| `CellMeshBuilder` | Только построение меша |
| `JsonSettingsRepository` | Только чтение/запись JSON |
| `MapGenerator` | Только запуск шагов в порядке |
| `GameLauncher` | Только создание зависимостей |

❌ Плохо: `VoronoiGenerator` который ещё и создаёт GameObject'ы  
✅ Хорошо: `VoronoiGenerator` возвращает List<VoronoiCell>, а `TerrainStep` создаёт объекты

### O — Open/Closed (Открыт для расширения, закрыт для модификации)
Нужно добавить животных? Создай новый файл `AnimalsStep.cs`. Не трогай `MapGenerator.cs`.

```csharp
// В GameLauncher.cs добавить ОДНУ строку в список шагов:
new AnimalsStep(),
```

### L — Liskov Substitution (Принцип подстановки Лисков)
`MapGenerator` принимает `List<IMapGenerationStep>`. Любой шаг можно заменить на другой.
Нельзя: шаг который бросает исключение вместо выполнения, или игнорирует контекст.

### I — Interface Segregation (Разделение интерфейсов)
`IMapGenerationStep` — только `Name` и `Execute`. Не `Initialize`, не `Cleanup`, не `Validate`.
Если бы у интерфейса было 10 методов, каждый шаг должен был бы реализовать 10 методов.

### D — Dependency Inversion (Инверсия зависимостей)
```csharp
// Плохо: MapGenerator знает про конкретные классы
private VoronoiStep _step1;
private BiomeAssignStep _step2;

// Хорошо: MapGenerator знает только про интерфейс
private List<IMapGenerationStep> _steps;
```

`GameLauncher` — единственное место где создаются `new VoronoiStep()`. Это Composition Root.
