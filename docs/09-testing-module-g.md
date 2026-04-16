# Модуль Ж — Тестирование и билд (8 баллов, 4 часа)

Финальный модуль. Проверяй ВСЁ по чеклисту.

---

## Как сделать Windows Build

### Шаги
1. `File` → `Build Settings` (Ctrl+Shift+B)
2. Платформа: `Windows, Mac, Linux` (должна быть выбрана по умолчанию)
3. Target Platform: `Windows`
4. Architecture: `x86_64`
5. Нажми **Add Open Scenes** — SampleScene появится в списке
6. Нажми **Player Settings...** → настрой (см. ниже)
7. Нажми **Build** → выбери папку `Builds/Final/`
8. Подожди (1–5 минут)
9. Проверь: запусти .exe из папки, убедись что работает БЕЗ Unity Editor

### Player Settings
- **Company Name:** твоё имя или школа
- **Product Name:** `VoronoiSurvival`
- **Version:** `1.0`
- **Default Screen Width:** `1920`
- **Default Screen Height:** `1080`
- **Fullscreen Mode:** `Fullscreen Window`
- **Run In Background:** ✅ (не обязательно)
- **Icon:** опционально

### Что происходит при билде
- Unity компилирует весь C# код
- Собирает все Assets которые используются в сценах
- StreamingAssets копируется автоматически (целая папка!)
- Создаётся .exe + папка _Data/ со всем содержимым

---

## Чеклист перед каждым билдом

```
□ Все сцены добавлены в Build Settings (File → Build Settings → Add Open Scenes)
□ StreamingAssets содержит все JSON файлы (mapSettings.json, settings.json, etc.)
□ Player Settings настроен (название, разрешение)
□ Нет Console errors (Window → General → Console → проверить Error колонку)
□ В Play Mode всё работает
□ Сцена сохранена (Ctrl+S)
```

---

## Чеклист после билда

```
□ .exe запускается без Unity Editor
□ Окно открывается на 1920×1080
□ Главное меню отображается
□ Кнопка "Играть" открывает настройки карты
□ Генерация карты работает (нажать "Генерировать!")
□ Карта видна в 3D
□ Игрок заспавнился
□ WASD двигает игрока
□ StreamingAssets файлы можно редактировать в папке билда
□ После редактирования mapSettings.json изменения применяются при следующей генерации
```

---

## Export Package — для каждого модуля

### Пошагово
1. В Project окне выбери папку `Assets` (или конкретные папки)
2. Меню: `Assets` → `Export Package...`
3. В окне отмечены все файлы — убедись что галочки везде
4. Опционально нажми **Include dependencies**
5. Нажми `Export...`
6. Сохрани: `ModuleA.unitypackage`, `ModuleB.unitypackage` и т.д.

### Что включать
- Assets/Scripts/ — всё
- Assets/StreamingAssets/ — всё
- Assets/ScriptableObjects/ — биомы
- Assets/Prefabs/ — все префабы
- Assets/Materials/ — все материалы
- Assets/Scenes/ — SampleScene

### Когда делать Export Package
После завершения каждого модуля — не откладывай на потом!

---

## Функциональные тесты по модулям

### Модуль А — Структура
```
□ Папки Data/, Interfaces/, Algorithms/, Steps/, Persistence/, UI/ существуют
□ BiomeType ScriptableObject создаётся через меню "Voronoi/Biome Type"
□ 5 биомов: Grass(старт), Forest, Rocks, Water(вода), Desert
□ У Grass: isStartBiome = true
□ У Water: hasWater = true
□ Сцена: GameManager, Camera (50,80,50), DirectionalLight
```

### Модуль Б — UI
```
□ MainMenu показывается при старте
□ 4 кнопки работают: Играть, Загрузить, Настройки, Выход
□ Главное меню: 3+ элемента природы на фоне
□ MapSettings: 8 InputField + Toggle + 2 кнопки
□ GameHUD: HP/Hunger/Thirst слайдеры + день/время
□ CraftPanel: 6 вкладок + ScrollView
□ PausePanel: 5 кнопок + полупрозрачный фон
□ GameOverPanel: 2 кнопки + тёмный фон
□ SettingsPanel: 5 ползунков/списков
□ Все панели Canvas, разрешение 1920×1080
□ Нестандартный шрифт (если выданы ресурсы)
```

### Модуль В — JSON
```
□ mapSettings.json существует в StreamingAssets/
□ Изменение pointCount в JSON → при следующей генерации применяется
□ settings.json существует
□ craft.json содержит минимум 3 рецепта
□ locale_ru.json содержит переводы
□ locale_en.json содержит переводы
□ save.json создаётся при сохранении
□ Автосохранение каждые 30 секунд
□ Загрузка сохранения восстанавливает позицию игрока
□ Локализация переключается (ru/en)
```

### Модуль Г — Генерация и выживание
```
□ Вороной строится за ≤0.5с на зону
□ Количество клеток ±10% от pointCount
□ Биомы назначаются (разные цвета)
□ Меш террейна виден (3D поверхность)
□ 4 стены по краям карты (невидимые или видимые)
□ Объекты в биомах РАЗНЫЕ (деревья vs камни vs кактусы)
□ Игрок спавнится в стартовом биоме
□ HP/Hunger/Thirst убывают со временем
□ При HP=0 → экран проигрыша
□ 3 типа животных (рыбы, нейтральные, агрессивные)
□ Нейтральные убегают при атаке
□ Агрессивные атакуют первыми
□ Рыбы только в водных биомах
□ Стартовый биом без агрессивных
```

### Модуль Д — Управление
```
□ WASD — движение
□ Shift — бег (скорость ×2)
□ Q/E — вращение камеры
□ Mouse Wheel — приближение/удаление
□ C — открывает крафт
□ Pause — пауза
□ F11 — экран проигрыша
□ Space — автоподбор
□ ЛКМ — атака/взаимодействие
□ ПКМ на объекте — контекстное меню
□ При IsExhausted скорость ×0.5
□ Камера следует за игроком
□ Контекстное меню: дерево+руки → осмотреть, удар
```

### Модуль Е — Анимации и аудио
```
□ Прогресс-бар при рубке дерева (зелёный, "Работаем...")
□ Красное мигание при отмене прогресс-бара
□ 5 анимаций животных (даже если фиктивные клипы)
□ 6 анимаций персонажа (Idle/Walk/Run/Attack/Eat/WeaponSwitch)
□ WeaponSwitch 0.5с длительность
□ День/ночь: 2 минуты цикл
□ Ночью: животные стоят, урон ×0.5
□ Прогресс-бар дня/ночи в HUD
□ Экран краснеет при получении урона
□ Числа урона всплывают над целью
□ Инвентарь выезжает анимированно (0.3с)
□ Звук проигрыша ✅
□ Звук строительства ✅
```

### Модуль Ж — Финальный
```
□ Билд запускается без Unity
□ 1920×1080 fullscreen
□ Player Settings заполнены
□ StreamingAssets в папке билда, JSON редактируемы
□ Нет крашей при 5-минутной игре
□ Export Package для каждого модуля создан
```

---

## Распространённые ошибки Unity 6 и их исправления

### "FindObjectOfType is obsolete"
```csharp
// Устаревший способ:
FindObjectOfType<SomeClass>()

// Unity 6 способ:
FindFirstObjectByType<SomeClass>()
FindAnyObjectByType<SomeClass>()
FindObjectsByType<SomeClass>(FindObjectsSortMode.None)
```

### "NullReferenceException" на SerializeField
Ошибка: `NullReferenceException: Object reference not set...`
- Причина: поле помечено `[SerializeField]` но не назначено в Inspector
- Решение:
  1. Выбери объект в Hierarchy
  2. В Inspector найди компонент
  3. Перетащи нужный объект в пустое поле

### "Shader not found" в билде
```csharp
// Проблема: URP шейдеры не включены в билд
// Решение: Edit → Project Settings → Graphics
// → Always Included Shaders → добавить нужные шейдеры

// Или использовать Shader.Find с проверкой:
var shader = Shader.Find("Universal Render Pipeline/Lit");
if (shader == null)
    shader = Shader.Find("Standard"); // fallback
```

### "Physics не работает" — объекты проваливаются
- Причина 1: нет Collider на объекте
- Причина 2: Rigidbody Is Kinematic = true
- Причина 3: Layer Collision Matrix не настроена
- Решение: Edit → Project Settings → Physics → Layer Collision Matrix
  - Убедись что Default ↔ Default включено

### "Корутина не запускается"
```csharp
// Нельзя запустить корутину на неактивном объекте:
// gameObject.SetActive(false)  →  StartCoroutine не работает

// Нельзя запустить на обычном классе (не MonoBehaviour):
// public static class MyClass  →  нет StartCoroutine

// Правильно: вызывать StartCoroutine только на активном MonoBehaviour
```

### "JsonUtility не сериализует"
```csharp
// Класс ДОЛЖЕН быть помечен [Serializable]:
[System.Serializable]
public class MyData { ... }

// Поля ДОЛЖНЫ быть public:
public int myField; // ✅

// Или иметь [SerializeField]:
[SerializeField] private int _myField; // ✅ для JsonUtility тоже работает
```

### "Time.timeScale = 0 блокирует корутины"
```csharp
// НЕ работает при паузе:
yield return new WaitForSeconds(0.3f);

// РАБОТАЕТ при паузе:
yield return new WaitForSecondsRealtime(0.3f);

// В Update (НЕ работает при паузе):
Time.deltaTime

// В Update (РАБОТАЕТ при паузе):
Time.unscaledDeltaTime
```

### "Объект в сцене после Destroy всё ещё null check проходит"
```csharp
// Unity перегружает оператор == для уничтоженных объектов:
if (myObj != null) // проверяет C# null И Unity "destroyed"
{
    // ...
}

// Если нужно точно:
if (myObj != null && myObj)
{
    // ...
}
```

---

## Производительность — советы

### Общий материал (sharedMaterial vs material)
```csharp
// material создаёт копию (плохо — много копий):
renderer.material.color = Color.red; // ❌ создаёт копию материала

// sharedMaterial меняет оригинал (хорошо для статичных объектов):
renderer.sharedMaterial.color = Color.red; // ✅ одна копия на всех
```

Для клеток Вороного: создаём отдельный материал на каждую клетку (из-за разных цветов). Это нормально для нашего проекта.

### Статический батчинг для террейна
Для повышения производительности — пометь объекты террейна как Static:
```csharp
// В TerrainStep после создания объекта:
go.isStatic = true;
```
После этого Unity объединит меши в один draw call.

**ОСТОРОЖНО:** Static объекты нельзя двигать в runtime!

### Pooling для частиц и объектов
Если создаёшь/уничтожаешь много объектов (числа урона, стрелы) — используй пул:
```csharp
// Простейший пул (если нужно):
// Создай 10 объектов в Start(), при использовании SetActive(true),
// после использования SetActive(false) — вместо Destroy/Instantiate.
```

### FindObjectOfType — дорого
```csharp
// Плохо (вызывается каждый кадр):
void Update()
{
    var dayNight = FindFirstObjectByType<DayNightCycle>(); // дорого!
}

// Хорошо (кешировать в Start/Awake):
private DayNightCycle _dayNight;
void Start()
{
    _dayNight = FindFirstObjectByType<DayNightCycle>();
}
void Update()
{
    if (_dayNight != null) { ... }
}
```

---

## Последовательность на день соревнования

### День 1 (примерно 5-6 часов)
1. Создать проект + папки (30 мин = Модуль А)
2. Написать все Data/, Interfaces/ скрипты (30 мин)
3. Написать VoronoiGenerator + CellMeshBuilder (40 мин)
4. Написать все Steps/ (1 ч)
5. Написать MapGenerator + GameLauncher (20 мин)
6. Настроить сцену, проверить в Play Mode (20 мин)
7. **Export Package + Build** (= Модуль А завершён)
8. Написать AnimalsStep (30 мин)
9. Добавить PlayerStats, DayNightCycle (30 мин)
10. **Export Package + Build** (= Модуль Г частично)

### День 2 (5-6 часов)
11. Создать Canvas + все панели (2 ч = Модуль Б)
12. **Export Package + Build**
13. Создать все JSON файлы (30 мин)
14. Написать SaveSystem, LocalizationManager, CraftDatabase (2 ч = Модуль В)
15. **Export Package + Build**
16. Написать PlayerController + ContextMenu (1 ч = Модуль Д частично)

### День 3 (4-5 часов)
17. Завершить PlayerController (1 ч = Модуль Д)
18. **Export Package + Build**
19. Animator, звуки, UI анимации (2 ч = Модуль Е)
20. **Export Package + Build**
21. Финальное тестирование (1 ч)
22. Исправить найденные баги
23. **Финальный Export Package + Build** (= Модуль Ж)
24. Сдать всё

---

## Финальный чеклист перед сдачей

```
□ Export Package для каждого из 7 модулей создан и сохранён
□ Build запускается на Windows без Unity
□ Разрешение 1920×1080
□ Все 7 модулей проверены по чеклистам выше
□ Именование: PascalCase, без транслита, без русских букв в коде
□ Структура папок: Data/, Interfaces/, Algorithms/, Steps/, Persistence/, UI/
□ SOLID: каждый класс делает одно, интерфейсы маленькие, DI через GameLauncher
□ JSON файлы в StreamingAssets/ читаемы и редактируемы в билде
□ Нет консольных ошибок (только Warning — допустимо)
```
