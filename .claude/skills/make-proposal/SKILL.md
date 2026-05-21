---
name: make-proposal
description: Use when user wants to create a commercial proposal (КП / commercial proposal / sales pitch) for a client. Skill reads a transcript of a discovery call from transcripts/<client>.txt, extracts SPSV (Ситуация / Проблема / Решение / Ценность), runs compact market research on the client, brainstorms solution options with the user via superpowers:brainstorming, then renders a one-page HTML proposal in the user's brand identity. Free of LLM-style meta-commentary. Triggers on "КП", "коммерческое предложение", "commercial proposal", "сделай предложение", "собери КП", "make proposal".
---

# Make Proposal Skill

Skill для сборки одностраничного HTML-КП **из транскрипта звонка с клиентом** — с извлечением SPSV, компактным market research, и brainstormингом решения.

## Три входа

Skill определяет режим работы по содержимому проекта:

| Что есть в проекте | Режим |
|---|---|
| `transcripts/<client>.txt` | **Transcript-driven** (главный, рекомендуемый) |
| `brief.md` заполнен (≠ brief.example.md) | **Brief-driven** (fallback, если транскрипта нет) |
| Ничего | **Intake** (skill сам задаёт 8 вопросов) |

Дальше описан главный — **Transcript-driven** workflow. Brief-driven и Intake описаны в конце.

## Workflow (Transcript-driven)

### Phase 0 — Поиск транскрипта

1. Скани `transcripts/*.txt`. Если файлов несколько — спроси, какой использовать. Если один — используй его.
2. Прочитай транскрипт целиком.
3. Прочитай `brief.md` — там должна быть информация о **твоей компании** (название, identity, команда, кейсы, контакты). Если brief.md нет или он совпадает с brief.example.md — попроси заполнить (это «постоянная часть», нужна один раз).

### Phase 1 — SPSV Extraction

Прочитай `references/spsv-extraction.md` и извлеки из транскрипта:
- **Ситуация** (контекст клиента — размер, инструменты, текущая практика)
- **Проблема** (боль с его собственными словами)
- **Решение** (как клиент сам видит решение)
- **Ценность** (KPI, сроки, бюджет)
- **3-5 ключевых цитат** — дословно
- **Подводные камни** (озвученные риски, прошлые провалы)
- **Команда клиента** (стейкхолдер, ключевой пользователь)
- **Формат сдачи** (HTML/PDF/презентация, сроки)

Сохрани в `active/diagnosis-<client-slug>.md` по шаблону из spsv-extraction.md.

**Показ пользователю:**
```
Вот что я понял из звонка:

Ситуация: ...
Проблема: ...
Решение (как видит клиент): ...
Ценность: ... (KPI, сроки, бюджет)

Ключевые цитаты:
1. «...»
2. «...»

Риски, на которые надо ответить в КП:
- ...

Так понял или скорректировать?
```

Жди подтверждения. Если правки — обнови diagnosis. Если ок — фаза 2.

### Phase 2 — Compact Market Research

Прочитай `references/market-research.md`. Сделай **компактный** research (3-5 WebSearch + 3-5 WebFetch):
- Свежие новости клиента (≤30 дней) — 1-2 события
- 1-2 конкурента и их недавние ходы
- 1 отраслевой тренд с источником

Сохрани в `active/research-<client-slug>.md`.

**Показ пользователю:**
```
Что нашёл на рынке:

Свежие события клиента:
- ... (дата, источник)

Конкуренты:
- ... (что сделали)

Отраслевой тренд:
- ... (источник, дата)

Как использую в КП:
- В hero: ...
- В advantages: ...
- В portfolio: ...

Использовать эти факты или скорректировать?
```

Жди подтверждения.

### Phase 3 — Brainstorming решения

**Это ключевая фаза.** На этом этапе мы переходим от «что у клиента болит» к «что мы предлагаем».

**Вызови superpowers:brainstorming skill.** Передай ему:
- Diagnosis (SPSV-материал)
- Research (market-факты)
- Brief.md (информация о твоей компании — что мы умеем)

Brainstorming проведёт пользователя через:
- Уточнение scope решения
- Обсуждение 2-3 подходов с trade-offs
- Финальный design — что предлагаем, чем измеряем, какие риски снимаем

**Важно:** brainstorming — интерактивный. Он задаёт вопросы по одному. Не пытайся обойти его — пройди по его шагам с пользователем.

Результат: design-spec, который brainstorming сохраняет в `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md` (или эквивалент по spec'у skill'а).

**Из design-spec извлеки** для КП:
- **Hero h1** (новая формулировка решения, не из транскрипта)
- **Approach** (1-2 параграфа: как мы решаем проблему)
- **Phases / Milestones** (этапы проекта)
- **Risks addressed** (как закрываем риски, которые озвучил клиент)
- **Success metrics** (как поймём что сработало — связано с Ценностью клиента)

### Phase 4 — Sketch КП (текстовый превью)

**Перед** генерацией HTML — покажи пользователю **текстовый сценарий КП**:

```
Скетч КП для <Клиент>:

HERO H1: «<формулировка>»
LEAD: <2-3 строки>

МЕТРИКИ: <4 цифры из brief.md о нашей компании>

НАПРАВЛЕНИЯ (3-4 карточки из brief.md, отфильтрованных под этот кейс):
- ...

ПРЕИМУЩЕСТВА (3-4 — выбираем те, что закрывают риски клиента):
- ...

КОМАНДА (2-4 человека из brief.md):
- ...

ПОРТФОЛИО (3-4 кейса из brief.md, релевантных индустрии/задаче клиента):
- ...

ПОДХОД (новая секция, из brainstorming-spec):
- ...

CTA: <предлагаемая следующая встреча / срок ответа>

Так делаем или корректируем?
```

Жди подтверждения. Если правки — переработай скетч.

### Phase 5 — Generate HTML draft

Когда скетч одобрен:

1. Прочитай `references/brand-quickref-template.md`.
2. Прочитай `references/llm-artifacts.md` — список запрещённых фраз.
3. Возьми `templates/proposal.html.template`.
4. Подставь:
   - Identity из `brief.md` (цвета, шрифты, ваша компания)
   - Hero/lead/approach из brainstorming-spec
   - Команда / портфолио из `brief.md`
   - `client_name` из diagnosis
5. Сохрани в `active/proposals/<client-slug>-draft.html`.
6. **Перед открытием — grep на LLM-артефакты:**
   ```bash
   grep -nE "лежит в основе|нашего предложения|шаблон формата|близкий референс|это бьёт|ровно то|на встрече|вы сказал|Релевантно вам|точечный референс|в рамках предложения" active/proposals/<slug>-draft.html
   ```
   Если что-то нашлось — переписать и пере-grep'нуть.
7. Пройди `references/checklist.md`.
8. Открой: `open active/proposals/<slug>-draft.html`.

### Phase 6 — Standalone (опционально)

Спроси: «Собирать standalone-версию с base64-картинками для отправки одним файлом?». Если да — собери (см. секцию «Сборка standalone HTML»).

## Сборка standalone HTML

Цель — один самодостаточный `.html`, который можно переслать по почте/Telegram.

Что встраиваем:
- **Логотип, фото команды** → `data:image/...;base64,...` data URI в `src`. SVG предпочтительнее.
- **Google Fonts через `@import`** — оставляем, шрифты подгружаются у получателя.

```bash
# 1. Скопировать draft → standalone
cp active/proposals/<slug>-draft.html active/proposals/<slug>-standalone.html

# 2. Для каждого внешнего изображения
base64 -i assets/logo.svg | tr -d '\n' > /tmp/logo_b64.txt

# 3. Заменить пути на data URI
python3 -c "
b64 = open('/tmp/logo_b64.txt').read()
path = 'active/proposals/<slug>-standalone.html'
html = open(path).read()
html = html.replace('assets/logo.svg', f'data:image/svg+xml;base64,{b64}')
open(path, 'w').write(html)
"

# 4. Проверить, что не осталось внешних путей
grep -E 'src=' active/proposals/<slug>-standalone.html | grep -v 'data:image'
# должно быть пусто

# 5. Открыть
open active/proposals/<slug>-standalone.html
```

## Логотип на тёмном фоне

Монохромный чёрный SVG на dark-хедере → CSS-фильтр:
```css
.logo-svg { filter: brightness(0) invert(1); }
```

Для цветного — без фильтра, но проверь контраст на тёмном.

## Brief-driven workflow (fallback)

Если `transcripts/` пуст, но `brief.md` заполнен:
1. Прочитай brief.md
2. Пропусти Phase 1-2 (SPSV / research) — у нас нет звонка для извлечения
3. Сразу переходи к **Phase 3 (Brainstorming)**, но дай brainstorming весь brief как контекст
4. Дальше как обычно: Phase 4 (sketch) → Phase 5 (HTML) → Phase 6 (standalone)

Этот режим — для случая, когда КП делается без discovery-звонка (например, по входящей заявке с понятным запросом).

## Intake workflow (если ничего нет)

Если ни `transcripts/`, ни заполненного `brief.md` нет:
1. Прочитай `references/intake-questions.md`
2. Задай 8 вопросов **по одному**, сохраняя ответы в `brief.md`
3. После того как brief.md заполнен — переключайся в Brief-driven workflow

Этот режим — для самого первого использования skill'а, когда пользователь ещё не настроил айдентику.

## Без LLM-артефактов

**Критическое правило.** Запрещённые фразы в `references/llm-artifacts.md`. Перед финализацией — grep-проверка обязательна.

## Без эмодзи

В деловом КП эмодзи не используются. Маркеры/категории — через точки, бордеры, тэги.

## Output location

- Транскрипты → `transcripts/<client>.txt` (вход, кладёт пользователь)
- Diagnosis → `active/diagnosis-<client-slug>.md`
- Research → `active/research-<client-slug>.md`
- Brainstorming-spec → `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md` (управляется superpowers:brainstorming)
- Brief.md → корень проекта (информация о твоей компании, заполняется один раз)
- Draft HTML → `active/proposals/<client-slug>-draft.html`
- Standalone HTML → `active/proposals/<client-slug>-standalone.html`
- Build-скрипты → `active/scripts/`
- Base64 промежутки → `active/tmp/`

## Confirmation gates (явные)

Skill делает паузы и ждёт подтверждение в этих точках:

1. После Phase 1 (SPSV) — «так понял ситуацию?»
2. После Phase 2 (Research) — «факты использовать?»
3. После Phase 3 (Brainstorming) — управляется самим brainstorming-skill'ом
4. После Phase 4 (Sketch) — «так делаем КП?»
5. После Phase 6 — «собирать standalone?»

Не пропускай эти gate'ы — это контракт со пользователем, что мы не уходим в дебри без его кивка.

## Язык

По умолчанию — язык транскрипта. Если транскрипт на русском — КП на русском. Если на английском — на английском. Стиль и структура одинаковы.
