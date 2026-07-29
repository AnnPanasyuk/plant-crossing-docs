# Switch / SwitchField — спека

Дизайн-референс: `switch_design.html`
Контекст використання: `ListingForm` → Крок 4 (Ціна / Обмін), рядок «Тільки обмін».

---

## 1. Огляд компонентів

| Компонент | Папка | Відповідальність |
|---|---|---|
| `Switch` | `components/ui/Switch/` | `<button role="switch">`, трек + knob. Не знає про label, hint, error. |
| `SwitchField` | `components/ui/Switch/` | `BaseField` + `Switch`, генерує `id`/aria-звʼязок. |

Колокація `SwitchField` поруч зі `Switch` — за прецедентом `InputField` / `TextareaField`
(`components/ui/Input/`, `components/ui/Textarea/`).

> **Зафіксована розбіжність з `CLAUDE.md`.** Правило «Структура компонентів» вимагає, щоб кожен
> компонент жив у власній папці `ComponentName/`. `*Field`-компоненти — свідомий виняток із цього
> правила (уже діючий для `InputField`, `TextareaField`, `PasswordInputField`, `TelInputField`).
> `CLAUDE.md` потребує уточнення формулювання — див. §12.

**Контрольований, без `defaultChecked`.** `isChecked` + `onChange` обовʼязкові, як в `Input`/`Textarea`.

---

## 2. Семантика та анатомія

```tsx
<button
  type="button"
  role="switch"
  aria-checked={isChecked}
  disabled={isDisabled}
  onClick={() => onChange(!isChecked)}
  className={mergeClassNames(styles.switch, isChecked && styles.checked)}
  {...rest}
>
  <span className={styles.knob} aria-hidden="true" />
</button>
```

**ARIA APG Switch pattern**, не toggle button. `aria-pressed` тут не використовується — скрінрідер
озвучив би контрол як «toggle button, pressed», а не «switch, on/off». Це свідоме відхилення від
поточного формулювання `CLAUDE.md` → «Toggle / multi-select елементи» (§12).

`knob` — `<span aria-hidden>`, суто презентаційний, не інтерактивний.

**Доступне імʼя.** У `<button>` немає текстового вмісту, тому імʼя приходить ззовні:
- через `SwitchField` — автоматично (`BaseField` віддає `id`, label звʼязується);
- через голий `Switch` — **обовʼязково** `aria-labelledby` (id видимого тексту) або `aria-label`.
  Без цього — провал WCAG 4.1.2. Перевагу має `aria-labelledby`, коли видимий текст існує:
  дублювати його в `aria-label` заборонено.

---

## 3. Типи

### `types.ts`

```ts
import type { ComponentPropsWithoutRef } from 'react';

export interface SwitchProps
  extends Omit<
    ComponentPropsWithoutRef<'button'>,
    'onChange' | 'disabled' | 'type' | 'role' | 'aria-checked' | 'children' | 'className'
  > {
  isChecked: boolean;
  onChange: (isChecked: boolean) => void;
  isDisabled?: boolean;
}
```

**Обґрунтування кожного ключа в `Omit`:**

| Ключ | Причина |
|---|---|
| `onChange` | `DOMAttributes` вішає `onChange` на всі елементи як `(e: FormEvent) => void`. Несумісна сигнатура — перетин типів впаде. |
| `disabled` | Друге джерело правди поруч з `isDisabled`. |
| `type` | Завжди `"button"` — інакше сабмітить форму. |
| `role`, `aria-checked` | Компонент проставляє сам, ззовні перевизначати не можна. |
| `children` | Вміст фіксований (knob), слоту немає. |
| `className` | Заборона прокидання стилів ззовні (`CLAUDE.md`). |

`id`, `aria-label`, `aria-labelledby`, `aria-describedby`, `aria-invalid`, `name`, `onFocus`,
`onBlur` тощо **не описуються окремо** — приходять нативно через `ComponentPropsWithoutRef<'button'>`
і прокидаються через `...rest`.

### `types.ts` (продовження)

```ts
export interface SwitchFieldProps
  extends Omit<BaseFieldProps, 'children' | 'counter'>,
    Omit<SwitchProps, 'id' | 'aria-describedby' | 'aria-invalid' | 'aria-required'> {}
```

`counter` відрізаний — булеве поле не рахує символів (симетрично з `TextareaFieldProps`,
де відрізано `children`).

### `SwitchField.tsx`

```tsx
export default function SwitchField({ label, required, hint, error, ...switchProps }: SwitchFieldProps) {
  return (
    <BaseField label={label} required={required} hint={hint} error={error}>
      {(fieldProps) => <Switch {...fieldProps} {...switchProps} />}
    </BaseField>
  );
}
```

`forwardRef` **не потрібен** — програмного фокусу на свіч у жодному зі сценаріїв немає
(на відміну від `Input`, де є `autoFocus` кроків форми). Додається пізніше, якщо зʼявиться кейс.

---

## 4. Поведінка

- **Клік / Space / Enter** → `onChange(!isChecked)`. Space і Enter обробляє нативна `<button>`,
  власних `onKeyDown` не пишемо.
- **`isDisabled`** → нативний `disabled` на кнопці. Події не спрацьовують, з tab-послідовності
  випадає — поведінка нативна, не емулюється.
- **Анімація** — `transform: translateX()` на knob, `background` на треку.
  `left`/`right` **заборонені** (layout-анімація замість композитної).
- **`prefers-reduced-motion: reduce`** → `transition: none` і на knob, і на треку.
- Жодного внутрішнього стану, `useState` всередині немає.

---

## 5. Стани

| # | Стан | Селектор | Що змінюється |
|---|---|---|---|
| 1 | off | `.switch` | `--switch-track-off-bg`, бордер `--switch-track-off-border-color`, knob зліва |
| 2 | off + hover | `.switch:hover:not(:disabled)` | `--switch-track-off-hover-bg` |
| 3 | off + focus-visible | `.switch:focus-visible` | + focus ring другим шаром box-shadow |
| 4 | on | `.switch.checked` | `--switch-track-on-bg`, бордер `--switch-track-on-border-color`, `translateX(--switch-knob-travel)` |
| 5 | on + hover | `.switch.checked:hover:not(:disabled)` | `--switch-track-on-hover-bg` |
| 6 | on + focus-visible | `.switch.checked:focus-visible` | + focus ring |
| 7 | disabled · off | `.switch:disabled` | `opacity: var(--switch-disabled-opacity)`, `cursor: not-allowed` |
| 8 | disabled · on | `.switch.checked:disabled` | те саме поверх on |

Станів `loading` і `error` у самого `Switch` **немає**: стан локальний і синхронний, а помилка —
рівень `SwitchField` (текст під контролом).

`checked` — CSS-клас через `mergeClassNames`, не data-атрибут (узгоджено з `Input`, де
`filled`/`error`/`success` теж класи).

---

## 6. CSS-правила

```css
.switch {
  position: relative;
  display: inline-block;
  flex-shrink: 0;
  width: var(--switch-width);
  height: var(--switch-height);
  padding: 0;
  border: none;
  border-radius: var(--switch-radius);
  background: var(--switch-track-off-bg);
  box-shadow: inset 0 0 0 var(--border-size) var(--switch-track-off-border-color);
  cursor: pointer;
  transition:
    background var(--switch-transition-duration) var(--easing-base),
    box-shadow var(--switch-transition-duration) var(--easing-base);
}
```

- **Бордер — `box-shadow: inset`, ніколи `border`** (правило дизайн-системи).
- **Порядок шарів box-shadow:** бордер завжди першим, focus-ring другим.
- **Hit-area** — псевдоелемент, не збільшення розмірів:

```css
.switch::after {
  content: '';
  position: absolute;
  inset: 50% auto auto 50%;
  transform: translate(-50%, -50%);
  width: 100%;
  min-width: 44px;
  height: 44px;
}
```

Візуально 52×30, зона натискання 52×44. Без цього — 30px по висоті і провал WCAG 2.5.8 на мобільному.

```css
.knob {
  position: absolute;
  top: var(--switch-knob-inset);
  left: var(--switch-knob-inset);
  width: var(--switch-knob-size);
  height: var(--switch-knob-size);
  border-radius: var(--radius-full);
  background: var(--switch-knob-bg);
  box-shadow: var(--switch-knob-shadow);
  transition: transform var(--switch-transition-duration) var(--easing-base);
}

.checked .knob { transform: translateX(var(--switch-knob-travel)); }

@media (prefers-reduced-motion: reduce) {
  .switch, .knob { transition: none; }
}
```

`--switch-knob-travel: 22px` = `width − knob − inset × 2`. Значення фіксоване токеном,
не рахується через `calc()` в рантаймі.

---

## 7. Токени — секція `/* === SWITCH === */` у `globals.css`

```css
/* Геометрія */
--switch-width: 52px;
--switch-height: 30px;
--switch-radius: var(--radius-full);
--switch-knob-size: 24px;
--switch-knob-inset: 3px;
--switch-knob-travel: 22px;

/* Knob */
--switch-knob-bg: var(--color-inverse);
--switch-knob-shadow: 0 1px 3px rgba(0, 0, 0, 0.22);

/* Трек — off */
--switch-track-off-bg: rgba(122, 136, 120, 0.35);
--switch-track-off-hover-bg: rgba(122, 136, 120, 0.5);
--switch-track-off-border-color: var(--color-secondary);

/* Трек — on */
--switch-track-on-bg: linear-gradient(160deg, var(--color-accent-light) 0%, var(--color-accent-main) 100%);
--switch-track-on-hover-bg: linear-gradient(160deg, #c4828f 0%, #b46b79 100%);
--switch-track-on-border-color: var(--color-inverse-muted-2);

/* Стани */
--switch-disabled-opacity: 0.38;
--switch-focus-ring: rgba(212, 145, 158, 0.18);
--switch-focus-ring-size: 2px;
--switch-transition-duration: 180ms;
```

Нових базових кольорів не вводиться. `--switch-track-on-bg` / `--switch-track-on-hover-bg` навмисно
повторюють значення `--chip-selected-bg` / `--chip-selected-hover-bg` — не посилаються на них,
бо це різні компоненти, і звʼязок через чужий namespace створив би приховану залежність.

---

## 8. Доступність — контраст

| Пара | На білому | На `#f2f4f1` | Поріг |
|---|---|---|---|
| off-бордер `#7a8878` → фон | 3.74 ✓ | 3.38 ✓ | 3:1 (1.4.11) |
| off-заливка → фон | 1.49 ✗ | 1.45 ✗ | — |
| білий knob → off-заливка | 1.49 | 1.60 | — |
| білий knob → on-трек | 3.24 ✓ | 3.24 ✓ | 3:1 |

**Носієм контрасту в off-стані є бордер, а не заливка.** Це свідоме рішення: заливка обрана за
візуальним критерієм, вимогу 1.4.11 виконує межа контрола.

Розрізнення on/off спирається на **позицію knob** + зміну треку сірий → рожевий, тобто не тільки на
колір — WCAG 1.4.1 (Use of Color) виконано.

**Зафіксований ризик.** Бордер лишається `var(--border-size)` = 0.5px. На дисплеях з DPR 1 браузер
згладжує субпіксельну лінію, і фактичний контраст просідає нижче розрахункового. Для інших
компонентів 0.5px — косметика, тут — єдиний носій контрасту. Прийнято свідомо; якщо на реальних
пристроях межа виявиться невидимою, рішення — локальний `--switch-track-border-size: 1px`,
не зміна кольору заливки.

---

## 9. Файлова структура

### Створити

```
components/ui/Switch/
  Switch.tsx              — <button role="switch">, трек + knob, без стану
  SwitchField.tsx         — BaseField + Switch
  types.ts                — SwitchProps, SwitchFieldProps
  styles.module.css       — усі 8 станів, hit-area, reduced-motion
  index.ts                — export { default as Switch } from './Switch';
                            export { default as SwitchField } from './SwitchField';
                            export type { SwitchProps, SwitchFieldProps } from './types';

app/dev/switch/
  page.tsx                — default export SwitchDemoPage
  styles.module.css
```

`styles.module.css` і `types.ts` спільні для обох компонентів — `SwitchField` власних стилів не має,
вертикальний layout повністю на `BaseField`.

### Імпорти

- `BaseField` у `SwitchField.tsx` — alias: `@plant-crossing/components/ui/Field`
- `mergeClassNames` — alias з `@plant-crossing/lib/utils` (глобальна утиліта, `CLAUDE.md` →
  «File Organization»). Якщо утиліти за цим шляхом немає — **зупинитись і повідомити**, не писати
  власну і не використовувати `clsx`/`classnames`.
- `Switch` у `SwitchField.tsx` — relative (`./Switch`), та сама папка.
- CSS-імпорт — завжди останнім рядком import-блоку.

### Оновити

```
globals.css               — додати секцію /* === SWITCH === */ (§7)
app/dev/page.tsx          — додати посилання на /dev/switch
CLAUDE.md                 — три блоки правил (§12)
```

---

## 10. Використання в `PriceStep`

Сірий рядок, заголовок «Тільки обмін» і підзаголовок — **не частина `Switch`**, вони живуть у
`PriceStep.module.css`. Компонент не має ні фону рядка, ні відступів контейнера.

```tsx
const swapOnlyId = useId();

<div className={styles.swapRow}>
  <label htmlFor={swapOnlyId} className={styles.swapText}>
    <span className={styles.swapTitle}>Тільки обмін</span>
    <span className={styles.swapHint}>Без ціни — рослина доступна лише для обміну</span>
  </label>
  <Switch id={swapOnlyId} isChecked={swapOnly} onChange={setSwapOnly} />
</div>
```

`<label htmlFor>` — `<button>` є labelable-елементом, тому клік по тексту перемикає свіч нативно,
і доступне імʼя формується без `aria-label`. `aria-labelledby` тут не потрібен.

`SwitchField` у цьому кроці **не використовується** — ні лейбла, ні хінта в розумінні `BaseField`
тут немає, це власний layout рядка.

---

## 11. Dev-сторінка `app/dev/switch/`

Функція — `export default function SwitchDemoPage()`.

Колонки-стани, підписані: `off` / `off + hover` / `off + focus-visible` / `on` / `on + hover` /
`on + focus-visible` / `disabled · off` / `disabled · on`.

**Дві секції фону** — критично саме для цього компонента:
1. на білому (контекст `SwitchField`);
2. на `--info-banner-info-bg` (сірий рядок Кроку 4) — тут перевіряється видимість бордера.

Інтерактивні колонки (`hover`, `focus-visible`) підписані «interactive →» і показуються реальними
станами, не класами-імітаціями.

Окрема секція `SwitchField`: `label` + `hint`, `label` + `required`, `label` + `error`,
`label` + `isDisabled`.

Окрема секція «контекст»: відтворений рядок «Тільки обмін» з `<label htmlFor>` — перевірка того,
що клік по тексту перемикає.

---

## 12. Правки в `CLAUDE.md`

```md
### Композиція типів пропсів
- Пропси компонента ніколи не переписуються вручну, якщо існує тип-джерело.
  Тип збирається перетином: власні пропси + нативні через
  `ComponentPropsWithoutRef<'element'>` + чужі компонентні типи.
- Виключення робиться тільки через `Omit`, з явним переліком ключів. Причини:
  (1) компонент проставляє атрибут сам і ззовні його міняти не можна (`role`,
  `aria-checked`, `type`); (2) нативний проп конкурує з власним і створює друге
  джерело правди (`disabled` vs `isDisabled`); (3) сигнатура нативного пропа
  несумісна з власною (`onChange` на `<button>` приходить з `DOMAttributes`
  як `FormEvent`-handler).
- `*Field`-компоненти: `Omit<BaseFieldProps, 'children' | ...>` +
  `Omit<PrimitiveProps, 'id' | 'aria-describedby' | 'aria-invalid' | 'aria-required'>`.
  Aria-звʼязок генерує `BaseField`, примітив його не приймає ззовні.
- `FieldRenderProps` окремо не мерджиться в примітиви — `id`/`aria-*` приходять
  нативно через `ComponentPropsWithoutRef`.

### Toggle / multi-select елементи (уточнення)
- Multi-select і елементи вибору (chip, картка-вибір) —
  `<button type="button" aria-pressed={isSelected}>`, ARIA APG toggle button.
- Switch — виняток: `<button type="button" role="switch" aria-checked={isChecked}>`,
  ARIA APG switch pattern. `aria-pressed` для switch не використовується.
- В обох випадках — семантична `<button>`, ніколи `<div>`/`<span>` + `onClick`.

### Іменування булевих пропсів
- Префікс `is` для стану компонента: `isChecked`, `isDisabled`, `isSelected`, `isError`.
- Нативний однойменний проп (`disabled`, `checked`) відрізається через `Omit`.

### Структура компонентів — виняток для *Field
- Кожен компонент живе у власній папці `ComponentName/`, ОКРІМ `*Field`-обгорток:
  вони лежать поруч зі своїм примітивом (`InputField` у `components/ui/Input/`,
  `SwitchField` у `components/ui/Switch/`) і ділять з ним `styles.module.css` та `types.ts`.
  Це єдиний випадок, коли в папці компонента лежить другий файл з іменем компонента.
```

---

## 13. Заборонено

- `<div>`/`<span>` + `onClick` замість `<button>`
- `aria-pressed` на свічі
- `left`/`right` для анімації knob
- CSS `border` для бордера треку
- `className` як проп
- Власний `onKeyDown` для Space/Enter
- Внутрішній `useState` (компонент контрольований)
- Хардкод значень — тільки токени з `globals.css`
- Фон сірого рядка, padding контейнера, тексти «Тільки обмін» всередині компонента

---

## 14. Поза межами спеки

- `size`-варіанти (`s`/`m`) — одного розміру достатньо, поки немає другого кейсу
- inline-layout обгортка (лейбл ліворуч від контрола) — окремий компонент, коли зʼявиться кейс
- `forwardRef` — коли зʼявиться потреба в програмному фокусі
- бізнес-кейс для `isDisabled` — проп і токен готові, умови вмикання визначає споживач
