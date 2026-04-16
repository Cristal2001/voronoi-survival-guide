# Модуль Б — Пользовательский интерфейс (22 балла, 2 часа)

Это самый визуальный модуль. Все 7 панелей через Canvas.

---

## Архитектура UI

```
Canvas (Screen Space - Overlay)
├── MainMenuPanel       ← Главное меню
├── MapSettingsPanel    ← Окно настройки генерации
├── GameHUD             ← HUD во время игры
├── CraftPanel          ← Панель крафта
├── PausePanel          ← Пауза
├── GameOverPanel       ← Экран проигрыша
└── SettingsPanel       ← Настройки игры
```

---

## Шаг 1: Создание Canvas

1. ПКМ в Hierarchy → `UI` → `Canvas`
2. Выбери Canvas, в Inspector:
   - **Canvas компонент:**
     - Render Mode: `Screen Space - Overlay`
   - **Canvas Scaler компонент:**
     - UI Scale Mode: `Scale With Screen Size`
     - Reference Resolution: X=`1920`, Y=`1080`
     - Screen Match Mode: `Match Width Or Height`
     - Match: `0.5`

3. Выбери **EventSystem** (создался автоматически) — оставь как есть.

---

## Шаг 2: Создание всех панелей (пустые)

Создай 7 дочерних Panel под Canvas:

Для каждой панели:
1. ПКМ на Canvas → `UI` → `Panel`
2. Назови как указано
3. В RectTransform: Stretch оба (Alt+Shift+клик в пресете → нижний правый угол = заполнить весь экран)

### Начальная видимость
После создания всех панелей отключи все кроме MainMenuPanel:
- Выбери панель → в Inspector → нажми галочку слева от имени (SetActive = false)

Активная при старте: `MainMenuPanel` ✅  
Остальные: ❌

---

## Шаг 3: MainMenuPanel — Главное меню

### Структура
```
MainMenuPanel (Panel)
├── Background (Image)       ← фоновое изображение
├── TitleText (TextMeshPro)  ← "Воронои Выживание"
├── DecoTree1 (Image)        ← декоративное дерево слева
├── DecoTree2 (Image)        ← декоративное дерево справа
├── DecoCampfire (Image)     ← костёр по центру внизу
├── ButtonsContainer (VerticalLayoutGroup)
│   ├── PlayButton (Button)     ← "Играть"
│   ├── LoadButton (Button)     ← "Загрузить"
│   ├── SettingsButton (Button) ← "Настройки"
│   └── ExitButton (Button)     ← "Выход"
```

### Настройки компонентов

**MainMenuPanel (Image компонент):**
- Color: тёмно-зелёный (R:0.1, G:0.15, B:0.08, A:1) или загрузи изображение фона

**TitleText (TextMeshPro — Text):**
- Position: верхняя часть экрана (Y: 400)
- Текст: `Воронои Выживание`
- Font Size: `72`
- Color: белый или светло-жёлтый

**ButtonsContainer:**
- Anchors: по центру
- Width: `300`, Height: `300`
- VerticalLayoutGroup: spacing = `15`, padding = `10`

**Каждая кнопка (Button):**
- Width: `280`, Height: `60`
- Text: русское название
- Color Normal: коричневый (R:0.4, G:0.25, B:0.1)
- Color Highlighted: светлее
- Color Pressed: тёмнее

### Три декоративных элемента природы
Это обязательное требование — минимум 3 элемента природы на фоне.

1. **DecoTree1** — Image слева, заполни белым прямоугольником (в реальности — спрайт дерева)
2. **DecoTree2** — Image справа
3. **DecoCampfire** — Image внизу по центру

Если нет спрайтов — используй простые цветные прямоугольники. Оценивается наличие, а не красота.

---

## Шаг 4: MapSettingsPanel — Настройки карты

### Структура
```
MapSettingsPanel (Panel)
├── Title (Text) — "Настройки карты"
├── ScrollView
│   └── Content (VerticalLayoutGroup)
│       ├── Row_PointCount
│       │   ├── Label (Text) — "Количество точек"
│       │   └── PointCountInput (InputField)
│       ├── Row_MapSize
│       │   ├── Label (Text) — "Размер карты"
│       │   └── MapSizeInput (InputField)
│       ├── Row_BiomeZones
│       │   ├── Label (Text) — "Количество биомов"
│       │   └── BiomeZonesInput (InputField)
│       ├── Row_ObjAttempts
│       │   ├── Label (Text) — "Попыток объектов"
│       │   └── ObjAttemptsInput (InputField)
│       ├── Row_Visualize
│       │   ├── Label (Text) — "Визуализация шагов"
│       │   └── VisualizeToggle (Toggle)
│       ├── Row_MaxHealth
│       │   ├── Label (Text) — "Макс. здоровье"
│       │   └── MaxHealthInput (InputField)
│       ├── Row_MaxHunger
│       │   ├── Label (Text) — "Макс. сытость"
│       │   └── MaxHungerInput (InputField)
│       ├── Row_MaxThirst
│       │   ├── Label (Text) — "Макс. жажда"
│       │   └── MaxThirstInput (InputField)
│       ├── Row_HpOnHunger
│       │   ├── Label (Text) — "Урон HP при голоде"
│       │   └── HpHungerInput (InputField)
│       └── Row_HpOnThirst
│           ├── Label (Text) — "Урон HP при жажде"
│           └── HpThirstInput (InputField)
├── GenerateButton (Button) — "Генерировать!"
└── BackButton (Button) — "Назад"
```

**Каждая строка Row_*:**
- HorizontalLayoutGroup
- Height: `40`
- Label: Width `250`, TextAlignment Right
- InputField: Width `150`

**InputField настройки:**
- Content Type: `Integer Number` (для pointCount, mapSize) или `Decimal Number`
- Placeholder текст: значение по умолчанию ("50", "100" и т.д.)

---

## Шаг 5: GameHUD — Интерфейс во время игры

### Структура
```
GameHUD (Panel, прозрачный фон)
├── TopLeft
│   └── DayText (Text) — "День 1 | 12:00"
├── TopRight
│   ├── HPBar (Slider)
│   ├── HPLabel (Text) — "❤ HP: 100/100"
│   ├── HungerBar (Slider)
│   ├── HungerLabel (Text) — "🍗 Сытость: 100"
│   ├── ThirstBar (Slider)
│   ├── ThirstLabel (Text) — "💧 Жажда: 100"
│   └── PauseButton (Button) — "⏸"
├── BottomLeft
│   ├── CraftButton (Button, наковальня) — "⚒"
│   ├── InventorySlot1 (Button, рамка)
│   └── InventorySlot2 (Button, рамка)
└── DayNightBar (Slider) — прогресс дня/ночи, внизу
```

**Слайдеры HP/Hunger/Thirst:**
- Direction: Left To Right
- Min: 0, Max: 100
- Value: 100
- Handle Rect: убери (нам не нужна ручка, только заполнение)
- Fill: зелёный для HP, оранжевый для Hunger, синий для Thirst

**DayNightBar:**
- Anchors: Bottom Center
- Width: `600`, Height: `15`
- Fill Color: тёмно-коричневый

---

## Шаг 6: CraftPanel — Панель крафта

### Структура
```
CraftPanel (Panel)
├── Title (Text) — "Крафт"
├── CloseButton (Button) — "✕"
├── TabContainer (HorizontalLayoutGroup)
│   ├── Tab_Tools (Button) — "Инструменты"
│   ├── Tab_Weapons (Button) — "Оружие"
│   ├── Tab_Food (Button) — "Еда"
│   ├── Tab_Buildings (Button) — "Сооружения"
│   ├── Tab_Other (Button) — "Прочее"
│   └── Tab_Clothing (Button) — "Одежда"
└── ScrollView
    └── Content (GridLayoutGroup)
        └── [сюда добавляются кнопки рецептов динамически]
```

**GridLayoutGroup для рецептов:**
- Cell Size: X=`180`, Y=`200`
- Spacing: X=`10`, Y=`10`
- Constraint: Fixed Column Count = `4`

**Каждая кнопка рецепта (CraftItemButton префаб):**
```
CraftItemButton (Button)
├── Icon (Image) — иконка предмета
├── NameText (Text) — название
├── ResourcesText (Text) — "Дерево x3, Камень x1"
└── DurabilityText (Text) — "⚒ 10 ударов"
```

---

## Шаг 7: PausePanel — Экран паузы

### Структура
```
PausePanel (Panel)
├── Background (Image, черный, alpha=0.75)
├── PauseTitle (Text) — "ПАУЗА"
└── ButtonsContainer (VerticalLayoutGroup)
    ├── ResumeButton (Button) — "Продолжить"
    ├── SaveButton (Button) — "Сохранить"
    ├── LoadButton (Button) — "Загрузить"
    ├── NewGameButton (Button) — "Новая игра"
    └── ToggleAudioButton (Button) — "Вкл/Выкл аудио"
```

**Background Image:**
- Color: (0, 0, 0, 0.75) — полупрозрачный чёрный

---

## Шаг 8: GameOverPanel — Экран проигрыша

### Структура
```
GameOverPanel (Panel)
├── Background (Image, красный-чёрный, alpha=0.85)
├── GameOverTitle (Text) — "ВЫ ПОГИБЛИ"
├── SubtitleText (Text) — "Не сдавайся!"
└── ButtonsContainer (VerticalLayoutGroup)
    ├── LoadButton (Button) — "Загрузить последнее сохранение"
    └── NewGameButton (Button) — "Новая игра"
```

---

## Шаг 9: SettingsPanel — Настройки игры

### Структура
```
SettingsPanel (Panel)
├── Title (Text) — "Настройки"
├── ScrollView
│   └── Content (VerticalLayoutGroup)
│       ├── Row_SoundVolume
│       │   ├── Label — "Громкость звуков"
│       │   └── SoundSlider (Slider)
│       ├── Row_MusicVolume
│       │   ├── Label — "Громкость музыки"
│       │   └── MusicSlider (Slider)
│       ├── Row_Resolution
│       │   ├── Label — "Разрешение"
│       │   └── ResolutionDropdown (Dropdown)
│       ├── Row_Language
│       │   ├── Label — "Язык"
│       │   └── LanguageDropdown (Dropdown)
│       └── Row_MouseSensitivity
│           ├── Label — "Чувствительность мыши"
│           └── SensitivitySlider (Slider)
└── BackButton (Button) — "Назад"
```

---

## Код: UIManager.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/UI/UIManager.cs
using UnityEngine;

public class UIManager : MonoBehaviour
{
    // Синглтон
    public static UIManager Instance { get; private set; }

    [Header("Панели")]
    [SerializeField] private GameObject mainMenuPanel;
    [SerializeField] private GameObject mapSettingsPanel;
    [SerializeField] private GameObject gameHUD;
    [SerializeField] private GameObject craftPanel;
    [SerializeField] private GameObject pausePanel;
    [SerializeField] private GameObject gameOverPanel;
    [SerializeField] private GameObject settingsPanel;

    private void Awake()
    {
        if (Instance != null && Instance != this)
        {
            Destroy(gameObject);
            return;
        }
        Instance = this;
    }

    private void Start()
    {
        ShowMainMenu();
    }

    // Показать только одну панель (все остальные скрыть)
    private void ShowOnly(GameObject panel)
    {
        mainMenuPanel.SetActive(false);
        mapSettingsPanel.SetActive(false);
        gameHUD.SetActive(false);
        craftPanel.SetActive(false);
        pausePanel.SetActive(false);
        gameOverPanel.SetActive(false);
        settingsPanel.SetActive(false);

        if (panel != null)
            panel.SetActive(true);
    }

    public void ShowMainMenu()
    {
        ShowOnly(mainMenuPanel);
        Time.timeScale = 1f;
    }

    public void ShowMapSettings()
    {
        ShowOnly(mapSettingsPanel);
    }

    public void ShowGameHUD()
    {
        ShowOnly(gameHUD);
    }

    public void ShowCraftPanel()
    {
        // Крафт показывается поверх HUD
        craftPanel.SetActive(true);
    }

    public void HideCraftPanel()
    {
        craftPanel.SetActive(false);
    }

    public void ShowPause()
    {
        pausePanel.SetActive(true);
        Time.timeScale = 0f;
    }

    public void HidePause()
    {
        pausePanel.SetActive(false);
        Time.timeScale = 1f;
    }

    public void ShowGameOver()
    {
        ShowOnly(gameOverPanel);
        Time.timeScale = 0f;
    }

    public void ShowSettings()
    {
        ShowOnly(settingsPanel);
    }

    public void QuitGame()
    {
#if UNITY_EDITOR
        UnityEditor.EditorApplication.isPlaying = false;
#else
        Application.Quit();
#endif
    }
}
```

---

## Код: MapSettingsPanel.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/UI/MapSettingsPanel.cs
using TMPro;
using UnityEngine;
using UnityEngine.UI;

public class MapSettingsPanel : MonoBehaviour
{
    [Header("Поля ввода")]
    [SerializeField] private TMP_InputField pointCountInput;
    [SerializeField] private TMP_InputField mapSizeInput;
    [SerializeField] private TMP_InputField biomeZonesInput;
    [SerializeField] private TMP_InputField objAttemptsInput;
    [SerializeField] private Toggle         visualizeToggle;
    [SerializeField] private TMP_InputField maxHealthInput;
    [SerializeField] private TMP_InputField maxHungerInput;
    [SerializeField] private TMP_InputField maxThirstInput;
    [SerializeField] private TMP_InputField hpHungerInput;
    [SerializeField] private TMP_InputField hpThirstInput;

    [Header("Ссылки")]
    [SerializeField] private GameLauncher gameLauncher;

    private void OnEnable()
    {
        // Загружаем текущие настройки при открытии панели
        var repo     = new JsonSettingsRepository();
        var settings = repo.Load();
        FillFields(settings);
    }

    private void FillFields(MapSettings s)
    {
        pointCountInput.text   = s.pointCount.ToString();
        mapSizeInput.text      = s.mapSize.ToString();
        biomeZonesInput.text   = s.biomeZones.ToString();
        objAttemptsInput.text  = s.objectPlacementAttempts.ToString();
        visualizeToggle.isOn   = s.visualizeSteps;
        maxHealthInput.text    = s.maxHealth.ToString();
        maxHungerInput.text    = s.maxHunger.ToString();
        maxThirstInput.text    = s.maxThirst.ToString();
        hpHungerInput.text     = s.hpLossOnHunger.ToString();
        hpThirstInput.text     = s.hpLossOnThirst.ToString();
    }

    public void OnGenerateClicked()
    {
        var settings = new MapSettings
        {
            pointCount              = ParseInt(pointCountInput.text, 50),
            mapSize                 = ParseInt(mapSizeInput.text, 100),
            biomeZones              = ParseInt(biomeZonesInput.text, 5),
            objectPlacementAttempts = ParseInt(objAttemptsInput.text, 10),
            visualizeSteps          = visualizeToggle.isOn,
            maxHealth               = ParseFloat(maxHealthInput.text, 100f),
            maxHunger               = ParseFloat(maxHungerInput.text, 100f),
            maxThirst               = ParseFloat(maxThirstInput.text, 100f),
            hpLossOnHunger          = ParseFloat(hpHungerInput.text, 2f),
            hpLossOnThirst          = ParseFloat(hpThirstInput.text, 3f),
        };

        UIManager.Instance.ShowGameHUD();
        gameLauncher.StartWithSettings(settings);
    }

    public void OnBackClicked()
    {
        UIManager.Instance.ShowMainMenu();
    }

    private int   ParseInt(string text, int   fallback) =>
        int.TryParse(text, out int v)     ? v : fallback;

    private float ParseFloat(string text, float fallback) =>
        float.TryParse(text, System.Globalization.NumberStyles.Float,
            System.Globalization.CultureInfo.InvariantCulture, out float v) ? v : fallback;
}
```

---

## Код: GameHUD.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/UI/GameHUD.cs
using TMPro;
using UnityEngine;
using UnityEngine.UI;

public class GameHUD : MonoBehaviour
{
    public static GameHUD Instance { get; private set; }

    [Header("Атрибуты")]
    [SerializeField] private Slider     hpBar;
    [SerializeField] private Slider     hungerBar;
    [SerializeField] private Slider     thirstBar;
    [SerializeField] private TMP_Text   hpLabel;
    [SerializeField] private TMP_Text   hungerLabel;
    [SerializeField] private TMP_Text   thirstLabel;

    [Header("День/ночь")]
    [SerializeField] private TMP_Text   dayTimeText;
    [SerializeField] private Slider     dayNightBar;

    [Header("Кнопки")]
    [SerializeField] private Button     craftButton;
    [SerializeField] private Button     pauseButton;

    private void Awake()
    {
        if (Instance != null) { Destroy(gameObject); return; }
        Instance = this;
    }

    private void Start()
    {
        craftButton.onClick.AddListener(UIManager.Instance.ShowCraftPanel);
        pauseButton.onClick.AddListener(UIManager.Instance.ShowPause);
    }

    public void UpdateStats(float hp, float maxHp, float hunger, float maxHunger,
                             float thirst, float maxThirst)
    {
        hpBar.value     = hp / maxHp;
        hungerBar.value = hunger / maxHunger;
        thirstBar.value = thirst / maxThirst;

        hpLabel.text     = $"HP: {Mathf.RoundToInt(hp)}/{Mathf.RoundToInt(maxHp)}";
        hungerLabel.text = $"Сытость: {Mathf.RoundToInt(hunger)}";
        thirstLabel.text = $"Жажда: {Mathf.RoundToInt(thirst)}";

        // Меняем цвет баров при низком значении
        hpBar.fillRect.GetComponent<Image>().color =
            hp / maxHp < 0.25f ? Color.red : Color.green;
    }

    public void UpdateDayTime(int day, float hour)
    {
        int h    = Mathf.FloorToInt(hour);
        int m    = Mathf.FloorToInt((hour - h) * 60f);
        bool isNight = hour < 6f || hour >= 20f;
        dayTimeText.text = $"День {day} | {h:D2}:{m:D2} {(isNight ? "🌙" : "☀")}";
    }

    public void UpdateDayNightBar(float progress)
    {
        dayNightBar.value = progress;
    }
}
```

---

## Код: PlayerStats.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/UI/PlayerStats.cs
using UnityEngine;

public class PlayerStats : MonoBehaviour
{
    public float MaxHealth  { get; private set; }
    public float MaxHunger  { get; private set; }
    public float MaxThirst  { get; private set; }

    public float CurrentHealth  { get; private set; }
    public float CurrentHunger  { get; private set; }
    public float CurrentThirst  { get; private set; }

    public bool IsExhausted => CurrentHunger <= 0f || CurrentThirst <= 0f;
    public bool IsDead      => CurrentHealth <= 0f;

    private float _hpLossOnHunger;
    private float _hpLossOnThirst;

    // Скорости убывания в час (переводим в секунды)
    private const float HungerDecayPerHour = 10f;
    private const float ThirstDecayPerHour = 15f;

    // Игровое время: 1 реальная минута = 1 игровой час
    private const float RealSecondsPerGameHour = 60f;

    public void Initialize(MapSettings settings)
    {
        MaxHealth  = settings.maxHealth;
        MaxHunger  = settings.maxHunger;
        MaxThirst  = settings.maxThirst;

        CurrentHealth = MaxHealth;
        CurrentHunger = MaxHunger;
        CurrentThirst = MaxThirst;

        _hpLossOnHunger = settings.hpLossOnHunger;
        _hpLossOnThirst = settings.hpLossOnThirst;
    }

    private void Update()
    {
        if (IsDead) return;

        float dt = Time.deltaTime / RealSecondsPerGameHour;

        // Убываем голод и жажду
        CurrentHunger = Mathf.Max(0f, CurrentHunger - HungerDecayPerHour * dt);
        CurrentThirst = Mathf.Max(0f, CurrentThirst - ThirstDecayPerHour * dt);

        // HP потеря при 0
        if (CurrentHunger <= 0f)
            CurrentHealth -= _hpLossOnHunger * dt;
        if (CurrentThirst <= 0f)
            CurrentHealth -= _hpLossOnThirst * dt;

        CurrentHealth = Mathf.Max(0f, CurrentHealth);

        // Обновляем HUD
        if (GameHUD.Instance != null)
            GameHUD.Instance.UpdateStats(CurrentHealth, MaxHealth,
                                          CurrentHunger, MaxHunger,
                                          CurrentThirst, MaxThirst);

        // Проигрыш
        if (IsDead)
            UIManager.Instance.ShowGameOver();
    }

    public void TakeDamage(float amount)
    {
        CurrentHealth = Mathf.Max(0f, CurrentHealth - amount);
    }

    public void Eat(float amount)
    {
        CurrentHunger = Mathf.Min(MaxHunger, CurrentHunger + amount);
    }

    public void Drink(float amount)
    {
        CurrentThirst = Mathf.Min(MaxThirst, CurrentThirst + amount);
    }

    public void Heal(float amount)
    {
        CurrentHealth = Mathf.Min(MaxHealth, CurrentHealth + amount);
    }
}
```

---

## Код: DayNightCycle.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/UI/DayNightCycle.cs
using UnityEngine;

public class DayNightCycle : MonoBehaviour
{
    [SerializeField] private Light  sunLight;

    // 2 минуты = 1 полный цикл (день + ночь)
    public float CycleDurationSeconds = 120f;

    public int   CurrentDay  { get; private set; } = 1;
    public float CurrentHour { get; private set; } = 6f; // начинаем в 6 утра
    public bool  IsNight     => CurrentHour < 6f || CurrentHour >= 20f;

    private float _elapsed = 0f;

    // Градиент освещения: утро → день → вечер → ночь
    private readonly Color _dayColor   = new Color(1f, 0.95f, 0.8f);
    private readonly Color _nightColor = new Color(0.1f, 0.1f, 0.3f);

    private void Update()
    {
        _elapsed += Time.deltaTime;

        // 24 игровых часа за CycleDurationSeconds реальных секунд
        float progress   = (_elapsed / CycleDurationSeconds) % 1f;
        CurrentHour      = progress * 24f;

        // Счётчик дней
        int newDay = Mathf.FloorToInt(_elapsed / CycleDurationSeconds) + 1;
        if (newDay != CurrentDay)
            CurrentDay = newDay;

        UpdateLighting(progress);
        UpdateHUD();
    }

    private void UpdateLighting(float progress)
    {
        if (sunLight == null) return;

        // Поворот солнца: рассвет (6:00 = 0.25 цикла) → закат (20:00 = 0.83 цикла)
        float sunAngle   = progress * 360f - 90f;
        sunLight.transform.rotation = Quaternion.Euler(sunAngle, -30f, 0f);

        // Интенсивность: 0 ночью, 1.5 днём
        float t = Mathf.Sin(progress * Mathf.PI);
        t = Mathf.Clamp01(t);
        sunLight.intensity = Mathf.Lerp(0.05f, 1.5f, t);

        // Цвет освещения
        sunLight.color = Color.Lerp(_nightColor, _dayColor, t);

        // Ambient освещение
        RenderSettings.ambientLight = Color.Lerp(_nightColor * 0.5f, _dayColor * 0.5f, t);
    }

    private void UpdateHUD()
    {
        if (GameHUD.Instance == null) return;

        float progress = (_elapsed / CycleDurationSeconds) % 1f;
        GameHUD.Instance.UpdateDayTime(CurrentDay, CurrentHour);
        GameHUD.Instance.UpdateDayNightBar(IsNight ? 0f : 1f - (progress % 0.5f) * 2f);
    }
}
```

---

## Таблица: кнопка → метод

| Панель | Кнопка | OnClick() метод |
|--------|--------|-----------------|
| MainMenu | PlayButton | UIManager.ShowMapSettings |
| MainMenu | LoadButton | SaveSystem.LoadGame |
| MainMenu | SettingsButton | UIManager.ShowSettings |
| MainMenu | ExitButton | UIManager.QuitGame |
| MapSettings | GenerateButton | MapSettingsPanel.OnGenerateClicked |
| MapSettings | BackButton | MapSettingsPanel.OnBackClicked |
| GameHUD | CraftButton | UIManager.ShowCraftPanel |
| GameHUD | PauseButton | UIManager.ShowPause |
| CraftPanel | CloseButton | UIManager.HideCraftPanel |
| Pause | ResumeButton | UIManager.HidePause |
| Pause | SaveButton | SaveSystem.SaveGame |
| Pause | NewGameButton | UIManager.ShowMainMenu |
| Pause | ToggleAudioButton | AudioManager.Toggle |
| GameOver | LoadButton | SaveSystem.LoadGame |
| GameOver | NewGameButton | UIManager.ShowMainMenu |
| Settings | BackButton | UIManager.ShowMainMenu |

---

## Временной план Модуля Б (2 часа)

| Время | Задача |
|-------|--------|
| 0:00–0:05 | Создать Canvas, настроить Canvas Scaler |
| 0:05–0:20 | MainMenuPanel (фон + 4 кнопки + 3 декора) |
| 0:20–0:40 | MapSettingsPanel (10 полей ввода + 2 кнопки) |
| 0:40–0:55 | GameHUD (3 слайдера + текст + 2 кнопки) |
| 0:55–1:10 | PausePanel + GameOverPanel (5+2 кнопки) |
| 1:10–1:20 | SettingsPanel (5 ползунков) |
| 1:20–1:35 | CraftPanel (6 вкладок + ScrollView) |
| 1:35–1:50 | UIManager.cs + MapSettingsPanel.cs + GameHUD.cs |
| 1:50–2:00 | Подключить кнопки через Inspector (OnClick) |

---

## Типичные ошибки Модуля Б

**Ошибка:** Кнопка не нажимается  
**Решение:** Проверь EventSystem в Hierarchy — он должен быть. Проверь что Canvas Render Mode = Screen Space Overlay.

**Ошибка:** Text не видно  
**Решение:** Проверь цвет текста (не белый на белом фоне). Проверь что компонент Text/TextMeshPro активен.

**Ошибка:** Панели накладываются друг на друга  
**Решение:** Убедись что в UIManager.ShowOnly() все панели скрываются перед показом одной.

**Ошибка:** Slider не меняет цвет  
**Решение:** Fill объект у Slider → Image компонент → Color.

**Ошибка:** SetActive(false) в Start() — панель всё равно видна  
**Решение:** Вызывай ShowMainMenu() в Start() UIManager — он сам скроет всё остальное.
