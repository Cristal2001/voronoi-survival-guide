# Все скрипты проекта — полный код

Копируй ТОЧНО. Не меняй имена классов, методов, пространств имён.

---

## Data/MapSettings.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/Data/MapSettings.cs
using System;
using UnityEngine;

[Serializable]
public class MapSettings
{
    public int   pointCount              = 50;
    public int   mapSize                 = 100;
    public int   biomeZones              = 5;
    public int   objectPlacementAttempts = 10;
    public bool  visualizeSteps          = false;
    public float maxHealth               = 100f;
    public float maxHunger               = 100f;
    public float maxThirst               = 100f;
    public float hpLossOnHunger          = 2f;
    public float hpLossOnThirst          = 3f;
}
```

---

## Data/VoronoiCell.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/Data/VoronoiCell.cs
using System.Collections.Generic;
using UnityEngine;

public class VoronoiCell
{
    public Vector2       Site       { get; set; }
    public List<Vector2> Polygon    { get; set; } = new List<Vector2>();
    public int           BiomeIndex { get; set; }
}
```

---

## Data/BiomeType.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/Data/BiomeType.cs
using UnityEngine;

[CreateAssetMenu(menuName = "Voronoi/Biome Type")]
public class BiomeType : ScriptableObject
{
    public string       biomeName;
    public Color        groundColor     = Color.green;
    public float        heightVariation = 1f;
    public bool         isStartBiome    = false;
    public bool         hasWater        = false;
    public GameObject[] objectPrefabs;
}
```

---

## Interfaces/IMapGenerationStep.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/Interfaces/IMapGenerationStep.cs
using System.Collections;

public interface IMapGenerationStep
{
    string      Name    { get; }
    IEnumerator Execute(GenerationContext ctx);
}
```

---

## Interfaces/ISettingsRepository.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/Interfaces/ISettingsRepository.cs
public interface ISettingsRepository
{
    MapSettings Load();
    void        Save(MapSettings settings);
}
```

---

## GenerationContext.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/GenerationContext.cs
using System.Collections.Generic;
using UnityEngine;

public class GenerationContext
{
    public MapSettings       Settings     { get; set; }
    public BiomeType[]       Biomes       { get; set; }
    public Transform         Root         { get; set; }
    public List<VoronoiCell> Cells        { get; set; } = new List<VoronoiCell>();
    public float             NoiseOffsetX { get; set; }
    public float             NoiseOffsetY { get; set; }
    public GameObject        PlayerPrefab { get; set; }
}
```

---

## Persistence/JsonSettingsRepository.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/Persistence/JsonSettingsRepository.cs
using System.IO;
using UnityEngine;

public class JsonSettingsRepository : ISettingsRepository
{
    private string FilePath =>
        System.IO.Path.Combine(Application.streamingAssetsPath, "mapSettings.json");

    public MapSettings Load()
    {
        if (File.Exists(FilePath))
        {
            try
            {
                string json = File.ReadAllText(FilePath);
                return JsonUtility.FromJson<MapSettings>(json);
            }
            catch (System.Exception e)
            {
                Debug.LogWarning($"[JsonSettingsRepository] Ошибка загрузки: {e.Message}");
            }
        }
        return new MapSettings();
    }

    public void Save(MapSettings settings)
    {
        string json = JsonUtility.ToJson(settings, true);
        File.WriteAllText(FilePath, json);
        Debug.Log($"[JsonSettingsRepository] Сохранено в {FilePath}");
    }
}
```

---

## Algorithms/VoronoiGenerator.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/Algorithms/VoronoiGenerator.cs
using System.Collections.Generic;
using UnityEngine;

public static class VoronoiGenerator
{
    /// <summary>
    /// Генерирует клетки Вороного методом пересечения полуплоскостей.
    /// </summary>
    public static List<VoronoiCell> Generate(int count, float size, int seed = 0)
    {
        if (seed == 0)
            Random.InitState(System.Environment.TickCount);
        else
            Random.InitState(seed);

        float pad = size * 0.05f; // отступ 5% от края
        var sites = new List<Vector2>(count);
        for (int i = 0; i < count; i++)
            sites.Add(new Vector2(
                Random.Range(pad, size - pad),
                Random.Range(pad, size - pad)
            ));

        var cells = new List<VoronoiCell>(count);

        for (int i = 0; i < count; i++)
        {
            var poly = ClipToRect(size);

            for (int j = 0; j < count; j++)
            {
                if (i == j) continue;
                poly = ClipByBisector(poly, sites[i], sites[j]);
                if (poly.Count < 3) break;
            }

            if (poly.Count >= 3)
            {
                cells.Add(new VoronoiCell
                {
                    Site    = sites[i],
                    Polygon = poly
                });
            }
        }

        return cells;
    }

    // Начальный прямоугольник карты (обход против часовой стрелки)
    private static List<Vector2> ClipToRect(float s) =>
        new List<Vector2>
        {
            new Vector2(0, 0),
            new Vector2(s, 0),
            new Vector2(s, s),
            new Vector2(0, s)
        };

    // Алгоритм Сазерленда-Ходжмана: отрезаем полигон по биссектрисе A-B
    private static List<Vector2> ClipByBisector(List<Vector2> poly, Vector2 a, Vector2 b)
    {
        Vector2 mid    = (a + b) * 0.5f;       // середина отрезка A-B
        Vector2 normal = (b - a).normalized;    // нормаль (направление от A к B)

        var result = new List<Vector2>();
        int cnt    = poly.Count;

        for (int i = 0; i < cnt; i++)
        {
            Vector2 cur = poly[i];
            Vector2 nxt = poly[(i + 1) % cnt];

            // Точка "внутри" (на стороне A) если dot <= 0
            bool curInside = Vector2.Dot(cur - mid, normal) <= 0;
            bool nxtInside = Vector2.Dot(nxt - mid, normal) <= 0;

            if (curInside)
                result.Add(cur);

            // Если ребро пересекает границу — добавить точку пересечения
            if (curInside != nxtInside)
            {
                Vector2 dir = nxt - cur;
                float   denom = Vector2.Dot(dir, normal);
                if (Mathf.Abs(denom) > 1e-6f)
                {
                    float t = Vector2.Dot(mid - cur, normal) / denom;
                    result.Add(cur + dir * t);
                }
            }
        }

        return result;
    }
}
```

---

## Algorithms/CellMeshBuilder.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/Algorithms/CellMeshBuilder.cs
using System.Collections.Generic;
using UnityEngine;

public static class CellMeshBuilder
{
    /// <summary>
    /// Строит Unity Mesh из 2D многоугольника с Perlin-высотами.
    /// Верхние вершины: 0..n-1, Центр: n, Нижние вершины: n+1..2n
    /// </summary>
    public static Mesh Build(List<Vector2> polygon, float heightVariation = 0f,
                              float offsetX = 0f, float offsetY = 0f)
    {
        int n = polygon.Count;
        if (n < 3) return new Mesh();

        // Вершины: n верхних + 1 центральная + n нижних = 2n+1
        var vertices = new Vector3[2 * n + 1];

        // Функция высоты через Perlin
        float H(float x, float y) =>
            Mathf.PerlinNoise(x * 0.05f + offsetX, y * 0.05f + offsetY) * heightVariation;

        // Верхние вершины (с высотой)
        for (int i = 0; i < n; i++)
        {
            float x = polygon[i].x;
            float y = polygon[i].y;
            vertices[i] = new Vector3(x, H(x, y), y);
        }

        // Нижние вершины (Y = 0, для боковых стенок)
        for (int i = 0; i < n; i++)
        {
            vertices[n + 1 + i] = new Vector3(polygon[i].x, 0f, polygon[i].y);
        }

        // Центральная вершина (среднее верхних)
        Vector2 center = Vector2.zero;
        for (int i = 0; i < n; i++) center += polygon[i];
        center /= n;
        vertices[n] = new Vector3(center.x, H(center.x, center.y), center.y);

        // Треугольники: n веерных (верх) + n*2 боковых
        var triangles = new int[n * 3 + n * 6];

        // Верхние веерные треугольники: центр → вершина[i+1] → вершина[i]
        for (int i = 0; i < n; i++)
        {
            triangles[i * 3 + 0] = n;              // центр
            triangles[i * 3 + 1] = (i + 1) % n;   // следующая
            triangles[i * 3 + 2] = i;              // текущая
        }

        // Боковые треугольники (два на каждое ребро)
        int offset = n * 3;
        for (int i = 0; i < n; i++)
        {
            int t0 = i;                 // верхняя текущая
            int t1 = (i + 1) % n;      // верхняя следующая
            int f0 = n + 1 + i;        // нижняя текущая
            int f1 = n + 1 + (i + 1) % n; // нижняя следующая

            triangles[offset + i * 6 + 0] = t0;
            triangles[offset + i * 6 + 1] = t1;
            triangles[offset + i * 6 + 2] = f1;

            triangles[offset + i * 6 + 3] = t0;
            triangles[offset + i * 6 + 4] = f1;
            triangles[offset + i * 6 + 5] = f0;
        }

        var mesh = new Mesh
        {
            vertices  = vertices,
            triangles = triangles
        };
        mesh.RecalculateNormals();
        mesh.RecalculateBounds();
        return mesh;
    }
}
```

---

## MapGenerator.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/MapGenerator.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class MapGenerator : MonoBehaviour
{
    private GenerationContext        _context;
    private List<IMapGenerationStep> _steps = new List<IMapGenerationStep>();

    public void SetContext(GenerationContext ctx)
    {
        _context = ctx;
    }

    public void SetSteps(List<IMapGenerationStep> steps)
    {
        _steps = steps;
    }

    public IEnumerator Generate()
    {
        if (_context == null)
        {
            Debug.LogError("[MapGenerator] Контекст не установлен! Вызови SetContext() сначала.");
            yield break;
        }

        Debug.Log("[MapGenerator] Начало генерации...");

        foreach (var step in _steps)
        {
            Debug.Log($"[MapGenerator] Шаг: {step.Name}");
            yield return StartCoroutine(step.Execute(_context));
        }

        Debug.Log("[MapGenerator] Генерация завершена!");
    }
}
```

---

## GameLauncher.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/GameLauncher.cs
using System.Collections.Generic;
using UnityEngine;

[RequireComponent(typeof(MapGenerator))]
public class GameLauncher : MonoBehaviour
{
    [SerializeField] private BiomeType[]  biomes;
    [SerializeField] private GameObject   playerPrefab;
    [SerializeField] private bool         autoGenerateOnStart = false;

    private MapGenerator        _mapGenerator;
    private ISettingsRepository _settingsRepository;
    private GenerationContext   _context;

    private void Awake()
    {
        _mapGenerator       = GetComponent<MapGenerator>();
        _settingsRepository = new JsonSettingsRepository();

        _context = new GenerationContext
        {
            Settings     = _settingsRepository.Load(),
            Biomes       = biomes,
            Root         = transform,
            PlayerPrefab = playerPrefab,
            NoiseOffsetX = Random.value * 1000f,
            NoiseOffsetY = Random.value * 1000f,
        };

        _mapGenerator.SetContext(_context);
        _mapGenerator.SetSteps(CreateSteps());

        if (autoGenerateOnStart)
            StartCoroutine(_mapGenerator.Generate());
    }

    private List<IMapGenerationStep> CreateSteps()
    {
        return new List<IMapGenerationStep>
        {
            new InitializeStep(),
            new VoronoiStep(),
            new BiomeAssignStep(),
            new TerrainStep(),
            new WallsStep(),
            new ObjectsStep(),
            new PlayerSpawnStep(),
            // Добавь сюда: new AnimalsStep(),
        };
    }

    /// <summary>
    /// Вызывается из UI когда игрок нажал "Играть" с настройками.
    /// </summary>
    public void StartWithSettings(MapSettings settings)
    {
        _settingsRepository.Save(settings);
        _context.Settings    = settings;
        _context.NoiseOffsetX = Random.value * 1000f;
        _context.NoiseOffsetY = Random.value * 1000f;
        StartCoroutine(_mapGenerator.Generate());
    }

    /// <summary>
    /// Вызывается из UI для повторной генерации с теми же настройками.
    /// </summary>
    public void Regenerate()
    {
        _context.NoiseOffsetX = Random.value * 1000f;
        _context.NoiseOffsetY = Random.value * 1000f;
        StartCoroutine(_mapGenerator.Generate());
    }
}
```

---

## Steps/InitializeStep.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/Steps/InitializeStep.cs
using System.Collections;
using UnityEngine;

public class InitializeStep : IMapGenerationStep
{
    public string Name => "Initialize";

    public IEnumerator Execute(GenerationContext ctx)
    {
        Debug.Log("[InitializeStep] Очистка сцены...");

        // Уничтожаем все дочерние объекты с известными именами
        string[] toDestroy = { "Terrain", "Walls", "Objects", "Player" };

        for (int i = ctx.Root.childCount - 1; i >= 0; i--)
        {
            Transform child = ctx.Root.GetChild(i);
            foreach (string name in toDestroy)
            {
                if (child.name == name)
                {
                    Object.Destroy(child.gameObject);
                    break;
                }
            }
        }

        // Также уничтожаем объекты с тегом Player (на случай если вне Root)
        GameObject[] players = GameObject.FindGameObjectsWithTag("Player");
        foreach (var p in players)
            Object.Destroy(p);

        // Очищаем список клеток
        ctx.Cells.Clear();

        yield return null; // один кадр паузы
    }
}
```

---

## Steps/VoronoiStep.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/Steps/VoronoiStep.cs
using System.Collections;
using UnityEngine;

public class VoronoiStep : IMapGenerationStep
{
    public string Name => "Voronoi";

    public IEnumerator Execute(GenerationContext ctx)
    {
        Debug.Log("[VoronoiStep] Генерация клеток Вороного...");

        ctx.Cells = VoronoiGenerator.Generate(
            ctx.Settings.pointCount,
            ctx.Settings.mapSize
        );

        Debug.Log($"[VoronoiStep] Создано {ctx.Cells.Count} клеток");

        // Визуализация: рисуем контуры клеток
        if (ctx.Settings.visualizeSteps)
        {
            GameObject outlineRoot = new GameObject("VoronoiOutlines");
            outlineRoot.transform.parent = ctx.Root;

            foreach (var cell in ctx.Cells)
            {
                DrawCellOutline(cell, outlineRoot.transform);
                yield return new WaitForSeconds(0.05f); // задержка для визуализации
            }
        }

        yield return null;
    }

    private void DrawCellOutline(VoronoiCell cell, Transform parent)
    {
        if (cell.Polygon.Count < 3) return;

        var go = new GameObject($"Outline_{cell.Site}");
        go.transform.parent = parent;

        var lr = go.AddComponent<LineRenderer>();
        lr.startWidth    = 0.2f;
        lr.endWidth      = 0.2f;
        lr.loop          = true;
        lr.positionCount = cell.Polygon.Count;
        lr.useWorldSpace = true;

        // Материал для LineRenderer
        lr.material = new Material(Shader.Find("Sprites/Default"));
        lr.startColor = Color.white;
        lr.endColor   = Color.white;

        for (int i = 0; i < cell.Polygon.Count; i++)
        {
            var p = cell.Polygon[i];
            lr.SetPosition(i, new Vector3(p.x, 0.1f, p.y));
        }
    }
}
```

---

## Steps/BiomeAssignStep.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/Steps/BiomeAssignStep.cs
using System.Collections;
using UnityEngine;

public class BiomeAssignStep : IMapGenerationStep
{
    public string Name => "BiomeAssign";

    public IEnumerator Execute(GenerationContext ctx)
    {
        Debug.Log("[BiomeAssignStep] Назначение биомов...");

        int biomeCount = Mathf.Clamp(ctx.Settings.biomeZones, 1, ctx.Biomes.Length);

        foreach (var cell in ctx.Cells)
        {
            // Perlin noise для плавного распределения биомов
            float nx    = cell.Site.x / ctx.Settings.mapSize;
            float ny    = cell.Site.y / ctx.Settings.mapSize;
            float noise = Mathf.PerlinNoise(
                nx * 3f + ctx.NoiseOffsetX * 0.01f,
                ny * 3f + ctx.NoiseOffsetY * 0.01f
            );

            // noise: 0..1 → biomeIndex: 0..biomeCount-1
            int biomeIndex = Mathf.Clamp(
                Mathf.FloorToInt(noise * biomeCount),
                0,
                biomeCount - 1
            );

            cell.BiomeIndex = biomeIndex;
        }

        if (ctx.Settings.visualizeSteps)
            yield return new WaitForSeconds(0.3f);

        yield return null;
    }
}
```

---

## Steps/TerrainStep.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/Steps/TerrainStep.cs
using System.Collections;
using UnityEngine;

public class TerrainStep : IMapGenerationStep
{
    public string Name => "Terrain";

    public IEnumerator Execute(GenerationContext ctx)
    {
        Debug.Log("[TerrainStep] Построение рельефа...");

        // Родительский объект для всего террейна
        GameObject terrainRoot = new GameObject("Terrain");
        terrainRoot.transform.parent = ctx.Root;

        // Один общий материал для всего террейна
        var mat = new Material(Shader.Find("Universal Render Pipeline/Lit"));

        for (int i = 0; i < ctx.Cells.Count; i++)
        {
            var cell  = ctx.Cells[i];
            var biome = ctx.Biomes[cell.BiomeIndex];

            // Строим меш
            Mesh mesh = CellMeshBuilder.Build(
                cell.Polygon,
                biome.heightVariation,
                ctx.NoiseOffsetX,
                ctx.NoiseOffsetY
            );

            // Создаём GameObject
            var go = new GameObject($"Cell_{i}");
            go.transform.parent = terrainRoot.transform;

            // MeshFilter
            var mf  = go.AddComponent<MeshFilter>();
            mf.mesh = mesh;

            // MeshRenderer
            var mr       = go.AddComponent<MeshRenderer>();
            var cellMat  = new Material(mat);

            // Вариация цвета ±15%
            Color baseColor = biome.groundColor;
            float variation = Random.Range(-0.15f, 0.15f);
            cellMat.color = new Color(
                Mathf.Clamp01(baseColor.r + variation),
                Mathf.Clamp01(baseColor.g + variation),
                Mathf.Clamp01(baseColor.b + variation)
            );
            mr.material = cellMat;

            // MeshCollider
            var mc  = go.AddComponent<MeshCollider>();
            mc.sharedMesh = mesh;

            if (ctx.Settings.visualizeSteps && i % 5 == 0)
                yield return new WaitForSeconds(0.1f);
        }

        Object.Destroy(mat); // Освобождаем шаблонный материал
        yield return null;
    }
}
```

---

## Steps/WallsStep.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/Steps/WallsStep.cs
using System.Collections;
using UnityEngine;

public class WallsStep : IMapGenerationStep
{
    public string Name => "Walls";

    public IEnumerator Execute(GenerationContext ctx)
    {
        Debug.Log("[WallsStep] Установка стен...");

        float size = ctx.Settings.mapSize;

        GameObject wallsRoot = new GameObject("Walls");
        wallsRoot.transform.parent = ctx.Root;

        CreateWall(wallsRoot.transform, "Wall_North",
            new Vector3(size / 2f, 5f, size),
            new Vector3(size, 10f, 1f));

        CreateWall(wallsRoot.transform, "Wall_South",
            new Vector3(size / 2f, 5f, 0f),
            new Vector3(size, 10f, 1f));

        CreateWall(wallsRoot.transform, "Wall_East",
            new Vector3(size, 5f, size / 2f),
            new Vector3(1f, 10f, size));

        CreateWall(wallsRoot.transform, "Wall_West",
            new Vector3(0f, 5f, size / 2f),
            new Vector3(1f, 10f, size));

        yield return null;
    }

    private void CreateWall(Transform parent, string wallName, Vector3 position, Vector3 scale)
    {
        var wall = GameObject.CreatePrimitive(PrimitiveType.Cube);
        wall.name                     = wallName;
        wall.transform.parent         = parent;
        wall.transform.position       = position;
        wall.transform.localScale     = scale;

        // Невидимые стены (можно сделать видимыми убрав эту строку)
        var renderer = wall.GetComponent<MeshRenderer>();
        if (renderer != null)
            renderer.enabled = false;
    }
}
```

---

## Steps/ObjectsStep.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/Steps/ObjectsStep.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ObjectsStep : IMapGenerationStep
{
    public string Name => "Objects";

    public IEnumerator Execute(GenerationContext ctx)
    {
        Debug.Log("[ObjectsStep] Размещение объектов...");

        GameObject objectsRoot = new GameObject("Objects");
        objectsRoot.transform.parent = ctx.Root;

        foreach (var cell in ctx.Cells)
        {
            var biome = ctx.Biomes[cell.BiomeIndex];

            // Пропускаем если нет префабов
            if (biome.objectPrefabs == null || biome.objectPrefabs.Length == 0)
                continue;

            for (int attempt = 0; attempt < ctx.Settings.objectPlacementAttempts; attempt++)
            {
                // Случайная точка в bounding box клетки
                Vector2 point = GetRandomPointInBounds(cell.Polygon);

                // Проверяем что точка внутри полигона
                if (!IsPointInPolygon(point, cell.Polygon))
                    continue;

                // Выбираем случайный префаб из биома
                var prefab = biome.objectPrefabs[Random.Range(0, biome.objectPrefabs.Length)];
                if (prefab == null) continue;

                // Инстанциируем
                Vector3 worldPos = new Vector3(point.x, 0f, point.y);
                var obj          = Object.Instantiate(prefab, worldPos, Quaternion.identity);
                obj.transform.parent = objectsRoot.transform;

                // Небольшое случайное вращение
                obj.transform.rotation = Quaternion.Euler(0f, Random.Range(0f, 360f), 0f);
            }

            if (ctx.Settings.visualizeSteps)
                yield return new WaitForSeconds(0.05f);
        }

        yield return null;
    }

    // Случайная точка в bounding box полигона
    private Vector2 GetRandomPointInBounds(List<Vector2> polygon)
    {
        float minX = float.MaxValue, maxX = float.MinValue;
        float minY = float.MaxValue, maxY = float.MinValue;

        foreach (var p in polygon)
        {
            if (p.x < minX) minX = p.x;
            if (p.x > maxX) maxX = p.x;
            if (p.y < minY) minY = p.y;
            if (p.y > maxY) maxY = p.y;
        }

        return new Vector2(Random.Range(minX, maxX), Random.Range(minY, maxY));
    }

    // Ray casting алгоритм: нечётное число пересечений = внутри полигона
    private bool IsPointInPolygon(Vector2 point, List<Vector2> polygon)
    {
        int n         = polygon.Count;
        bool isInside = false;

        for (int i = 0, j = n - 1; i < n; j = i++)
        {
            Vector2 vi = polygon[i];
            Vector2 vj = polygon[j];

            if (((vi.y > point.y) != (vj.y > point.y)) &&
                (point.x < (vj.x - vi.x) * (point.y - vi.y) / (vj.y - vi.y) + vi.x))
            {
                isInside = !isInside;
            }
        }

        return isInside;
    }
}
```

---

## Steps/PlayerSpawnStep.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/Steps/PlayerSpawnStep.cs
using System.Collections;
using UnityEngine;

public class PlayerSpawnStep : IMapGenerationStep
{
    public string Name => "PlayerSpawn";

    public IEnumerator Execute(GenerationContext ctx)
    {
        Debug.Log("[PlayerSpawnStep] Спавн игрока...");

        // Ищем клетку с стартовым биомом
        VoronoiCell startCell = null;
        foreach (var cell in ctx.Cells)
        {
            if (cell.BiomeIndex < ctx.Biomes.Length &&
                ctx.Biomes[cell.BiomeIndex].isStartBiome)
            {
                startCell = cell;
                break;
            }
        }

        // Если стартового биома нет — берём первую клетку
        if (startCell == null && ctx.Cells.Count > 0)
        {
            Debug.LogWarning("[PlayerSpawnStep] Стартовый биом не найден, использую первую клетку");
            startCell = ctx.Cells[0];
        }

        if (startCell == null)
        {
            Debug.LogError("[PlayerSpawnStep] Нет клеток для спавна!");
            yield break;
        }

        // Центр полигона
        Vector2 center2D = Vector2.zero;
        foreach (var p in startCell.Polygon) center2D += p;
        center2D /= startCell.Polygon.Count;

        Vector3 spawnPosition = new Vector3(center2D.x, 2f, center2D.y);

        GameObject player;
        if (ctx.PlayerPrefab != null)
        {
            player = Object.Instantiate(ctx.PlayerPrefab, spawnPosition, Quaternion.identity);
        }
        else
        {
            // Временная капсула для тестирования
            player           = GameObject.CreatePrimitive(PrimitiveType.Capsule);
            player.name      = "Player";
            player.transform.position = spawnPosition;
            player.tag       = "Player";

            // Добавляем физику
            var rb           = player.AddComponent<Rigidbody>();
            rb.freezeRotation = true;
        }

        player.name             = "Player";
        player.tag              = "Player";
        player.transform.parent = ctx.Root;

        Debug.Log($"[PlayerSpawnStep] Игрок заспавнен в {spawnPosition}");

        yield return null;
    }
}
```

---

## Как добавить AnimalsStep (пример расширения)

Создай файл `Steps/AnimalsStep.cs`:

```csharp
// Assets/Scripts/VoronoiMapGenerator/Steps/AnimalsStep.cs
using System.Collections;
using UnityEngine;

public class AnimalsStep : IMapGenerationStep
{
    public string Name => "Animals";

    public IEnumerator Execute(GenerationContext ctx)
    {
        Debug.Log("[AnimalsStep] Спавн животных...");

        GameObject animalsRoot = new GameObject("Animals");
        animalsRoot.transform.parent = ctx.Root;

        foreach (var cell in ctx.Cells)
        {
            var biome = ctx.Biomes[cell.BiomeIndex];

            // В стартовом биоме нет агрессивных животных
            bool isStartBiome = biome.isStartBiome;

            // Вычисляем центр клетки
            Vector2 center = Vector2.zero;
            foreach (var p in cell.Polygon) center += p;
            center /= cell.Polygon.Count;

            Vector3 pos = new Vector3(center.x, 1f, center.y);

            // Спавним нейтральных в каждой 3-й клетке
            if (Random.value < 0.3f)
            {
                SpawnNeutralAnimal(pos, animalsRoot.transform);
            }

            // Агрессивных только вне стартового биома
            if (!isStartBiome && Random.value < 0.1f)
            {
                SpawnAggressiveAnimal(pos, animalsRoot.transform);
            }

            // Рыб только в водных биомах
            if (biome.hasWater && Random.value < 0.5f)
            {
                SpawnFishAnimal(pos, animalsRoot.transform);
            }
        }

        yield return null;
    }

    private void SpawnNeutralAnimal(Vector3 pos, Transform parent)
    {
        // TODO: заменить на реальные префабы
        var go           = GameObject.CreatePrimitive(PrimitiveType.Sphere);
        go.name          = "Rabbit";
        go.transform.parent   = parent;
        go.transform.position = pos;
        go.transform.localScale = Vector3.one * 0.5f;
        go.GetComponent<MeshRenderer>().material.color = Color.white;

        var ai = go.AddComponent<AnimalAI>();
        ai.Initialize(AnimalType.Neutral, 10f, 0f, 2f, 1.5f, 2f);
    }

    private void SpawnAggressiveAnimal(Vector3 pos, Transform parent)
    {
        var go           = GameObject.CreatePrimitive(PrimitiveType.Sphere);
        go.name          = "Wolf";
        go.transform.parent   = parent;
        go.transform.position = pos;
        go.GetComponent<MeshRenderer>().material.color = Color.gray;

        var ai = go.AddComponent<AnimalAI>();
        ai.Initialize(AnimalType.Aggressive, 40f, 2f, 8f, 2.5f, 1f);
    }

    private void SpawnFishAnimal(Vector3 pos, Transform parent)
    {
        var go           = GameObject.CreatePrimitive(PrimitiveType.Sphere);
        go.name          = "SmallFish";
        go.transform.parent   = parent;
        go.transform.position = pos;
        go.transform.localScale = Vector3.one * 0.3f;
        go.GetComponent<MeshRenderer>().material.color = Color.cyan;

        var ai = go.AddComponent<AnimalAI>();
        ai.Initialize(AnimalType.Fish, 5f, 0f, 0f, 0f, 0f);
    }
}
```

В `GameLauncher.cs` в метод `CreateSteps()` добавь:
```csharp
new AnimalsStep(),
```

Больше ничего менять не нужно. Это и есть Open/Closed принцип.
