# Brand QuickRef — generic CSS variables для КП

Шаблонизированная версия. Подставляй значения из `brief.md` (секция identity).

## CSS-переменные

Копируй в `:root` каждого КП. Заменяй `{{primary_dark}}`, `{{accent}}`, `{{background_light}}` на HEX из brief.

```css
:root {
  /* Основные (3 цвета из brief.md) */
  --primary-dark: {{primary_dark}};        /* напр. #0B1A2B (navy) */
  --primary-dark-light: {{primary_dark_lighter}};  /* +10% к лайтнесс */
  --accent: {{accent}};                    /* напр. #3ECFA0 (teal) */
  --accent-hover: {{accent_darker}};       /* −8% к лайтнесс */
  --background-light: {{background_light}}; /* напр. #F5F1EB (cream) */
  --white: #FFFFFF;

  /* Текст */
  --text-on-light: {{primary_dark}};
  --text-on-light-secondary: #4A5568;
  --text-on-light-muted: #64748B;
  --text-on-dark: #FFFFFF;
  --text-on-dark-secondary: #94A3B8;
  --text-on-dark-muted: #CBD5E1;

  /* Бордеры */
  --border-on-dark: {{primary_dark_lighter}};
  --border-on-light: #E2E8F0;

  /* Скругления */
  --radius: 16px;
  --radius-pill: 100px;

  /* Шрифты (2 из brief.md) */
  --font-display: '{{font_display}}', Georgia, serif;
  --font-body: '{{font_body}}', -apple-system, BlinkMacSystemFont, sans-serif;
}
```

### Как вычислить `primary_dark_lighter` и `accent_darker`

В голове: `#0B1A2B` → светлее на 10% → `#112240`. Для KP-сборки этого достаточно. Если хочешь точнее:

```python
# Простая утилита, не критична для КП
def lighten(hex_color, percent):
    h = hex_color.lstrip('#')
    r, g, b = int(h[0:2], 16), int(h[2:4], 16), int(h[4:6], 16)
    r = min(255, int(r + (255 - r) * percent / 100))
    g = min(255, int(g + (255 - g) * percent / 100))
    b = min(255, int(b + (255 - b) * percent / 100))
    return f'#{r:02X}{g:02X}{b:02X}'
```

## Подключение шрифтов (Google Fonts)

Большинство дефолтных пар работают через Google Fonts. Заменяй только имена.

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family={{font_body}}:wght@400;500;600;700&family={{font_display}}:ital@0;1&display=swap" rel="stylesheet">
```

Для не-Google шрифтов (Helvetica Now, Apple SF Pro и т.д.) — пользователь сам подключает либо через @font-face с локальным файлом, либо использует системные fallback.

## Типографика

| Уровень | Размер | Шрифт | Где |
|---------|--------|-------|-----|
| H1 hero | 56-64px | display | Hero-блок |
| H2 section-title | 36-42px | display **italic**, цвет адаптивный (text-on-light на light-секциях / text-on-dark на dark) | Заголовки секций |
| H3 card-title | 20-24px | body Bold (700) | Заголовки карточек |
| Body | 16px | body Regular | Описания, параграфы |
| Section-label | 12-13px | body Semibold uppercase | `// Команда`, `// Портфолио` |
| Metrics | 32-44px | display, color accent | Цифры в hero/metrics |

## Базовые компоненты

### Кнопка (Primary CTA)
```css
.btn-primary {
  display: inline-block;
  padding: 12px 28px;
  background: var(--accent);
  color: var(--primary-dark);
  border-radius: var(--radius-pill);
  font-weight: 600;
  text-decoration: none;
  transition: background 0.2s;
}
.btn-primary:hover { background: var(--accent-hover); }
```

### Бейдж pill (на тёмном)
```css
.badge {
  display: inline-block;
  padding: 4px 14px;
  border: 1px solid var(--accent);
  color: var(--accent);
  border-radius: var(--radius-pill);
  font-size: 12px;
  text-transform: uppercase;
  letter-spacing: 0.03em;
}
```

### Карточка на светлом
```css
.card {
  background: var(--white);
  border: 1px solid var(--border-on-light);
  border-radius: var(--radius);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.06);
  padding: 28px;
}
```

### Карточка на тёмном
```css
.card-dark {
  background: var(--primary-dark-light);
  border: 1px solid var(--border-on-dark);
  border-radius: var(--radius);
  padding: 32px;
}
```

## Co-branded header

Хедер показывает партнёрство: «{your company} × {client name}»:

```html
<div class="header-logo" style="display: flex; align-items: center; gap: 18px;">
  <div style="display: flex; align-items: center; gap: 10px;">
    <img src="logo.svg" alt="{{your_company}}" class="logo-svg">
    <span style="font-family: var(--font-display); font-size: 22px; color: var(--text-on-dark);">{{your_company}}</span>
  </div>
  <span style="color: var(--accent); font-size: 18px; opacity: 0.6;">×</span>
  <span style="font-family: var(--font-display); font-size: 20px; color: var(--text-on-dark);">{{client_name}}</span>
</div>
```

Зазор иконка ↔ имя компании — 10px. До `×` — 18px. `×` всегда accent-цвета с `opacity: 0.6`.

## Структура секций

Чередование dark ↔ light, hero и footer всегда dark:

```
┌─ Header (dark, co-branded) ──┐
├─ Hero (dark) ────────────────┤
├─ Метрики (dark) ─────────────┤
├─ Клиенты-логотипы (dark) ────┤  ← опционально
├─ Направления (light) ────────┤
├─ Преимущества (dark) ────────┤
├─ Команда (light) ────────────┤
├─ Портфолио (white) ──────────┤
├─ Footer (dark) ──────────────┘
```

## Wave divider (опционально между секциями)

```html
<svg class="wave-divider" viewBox="0 0 1440 100" fill="none" style="width: 100%; height: 80px; display: block;">
  <path d="M0,40 C360,100 720,0 1440,60 L1440,100 L0,100 Z" fill="var(--primary-dark)"/>
</svg>
```

## Логотип

В шапке (на dark):
```html
<img src="logo.svg" alt="{{your_company}}" class="logo-svg">
```

Если SVG монохромный чёрный, нужен фильтр:
```css
.logo-svg {
  height: 36px;
  width: auto;
  filter: brightness(0) invert(1);  /* чёрный SVG → белый */
}
```

Для цветного SVG / PNG — фильтр не нужен.
