# StepProgress

Індикатор прогресу у multi-step flow. Чистий UI primitive — не знає контексту де використовується.

Дизайн: `step_progress_design.html`

---

## Файлова структура

```
components/ui/StepProgress/
  StepProgress.types.ts     — StepProgressType, StepSegmentState
  StepProgress.tsx          — компонент
  StepProgress.module.css   — стилі
  index.ts                  — реекспорт

app/dev/step-progress/
  page.tsx                  — dev-сторінка
  StepProgressDemo.module.css
```

**globals.css** — додати нову секцію `STEP PROGRESS`, видалити `--step-popup-progress-*` токени з секції `STEP POPUP`.

---

## Types

```ts
// StepProgress.types.ts

export type StepProgressType = 'fill' | 'stages';

export type StepSegmentState = 'done' | 'active' | 'upcoming';
```

**`fill`** — 2 логічних стани. Всі сегменти ≤ `currentStep` → `done` або `active`, колір однаковий. Поточний крок виглядає як вже заповнений.

**`stages`** — 3 логічних стани. Сегменти < `currentStep` → `done`, = `currentStep` → `active`, > `currentStep` → `upcoming`. Поточний крок візуально виділений.

---

## Props

```ts
// StepProgress.tsx

export interface StepProgressProps {
  totalSteps: number;
  currentStep: number;        // 1-indexed. Clamp 1..totalSteps всередині компонента.
  type?: StepProgressType;    // default: 'fill'
  ariaLabel?: string;
}
```

---

## Логіка

`currentStep` clamp перед рендером:

```ts
const clampedStep = Math.min(Math.max(currentStep, 1), totalSteps);
```

Стан сегмента (`i` — 1-indexed):

```ts
// type: 'fill'
const resolveSegmentStateFill = (i: number, current: number): StepSegmentState =>
  i <= current ? 'done' : 'upcoming';
// active ніколи не виставляється — fill не розрізняє done і active візуально

// type: 'stages'
const resolveSegmentStateStages = (i: number, current: number): StepSegmentState =>
  i < current ? 'done' : i === current ? 'active' : 'upcoming';
```

`data-state` на кожному сегменті → CSS керує кольором без жодної логіки в компоненті.

---

## Токени → globals.css

Замінюють `--step-popup-progress-*` (ті 4 токени видаляються з секції `STEP POPUP`).

```css
/* === STEP PROGRESS === */
--progress-height: 2px;
--progress-gap:    4px;
--progress-radius: var(--radius-full);

/* fill — FilterPopup (light bg) */
--step-progress-fill-upcoming: rgba(122, 136, 120, 0.18);
--step-progress-fill-done:     var(--color-accent-main);
--step-progress-fill-active:   var(--color-accent-main);

/* stages — SignupForm (dark/forest bg) */
--step-progress-stages-upcoming: var(--color-inverse-muted-2);
--step-progress-stages-done:     var(--color-inverse-muted-4);
--step-progress-stages-active:   var(--color-accent-light);
```

---

## StepProgress.module.css

```css
.track {
  display: flex;
  gap: var(--progress-gap);
}

.segment {
  flex: 1;
  height: var(--progress-height);
  border-radius: var(--progress-radius);
  transition: background-color var(--duration-fast) var(--easing-base);
}

@media (prefers-reduced-motion: reduce) {
  .segment {
    transition: none;
  }
}

/* fill */
.fill[data-state='upcoming'] { background: var(--step-progress-fill-upcoming); }
.fill[data-state='done']     { background: var(--step-progress-fill-done); }
.fill[data-state='active']   { background: var(--step-progress-fill-active); }

/* stages */
.stages[data-state='upcoming'] { background: var(--step-progress-stages-upcoming); }
.stages[data-state='done']     { background: var(--step-progress-stages-done); }
.stages[data-state='active']   { background: var(--step-progress-stages-active); }
```

---

## Accessibility

```tsx
role="progressbar"
aria-valuenow={clampedStep}
aria-valuemin={1}
aria-valuemax={totalSteps}
aria-label={ariaLabel}
```

---

## Використання

```tsx
// FilterPopup — тип fill, 6 кроків
<StepProgress totalSteps={6} currentStep={step} ariaLabel="Крок фільтру" />

// SignupForm — тип stages, 3 кроки
<StepProgress totalSteps={3} currentStep={step} type="stages" ariaLabel="Крок реєстрації" />
```

---

## Dev-сторінка

`app/dev/step-progress/page.tsx`

Дві секції — `fill` і `stages`. Кожна секція показує всі можливі `currentStep` на реальному контексті.

**fill** — білий фон (відтворює FilterPopup), 6 сегментів. Колонки: крок 1/6, 3/6, 6/6.

**stages** — forest-градієнт фон (відтворює SignupForm), 3 сегменти. Колонки: крок 1/3, 2/3, 3/3.

Мітки над кожною колонкою: `currentStep / totalSteps`. Компонент без зайвих обгорток — просто `<StepProgress>` на фоні.

---

## Міграція

### globals.css

Видалити 4 токени з секції `STEP POPUP`:

```css
/* видалити */
--step-popup-progress-h: 2px;
--step-popup-progress-bg: rgba(122, 136, 120, 0.2);
--step-popup-progress-active: var(--color-accent-main);
--step-popup-progress-gap: 4px;
```

Додати нову секцію `STEP PROGRESS` (токени описані вище).

### FilterStep.tsx

`progress-row` — flex-рядок із сегментами та кнопкою закриття — зараз рендериться вручну div-сегментами. Замінити сегменти на `<StepProgress>`:

```tsx
// БУЛО — ручні div-сегменти
<div className={styles.progressRow}>
  <div className={styles.dots}>
    {Array.from({ length: totalSteps }, (_, i) => (
      <div key={i} className={mergeClassNames(styles.dot, i < stepIndex && styles.dotDone)} />
    ))}
  </div>
  <button className={styles.close} onClick={onClose}>…</button>
</div>

// СТАЛО
<div className={styles.progressRow}>
  <StepProgress
    totalSteps={totalSteps}
    currentStep={stepIndex + 1}
    ariaLabel="Крок фільтру"
  />
  <button className={styles.close} onClick={onClose}>…</button>
</div>
```

`type` не передається — дефолт `'fill'` відповідає поведінці FilterPopup.

### FilterStep.module.css

Видалити стилі `.dots`, `.dot`, `.dotDone` — вони переходять в `StepProgress.module.css` через токени.

### filter-popup-spec.md

Секцію "Progress bar" оновити: замінити опис ручної розмітки на референс `<StepProgress type="fill" totalSteps={6} currentStep={step} ariaLabel="Крок фільтру" />`.
