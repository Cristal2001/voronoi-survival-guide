# 📄 Листок День 3 — Модули Е + Ж (20 баллов)

> ⏱️ Читать 5 минут → переписать нужное на листок → работать
> 💡 Последний день — добить анимации и сдать билд!

---

## 🎬 Модуль Е — Анимации и аудио (12 баллов, 2 ч)

### 🌞 День/Ночь (DayNightCycle)

- Полный цикл = **2 минуты** реального времени
- Ночь: **20:00 – 06:00** (животные не ходят, урон ×0.5)
- Вешать на **Directional Light**

```csharp
float t = _elapsed / 120f;            // 0..1 за 2 минуты
float sunAngle = (t - 0.25f) * 360f;  // полдень при t=0.5
transform.rotation = Quaternion.Euler(sunAngle, -30f, 0f);
sun.intensity = Mathf.Clamp01(Mathf.Sin(t * Mathf.PI));
float gameHour = t * 24f;             // игровые часы 0..24
```

---

### 🎭 Animator — состояния

**Животные (5 состояний):**

```
Idle → Walk  (hasTarget)
            → Chase (isAggressive && enemyNear)
            → Flee  (isNeutral && damaged)
Any State   → Hurt  (hurtTrigger)
Any State   → Death (isDead)
```

**Персонаж (6 состояний):**

```
Idle ↔ Walk  (isMoving)
Walk ↔ Run   (isRunning)
Any  → Attack       (attackTrigger)
Any  → Eat          (eatTrigger)
Any  → WeaponSwitch (switchTrigger, 0.5s)
```

**Параметры Animator для Animal:**

| Параметр | Тип |
|----------|-----|
| `isMoving` | bool |
| `isRunning` | bool |
| `attackTrigger` | trigger |
| `hurtTrigger` | trigger |
| `isDead` | bool |

---

### ✨ UI-эффекты

**Красный экран при уроне (0.3 сек):**

```csharp
IEnumerator DamageFlash() {
    _overlay.color = new Color(1, 0, 0, 0.5f);  // красный
    yield return new WaitForSeconds(0.3f);
    _overlay.color = Color.clear;
}
```

> `_overlay` = полноэкранный Image на Canvas поверх всего

**Числа урона (всплывают и тают):**

- World Space Canvas над врагом
- Text `"-8"` красный
- Coroutine: за 1 сек поднимается на 2 единицы, alpha → 0

**Инвентарь выезжает сверху (0.3 сек):**

```csharp
IEnumerator SlideIn() {
    float t = 0;
    Vector2 start = new Vector2(0, 200f);  // за экраном сверху
    Vector2 end   = Vector2.zero;
    while (t < 0.3f) {
        rt.anchoredPosition = Vector2.Lerp(start, end, t / 0.3f);
        t += Time.unscaledDeltaTime;  // unscaled — работает на паузе!
        yield return null;
    }
    rt.anchoredPosition = end;
}
```

---

### 🔊 Обязательные звуки

| Звук | Когда | AudioSource на объекте |
|------|-------|----------------------|
| GameOver | `ShowGameOver()` | UIManager |
| Building | Размещение постройки | BuildingSystem |

```csharp
[SerializeField] private AudioClip _gameOverClip;

// В ShowGameOver():
GetComponent<AudioSource>().PlayOneShot(_gameOverClip);
```

---

### 📊 Прогресс-бар действий

Появляется над целью при рубке дерева / рыбалке:

- 🟢 Зелёная заливка слева направо
- Текст `"Работаем..."` + иконка
- При отмене: мгновенное красное мигание → исчезает

---

## 🏗️ Модуль Ж — Тестирование и билд (8 баллов, 4 ч)

### 📦 Export Package (после каждого модуля!)

```
Assets → ПКМ → Export Package
→ Include Dependencies ✅
→ Сохранить как: ModuleA.unitypackage
                 ModuleB.unitypackage  ...и т.д.
```

### 🔨 Windows Build

```
File → Build Settings
→ Platform: Windows
→ Add Open Scenes       ← не забыть!
→ Player Settings → Resolution: 1920×1080, Fullscreen
→ Build
→ Проверить что запускается БЕЗ Unity Editor
```

---

### ❗ Частые ошибки Unity 6

| Ошибка | Исправление |
|--------|------------|
| `FindObjectOfType` obsolete | `FindFirstObjectByType<T>()` |
| NullRef на SerializeField | Проверь инспектор — не привязано |
| Shader not found в билде | Build Settings → Always Included Shaders |
| JSON не сохраняется | StreamingAssets в билде read-only на Android, но не на Windows |
| `[Serializable]` забыл | `JsonUtility` не видит класс без атрибута |
| `sharedMaterial` vs `material` | `material` создаёт копию (утечка памяти), используй `sharedMaterial` |
| `Time.deltaTime` на паузе | Используй `Time.unscaledDeltaTime` для UI-анимаций |

---

### ✅ Чеклист перед сдачей

- [ ] Все 7 модулей реализованы
- [ ] Build запускается без ошибок
- [ ] Export Package для каждого модуля
- [ ] StreamingAssets: все 6 JSON файлов на месте
- [ ] Имена классов PascalCase, без русских букв
- [ ] SOLID: новые шаги через новый файл, не трогая старые
- [ ] UI: Canvas, 1920×1080, Scale With Screen Size
- [ ] Все кнопки работают
- [ ] Пауза останавливает время (`Time.timeScale = 0`)
- [ ] Game Over при HP = 0
