# 🗺️ Voronoi Survival — Чемпионат Профессионалы 2026

![Unity](https://img.shields.io/badge/Unity-6.0.4-black?logo=unity) ![URP](https://img.shields.io/badge/Render-URP-blue) ![C#](https://img.shields.io/badge/Language-C%23-purple) ![Points](https://img.shields.io/badge/Баллов-100-gold)

> **3D survival-игра** с процедурной генерацией карты через диаграмму Вороного.
> Unity 6 · URP · 3 дня · 16 часов · 7 модулей.

---

## 📋 Как пользоваться этим репозиторием

```
Каждый день на чемпионате:
  1. Идёшь в интернет-кафе (15 мин)
  2. Открываешь ЛИСТОК нужного дня (см. таблицу ниже)
  3. Запоминаешь / переписываешь на А4 только формулы
  4. Работаешь весь день с листком
  5. Если нужна деталь — открываешь нужный DOC
```

---

## 📅 Листки по дням

> Открывай с утра в интернет-кафе. Каждый читается за 5 минут.

| День | Модули | Баллы | Время | Листок |
|------|--------|:-----:|:-----:|:------:|
| **День 1** | А — Структура проекта + Б — UI | **28** | ~5 ч | [📄 Открыть](sheets/day1.md) |
| **День 2** | В — JSON + Г — Карта + Д — Управление | **52** | ~8 ч | [📄 Открыть](sheets/day2.md) |
| **День 3** | Е — Анимации + Ж — Билд и тесты | **20** | ~6 ч | [📄 Открыть](sheets/day3.md) |

---

## 🚀 С чего начать — пустой проект

> Это первое что делаешь на чемпионате, ещё до листка.

### Шаг 1 — Создать проект Unity
```
Unity Hub → New Project → 3D (URP) → Имя: Dernov_2027 → Create
```

### Шаг 2 — Создать структуру папок
В `Assets/` создать `Scripts/VoronoiMapGenerator/`, внутри неё 6 папок:
```
Data/           ← модели данных
Interfaces/     ← контракты (интерфейсы)
Algorithms/     ← математика
Steps/          ← шаги генерации
Persistence/    ← работа с файлами
UI/             ← скрипты интерфейса
```
И три файла прямо в `VoronoiMapGenerator/` (не в подпапках):
```
GenerationContext.cs
MapGenerator.cs
GameLauncher.cs
```

### Шаг 3 — Настроить сцену
```
Создать пустой объект → назвать: GameManager
  → добавить компоненты: MapGenerator + GameLauncher

Main Camera:
  → Position:  X=50  Y=80  Z=50
  → Rotation:  X=60  Y=0   Z=0
```

### Шаг 4 — Создать StreamingAssets
```
Assets/ → ПКМ → Create Folder → StreamingAssets
Создать внутри 6 текстовых файлов (расширение .json):
  mapSettings.json   ← настройки генерации и атрибуты игрока
  settings.json      ← звук, разрешение, язык, мышь
  save.json          ← состояние игры (автосохранение)
  craft.json         ← рецепты крафта
  locale_ru.json     ← тексты интерфейса на русском
  locale_en.json     ← тексты интерфейса на английском
```

### Шаг 5 — Порядок написания скриптов
```
1.  Data/MapSettings.cs               ← [Serializable] класс настроек
2.  Data/VoronoiCell.cs               ← Site + Polygon + BiomeIndex
3.  Data/BiomeType.cs                 ← ScriptableObject биома
4.  Interfaces/IMapGenerationStep.cs  ← Name + Execute(ctx)
5.  Interfaces/ISettingsRepository.cs ← Load() + Save()
6.  GenerationContext.cs              ← общий "рюкзак" данных
7.  Persistence/JsonSettingsRepository.cs
8.  Algorithms/VoronoiGenerator.cs   ← самый сложный!
9.  Algorithms/CellMeshBuilder.cs    ← формулы меша!
10. MapGenerator.cs                  ← оркестратор шагов
11. Steps/InitializeStep.cs
12. Steps/VoronoiStep.cs
13. Steps/BiomeAssignStep.cs
14. Steps/TerrainStep.cs
15. Steps/WallsStep.cs
16. Steps/ObjectsStep.cs
17. Steps/PlayerSpawnStep.cs
18. GameLauncher.cs                  ← всё соединяет
19. UI/UIManager.cs
20. UI/MapSettingsPanel.cs
21. UI/GameHUD.cs
22. UI/PlayerStats.cs
23. UI/DayNightCycle.cs
```

---

## 📚 Документация по модулям

> Открывай если нужны детали, полный код или объяснения.

| # | Файл | Что внутри |
|---|------|-----------|
| 01 | [Настройка проекта](docs/01-project-setup.md) | Unity 6 URP, папки, биомы, сцена, Export Package |
| 02 | [Алгоритм Вороного](docs/02-voronoi-algorithm.md) | Как работает, SOLID, аналогии, порядок вызовов |
| 03 | [Все скрипты](docs/03-all-scripts.md) | **Полный код всех 18+ классов** |
| 04 | [UI — Модуль Б](docs/04-ui-module-b.md) | 7 панелей, иерархия Canvas, скрипты, кнопки |
| 05 | [JSON — Модуль В](docs/05-json-module-c.md) | Все JSON-файлы, SaveSystem, LocalizationManager |
| 06 | [Геймплей — Модуль Г](docs/06-gameplay-module-d.md) | Животные, оружие, броня, атрибуты |
| 07 | [Управление — Модуль Д](docs/07-controls-module-e.md) | PlayerController, камера, контекстное меню |
| 08 | [Анимации — Модуль Е](docs/08-animations-module-f.md) | Animator, звуки, день/ночь, UI-эффекты |
| 09 | [Тестирование — Модуль Ж](docs/09-testing-module-g.md) | Билд, ошибки Unity 6, чеклисты |
| 10 | [Шпаргалка](docs/10-cheatsheet.md) | Формулы, SOLID кратко, ловушки |

---

## 🏆 Правила оценки — не забыть!

### Именование (проверяется строго)
| Что | Стиль | Пример |
|-----|-------|--------|
| Классы, методы, свойства | PascalCase | `MapGenerator`, `Execute()` |
| Приватные поля | _camelCase | `_context`, `_steps` |
| Локальные переменные | camelCase | `cell`, `biomeIndex` |
| ❌ Нельзя | Транслит / русские буквы | `Generacia`, `генерация` |

### SOLID (проверяется на каждом модуле)
| Принцип | Правило в этом проекте |
|---------|----------------------|
| **S** — Single Responsibility | Один класс = одно дело |
| **O** — Open/Closed | Новый шаг = новый файл, старые не трогать |
| **L** — Liskov | Все `IMapGenerationStep` взаимозаменяемы |
| **I** — Interface Segregation | Интерфейс маленький: только `Name` + `Execute` |
| **D** — Dependency Inversion | `MapGenerator` зависит от интерфейса, не от конкретных классов |

### После каждого модуля сдать
```
1. Build  →  File → Build Settings → Add Open Scenes → Build
2. Export Package  →  Assets → ПКМ → Export Package → ModuleX.unitypackage
```

---

## 📊 Таблица модулей

| Модуль | Название | Баллы | Время |
|:------:|----------|:-----:|:-----:|
| А | Структура проекта и импорт | 6 | 30 мин |
| Б | Пользовательский интерфейс | 22 | 2 ч |
| В | Хранение данных (JSON) | 17 | 4 ч |
| Г | Генерация карты + механики выживания | 24 | 3 ч |
| Д | Управление персонажем | 11 | 2 ч 30 мин |
| Е | Анимации и аудио | 12 | 2 ч |
| Ж | Тестирование и билд | 8 | 4 ч |
| | **ИТОГО** | **100** | **16 ч** |

---

*Репозиторий обновляется по ходу подготовки. Все изменения и дополнения публикуются сюда автоматически.*
