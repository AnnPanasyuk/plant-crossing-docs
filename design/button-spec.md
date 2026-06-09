# Button — специфікація компонента

## Огляд

Єдиний компонент для всіх інтерактивних кнопок платформи. Шість варіантів (`variant`) покривають всі сценарії використання.

---

## Props

```ts
type ButtonVariant = 'primary' | 'white' | 'secondary' | 'ghost' | 'danger' | 'plain'
type ButtonSize    = 'sm' | 'md' | 'lg'

interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?:   ButtonVariant   // default: 'primary'
  size?:      ButtonSize      // default: 'md'
  loading?:   boolean
  failure?:   boolean
  iconLeft?:  ReactElement
  iconRight?: ReactElement
  children?:  ReactNode
}
```

`loading={true}` → shimmer-анімація, `disabled` автоматично, курсор `default`.  
`failure={true}` → failure-стиль варіанту; можна комбінувати з `disabled`.  
`disabled` → стан disabled, курсор `not-allowed`.

---

## Висота і бордер — критична деталь

Базовий `border: 0.5px solid transparent` на всіх кнопках без винятку — забезпечує однакову висоту.

**Видимі бордери — тільки через `box-shadow: inset 0 0 0 0.5px <color>`.**  
`border-color` завжди `transparent`. `box-shadow` не впливає на box model → стабільна висота на всіх браузерах.

Застосовується до: secondary (всі стани), ghost (всі стани), failure-стани primary / white / secondary / ghost / danger.

---

## Розміри

| Size | font-size | padding      | icon size |
|------|-----------|--------------|-----------|
| `sm` | 12px      | 6px 10px     | 13px      |
| `md` | 14px      | 8px 14px     | 15px      |
| `lg` | 14px      | 10px 18px    | 15px      |

`border-radius: var(--radius-full)` — всі розміри.  
Icon-only: `padding: 9px` (md), квадратна форма.  
`font-weight: 400` — всі варіанти.  
`gap: 6px` між іконкою та текстом.

---

## Варіанти і стани

### Primary
Головна дія. На page-bg.

| Стан     | background                                             | color                        | box-shadow                              |
|----------|--------------------------------------------------------|------------------------------|-----------------------------------------|
| default  | `linear-gradient(135deg, accent-light, accent-main)`   | `--color-inverse`            | —                                       |
| hover    | `linear-gradient(135deg, #C4828F, #B46B79)`            | `--color-inverse`            | —                                       |
| focus    | `linear-gradient(135deg, accent-light, accent-main)`   | `--color-inverse`            | `0 0 0 2px var(--color-accent-deep)`    |
| disabled | `linear-gradient(135deg, rgba(212,145,158,.42), rgba(196,122,136,.42))` | `--color-inverse-muted-3` | —               |
| loading  | shimmer поверх primary gradient                        | `transparent`                | —                                       |
| failure  | `--color-error-secondary`                              | `--color-error-primary`      | `inset 0 0 0 0.5px --color-error-primary` |

---

### White
Головна дія на кольорових поверхнях (pink card, forest bg).

| Стан     | background                  | color                       | box-shadow                                                      |
|----------|-----------------------------|-----------------------------|------------------------------------------------------------------|
| default  | `--color-inverse`           | `--color-accent-main`       | —                                                                |
| hover    | `#F5F3F4`                   | `#B46B79`                   | —                                                                |
| focus    | `--color-inverse`           | `--color-accent-main`       | `0 0 0 2px rgba(255,255,255,.6)`                                 |
| disabled | `rgba(255,255,255,.55)`     | `rgba(196,122,136,.45)`     | —                                                                |
| loading  | shimmer поверх white bg     | `transparent`               | —                                                                |
| failure  | `rgba(255,255,255,.88)`     | `--color-error-primary`     | `inset 0 0 0 0.5px --color-error-muted, 0 0 0 2px rgba(255,255,255,.6)` |

---

### Secondary
Додаткова дія на кольорових поверхнях. Frosted glass.

| Стан     | background                    | color                         | box-shadow (inset)                                                    |
|----------|-------------------------------|-------------------------------|-----------------------------------------------------------------------|
| default  | `--color-inverse-muted-1`     | `--color-inverse`             | `inset 0 0 0 0.5px --color-inverse-muted-3`                          |
| hover    | `--color-inverse-muted-2`     | `--color-inverse`             | `inset 0 0 0 0.5px --color-inverse-muted-4`                          |
| focus    | `--color-inverse-muted-1`     | `--color-inverse`             | `inset 0 0 0 0.5px --color-inverse-muted-3, 0 0 0 2px rgba(255,255,255,.3)` |
| disabled | `rgba(255,255,255,0.07)`      | `rgba(255,255,255,0.35)`      | `inset 0 0 0 0.5px rgba(255,255,255,0.20)`                           |
| loading  | shimmer поверх secondary bg   | `transparent`                 | `inset 0 0 0 0.5px --color-inverse-muted-3`                          |
| failure  | `rgba(191,61,82,.2)`          | `--color-inverse`             | `inset 0 0 0 0.5px rgba(191,61,82,.5)`                              |

---

### Ghost
Навігація, скасування. На page-bg.

| Стан     | background                    | color                         | box-shadow                                                                    |
|----------|-------------------------------|-------------------------------|-------------------------------------------------------------------------------|
| default  | `transparent`                 | `--color-secondary`           | `inset 0 0 0 0.5px --color-secondary-border`                                 |
| hover    | `rgba(122,136,120,.1)`        | `--color-secondary-border`    | `inset 0 0 0 0.5px --color-secondary-border`                                 |
| focus    | `transparent`                 | `--color-secondary-border`    | `inset 0 0 0 0.5px --color-secondary-border, 0 0 0 2px rgba(122,136,120,.25)` |
| disabled | `transparent`                 | `rgba(122,136,120,.38)`       | `inset 0 0 0 0.5px rgba(122,136,120,.35)`                                    |
| loading  | `transparent`                 | `transparent`                 | `inset 0 0 0 0.5px --color-secondary-border`                                 |
| failure  | `transparent`                 | `--color-error-primary`       | `inset 0 0 0 0.5px --color-error-primary`                                    |

Ghost loading: shimmer через `::after` з `position: absolute; inset: 0; overflow: hidden`.

---

### Danger
Деструктивна дія.

| Стан     | background                                          | color                     | box-shadow                               |
|----------|-----------------------------------------------------|---------------------------|------------------------------------------|
| default  | `linear-gradient(135deg, #BF3D52, #A02840)`         | `--color-inverse`         | —                                        |
| hover    | `linear-gradient(135deg, #A02840, #841A2E)`         | `--color-inverse`         | —                                        |
| focus    | `linear-gradient(135deg, #BF3D52, #A02840)`         | `--color-inverse`         | `0 0 0 2px rgba(191,61,82,0.35)`         |
| disabled | `linear-gradient(135deg, rgba(191,61,82,.32), rgba(160,40,64,.32))` | `rgba(255,255,255,.4)` | —              |
| loading  | shimmer поверх danger gradient                      | `transparent`             | —                                        |
| failure  | `--color-error-muted`                               | `--color-error-primary`   | `inset 0 0 0 0.5px --color-error-primary` |

---

### Plain
Tertiary-дія без візуального навантаження. "Пропустити крок", "Скинути фільтр".  
Не має loading і failure станів.

| Стан     | background                     | color                          | box-shadow                              |
|----------|--------------------------------|--------------------------------|-----------------------------------------|
| default  | `transparent`                  | `--color-text-secondary`       | —                                       |
| hover    | `rgba(122,136,120,.08)`        | `#5A6858`                      | —                                       |
| focus    | `transparent`                  | `--color-text-secondary`       | `0 0 0 2px rgba(122,136,120,.25)`       |
| disabled | `transparent`                  | `rgba(122,136,120,.38)`        | —                                       |

---

## Loading shimmer — реалізація

```css
@keyframes shimmer {
  0%   { background-position: 200% center; }
  100% { background-position: -200% center; }
}
```

Primary / White / Danger loading: shimmer-шар `linear-gradient(90deg, transparent 0%, rgba(255,255,255,.2) 50%, transparent 100%)` з `background-size: 200% 100%` накладається поверх основного background через multi-layer background.

Ghost loading: shimmer через `::after { position: absolute; inset: 0; }`.

Loading: `color: transparent` — ховає текст зберігаючи розмір кнопки.

---

## CSS токени — додати в globals.css `@theme`

```css
/* ── Button ── */
--btn-primary-bg-from:          var(--color-accent-light);
--btn-primary-bg-to:            var(--color-accent-main);
--btn-primary-color:            var(--color-inverse);
--btn-primary-hover-bg-from:    #C4828F;
--btn-primary-hover-bg-to:      #B46B79;
--btn-primary-focus-box-shadow: 0 0 0 2px var(--color-accent-deep);
--btn-primary-disabled-bg:      linear-gradient(135deg, rgba(212,145,158,.42), rgba(196,122,136,.42));
--btn-primary-disabled-color:   var(--color-inverse-muted-3);
--btn-primary-failure-bg:       var(--color-error-secondary);
--btn-primary-failure-color:    var(--color-error-primary);

--btn-white-bg:                 var(--color-inverse);
--btn-white-color:              var(--color-accent-main);
--btn-white-hover-bg:           #F5F3F4;
--btn-white-hover-color:        #B46B79;
--btn-white-disabled-bg:        rgba(255,255,255,.55);
--btn-white-disabled-color:     rgba(196,122,136,.45);
--btn-white-failure-bg:         rgba(255,255,255,.88);
--btn-white-failure-color:      var(--color-error-primary);

--btn-secondary-bg:             var(--color-inverse-muted-1);
--btn-secondary-color:          var(--color-inverse);
--btn-secondary-hover-bg:       var(--color-inverse-muted-2);
--btn-secondary-disabled-bg:    rgba(255,255,255,0.07);
--btn-secondary-disabled-color: rgba(255,255,255,0.35);
--btn-secondary-failure-bg:     rgba(191,61,82,.2);
--btn-secondary-failure-color:  var(--color-inverse);

--btn-ghost-color:              var(--color-secondary);
--btn-ghost-hover-bg:           rgba(122,136,120,.1);
--btn-ghost-disabled-color:     rgba(122,136,120,.38);
--btn-ghost-failure-color:      var(--color-error-primary);

--btn-danger-bg-from:           var(--color-error-primary);
--btn-danger-bg-to:             #A02840;
--btn-danger-color:             var(--color-inverse);
--btn-danger-hover-from:        #A02840;
--btn-danger-hover-to:          #841A2E;
--btn-danger-focus-box-shadow:  0 0 0 2px rgba(191,61,82,0.35);
--btn-danger-disabled-from:     rgba(191,61,82,.32);
--btn-danger-disabled-to:       rgba(160,40,64,.32);
--btn-danger-disabled-color:    rgba(255,255,255,.4);
--btn-danger-failure-bg:        var(--color-error-muted);
--btn-danger-failure-color:     var(--color-error-primary);

--btn-plain-color:              var(--color-text-secondary);
--btn-plain-hover-bg:           rgba(122,136,120,.08);
--btn-plain-hover-color:        #5A6858;
--btn-plain-focus-color:        var(--color-text-secondary);
--btn-plain-focus-box-shadow:   0 0 0 2px rgba(122,136,120,.25);
--btn-plain-disabled-color:     rgba(122,136,120,.38);
```

---

## Файлова структура

```
src/
  components/
    ui/
      Button/
        Button.tsx          — компонент, props, варіанти
        Button.module.css   — стилі всіх варіантів і станів
        index.ts            — re-export
  app/
    globals.css             — додати токени вище в @theme
```

### Button.tsx — відповідальність
- Приймає `variant`, `size`, `loading`, `iconLeft`, `iconRight`
- Рендерить `<button>` з правильними CSS Module класами
- `loading={true}` → додає `aria-busy="true"`, `disabled`, курсор `default`
- `disabled` → `aria-disabled="true"`
- Icon-only (немає `children`) → додає `aria-label` (обов'язково передати через props)

### Button.module.css — відповідальність
- Базовий клас `.btn`: `border: 0.5px solid transparent`, `border-radius: var(--radius-full)`, `font-weight: 400`, `outline: none`, `position: relative`, `display: inline-flex`, `align-items: center`, `gap: 6px`
- Розміри: `.sm`, `.md`, `.lg`
- Варіанти: `.primary`, `.white`, `.secondary`, `.ghost`, `.danger`, `.plain`
- Всі псевдо-класи через CSS selectors: `:hover:not(:disabled)`, `:focus-visible`, `:disabled`
- **Видимі бордери — виключно через `box-shadow: inset 0 0 0 0.5px <color>`**
- Loading shimmer — `@keyframes shimmer` тут, не в globals

### index.ts
```ts
export { Button } from './Button'
export type { ButtonProps } from './Button'
```

---

## Заборонено

- `border: <visible-color>` в будь-якому стані — тільки `inset box-shadow`
- Hardcode кольори поза токенами
- Окремі компоненти для кожного варіанту
- `useCallback` / `useMemo` — немає потреби
- `'use client'` — Button є client компонентом через event handlers, але тільки якщо є `onClick` або `onSubmit`; якщо передається як Server Component child без handlers — не потрібно
