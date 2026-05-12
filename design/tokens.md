# Design Tokens

Єдине джерело правди для всіх візуальних констант платформи.
Всі токени визначені в `globals.css` через Tailwind v4 `@theme`.
CSS Modules компонентів використовують ці змінні через `var(--...)`.

---

## Кольори

### Base (фони)
| Токен | Значення | Використання |
|-------|----------|--------------|
| `--color-base-canvas` | `#FDF8F6` | Основний фон сторінок |
| `--color-base-warm` | `#F0EDDC` | Вторинний фон, картки, секції |

### Accent (рожевий діапазон)
| Токен | Значення | Використання |
|-------|----------|--------------|
| `--color-accent-blush` | `#E8C4C4` | Hover стани, підсвічування |
| `--color-accent-rose` | `#D4AAAA` | Borders активних елементів, теги |
| `--color-accent-dusty` | `#C9A0A0` | Primary кнопки, акценти |

### Neutral (нейтральний)
| Токен | Значення | Використання |
|-------|----------|--------------|
| `--color-neutral-stone` | `#B9AC8C` | Borders, secondary text, dividers |
| `--color-neutral-sand` | `#D4C9A8` | Disabled стани, placeholder borders |

### Text
| Токен | Значення | Використання |
|-------|----------|--------------|
| `--color-text-primary` | `#845F4A` | Основний текст (Roman Coffee) |
| `--color-text-secondary` | `#B9AC8C` | Вторинний текст, мета-інфо |
| `--color-text-inverse` | `#FDF8F6` | Текст на темних поверхнях |

### Semantic
| Токен | Значення | Використання |
|-------|----------|--------------|
| `--color-success` | `#A8C5A0` | Успішні стани (м'який зелений) |
| `--color-error` | `#C9807A` | Помилки (м'який червоний) |
| `--color-warning` | `#D4B896` | Попередження |

---

## Типографія

### Typeface
Єдиний шрифт платформи: **Inter**.
Серифні шрифти протестовані і відхилені.

### Scale
| Токен | Значення | Використання |
|-------|----------|--------------|
| `--font-sans` | `'Inter', sans-serif` | Весь текст платформи |
| `--text-xs` | `0.75rem / 12px` | Мета, лейбли, badges |
| `--text-sm` | `0.875rem / 14px` | Secondary text, captions |
| `--text-base` | `1rem / 16px` | Body text |
| `--text-lg` | `1.125rem / 18px` | Підзаголовки, card titles |
| `--text-xl` | `1.25rem / 20px` | Section headings |
| `--text-2xl` | `1.5rem / 24px` | Page headings |
| `--text-3xl` | `1.875rem / 30px` | Hero headings |

### Weight
| Токен | Значення |
|-------|----------|
| `--font-normal` | `400` |
| `--font-medium` | `500` |
| `--font-semibold` | `600` |

---

## Border Radius

| Токен | Значення | Використання |
|-------|----------|--------------|
| `--radius-sm` | `8px` | Badges, теги, chips |
| `--radius-md` | `12px` | Інпути, невеликі картки |
| `--radius-lg` | `16px` | Картки товарів, модалі |
| `--radius-xl` | `24px` | Великі панелі, drawer |
| `--radius-full` | `9999px` | Кнопки-пілюлі, аватари |

---

## Spacing

Базова одиниця: `4px`. Tailwind v4 spacing scale використовується як є.
Кастомні токени лише для специфічних потреб платформи.

| Токен | Значення | Використання |
|-------|----------|--------------|
| `--spacing-card-pad` | `16px` | Внутрішній відступ картки |
| `--spacing-section` | `48px` | Відступ між секціями |
| `--spacing-page-x` | `24px` | Горизонтальний padding сторінки (мобайл) |

---

## Тіні

| Токен | Значення | Використання |
|-------|----------|--------------|
| `--shadow-card` | `0 2px 12px rgba(132, 95, 74, 0.08)` | Картки в спокої |
| `--shadow-card-hover` | `0 4px 24px rgba(132, 95, 74, 0.14)` | Картки на hover |
| `--shadow-drawer` | `0 0 40px rgba(132, 95, 74, 0.12)` | Side drawer, bottom sheet |
| `--shadow-sticky` | `0 -2px 16px rgba(132, 95, 74, 0.08)` | Sticky CTA бар знизу |

---

## Ефекти

**Frosted glass** — використовується sparingly (хедер при скролі, попапи).
```css
backdrop-filter: blur(12px);
background: rgba(253, 248, 246, 0.85);
```

**Watercolor texture** — тільки в hero і empty state зонах.
Ніколи на картках товарів.

---

## Що не використовуємо

- Чорний (`#000000`) і білий (`#ffffff`) як основні кольори — замінені на warm eggshell
- Темно-зелений текст — конкурує з фото рослин
- Яскравий помаранчевий, лаймовий зелений — порушують тональність
- Dark mode — не в MVP
