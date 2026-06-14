# Tag — Component Spec

## Призначення

Статична, не-інтерактивна мітка для відображення категорій, характеристик рослин
та системних позначок (наприклад, "згенеровано AI").
Для інтерактивних сценаріїв (фільтрація) — компонент `Chip`.

---

## Варіанти

| Variant   | Поверхня                                                                  |
|-----------|----------------------------------------------------------------------------|
| `default` | page bg, WHITE-картка                                                       |
| `white`   | translucent WHITE/MATTE-картка — AI-мітки, low-emphasis теги               |

**Usage constraint для `white`:** дизайн спирається на накладання прозоростей
(`--tag-white-bg` = `rgba(255,255,255,0.48)` сидить на картці з тим самим токеном
→ ефективно ~73% білого, border стає візуально непомітним). На суцільному page bg
(без translucent card-обгортки) той самий токен дасть видимий білий pill з помітним
border — інший вигляд, ніж на скріні-референсі. Використовувати `white` лише всередині
`.card-white` / `.card-matte`.

---

## Розміри

| Size | Font size token       | px  | Контекст використання              |
|------|-----------------------|-----|-------------------------------------|
| `s`  | `--font-size-xxs`     | 9px | PlantCard, PlantCardRow             |
| `l`  | `--font-size-s`       | 12px| Деталь-сторінка, основний контент   |

---

## Props

```typescript
import { type LucideIcon } from 'lucide-react';
import { type ReactNode } from 'react';

type TagSize = 's' | 'l';
type TagVariant = 'default' | 'white';

type TagProps = {
  size: TagSize;
  variant?: TagVariant; // default: 'default'
  icon?: LucideIcon;
  children: ReactNode;
  className?: string;
};
```

---

## DOM-структура

```html
<!-- без іконки -->
<span class="tag tag--s">
  <span class="tag__text">Монстера</span>
</span>

<!-- з іконкою -->
<span class="tag tag--l">
  <svg class="tag__icon" aria-hidden="true" />
  <span class="tag__text">Розсіяне світло</span>
</span>

<!-- variant white, на card-white, приклад: AI-мітка -->
<span class="tag tag--l tag--white">
  <svg class="tag__icon" aria-hidden="true" /> <!-- lucide Star, outline -->
  <span class="tag__text">згенеровано AI</span>
</span>
```

---

## Поведінка

- `white-space: nowrap` на кореневому `<span>` — тег ніколи не переноситься
- `text-overflow: ellipsis` — реалізується через вкладений `.text` span:
  - `.tag` → `overflow: hidden; max-width: var(--tag-max-width)`
  - `.text` → `overflow: hidden; text-overflow: ellipsis; white-space: nowrap; min-width: 0`
- `--tag-max-width` за замовчуванням `none`; батьківський контекст перевизначає через CSS cascade
- Порожній `children` — не обробляється компонентом, відповідальність батька
- Іконка без тексту — не підтримується, не в scope

---

## Токени (globals.css — секція `=== TAG ===`)

```css
/* === TAG === */
--tag-radius: var(--radius-full);
--tag-max-width: none;

/* size s */
--tag-font-size-s: var(--font-size-xxs);
--tag-padding-s: 1px 6px;
--tag-icon-size-s: 10px;
--tag-icon-gap-s: 3px;

/* size l */
--tag-font-size-l: var(--font-size-s);
--tag-padding-l: 2px 7px;
--tag-icon-size-l: 12px;
--tag-icon-gap-l: 4px;

/* default */
--tag-color: var(--color-secondary);
--tag-bg: var(--color-base-linear-gradient);
--tag-shadow: inset 0 0 0 0.5px var(--color-secondary-border);

/* white */
--tag-white-color: var(--color-text-secondary);
--tag-white-bg: var(--color-inverse-muted-3);
--tag-white-shadow: inset 0 0 0 0.5px var(--color-inverse-muted-4);
```

---

## CSS-правила (Tag.module.css)

```css
.tag {
  display: inline-flex;
  align-items: center;
  font-family: var(--font-sans);
  font-weight: var(--font-weight-normal);
  line-height: 1.5;
  border-radius: var(--tag-radius);
  white-space: nowrap;
  user-select: none;
  overflow: hidden;
  max-width: var(--tag-max-width);

  color: var(--tag-color);
  background: var(--tag-bg);
  box-shadow: var(--tag-shadow); /* НЕ border — box-shadow: inset */
}

/* --- size variants --- */

.s {
  font-size: var(--tag-font-size-s);
  padding: var(--tag-padding-s);
}

.l {
  font-size: var(--tag-font-size-l);
  padding: var(--tag-padding-l);
}

/* --- white --- */

.white {
  color: var(--tag-white-color);
  background: var(--tag-white-bg);
  box-shadow: var(--tag-white-shadow);
}

/* --- icon --- */

.icon {
  flex-shrink: 0;
  stroke: currentColor;
  fill: none;
  stroke-width: 1.75;
  stroke-linecap: round;
  stroke-linejoin: round;
}

.s .icon {
  width: var(--tag-icon-size-s);
  height: var(--tag-icon-size-s);
  margin-right: var(--tag-icon-gap-s);
}

.l .icon {
  width: var(--tag-icon-size-l);
  height: var(--tag-icon-size-l);
  margin-right: var(--tag-icon-gap-l);
}

/* --- text --- */

.text {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  min-width: 0;
}
```

---

## Файлова структура

| Файл                                      | Відповідальність                              |
|---------------------------------------------|-------------------------------------------------|
| `components/ui/Tag/Tag.tsx`               | Компонент                                     |
| `components/ui/Tag/Tag.module.css`        | Стилі                                         |
| `components/ui/Tag/index.ts`              | Re-export: `export { Tag } from './Tag'`      |
| `globals.css`                             | Додати секцію `=== TAG ===` з токенами        |
| `app/dev/tag/page.tsx`                    | Dev-сторінка                                  |
| `app/dev/tag/TagDemo.module.css`          | Стилі dev-сторінки                            |

---

## Dev-сторінка

Матриця: **4 колонки** (без іконки · short / без іконки · long / з іконкою · short / з іконкою · long)
× **2 рядки** (size `s` / size `l`)
× **2 секції**:

- `default` — на page bg (без card-обгортки)
- `white` — всередині `.card-white` (демонструє накладання прозоростей), включає реальний кейс AI-мітки

Демо-контент:

**default:**
- short без іконки: `Монстера`
- long без іконки: `Розсіяне світло`
- short з іконкою: `<Moon />` + `Тінь`
- long з іконкою: `<Droplets />` + `Ґрунт підсох на 3 см`

**white:**
- short без іконки: `AI`
- long без іконки: `Перевірено`
- short з іконкою: `<Star />` + `AI`
- long з іконкою: `<Star />` + `згенеровано AI` ← реальний кейс з DescriptionCard
