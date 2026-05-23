# Motion & Loading

Принципи анімації і стани завантаження платформи.
Анімації підсилюють UI — не відволікають від рослин.

---

## Принципи

- Анімація має бути непомітною — юзер не повинен думати про неї
- `sm` і `md` (touch): мінімальні анімації, без hover
- `lg` (desktop): повні анімації і hover стани
- Ніколи не блокувати UI під час async операцій

---

## Базові токени

| Токен | Значення | Використання |
|-------|----------|--------------|
| `--duration-fast` | `120ms` | Hover, fade кнопок |
| `--duration-base` | `200ms` | Більшість переходів |
| `--duration-slow` | `350ms` | Модалі, drawer, попапи |
| `--duration-enter` | `400ms` | Появлення великих елементів |
| `--easing-base` | `cubic-bezier(0.4, 0, 0.2, 1)` | Загальний (Material standard) |
| `--easing-enter` | `cubic-bezier(0, 0, 0.2, 1)` | Елементи що з'являються |
| `--easing-exit` | `cubic-bezier(0.4, 0, 1, 1)` | Елементи що зникають |
| `--easing-spring` | `cubic-bezier(0.34, 1.56, 0.64, 1)` | Toast shake, пружні ефекти |

---

## Компоненти — анімації

### PlantCard

```css
/* hover тільки на desktop */
@media (hover: hover) {
  .card:hover {
    transform: translateY(-2px);
    background: rgba(255, 255, 255, 0.68);
    border-color: rgba(255, 255, 255, 0.92);
    /* без box-shadow */
  }
}

/* скидаємо на touch щоб не залипав */
@media (hover: none) {
  .card:hover {
    transform: none;
    background: var(--card-bg);
    border-color: var(--card-border);
  }
}

.card {
  transition: transform var(--duration-base) var(--easing-base),
              background var(--duration-base) var(--easing-base),
              border-color var(--duration-base) var(--easing-base);
  overflow: visible;       /* не hidden — уникаємо layout pass при transform */
  will-change: transform;  /* compositor layer заздалегідь */
}

/* overflow тільки на фото, не на картці */
.photo {
  overflow: hidden;
  border-radius: var(--radius-lg) var(--radius-lg) 0 0;
}

/* wishlist — spring scale */
.wishlist:hover {
  transform: scale(1.18);
  transition: transform var(--duration-fast) var(--easing-spring);
}
```

- Без `box-shadow` на hover — фон світлішає замість тіні
- `overflow: hidden` на `.card` тригерить layout pass при `transform` — переноситься на `.photo`
- `will-change: transform` прибрати після підтвердження плавності в браузері

---

### CartDrawer / Bottom Sheet
```css
/* desktop — slide from right */
.drawer {
  transform: translateX(100%);
  transition: transform var(--duration-slow) var(--easing-enter);
}
.drawer.open {
  transform: translateX(0);
}

/* mobile — slide from bottom */
.bottom-sheet {
  transform: translateY(100%);
  transition: transform var(--duration-slow) var(--easing-enter);
}
.bottom-sheet.open {
  transform: translateY(0);
}
```

---

### AccountPopup
```css
.popup {
  opacity: 0;
  transform: scale(0.96) translateY(-4px);
  transition: opacity var(--duration-base) var(--easing-enter),
              transform var(--duration-base) var(--easing-enter);
}
.popup.open {
  opacity: 1;
  transform: scale(1) translateY(0);
}
```

---

### FilterPopup
Розпливається від низу — scale по обох осях одночасно.
```css
.filter-popup {
  transform-origin: bottom center;
  transform: scale(0, 0);
  opacity: 0;
  transition: transform var(--duration-slow) var(--easing-enter),
              opacity var(--duration-fast) var(--easing-enter);
}
.filter-popup.open {
  transform: scale(1, 1);
  opacity: 1;
}
```

---

### EditProfileModal / Будь-який модал
```css
.backdrop {
  opacity: 0;
  transition: opacity var(--duration-base) var(--easing-base);
}
.backdrop.open { opacity: 1; }

.modal {
  opacity: 0;
  transform: scale(0.97) translateY(8px);
  transition: opacity var(--duration-slow) var(--easing-enter),
              transform var(--duration-slow) var(--easing-enter);
}
.modal.open {
  opacity: 1;
  transform: scale(1) translateY(0);
}
```

---

### Toast
Залітає зверху + shake після приземлення.
```css
@keyframes toast-enter {
  0%   { transform: translateY(-100%); opacity: 0; }
  60%  { transform: translateY(0);     opacity: 1; }
  75%  { transform: translateX(4px); }
  85%  { transform: translateX(-4px); }
  92%  { transform: translateX(2px); }
  100% { transform: translateX(0); }
}

.toast {
  animation: toast-enter 500ms var(--easing-spring) forwards;
}
```

---

### Tab перемикання (Profile)
```css
.tab-indicator {
  transition: transform var(--duration-base) var(--easing-base),
              width var(--duration-base) var(--easing-base);
}
```
Підкреслення ковзає між табами — не миготить.

---

### Button
```css
.btn {
  transition: background var(--duration-fast) var(--easing-base),
              opacity var(--duration-fast) var(--easing-base),
              transform var(--duration-fast) var(--easing-base);
}
.btn:hover  { background: var(--color-accent-rose); }
.btn:active { transform: scale(0.98); }
```

---

### StickyCTA — поява при скролі
```css
.sticky-cta {
  opacity: 0;
  transform: translateY(100%);
  transition: opacity var(--duration-base) var(--easing-enter),
              transform var(--duration-base) var(--easing-enter);
}
.sticky-cta.visible {
  opacity: 1;
  transform: translateY(0);
}
```
Показується коли основні кнопки дій виходять з viewport.

---

## Loading стани

### Skeleton з shimmer
Використовується як `fallback` в React `<Suspense>`.
Показується рідко завдяки Next.js Server Components streaming.

```css
@keyframes shimmer {
  0%   { background-position: -400px 0; }
  100% { background-position:  400px 0; }
}

.skeleton {
  background: linear-gradient(
    90deg,
    var(--color-base-warm) 25%,
    var(--color-neutral-sand) 50%,
    var(--color-base-warm) 75%
  );
  background-size: 800px 100%;
  animation: shimmer 1.4s infinite linear;
  border-radius: var(--radius-md);
}
```

**Варіанти skeleton:**
- `PlantCard` skeleton — фото блок + 3 лінії тексту
- `PlantDetails` skeleton — ліва колонка + права колонка
- `OrderItem` skeleton — маленьке фото + 2 лінії
- `ProfileHeader` skeleton — круг аватара + 2 лінії

---

### Page-level loading
`loading.js` в Next.js App Router — автоматичний.
Progress bar у хедері під час навігації між сторінками.

```css
@keyframes page-progress {
  0%  { width: 0%; }
  20% { width: 30%; }
  50% { width: 60%; }
  80% { width: 85%; }
  95% { width: 95%; }
}

.page-progress {
  height: 2px;
  background: var(--color-accent-dusty);
  animation: page-progress 2.5s ease-out forwards;
  position: absolute;
  bottom: 0;
  left: 0;
}
```

---

### useTransition — inline async дії
Для фільтрів, wishlist, swap, будь-яких дій що не є навігацією.
UI не блокується — кнопка залишається активною.

```tsx
const [isPending, startTransition] = useTransition()

const handleFilter = () => {
  startTransition(async () => {
    const res = await fetchPlants(filters)
    setPlants(res)
  })
}

return (
  <button disabled={isPending}>
    {isPending
      ? <Spinner size="sm" />
      : 'Застосувати фільтр'
    }
  </button>
)
```

**Правило:** `useTransition` скрізь де є async дія без зміни URL. `loading.js` — тільки для навігації між сторінками.

---

### Button loading стан
Для submit кнопок (checkout, signup, збереження форми).
```tsx
<Button disabled={isPending}>
  {isPending
    ? <><Spinner size="sm" /> Зберігаємо...</>
    : 'Зберегти'
  }
</Button>
```

---

## Що не робимо

- Глобальний spinner/overlay — замінено на `useTransition` + skeleton
- Анімації що тривають довше `400ms` для UI елементів
- Hover анімації на `sm` і `md`
- `transition: all` — завжди перераховуємо конкретні властивості
- Scroll-triggered анімації — post-MVP
- Page transitions між роутами — post-MVP
