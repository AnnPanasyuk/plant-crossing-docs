# Spec: PlantHealthTag

Дизайн: `plant_health_tag.html`

Доменний компонент. Відображає стан здоров'я рослини у вигляді кольорової таблетки: dot + текст.
Без інтерактивності. Без skeleton. Без `className` prop.

---

## Файлова структура

```
components/features/plant/PlantHealthTag/
├── PlantHealthTag.tsx      — компонент
├── styles.module.css       — стилі + всі --plant-health-tag-* токени
├── constants.ts            — PLANT_HEALTH_LABELS
├── types.ts                — PlantHealth type
└── index.ts                — public API

app/dev/plant-health-tag/
├── page.tsx                — dev-сторінка (PlantHealthTagDemoPage)
└── styles.module.css       — стилі dev-сторінки
```

**Файли для оновлення:**
- `app/globals.css` — видалити блок `=== STATUS BADGE ===` повністю (14 рядків: від `/* === STATUS BADGE === */` до `--status-sick-dot` включно)
- `app/dev/page.tsx` — додати посилання на `/dev/plant-health-tag`

---

## Types

**`types.ts`**

```ts
export type PlantHealth = 'healthy' | 'care' | 'sick';
```

Named export. `type`, не `interface` — union.

---

## Constants

**`constants.ts`**

```ts
import type { PlantHealth } from './types';

export const PLANT_HEALTH_LABELS: Record<PlantHealth, string> = {
  healthy: 'Здорова',
  care: 'Потребує догляду',
  sick: 'Потребує лікування',
};
```

Named export.

---

## Компонент

**`PlantHealthTag.tsx`**

```tsx
import type { ReactElement } from 'react';

import { PLANT_HEALTH_LABELS } from './constants';
import styles from './styles.module.css';
import type { PlantHealth } from './types';

interface PlantHealthTagProps {
  status: PlantHealth;
}

export default function PlantHealthTag({ status }: PlantHealthTagProps): ReactElement {
  return (
    <span className={`${styles.root} ${styles[status]}`}>
      <span className={styles.dot} aria-hidden="true" />
      {PLANT_HEALTH_LABELS[status]}
    </span>
  );
}
```

---

## Styles

**`styles.module.css`**

Всі `--plant-health-tag-*` токени оголошуються на `.root` — CSS Modules скопує їх автоматично.
Семантичні кольори (`--color-success-primary` тощо) — посилання на `globals.css`, не дублювати.
`--_color` / `--_bg` / `--_dot` — underscore-prefix: component-private vars, які встановлюють
modifier-класи. Не виносити в globals.css.

```css
.root {
  /* === Component tokens === */
  --plant-health-tag-font-size: 11px;
  --plant-health-tag-padding:   2px 8px;
  --plant-health-tag-dot-size:  6px;
  --plant-health-tag-gap:       5px;
  --plant-health-tag-radius:    var(--radius-full);

  /* healthy */
  --plant-health-tag-healthy-color: var(--color-success-primary);
  --plant-health-tag-healthy-bg:    var(--color-success-secondary);
  --plant-health-tag-healthy-dot:   var(--color-success-primary);

  /* care */
  --plant-health-tag-care-color:    var(--color-warning-primary);
  --plant-health-tag-care-bg:       var(--color-warning-secondary);
  --plant-health-tag-care-dot:      var(--color-warning-primary);

  /* sick */
  --plant-health-tag-sick-color:    var(--color-error-primary);
  --plant-health-tag-sick-bg:       var(--color-error-secondary);
  --plant-health-tag-sick-dot:      var(--color-error-primary);

  /* === Layout === */
  display: inline-flex;
  align-items: center;
  gap: var(--plant-health-tag-gap);
  padding: var(--plant-health-tag-padding);
  border-radius: var(--plant-health-tag-radius);
  font-size: var(--plant-health-tag-font-size);
  font-weight: var(--font-weight-normal);
  line-height: 1.5; /* explicit override від --line-height-base: 1 */
  white-space: nowrap;
  background: var(--_bg);
  color: var(--_color);
}

.dot {
  width: var(--plant-health-tag-dot-size);
  height: var(--plant-health-tag-dot-size);
  border-radius: 50%;
  flex-shrink: 0;
  background: var(--_dot);
}

/* Variant modifiers — встановлюють component-private vars */

.healthy {
  --_color: var(--plant-health-tag-healthy-color);
  --_bg:    var(--plant-health-tag-healthy-bg);
  --_dot:   var(--plant-health-tag-healthy-dot);
}

.care {
  --_color: var(--plant-health-tag-care-color);
  --_bg:    var(--plant-health-tag-care-bg);
  --_dot:   var(--plant-health-tag-care-dot);
}

.sick {
  --_color: var(--plant-health-tag-sick-color);
  --_bg:    var(--plant-health-tag-sick-bg);
  --_dot:   var(--plant-health-tag-sick-dot);
}
```

---

## Re-export

**`index.ts`**

```ts
export { default as PlantHealthTag } from './PlantHealthTag';
export type { PlantHealth } from './types';
```

`PlantHealth` реекспортується через `index.ts` — споживачі не імпортують з `./types` напряму.

---

## Dev-сторінка

**`app/dev/plant-health-tag/page.tsx`**

```tsx
import { PlantHealthTag } from '@plant-crossing/components/features/plant/PlantHealthTag';

import styles from './styles.module.css';

export default function PlantHealthTagDemoPage() {
  return (
    <main className={styles.page}>
      <section className={styles.section}>
        <p className={styles.label}>Варіанти</p>
        <div className={styles.row}>
          <PlantHealthTag status="healthy" />
          <PlantHealthTag status="care" />
          <PlantHealthTag status="sick" />
        </div>
      </section>

      <section className={styles.section}>
        <p className={styles.label}>Контекст · на білій поверхні</p>
        <div className={styles.cardWhite}>
          <PlantHealthTag status="healthy" />
        </div>
      </section>
    </main>
  );
}
```

Секція "Контекст" — картка з `--color-inverse-muted-3` фоном, щоб перевірити
читабельність `healthy` на білій поверхні.

---

## Accessibility

- Зовнішній `<span>` — семантично нейтральний, роль не потрібна.
- `.dot` — `aria-hidden="true"`, декоративний.
- Текст лейблу самостійно conveying meaning.
- Якщо батьківський контекст потребує повного контексту для скрінрідера —
  `aria-label` на обгортці батька, не всередині компонента.

---

## Обмеження

- Не приймає `className`.
- Не приймає `children`.
- Без hover / focus / active — не інтерактивний.
- Не знає де рендериться.
- `status` завжди валідний `PlantHealth` — guard на `null` відсутній,
  батько відповідає за mapping з API.

---

## Variants table

| `status`  | Лейбл               | `--_color`                          | `--_bg`                          | `--_dot`                          |
|-----------|---------------------|-------------------------------------|----------------------------------|-----------------------------------|
| `healthy` | Здорова             | `--plant-health-tag-healthy-color`  | `--plant-health-tag-healthy-bg`  | `--plant-health-tag-healthy-dot`  |
| `care`    | Потребує догляду    | `--plant-health-tag-care-color`     | `--plant-health-tag-care-bg`     | `--plant-health-tag-care-dot`     |
| `sick`    | Потребує лікування  | `--plant-health-tag-sick-color`     | `--plant-health-tag-sick-bg`     | `--plant-health-tag-sick-dot`     |
