# StepPopup

Generic wizard-попап. Не знає про домен — тільки layout, progress, навігація.
Використовується в `CatalogFilterPopup` і `SwapPopup`.

`@radix-ui/react-dialog` керує lifecycle, portal, focus trap, Escape, backdrop click.

## Файлова структура

```
components/ui/StepPopup/
  StepPopup.types.ts    — StepPopupDirection, StepPopupProps
  StepPopup.tsx
  StepPopup.module.css
  index.ts

components/ui/FilterStep/
  FilterStep.types.ts   — StepDirection
  FilterStep.tsx
  FilterStep.module.css
  index.ts
```

## Токени → globals.css

```css
/* === STEP POPUP === */
--step-popup-width:           480px;
--step-popup-radius:          var(--radius-xl);
--step-popup-bg:              #FFFFFF;
--step-popup-padding-x:       20px;
--step-popup-progress-h:      2px;
--step-popup-progress-bg:     rgba(122, 136, 120, 0.20);
--step-popup-progress-active: var(--color-accent-main);
--step-popup-progress-gap:    4px;
--step-popup-footer-border:   var(--border-size) solid rgba(122, 136, 120, 0.10);
--step-popup-backdrop-bg:     rgba(28, 28, 28, 0.35);
--step-popup-close-size:      28px;
--step-popup-nav-btn-size:    28px;
--step-popup-nav-btn-border:  var(--border-size) solid rgba(122, 136, 120, 0.25);
```

Z-INDEX секція:
```css
--z-step-popup-backdrop: /* підібрати: вище --z-filter-btn, нижче --z-header */
--z-step-popup:          /* вище --z-step-popup-backdrop */
```

## StepPopup.types.ts

```ts
import { ReactNode } from 'react';

export type StepPopupDirection = 'forward' | 'back';

export interface StepPopupProps {
  open: boolean;
  onClose: () => void;
  totalSteps: number;
  currentStep: number;          // 0-based
  direction: StepPopupDirection;
  children: ReactNode;
  onBack: () => void;
  onNext: () => void;
  onSkip: () => void;
  isFirst: boolean;
  isLast: boolean;
  onSubmit: () => void;
  onReset: () => void;
  submitLabel: string;
  submitDisabled?: boolean;
  submitLoading?: boolean;
}
```

## StepPopup.tsx

```tsx
import * as Dialog from '@radix-ui/react-dialog';
import { ChevronLeft, ChevronRight, RotateCcw, X } from 'lucide-react';
import { ReactElement } from 'react';

import { StepPopupProps } from './StepPopup.types';
import styles from './StepPopup.module.css';

export const StepPopup = ({
  open,
  onClose,
  totalSteps,
  currentStep,
  direction,
  children,
  onBack,
  onNext,
  onSkip,
  isFirst,
  isLast,
  onSubmit,
  onReset,
  submitLabel,
  submitDisabled = false,
  submitLoading = false,
}: StepPopupProps): ReactElement => (
  <Dialog.Root open={open} onOpenChange={(isOpen) => { if (!isOpen) onClose(); }}>
    <Dialog.Portal>
      <Dialog.Overlay className={styles.backdrop} />
      <Dialog.Content className={styles.popup} aria-label="Фільтри каталогу">

        <div className={styles.topBar}>
          <div className={styles.progress} aria-label={`Крок ${currentStep + 1} з ${totalSteps}`}>
            {Array.from({ length: totalSteps }).map((_, i) => (
              <span key={i} className={i <= currentStep ? styles.progressDotActive : styles.progressDot} />
            ))}
          </div>
          <Dialog.Close asChild>
            <button type="button" className={styles.closeButton} aria-label="Закрити">
              <X className={styles.closeIcon} aria-hidden="true" />
            </button>
          </Dialog.Close>
        </div>

        <div key={currentStep} className={direction === 'forward' ? styles.bodyForward : styles.bodyBack}>
          {children}
        </div>

        <footer className={styles.footer}>
          <button
            type="button"
            className={styles.navButton}
            onClick={onBack}
            disabled={isFirst}
            aria-label="Назад"
          >
            <ChevronLeft className={styles.navIcon} aria-hidden="true" />
          </button>

          {isLast ? (
            <>
              <button type="button" className={styles.resetButton} onClick={onReset}>
                <RotateCcw className={styles.resetIcon} aria-hidden="true" />
                Скинути все
              </button>
              <button
                type="button"
                className={styles.submitButton}
                onClick={onSubmit}
                disabled={submitDisabled || submitLoading}
              >
                {submitLoading
                  ? <span className={styles.spinner} aria-label="Завантаження" />
                  : submitLabel
                }
              </button>
            </>
          ) : (
            <>
              <button type="button" className={styles.skipButton} onClick={onSkip}>
                Пропустити
              </button>
              <button
                type="button"
                className={styles.navButton}
                onClick={onNext}
                aria-label="Далі"
              >
                <ChevronRight className={styles.navIcon} aria-hidden="true" />
              </button>
            </>
          )}
        </footer>

      </Dialog.Content>
    </Dialog.Portal>
  </Dialog.Root>
);
```

## StepPopup.module.css

```css
/* Backdrop */
.backdrop {
  position: fixed;
  inset: 0;
  background: var(--step-popup-backdrop-bg);
  z-index: var(--z-step-popup-backdrop);
}

.backdrop[data-state='open']   { animation: fadeIn var(--duration-base) var(--easing-enter); }
.backdrop[data-state='closed'] { animation: fadeOut var(--duration-fast) var(--easing-exit); }

/* Popup */
.popup {
  position: fixed;
  bottom: 24px;
  left: 50%;
  transform: translateX(-50%);
  width: var(--step-popup-width);
  background: var(--step-popup-bg);
  border-radius: var(--step-popup-radius);
  box-shadow: var(--shadow-drawer);
  z-index: var(--z-step-popup);
  overflow: hidden;
}

.popup[data-state='open']   { animation: popupIn var(--duration-slow) var(--easing-enter); }
.popup[data-state='closed'] { animation: popupOut var(--duration-base) var(--easing-exit); }

/* Top bar: progress + close */
.topBar {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 14px var(--step-popup-padding-x) 10px;
}

.progress {
  flex: 1;
  display: flex;
  gap: var(--step-popup-progress-gap);
}

.progressDot,
.progressDotActive {
  flex: 1;
  height: var(--step-popup-progress-h);
  border-radius: var(--radius-full);
}

.progressDot       { background: var(--step-popup-progress-bg); }
.progressDotActive { background: var(--step-popup-progress-active); }

/* Close */
.closeButton {
  flex-shrink: 0;
  width: var(--step-popup-close-size);
  height: var(--step-popup-close-size);
  border-radius: 50%;
  border: none;
  background: transparent;
  color: var(--color-text-secondary);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: background var(--duration-fast) var(--easing-base);
}

.closeButton:hover { background: rgba(122, 136, 120, 0.10); }
.closeIcon { width: 14px; height: 14px; }

/* Step body — key={currentStep} triggers re-mount → animation */
.bodyForward { animation: slideInForward var(--duration-base) var(--easing-enter); }
.bodyBack    { animation: slideInBack var(--duration-base) var(--easing-enter); }

/* Footer */
.footer {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px var(--step-popup-padding-x) 16px;
  border-top: var(--step-popup-footer-border);
}

.navButton {
  width: var(--step-popup-nav-btn-size);
  height: var(--step-popup-nav-btn-size);
  border-radius: 50%;
  border: var(--step-popup-nav-btn-border);
  background: transparent;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: background var(--duration-fast) var(--easing-base);
}

.navButton:hover:not(:disabled) { background: rgba(122, 136, 120, 0.08); }
.navButton:disabled { opacity: 0.35; cursor: not-allowed; }
.navIcon { width: 13px; height: 13px; stroke: var(--color-text-secondary); }

.skipButton {
  font-size: var(--font-size-s);
  color: var(--color-text-secondary);
  background: transparent;
  border: none;
  cursor: pointer;
  padding: 0;
}

.resetButton {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  font-size: var(--font-size-s);
  color: var(--color-text-secondary);
  background: transparent;
  border: none;
  cursor: pointer;
  padding: 0;
}

.resetIcon { width: 12px; height: 12px; }

.submitButton {
  display: inline-flex;
  align-items: center;
  padding: 7px 16px;
  border-radius: var(--radius-full);
  border: none;
  background: linear-gradient(160deg, var(--color-accent-light), var(--color-accent-main));
  color: var(--color-text-inverse);
  font-size: var(--font-size-m);
  cursor: pointer;
  transition: opacity var(--duration-fast) var(--easing-base);
}

.submitButton:disabled { opacity: 0.5; cursor: not-allowed; }

.spinner {
  display: inline-block;
  width: 14px;
  height: 14px;
  border: 1.5px solid rgba(255, 255, 255, 0.35);
  border-top-color: #FFFFFF;
  border-radius: 50%;
  animation: spin 600ms linear infinite;
}

/* Keyframes */
@keyframes fadeIn {
  from { opacity: 0; }
  to   { opacity: 1; }
}

@keyframes fadeOut {
  from { opacity: 1; }
  to   { opacity: 0; }
}

@keyframes popupIn {
  from { opacity: 0; transform: translateX(-50%) translateY(12px) scale(0.97); }
  to   { opacity: 1; transform: translateX(-50%) translateY(0) scale(1); }
}

@keyframes popupOut {
  from { opacity: 1; transform: translateX(-50%) translateY(0) scale(1); }
  to   { opacity: 0; transform: translateX(-50%) translateY(12px) scale(0.97); }
}

@keyframes slideInForward {
  from { opacity: 0; transform: translateX(12px); }
  to   { opacity: 1; transform: translateX(0); }
}

@keyframes slideInBack {
  from { opacity: 0; transform: translateX(-12px); }
  to   { opacity: 1; transform: translateX(0); }
}

@keyframes spin {
  to { transform: rotate(360deg); }
}
```

## index.ts

```ts
export { StepPopup } from './StepPopup';
export type { StepPopupProps, StepPopupDirection } from './StepPopup.types';
```

## Використання

```tsx
import { StepPopup } from '@plant-crossing/components/ui/StepPopup';
import { FilterStep } from '@plant-crossing/components/ui/FilterStep';

<StepPopup
  open={isOpen}
  onClose={handleClose}
  totalSteps={STEPS.length}
  currentStep={stepIdx}
  direction={direction}
  onBack={handleBack}
  onNext={handleNext}
  onSkip={handleNext}
  isFirst={stepIdx === 0}
  isLast={stepIdx === STEPS.length - 1}
  onSubmit={handleApply}
  onReset={handleReset}
  submitLabel={submitLabel}
  submitDisabled={resultCount === 0 || resultLoading}
  submitLoading={resultLoading}
>
  <FilterStep title={step.title} tooltip={step.tooltip}>
    {/* крок */}
  </FilterStep>
</StepPopup>
```
