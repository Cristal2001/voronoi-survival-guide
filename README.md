# Voronoi Survival — Чемпионат Профессионалы 2026

Полное руководство для воспроизведения проекта с нуля на чемпионате без интернета.

**Движок:** Unity 6.0.4, Universal Render Pipeline (URP)  
**Язык:** C#  
**Тип:** 3D survival-игра с процедурной генерацией карты (диаграмма Вороного)  
**Время:** 16 часов, 3 дня  
**Баллы:** 100

---

## Содержание

| Файл | Что внутри |
|------|-----------|
| [docs/01-project-setup.md](docs/01-project-setup.md) | Модуль А — создание проекта, папки, ScriptableObject |
| [docs/02-voronoi-algorithm.md](docs/02-voronoi-algorithm.md) | Модуль — алгоритм Вороного, объяснение архитектуры |
| [docs/03-all-scripts.md](docs/03-all-scripts.md) | ВСЕ скрипты проекта — полный код |
| [docs/04-ui-module-b.md](docs/04-ui-module-b.md) | Модуль Б — весь UI, Canvas, панели |
| [docs/05-json-module-c.md](docs/05-json-module-c.md) | Модуль В — JSON, сохранения, локализация |
| [docs/06-gameplay-module-d.md](docs/06-gameplay-module-d.md) | Модуль Г — механики выживания, животные |
| [docs/07-controls-module-e.md](docs/07-controls-module-e.md) | Модуль Д — управление персонажем |
| [docs/08-animations-module-f.md](docs/08-animations-module-f.md) | Модуль Е — анимации и аудио |
| [docs/09-testing-module-g.md](docs/09-testing-module-g.md) | Модуль Ж — тестирование и билд |
| [docs/10-cheatsheet.md](docs/10-cheatsheet.md) | Шпаргалка на день соревнования |

---

## Таблица модулей

| Модуль | Название | Баллы | Время | Что делать |
|--------|----------|-------|-------|------------|
| А | Структура проекта и импорт | 6 | 30 мин | Создать проект, папки, ScriptableObject, 5 биомов, первый билд |
| Б | Пользовательский интерфейс | 22 | 2 ч | 7 панелей UI: меню, HUD, крафт, пауза, проигрыш, настройки, карта |
| В | Хранение данных (JSON) | 17 | 4 ч | JSON файлы, сохранение, загрузка, локализация, крафт-база |
| Г | Генерация карты + механики выживания | 24 | 3 ч | Вороной, биомы, животные, оружие, броня, атрибуты игрока |
| Д | Управление персонажем | 11 | 2 ч 30 мин | PlayerController, камера, ввод, контекстное меню |
| Е | Анимации и аудио | 12 | 2 ч | Animator, день/ночь, звуки, UI анимации, прогресс-бары |
| Ж | Тестирование и билд | 8 | 4 ч | Финальный билд, тесты, Export Package для каждого модуля |

---

## После каждого модуля — сдать:

### 1. Билд (Build)
```
File → Build Settings → Windows, x86_64
→ Add Open Scenes
→ Build
→ Выбрать папку: Builds/ModuleX/
```

### 2. Export Package
```
В Project окне → выделить папку Assets
→ ПКМ → Export Package...
→ Галочка "Include dependencies"
→ Export → сохранить как "ModuleX.unitypackage"
```

**ВАЖНО:** Делай Export Package после КАЖДОГО модуля. Не жди конца дня.

---

## Правила чемпионата (обязательно)

### Именование
- Классы и методы — **PascalCase**: `MapGenerator`, `Execute`, `BiomeIndex`
- Приватные поля — **_camelCase**: `_mapGenerator`, `_context`
- Локальные переменные — **camelCase**: `cellCount`, `biomeIndex`
- **НЕТ транслита**: не `igrok`, не `biom`, не `karta`
- **НЕТ русских букв** в именах классов, переменных, папок, файлов
- Имена отражают суть: `VoronoiStep` хорошо, `Step1` плохо

### Архитектура SOLID
| Принцип | Правило | Пример |
|---------|---------|--------|
| S — Single Responsibility | Один класс = одно дело | `VoronoiGenerator` только математика |
| O — Open/Closed | Новый шаг = новый файл | `AnimalsStep.cs` без правки других |
| L — Liskov Substitution | Все шаги взаимозаменяемы | Любой `IMapGenerationStep` можно заменить |
| I — Interface Segregation | Интерфейсы маленькие | `IMapGenerationStep` — 2 члена, не 10 |
| D — Dependency Inversion | Зависим от абстракций | `MapGenerator` знает только `IMapGenerationStep` |

### Структура папок (обязательная)
```
Assets/
└── Scripts/
    └── VoronoiMapGenerator/
        ├── Data/
        ├── Interfaces/
        ├── Algorithms/
        ├── Steps/
        ├── Persistence/
        └── UI/
```

---

## Порядок работы на чемпионате

### День 1 (утро) — Модуль А (30 мин)
1. Создать Unity проект (3D URP)
2. Создать все папки в Scripts/VoronoiMapGenerator/
3. Написать `BiomeType.cs` с `[CreateAssetMenu]`
4. Создать 5 BiomeType активов
5. Настроить сцену (GameManager, Camera, Light)
6. Сделать билд + Export Package

### День 1 (основная часть) — Модуль Г (3 ч)
7. Написать все Data/ классы
8. Написать Interfaces/
9. Написать Algorithms/ (VoronoiGenerator, CellMeshBuilder)
10. Написать все Steps/
11. Написать GenerationContext, MapGenerator, GameLauncher
12. Проверить генерацию в Play Mode
13. Добавить животных (AnimalsStep)
14. Сделать билд + Export Package

### День 1-2 — Модуль Б (2 ч)
15. Создать Canvas с 7 панелями
16. Написать UIManager, GameHUD, MapSettingsPanel
17. Подключить кнопки
18. Сделать билд + Export Package

### День 2 — Модуль В (4 ч)
19. Создать все JSON файлы в StreamingAssets/
20. Написать SettingsRepository, SaveSystem
21. Написать LocalizationManager
22. Написать CraftDatabase
23. Подключить к UI
24. Сделать билд + Export Package

### День 2-3 — Модуль Д (2.5 ч)
25. Написать PlayerController
26. Настроить Input (WASD, Shift, Q/E, мышь)
27. Контекстное меню (ПКМ)
28. Сделать билд + Export Package

### День 3 — Модуль Е (2 ч)
29. Animator Controller для игрока
30. Animator Controller для животных
31. DayNightCycle
32. Звуки (AudioSource)
33. UI анимации (урон, инвентарь)
34. Сделать билд + Export Package

### День 3 — Модуль Ж (4 ч)
35. Финальное тестирование ВСЕГО
36. Исправить ошибки
37. Финальный билд
38. Export Package финальный
39. Сдача

---

## Быстрая справка по Unity 6

| Задача | Путь |
|--------|------|
| Создать проект | Unity Hub → New Project → 3D (URP) Core |
| Создать папку | Project → ПКМ → Create → Folder |
| Создать C# скрипт | Project → ПКМ → Create → C# Script |
| Создать ScriptableObject | ПКМ в Project → Voronoi → Biome Type |
| Package Manager | Window → Package Manager |
| Build Settings | File → Build Settings |
| Export Package | Assets → Export Package |
| Play Mode | Ctrl+P |
| Консоль | Window → General → Console |
