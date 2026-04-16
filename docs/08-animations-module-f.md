# Модуль Е — Анимации и аудио (12 баллов, 2 часа)

---

## Прогресс-бар действий

Появляется при длительных действиях (рубка дерева, рыбалка, строительство).

### Внешний вид
- Линейный бар по центру экрана (или над целью)
- Зелёная заливка слева направо
- Текст "Работаем..." сверху
- Иконка действия слева
- При отмене — красное мигание и исчезновение

### Иерархия UI
```
ActionProgressBar (Panel, центр экрана)
├── Background (Image, тёмный полупрозрачный)
├── Icon (Image, 40x40)
├── Label (TMP_Text) — "Работаем..."
└── BarBackground (Image, серый)
    └── BarFill (Image, зелёный) ← width меняем через code
```

### Код: ActionProgressBar.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/UI/ActionProgressBar.cs
using System.Collections;
using TMPro;
using UnityEngine;
using UnityEngine.UI;

public class ActionProgressBar : MonoBehaviour
{
    public static ActionProgressBar Instance { get; private set; }

    [SerializeField] private GameObject panelRoot;
    [SerializeField] private Image      fillImage;
    [SerializeField] private TMP_Text   actionLabel;
    [SerializeField] private Image      icon;

    private Coroutine _activeCoroutine;
    private Color     _greenColor = new Color(0.2f, 0.8f, 0.2f);
    private Color     _redColor   = Color.red;

    private void Awake()
    {
        if (Instance != null) { Destroy(gameObject); return; }
        Instance = this;
        panelRoot.SetActive(false);
    }

    /// <summary>
    /// Запускает прогресс-бар на duration секунд.
    /// onComplete вызывается по завершении.
    /// </summary>
    public void StartAction(string label, float duration, System.Action onComplete, Sprite actionIcon = null)
    {
        if (_activeCoroutine != null)
            StopCoroutine(_activeCoroutine);

        _activeCoroutine = StartCoroutine(RunAction(label, duration, onComplete, actionIcon));
    }

    public void CancelAction()
    {
        if (_activeCoroutine != null)
        {
            StopCoroutine(_activeCoroutine);
            _activeCoroutine = null;
        }
        StartCoroutine(CancelFlash());
    }

    private IEnumerator RunAction(string label, float duration, System.Action onComplete, Sprite actionIcon)
    {
        panelRoot.SetActive(true);
        actionLabel.text = label;
        fillImage.color  = _greenColor;
        if (icon != null && actionIcon != null)
            icon.sprite = actionIcon;

        float elapsed = 0f;
        while (elapsed < duration)
        {
            elapsed += Time.deltaTime;
            fillImage.fillAmount = elapsed / duration;
            yield return null;
        }

        fillImage.fillAmount = 1f;
        panelRoot.SetActive(false);
        _activeCoroutine = null;
        onComplete?.Invoke();
    }

    private IEnumerator CancelFlash()
    {
        panelRoot.SetActive(true);
        fillImage.color = _redColor;

        float t = 0f;
        while (t < 0.3f)
        {
            t += Time.deltaTime;
            float alpha = 1f - t / 0.3f;
            var c       = fillImage.color;
            fillImage.color = new Color(c.r, c.g, c.b, alpha);
            yield return null;
        }

        panelRoot.SetActive(false);
        fillImage.color = _greenColor;
    }
}
```

**Использование:**
```csharp
// Рубка дерева (2 секунды)
ActionProgressBar.Instance.StartAction(
    "Работаем...",
    2f,
    onComplete: () => { /* получить дерево */ }
);

// Отмена при движении
ActionProgressBar.Instance.CancelAction();
```

---

## Animator Controller — Животные

Каждое животное должно иметь Animator с 5 состояниями.

### Как создать Animator Controller

1. ПКМ в Project → `Create` → `Animator Controller`
2. Назови `AnimalAnimator`
3. Двойной клик — откроется окно Animator

### Состояния и переходы

```
Entry → Idle
Idle → Walk     : параметр IsMoving = true
Walk → Idle     : параметр IsMoving = false
Idle → Attack   : параметр Attack (Trigger)
Attack → Idle   : Exit Time (Has Exit Time)
Idle → Hurt     : параметр Hurt (Trigger)
Hurt → Idle     : Exit Time
Idle → Death    : параметр IsDead = true
```

**Создать параметры (в Animator окне → Parameters вкладка):**
- `IsMoving` — Bool
- `Attack` — Trigger
- `Hurt` — Trigger
- `IsDead` — Bool

### Если нет анимаций — делаем фиктивные

Создай пустые клипы:
1. ПКМ в Project → `Create` → `Animation Clip`
2. Назови `Animal_Idle`, `Animal_Walk`, etc.
3. Назначь их в Animator Controller

### Подключение в AnimalAI.cs

```csharp
// Добавить в AnimalAI:
private Animator _animator;

private void Start()
{
    _animator = GetComponent<Animator>();
    // ... остальной Start код
}

private void Update()
{
    // ... в конце Update:
    if (_animator != null)
    {
        _animator.SetBool("IsMoving", _state == State.Wander ||
                                       _state == State.Chase  ||
                                       _state == State.Flee);
        _animator.SetBool("IsDead", _state == State.Dead);
    }
}

// В DealDamage():
_animator?.SetTrigger("Attack");

// В TakeDamage():
_animator?.SetTrigger("Hurt");
```

---

## Animator Controller — Персонаж

### Состояния

```
Entry → Idle
Idle → Walk     : Speed > 0.1
Walk → Idle     : Speed < 0.1
Walk → Run      : IsRunning = true
Run  → Walk     : IsRunning = false
Idle → Attack   : Attack (Trigger)
Attack → Idle   : Exit Time
Idle → Eat      : Eat (Trigger)
Eat  → Idle     : Exit Time
Any  → WeaponSwitch : WeaponSwitch (Trigger), длительность 0.5с
```

**Параметры:**
- `Speed` — Float
- `IsRunning` — Bool
- `Attack` — Trigger
- `Eat` — Trigger
- `WeaponSwitch` — Trigger

### Подключение в PlayerController.cs

```csharp
// Добавить в PlayerController:
private Animator _animator;

private void Awake()
{
    _animator = GetComponentInChildren<Animator>();
    // ...
}

private void FixedUpdate()
{
    HandleMovement();
    UpdateAnimator();
}

private void UpdateAnimator()
{
    if (_animator == null) return;

    float speed = new Vector3(_rb.linearVelocity.x, 0f, _rb.linearVelocity.z).magnitude;
    _animator.SetFloat("Speed", speed);
    _animator.SetBool("IsRunning", Input.GetKey(KeyCode.LeftShift) && speed > 0.1f);
}

// Вызывать при атаке:
private void Attack()
{
    _animator?.SetTrigger("Attack");
}
```

---

## Выстрел из лука

```csharp
// ArrowProjectile.cs
using System.Collections;
using UnityEngine;

public class ArrowProjectile : MonoBehaviour
{
    [SerializeField] private float speed   = 30f;
    [SerializeField] private float damage  = 15f;
    [SerializeField] private float maxDist = 25f;

    private Vector3 _direction;
    private float   _travelled;

    public void Launch(Vector3 direction)
    {
        _direction = direction.normalized;
    }

    private void Update()
    {
        float step = speed * Time.deltaTime;
        transform.position  += _direction * step;
        transform.rotation  = Quaternion.LookRotation(_direction);
        _travelled          += step;

        if (_travelled >= maxDist)
            Destroy(gameObject);
    }

    private void OnTriggerEnter(Collider other)
    {
        var animal = other.GetComponent<AnimalAI>();
        if (animal != null)
        {
            animal.TakeDamage(damage);
            ShowHitEffect();
        }

        // Застреваем в земле/стенах
        Destroy(GetComponent<Rigidbody>());
        Destroy(this);
    }

    private void ShowHitEffect()
    {
        // Красная вспышка (TODO: частицы)
        StartCoroutine(FlashRed());
    }

    private IEnumerator FlashRed()
    {
        var renderer = GetComponent<MeshRenderer>();
        if (renderer == null) yield break;

        var mat = renderer.material;
        mat.color = Color.red;
        yield return new WaitForSeconds(0.1f);
        mat.color = Color.white;
    }
}
```

---

## Бросок копья

```csharp
// SpearThrow.cs
using System.Collections;
using UnityEngine;

public class SpearThrow : MonoBehaviour
{
    [SerializeField] private float speed  = 20f;
    [SerializeField] private float damage = 18f;
    [SerializeField] private bool  isStuck;

    private Rigidbody _rb;

    private void Awake()
    {
        _rb = GetComponent<Rigidbody>();
    }

    public void Throw(Vector3 direction)
    {
        if (_rb != null)
        {
            _rb.isKinematic = false;
            _rb.linearVelocity = direction.normalized * speed;
            _rb.angularVelocity = Vector3.zero;
        }
    }

    private void OnCollisionEnter(Collision collision)
    {
        if (isStuck) return;
        isStuck = true;

        var animal = collision.collider.GetComponent<AnimalAI>();
        if (animal != null)
            animal.TakeDamage(damage);

        // Застреваем вертикально
        if (_rb != null)
        {
            _rb.isKinematic = true;
            _rb.linearVelocity    = Vector3.zero;
        }

        // Поворачиваем вертикально (копьё торчит вертикально)
        transform.rotation = Quaternion.Euler(90f, transform.rotation.eulerAngles.y, 0f);

        Destroy(gameObject, 10f); // убираем через 10 сек
    }
}
```

---

## UI Анимации

### Инвентарь — выезжает сверху за 0.3с

```csharp
// InventorySlideIn.cs
using System.Collections;
using UnityEngine;

public class InventorySlideIn : MonoBehaviour
{
    [SerializeField] private RectTransform panel;
    [SerializeField] private float         slideDuration = 0.3f;

    private Vector2 _hiddenPos;
    private Vector2 _shownPos;

    private void Awake()
    {
        _shownPos  = panel.anchoredPosition;
        _hiddenPos = new Vector2(_shownPos.x, Screen.height + panel.rect.height);
        panel.anchoredPosition = _hiddenPos;
    }

    public void Show()
    {
        gameObject.SetActive(true);
        StopAllCoroutines();
        StartCoroutine(Slide(_hiddenPos, _shownPos));
    }

    public void Hide()
    {
        StopAllCoroutines();
        StartCoroutine(SlideOut());
    }

    private IEnumerator Slide(Vector2 from, Vector2 to)
    {
        float t = 0f;
        while (t < slideDuration)
        {
            t += Time.unscaledDeltaTime; // работает при паузе (timeScale = 0)
            panel.anchoredPosition = Vector2.Lerp(from, to, t / slideDuration);
            yield return null;
        }
        panel.anchoredPosition = to;
    }

    private IEnumerator SlideOut()
    {
        yield return StartCoroutine(Slide(_shownPos, _hiddenPos));
        gameObject.SetActive(false);
    }
}
```

> **Важно:** Используй `Time.unscaledDeltaTime` для анимаций паузы! Иначе при `Time.timeScale = 0` анимация не проиграется.

---

### Числа урона — всплывают над целью

```csharp
// DamageNumber.cs
using System.Collections;
using TMPro;
using UnityEngine;

public class DamageNumber : MonoBehaviour
{
    [SerializeField] private TMP_Text text;

    private Camera _cam;

    public void Show(float damage, Vector3 worldPos)
    {
        _cam           = Camera.main;
        text.text      = $"-{Mathf.RoundToInt(damage)}";
        text.color     = Color.red;
        transform.position = worldPos + Vector3.up * 2f;
        StartCoroutine(Animate());
    }

    private IEnumerator Animate()
    {
        float duration = 1.5f;
        float elapsed  = 0f;
        Vector3 startPos = transform.position;

        while (elapsed < duration)
        {
            elapsed += Time.deltaTime;
            float t = elapsed / duration;

            // Двигаем вверх
            transform.position = startPos + Vector3.up * t * 2f;

            // Смотрим на камеру
            if (_cam != null)
                transform.LookAt(transform.position + _cam.transform.forward);

            // Тускнеем
            var c   = text.color;
            text.color = new Color(c.r, c.g, c.b, 1f - t);

            yield return null;
        }

        Destroy(gameObject);
    }

    // Статический метод для удобного создания
    public static void Spawn(float damage, Vector3 worldPos, DamageNumber prefab)
    {
        var dn = Instantiate(prefab, worldPos, Quaternion.identity);
        dn.Show(damage, worldPos);
    }
}
```

**Как использовать:**
1. Создай Canvas в World Space (для 3D числа)
2. Создай префаб с TMP_Text + DamageNumber скриптом
3. В AnimalAI.TakeDamage вызывай `DamageNumber.Spawn(damage, transform.position, dmgPrefab)`

---

### Экран краснеет при уроне

```csharp
// DamageFlash.cs
using System.Collections;
using UnityEngine;
using UnityEngine.UI;

public class DamageFlash : MonoBehaviour
{
    public static DamageFlash Instance { get; private set; }

    [SerializeField] private Image flashImage;  // полноэкранный красный Image

    private void Awake()
    {
        if (Instance != null) { Destroy(gameObject); return; }
        Instance = this;
        // Начинаем прозрачным
        var c = flashImage.color;
        flashImage.color = new Color(c.r, c.g, c.b, 0f);
    }

    public void Flash()
    {
        StopAllCoroutines();
        StartCoroutine(DoFlash());
    }

    private IEnumerator DoFlash()
    {
        float duration = 0.3f;
        float half     = duration / 2f;

        // Появляемся
        float t = 0f;
        while (t < half)
        {
            t += Time.unscaledDeltaTime;
            SetAlpha(Mathf.Lerp(0f, 0.5f, t / half));
            yield return null;
        }

        // Исчезаем
        t = 0f;
        while (t < half)
        {
            t += Time.unscaledDeltaTime;
            SetAlpha(Mathf.Lerp(0.5f, 0f, t / half));
            yield return null;
        }

        SetAlpha(0f);
    }

    private void SetAlpha(float a)
    {
        var c        = flashImage.color;
        flashImage.color = new Color(c.r, c.g, c.b, a);
    }
}
```

**Как использовать:**
- В DamageFlash нужен Image на весь экран (черта через AnchorPreset = Stretch)
- Color Image: красный (R:1, G:0, B:0)
- Raycast Target: ❌ (чтобы не блокировал клики)
- В PlayerStats.TakeDamage: `DamageFlash.Instance?.Flash();`

---

## День/ночь цикл — полная реализация

```csharp
// Уже написан в 04-ui-module-b.md — DayNightCycle.cs
// Доп. поведение ночью для животных:

// В AnimalAI.Update() добавить в начало:
var dayNight = FindFirstObjectByType<DayNightCycle>();
bool isNight = dayNight != null && dayNight.IsNight;

if (isNight && _state == State.Wander)
{
    // Ночью стоим на месте
    _state = State.Idle;
    _rb?.Sleep();
}
```

---

## Прогресс-бар дня/ночи

На GameHUD в нижней части:
```
DayNightBarContainer
├── Label (Text) — "🌙 Ночь" или "☀ День"
└── DayNightBar (Slider)
    ├── Fill: тёмно-коричневый цвет
    └── Direction: Right To Left (убывает)
```

В `DayNightCycle.cs` уже есть вызов `GameHUD.Instance.UpdateDayNightBar(progress)`.

---

## Звуки — обязательные

### Настройка AudioManager

1. На объект `GameManager` добавь два `AudioSource` компонента
2. Первый — для фоновой музыки (`Play On Awake = true`, `Loop = true`)
3. Второй — для SFX (`Play On Awake = false`)

AudioManager скрипт уже написан в `05-json-module-c.md`.

### Добавление звуков в Inspector

В GameManager → AudioManager компонент:
- `Game Over Clip` — перетащи аудиоклип из Project
- `Build Clip` — перетащи аудиоклип из Project

### Где звуки вызываются

```csharp
// Проигрыш (в UIManager.ShowGameOver()):
AudioManager.Instance?.PlayGameOver();

// Строительство (в ObjectsStep.Execute() или при постройке):
AudioManager.Instance?.PlayBuild();

// Урон игроку (в PlayerStats.TakeDamage()):
AudioManager.Instance?.PlaySFX(damageClip);
```

---

## Чеклист Модуля Е

```
□ ActionProgressBar создан и работает (появляется, зеленая заливка, красный при отмене)
□ Animator Controller для животных: 5 состояний (Idle, Walk, Attack, Hurt, Death)
□ Animator Controller для персонажа: 6 состояний
□ Переходы между состояниями настроены (параметры: Speed, IsRunning, Attack trigger)
□ DayNightCycle работает: день=2мин, ночь=2мин (итого 4мин цикл нет, 2мин полный)
□ Ночью животные стоят на месте и наносят 50% урона
□ Прогресс-бар дня/ночи в HUD обновляется
□ DamageFlash: экран краснеет при уроне (0.3с)
□ DamageNumber: числа всплывают над целью и тают
□ Инвентарь выезжает сверху за 0.3с
□ AudioManager на GameManager: 2 AudioSource
□ Звук проигрыша работает
□ Звук строительства работает
□ ArrowProjectile летит и наносит урон
□ SpearThrow летит и застревает вертикально
```

---

## Типичные ошибки Модуля Е

**Ошибка:** Animator не переключает состояния  
**Решение:** Проверь параметры (Bool/Trigger) — они должны совпадать по имени с кодом. Проверь условия переходов.

**Ошибка:** UI анимация не работает при паузе (timeScale = 0)  
**Решение:** Используй `Time.unscaledDeltaTime` вместо `Time.deltaTime` для всех UI анимаций.

**Ошибка:** AudioSource ничего не играет  
**Решение:** Проверь: Volume > 0, AudioListener есть в сцене (обычно на Main Camera), AudioClip назначен.

**Ошибка:** DamageNumber не смотрит на камеру  
**Решение:** Используй `transform.LookAt(transform.position + Camera.main.transform.forward)` (не `transform.LookAt(camera)`).

**Ошибка:** Стрела не попадает в цель  
**Решение:** Убедись что у цели есть Collider. Если стрела использует Rigidbody Physics — нужен тег коллизии. Если Trigger — нужен OnTriggerEnter.
