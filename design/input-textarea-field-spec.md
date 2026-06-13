# Input / Textarea / BaseField — спека

Дизайн-референс: `inputs.html` (потребує оновлення згідно з токенами нижче).
Контекст ghost-варіанту: `registration-step1/2/3.html`, `registration_flow.html`.

---

## 1. Огляд компонентів

| Компонент | Папка | Відповідальність |
|---|---|---|
| `BaseField` | `components/ui/Field/` | Вертикальний layout: label (+ required marker), control-slot, hint/error/counter. Не знає нічого про Input/Textarea. |
| `Input` | `components/ui/Input/` | `<input>`, варіанти `white`/`ghost`, слоти icon-left/icon-right/prefix. |
| `Textarea` | `components/ui/Textarea/` | `<textarea>`, варіанти `white`/`ghost`, resize handle, min-height 200px. |
| `InputField` | `components/ui/Input/` | `BaseField` + `Input`, генерує `id`/aria. |
| `TextareaField` | `components/ui/Textarea/` | `BaseField` + `Textarea`, генерує `id`/aria, прокидає counter. |
| `PasswordInputField` | `components/ui/Input/` | `InputField` + toggle видимості пароля (`Button variant="plain"`). |
| `TelInputField` | `components/ui/Input/` | `InputField` + маска `+380 XX XXX-XX-XX`, prefix. |

Усі контрольовані (`value` + `onChange`), без `defaultValue`. `ref` через `forwardRef` — для `autoFocus`/програмного фокусу.

---

## 2. Дизайн-токени — зміни відносно поточного `inputs.html`

### Додати (справді нові)

```css
/* hover-overlay — накладається ПОВЕРХ будь-якого стану, крім focus/disabled/readonly */
--input-white-hover-overlay: rgba(122,136,120,0.05);
--input-ghost-hover-overlay: rgba(255,255,255,0.05);

/* ghost focus змінює і фон, не тільки border */
--input-ghost-focus-bg: rgba(255,255,255,0.18);
```

### Не додавати — вже існують глобально

`--input-white-error-bg/border/color` і `--input-white-success-border/color` **вже посилаються** на `--color-error-secondary`/`--color-error-primary`/`--color-success-primary` з SEMANTIC-секції `globals.css` — ті самі токени, що й `--btn-primary-failure-*`, `--btn-danger-*`, `.col-h.err`. Нових error/success кольорів для Input не потрібно, вони вже спільні з Button.

### Змінити

- `--input-padding-y: 8px` (було 10px) — і `--textarea-padding-y: 8px`
- `--textarea-min-height: 200px` (було 80px)
- Усі `border: var(--input-border-size) solid <color>` → `box-shadow: inset 0 0 0 var(--border-size) <color>` (властивість `border` прибирається з `input.f`/`textarea.f`)
- `border-radius: var(--radius-full)` для **Input** (обидва варіанти white/ghost). **Textarea лишається `var(--radius-lg)` (16px)** — на `min-height: 200px` `radius-full` дав би "капсулу", що було помилковим рішенням, відкочено.
### Глобальне (globals.css, SEMANTIC секція — поза Input/Textarea)

```css
--line-height-base: 1;
--color-placeholder: rgba(28, 28, 28, 0.32); /* нейтральний сірий, похідний від --color-text-primary (#1C1C1C) */
```

`body`/base reset отримує `line-height: var(--line-height-base)`. Текстові блоки з більшим line-height (`.hero-sub: 1.65`, `.footer-desc: 1.65` тощо) лишаються explicit override.

`--color-placeholder` — у проєкті раніше не було жодного нейтрального "сірого" для плейсхолдерів (тільки зелено-сіра `--color-secondary`/`--color-text-secondary`). Новий токен — для `--input-white-placeholder` (раніше дублював `--color-text-secondary`, тепер окремий, неоднаковий з кольором label/hint). Ghost-плейсхолдер (`rgba(255,255,255,0.38)`) лишається без змін — він і так нейтральний.

### Input/Textarea токени

```css
--input-font-size: var(--font-size-m);    /* 14px, було 12px */
--input-hint-font-size: var(--font-size-s);    /* 12px, було 10px */
--input-counter-font-size: var(--font-size-s); /* 12px, було 10px — той самий рядок, що hint */
--input-label-font-size: var(--font-size-s);   /* 12px, було 10px; можливо 14px після перевірки на реальних сторінках */
--input-icon-size: 14px;                  /* було 15px */
--input-prefix-padding: 50px;             /* було 46px — "+380"/"@" при 14px ширші, перевірити візуально */
--input-white-placeholder: var(--color-placeholder); /* було var(--color-text-secondary) — тепер відрізняється від label/hint */
--input-line-height: var(--line-height-base);
--textarea-line-height: var(--input-line-height);
--textarea-hint-font-size: var(--input-hint-font-size);
--textarea-counter-font-size: var(--input-counter-font-size);
--textarea-label-font-size: var(--input-label-font-size);
```

`input.f`, `textarea.f` та `.il`/`.ih`/`.ic`/`.pfx-label` використовують `line-height: var(--input-line-height)` (textarea — `var(--textarea-line-height)`), а не літерал.

### appearance:none

```css
input.f, textarea.f {
  appearance: none;
  -webkit-appearance: none;
  -moz-appearance: none;
}
```

Прибирає UA-стилі form-controls (зайвий intrinsic padding/border) і нативну "x" в `type="search"` у Webkit.

**Висота поля**: явний `height`, не покладаємось на auto-обчислення з line-height (для `<input>` це ненадійно — давало 33px замість очікуваних 30px):

```css
.input {
  height: calc(var(--input-padding-y) * 2 + var(--input-font-size) * var(--input-line-height));
}
```

= `8*2 + 14*1 = 30px` (border-box). `line-height:1` лишається — керує метриками тексту, `height` фіксує box. Textarea — без явної `height`, лишається `min-height: 200px` (multiline/resize).

### Кольори border/disabled (white) — прямий rgba, без нового токена

```css
--input-white-border: rgba(122, 136, 120, 0.30);
--input-white-filled-border: rgba(122, 136, 120, 0.50);
--input-white-disabled-border: rgba(122, 136, 120, 0.18);
--input-white-disabled-color: rgba(122, 136, 120, 0.45);
--input-white-readonly-border: rgba(122, 136, 120, 0.20);
--input-white-hover-overlay: rgba(122, 136, 120, 0.05);
```

`rgba(122,136,120,X)` = rgb-канали `--color-secondary` (#7A8878) — той самий magic-number паттерн, що вже скрізь у `globals.css` (Button ghost-disabled, Tooltip, StepPopup, ListingStatusBanner). Нового токена не вводимо — консистентність з існуючим кодом важливіша за DRY для значення, яке роками не змінювалось.

Label/placeholder/hint (`--input-white-label-color`, `--input-white-placeholder`, `--input-white-hint-color`) — без змін, вже `var(--color-text-secondary)` = `var(--color-secondary)` = `#7A8878`, вже глобальний.

### Видалити

- Усі правила `.loading` (`input.f.w.loading`, `input.f.gh.loading`) — стан прибрано, сценарію немає
- SVG-іконки error/circle-i та success/checkmark з усієх таблиць станів (`.ico` всередині `.ib` для статусів) — статусні іконки відкладені, сигнал тільки border + текст

### Залишити без змін (просто не використовуються активно)

- `--input-*-success-border` / `--input-*-success-color` — токени лишаються, CSS-правило для `.success` пишемо (дешево, симетрично з `.error`), але в dev-сторінці success як стан показуємо без іконки, як заготовку на майбутнє

---

## 3. Hover — "інтерактивний елемент ніколи не мертвий"

Hover не замінює стан, а **накладається на нього** додатковим шаром `box-shadow` (inset з величезним spread = заливка всієї площі під border):

```css
/* WHITE — default */
.input.white {
  box-shadow: inset 0 0 0 var(--border-size) var(--input-white-border);
}
.input.white:hover:not(:disabled):not(:read-only):not(:focus) {
  box-shadow:
    inset 0 0 0 var(--border-size) var(--input-white-border),
    inset 0 0 0 9999px var(--input-white-hover-overlay);
}

/* WHITE — filled */
.input.white.filled {
  box-shadow: inset 0 0 0 var(--border-size) var(--input-white-filled-border);
}
.input.white.filled:hover:not(:focus) {
  box-shadow:
    inset 0 0 0 var(--border-size) var(--input-white-filled-border),
    inset 0 0 0 9999px var(--input-white-hover-overlay);
}

/* WHITE — error */
.input.white.error {
  box-shadow: inset 0 0 0 var(--border-size) var(--input-white-error-border);
  background: var(--input-white-error-bg);
}
.input.white.error:hover:not(:focus) {
  box-shadow:
    inset 0 0 0 var(--border-size) var(--input-white-error-border),
    inset 0 0 0 9999px var(--input-white-hover-overlay);
}
```

Порядок шарів важливий: тонкий border-shadow (0.5px) — перший, overlay (9999px) — другий, інакше overlay перекриє border.

Те саме для `.ghost` з відповідними `--input-ghost-*` токенами, плюс `.success` симетрично до `.error`.

**Виключення з hover** (немає інтеракції → немає hover): `:disabled`, `:read-only`.
**Focus придушує hover-overlay** (`:not(:focus)`) — focus-ring вже є максимальним сигналом, overlay зайвий.

---

## 4. BaseField (`components/ui/Field/BaseField.tsx`)

Вертикальний layout, без знання про Input/Textarea. Generic для майбутніх checkbox/switch/range — **але якщо їм знадобиться інший layout (inline label), вони отримають власний wrapper, не модифікацію цього**.

```ts
export interface FieldRenderProps {
  id: string;
  'aria-describedby'?: string;
  'aria-invalid'?: true;
  'aria-required'?: true;
}

export interface FieldCounter {
  current: number;
  max: number;
}

export interface BaseFieldProps {
  label?: string;
  required?: boolean;
  hint?: string;
  error?: string;
  counter?: FieldCounter;
  children: (fieldProps: FieldRenderProps) => ReactNode;
}
```

Логіка:
- `id = useId()`
- якщо `error` — `aria-invalid: true`, `aria-describedby` вказує на error-текст; якщо немає `error`, але є `hint` — `aria-describedby` вказує на hint
- якщо `required` — `aria-required: true`, label рендериться як `{label}*` (зірочка — `<span aria-hidden>`, сам текст не повторює "обов'язкове" для screen reader, бо це покриває `aria-required`)
- hint/error — один слот: якщо `error` присутній, рендериться він замість `hint` (колір з `--input-*-error-color`, тип тексту інший)
- `counter` — рендериться праворуч у тому ж рядку, що hint/error (`X / max`), кольором `--input-*-hint-color` (або error-color, якщо `counter.current > counter.max`)

---

## 5. Input (`components/ui/Input/Input.tsx`)

```ts
export type InputVariant = 'white' | 'ghost';

export interface InputProps extends ComponentPropsWithoutRef<'input'> {
  variant?: InputVariant; // default 'white'
  isError?: boolean;
  isSuccess?: boolean;
  iconLeft?: ReactNode;
  iconRight?: ReactNode;
  prefix?: string;
}
```

`type` — нативний `HTMLInputTypeAttribute` з `ComponentPropsWithoutRef<'input'>`, без кастомного union. Input не знає семантики "email"/"password"/"search" — просто прокидає `type` в `<input>`. Усі інші нативні атрибути (`placeholder`, `name`, `autoComplete`, `maxLength`, `inputMode` тощо) теж приходять через `ComponentPropsWithoutRef<'input'>` і прокидаються через `...rest` без додаткового опису.

`forwardRef<HTMLInputElement, InputProps>`.

Стани через CSS Module класи (через `mergeClassNames`, не data-атрибути — узгоджено з принципом `failure` як prop, не data-атрибут):
- `styles.input` + `styles[variant]` завжди
- `styles.filled` — коли `value.length > 0`
- `styles.error` — коли `isError`
- `styles.success` — коли `isSuccess`
- `:disabled`, `:read-only`, `:focus`, `:hover` — нативні pseudo-classes, без класів

Слоти:
- `iconLeft` / `iconRight` — `ReactNode`, рендеряться в обгортці з `pointer-events: none` якщо це не інтерактивний елемент (для `Button variant="plain"` — pointer-events не блокуються, це вирішує сам слот-контент)
- `prefix` — `string`, рендериться як нередаговний `<span>` перед input (`pfx-label`), input отримує `padding-left: var(--input-prefix-padding)` коли `prefix` присутній
- `icl`/`icr` padding застосовується незалежно одне від одного, коли є `iconLeft`/`iconRight`

---

## 6. Textarea (`components/ui/Textarea/Textarea.tsx`)

```ts
export interface TextareaProps extends ComponentPropsWithoutRef<'textarea'> {
  variant?: InputVariant; // default 'white', імпорт типу з Input.types
  isError?: boolean;
  isSuccess?: boolean;
}
```

`forwardRef<HTMLTextareaElement, TextareaProps>`. Без icon/prefix слотів. `resize: vertical`, `min-height: 200px`. Resize-handle іконка приглушується (`opacity`) в `disabled`/`readonly`.

---

## 7. InputField / TextareaField

```ts
// InputField.tsx
export interface InputFieldProps
  extends Omit<BaseFieldProps, 'children'>,
    Omit<InputProps, 'id' | 'aria-describedby' | 'aria-invalid' | 'aria-required'> {}
```

```tsx
export const InputField = forwardRef<HTMLInputElement, InputFieldProps>(
  ({ label, required, hint, error, ...inputProps }, ref) => (
    <BaseField label={label} required={required} hint={hint} error={error}>
      {(fieldProps) => (
        <Input ref={ref} {...fieldProps} isError={Boolean(error)} {...inputProps} />
      )}
    </BaseField>
  )
);
```

`TextareaField` — аналогічно, додатково:
```ts
export interface TextareaFieldProps
  extends Omit<BaseFieldProps, 'children' | 'counter'>,
    Omit<TextareaProps, 'id' | 'aria-describedby' | 'aria-invalid' | 'aria-required'> {
  showCounter?: boolean; // якщо true і є maxLength — рахує counter з value.length
}
```
`counter` для `BaseField` обчислюється всередині `TextareaField`: `{ current: value.length, max: maxLength }`, якщо `showCounter && maxLength`.

**`variant` ніде не хардкодиться.** `InputField`, `TextareaField`, `PasswordInputField`, `TelInputField` — усі приймають `variant?: InputVariant` (default `'white'`) як звичайний проп з `InputProps`/`TextareaProps` і прокидають його в `Input`/`Textarea` без власного значення за замовчуванням, відмінного від базового. Те, що `PasswordInputField` зараз демонструється в ghost-контексті (auth popup) — це контекст використання, не дефолт компонента.

---

## 8. PasswordInputField (`components/ui/Input/PasswordInputField.tsx`)

```ts
export interface PasswordInputFieldProps
  extends Omit<InputFieldProps, 'type' | 'iconRight'> {}
```

Використовує `usePasswordVisibility` (`components/ui/Input/usePasswordVisibility.ts`):

```ts
export function usePasswordVisibility() {
  const [isVisible, setIsVisible] = useState(false);
  const toggleVisibility = (): void => setIsVisible((prev) => !prev);
  return { isVisible, toggleVisibility };
}
```

`iconRight` = `<Button variant="plain" size="icon" aria-label={isVisible ? 'Сховати пароль' : 'Показати пароль'} aria-pressed={isVisible} onClick={toggleVisibility}>{/* eye / eye-off svg */}</Button>`, `type={isVisible ? 'text' : 'password'}`.

**Re-render на toggle — навмисний, не проблема.** Зміна `isVisible` змінює DOM-атрибут `type` інпута — це і є мета toggle, тож ре-рендер `Input` (один елемент, без дітей) неминучий і дешевий. Це не той кейс, що з FilterPopup (ре-рендер на кожен keystroke у широкому дереві) — тут одна подія за весь lifecycle поля. Імперативна альтернатива (`inputRef.current.type = ...` через ref, в обхід React state) порушила б "за патернами react.dev" з CLAUDE.md заради невимірюваної вигоди — не робимо цього.

---

## 9. TelInputField (`components/ui/Input/TelInputField.tsx`)

```ts
export interface TelInputFieldProps
  extends Omit<InputFieldProps, 'type' | 'prefix' | 'value' | 'onChange'> {
  value: string; // зберігається БЕЗ +380, тільки цифри: "671234567"
  onChange: (digits: string) => void;
}
```

Формат відображення: `+380 XX XXX-XX-XX`. `prefix="+380"` фіксований.

`usePhoneMask` (`components/ui/Input/usePhoneMask.ts`) — без бібліотек, чистий форматер:

```ts
export function formatPhoneDigits(digits: string): string {
  const area = digits.slice(0, 2);
  const part1 = digits.slice(2, 5);
  const part2 = digits.slice(5, 7);
  const part3 = digits.slice(7, 9);

  let result = area ? ` ${area}` : '';
  if (part1) result += ` ${part1}`;
  if (part2) result += `-${part2}`;
  if (part3) result += `-${part3}`;

  return result;
}

export function extractPhoneDigits(formatted: string): string {
  return formatted.replace(/\D/g, '').slice(0, 9);
}

export function usePhoneMask(value: string, onChange: (digits: string) => void) {
  const displayValue = formatPhoneDigits(value);
  const handleChange = (e: ChangeEvent<HTMLInputElement>): void => {
    onChange(extractPhoneDigits(e.target.value));
  };
  return { displayValue, handleChange };
}
```

Курсор: після форматування курсор виставляється в кінець введеного значення (просте рішення для MVP; складніша логіка збереження позиції — якщо реально знадобиться після тестування).

---

## 10. Файлова структура

```
components/ui/Field/
  BaseField.tsx
  BaseField.module.css
  BaseField.types.ts

components/ui/Input/
  Input.tsx
  Input.module.css
  Input.types.ts
  InputField.tsx
  PasswordInputField.tsx
  usePasswordVisibility.ts
  TelInputField.tsx
  usePhoneMask.ts

components/ui/Textarea/
  Textarea.tsx
  Textarea.module.css
  Textarea.types.ts
  TextareaField.tsx

app/dev/input/
  page.tsx
  InputDemo.module.css

app/dev/textarea/
  page.tsx
  TextareaDemo.module.css

app/dev/page.tsx  — оновити, додати лінки на /dev/input, /dev/textarea
```

---

## 11. Dev-сторінки

**`/dev/input`** — таблиці для кожного варіанту (`white` на page-bg, `ghost` на forest-gradient bg, як у `inputs.html`):
- Стани: default, hover, focus, filled, filled+hover, error, error+hover, success, success+hover, disabled, readonly
- Icon left (search)
- Icon right (password toggle — `PasswordInputField`)
- Icon left + right (search + clear)
- Prefix (`TelInputField`, `+380`)
- З label / без label (з `aria-label`)
- Required (з `*`) / без

**`/dev/textarea`** — ті ж стани (без icon/prefix), + варіант з `showCounter` і `maxLength`, у тому числі counter у error-кольорі при перевищенні.

Кожен стан — окрема колонка з підписом, hover-стани показати окремою колонкою поруч з base-станом (default | default+hover | filled | filled+hover | ...), щоб було видно різницю overlay.
