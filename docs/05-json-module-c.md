# Модуль В — Хранение данных (17 баллов, 4 часа)

Все данные — в JSON файлах в StreamingAssets. Читаются и редактируются в билде.

---

## Список JSON файлов

| Файл | Что хранит | Класс |
|------|-----------|-------|
| `mapSettings.json` | Настройки генерации + атрибуты игрока | `MapSettings` |
| `settings.json` | Звук, разрешение, язык, мышь | `GameSettings` |
| `save.json` | Состояние игры (позиция, атрибуты, инвентарь, день) | `SaveData` |
| `craft.json` | Рецепты крафта | `CraftRecipe[]` |
| `locale_ru.json` | Русский интерфейс (ключ → текст) | `Dictionary<string,string>` |
| `locale_en.json` | Английский интерфейс (ключ → текст) | `Dictionary<string,string>` |

**Важно:** Все файлы в `Assets/StreamingAssets/`. В билде они в `<BuildFolder>/<GameName>_Data/StreamingAssets/`.

---

## mapSettings.json

Автоматически создаётся `JsonSettingsRepository` при первом сохранении.

Пример содержимого:
```json
{
    "pointCount": 50,
    "mapSize": 100,
    "biomeZones": 5,
    "objectPlacementAttempts": 10,
    "visualizeSteps": false,
    "maxHealth": 100.0,
    "maxHunger": 100.0,
    "maxThirst": 100.0,
    "hpLossOnHunger": 2.0,
    "hpLossOnThirst": 3.0
}
```

Соответствует классу `MapSettings` (уже написан в Data/MapSettings.cs).

---

## settings.json

```json
{
    "soundVolume": 1.0,
    "musicVolume": 0.8,
    "resolutionIndex": 0,
    "language": "ru",
    "mouseSensitivity": 1.0,
    "keyMove": "WASD",
    "keyRun": "LeftShift",
    "keyCraft": "C",
    "keyPause": "Pause"
}
```

---

## save.json

```json
{
    "playerX": 50.0,
    "playerY": 2.0,
    "playerZ": 50.0,
    "currentHealth": 87.5,
    "currentHunger": 65.0,
    "currentThirst": 42.0,
    "currentDay": 3,
    "currentHour": 14.5,
    "inventory": [
        {
            "itemId": "Wood",
            "quantity": 15
        },
        {
            "itemId": "Stone",
            "quantity": 8
        }
    ],
    "mapSeed": 12345,
    "timestamp": "2026-04-16T10:30:00"
}
```

---

## craft.json

```json
[
    {
        "id": "Club",
        "displayName": "Дубина",
        "category": "Weapons",
        "description": "Простое оружие из ветки. 12 урона, 7 ударов.",
        "ingredients": [
            { "itemId": "Wood", "amount": 3 },
            { "itemId": "Fiber", "amount": 2 }
        ],
        "resultItemId": "Club",
        "resultAmount": 1
    },
    {
        "id": "FishingRod",
        "displayName": "Удочка",
        "category": "Tools",
        "description": "Для рыбалки. +20% шанс рыбы.",
        "ingredients": [
            { "itemId": "Wood", "amount": 2 },
            { "itemId": "Fiber", "amount": 4 }
        ],
        "resultItemId": "FishingRod",
        "resultAmount": 1
    },
    {
        "id": "LeatherJacket",
        "displayName": "Кожаная куртка",
        "category": "Clothing",
        "description": "Armor 5, снижает урон на 25%.",
        "ingredients": [
            { "itemId": "Hide", "amount": 4 },
            { "itemId": "Fiber", "amount": 2 }
        ],
        "resultItemId": "LeatherJacket",
        "resultAmount": 1
    }
]
```

---

## locale_ru.json

```json
{
    "menu.play": "Играть",
    "menu.load": "Загрузить",
    "menu.settings": "Настройки",
    "menu.exit": "Выход",
    "settings.sound": "Громкость звуков",
    "settings.music": "Громкость музыки",
    "settings.resolution": "Разрешение",
    "settings.language": "Язык",
    "settings.sensitivity": "Чувствительность мыши",
    "settings.back": "Назад",
    "mapsettings.title": "Настройки карты",
    "mapsettings.points": "Количество точек",
    "mapsettings.mapsize": "Размер карты",
    "mapsettings.biomes": "Количество биомов",
    "mapsettings.objects": "Попыток объектов",
    "mapsettings.visualize": "Визуализация шагов",
    "mapsettings.health": "Макс. здоровье",
    "mapsettings.hunger": "Макс. сытость",
    "mapsettings.thirst": "Макс. жажда",
    "mapsettings.generate": "Генерировать!",
    "hud.hp": "HP",
    "hud.hunger": "Сытость",
    "hud.thirst": "Жажда",
    "pause.title": "ПАУЗА",
    "pause.resume": "Продолжить",
    "pause.save": "Сохранить",
    "pause.load": "Загрузить",
    "pause.newgame": "Новая игра",
    "pause.audio": "Вкл/Выкл аудио",
    "gameover.title": "ВЫ ПОГИБЛИ",
    "gameover.load": "Загрузить последнее сохранение",
    "gameover.newgame": "Новая игра",
    "craft.title": "Крафт",
    "craft.tools": "Инструменты",
    "craft.weapons": "Оружие",
    "craft.food": "Еда",
    "craft.buildings": "Сооружения",
    "craft.other": "Прочее",
    "craft.clothing": "Одежда",
    "action.working": "Работаем...",
    "action.cancel": "Отмена"
}
```

---

## locale_en.json

```json
{
    "menu.play": "Play",
    "menu.load": "Load",
    "menu.settings": "Settings",
    "menu.exit": "Exit",
    "settings.sound": "Sound Volume",
    "settings.music": "Music Volume",
    "settings.resolution": "Resolution",
    "settings.language": "Language",
    "settings.sensitivity": "Mouse Sensitivity",
    "settings.back": "Back",
    "mapsettings.title": "Map Settings",
    "mapsettings.points": "Point Count",
    "mapsettings.mapsize": "Map Size",
    "mapsettings.biomes": "Biome Zones",
    "mapsettings.objects": "Object Attempts",
    "mapsettings.visualize": "Visualize Steps",
    "mapsettings.health": "Max Health",
    "mapsettings.hunger": "Max Hunger",
    "mapsettings.thirst": "Max Thirst",
    "mapsettings.generate": "Generate!",
    "hud.hp": "HP",
    "hud.hunger": "Hunger",
    "hud.thirst": "Thirst",
    "pause.title": "PAUSED",
    "pause.resume": "Resume",
    "pause.save": "Save",
    "pause.load": "Load",
    "pause.newgame": "New Game",
    "pause.audio": "Toggle Audio",
    "gameover.title": "YOU DIED",
    "gameover.load": "Load Last Save",
    "gameover.newgame": "New Game",
    "craft.title": "Craft",
    "craft.tools": "Tools",
    "craft.weapons": "Weapons",
    "craft.food": "Food",
    "craft.buildings": "Buildings",
    "craft.other": "Other",
    "craft.clothing": "Clothing",
    "action.working": "Working...",
    "action.cancel": "Cancel"
}
```

---

## Код: GameSettings.cs (Data)

```csharp
// Assets/Scripts/VoronoiMapGenerator/Data/GameSettings.cs
using System;
using UnityEngine;

[Serializable]
public class GameSettings
{
    public float  soundVolume       = 1f;
    public float  musicVolume       = 0.8f;
    public int    resolutionIndex   = 0;
    public string language          = "ru";
    public float  mouseSensitivity  = 1f;
}
```

---

## Код: SaveData.cs (Data)

```csharp
// Assets/Scripts/VoronoiMapGenerator/Data/SaveData.cs
using System;
using System.Collections.Generic;
using UnityEngine;

[Serializable]
public class SaveData
{
    public float  playerX       = 50f;
    public float  playerY       = 2f;
    public float  playerZ       = 50f;
    public float  currentHealth = 100f;
    public float  currentHunger = 100f;
    public float  currentThirst = 100f;
    public int    currentDay    = 1;
    public float  currentHour   = 6f;
    public int    mapSeed       = 0;
    public string timestamp     = "";
    public List<InventorySlotData> inventory = new List<InventorySlotData>();
}

[Serializable]
public class InventorySlotData
{
    public string itemId;
    public int    quantity;
}
```

---

## Код: CraftRecipe.cs (Data)

```csharp
// Assets/Scripts/VoronoiMapGenerator/Data/CraftRecipe.cs
using System;
using System.Collections.Generic;

[Serializable]
public class CraftRecipe
{
    public string              id;
    public string              displayName;
    public string              category;
    public string              description;
    public List<Ingredient>    ingredients = new List<Ingredient>();
    public string              resultItemId;
    public int                 resultAmount = 1;
}

[Serializable]
public class Ingredient
{
    public string itemId;
    public int    amount;
}

// Обёртка для JsonUtility (не может десериализовать List напрямую)
[Serializable]
public class CraftRecipeList
{
    public List<CraftRecipe> items = new List<CraftRecipe>();
}
```

---

## Код: SettingsRepository.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/Persistence/SettingsRepository.cs
using System.IO;
using UnityEngine;

public class SettingsRepository
{
    private string FilePath =>
        Path.Combine(Application.streamingAssetsPath, "settings.json");

    public GameSettings Load()
    {
        if (File.Exists(FilePath))
        {
            try
            {
                return JsonUtility.FromJson<GameSettings>(File.ReadAllText(FilePath));
            }
            catch (System.Exception e)
            {
                Debug.LogWarning($"[SettingsRepository] Ошибка: {e.Message}");
            }
        }
        return new GameSettings();
    }

    public void Save(GameSettings settings)
    {
        File.WriteAllText(FilePath, JsonUtility.ToJson(settings, true));
    }
}
```

---

## Код: SaveSystem.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/Persistence/SaveSystem.cs
using System.IO;
using UnityEngine;

public class SaveSystem : MonoBehaviour
{
    public static SaveSystem Instance { get; private set; }

    private string FilePath =>
        Path.Combine(Application.streamingAssetsPath, "save.json");

    // Автосохранение каждые 30 секунд
    private float _autoSaveTimer = 0f;
    private const float AutoSaveInterval = 30f;

    [SerializeField] private PlayerStats    playerStats;
    [SerializeField] private DayNightCycle  dayNightCycle;

    private void Awake()
    {
        if (Instance != null) { Destroy(gameObject); return; }
        Instance = this;
    }

    private void Update()
    {
        _autoSaveTimer += Time.deltaTime;
        if (_autoSaveTimer >= AutoSaveInterval)
        {
            _autoSaveTimer = 0f;
            SaveGame();
        }
    }

    public void SaveGame()
    {
        if (playerStats == null) return;

        var player = GameObject.FindGameObjectWithTag("Player");
        Vector3 pos = player != null ? player.transform.position : Vector3.zero;

        var data = new SaveData
        {
            playerX       = pos.x,
            playerY       = pos.y,
            playerZ       = pos.z,
            currentHealth = playerStats.CurrentHealth,
            currentHunger = playerStats.CurrentHunger,
            currentThirst = playerStats.CurrentThirst,
            currentDay    = dayNightCycle != null ? dayNightCycle.CurrentDay  : 1,
            currentHour   = dayNightCycle != null ? dayNightCycle.CurrentHour : 6f,
            timestamp     = System.DateTime.Now.ToString("o"),
        };

        File.WriteAllText(FilePath, JsonUtility.ToJson(data, true));
        Debug.Log($"[SaveSystem] Сохранено: {FilePath}");
    }

    public void LoadGame()
    {
        if (!File.Exists(FilePath))
        {
            Debug.LogWarning("[SaveSystem] Файл сохранения не найден");
            return;
        }

        try
        {
            var data   = JsonUtility.FromJson<SaveData>(File.ReadAllText(FilePath));
            var player = GameObject.FindGameObjectWithTag("Player");

            if (player != null)
                player.transform.position = new Vector3(data.playerX, data.playerY, data.playerZ);

            if (playerStats != null)
            {
                // Применяем сохранённые значения через рефлексию не получится —
                // делаем публичные методы в PlayerStats
                playerStats.LoadFromSave(data.currentHealth, data.currentHunger, data.currentThirst);
            }

            Debug.Log($"[SaveSystem] Загружено. День {data.currentDay}, час {data.currentHour:F1}");
        }
        catch (System.Exception e)
        {
            Debug.LogError($"[SaveSystem] Ошибка загрузки: {e.Message}");
        }
    }
}
```

Добавь в `PlayerStats.cs` метод:
```csharp
public void LoadFromSave(float health, float hunger, float thirst)
{
    CurrentHealth = Mathf.Clamp(health, 0f, MaxHealth);
    CurrentHunger = Mathf.Clamp(hunger, 0f, MaxHunger);
    CurrentThirst = Mathf.Clamp(thirst, 0f, MaxThirst);
}
```

---

## Код: LocalizationManager.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/Persistence/LocalizationManager.cs
using System.Collections.Generic;
using System.IO;
using UnityEngine;

/// <summary>
/// JsonUtility не умеет Dictionary — используем список пар ключ/значение.
/// </summary>
[System.Serializable]
public class LocalizationEntry
{
    public string key;
    public string value;
}

[System.Serializable]
public class LocalizationData
{
    public List<LocalizationEntry> entries = new List<LocalizationEntry>();
}

public class LocalizationManager : MonoBehaviour
{
    public static LocalizationManager Instance { get; private set; }

    private Dictionary<string, string> _strings = new Dictionary<string, string>();
    private string _currentLanguage = "ru";

    private void Awake()
    {
        if (Instance != null) { Destroy(gameObject); return; }
        Instance = this;
        DontDestroyOnLoad(gameObject);
        LoadLanguage(_currentLanguage);
    }

    public void LoadLanguage(string lang)
    {
        _currentLanguage = lang;
        string path = Path.Combine(Application.streamingAssetsPath, $"locale_{lang}.json");

        if (!File.Exists(path))
        {
            Debug.LogWarning($"[LocalizationManager] Файл не найден: {path}");
            return;
        }

        try
        {
            // Читаем JSON как обёртку с массивом
            var data = JsonUtility.FromJson<LocalizationData>(File.ReadAllText(path));
            _strings.Clear();
            foreach (var entry in data.entries)
                _strings[entry.key] = entry.value;

            Debug.Log($"[LocalizationManager] Загружено {_strings.Count} строк ({lang})");
        }
        catch (System.Exception e)
        {
            Debug.LogError($"[LocalizationManager] Ошибка: {e.Message}");
        }
    }

    public string Get(string key)
    {
        return _strings.TryGetValue(key, out string val) ? val : $"[{key}]";
    }
}
```

**Важно:** locale_ru.json/locale_en.json нужно хранить в другом формате для JsonUtility:

```json
{
    "entries": [
        { "key": "menu.play",  "value": "Играть" },
        { "key": "menu.load",  "value": "Загрузить" },
        { "key": "menu.exit",  "value": "Выход" }
    ]
}
```

---

## Код: CraftDatabase.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/Persistence/CraftDatabase.cs
using System.Collections.Generic;
using System.IO;
using UnityEngine;

public class CraftDatabase : MonoBehaviour
{
    public static CraftDatabase Instance { get; private set; }

    private List<CraftRecipe> _recipes = new List<CraftRecipe>();

    private string FilePath =>
        Path.Combine(Application.streamingAssetsPath, "craft.json");

    private void Awake()
    {
        if (Instance != null) { Destroy(gameObject); return; }
        Instance = this;
        LoadRecipes();
    }

    private void LoadRecipes()
    {
        if (!File.Exists(FilePath))
        {
            Debug.LogWarning("[CraftDatabase] craft.json не найден");
            return;
        }

        try
        {
            // JsonUtility не умеет List<T> как корневой объект —
            // оборачиваем в CraftRecipeList
            string raw = File.ReadAllText(FilePath);

            // Пробуем обёртку
            var list = JsonUtility.FromJson<CraftRecipeList>("{\"items\":" + raw + "}");
            _recipes  = list.items;
            Debug.Log($"[CraftDatabase] Загружено {_recipes.Count} рецептов");
        }
        catch (System.Exception e)
        {
            Debug.LogError($"[CraftDatabase] Ошибка: {e.Message}");
        }
    }

    public List<CraftRecipe> GetByCategory(string category)
    {
        return _recipes.FindAll(r => r.category == category);
    }

    public CraftRecipe GetById(string id)
    {
        return _recipes.Find(r => r.id == id);
    }

    public List<CraftRecipe> All => _recipes;
}
```

---

## Код: AudioManager.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/UI/AudioManager.cs
using UnityEngine;

public class AudioManager : MonoBehaviour
{
    public static AudioManager Instance { get; private set; }

    [Header("Источники звука")]
    [SerializeField] private AudioSource musicSource;
    [SerializeField] private AudioSource sfxSource;

    [Header("Клипы")]
    [SerializeField] private AudioClip gameOverClip;
    [SerializeField] private AudioClip buildClip;

    private bool _isMuted = false;

    private void Awake()
    {
        if (Instance != null) { Destroy(gameObject); return; }
        Instance = this;
        DontDestroyOnLoad(gameObject);
    }

    public void SetSoundVolume(float vol)
    {
        sfxSource.volume = vol;
    }

    public void SetMusicVolume(float vol)
    {
        musicSource.volume = vol;
    }

    public void Toggle()
    {
        _isMuted = !_isMuted;
        AudioListener.pause = _isMuted;
    }

    public void PlayGameOver()
    {
        if (gameOverClip != null)
            sfxSource.PlayOneShot(gameOverClip);
    }

    public void PlayBuild()
    {
        if (buildClip != null)
            sfxSource.PlayOneShot(buildClip);
    }

    public void PlaySFX(AudioClip clip)
    {
        if (clip != null)
            sfxSource.PlayOneShot(clip);
    }
}
```

---

## Как тестировать JSON в реальном времени

1. Запусти игру в Play Mode (Ctrl+P)
2. Открой файл в Проводнике: `Assets/StreamingAssets/mapSettings.json`
3. Измени значения (например `"pointCount": 100`)
4. Сохрани файл
5. В Unity нажми кнопку "Regenerate" в GameLauncher (если такая есть) или Restart

Для проверки сохранения:
1. Запусти игру
2. Поиграй несколько секунд
3. Нажми Save (или подожди 30 секунд автосохранения)
4. Открой `save.json` в редакторе — увидишь актуальные данные

---

## Проблемы с JsonUtility

**JsonUtility НЕ поддерживает:**
- `Dictionary<K, V>` — используй список пар
- `List<T>` как корневой объект — оборачивай в класс
- Наследование — только `[Serializable]` классы
- `null` значения — используй пустые строки / -1

**Как десериализовать массив (craft.json):**
```csharp
// Трюк: оборачиваем в объект с полем items
string wrapped = "{\"items\":" + rawJsonArray + "}";
var result = JsonUtility.FromJson<Wrapper>(wrapped);
```

**Все поля в `[Serializable]` классе должны быть:**
- `public` ИЛИ
- `private` с атрибутом `[SerializeField]`
