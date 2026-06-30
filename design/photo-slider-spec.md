# ui/PhotoSlider — Специфікація

Чистий viewer-примітив для відображення масиву фото.
Без логіки завантаження файлів, видалення чи сортування — тільки рендер і навігація.

---

## Файлова структура

```
components/
  ui/
    PhotoSlider/
      PhotoSlider.tsx        ← компонент
      PhotoSlider.types.ts   ← PhotoSliderProps
      styles.module.css      ← всі стилі + токени компонента
      index.ts               ← re-export

app/
  dev/
    photo-slider/
      page.tsx               ← DevPage: всі варіанти і стани
      styles.module.css
```

**Оновлюються:**
- `components/ui/index.ts` — додати `export { default as PhotoSlider } from './PhotoSlider'`
- `app/dev/page.tsx` — додати посилання на `/dev/photo-slider`

---

## Props

```ts
// PhotoSlider.types.ts
export interface PhotoSliderProps {
  /** URL-и зображень (https:// або object URL для локальних файлів) */
  photos: string[];
  /** Початковий активний індекс (uncontrolled). Default: 0 */
  defaultActiveIndex?: number;
  /** Controlled: активний індекс ззовні */
  activeIndex?: number;
  /** Controlled: викликається при зміні активного індексу */
  onActiveIndexChange?: (index: number) => void;
}
```

---

## Внутрішній стан і controlled/uncontrolled

```tsx
const [internalIndex, setInternalIndex] = useState(
  clampIndex(defaultActiveIndex ?? 0, photos.length)
);

const isControlled = activeIndex !== undefined;
const currentIndex = isControlled
  ? clampIndex(activeIndex, photos.length)
  : internalIndex;

function handleChange(index: number) {
  if (!isControlled) setInternalIndex(index);
  onActiveIndexChange?.(index);
}
```

`clampIndex` — чиста функція в `PhotoSlider.tsx`, не виносити в окремий файл:
```ts
function clampIndex(index: number, length: number): number {
  if (length === 0) return 0;
  return Math.min(Math.max(index, 0), length - 1);
}
```

---

## Error стан зображення

Loading state — відповідальність батька (TanStack Query / Skeleton).
`PhotoSlider` рендериться тільки коли батько вже має URL-и.

Error state: `onError` на `<img>` приховує зламане зображення,
показується `CameraFallbackIcon` з `icons/ui/CameraFallbackIcon.tsx`.

```tsx
const [errorIndexes, setErrorIndexes] = useState<Set<number>>(new Set());

function handleImageError(index: number) {
  setErrorIndexes(prev => new Set(prev).add(index));
}
```

```tsx
<div className={styles.imageWrap}>
  {errorIndexes.has(currentIndex) ? (
    <div className={styles.placeholder} aria-hidden="true">
      <CameraFallbackIcon />
    </div>
  ) : (
    <img
      src={photos[currentIndex]}
      alt=""
      className={styles.image}
      onError={() => handleImageError(currentIndex)}
    />
  )}
</div>
```

Перевага: кешує які індекси вже зламані — при поверненні до зламаного фото
не намагається завантажити ще раз.

---

## Навігація — thumb click

```tsx
function handleThumbClick(index: number) {
  handleChange(index);
}
```

---

## Навігація — keyboard

```tsx
function handleKeyDown(event: KeyboardEvent<HTMLDivElement>) {
  if (photos.length <= 1) return;
  if (event.key === 'ArrowLeft') {
    event.preventDefault();
    handleChange(currentIndex === 0 ? photos.length - 1 : currentIndex - 1);
  }
  if (event.key === 'ArrowRight') {
    event.preventDefault();
    handleChange((currentIndex + 1) % photos.length);
  }
}
```

---

## Навігація — swipe (pointer events)

```tsx
const swipeStartX = useRef(0);

function handlePointerDown(e: PointerEvent<HTMLDivElement>) {
  if (!e.isPrimary) return;                          // ігноруємо другий палець (pinch-zoom)
  swipeStartX.current = e.clientX;
  e.currentTarget.setPointerCapture(e.pointerId);   // отримуємо події навіть поза межами елемента
}

function handlePointerUp(e: PointerEvent<HTMLDivElement>) {
  if (!e.isPrimary) return;
  const delta = e.clientX - swipeStartX.current;
  if (Math.abs(delta) < 50 || photos.length <= 1) return;
  if (delta < 0) {
    handleChange((currentIndex + 1) % photos.length);
  } else {
    handleChange(currentIndex === 0 ? photos.length - 1 : currentIndex - 1);
  }
}
```

`useRef` — зберігає `startX` між events без re-render. Виправданий.

В CSS на `.slider`:
```css
touch-action: pan-y;   /* вертикальний scroll сторінки не блокується */
user-select: none;     /* не виділяє текст під час свайпу */
```

Прив'язати до кореневого `.slider`: `onPointerDown` + `onPointerUp`.

---

## Анатомія (DOM-структура)

```html
<div
  class="slider"
  role="region"
  aria-label="Фото товару"
  tabindex="0"
  onKeyDown onPointerDown onPointerUp
>
  <!-- Main image -->
  <div class="imageWrap">
    <!-- loaded state -->
    <img class="image" src="..." alt="" onError />

    <!-- error state (замість img) -->
    <div class="placeholder" aria-hidden="true">
      <CameraFallbackIcon />
    </div>
  </div>

  <!-- Counter (тільки коли photos.length > 1) -->
  <div class="counter" aria-live="polite">2 / 5</div>

  <!-- Thumbs strip (тільки коли photos.length > 1) -->
  <div class="thumbs">
    <button type="button" class="thumb" aria-pressed="false" onClick>
      <img src="..." alt="" class="thumbImage" />
    </button>
    <button type="button" class="thumb" aria-pressed="true" onClick>
      <img src="..." alt="" class="thumbImage" />
    </button>
  </div>
</div>
```

---

## CSS токени (в `styles.module.css`)

```css
/* ─── Динамічні токени (inject з батька через style prop): немає ─── */

/* ─── Компонентні токени ─── */
.slider {
  /* container */
  --photo-slider-radius: var(--radius-lg);
  --photo-slider-bg: var(--color-inverse-muted-3);
  --photo-slider-aspect-ratio: 1 / 1;

  /* counter */
  --photo-slider-counter-top: 12px;
  --photo-slider-counter-left: 14px;
  --photo-slider-counter-font-size: 11px;
  --photo-slider-counter-lh: 1.5;
  --photo-slider-counter-color: rgba(255, 255, 255, 0.85);
  --photo-slider-counter-bg: rgba(0, 0, 0, 0.22);
  --photo-slider-counter-padding: 3px 9px;

  /* thumbs strip */
  --photo-slider-thumbs-bottom: 12px;
  --photo-slider-thumbs-gap: 6px;

  /* thumb */
  --photo-slider-thumb-size: 48px;
  --photo-slider-thumb-radius: var(--radius-sm);
  --photo-slider-thumb-inactive-opacity: 0.55;
  --photo-slider-thumb-border-default: 1px solid var(--color-inverse-muted-4);
  --photo-slider-thumb-border-active: 1px solid var(--color-accent-main);

  /* placeholder (error) */
  --photo-slider-placeholder-size: 100px;
}
```

---

## Стилі (поведінка)

```css
.slider {
  position: relative;
  border-radius: var(--photo-slider-radius);
  background: var(--photo-slider-bg);
  box-shadow: inset 0 0 0 var(--border-size) var(--color-inverse-muted-4);
  aspect-ratio: var(--photo-slider-aspect-ratio);
  overflow: hidden;
  outline: none;
  touch-action: pan-y;
  user-select: none;
}

.slider:focus-visible {
  box-shadow:
    inset 0 0 0 var(--border-size) var(--color-inverse-muted-4),
    0 0 0 2px var(--color-accent-deep);
}

.imageWrap {
  position: absolute;
  inset: 0;
}

.image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.placeholder {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
}

/* CameraFallbackIcon вже має вбудований opacity="0.50" */

.counter {
  position: absolute;
  top: var(--photo-slider-counter-top);
  left: var(--photo-slider-counter-left);
  z-index: 2;
  font-size: var(--photo-slider-counter-font-size);
  font-weight: var(--font-weight-normal);
  line-height: var(--photo-slider-counter-lh);
  color: var(--photo-slider-counter-color);
  background: var(--photo-slider-counter-bg);
  backdrop-filter: blur(6px);
  -webkit-backdrop-filter: blur(6px);
  border-radius: var(--radius-full);
  padding: var(--photo-slider-counter-padding);
}

.thumbs {
  position: absolute;
  bottom: var(--photo-slider-thumbs-bottom);
  left: 50%;
  transform: translateX(-50%);
  z-index: 2;
  display: flex;
  gap: var(--photo-slider-thumbs-gap);
}

.thumb {
  width: var(--photo-slider-thumb-size);
  height: var(--photo-slider-thumb-size);
  border-radius: var(--photo-slider-thumb-radius);
  border: var(--photo-slider-thumb-border-default);
  overflow: hidden;
  cursor: pointer;
  flex-shrink: 0;
  backdrop-filter: blur(4px);
  -webkit-backdrop-filter: blur(4px);
  padding: 0;
  display: block;
  opacity: var(--photo-slider-thumb-inactive-opacity);
  transition: opacity var(--duration-fast) var(--easing-base),
              border var(--duration-fast) var(--easing-base);
}

.thumb[aria-pressed="true"] {
  opacity: 1;
  border: var(--photo-slider-thumb-border-active);
}

.thumb:focus-visible {
  outline: 2px solid var(--color-accent-deep);
  outline-offset: 2px;
}

.thumbImage {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}
```

**Увага:** thumb `border` — не `box-shadow: inset`. Свідомий виняток із загального
правила "all visible borders = inset box-shadow". Зафіксовано тут явно.

---

## Стани

| Стан | photos.length | Counter | Thumbs | Image |
|------|--------------|---------|--------|-------|
| Single | 1 | прихований | прихований | `<img>` |
| Multiple | 2–5 | `"N / total"` | видимі | `<img>` |
| Error | будь-яке | за кількістю | за кількістю | прихований, `CameraFallbackIcon` |
| Empty `[]` | 0 | прихований | прихований | `CameraFallbackIcon` |

---

## Accessibility

- `role="region"` + `aria-label="Фото товару"` на кореневому `<div>`
- `tabIndex={0}` на кореневому `<div>` — слайдер фокусується як один елемент
- `<button type="button" aria-pressed={isActive}>` на кожному thumb
- `aria-live="polite"` на counter — screen reader оголошує зміну фото
- `alt=""` на всіх зображеннях — декоративні, підпис несе `aria-label` регіону
- `outline: none` + явний `:focus-visible` — не приховувати focus для keyboard

---

## Що не робимо

- Не приймаємо `className` — позиціонування через батьківський wrapper
- Не використовуємо `next/image` — примітив universal, `blob:` URL сумісний
- Не додаємо `useCallback`/`useMemo` без реальної потреби
- Не хардкодимо значення в CSS — тільки токени
- Не реалізуємо loading state — відповідальність батька
- Не виносимо `clampIndex` в окремий файл — тривіальна, internal
- Thumb `border` — не `box-shadow: inset` (свідомий виняток, задокументований вище)
