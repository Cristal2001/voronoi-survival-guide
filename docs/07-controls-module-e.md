# Модуль Д — Управление персонажем (11 баллов, 2.5 часа)

---

## Таблица управления

| Действие | Клавиша |
|----------|---------|
| Движение вперёд | W |
| Движение назад | S |
| Движение влево | A |
| Движение вправо | D |
| Движение к точке | ПКМ+ЛКМ (клик по земле) |
| Бег | Shift (держать) |
| Открыть крафт | C |
| Камера вверх/вниз | Колесо мыши |
| Вращение камеры | Q (влево) / E (вправо) |
| Взять / Ударить / Собрать | ЛКМ |
| Атака / Поставить постройку | ПКМ |
| Автоподбор предметов | Пробел |
| Выделить врага | T |
| Вращать постройку | R |
| Рыбалка | ПКМ (над водой) |
| Поставить ловушку | ЛКМ (на земле) |
| Контекстное меню | ПКМ (на объекте) |
| Пауза | Pause |
| Экран проигрыша (тест) | F11 |

---

## Архитектура управления

```
PlayerController (MonoBehaviour)
├── движение WASD + Shift
├── управление камерой (Q/E + колесо мыши)
├── ЛКМ — атака/взаимодействие
├── ПКМ — контекстное меню / атака
├── C — открыть крафт
└── Space — автоподбор

ContextMenuPanel (UI Panel)
├── появляется при ПКМ на объект
└── кнопки действий (зависит от объекта)
```

---

## Код: PlayerController.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/PlayerController.cs
using UnityEngine;
using UnityEngine.EventSystems;

[RequireComponent(typeof(Rigidbody))]
public class PlayerController : MonoBehaviour
{
    [Header("Движение")]
    [SerializeField] private float walkSpeed = 5f;
    [SerializeField] private float runSpeed  = 10f;

    [Header("Камера")]
    [SerializeField] private Transform cameraTransform;
    [SerializeField] private float cameraDistance   = 20f;
    [SerializeField] private float cameraMinDistance = 5f;
    [SerializeField] private float cameraMaxDistance = 50f;
    [SerializeField] private float cameraHeight      = 15f;
    [SerializeField] private float cameraAngle       = 60f;  // наклон вниз
    [SerializeField] private float cameraRotateSpeed = 90f;  // градусов/сек
    [SerializeField] private float scrollSpeed       = 5f;

    [Header("Взаимодействие")]
    [SerializeField] private float interactRange = 3f;
    [SerializeField] private ContextMenuPanel contextMenuPanel;

    [Header("Ссылки")]
    [SerializeField] private PlayerStats playerStats;

    private Rigidbody _rb;
    private float     _cameraYaw = 0f;  // горизонтальный угол камеры
    private Camera    _cam;
    private int       _layerMask;

    private void Awake()
    {
        _rb        = GetComponent<Rigidbody>();
        _rb.freezeRotation = true;
        _cam       = Camera.main;
        _layerMask = LayerMask.GetMask("Default");
    }

    private void Update()
    {
        HandleCameraInput();
        HandleActions();
        HandleHotkeys();
    }

    private void FixedUpdate()
    {
        HandleMovement();
    }

    // ─── Движение ───────────────────────────────────────────────────────────

    private void HandleMovement()
    {
        float h = Input.GetAxisRaw("Horizontal"); // A/D
        float v = Input.GetAxisRaw("Vertical");   // W/S

        // Направление движения относительно поворота камеры
        Vector3 forward = Quaternion.Euler(0, _cameraYaw, 0) * Vector3.forward;
        Vector3 right   = Quaternion.Euler(0, _cameraYaw, 0) * Vector3.right;
        Vector3 move    = (forward * v + right * h).normalized;

        bool isRunning = Input.GetKey(KeyCode.LeftShift);

        // Уменьшаем скорость при истощении
        float speedMult = (playerStats != null && playerStats.IsExhausted) ? 0.5f : 1f;
        float speed     = (isRunning ? runSpeed : walkSpeed) * speedMult;

        _rb.linearVelocity = new Vector3(
            move.x * speed,
            _rb.linearVelocity.y,
            move.z * speed
        );

        // Поворот персонажа в направлении движения
        if (move.sqrMagnitude > 0.01f)
            transform.rotation = Quaternion.LookRotation(move);
    }

    // ─── Камера ─────────────────────────────────────────────────────────────

    private void HandleCameraInput()
    {
        // Q/E — вращение камеры
        if (Input.GetKey(KeyCode.Q))
            _cameraYaw -= cameraRotateSpeed * Time.deltaTime;
        if (Input.GetKey(KeyCode.E))
            _cameraYaw += cameraRotateSpeed * Time.deltaTime;

        // Колесо мыши — приближение/удаление
        float scroll = Input.GetAxis("Mouse ScrollWheel");
        cameraDistance -= scroll * scrollSpeed;
        cameraDistance  = Mathf.Clamp(cameraDistance, cameraMinDistance, cameraMaxDistance);

        UpdateCameraPosition();
    }

    private void UpdateCameraPosition()
    {
        if (cameraTransform == null) return;

        // Позиция камеры: сзади и выше игрока, повёрнута на _cameraYaw
        Quaternion rotation = Quaternion.Euler(0f, _cameraYaw, 0f);
        Vector3 offset      = rotation * new Vector3(0f, cameraHeight, -cameraDistance);

        cameraTransform.position = transform.position + offset;

        // Смотрим на игрока с углом вниз
        cameraTransform.LookAt(transform.position + Vector3.up * 1.5f);
    }

    // ─── Действия (ЛКМ/ПКМ) ────────────────────────────────────────────────

    private void HandleActions()
    {
        // Не обрабатываем клики по UI
        if (EventSystem.current.IsPointerOverGameObject()) return;

        // ПКМ — контекстное меню или атака
        if (Input.GetMouseButtonDown(1))
        {
            HandleRightClick();
        }

        // ЛКМ — атака/взаимодействие
        if (Input.GetMouseButtonDown(0))
        {
            HandleLeftClick();
        }
    }

    private void HandleRightClick()
    {
        Ray ray = _cam.ScreenPointToRay(Input.mousePosition);
        if (Physics.Raycast(ray, out RaycastHit hit, 100f))
        {
            var interactive = hit.collider.GetComponent<IInteractable>();
            if (interactive != null)
            {
                // Показываем контекстное меню
                var actions = interactive.GetContextActions(this);
                if (contextMenuPanel != null)
                    contextMenuPanel.Show(actions, Input.mousePosition);
            }
            else
            {
                // Атака в пустоту / движение к точке
                MoveToPoint(hit.point);
            }
        }
    }

    private void HandleLeftClick()
    {
        Ray ray = _cam.ScreenPointToRay(Input.mousePosition);
        if (Physics.Raycast(ray, out RaycastHit hit, interactRange + 50f))
        {
            // Атака животного
            var animal = hit.collider.GetComponent<AnimalAI>();
            if (animal != null)
            {
                float dist = Vector3.Distance(transform.position, hit.point);
                if (dist <= interactRange)
                    animal.TakeDamage(12f); // базовый урон руками
                return;
            }

            // Взаимодействие с объектом
            var interactable = hit.collider.GetComponent<IInteractable>();
            if (interactable != null)
            {
                float dist = Vector3.Distance(transform.position, hit.point);
                if (dist <= interactRange)
                    interactable.Interact(this);
            }
        }
    }

    private void MoveToPoint(Vector3 point)
    {
        // Простое движение к точке (без навмеша — просто поворачиваем)
        StartCoroutine(MoveToPointCoroutine(point));
    }

    private System.Collections.IEnumerator MoveToPointCoroutine(Vector3 target)
    {
        target.y = transform.position.y;
        float timeout = 5f;
        while (Vector3.Distance(transform.position, target) > 0.5f && timeout > 0f)
        {
            Vector3 dir = (target - transform.position).normalized;
            _rb.linearVelocity = new Vector3(dir.x * walkSpeed, _rb.linearVelocity.y, dir.z * walkSpeed);
            transform.rotation = Quaternion.LookRotation(dir);
            timeout -= Time.deltaTime;
            yield return null;
        }
        _rb.linearVelocity = new Vector3(0f, _rb.linearVelocity.y, 0f);
    }

    // ─── Горячие клавиши ────────────────────────────────────────────────────

    private void HandleHotkeys()
    {
        // C — открыть/закрыть крафт
        if (Input.GetKeyDown(KeyCode.C))
            UIManager.Instance.ShowCraftPanel();

        // Pause — пауза
        if (Input.GetKeyDown(KeyCode.Pause))
            UIManager.Instance.ShowPause();

        // F11 — экран проигрыша (тест)
        if (Input.GetKeyDown(KeyCode.F11))
            UIManager.Instance.ShowGameOver();

        // Space — автоподбор
        if (Input.GetKeyDown(KeyCode.Space))
            AutoPickup();

        // T — выделить ближайшего врага
        if (Input.GetKeyDown(KeyCode.T))
            TargetNearestEnemy();

        // R — вращать постройку (TODO)
        if (Input.GetKeyDown(KeyCode.R))
            Debug.Log("Rotate building");
    }

    private void AutoPickup()
    {
        // Подбираем все объекты с компонентом Pickable в радиусе
        var pickables = Physics.OverlapSphere(transform.position, interactRange);
        foreach (var col in pickables)
        {
            var pickable = col.GetComponent<IPickable>();
            if (pickable != null)
                pickable.PickUp(this);
        }
    }

    private void TargetNearestEnemy()
    {
        float minDist = float.MaxValue;
        AnimalAI nearest = null;

        foreach (var animal in FindObjectsByType<AnimalAI>(FindObjectsSortMode.None))
        {
            float d = Vector3.Distance(transform.position, animal.transform.position);
            if (d < minDist)
            {
                minDist = d;
                nearest = animal;
            }
        }

        if (nearest != null)
            Debug.Log($"Цель: {nearest.name} ({minDist:F1}м)");
    }
}
```

---

## Интерфейсы взаимодействия

```csharp
// Assets/Scripts/VoronoiMapGenerator/Interfaces/IInteractable.cs
using System.Collections.Generic;

public interface IInteractable
{
    string[]              GetContextActions(PlayerController player);
    void                  Interact(PlayerController player);
}
```

```csharp
// Assets/Scripts/VoronoiMapGenerator/Interfaces/IPickable.cs
public interface IPickable
{
    void PickUp(PlayerController player);
}
```

---

## Контекстное меню — примеры

| Объект | Предмет в руке | Пункты меню |
|--------|---------------|-------------|
| Дерево | Топор | Собрать дерево, Удар, Осмотреть |
| Дерево | Руки | Осмотреть, Удар (медленно) |
| Кролик | Дубина | Атаковать, Подойти, Осмотреть |
| Вода | Кувшин | Наполнить, Пить, Осмотреть |
| Костёр | — | Развести, Греться, Готовить |
| Сундук | — | Открыть, Осмотреть |

---

## Код: ContextMenuPanel.cs

```csharp
// Assets/Scripts/VoronoiMapGenerator/UI/ContextMenuPanel.cs
using System;
using System.Collections.Generic;
using TMPro;
using UnityEngine;
using UnityEngine.UI;

public class ContextMenuPanel : MonoBehaviour
{
    [SerializeField] private GameObject     buttonPrefab;  // кнопка с TMP_Text
    [SerializeField] private Transform      buttonContainer;
    [SerializeField] private RectTransform  panel;

    private readonly List<GameObject> _buttons = new List<GameObject>();

    /// <summary>
    /// Показывает контекстное меню с действиями.
    /// actions: массив пар (название, callback)
    /// </summary>
    public void Show(string[] actionNames, Vector2 screenPos, Action[] callbacks = null)
    {
        // Очищаем старые кнопки
        foreach (var btn in _buttons)
            Destroy(btn);
        _buttons.Clear();

        // Создаём новые кнопки
        for (int i = 0; i < actionNames.Length; i++)
        {
            int idx = i; // захват для лямбды
            var btnGo  = Instantiate(buttonPrefab, buttonContainer);
            var btnText = btnGo.GetComponentInChildren<TMP_Text>();
            if (btnText != null) btnText.text = actionNames[idx];

            var btn = btnGo.GetComponent<Button>();
            if (btn != null)
            {
                if (callbacks != null && idx < callbacks.Length)
                    btn.onClick.AddListener(() => { callbacks[idx]?.Invoke(); Hide(); });
                else
                    btn.onClick.AddListener(() => Hide());
            }

            _buttons.Add(btnGo);
        }

        // Позиционируем панель рядом с курсором
        panel.position = screenPos + new Vector2(10f, -10f);
        gameObject.SetActive(true);
    }

    /// <summary>Упрощённая версия — только названия без колбеков</summary>
    public void Show(string[] actionNames, Vector2 screenPos)
    {
        Show(actionNames, screenPos, null);
    }

    public void Hide()
    {
        gameObject.SetActive(false);
    }

    private void Update()
    {
        // Закрываем меню при нажатии Escape или ЛКМ в другом месте
        if (gameObject.activeSelf)
        {
            if (Input.GetKeyDown(KeyCode.Escape))
                Hide();
        }
    }
}
```

---

## Код: TreeInteractable.cs (пример интерактивного объекта)

```csharp
// Assets/Scripts/VoronoiMapGenerator/TreeInteractable.cs
using UnityEngine;

public class TreeInteractable : MonoBehaviour, IInteractable
{
    [SerializeField] private int woodAmount = 3;
    private bool _harvested = false;

    public string[] GetContextActions(PlayerController player)
    {
        // TODO: проверить есть ли топор у игрока
        bool hasAxe = false; // упрощённо

        return hasAxe
            ? new[] { "Собрать дерево", "Удар", "Осмотреть" }
            : new[] { "Осмотреть", "Удар (медленно)" };
    }

    public void Interact(PlayerController player)
    {
        // По умолчанию — стандартное взаимодействие
        Debug.Log($"Взаимодействие с деревом. Дерева: {woodAmount}");
        // TODO: добавить в инвентарь
    }

    public void Chop()
    {
        if (_harvested) return;
        _harvested = true;
        Debug.Log($"Срублено дерево, получено дерева: {woodAmount}");
        // TODO: добавить Wood x{woodAmount} в инвентарь
        Destroy(gameObject, 0.5f);
    }
}
```

---

## Настройка Rigidbody для персонажа

На объекте Player:
1. Add Component → `Rigidbody`
2. Mass: `70`
3. Drag: `5` (торможение)
4. Angular Drag: `0.5`
5. Use Gravity: ✅
6. Is Kinematic: ❌
7. Freeze Rotation: X ✅, Y ✅, Z ✅ (персонаж не опрокидывается)

На объекте Player также:
- Add Component → `Capsule Collider`
  - Height: `2`
  - Radius: `0.4`
  - Center: Y = `1`

---

## Настройка Input (через старую систему)

Edit → Project Settings → Input Manager

Оси уже настроены по умолчанию:
- `Horizontal` — A/D и стрелки
- `Vertical` — W/S и стрелки
- `Mouse ScrollWheel` — колесо мыши
- `Mouse X`, `Mouse Y` — движение мыши

Дополнительные кнопки задаём через `Input.GetKeyDown(KeyCode.X)` прямо в коде.

---

## Настройка Main Camera для следования за игроком

Если используешь Main Camera напрямую (не дочернюю):

Вариант 1: PlayerController управляет камерой (код выше, `cameraTransform = Camera.main.transform`)

Вариант 2: Отдельный скрипт

```csharp
// CameraFollow.cs — вешается на Main Camera
using UnityEngine;

public class CameraFollow : MonoBehaviour
{
    [SerializeField] private Transform target;
    [SerializeField] private float distance  = 20f;
    [SerializeField] private float height    = 15f;
    [SerializeField] private float smoothing = 5f;

    private float _yaw = 0f;

    private void LateUpdate()
    {
        if (target == null) return;

        if (Input.GetKey(KeyCode.Q)) _yaw -= 90f * Time.deltaTime;
        if (Input.GetKey(KeyCode.E)) _yaw += 90f * Time.deltaTime;

        float scroll = Input.GetAxis("Mouse ScrollWheel");
        distance = Mathf.Clamp(distance - scroll * 5f, 5f, 50f);

        Quaternion rotation = Quaternion.Euler(0f, _yaw, 0f);
        Vector3 offset      = rotation * new Vector3(0f, height, -distance);
        Vector3 desired     = target.position + offset;

        transform.position = Vector3.Lerp(transform.position, desired, smoothing * Time.deltaTime);
        transform.LookAt(target.position + Vector3.up * 1.5f);
    }
}
```

---

## Чеклист Модуля Д

```
□ PlayerController.cs создан и прикреплён к Player
□ Движение WASD работает (FixedUpdate + Rigidbody)
□ Shift увеличивает скорость
□ Q/E вращают камеру
□ Колесо мыши приближает/удаляет
□ Камера всегда следует за игроком
□ ЛКМ наносит урон животным
□ ПКМ открывает контекстное меню
□ C открывает крафт
□ Space автоподбирает предметы
□ Pause открывает паузу
□ F11 открывает экран проигрыша
□ При IsExhausted скорость ×0.5
□ ContextMenuPanel показывает правильные пункты
```
