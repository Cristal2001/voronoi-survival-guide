# Модуль Г — Генерация карты и механики выживания (24 балла, 3 часа)

Самый сложный модуль. Планируй его первым после Модуля А.

---

## Объекты биомов (ДОЛЖНЫ быть разными!)

Это требование задания: объекты в разных биомах должны отличаться.

| Биом | Объекты | Префабы |
|------|---------|---------|
| Grass (старт) | Кусты, цветы, трава | BushPrefab, FlowerPrefab, GrassTuftPrefab |
| Forest | Деревья, пни, папоротники | TreePrefab, StumpPrefab, FernPrefab |
| Rocks | Камни, валуны, кристаллы | SmallRockPrefab, BoulderPrefab, CrystalPrefab |
| Water | Кувшинки, тростник | LilyPadPrefab, ReedPrefab |
| Desert | Кактусы, сухие деревья | CactusPrefab, DeadTreePrefab |

Если нет реальных моделей — используй примитивы с разными Scale и Color.

**Как быстро сделать примитивные префабы:**
1. Создай в сцене Cube/Sphere/Cylinder
2. Настрой Scale и Color материала
3. Перетащи из Hierarchy в папку Prefabs/
4. Удали из сцены
5. Назначь в BiomeType.objectPrefabs[]

---

## Таблица животных — Рыбы (только в водных биомах)

Ловятся удочкой.

| Животное | HP | Добыча |
|----------|----|--------|
| Small Fish | 5 | FishMeat x1 |
| Trout | 8 | FishMeat x1-2, FishScales x1 |
| Salmon | 12 | FishMeat x2-3, FishScales x2 |

---

## Таблица животных — Нейтральные (убегают при атаке на 10м)

| Животное | HP | Броня | Урон | Дальность | Перезарядка | Добыча |
|----------|----|-------|------|-----------|-------------|--------|
| Rabbit | 10 | 0 | 2 | 1.5m | 2s | RabbitMeat x1-2, RabbitHide x1 |
| Butterfly | 1 | 0 | 0 | — | — | ButterflyWings x1 (30%) |
| Beehive | 5 | 0 | 1 | 2m | 1s | Honey x1-2, Wax x1 |
| Deer | 30 | 1 | 5 | 2m | 1.5s | DeerMeat x3-5, DeerHide x2, Antler x1 |
| Bison | 60 | 3 | 12 | 2.5m | 2s | BisonMeat x6-8, BisonHide x4, Horn x1-2 |
| Horse | 45 | 2 | 8 | 2m | 1.5s | HorseMeat x4, HorseHide x3, Mane x1 |

---

## Таблица животных — Агрессивные (атакуют первыми)

| Животное | HP | Броня | Урон | Дальность | Перезарядка | Добыча |
|----------|----|-------|------|-----------|-------------|--------|
| Bear | 100 | 5 | 20 | 3m | 2s | BearMeat x8-12, BearHide x5, BearClaw x2, BearFat x1 |
| Boar | 55 | 3 | 15 | 2.5m | 1.2s | BoarMeat x4-6, BoarHide x3, BoarTusk x1-2 |
| Wolf | 40 | 2 | 8 | 2.5m | 1s | WolfMeat x2-4, WolfHide x3, WolfTeeth x1 |

---

## Таблица оружия

| Оружие | Урон | Дальность | Прочность |
|--------|------|-----------|-----------|
| Club (Дубина) | 12 | 2m | 7 ударов |
| Spear (Копьё) | 18 | 3m | 10 ударов |
| Bow (Лук) | 15 | 25m | 5 выстрелов |
| Arrows (Стрелы) | 15 | — | 1 использование |

**UI прочности:** 3D индикатор над объектом.
- Зелёный: 100–80%
- Жёлтый: 80–40%
- Красный: <40%

---

## Таблица инструментов

| Инструмент | Применение | Эффект |
|------------|------------|--------|
| Fishing Rod (Удочка) | Рыбалка | +20% шанс рыбы |
| Jug (Кувшин) | Сбор воды | Наполняет из водоёмов |
| Compass (Компас) | Навигация | Показывает север/базу |
| Rope Lure (Приманка) | Рыбалка | +50% шанс крупной рыбы |
| Trap (Ловушка) | Пассивная охота | Ловит зайцев автоматически |

---

## Таблица брони

| Предмет | Броня | Снижение урона | Крафт |
|---------|-------|----------------|-------|
| Cloth Shirt | 2 | 10% | Cotton x3 |
| Leather Jacket | 5 | 25% | Hide x4, Fiber x2 |
| Wolf Armor | 10 | 40% | WolfHide x6 |
| Bear Coat | 15 | 50% | BearHide x8 |
| Leather Pants | 3 | 15% | Hide x3 |
| Straw Hat | 1 | 5% | Straw x4 |
| Leather Boots | 2 | 10% | Hide x2 |

---

## Атрибуты персонажа

| Атрибут | Старт | Макс | Убывание | При 0 → потеря HP |
|---------|-------|------|----------|------------------|
| Health | 100 | 100 | — | — |
| Hunger | 100 | 100 | -10/ч | -2 HP/ч |
| Thirst | 100 | 100 | -15/ч | -3 HP/ч |

**Дополнительные правила:**
- При HP = 0 → экран проигрыша
- При Hunger = 0 ИЛИ Thirst = 0 → скорость движения ×0.5
- Все значения задаются в MapSettings и сохраняются в JSON

---

## Код: AnimalType.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/Data/AnimalType.cs
public enum AnimalType
{
    Fish,
    Neutral,
    Aggressive
}
```

---

## Код: AnimalAI.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/Steps/AnimalAI.cs
using UnityEngine;

public class AnimalAI : MonoBehaviour
{
    // Параметры
    private AnimalType _type;
    private float _maxHp;
    private float _armor;
    private float _damage;
    private float _attackRange;
    private float _attackCooldown;

    // Состояние
    private float _currentHp;
    private float _attackTimer;
    private Transform _player;

    // FSM состояния
    private enum State { Idle, Wander, Flee, Chase, Attack, Dead }
    private State _state = State.Idle;

    // Для Wander
    private Vector3 _wanderTarget;
    private float   _wanderTimer;

    private float _speed      = 3f;
    private float _fleeSpeed  = 6f;
    private float _chaseSpeed = 4f;
    private const float FleeDistance  = 10f;
    private const float DetectDistance = 15f;

    public void Initialize(AnimalType type, float hp, float armor,
                            float damage, float range, float cooldown)
    {
        _type           = type;
        _maxHp          = hp;
        _currentHp      = hp;
        _armor          = armor;
        _damage         = damage;
        _attackRange    = range;
        _attackCooldown = cooldown;
    }

    private void Start()
    {
        _player = GameObject.FindGameObjectWithTag("Player")?.transform;
        SetWanderTarget();
    }

    private void Update()
    {
        if (_state == State.Dead) return;
        if (_player == null)
        {
            _player = GameObject.FindGameObjectWithTag("Player")?.transform;
            return;
        }

        // Рыбы просто плавают
        if (_type == AnimalType.Fish)
        {
            Wander();
            return;
        }

        float distToPlayer = Vector3.Distance(transform.position, _player.position);

        // Ночное поведение: животные не ходят
        var dayNight = FindFirstObjectByType<DayNightCycle>();
        if (dayNight != null && dayNight.IsNight)
        {
            _state = State.Idle;
            return;
        }

        switch (_state)
        {
            case State.Idle:
                _wanderTimer -= Time.deltaTime;
                if (_wanderTimer <= 0f) _state = State.Wander;
                CheckForPlayer(distToPlayer);
                break;

            case State.Wander:
                Wander();
                CheckForPlayer(distToPlayer);
                break;

            case State.Flee:
                Flee();
                if (distToPlayer > FleeDistance) _state = State.Wander;
                break;

            case State.Chase:
                Chase();
                if (distToPlayer <= _attackRange) _state = State.Attack;
                if (distToPlayer > DetectDistance) _state = State.Wander;
                break;

            case State.Attack:
                _attackTimer -= Time.deltaTime;
                if (_attackTimer <= 0f)
                {
                    DealDamage();
                    _attackTimer = _attackCooldown;
                }
                if (distToPlayer > _attackRange) _state = State.Chase;
                break;
        }
    }

    private void CheckForPlayer(float dist)
    {
        if (_type == AnimalType.Aggressive && dist < DetectDistance)
            _state = State.Chase;
    }

    private void Wander()
    {
        float dist = Vector3.Distance(transform.position, _wanderTarget);
        if (dist < 1f)
        {
            _state       = State.Idle;
            _wanderTimer = Random.Range(2f, 5f);
            return;
        }
        MoveToward(_wanderTarget, _speed);
    }

    private void Flee()
    {
        Vector3 dir = (transform.position - _player.position).normalized;
        transform.position += dir * _fleeSpeed * Time.deltaTime;
    }

    private void Chase()
    {
        MoveToward(_player.position, _chaseSpeed);
    }

    private void MoveToward(Vector3 target, float speed)
    {
        Vector3 dir = (target - transform.position).normalized;
        dir.y = 0f;
        transform.position += dir * speed * Time.deltaTime;
        if (dir != Vector3.zero)
            transform.rotation = Quaternion.LookRotation(dir);
    }

    private void SetWanderTarget()
    {
        Vector3 offset = new Vector3(
            Random.Range(-10f, 10f), 0f, Random.Range(-10f, 10f));
        _wanderTarget = transform.position + offset;
    }

    private void DealDamage()
    {
        var stats = _player.GetComponent<PlayerStats>();
        if (stats != null)
        {
            // Ночью наносим 50% урона
            var dayNight = FindFirstObjectByType<DayNightCycle>();
            float mult   = (dayNight != null && dayNight.IsNight) ? 0.5f : 1f;
            stats.TakeDamage(_damage * mult);
        }
    }

    public void TakeDamage(float amount)
    {
        float actual = Mathf.Max(0f, amount - _armor);
        _currentHp  -= actual;

        // Нейтральные убегают при атаке
        if (_type == AnimalType.Neutral)
            _state = State.Flee;

        if (_currentHp <= 0f)
            Die();
    }

    private void Die()
    {
        _state = State.Dead;
        Debug.Log($"{name} погиб!");
        // TODO: дроп лута
        Destroy(gameObject, 2f); // уничтожить через 2 секунды
    }

    // Вызывается при атаке игрока (из PlayerController)
    private void OnMouseDown()
    {
        TakeDamage(12f); // базовый урон кулаком
    }
}
```

---

## Код: WeaponData.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/Data/WeaponData.cs
using UnityEngine;

[CreateAssetMenu(menuName = "Voronoi/Weapon")]
public class WeaponData : ScriptableObject
{
    public string weaponName;
    public float  damage;
    public float  range;
    public int    maxDurability;
    public int    currentDurability;

    public bool IsBroken => currentDurability <= 0;

    public void Use()
    {
        if (currentDurability > 0)
            currentDurability--;
    }

    public float DurabilityPercent =>
        maxDurability > 0 ? (float)currentDurability / maxDurability : 0f;
}
```

---

## Код: DurabilityIndicator.cs (3D над объектом)

```csharp
// Assets/Scripts/VoronoiMapGenerator/UI/DurabilityIndicator.cs
using UnityEngine;
using UnityEngine.UI;

public class DurabilityIndicator : MonoBehaviour
{
    [SerializeField] private Slider    slider;
    [SerializeField] private Image     fillImage;

    private Camera _cam;

    private void Start()
    {
        _cam = Camera.main;
    }

    private void LateUpdate()
    {
        // Всегда смотреть на камеру
        transform.LookAt(transform.position + _cam.transform.forward);
    }

    public void SetDurability(float percent)
    {
        slider.value = percent;

        if (percent > 0.8f)
            fillImage.color = Color.green;
        else if (percent > 0.4f)
            fillImage.color = Color.yellow;
        else
            fillImage.color = Color.red;
    }
}
```

---

## Как добавить AnimalsStep в проект

1. Создай файл `Assets/Scripts/VoronoiMapGenerator/Steps/AnimalsStep.cs` (код в файле 03-all-scripts.md)
2. В `GameLauncher.cs` в методе `CreateSteps()` добавь строку:

```csharp
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
        new AnimalsStep(),  // ← добавить только эту строку
    };
}
```

Больше НИЧЕГО не меняй. Это Open/Closed принцип в действии.

---

## Правила стартового биома (Grass)

- `isStartBiome = true` у BiomeType
- Нет агрессивных животных (проверяй в AnimalsStep)
- Есть вода рядом (кувшинки, ручей — декоративно)
- Есть камни и деревья (кусты, пни)
- Игрок спавнится именно здесь

В AnimalsStep пишем проверку:
```csharp
bool isStartBiome = ctx.Biomes[cell.BiomeIndex].isStartBiome;
if (!isStartBiome) { /* спавним агрессивных */ }
```

---

## Ночной цикл — влияние на животных

| Время | Животные | Урон |
|-------|----------|------|
| 06:00–20:00 (день) | Ходят нормально | 100% |
| 20:00–06:00 (ночь) | Стоят на месте | 50% |

В `AnimalAI.Update()` проверяй `DayNightCycle.IsNight`.

---

## Параметры по умолчанию в окне настроек

| Параметр | Значение | Диапазон |
|----------|----------|----------|
| pointCount | 50 | 10–200 |
| mapSize | 100 | 50–300 |
| biomeZones | 5 | 1–5 |
| objectPlacementAttempts | 10 | 0–50 |
| visualizeSteps | false | bool |
| maxHealth | 100 | 1–1000 |
| maxHunger | 100 | 1–1000 |
| maxThirst | 100 | 1–1000 |
| hpLossOnHunger | 2 | 0–100 |
| hpLossOnThirst | 3 | 0–100 |

---

## Проверка генерации (требования задания)

| Требование | Как проверить |
|------------|--------------|
| ≤ 0.5с на зону Вороного | Засечь время в Debug.Log |
| Количество клеток ±10% от заданного | `ctx.Cells.Count >= pointCount * 0.9` |
| Минимум 3 вершины у каждой клетки | `polygon.Count >= 3` (уже в коде) |
| Разная форма биомов | Perlin noise даёт органичные формы |
| ≤ 0.5с на биом | Визуализация с паузами, но не проверяется строго |
| 2–5 попыток на биом | `objectPlacementAttempts` (не жёстко) |
| 4 стены по границам | WallsStep ставит 4 куба |
| Стартовый биом безопасный | isStartBiome + проверка в AnimalsStep |
| 3 типа животных | Fish + Neutral + Aggressive |
