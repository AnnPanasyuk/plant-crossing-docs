# Chip — спека

## Призначення

Toggle-чип для мультивибору в `FilterPopup`, кроки 1–4 (стан рослини, складність догляду, діаметр горщика, тип угоди). Єдине місце використання — `FilterStep` body для chip-кроків.

**Поза межами цієї спеки:** чип з іконкою `×` (видалення з мультивибору в інпуті) — окремий майбутній компонент, без зв'язку з `Chip`. Не додавати плейсхолдери чи опціональні пропи під нього.

---

## Семантика

```tsx
<button type="button" aria-pressed={isSelected} disabled={isDisabled}>
  {label}
</button>
```

ARIA APG toggle button pattern. Один елемент, один `onClick` = `onToggle`, без розділення на select/deselect.

---

## Props (`Chip.types.ts`)

```ts
export type ChipProps = {
  label: string;
  isSelected: boolean;
  onToggle: () => void;
  isDisabled?: boolean;
};
```

`isDisabled` — проп присутній, токен (`--chip-disabled-opacity`) готовий. Коли і за яких умов `FilterStep` передає `isDisabled={true}` — **не визначено в цій спеці**, бізнес-логіка (наприклад, "0 результатів для цього варіанту") додається пізніше окремо.

---

## Стани та токени (`globals.css`)

| Стан | Клас / селектор | Токени |
|---|---|---|
| default | `.chip` | `--chip-bg`, `--chip-color`, box-shadow inset `--chip-border-color` |
| hover | `.chip:hover` | `--chip-hover-bg`, `--chip-hover-color`, box-shadow inset `--chip-hover-border-color` |
| focus-visible | `.chip:focus-visible` | додатково `0 0 0 var(--chip-focus-ring-size) var(--chip-focus-ring)` до box-shadow |
| selected | `.chip.selected` | `--chip-selected-bg`, `--chip-selected-color`, box-shadow inset `--chip-selected-border-color` |
| selected + hover | `.chip.selected:hover` | `--chip-selected-hover-bg`, border-color як у selected |
| selected + focus-visible | `.chip.selected:focus-visible` | selected + focus ring |
| disabled | `.chip:disabled` | `opacity: var(--chip-disabled-opacity)`, `cursor: not-allowed` |

### Зміни в `globals.css` (секція `/* === CHIP === */`)

**Border → box-shadow.** Замінити `border: var(--border-size) solid var(--chip-border-color)` на `box-shadow: inset 0 0 0 var(--border-size) var(--chip-border-color)` — за правилом дизайн-системи (CLAUDE.md, "Visible borders"). Стосується всіх `--chip-*-border-color` варіантів.

**Нові токени:**
```css
--chip-focus-ring: rgba(212, 145, 158, 0.18);
--chip-focus-ring-size: 2px;
```
(значення узгоджені з `--input-white-focus-ring` / `--input-white-focus-ring-size`)

**Видалити:** `--chip-icon-size`, `--chip-icon-gap` — належали × -варіанту, який поза межами цієї спеки. Якщо майбутній чип-2 буде розроблятись на основі `--chip-*` namespace, токени додаються заново разом з його дизайном.

**Padding:** `--chip-padding: 2px 7px` → `--chip-padding: 4px 8px`. Часткове покращення touch target, до рекомендованих 44px не дотягує — прийнятно для popup-контексту.

**font-size:** `--chip-font-size: var(--font-size-s)` (12px) → `--chip-font-size: var(--font-size-m)` (14px).

**Висота чипа (підсумок):** 12px / pad 2px / lh 1 = 16px (було) → 14px / pad 4px 8px / lh 1 = **22px** (нове). Разом padding + font-size дають +6px до висоти.

**line-height:** `1` (конвенція проєкту), не `1.4` — стосується `.chip` явно, токена немає.

---

## Файлова структура

```
components/
  ui/
    Chip/
      Chip.tsx           — компонент, <button type="button" aria-pressed>
      Chip.module.css    — стилі: .chip, .selected, :hover/:focus-visible/:disabled
      Chip.types.ts       — ChipProps

app/
  dev/
    chip/
      page.tsx              — dev-сторінка
      ChipDemo.module.css   — стилі демо-таблиці
```

---

## Dev-сторінка (`app/dev/chip/page.tsx`)

- Інтерактивний блок: 4–5 чипів з власним `useState<boolean>` для `isSelected`, реальні hover/focus через CSS — підпис "interactive →"
- Окремий приклад з `isDisabled={true}` — статичний, лейбл "Немає"
- Контрастний фон не потрібен (чип розрахований на page-bg / `--color-base-linear-gradient`, той самий що в `app/dev/page.tsx`)
- Додати посилання на `/dev/chip` в `app/dev/page.tsx`

---

## Інтеграція з FilterStep (для довідки, не частина цієї спеки)

```tsx
{options.map((option) => (
  <Chip
    key={option.value}
    label={option.label}
    isSelected={selectedValues.includes(option.value)}
    onToggle={() => toggleValue(option.value)}
  />
))}
```

Контейнер — `display: flex; flex-wrap: wrap; gap: ...` (токен з globals.css для gap між чипами в `FilterStep.module.css`, не дублювати тут).
