# Модуль А — Структура проекта и импорт (6 баллов, 30 минут)

Этот модуль — фундамент. Делай его первым. Потом всё остальное строится поверх.

---

## Шаг 1: Создание Unity проекта

1. Открой **Unity Hub**
2. Нажми **New project** (синяя кнопка)
3. В списке шаблонов выбери **3D (URP) Core** (не просто 3D!)
4. Название проекта: `VoronoiSurvival` (без пробелов, без русских букв)
5. Путь: любая удобная папка
6. Нажми **Create project**

> Почему URP? Задание требует Universal Render Pipeline. Шейдеры и материалы URP отличаются от встроенного рендера.

После создания Unity откроет пустую сцену `SampleScene`.

---

## Шаг 2: Настройка Package Manager

Зайди в **Window → Package Manager**.

Убедись, что установлены:
- **Universal RP** (Universal Render Pipeline) — должен быть, раз выбрал URP шаблон
- **Input System** — для нового ввода (если нужен)

Если Universal RP не установлен:
1. В Package Manager нажми **+** → **Add package by name**
2. Введи: `com.unity.render-pipelines.universal`
3. Нажми **Add**

---

## Шаг 3: Создание структуры папок

В окне **Project** (снизу слева), правой кнопкой нажми на папку **Assets**:

**Создай точно такую структуру:**

```
Assets/
├── Scripts/
│   └── VoronoiMapGenerator/
│       ├── Data/
│       ├── Interfaces/
│       ├── Algorithms/
│       ├── Steps/
│       ├── Persistence/
│       └── UI/
├── Materials/
├── Prefabs/
├── ScriptableObjects/
│   └── Biomes/
├── Scenes/
└── StreamingAssets/
```

**Как создавать папки:**
- ПКМ на `Assets` → `Create` → `Folder`
- Назвать папку
- Повторить

> ВАЖНО: Никаких русских букв в именах папок! Это критерий оценки.

---

## Шаг 4: Настройка StreamingAssets

В папке `StreamingAssets` создай пустые JSON файлы.

Сначала создай папку `StreamingAssets` в `Assets/`.

Потом создай текстовые файлы (Create → TextAsset или прямо в проводнике):
- `mapSettings.json` — содержимое: `{}`
- `settings.json` — содержимое: `{}`
- `locale_ru.json` — содержимое: `{}`
- `locale_en.json` — содержимое: `{}`
- `craft.json` — содержимое: `[]`
- `save.json` — содержимое: `{}`

> StreamingAssets — специальная папка Unity. Файлы из неё копируются в билд и доступны через `Application.streamingAssetsPath`. Это единственный способ читать/писать файлы в билде.

---

## Шаг 5: Создание BiomeType ScriptableObject

ScriptableObject — это файл данных в Unity. Создаём один раз, потом просто заполняем в Inspector.

**Создай файл:** `Assets/Scripts/VoronoiMapGenerator/Data/BiomeType.cs`

```csharp
// Assets/Scripts/VoronoiMapGenerator/Data/BiomeType.cs
using UnityEngine;

[CreateAssetMenu(menuName = "Voronoi/Biome Type")]
public class BiomeType : ScriptableObject
{
    public string     biomeName;
    public Color      groundColor     = Color.green;
    public float      heightVariation = 1f;
    public bool       isStartBiome    = false;
    public bool       hasWater        = false;
    public GameObject[] objectPrefabs;
}
```

**Что делает `[CreateAssetMenu]`:**
Добавляет пункт в меню ПКМ → Create → Voronoi → Biome Type.
Без этого атрибута нельзя создать файл ScriptableObject через интерфейс Unity.

**После сохранения скрипта** подожди пока Unity скомпилирует (внизу статус-бар).

---

## Шаг 6: Создание 5 биомов

Теперь создай 5 ScriptableObject файлов биомов.

**Путь для создания:** ПКМ на папку `Assets/ScriptableObjects/Biomes/` → `Create` → `Voronoi` → `Biome Type`

Создай и настрой каждый:

### Biome_Grass
- `biomeName`: `Grass`
- `groundColor`: светло-зелёный (R:0.5, G:0.8, B:0.3, A:1)
- `heightVariation`: `0.5`
- `isStartBiome`: ✅ (галочка — ОБЯЗАТЕЛЬНО только у одного!)
- `hasWater`: ❌
- `objectPrefabs`: пусто (заполнишь позже)

### Biome_Forest
- `biomeName`: `Forest`
- `groundColor`: зелёный (R:0.2, G:0.5, B:0.1, A:1)
- `heightVariation`: `2.0`
- `isStartBiome`: ❌
- `hasWater`: ❌

### Biome_Rocks
- `biomeName`: `Rocks`
- `groundColor`: серый (R:0.5, G:0.5, B:0.5, A:1)
- `heightVariation`: `4.0`
- `isStartBiome`: ❌
- `hasWater`: ❌

### Biome_Water
- `biomeName`: `Water`
- `groundColor`: синий (R:0.2, G:0.4, B:0.8, A:1)
- `heightVariation`: `0.0`
- `isStartBiome`: ❌
- `hasWater`: ✅

### Biome_Desert
- `biomeName`: `Desert`
- `groundColor`: жёлтый (R:0.9, G:0.8, B:0.3, A:1)
- `heightVariation`: `1.0`
- `isStartBiome`: ❌
- `hasWater`: ❌

---

## Шаг 7: Настройка сцены

### Иерархия объектов

В окне **Hierarchy** удали всё что есть (Main Camera и Directional Light можно оставить).

Создай структуру:
```
SampleScene
├── GameManager          ← пустой объект
├── Main Camera          ← уже есть
└── Directional Light   ← уже есть
```

**Создать пустой объект:**
- ПКМ в Hierarchy → `Create Empty`
- Назвать `GameManager`

### Настройка Main Camera

Выбери Main Camera в Hierarchy. В Inspector:

**Transform:**
- Position: X=`50`, Y=`80`, Z=`50`
- Rotation: X=`60`, Y=`0`, Z=`0`

Это изометрический вид сверху под углом 60°.

### Настройка Directional Light

Выбери Directional Light. В Inspector:

**Transform → Rotation:**
- X=`50`, Y=`-30`, Z=`0`

**Light компонент:**
- Color: белый или чуть тёплый (R:1, G:0.95, B:0.9)
- Intensity: `1.2`

---

## Шаг 8: Добавить MapGenerator и GameLauncher на GameManager

После написания всех скриптов (Модуль Г):
1. Выбери `GameManager` в Hierarchy
2. В Inspector нажми **Add Component**
3. Найди `MapGenerator` → добавь
4. Найди `GameLauncher` → добавь (автоматически добавит MapGenerator если его нет)

В **GameLauncher** (Inspector):
- `Biomes` → Size: `5`, перетащи все 5 ScriptableObject биомов
- `Player Prefab` → перетащи префаб игрока (или оставь пустым для тест-капсулы)
- `Auto Generate On Start` → ✅

---

## Шаг 9: Создание материала для террейна

1. ПКМ в `Assets/Materials/` → `Create` → `Material`
2. Назови `TerrainMaterial`
3. В Inspector измени Shader на **Universal Render Pipeline/Lit** (или Particles/Unlit для простоты)
4. Режим: `Opaque`

---

## Шаг 10: Export Package (Модуль А)

**После завершения Модуля А:**

1. В меню Unity: `Assets` → `Export Package...`
2. Убедись что все галочки стоят
3. Нажми `Export...`
4. Сохрани как `ModuleA.unitypackage`

---

## Шаг 11: Сборка билда (Модуль А)

1. `File` → `Build Settings`
2. Платформа: `Windows, Mac, Linux`
3. Нажми `Add Open Scenes` (добавит SampleScene)
4. Architecture: `x86_64`
5. Нажми `Build`
6. Выбери папку, например `Builds/ModuleA/`
7. Подожди завершения

**Player Settings** (нажми кнопку Player Settings в Build Settings):
- Company Name: твоё имя / школа
- Product Name: `VoronoiSurvival`
- Default Screen Width: `1920`
- Default Screen Height: `1080`
- Fullscreen Mode: `Fullscreen Window`

---

## Чеклист Модуля А

```
□ Создан проект Unity 6 с URP шаблоном
□ Созданы все папки в Assets/Scripts/VoronoiMapGenerator/
□ Создана папка StreamingAssets с пустыми JSON файлами
□ Написан BiomeType.cs с [CreateAssetMenu]
□ Созданы 5 ScriptableObject биомов с правильными настройками
□ Один биом имеет isStartBiome = true (Grass)
□ Один биом имеет hasWater = true (Water)
□ Настроена сцена: GameManager, Camera (50,80,50 rot 60,0,0), Light
□ Сделан Export Package → ModuleA.unitypackage
□ Сделан Build → Builds/ModuleA/
```

---

## Типичные ошибки Модуля А

**Ошибка:** Меню "Voronoi/Biome Type" не появляется  
**Решение:** Подожди компиляцию Unity (статус-бар внизу). Если ошибок нет — перезапусти Unity.

**Ошибка:** Файл .cs создан но Unity его не видит  
**Решение:** Убедись что файл в папке Assets/ (не снаружи проекта). Unity компилирует только файлы внутри Assets/.

**Ошибка:** После создания папки видно "Empty Folder"  
**Решение:** Это нормально. Unity не показывает пустые папки в некоторых режимах. Добавь хоть один файл.
