# ButtonWithTooltip — специфікація

## Контекст

Компонент об'єднує `Button` і `Tooltip` в єдину одиницю.
Паралельно оновлюється API `Tooltip` — `RadixTooltip.Trigger asChild` переноситься всередину `Tooltip.tsx`.
`TooltipInfoTrigger.tsx` видаляється — новий API `Tooltip` робить його зайвим.

---

## Зміни до існуючих компонентів

### `Tooltip.tsx` — ОНОВИТИ

`RadixTooltip.Trigger asChild` переноситься всередину компонента.
`children` автоматично стає trigger-елементом — жодних змін на місцях використання, окрім оновлення `TooltipInfoTrigger`.

```tsx
type RadixSide = 'top' | 'bottom' | 'left' | 'right';
type RadixAlign = 'start' | 'center' | 'end';

const parsePlacement = (placement: TooltipPlacement): { side: RadixSide; align: RadixAlign } => {
  const [side, align] = placement.split('-') as [RadixSide, RadixAlign | undefined];
  return { side, align: align ?? 'center' };
};

export const Tooltip = ({
  content,
  children,
  placement = 'top',
  delayDuration = 400,
}: TooltipProps): ReactElement => {
  const { side, align } = parsePlacement(placement);
  return (
    <RadixTooltip.Root delayDuration={delayDuration}>
      <RadixTooltip.Trigger asChild>
        {children}
      </RadixTooltip.Trigger>
      <RadixTooltip.Portal>
        <RadixTooltip.Content className={styles.content} side={side} align={align} sideOffset={8}>
          {content}
          <RadixTooltip.Arrow className={styles.arrow} />
        </RadixTooltip.Content>
      </RadixTooltip.Portal>
    </RadixTooltip.Root>
  );
};
```

### `TooltipInfoTrigger.tsx` — ВИДАЛИТИ

Більше не потрібен. `Tooltip` сам загортає children у Trigger.

### `Tooltip/index.ts` — ОНОВИТИ

Прибрати експорт `TooltipInfoTrigger` і `TooltipInfoTriggerProps`.

```ts
export { Tooltip, TooltipProvider } from './Tooltip';
export type { TooltipProps } from './Tooltip';
```

### Оновити місця використання `TooltipInfoTrigger`

Замінити патерн:
```tsx
// БУЛО
<Tooltip content={tooltip}>
  <TooltipInfoTrigger label="Підказка" />
</Tooltip>

// СТАЛО
<Tooltip content={tooltip}>
  <Button variant="plain" size="sm" iconLeft={<Info />} aria-label="Підказка" />
</Tooltip>
```

Стилі `.trigger` / `.triggerIcon` з `Tooltip.module.css` — видалити повністю (більше не потрібні).

---

## Новий компонент: `ButtonWithTooltip`

### Props

```ts
// Button/ButtonWithTooltip.types.ts
import type { TooltipPlacement } from '../Tooltip/Tooltip.types';
import type { ButtonProps } from './Button';

export interface ButtonWithTooltipProps extends ButtonProps {
  tooltipContent: string;
  tooltipPlacement?: TooltipPlacement; // default: 'top'
  tooltipDelay?: number;               // default: 400
}
```

`TooltipPlacement` тепер включає `top | top-start | top-end | bottom | bottom-start | bottom-end | left | left-start | left-end | right | right-start | right-end`.

### Логіка рендеру

| Умова | Поведінка |
|---|---|
| `tooltipContent` порожній рядок | Рендерить plain `<Button>` без будь-якого tooltip |
| `loading={true}` | Рендерить plain `<Button loading>` без tooltip |
| `disabled` (і не `loading`) | `<Tooltip><span className={styles.disabledWrapper}><Button disabled /></span></Tooltip>` |
| Решта | `<Tooltip><Button /></Tooltip>` |

**Чому span для `disabled`:**
Нативний `disabled` знімає pointer-events — тултіп не спрацює.
`<span className={styles.disabledWrapper}>` перехоплює hover, кнопка всередині залишається disabled.

### `ButtonWithTooltip.tsx`

```tsx
import { ReactElement } from 'react';

import { Tooltip } from '../Tooltip';
import { Button } from './Button';
import type { ButtonWithTooltipProps } from './ButtonWithTooltip.types';
import styles from './ButtonWithTooltip.module.css';

export const ButtonWithTooltip = ({
  tooltipContent,
  tooltipPlacement = 'top',
  tooltipDelay = 400,
  disabled,
  loading,
  ...buttonProps
}: ButtonWithTooltipProps): ReactElement => {
  const showTooltip = Boolean(tooltipContent) && !loading;

  if (!showTooltip) {
    return <Button {...buttonProps} disabled={disabled} loading={loading} />;
  }

  if (disabled) {
    return (
      <Tooltip content={tooltipContent} placement={tooltipPlacement} delayDuration={tooltipDelay}>
        <span className={styles.disabledWrapper}>
          <Button {...buttonProps} disabled />
        </span>
      </Tooltip>
    );
  }

  return (
    <Tooltip content={tooltipContent} placement={tooltipPlacement} delayDuration={tooltipDelay}>
      <Button {...buttonProps} />
    </Tooltip>
  );
};
```

### `ButtonWithTooltip.module.css`

```css
.disabledWrapper {
  display: inline-flex;
}
```

---

## Файлова структура

```
components/ui/
  Tooltip/
    Tooltip.types.ts          — без змін
    Tooltip.tsx               — ОНОВИТИ: Trigger всередині
    TooltipInfoTrigger.tsx    — ВИДАЛИТИ
    Tooltip.module.css        — ОНОВИТИ: прибрати .trigger / .triggerIcon стилі
    index.ts                  — ОНОВИТИ: прибрати TooltipInfoTrigger exports
  Button/
    Button.tsx                — без змін
    Button.module.css         — без змін
    ButtonWithTooltip.tsx     — СТВОРИТИ
    ButtonWithTooltip.types.ts — СТВОРИТИ
    ButtonWithTooltip.module.css — СТВОРИТИ
    index.ts                  — ОНОВИТИ: додати ButtonWithTooltip export

app/dev/
  button-with-tooltip/
    page.tsx                  — СТВОРИТИ
    ButtonWithTooltipDemo.module.css — СТВОРИТИ
  page.tsx                    — ОНОВИТИ: додати посилання
```

---

## Dev-сторінка: `app/dev/button-with-tooltip/page.tsx`

Колонки: `Default` · `Disabled` · `Loading` · `Failure`
Рядки покривають усі сценарії:

| Рядок | tooltipContent | Примітка |
|---|---|---|
| Icon-only Primary | "Додати до вішліста" | placement top |
| Icon-only Ghost | "Назад" | placement top |
| Icon-only Danger | "Видалити" | placement top |
| Text Primary | "Власник отримає сповіщення" | placement top |
| Text Ghost | "Скасувати дію" | placement top |
| Placement bottom | "Підказка знизу" | icon-only ghost |
| Placement left | "Підказка зліва" | icon-only ghost |
| Empty tooltip | "" | рендерить plain Button |

Фон секцій: page-bg (для Primary/Ghost/Danger), pink gradient (для White/Secondary).
Для `interactive →` станів hover/focus — інтерактивні елементи, підписані меткою.

---

## Заборонено

- `style` в TSX — тільки CSS Modules
- Дублювати логіку Tooltip (позиціонування, анімації) — тільки через `<Tooltip>`
- Хардкод кольорів та відступів
- Inline union types — тільки named types
- Створювати файли не зі списку
