# 📄 Листок День 1 — Модули А + Б (28 баллов)

> ⏱️ Читать 5 минут → переписать формулы на листок → работать

---

## 🗂️ Модуль А — Структура проекта (6 баллов, 30 мин)

**Папки в `Assets/Scripts/VoronoiMapGenerator/`:**

```
Data/          ← MapSettings.cs, VoronoiCell.cs, BiomeType.cs
Interfaces/    ← IMapGenerationStep.cs, ISettingsRepository.cs
Algorithms/    ← VoronoiGenerator.cs, CellMeshBuilder.cs
Steps/         ← 7 файлов шагов
Persistence/   ← JsonSettingsRepository.cs
UI/            ← UIManager.cs, GameHUD.cs, MapSettingsPanel.cs...
GenerationContext.cs
MapGenerator.cs
GameLauncher.cs
```

**Настройка сцены:**

```
GameManager (пустой объект) → компоненты: MapGenerator + GameLauncher
Main Camera → Position: (50, 80, 50) | Rotation: (60, 0, 0)
```

**BiomeType ScriptableObject** (меню для создания через ПКМ):

```csharp
[CreateAssetMenu(menuName = "Voronoi/Biome Type")]
public class BiomeType : ScriptableObject { ... }
```

**5 биомов (создать в Project):**

| Название | Цвет | Height | isStart | hasWater |
|----------|------|--------|---------|---------|
| Biome_Grass | светло-зелёный | 0.5 | ✅ | ❌ |
| Biome_Forest | зелёный | 2.0 | ❌ | ❌ |
| Biome_Rocks | серый | 4.0 | ❌ | ❌ |
| Biome_Water | синий | 0.0 | ❌ | ✅ |
| Biome_Desert | жёлтый | 1.0 | ❌ | ❌ |

---

## 🖥️ Модуль Б — UI (22 балла, 2 ч)

**7 панелей в одном Canvas (Screen Space Overlay):**

Canvas → Reference Resolution: **1920×1080** | Scale: **Scale With Screen Size** | Match: **0.5**

```
Canvas
├── MainMenuPanel      ← активна при старте
├── MapSettingsPanel   ← по кнопке "Играть"
├── GameHUD            ← во время игры
├── CraftPanel         ← по C / кнопке наковальни
├── PausePanel         ← по Escape
├── GameOverPanel      ← при HP=0 или F11
└── SettingsPanel      ← из главного меню
```

> ⚠️ **ВАЖНО:** все панели `SetActive(false)` при старте, кроме `MainMenuPanel`!

**На что вешать скрипты:**

| Скрипт | Объект | Что заполнить в инспекторе |
|--------|--------|--------------------------|
| `UIManager.cs` | пустой объект UIManager | все 7 панелей + Text кнопки звука |
| `MapSettingsPanel.cs` | MapSettingsPanel | GameLauncher + 7 InputField + Toggle + Button |
| `GameHUD.cs` | GameHUD | 3 Slider + Text дня |
| `PlayerStats.cs` | Player | ничего |
| `DayNightCycle.cs` | Directional Light | Light + Gradient |

**Кнопки → методы UIManager:**

| Кнопка | Метод |
|--------|-------|
| Играть (главное меню) | `ShowMapSettings()` |
| Настройки | `ShowSettings()` |
| Загрузить | `OnLoad()` |
| Выход | `QuitGame()` |
| Генерировать (настройки карты) | вызывается из MapSettingsPanel |
| Пауза (HUD) | `ShowPause()` |
| Наковальня (HUD) | `ToggleCraft()` |
| Продолжить (пауза) | `HidePause()` |
| Сохранить | `OnSave()` |
| Новая игра | `OnNewGame()` |
| Звук | `ToggleAudio()` |
| Назад (настройки) | `HideSettings()` |
| Загрузить сохранение (game over) | `OnLoad()` |

**GameHUD — компоненты:**

- Слайдер HP: Fill Color = 🔴 красный, Min=0, Max=100
- Слайдер Hunger: Fill Color = 🟠 оранжевый
- Слайдер Thirst: Fill Color = 🔵 синий
- Текст дня: верхний левый угол `"День 1 · 08:00"`
- Кнопка паузы: верхний правый угол

**CraftPanel — GridLayoutGroup:**

- Cell Size: `100×100`
- Spacing: `10×10`
- Constraint: **Fixed Column Count = 4**
- Обернуть в **ScrollView**

**⌨️ Клавиши (в UIManager.Update):**

- `Escape` → пауза toggle
- `C` → крафт toggle
- `F11` → экран проигрыша

---

## 🔊 Готовые аудио файлы в проекте

**Путь:** `Assets/Resources/Audio/SFX/`

Загружать в коде через:
```csharp
AudioClip clip = Resources.Load<AudioClip>("Audio/SFX/sfx_build");
```

| Файл | Полное имя | Когда использовать |
|------|-----------|-------------------|
| `sfx_build` | sfx_build | Размещение постройки (обязателен по заданию) |
| `sfx_craft` | sfx_craft | Успешный крафт предмета |
| `sfx_hit_1` | sfx_hit_1 | Удар по врагу (вариант 1) |
| `sfx_hit_2` | sfx_hit_2 | Удар по врагу (вариант 2) |
| `sfx_hit_3` | sfx_hit_3 | Удар по врагу (вариант 3) |
| `sfx_hit_4` | sfx_hit_4 | Удар по врагу (вариант 4) |
| `sfx_pop` | sfx_pop | Подбор предмета / всплывающее уведомление |
| `sfx_ui` | sfx_ui | Клик по кнопке UI |
| `sfx_woosh` | sfx_woosh | Свист — бросок копья / выстрел из лука |

> ⚠️ **Звук проигрыша и звук строительства — обязательны по критериям оценки!**
> Используй `sfx_build` для строительства. Для проигрыша подойдёт любой резкий звук.

**Пример подключения звука в UIManager:**
```csharp
[SerializeField] private AudioClip gameOverClip; // перетащить sfx_pop или другой
private AudioSource _audio;

void Awake() {
    _audio = GetComponent<AudioSource>(); // добавить AudioSource на UIManager
}

public void ShowGameOver() {
    gameOverPanel.SetActive(true);
    Time.timeScale = 0f;
    _audio.PlayOneShot(gameOverClip);    // воспроизвести звук проигрыша
}
```
