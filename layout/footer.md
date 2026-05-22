# Footer Layout

Легкий інформаційний футер. Присутній на всіх сторінках.

---

## Стилі

```css
padding: 0 40px; /* desktop — однаковий з header і StickyCTA */
background: transparent; /* успадковує page background */
```

Внутрішня обгортка `.footer-inner`:
```css
border-top: var(--border-size) solid rgba(186,196,184,0.5);
padding: 36px 0 24px;
```

Footer bottom:
```css
border-top: var(--border-size) solid rgba(186,196,184,0.4);
padding-top: 14px;
font-size: 10px;
color: var(--color-text-secondary);
```

---

## Структура

Desktop: `grid-template-columns: 1.5fr 1fr 1fr`, `gap: 40px`
Mobile: одна колонка, секції в стеку

### Колонка 1 — Бренд
- Logo mark + "PlantCrossing"
- Короткий опис платформи (1–2 речення)

### Колонка 2 — Навігація
- About · Contact · Terms of Use · Privacy Policy · Cookies

### Колонка 3 — Спільнота
- Як працює обмін
- Додати оголошення
- Онлайн ярмарка — скоро *(opacity: 0.45)*

### Footer bottom
`© 2026 PlantCrossing · Зроблено з любов'ю до рослин 🌿`

---

## StickyCTA і footer

На сторінці деталей `StickyCTA` — `position: fixed`. Щоб вона не перекривала контент, footer отримує:
```css
padding-bottom: calc(60px + 16px);
```

---

## Що не робимо

- Темний або темно-зелений фон
- Великі блоки кольору
- Дублювання головної навігації хедера
- Соціальні мережі в MVP
