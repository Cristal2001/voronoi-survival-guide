# 📄 Листок День 2 — Модули В + Г + Д (52 балла)

> ⏱️ Читать 5 минут → переписать формулы на листок → работать
> 💡 Самый важный день — больше всего баллов!

---

## 💾 Модуль В — JSON (17 баллов, 4 ч)

**Файлы в `Assets/StreamingAssets/`:**

```
mapSettings.json   ← настройки генерации + атрибуты игрока
settings.json      ← звук, разрешение, язык, мышь
save.json          ← состояние игры (автосохранение 30 сек)
craft.json         ← рецепты крафта
locale_ru.json     ← все тексты на русском
locale_en.json     ← все тексты на английском
```

**Путь к StreamingAssets (работает в билде!):**

```csharp
string path = Path.Combine(Application.streamingAssetsPath, "mapSettings.json");
```

**Сохранение / загрузка:**

```csharp
// Сохранить
File.WriteAllText(path, JsonUtility.ToJson(obj, true));

// Загрузить
var obj = JsonUtility.FromJson<T>(File.ReadAllText(path));
```

**Структура `save.json`:**

```json
{
  "playerX": 50.0, "playerY": 0.5, "playerZ": 50.0,
  "hp": 85.0, "hunger": 60.0, "thirst": 40.0,
  "day": 3, "gameHour": 14.5,
  "inventory": []
}
```

**Автосохранение каждые 30 секунд:**

```csharp
private float _saveTimer;

void Update() {
    _saveTimer += Time.deltaTime;
    if (_saveTimer >= 30f) { Save(); _saveTimer = 0f; }
}
```

---

## 🗺️ Модуль Г — Генерация карты (24 балла, 3 ч)

### ⚙️ Алгоритм Вороного (самое сложное — ЗАПИШИ!)

**Sutherland-Hodgman — обрезка полигона по биссектрисе:**

```csharp
// Для каждой пары точек cur→nxt в полигоне:
bool curIn = Vector2.Dot(cur - mid, normal) <= 0;  // по какую сторону
bool nxtIn = Vector2.Dot(nxt - mid, normal) <= 0;

if (curIn) result.Add(cur);          // текущая внутри → добавить
if (curIn != nxtIn)                  // пересекает границу → пересечение
{
    float t = Vector2.Dot(mid - cur, normal) / Vector2.Dot(nxt - cur, normal);
    result.Add(cur + (nxt - cur) * t);
}
```

где `mid = (A+B)/2`, `normal = (B-A).normalized`

---

### 🧱 Меш клетки (CellMeshBuilder) — ЗАПИШИ ИНДЕКСЫ!

**Массив вершин** (`n` = кол-во точек полигона):

```
[0 .. n-1]    → верхний периметр (с высотой Perlin)
[n]           → центр (с высотой Perlin)
[n+1 .. 2n]   → нижний периметр (Y = 0, те же XZ)
```

**Треугольники крыши (веер из центра):**

```csharp
triangles[i*3]   = n;          // центр
triangles[i*3+1] = (i+1) % n; // следующая
triangles[i*3+2] = i;          // текущая
```

**Треугольники стенок (2 треугольника на грань):**

```csharp
int off = n * 3;
// t0=i, t1=(i+1)%n — верхние; f0=n+1+i, f1=n+1+(i+1)%n — нижние
triangles[off + i*6+0] = t0; triangles[off + i*6+1] = t1; triangles[off + i*6+2] = f1;
triangles[off + i*6+3] = t0; triangles[off + i*6+4] = f1; triangles[off + i*6+5] = f0;
```

**Perlin высота:**

```csharp
float H(float x, float y) =>
    Mathf.PerlinNoise(x * 0.05f + offsetX, y * 0.05f + offsetY) * heightNoise;
```

---

### 🐾 Животные

**Самый быстрый способ — один скрипт `AnimalAI.cs` на все типы:**
> Вешать на префаб животного. Никакого NavMesh — просто Transform.MoveTowards.

```csharp
using UnityEngine;
public class AnimalAI : MonoBehaviour
{
    public enum AnimalType { Neutral, Aggressive, Fish }

    [SerializeField] private AnimalType _type        = AnimalType.Neutral;
    [SerializeField] private float _hp               = 10f;
    [SerializeField] private float _damage           = 2f;
    [SerializeField] private float _moveSpeed        = 3f;
    [SerializeField] private float _runSpeed         = 6f;
    [SerializeField] private float _detectRange      = 10f;
    [SerializeField] private float _attackRange      = 2f;
    [SerializeField] private float _attackCooldown   = 1.5f;

    private Transform _player;
    private Vector3   _wanderTarget;
    private float     _wanderTimer;
    private float     _attackTimer;
    private bool      _isDead;

    private void Start()
    {
        var p = GameObject.FindGameObjectWithTag("Player");
        if (p != null) _player = p.transform;
        PickNewWanderTarget();
    }

    private void Update()
    {
        if (_isDead) return;
        if (_type == AnimalType.Fish) { Wander(_moveSpeed); return; }

        float dist = _player != null
            ? Vector3.Distance(transform.position, _player.position) : float.MaxValue;

        if (_type == AnimalType.Neutral)
        {
            if (dist < _detectRange) FleeFrom(_player.position);
            else                     Wander(_moveSpeed);
        }
        else // Aggressive
        {
            if      (dist < _attackRange)  AttackPlayer();
            else if (dist < _detectRange)  MoveTowards(_player.position, _runSpeed);
            else                           Wander(_moveSpeed);
        }
    }

    private void Wander(float speed)
    {
        _wanderTimer -= Time.deltaTime;
        if (_wanderTimer <= 0f) PickNewWanderTarget();
        MoveTowards(_wanderTarget, speed);
    }

    private void FleeFrom(Vector3 threat)
    {
        Vector3 dir = (transform.position - threat).normalized;
        MoveTowards(transform.position + dir * 5f, _runSpeed);
    }

    private void AttackPlayer()
    {
        _attackTimer -= Time.deltaTime;
        if (_attackTimer <= 0f)
        {
            _attackTimer = _attackCooldown;
            _player.GetComponent<PlayerStats>()?.TakeDamage(_damage);
        }
    }

    private void MoveTowards(Vector3 target, float speed)
    {
        Vector3 dir = target - transform.position; dir.y = 0f;
        if (dir.magnitude < 0.1f) return;
        transform.position += dir.normalized * speed * Time.deltaTime;
        transform.rotation  = Quaternion.LookRotation(dir.normalized);
    }

    private void PickNewWanderTarget()
    {
        Vector2 r  = Random.insideUnitCircle * 10f;
        _wanderTarget = transform.position + new Vector3(r.x, 0, r.y);
        _wanderTimer  = Random.Range(3f, 7f);
    }

    public void TakeDamage(float dmg)
    {
        if (_isDead) return;
        _hp -= dmg;
        if (_type == AnimalType.Neutral && _player != null) FleeFrom(_player.position);
        if (_hp <= 0f) { _isDead = true; Destroy(gameObject, 1f); }
    }
}
```

**Как настроить в инспекторе:**
| Тип | AnimalType | HP | Damage | DetectRange | AttackRange |
|-----|-----------|-----|--------|------------|------------|
| Кролик | Neutral | 10 | 0 | 10 | — |
| Волк | Aggressive | 40 | 8 | 10 | 2.5 |
| Медведь | Aggressive | 100 | 20 | 12 | 3 |
| Рыба | Fish | 5 | 0 | — | — |

> ⚠️ На объекте Player должен быть тег **"Player"** (Inspector → Tag → Player)

**Просто идёт на игрока — `AnimalMove.cs`:**
> Самый простой вариант. Вешать на животное, больше ничего не нужно.

```csharp
using UnityEngine;

public class AnimalMove : MonoBehaviour
{
    private Transform _player;
    [SerializeField] private float _speed = 3f;

    void Start()
    {
        _player = GameObject.FindGameObjectWithTag("Player").transform;
    }

    void Update()
    {
        transform.position = Vector3.MoveTowards(transform.position, _player.position, _speed * Time.deltaTime);
    }
}
```

---

**Нейтральные (убегают при атаке на 10м):**

| Животное | HP | Урон | Дальность | Лут |
|----------|-----|------|-----------|-----|
| Rabbit | 10 | 2 | 1.5m | RabbitMeat×1-2, RabbitHide×1 |
| Butterfly | 1 | 0 | — | ButterflyWings×1 (30%) |
| Deer | 30 | 5 | 2m | DeerMeat×3-5, DeerHide×2 |
| Bison | 60 | 12 | 2.5m | BisonMeat×6-8, BisonHide×4 |
| Horse | 45 | 8 | 2m | HorseMeat×4, HorseHide×3 |

**Агрессивные (атакуют первыми):**

| Животное | HP | Броня | Урон | Лут |
|----------|-----|-------|------|-----|
| Bear | 100 | 5 | 20 | BearMeat×8-12, BearHide×5 |
| Boar | 55 | 3 | 15 | BoarMeat×4-6, BoarTusk×1-2 |
| Wolf | 40 | 2 | 8 | WolfMeat×2-4, WolfHide×3 |

**Рыбы (только в воде):**

| Рыба | HP | Лут |
|------|-----|-----|
| Small Fish | 5 | FishMeat×1 |
| Trout | 8 | FishMeat×1-2, FishScales×1 |
| Salmon | 12 | FishMeat×2-3, FishScales×2 |

---

### ⚔️ Оружие и инструменты

| Оружие | Урон | Дальность | Прочность |
|--------|------|-----------|-----------|
| Club | 12 | 2m | 7 ударов |
| Spear | 18 | 3m | 10 ударов |
| Bow | 15 | 25m | 5 выстрелов |

| Инструмент | Назначение |
|------------|-----------|
| Fishing Rod | Рыбалка (+20% шанс) |
| Jug | Сбор воды |
| Compass | Показывает север/базу |
| Trap | Ловит зайцев пассивно |

---

### 🧍 Атрибуты персонажа

| Атрибут | Убыль | При 0 → HP |
|---------|-------|-----------|
| Hunger | −10 / час | −2 HP/час |
| Thirst | −15 / час | −3 HP/час |

> При любом = 0: **скорость ×0.5**.
> При HP = 0: **`ShowGameOver()`**.

---

## 🕹️ Модуль Д — Управление (11 баллов, 2.5 ч)

**Клавиши:**

| Действие | Клавиша |
|----------|---------|
| Движение | WASD |
| Бег | Shift (hold) |
| Камера zoom | Колесо мыши |
| Вращение камеры | Q / E |
| Атака / собрать | ЛКМ |
| Контекстное меню | ПКМ |
| Автоподбор | Пробел |
| Выделить врага | T |

**Камера (следует за игроком, в LateUpdate):**

```csharp
Vector3 offset = new Vector3(0, 15f, -20f); // смещение над и позади
transform.position = player.position + rotation * offset;
transform.LookAt(player.position + Vector3.up * 1.5f);
```

**Вращение камеры Q/E:**

```csharp
if (Input.GetKey(KeyCode.Q)) _yaw -= rotSpeed * Time.deltaTime;
if (Input.GetKey(KeyCode.E)) _yaw += rotSpeed * Time.deltaTime;
Quaternion rotation = Quaternion.Euler(0, _yaw, 0);
```

**Зум камеры колесом мыши — `UI/CameraZoom.cs`:**
> Вешать на объект **Main Camera**

```csharp
using UnityEngine;
public class CameraZoom : MonoBehaviour
{
    private Camera _camera;
    [SerializeField] private float zoomSpeed = 10f; // скорость зума
    [SerializeField] private float minZoom   = 20f; // минимальный угол обзора
    [SerializeField] private float maxZoom   = 60f; // максимальный угол обзора

    void Start()  => _camera = GetComponent<Camera>();

    void Update()
    {
        float scroll = Input.GetAxis("Mouse ScrollWheel"); // колесо: + вперёд, - назад
        if (scroll != 0)
        {
            _camera.fieldOfView -= scroll * zoomSpeed;                      // меняем угол обзора
            _camera.fieldOfView  = Mathf.Clamp(_camera.fieldOfView, minZoom, maxZoom); // зажимаем в пределах
        }
    }
}
```

**Подбор предмета кликом — `UI/InventoryRaycast.cs`:**
> Вешать на каждый **предмет в мире** (3D-объект который можно подобрать)
> В инспекторе указать: `_object` — кнопка слота инвентаря, `_objImage` — иконка предмета

```csharp
using UnityEngine;
using UnityEngine.UI;

public class InventoryRaycast : MonoBehaviour
{
    [SerializeField] private Button _object;   // кнопка слота инвентаря (выключена пока предмет лежит)
    [SerializeField] private Image  _objImage; // иконка предмета в инвентаре

    void Update()
    {
        // Пускаем луч из камеры через курсор мыши в мир
        Ray        ray = Camera.main.ScreenPointToRay(Input.mousePosition);
        RaycastHit hit;

        // Условие: луч попал в этот объект И игрок нажал ЛКМ
        if (Physics.Raycast(ray, out hit, 999f)
            && hit.collider.gameObject == gameObject
            && Input.GetMouseButtonDown(0))
        {
            Destroy(gameObject);        // удаляем предмет из мира
            _object.enabled   = true;  // включаем кнопку слота в инвентаре
            _objImage.enabled = true;  // показываем иконку предмета
        }
    }
}
```
