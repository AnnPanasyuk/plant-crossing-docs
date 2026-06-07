# Tooltip

`@radix-ui/react-tooltip` + CSS animations. Desktop only — trigger прихований на `< 1024px`.

## Файлова структура

```
components/ui/Tooltip/
  Tooltip.types.ts       — TooltipPlacement
  Tooltip.tsx            — TooltipProvider, Tooltip
  TooltipInfoTrigger.tsx — Info icon trigger
  Tooltip.module.css
  index.ts
```

## Токени → globals.css

```css
/* === TOOLTIP === */
--tooltip-bg:                 rgba(95, 107, 93, 0.95);
--tooltip-color:              #FFFFFF;
--tooltip-radius:             var(--radius-sm);
--tooltip-font-size:          var(--font-size-s);
--tooltip-padding:            8px 12px;
--tooltip-max-width:          220px;
--tooltip-arrow-size:         6px;
--tooltip-shadow:             0 4px 20px rgba(74, 90, 72, 0.20);
--tooltip-trigger-size:       20px;
--tooltip-trigger-color:      var(--color-secondary);
--tooltip-trigger-hover-bg:   rgba(122, 136, 120, 0.12);
--tooltip-trigger-focus-ring: 0 0 0 2px var(--color-accent-deep);
```

Z-INDEX секція — додати токен вище за всі overlay.

## Tooltip.types.ts

```ts
export type TooltipPlacement = 'top' | 'bottom' | 'left' | 'right';
```

## Tooltip.tsx

```tsx
import * as RadixTooltip from '@radix-ui/react-tooltip';
import { ReactElement } from 'react';

import { TooltipPlacement } from './Tooltip.types';
import styles from './Tooltip.module.css';

export interface TooltipProps {
  content: string;
  children: ReactElement;
  placement?: TooltipPlacement; // default: 'top'
  delayDuration?: number;       // default: 400
}

export const TooltipProvider = RadixTooltip.Provider;

export const Tooltip = ({
  content,
  children,
  placement = 'top',
  delayDuration = 400,
}: TooltipProps): ReactElement => (
  <RadixTooltip.Root delayDuration={delayDuration}>
    {children}
    <RadixTooltip.Portal>
      <RadixTooltip.Content className={styles.content} side={placement} sideOffset={8}>
        {content}
        <RadixTooltip.Arrow className={styles.arrow} />
      </RadixTooltip.Content>
    </RadixTooltip.Portal>
  </RadixTooltip.Root>
);
```

## TooltipInfoTrigger.tsx

```tsx
import * as RadixTooltip from '@radix-ui/react-tooltip';
import { Info } from 'lucide-react';
import { ReactElement } from 'react';

import styles from './Tooltip.module.css';

export interface TooltipInfoTriggerProps {
  label: string; // aria-label
}

export const TooltipInfoTrigger = ({ label }: TooltipInfoTriggerProps): ReactElement => (
  <RadixTooltip.Trigger asChild>
    <button type="button" className={styles.trigger} aria-label={label}>
      <Info className={styles.triggerIcon} aria-hidden="true" />
    </button>
  </RadixTooltip.Trigger>
);
```

## index.ts

```ts
export { Tooltip, TooltipProvider } from './Tooltip';
export type { TooltipProps } from './Tooltip';

export { TooltipInfoTrigger } from './TooltipInfoTrigger';
export type { TooltipInfoTriggerProps } from './TooltipInfoTrigger';
```

## Tooltip.module.css

```css
/* Trigger */
.trigger {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: var(--tooltip-trigger-size);
  height: var(--tooltip-trigger-size);
  border-radius: 50%;
  border: none;
  background: transparent;
  color: var(--tooltip-trigger-color);
  cursor: help;
  padding: 0;
  flex-shrink: 0;
  transition: background var(--duration-fast) var(--easing-base);
}

.trigger:hover        { background: var(--tooltip-trigger-hover-bg); }
.trigger:focus-visible { outline: none; box-shadow: var(--tooltip-trigger-focus-ring); }

@media (max-width: 1023px) {
  .trigger { display: none; }
}

.triggerIcon { width: 14px; height: 14px; }

/* Bubble */
.content {
  background: var(--tooltip-bg);
  color: var(--tooltip-color);
  font-size: var(--tooltip-font-size);
  line-height: 1.55;
  padding: var(--tooltip-padding);
  border-radius: var(--tooltip-radius);
  max-width: var(--tooltip-max-width);
  box-shadow: var(--tooltip-shadow);
  z-index: var(--z-tooltip);
}

.content[data-state='delayed-open'] { animation: tooltipIn var(--duration-fast) var(--easing-enter); }
.content[data-state='closed']       { animation: tooltipOut var(--duration-fast) var(--easing-exit); }

@keyframes tooltipIn {
  from { opacity: 0; transform: translateY(4px); }
  to   { opacity: 1; transform: translateY(0); }
}

@keyframes tooltipOut {
  from { opacity: 1; transform: translateY(0); }
  to   { opacity: 0; transform: translateY(4px); }
}

.arrow { fill: var(--tooltip-bg); }
```

## Використання

```tsx
// app/layout.tsx — один раз
import { TooltipProvider } from '@plant-crossing/components/ui/Tooltip';

<TooltipProvider>{children}</TooltipProvider>
```

```tsx
// FilterStep.tsx
import { Tooltip, TooltipInfoTrigger } from '@plant-crossing/components/ui/Tooltip';

{tooltip && (
  <Tooltip content={tooltip}>
    <TooltipInfoTrigger label="Підказка" />
  </Tooltip>
)}
```
