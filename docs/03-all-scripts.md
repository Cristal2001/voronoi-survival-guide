# 📜 Все скрипты проекта — полный код с комментариями

> Копируй точно. Не меняй имена классов, методов, пространств имён.
> Комментарий к каждой строке объясняет зачем она нужна.

---

## Data/MapSettings.cs

```csharp
using System;      // [Serializable]
using UnityEngine; // не нужен но Unity добавляет автоматически

// [Serializable] — обязателен чтобы JsonUtility мог сохранить этот класс в JSON.
// Без него JsonUtility.ToJson() вернёт пустую строку "{}".
[Serializable]
public class MapSettings
{
    public int   pointCount              = 50;   // количество точек Вороного (больше = мельче клетки)
    public int   mapSize                 = 100;  // размер карты (квадрат N×N Unity-единиц)
    public int   biomeZones              = 5;    // сколько разных биомов появится на карте
    public int   objectPlacementAttempts = 10;   // попыток разместить объект в каждой клетке
    public bool  visualizeSteps          = false; // показывать анимацию генерации по шагам
    public float maxHealth               = 100f; // максимальное здоровье игрока
    public float maxHunger               = 100f; // максимальный голод игрока
    public float maxThirst               = 100f; // максимальная жажда игрока
    public float hpLossOnHunger          = 2f;   // HP/час теряется при голоде = 0
    public float hpLossOnThirst          = 3f;   // HP/час теряется при жажде = 0
}
```

---

## Data/VoronoiCell.cs

```csharp
using System.Collections.Generic; // List<Vector2>
using UnityEngine;                 // Vector2

// Модель одной клетки Вороного. Только данные, никакой логики.
// Принцип S (SRP): хранит данные, не обрабатывает их.
public class VoronoiCell
{
    public Vector2       Site       { get; set; } // центральная точка клетки (seed-точка)
    public List<Vector2> Polygon    { get; set; } = new List<Vector2>(); // вершины контура клетки
    public int           BiomeIndex { get; set; } // индекс биома в массиве ctx.Biomes
}
```

---

## Data/BiomeType.cs

```csharp
using UnityEngine; // ScriptableObject, Color, GameObject, CreateAssetMenu

// ScriptableObject — ресурс (asset) в Project, не MonoBehaviour на объекте.
// Создаётся через: ПКМ в Project → Create → Voronoi → Biome Type
[CreateAssetMenu(menuName = "Voronoi/Biome Type")] // регистрирует пункт меню для создания
public class BiomeType : ScriptableObject
{
    public string     biomeName;              // название биома ("Forest", "Desert" и т.д.)
    public Color      groundColor = Color.green; // цвет земли этого биома
    public float      heightVariation = 1f;  // максимальная высота холмов (0 = плоский)
    public bool       isStartBiome   = false; // true у одного биома → туда спавнится игрок
    public bool       hasWater       = false; // true → в этом биоме есть вода (для рыбалки)
    public GameObject[] objectPrefabs;        // префабы объектов (деревья, камни, трава)
}
```

---

## Interfaces/IMapGenerationStep.cs

```csharp
using System.Collections; // IEnumerator

// Контракт (интерфейс) для каждого шага генерации.
// Принцип I (ISP): только два члена — больше не нужно.
// Принцип D (DIP): MapGenerator зависит от этого интерфейса, не от конкретных классов.
public interface IMapGenerationStep
{
    string      Name    { get; }              // имя шага для отображения в логе
    IEnumerator Execute(GenerationContext ctx); // выполнить шаг; ctx — общий "рюкзак" данных
}
```

---

## Interfaces/ISettingsRepository.cs

```csharp
// Контракт для хранилища настроек.
// Реализация: JsonSettingsRepository (читает/пишет JSON).
// Можно добавить PlayerPrefsRepository не трогая существующий код.
public interface ISettingsRepository
{
    MapSettings Load();                  // загрузить настройки (из файла, БД, и т.д.)
    void        Save(MapSettings settings); // сохранить настройки
}
```

---

## GenerationContext.cs

```csharp
using System.Collections.Generic; // List<VoronoiCell>
using UnityEngine;                 // Transform, GameObject

// "Рюкзак" — общий контейнер данных, который передаётся между всеми шагами генерации.
// Принцип S (SRP): только хранение данных, никакой логики.
// Каждый шаг читает нужные данные и/или записывает результат сюда.
public class GenerationContext
{
    public MapSettings       Settings     { get; set; } // параметры из JSON/UI
    public BiomeType[]       Biomes       { get; set; } // массив биомов из инспектора
    public Transform         Root         { get; set; } // родительский объект для генерации
    public List<VoronoiCell> Cells        { get; set; } = new List<VoronoiCell>(); // клетки
    public float             NoiseOffsetX { get; set; } // X-сдвиг Perlin (уникальность карты)
    public float             NoiseOffsetY { get; set; } // Y-сдвиг Perlin (уникальность карты)
    public GameObject        PlayerPrefab { get; set; } // префаб игрока (может быть null)
}
```

---

## Persistence/JsonSettingsRepository.cs

```csharp
using System.IO;   // File, Path
using UnityEngine; // Application, JsonUtility

// Реализует ISettingsRepository через JSON-файл в StreamingAssets.
// StreamingAssets — особая папка Unity: копируется в билд и доступна для чтения/записи.
public class JsonSettingsRepository : ISettingsRepository
{
    // Путь к файлу настроек. Application.streamingAssetsPath работает и в редакторе, и в билде.
    private string Path => System.IO.Path.Combine(
        Application.streamingAssetsPath, // папка StreamingAssets
        "mapSettings.json"               // имя файла
    );

    public MapSettings Load()
    {
        if (File.Exists(Path))      // проверяем что файл существует
            try
            {
                // ReadAllText → читаем весь файл как строку.
                // JsonUtility.FromJson<T> → десериализуем JSON в объект MapSettings.
                return JsonUtility.FromJson<MapSettings>(File.ReadAllText(Path));
            }
            catch { } // если JSON повреждён — возвращаем дефолтные настройки ниже

        return new MapSettings(); // файла нет или JSON сломан → дефолтные значения
    }

    public void Save(MapSettings s) =>
        // ToJson(s, true) → true включает форматирование с отступами (читаемый JSON).
        // WriteAllText → перезаписывает файл (создаёт если не существует).
        File.WriteAllText(Path, JsonUtility.ToJson(s, true));
}
```

---

## Algorithms/VoronoiGenerator.cs

```csharp
using System.Collections.Generic; // List<T>
using UnityEngine;                 // Vector2, Random

// ─────────────────────────────────────────────────────────────────────────────
// Строит диаграмму Вороного методом пересечения полуплоскостей.
//
// ИДЕЯ (простыми словами):
//   Представь карту как кусок теста. Ты воткнул в него N булавок (точек-sites).
//   Каждой булавке принадлежит та территория, до которой от неё БЛИЖЕ,
//   чем до любой другой булавки. Граница между двумя территориями —
//   это серединный перпендикуляр между двумя булавками.
//
// АЛГОРИТМ (Sutherland-Hodgman):
//   Для каждой точки A:
//     1. Берём квадрат всей карты как начальный полигон.
//     2. Для каждого соседа B:
//        - Находим середину отрезка A-B → это точка на границе.
//        - Вектор (B-A) — нормаль, смотрит в сторону B.
//        - Срезаем из полигона всё, что лежит по стороне B (dot > 0).
//     3. Что осталось — клетка Вороного для точки A.
// ─────────────────────────────────────────────────────────────────────────────
public static class VoronoiGenerator
{
    // Принимает список точек (sites) и размер квадратной карты.
    // Возвращает список готовых клеток Вороного.
    public static List<VoronoiCell> Build(List<Vector2> points, float mapSize)
    {
        var cells = new List<VoronoiCell>(); // сюда накапливаем готовые клетки

        foreach (Vector2 site in points) // перебираем каждую точку-"город"
        {
            // Начальный полигон — весь квадрат карты (4 угла по часовой стрелке).
            // Каждая итерация будет его урезать.
            var polygon = new List<Vector2>
            {
                new Vector2(0,       0      ), // нижний левый угол
                new Vector2(mapSize, 0      ), // нижний правый угол
                new Vector2(mapSize, mapSize), // верхний правый угол
                new Vector2(0,       mapSize), // верхний левый угол
            };

            foreach (Vector2 other in points) // сравниваем с каждой другой точкой
            {
                if (other == site) continue; // пропускаем саму себя

                // Середина между текущей точкой и соседом — граница их территорий.
                Vector2 mid    = (site + other) * 0.5f;

                // Нормаль — вектор от site к other, единичной длины.
                // Указывает «в сторону чужой территории».
                Vector2 normal = (other - site).normalized;

                // Срезаем часть полигона, которая ближе к other, чем к site.
                polygon = ClipPolygon(polygon, mid, normal);

                if (polygon.Count == 0) break; // полигон полностью срезан — пропускаем точку
            }

            // Клетка валидна только если после всех срезов осталось минимум 3 вершины.
            if (polygon.Count >= 3)
                cells.Add(new VoronoiCell { Site = site, Polygon = polygon });
        }

        return cells; // возвращаем список всех построенных клеток
    }

    // ─── Sutherland-Hodgman clip ───────────────────────────────────────────
    // Срезает часть полигона по прямой линии.
    //
    // Как определить по какую сторону прямой находится точка?
    //   Считаем dot(point - origin, normal):
    //   ≤ 0 → точка на нашей стороне (оставляем)
    //   > 0 → точка на чужой стороне (срезаем)
    // ──────────────────────────────────────────────────────────────────────
    private static List<Vector2> ClipPolygon(List<Vector2> poly, Vector2 origin, Vector2 normal)
    {
        var result = new List<Vector2>(); // сюда кладём уцелевшие вершины
        int n      = poly.Count;          // количество вершин в исходном полигоне

        for (int i = 0; i < n; i++) // перебираем каждое ребро полигона
        {
            Vector2 a  = poly[i];           // начало ребра (текущая вершина)
            Vector2 b  = poly[(i + 1) % n]; // конец ребра (следующая, % n → замкнутый полигон)

            // Расстояние со знаком от каждой вершины до разделяющей прямой.
            // da < 0 → вершина A на нашей стороне (оставляем)
            // da > 0 → вершина A на чужой стороне (срезаем)
            float da = Vector2.Dot(a - origin, normal);
            float db = Vector2.Dot(b - origin, normal);

            if (da <= 0) result.Add(a); // вершина A на нашей стороне — оставляем

            // Если знаки da и db разные — ребро пересекает границу.
            // Находим точку пересечения и добавляем в результат.
            if ((da < 0 && db > 0) || (da > 0 && db < 0))
            {
                // t = da / (da - db): параметр вдоль ребра A→B где пересекается граница.
                // При t=0 — в точке A, при t=1 — в точке B.
                float   t     = da / (da - db);  // коэффициент пересечения (0..1)
                Vector2 cross = a + (b - a) * t;  // точка пересечения = A + направление * t
                result.Add(cross);                // добавляем точку пересечения
            }
        }

        return result; // возвращаем обрезанный полигон
    }
}
```

---

## Algorithms/CellMeshBuilder.cs

```csharp
using System.Collections.Generic; // List<T>
using UnityEngine;                 // Vector2, Vector3, Mesh, Mathf

// ─────────────────────────────────────────────────────────────────────────────
// Строит 3D-меш одной клетки Вороного.
//
// СТРУКТУРА МАССИВА ВЕРШИН (n = кол-во вершин периметра):
//   [0 .. n-1]   — верхний периметр (Y = Perlin Noise * heightNoise)
//   [n]          — центральная точка крыши (тоже с Perlin высотой)
//   [n+1 .. 2n]  — нижний периметр (Y = 0, те же XZ что и верхний)
//
// СТРУКТУРА ТРЕУГОЛЬНИКОВ:
//   Первые  n*3 индексов — "веер" крыши (n треугольников из центра)
//   Следующие n*6 индексов — стенки (n граней × 2 треугольника = n*6)
//   Итого: n*9 индексов
// ─────────────────────────────────────────────────────────────────────────────
public static class CellMeshBuilder
{
    public static Mesh Build(List<Vector2> polygon, float heightNoise = 0f,
                             float offsetX = 0f, float offsetY = 0f)
    {
        int n = polygon.Count; // количество вершин периметра клетки

        // Выделяем массив: n верхних + 1 центр + n нижних = 2n+1 вершин.
        var vertices = new Vector3[2 * n + 1];

        // Локальная функция: высота Y в точке (x, y) через Perlin Noise.
        // 0.05f — масштаб: чем меньше, тем плавнее и крупнее холмы.
        // offsetX/Y — уникальный сдвиг (каждая генерация = своя карта высот).
        float H(float x, float y) =>
            Mathf.PerlinNoise(x * 0.05f + offsetX, y * 0.05f + offsetY) * heightNoise;

        for (int i = 0; i < n; i++) // заполняем верхний и нижний периметры
        {
            // Верхняя вершина [i]: XZ из полигона, Y = высота по Perlin.
            vertices[i] = new Vector3(
                polygon[i].x,                  // X горизонталь
                H(polygon[i].x, polygon[i].y), // Y высота (Perlin)
                polygon[i].y                   // Z глубина (в Unity Y-up: ось глубины = Z)
            );

            // Нижняя вершина [n+1+i]: те же XZ, но Y = 0 → стенки вертикальные.
            vertices[n + 1 + i] = new Vector3(
                polygon[i].x, // X = такой же как у верхней
                0f,           // Y = 0 → низ стенки на уровне земли
                polygon[i].y  // Z = такой же как у верхней
            );
        }

        // Центр крыши: среднее арифметическое всех вершин периметра.
        Vector2 center = Vector2.zero;          // начинаем с нуля
        for (int i = 0; i < n; i++) center += polygon[i]; // суммируем
        center /= n;                            // делим → среднее

        // Центральная вершина [n]: центр полигона с Perlin-высотой.
        vertices[n] = new Vector3(center.x, H(center.x, center.y), center.y);

        // Всего: n треугольников крыши + n*2 треугольников стенок = n*3 треугольника.
        // Каждый треугольник = 3 индекса → массив n*9.
        var triangles = new int[n * 9];

        // ── КРЫША: веер из центра ──────────────────────────────────────────
        // n треугольников: каждый = центр [n] + два соседних ребра периметра.
        // Порядок CCW (против часовой): нормаль смотрит вверх (видна сверху).
        for (int i = 0; i < n; i++)
        {
            triangles[i * 3]     = n;            // вершина [n] — центр
            triangles[i * 3 + 1] = (i + 1) % n; // следующая вершина (% n → замкнуть на 0)
            triangles[i * 3 + 2] = i;            // текущая вершина
        }

        // ── СТЕНКИ: 2 треугольника на каждое ребро периметра ─────────────
        //
        //   t0 ─── t1     Обозначения:
        //   │  ╲   │        t0 = верхняя вершина текущая    [i]
        //   │   ╲  │        t1 = верхняя вершина следующая  [(i+1)%n]
        //   f0 ─── f1       f0 = нижняя вершина под t0      [n+1+i]
        //                   f1 = нижняя вершина под t1      [n+1+(i+1)%n]
        //
        //   Треугольник 1: t0 → t1 → f1  (верхний правый треугольник)
        //   Треугольник 2: t0 → f1 → f0  (нижний левый треугольник)

        int off = n * 3; // смещение: начинаем записывать после индексов крыши

        for (int i = 0; i < n; i++)
        {
            int t0 = i;                    // верхняя вершина текущего ребра
            int t1 = (i + 1) % n;          // верхняя следующая (% n → замкнуть)
            int f0 = n + 1 + i;            // нижняя под t0
            int f1 = n + 1 + (i + 1) % n; // нижняя под t1

            // Треугольник 1 стенки (верхняя половина прямоугольника):
            triangles[off + i * 6 + 0] = t0; // верхний левый
            triangles[off + i * 6 + 1] = t1; // верхний правый
            triangles[off + i * 6 + 2] = f1; // нижний правый

            // Треугольник 2 стенки (нижняя половина прямоугольника):
            triangles[off + i * 6 + 3] = t0; // верхний левый
            triangles[off + i * 6 + 4] = f1; // нижний правый
            triangles[off + i * 6 + 5] = f0; // нижний левый
        }

        var mesh = new Mesh { vertices = vertices, triangles = triangles };
        mesh.RecalculateNormals(); // Unity вычислит нормали для освещения по треугольникам
        mesh.RecalculateBounds();  // обновляет BoundingBox (нужно для frustum culling)
        return mesh;               // возвращаем готовый меш
    }
}
```

---

## Steps/InitializeStep.cs

```csharp
using System.Collections; // IEnumerator
using UnityEngine;         // Object, Random

// Шаг 1 — очищает сцену от предыдущей генерации.
// Без этого шага каждая новая генерация накладывалась бы поверх старой.
public class InitializeStep : IMapGenerationStep
{
    public string Name => "Initialize"; // имя шага (отображается в логе)

    public IEnumerator Execute(GenerationContext ctx)
    {
        // Удаляем дочерние объекты Root с КОНЦА — важно!
        // Если удалять с начала, индексы сдвигаются и часть объектов пропускается.
        for (int i = ctx.Root.childCount - 1; i >= 0; i--)
            Object.Destroy(ctx.Root.GetChild(i).gameObject); // удаляем каждый дочерний объект

        ctx.Cells.Clear(); // очищаем список клеток от прошлой генерации

        // Новые случайные сдвиги Perlin Noise (0..1000).
        // Один общий сдвиг на всю карту → бесшовные стыки между клетками.
        ctx.NoiseOffsetX = Random.Range(0f, 1000f); // горизонтальный сдвиг
        ctx.NoiseOffsetY = Random.Range(0f, 1000f); // вертикальный сдвиг

        yield return null; // отдаём управление Unity на один кадр (обязательно для IEnumerator)
    }
}
```

---

## Steps/VoronoiStep.cs

```csharp
using System.Collections;       // IEnumerator, WaitForSeconds
using System.Collections.Generic; // List<Vector2>
using UnityEngine;               // Vector2, Vector3, Random, LineRenderer

// Шаг 2 — генерирует случайные точки и строит клетки Вороного.
// Если visualizeSteps = true — показывает как контуры появляются одна за другой.
public class VoronoiStep : IMapGenerationStep
{
    public string Name => "Voronoi"; // имя шага для лога

    public IEnumerator Execute(GenerationContext ctx)
    {
        // Отступ от края 5% — точки не попадают на самый край → клетки не обрезаются.
        float margin = ctx.Settings.mapSize * 0.05f;

        var points = new List<Vector2>(); // список случайных точек-"городов"

        for (int i = 0; i < ctx.Settings.pointCount; i++) // создаём pointCount точек
            points.Add(new Vector2(
                Random.Range(margin, ctx.Settings.mapSize - margin), // X в пределах карты
                Random.Range(margin, ctx.Settings.mapSize - margin)  // Z в пределах карты
            ));

        // Вызываем алгоритм Вороного → получаем список клеток с полигонами.
        ctx.Cells = VoronoiGenerator.Build(points, ctx.Settings.mapSize);

        // Один материал на все LineRenderer → экономия памяти.
        var outlineMat = new Material(Shader.Find("Unlit/Color")) { color = Color.white };

        foreach (var cell in ctx.Cells) // для каждой клетки рисуем контур
        {
            DrawCellOutline(cell, ctx.Root, outlineMat); // рисуем линию по вершинам полигона

            // Пауза 0.5 сек между клетками если включена визуализация.
            if (ctx.Settings.visualizeSteps) yield return new WaitForSeconds(0.5f);
        }

        if (!ctx.Settings.visualizeSteps) yield return null; // обязательный yield
    }

    // Рисует контур клетки через LineRenderer.
    private void DrawCellOutline(VoronoiCell cell, Transform parent, Material mat)
    {
        var lr = new GameObject("CellOutline").AddComponent<LineRenderer>(); // новый объект с LineRenderer
        lr.transform.SetParent(parent);              // дочерний к Root
        lr.positionCount  = cell.Polygon.Count + 1;  // точек на 1 больше чем вершин (замкнуть контур)
        lr.startWidth     = 0.2f;                    // толщина в начале
        lr.endWidth       = 0.2f;                    // толщина в конце
        lr.sharedMaterial = mat;                     // общий материал (без копирования)
        lr.useWorldSpace  = true;                    // координаты в мировом пространстве

        for (int i = 0; i < cell.Polygon.Count; i++) // расставляем точки по вершинам
            lr.SetPosition(i, new Vector3(
                cell.Polygon[i].x, // X
                0.5f,              // Y чуть над землёй → линия видна
                cell.Polygon[i].y  // Z
            ));

        // Замыкаем контур: последняя точка = первая.
        lr.SetPosition(cell.Polygon.Count, new Vector3(
            cell.Polygon[0].x, 0.5f, cell.Polygon[0].y
        ));
    }
}
```

---

## Steps/BiomeAssignStep.cs

```csharp
using System.Collections; // IEnumerator
using UnityEngine;         // Mathf, Debug

// Шаг 3 — назначает биом каждой клетке через 2-октавный Perlin Noise.
//
// ПОЧЕМУ PERLIN, А НЕ RANDOM?
//   Random.Range даёт хаотичный результат: соседние клетки = разные биомы.
//   Perlin Noise — плавный: соседние клетки = похожие значения → биомы пятнами.
//
// ПОЧЕМУ 2 ОКТАВЫ?
//   Октава 1 (крупные волны): задаёт форму зон.
//   Октава 2 (мелкие волны, частота ×2.1): добавляет рваные края.
public class BiomeAssignStep : IMapGenerationStep
{
    public string Name => "AssignBiomes";

    public IEnumerator Execute(GenerationContext ctx)
    {
        if (ctx.Biomes == null || ctx.Biomes.Length == 0) // защита от пустого массива
        {
            Debug.LogWarning("[BiomeAssignStep] Нет биомов!"); // предупреждение в консоль
            yield break; // прерываем шаг (аналог return в IEnumerator)
        }

        const float scale = 0.025f; // масштаб шума (меньше = крупнее пятна биомов)

        // Ограничиваем количество используемых биомов настройкой biomeZones.
        // Например: 5 биомов, biomeZones=3 → используем только первые 3.
        int biomeCount = Mathf.Clamp(ctx.Settings.biomeZones, 1, ctx.Biomes.Length);

        for (int i = 0; i < ctx.Cells.Count; i++) // для каждой клетки
        {
            // Координаты клетки в пространстве шума (с масштабом и сдвигом).
            float nx = ctx.Cells[i].Site.x * scale + ctx.NoiseOffsetX * 0.01f;
            float ny = ctx.Cells[i].Site.y * scale + ctx.NoiseOffsetY * 0.01f;

            // 2-октавный Perlin: сумма двух волн нормализованная в 0..1.
            // +5.3 и +7.1 — сдвиги чтобы вторая октава не совпадала с первой.
            float noise = Mathf.Clamp01(
                (Mathf.PerlinNoise(nx, ny) +                                       // октава 1
                 Mathf.PerlinNoise(nx * 2.1f + 5.3f, ny * 2.1f + 7.1f) * 0.5f   // октава 2
                ) / 1.5f                                                            // нормализация
            );

            // Переводим шум (0..1) в индекс биома (0..biomeCount-1).
            ctx.Cells[i].BiomeIndex = Mathf.Clamp(
                Mathf.FloorToInt(noise * biomeCount), // шум → индекс
                0,              // не меньше 0
                biomeCount - 1  // не больше последнего индекса
            );
        }

        yield return null; // обязательный yield
    }
}
```

---

## Steps/TerrainStep.cs

```csharp
using System.Collections; // IEnumerator, WaitForSeconds
using UnityEngine;         // GameObject, Mesh, MeshFilter, MeshRenderer, MeshCollider

// Шаг 4 — строит 3D-меш для каждой клетки и раскрашивает его.
// Меш "вырастает" за 2-5 шагов (требование задания).
// Один материал на все клетки (sharedMaterial = без копий в памяти).
public class TerrainStep : IMapGenerationStep
{
    public string Name => "Terrain";

    public IEnumerator Execute(GenerationContext ctx)
    {
        var parent = new GameObject("Terrain");       // родитель для всех клеток
        parent.transform.SetParent(ctx.Root);          // вешаем на Root

        // Ищем URP шейдер с поддержкой вершинных цветов. ?? = "если null, то следующий".
        Shader sh = Shader.Find("Universal Render Pipeline/Particles/Unlit")
                 ?? Shader.Find("Sprites/Default")
                 ?? Shader.Find("Standard");

        var mat = new Material(sh);                    // создаём материал
        if (mat.HasProperty("_BaseColor")) mat.SetColor("_BaseColor", Color.white); // URP
        if (mat.HasProperty("_Color"))     mat.SetColor("_Color",     Color.white); // Built-in

        foreach (var cell in ctx.Cells) // для каждой клетки Вороного
        {
            BiomeType biome = ctx.Biomes[cell.BiomeIndex]; // биом этой клетки

            var cellObj = new GameObject($"Cell_{biome.biomeName}"); // объект "Cell_Forest"
            cellObj.transform.SetParent(parent.transform);            // дочерний к Terrain

            var mf = cellObj.AddComponent<MeshFilter>();   // хранит геометрию
            var mr = cellObj.AddComponent<MeshRenderer>(); // рисует на экране
            var mc = cellObj.AddComponent<MeshCollider>(); // физика (хождение, raycast)
            mr.sharedMaterial = mat;                       // один материал на всех

            int attempts = Random.Range(2, 6); // 2..5 шагов роста

            for (int a = 0; a < attempts; a++) // цикл "роста" меша
            {
                // Высота нарастает от 1/attempts до 1.0 (полная высота на последнем шаге).
                float height = biome.heightVariation * (float)(a + 1) / attempts;

                Mesh mesh = CellMeshBuilder.Build(
                    cell.Polygon,     // 2D-контур клетки
                    height,           // текущая высота
                    ctx.NoiseOffsetX, // X-сдвиг Perlin
                    ctx.NoiseOffsetY  // Y-сдвиг Perlin
                );

                ColorMesh(mesh, biome.groundColor, ctx.NoiseOffsetX, ctx.NoiseOffsetY); // раскрасить

                mf.sharedMesh = mc.sharedMesh = mesh; // применить к рендеру и коллайдеру

                if (ctx.Settings.visualizeSteps) yield return new WaitForSeconds(0.2f); // пауза роста
            }
        }

        if (!ctx.Settings.visualizeSteps) yield return null; // обязательный yield
    }

    // Раскрашивает вершины меша: базовый цвет биома + Perlin-вариация яркости ±15%.
    // Clamp01 не даёт цвету выйти за 0..1.
    private void ColorMesh(Mesh mesh, Color baseColor, float ox, float oy)
    {
        Vector3[] verts  = mesh.vertices;           // все вершины меша
        Color[]   colors = new Color[verts.Length]; // массив цветов (один на вершину)

        for (int v = 0; v < verts.Length; v++) // для каждой вершины
        {
            // b = 0.82..1.12: яркость от -18% до +12% от базового цвета биома.
            float b = 0.82f + Mathf.PerlinNoise(
                verts[v].x * 0.08f + ox * 0.1f, // X в пространстве шума
                verts[v].z * 0.08f + oy * 0.1f  // Z в пространстве шума
            ) * 0.30f;

            colors[v] = new Color(
                Mathf.Clamp01(baseColor.r * b), // красный канал × яркость
                Mathf.Clamp01(baseColor.g * b), // зелёный канал × яркость
                Mathf.Clamp01(baseColor.b * b), // синий канал × яркость
                1f                              // альфа = непрозрачный
            );
        }

        mesh.colors = colors; // применяем цвета к мешу
    }
}
```

---

## MapGenerator.cs

```csharp
using System.Collections;       // IEnumerator, StartCoroutine
using System.Collections.Generic; // List<IMapGenerationStep>
using UnityEngine;               // MonoBehaviour, Debug

// Оркестратор: запускает шаги генерации один за другим через корутины.
// Не знает что делает каждый шаг — только вызывает Execute(ctx).
// Принцип D (DIP): зависит от IMapGenerationStep (интерфейс), не от конкретных классов.
public class MapGenerator : MonoBehaviour
{
    private GenerationContext        _context; // общий "рюкзак" данных
    private List<IMapGenerationStep> _steps = new List<IMapGenerationStep>(); // список шагов

    public void SetContext(GenerationContext ctx)         => _context = ctx;   // передать контекст
    public void SetSteps(List<IMapGenerationStep> steps) => _steps   = steps; // передать шаги

    public IEnumerator Generate()
    {
        if (_context == null) // защита от запуска без контекста
        {
            Debug.LogError("[MapGenerator] Контекст не установлен."); // ошибка в консоль
            yield break; // прерываем корутину
        }

        foreach (var step in _steps) // выполняем шаги по порядку
        {
            Debug.Log($"[MapGenerator] Шаг: {step.Name}"); // лог текущего шага
            yield return StartCoroutine(step.Execute(_context)); // ждём завершения шага
        }
    }
}
```

---

## GameLauncher.cs

```csharp
using System.Collections.Generic; // List<IMapGenerationStep>
using UnityEngine;                 // MonoBehaviour, RequireComponent, SerializeField

// Composition Root — единственное место где создаются конкретные классы.
// Принцип D (DIP): соединяет интерфейсы с реализациями.
// [RequireComponent] — Unity автоматически добавит MapGenerator если его нет.
[RequireComponent(typeof(MapGenerator))]
public class GameLauncher : MonoBehaviour
{
    [SerializeField] private BiomeType[]  biomes;               // 5 биомов из инспектора
    [SerializeField] private GameObject   playerPrefab;          // префаб игрока (необязательно)
    [SerializeField] private bool         autoGenerateOnStart = false; // старт при Play?

    private MapGenerator        _mapGenerator;       // оркестратор шагов
    private ISettingsRepository _settingsRepository; // хранилище настроек
    private GenerationContext   _context;            // общий "рюкзак" данных

    private void Awake()
    {
        _mapGenerator       = GetComponent<MapGenerator>();   // берём компонент с этого же объекта
        _settingsRepository = new JsonSettingsRepository();   // создаём реализацию хранилища

        // Создаём контекст и заполняем начальными данными.
        _context = new GenerationContext
        {
            Settings     = _settingsRepository.Load(), // загружаем настройки из JSON
            Biomes       = biomes,                     // биомы из инспектора
            Root         = transform,                  // этот объект = родитель генерации
            PlayerPrefab = playerPrefab,               // префаб игрока (или null)
            NoiseOffsetX = Random.value * 1000f,       // случайный сдвиг X для Perlin
            NoiseOffsetY = Random.value * 1000f,       // случайный сдвиг Y для Perlin
        };

        // Передаём контекст оркестратору.
        _mapGenerator.SetContext(_context);

        // Передаём список шагов (порядок важен!).
        _mapGenerator.SetSteps(new List<IMapGenerationStep>
        {
            new InitializeStep(),   // 1. очистка сцены
            new VoronoiStep(),      // 2. генерация точек и клеток
            new BiomeAssignStep(),  // 3. назначение биомов
            new TerrainStep(),      // 4. построение мешей
            new WallsStep(),        // 5. стены по краям карты
            new ObjectsStep(),      // 6. объекты в клетках (деревья, камни)
            new PlayerSpawnStep(),  // 7. спавн игрока в стартовом биоме
        });

        // Автозапуск при старте (удобно для тестирования).
        if (autoGenerateOnStart) StartCoroutine(_mapGenerator.Generate());
    }

    // Вызывается из UI (MapSettingsPanel) когда игрок нажимает "Генерировать".
    public void StartWithSettings(MapSettings settings)
    {
        _settingsRepository.Save(settings); // сохраняем новые настройки в JSON
        _context.Settings = settings;       // обновляем контекст
        StartCoroutine(_mapGenerator.Generate()); // запускаем генерацию
    }
}
```
