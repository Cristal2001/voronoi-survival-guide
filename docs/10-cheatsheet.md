# Шпаргалка — день соревнования

Одна страница. Всё самое важное.

---

## Ключевые формулы (учи наизусть)

### Perlin noise для высот
```csharp
float H(float x, float y) =>
    Mathf.PerlinNoise(x * 0.05f + offsetX, y * 0.05f + offsetY) * heightVariation;
```

### Тест принадлежности полуплоскости (half-plane)
```csharp
bool isInside = Vector2.Dot(point - midpoint, normal) <= 0;
```

### Веерная триангуляция (Fan Triangulation) — верхняя поверхность
```csharp
triangles[i * 3 + 0] = centerIndex;   // центр (индекс n)
triangles[i * 3 + 1] = (i + 1) % n;  // следующая вершина
triangles[i * 3 + 2] = i;             // текущая вершина
```

### Боковые стенки меша
```csharp
// t0=i, t1=(i+1)%n, f0=n+1+i, f1=n+1+(i+1)%n
triangles[off + i*6 + 0] = t0;  triangles[off + i*6 + 1] = t1;  triangles[off + i*6 + 2] = f1;
triangles[off + i*6 + 3] = t0;  triangles[off + i*6 + 4] = f1;  triangles[off + i*6 + 5] = f0;
```

### Ray casting — точка внутри полигона
```csharp
bool isInside = false;
for (int i = 0, j = n - 1; i < n; j = i++)
{
    if (((poly[i].y > pt.y) != (poly[j].y > pt.y)) &&
        (pt.x < (poly[j].x - poly[i].x) * (pt.y - poly[i].y) / (poly[j].y - poly[i].y) + poly[i].x))
        isInside = !isInside;
}
```

### Biome index из Perlin noise
```csharp
float noise      = Mathf.PerlinNoise(nx * 3f + offset, ny * 3f + offset);
int   biomeIndex = Mathf.Clamp(Mathf.FloorToInt(noise * biomeCount), 0, biomeCount - 1);
```

### Пересечение ребра с линией (Sutherland-Hodgman)
```csharp
Vector2 dir   = next - cur;
float   t     = Vector2.Dot(midpoint - cur, normal) / Vector2.Dot(dir, normal);
Vector2 inter = cur + dir * t;
```

---

## SOLID — одним предложением каждый

| Буква | Правило | Пример |
|-------|---------|--------|
| **S** | Один класс = одна ответственность | `VoronoiGenerator` только математика, не сцена |
| **O** | Новый шаг = новый файл, старые не трогать | `AnimalsStep.cs` без изменений `MapGenerator.cs` |
| **L** | Все `IMapGenerationStep` взаимозаменяемы | Нельзя бросать исключение вместо выполнения |
| **I** | `IMapGenerationStep` только `Name + Execute`, не 10 методов | Маленький интерфейс = легко реализовывать |
| **D** | `MapGenerator` знает только `IMapGenerationStep`, не `VoronoiStep` | `GameLauncher` = единственный `new VoronoiStep()` |

---

## Класс → Файл (все 18)

| Класс | Файл |
|-------|------|
| `MapSettings` | `Data/MapSettings.cs` |
| `VoronoiCell` | `Data/VoronoiCell.cs` |
| `BiomeType` | `Data/BiomeType.cs` |
| `GameSettings` | `Data/GameSettings.cs` |
| `SaveData` | `Data/SaveData.cs` |
| `CraftRecipe` | `Data/CraftRecipe.cs` |
| `IMapGenerationStep` | `Interfaces/IMapGenerationStep.cs` |
| `ISettingsRepository` | `Interfaces/ISettingsRepository.cs` |
| `VoronoiGenerator` | `Algorithms/VoronoiGenerator.cs` |
| `CellMeshBuilder` | `Algorithms/CellMeshBuilder.cs` |
| `GenerationContext` | `GenerationContext.cs` |
| `MapGenerator` | `MapGenerator.cs` |
| `GameLauncher` | `GameLauncher.cs` |
| `JsonSettingsRepository` | `Persistence/JsonSettingsRepository.cs` |
| `InitializeStep` | `Steps/InitializeStep.cs` |
| `VoronoiStep` | `Steps/VoronoiStep.cs` |
| `BiomeAssignStep` | `Steps/BiomeAssignStep.cs` |
| `TerrainStep` | `Steps/TerrainStep.cs` |
| `WallsStep` | `Steps/WallsStep.cs` |
| `ObjectsStep` | `Steps/ObjectsStep.cs` |
| `PlayerSpawnStep` | `Steps/PlayerSpawnStep.cs` |
| `AnimalsStep` | `Steps/AnimalsStep.cs` |

---

## Порядок создания скриптов (30 минут, с нуля)

1. `BiomeType.cs` — ScriptableObject с `[CreateAssetMenu]`
2. `MapSettings.cs` — `[Serializable]` без логики
3. `VoronoiCell.cs` — три свойства: Site, Polygon, BiomeIndex
4. `GenerationContext.cs` — "рюкзак" со всеми данными
5. `IMapGenerationStep.cs` — интерфейс: Name + Execute(ctx)
6. `ISettingsRepository.cs` — интерфейс: Load() + Save()
7. `VoronoiGenerator.cs` — static class, Generate() + ClipToRect() + ClipByBisector()
8. `CellMeshBuilder.cs` — static class, Build() с Perlin и веером
9. `JsonSettingsRepository.cs` — Load/Save через JsonUtility + File
10. `InitializeStep.cs` — очищает ctx.Root и ctx.Cells
11. `VoronoiStep.cs` — вызывает Generator, сохраняет в ctx.Cells
12. `BiomeAssignStep.cs` — Perlin → biomeIndex
13. `TerrainStep.cs` — CellMeshBuilder + MeshFilter + MeshRenderer + MeshCollider
14. `WallsStep.cs` — 4 куба по краям
15. `ObjectsStep.cs` — Instantiate в случайных точках полигона
16. `PlayerSpawnStep.cs` — ищет isStartBiome, спавнит
17. `MapGenerator.cs` — foreach step → yield StartCoroutine(step.Execute)
18. `GameLauncher.cs` — Awake: создаёт контекст + шаги, вызывает Generate

---

## Биомы — значения

| Биом | Color | Height | isStart | hasWater |
|------|-------|--------|---------|----------|
| Grass | светло-зелёный | 0.5 | ✅ | ❌ |
| Forest | зелёный | 2.0 | ❌ | ❌ |
| Rocks | серый | 4.0 | ❌ | ❌ |
| Water | синий | 0.0 | ❌ | ✅ |
| Desert | жёлтый | 1.0 | ❌ | ❌ |

---

## Стены — позиции и размеры

| Стена | Position | Scale |
|-------|----------|-------|
| North | (size/2, 5, size) | (size, 10, 1) |
| South | (size/2, 5, 0) | (size, 10, 1) |
| East | (size, 5, size/2) | (1, 10, size) |
| West | (0, 5, size/2) | (1, 10, size) |

---

## Атрибуты и животные — числа

### Персонаж
| Атрибут | Убывание | HP при 0 | Скорость при 0 |
|---------|----------|----------|----------------|
| Hunger | -10/ч | -2 HP/ч | ×0.5 |
| Thirst | -15/ч | -3 HP/ч | ×0.5 |

### Агрессивные
| Животное | HP | Armor | Dmg | Range | CD |
|----------|----|-------|-----|-------|----|
| Bear | 100 | 5 | 20 | 3m | 2s |
| Boar | 55 | 3 | 15 | 2.5m | 1.2s |
| Wolf | 40 | 2 | 8 | 2.5m | 1s |

### Оружие
| Оружие | Dmg | Range | Dur |
|--------|-----|-------|-----|
| Club | 12 | 2m | 7 |
| Spear | 18 | 3m | 10 |
| Bow | 15 | 25m | 5 |

---

## Canvas настройки (запомни)

```
Canvas → Screen Space - Overlay
Canvas Scaler → Scale With Screen Size
Reference Resolution: 1920 × 1080
Match Width Or Height: 0.5
```

---

## Частые ошибки и быстрые исправления

| Ошибка | Быстрое исправление |
|--------|---------------------|
| Меню видно поверх игры | `ShowOnly()` в UIManager скрывает все панели кроме одной |
| `sharedMaterial` vs `material` | `sharedMaterial` не создаёт копию — используй для статичных |
| Анимация не работает на паузе | `Time.unscaledDeltaTime` вместо `Time.deltaTime` |
| `JsonUtility` игнорирует поле | Добавь `[Serializable]` к классу и `public` к полям |
| `StreamingAssets` не работает в билде | Используй `Application.streamingAssetsPath` (не хардкодить путь) |
| `FindObjectOfType` устарел | `FindFirstObjectByType<T>()` |
| Объект провалился сквозь пол | Проверь Collider на объекте и на полу |
| Корутина не запускается | Объект должен быть `SetActive(true)` и быть `MonoBehaviour` |
| Список не сериализуется | Обернуть в класс с полем `items`: `{"items": [...]}` |
| `Time.timeScale = 0` → Update стоит | `Update` работает, `FixedUpdate` — нет. Проверь `unscaledDeltaTime` |

---

## JSON файлы — путь в коде

```csharp
// В любом классе:
string path = Path.Combine(Application.streamingAssetsPath, "mapSettings.json");

// Читать:
string json = File.ReadAllText(path);
var obj = JsonUtility.FromJson<MyClass>(json);

// Писать:
File.WriteAllText(path, JsonUtility.ToJson(obj, true));

// Десериализовать массив (трюк):
string wrapped = "{\"items\":" + rawArrayJson + "}";
var wrapper = JsonUtility.FromJson<Wrapper>(wrapped);
```

---

## Управление — горячие клавиши (все)

| Клавиша | Действие |
|---------|----------|
| WASD | Движение |
| Shift | Бег |
| Q/E | Вращение камеры |
| Mouse Wheel | Зум камеры |
| ЛКМ | Атака / взаимодействие |
| ПКМ | Контекстное меню / атака |
| C | Открыть крафт |
| Space | Автоподбор |
| T | Выделить врага |
| R | Вращать постройку |
| Pause | Пауза |
| F11 | Экран проигрыша (тест) |
| Escape | Закрыть меню |

---

## Структура иерархии сцены

```
SampleScene
├── GameManager               ← MapGenerator + GameLauncher
├── Main Camera               ← Position(50,80,50) Rotation(60,0,0)
├── Directional Light         ← Rotation(50,-30,0)
├── Canvas                    ← все UI панели
│   ├── MainMenuPanel
│   ├── MapSettingsPanel
│   ├── GameHUD
│   ├── CraftPanel
│   ├── PausePanel
│   ├── GameOverPanel
│   └── SettingsPanel
└── EventSystem               ← создаётся автоматически с Canvas
```

При запуске генерации создаётся:
```
GameManager (как Root)
├── Terrain/
│   ├── Cell_0
│   ├── Cell_1
│   └── ...
├── Walls/
│   ├── Wall_North
│   ├── Wall_South
│   ├── Wall_East
│   └── Wall_West
├── Objects/
│   └── ... префабы
├── Animals/
│   └── ... животные
└── Player
```

---

## Минута перед сдачей

```
1. File → Save (Ctrl+S)
2. Window → Console → проверить ошибки (красные)
3. Ctrl+P → проверить Play Mode
4. Ctrl+Shift+P → остановить Play Mode
5. Assets → Export Package → сохранить
6. File → Build Settings → Build
7. Запустить .exe — убедиться что работает
8. Готово!
```
