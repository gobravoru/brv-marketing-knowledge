# Ресёрч: Higgsfield Claude skill — как они строят промпты для генерации картинок (обложек/постеров)

**Дата:** 2026-07-02
**Вопрос:** как официальный skill Higgsfield для Claude Code и связанная экосистема устроены в части image-промптинга (обложки, постеры, статичные креативы с текстом), и что переносимо в наш конвейер обложек Telegram-постов на gpt-image-2.

---

## TL;DR

1. **Официальный скилл Higgsfield (`higgsfield-ai/skills`, 500+ звёзд) НЕ пишет image-промпты сам** — для брендовых картинок агенту прямо запрещено «freehand» промптинг: `--prompt` содержит только короткое описание намерения, а финальный промпт собирает бэкенд-«enhancer» с закрытыми шаблонами. Публично известна лишь схема: enhancer держит «mode-specific photography vocabulary and structural templates» и отправляет в `gpt_image_2`.
2. **Анти-однотипность у Higgsfield — это осознанная политика бэкенда:** при `--count N` «Backend asks the enhancer to **vary preset, lighting, angle, and palette across variants** — they will not be paraphrased copies of one another». А для каруселей наоборот: «Backend **locks the visual system** across all slides». Оси вариативности названы явно: пресет / свет / ракурс / палитра.
3. **Базовая структура промпта (официальный `prompt-engineering.md`):** Subject + setting + style → Camera (линза, ракурс) → Lighting → Style/medium; длина **до ~200 токенов** («Models distort with very long prompts»); негативы формулировать позитивно («no blur» → «tack sharp»).
4. **Самый богатый публичный корпус — комьюнити-скилл `OSideMedia/higgsfield-ai-prompt-skill`** (переведён с материалов автора, близкого к команде Higgsfield). Для gpt-image-2 там трёхформатная таксономия: **Format A — structured JSON** для макетов/постеров с зонами, **Format B — плотная кинематографичная проза** для одной сцены, **Format C — мета-промпт** «сам выведи всю композицию из темы». Для обложек с текстом основной — Format A.
5. **Текст на картинке:** точная копия в кавычках, дословно, с типографикой инлайн («large serif font», «bold white uppercase sans-serif headline»). То же в официальном гайде OpenAI по gpt-image-2: текст в кавычках/КАПСОМ, шрифт+размер+цвет+позиция как constraints, сложные слова — по буквам, quality `high` для мелкого текста.
6. **Постерная дисциплина из static-ads workflow:** дробные координатные зоны (`text_zone: top 0.10–0.35`), **safe-zone правило** — верхние и нижние 10% кадра свободны от текста/лого/кнопок (UI соцсетей их перекрывает), разделение «структура из референса / визуальная айдентика из брендбука».
7. **Против «пластикового» лица gpt-image-2:** не писать «photorealistic» — вместо этого язык плёночной фотографии («35mm film photograph», «direct camera flash», grain). Стиль — **One Style Anchor Rule**: 1 якорь + 1–2 поддерживающих токена, «beyond 2–3 style tokens model attention dilutes»; слово «cinematic» само по себе «adds zero information».
8. **Вариации N кандидатов:** различать **stylistic fan-out** (N разных осей: свет/ракурс/палитра при том же сюжете — для исследования) и **variance-harvesting** (N бросков одного и того же промпта — когда промпт верен, а бросок неудачен); при итерации менять **ровно одну переменную** за раз.

---

## Раздел 1. Higgsfield skill — что нашёл

### 1.1 Официальный репозиторий: `higgsfield-ai/skills`

URL: https://github.com/higgsfield-ai/skills — «AI agent skills for image/video generation via Higgsfield AI… Works with Claude Code, Cursor, Codex». Ставится как Claude Code plugin (`/plugin marketplace add higgsfield-ai/skills`). Пять скиллов, все гоняют CLI `higgsfield`; для обложек релевантны `higgsfield-generate`, `higgsfield-product-photoshoot`, `higgsfield-marketplace-cards`.

**Ключевой архитектурный факт: сборка image-промпта скрыта на бэкенде.** Из `higgsfield-product-photoshoot/SKILL.md` (https://raw.githubusercontent.com/higgsfield-ai/skills/main/higgsfield-product-photoshoot/SKILL.md):

> «The CLI calls a **backend prompt enhancer that holds mode-specific photography vocabulary and structural templates**, then submits to `gpt_image_2` and returns image URLs.»

> «5. **Never write the gpt_image_2 prompt yourself — backend assembles it.**»

> «Calling `higgsfield generate create gpt_image_2 --prompt ...` directly instead of `higgsfield product-photoshoot create` — **bypasses the prompt enhancer and produces noticeably worse output**.»

То есть Higgsfield считает, что LLM-агент должен передавать только *намерение* (`--prompt "bottle of cold-brew on a sunlit kitchen counter, IG feed"`), а фотословарь и структуру держит выделенный слой шаблонов. Сами шаблоны непубличны.

**Структура промпта (официальный справочник `higgsfield-generate/references/prompt-engineering.md`, дословно):**

> «Higgsfield models reward concrete, sensory prompts.
> - **Subject + setting + style**: "a red fox curled in a snowy pine forest, golden hour, cinematic"
> - **Camera**: lens (35mm, 85mm), angle (low, overhead), motion (dolly in, tracking shot)
> - **Lighting**: rim light, neon glow, moody backlight
> - **Style/medium**: oil painting, watercolor, photograph, anime, 3D render
>
> **Keep it under ~200 tokens. Models distort with very long prompts.**»

И про негативы:

> «Most models don't expose a `negative_prompt`. Phrase positively: Instead of "no blur" → "tack sharp"; Instead of "no people" → "uninhabited landscape".»

**Анти-однотипность (раздел Multi-variant, дословно):**

> «`--count 3` returns 3 distinct image URLs. **Backend asks the enhancer to vary preset, lighting, angle, and palette across variants — they will not be paraphrased copies of one another.**
> For `social_carousel` and `ad_creative_pack`, count = number of slides / variants in the pack. **Backend locks the visual system across all slides automatically.**»

Итого 4 официальные оси рандомизации серии: **preset, lighting, angle, palette** — и обратный режим «замок визуальной системы», когда серия должна быть узнаваемо единой.

**Параметризация вариаций через сцены.** В `COOKBOOK.md` (https://github.com/higgsfield-ai/skills/blob/main/COOKBOOK.md) N кандидатов из одного брифа задаются перечислением *разных сцен внутри одного промпта*:

> `--prompt "founder using product in 5 distinct IG-feed scenes: morning coffee setup, desk workspace, outdoor café, gym, home office" --count 5`
> «Backend assembles the prompt, varies preset/lighting/angle/palette across the 5 outputs.»
> «Use `--count 5` on photoshoot first — cheaper than 5 separate runs and **the backend coordinates the visual system across variants**.»

**Стилевые словари, которые скилл показывает пользователю** (интервью перед генерацией, ≤4 вопроса с вариантами): стиль — `[Clean studio / Lifestyle / Conceptual / With a model]`; эстетики для рестайла — `[Clean girl / Cottagecore / Quiet luxury / Dark academia / Y2K]`; сезонность — `[Christmas / Valentine's / Halloween / Black Friday]`; окружение — `[Studio clean / Outdoor natural / Street style / Editorial / Home cozy]`; кадрирование — `[Full body / Three-quarter / Waist up / Closeup on product area]`. 10 режимов (`product_shot`, `lifestyle_scene`, `closeup_product_with_person`, `moodboard_pin`, `hero_banner`, `social_carousel`, `ad_creative_pack`, `virtual_model_tryout`, `conceptual_product`, `restyle`), каждому бэкенд сам подбирает aspect ratio; resolution всегда `2k`.

**Выбор модели под текст на картинке** (`references/model-catalog.md`): «**GPT Image 2** — Default high-fidelity image generation. **Graphic design, UI, banners, typography, and any brief with on-image text.**»; Recraft V4.1 — «logos, icons, flat illustrations, controlled-palette visuals»; Soul 2.0 — «looks like a magazine cover» editorial.

**Текст/слоганы в DTC Ads** (`references/marketing-dtc-ads.md`): типографика управляется через **ad formats** — курируемые пресеты «that drive the visual structure of a generated image (`headline`, `bullet-points`, `us-vs-them`, etc.)», выбор формата обязателен (`--format-id`), плюс **brand kit** (цвета, шрифты, тон с сайта бренда) и `--batch-size 1..20`. Поддерживаемые AR включают `3:2`.

### 1.2 Официальный prompt-гайд Higgsfield (blog) и «house style»

- Блог SOUL 2.0 (https://higgsfield.ai/blog/SOUL-2.0-Realistic-AI-Image-Generator-for-Creative-Direction): «Approach your prompt **like a creative brief** – define the mood, references, and visual intent»; 20+ **пресетов как aesthetic anchors** («Y2K studio», «Street photography», «Flash editorial») — «This instantly structures lighting, texture, and tone. From there, your prompt **refines rather than builds from scratch**»; смена мира при стабильном персонаже: «keep the same character while shifting environments, lighting logic, camera feel, and cultural references».
- Сводка канонической структуры из Nano Banana Pro guide Higgsfield (цит. по https://github.com/LumabyteCo/clarifyprompt-mcp/blob/main/packs/higgsfield-creative-handbook.md): «Higgsfield's house style is **long-form natural-language prose, not keyword-tag soup**»:

```
[shot type / POV] [subject + key action] [setting + time-of-day] [lighting]
[textures + materials] [color palette + mood] [style cues + lens / film cues]
```

### 1.3 Комьюнити-корпус: `OSideMedia/higgsfield-ai-prompt-skill` (самый детальный публичный источник)

URL: https://github.com/OSideMedia/higgsfield-ai-prompt-skill — библиотека из ~25 скиллов; gpt-image-2-часть «translated from Adil Aliyev's `gpt-image-2-director` source corpus» (автор, обозначенный как «Higgsfield-team-adjacent»). Это ближайшее публичное приближение к тому, что у Higgsfield зашито в бэкенд-enhancer.

**A. Трёхформатная таксономия промптов gpt-image-2** (`skills/higgsfield-gpt-image-2/SKILL.md`):

| Формат | Когда | Тип результата |
|---|---|---|
| **A — Structured JSON** | есть дискретные зоны, лейблы, инфо-иерархия | UI-мокапы, инфографика, **multi-panel posters, magazine layouts**, brand-identity boards |
| **B — Dense cinematic prose** | одна сцена, один субъект, без «хрома» | портреты, сцены, иллюстрации |
| **C — Auto-derive meta-prompt** | дана только тема, модель сама выводит композицию | «make a poster about X» |

Tie-break: при сомнении между A и B — **A**, потому что «Layout precision is GPT Image 2.0's primary differentiator».

Ключевые свойства модели, из которых выведены форматы: «**Text rendering** … embed real text in quotation marks exactly as it should render; do not paraphrase»; «**Design and UI as sweet spot** … magazine covers, infographics»; «**Cinematic photorealism is the weakness.** Human faces often go plasticky on realism-flagged prompts… frame it as film photography (grain, flash, 35mm) rather than as "photorealistic"».

Поля Format A (JSON): `type` («infographic poster»), `style` («GTA V cover art style, cel-shaded, thick black panel borders»), `subject`/`character`, `layout` (вложенные регионы: `header`, `centerpiece`, `left_side`, `grid_panels`…), `background`, текст в кавычках. Паттерны: **count-and-label** (`"count": 7` + параллельный массив labels), **position-scoped regions** («top-left», «mid-right»), **inline typography** («title in large serif font», «Space Grotesk Bold Caps»).

Порядок Format B (проза, один абзац): «image type / medium → main subject with specific visual details → pose or action → background / setting → environmental details → lighting → color palette / film stock / texture → mood descriptor», причём «put the most concrete visual anchors early enough that the model commits to them».

Скелет Format C (мета-промпт для постера из темы, дословно):

```
Please automatically generate a [output type] centered around [THEME].
Require the AI to automatically derive and uniformly design the entire following
visual system based on this theme, without my extra specification:
- [core subject, supporting structure, color hierarchy, material contrast, lighting, typography, …]
[Overall Style] …
[Composition Rules] — premium quality, central order, negative space, hierarchy
[Visual Quality] …
[Typography System] — ratio of visual to text, title/subtitle generation, font temperament
```

Чек-лист перед выдачей промпта (6 пунктов): покрытие регионов; явные counts/labels; реальный текст в кавычках; для лиц — нет слова «photorealistic»; **style line специфична, а не «generic adjective stacking»**; валидность JSON.

**B. Постеры/статичные креативы с текстом** (`skills/higgsfield-gpt-image-2/static-ads-workflow.md`):

- **Дробные координатные зоны** (доли высоты кадра): `"text_zone": {"top": 0.10, "bottom": 0.35}, "product_zone": {"top": 0.40, "bottom": 0.77}, "button_zone": {"top": 0.81, "bottom": 0.91}, "disclaimer_zone": {"top": 0.91, "bottom": 0.97}`.
- **Safe-zone HARD RULE:** «Keep the top 10% and bottom 10% of the frame free from text, logos, icons, buttons, and UI elements — photographic content such as hands, arms, or product edges entering the frame is fine» — потому что «Instagram and TikTok's UI overlays consume the top and bottom ~10% of the frame».
- **Brand-vs-structure separation:** из референса брать только layout/пропорции/типы элементов; фон, шрифты и акцентные цвета — только из брендбука, с явным override в теле промпта: «Do not carry over any colour, typeface, or visual treatment from the reference — those belong to another brand».
- **Wireframe intermediation:** вместо чужого референса модели передают брендо-нейтральный каркас с подписанными прямоугольниками зон — наследуется структура без стилевого заражения.
- **spec.json на серию:** один артефакт со всеми входами и массивом `variations: [{slug, prompt}]` — кандидаты отличаются полностью разрешёнными zone-based промптами (разный копирайт/наполнение зон при общей сетке).
- Типографика в шаблонах задаётся прямым текстом: «large bold white **uppercase sans-serif headline** "[OFFER HEADLINE]"», «small [ACCENT COLOR] **all-caps label** "[URGENCY TAG]" in a rounded pill shape», «headline: "[INGREDIENT NAME]." — just the ingredient name with a period, confident and clinical». Негативные директивы против «рекламного дефолта»: «Do NOT use polished ad layouts», «No stars, no reviews, no CTA button».
- Паттерн точечного редактирования: перечислить правки и закрыть **load-bearing клаузой** «keep everything else exactly the same» — иначе кадр «дрейфует».

**C. Словари стиля и света** (`skills/higgsfield-style/SKILL.md`):

- **Color grade по настроению** (10 строк): «Cold thriller: teal and orange, desaturated, high contrast»; «Warm nostalgia: golden hour amber, soft shadows, low contrast»; «Sci-fi cold: ice blue and silver, stark white light»; «Epic fantasy: rich jewel tones, deep shadows, volumetric light»…
- **Свет:** бытовой словарь (golden hour, overcast, neon, volumetric, practical only, side-lit, backlit, low key, high key) + **13 именованных киношных схем**: Rembrandt, Butterfly/Paramount, Split, Rim/backlit, Motivated, Practical, Chiaroscuro, High-key, Low-key, Golden hour, Blue hour, Harsh midday sun, Overcast diffused/softbox — с колонками «Effect» и «Best for».
- **One Style Anchor Rule:** «ONE primary style anchor beats five adjectives. Beyond 2–3 style tokens, model attention dilutes… **Wrong:** "cinematic, anamorphic, moody, atmospheric, dramatic lighting, film grain, desaturated, noir-inspired, high contrast, vintage feel" **Right:** "anamorphic, subtle grain, muted palette"».
- **«Cinematic does nothing»:** заменять на конкретику — «shallow depth of field, warm highlights, cool shadows» / «35mm film stock, natural grain, Kodak Portra palette».
- **Period control:** не «1970s style», а материалы+свет эпохи: «Kodachrome warm tones, wood paneling, orange shag carpet, tungsten bulbs casting amber light».
- **CGI Material Contract:** для предметки 2–4 свойства материала на поверхность (base/roughness/imperfection/edge), чтобы убить «default plastic sheen».

**D. Композиция/ракурсы** (`skills/higgsfield-image-shots/SKILL.md`): 10 крупностей (EWS → Macro, включая Cowboy Shot), 10 ракурсов с эмоциональной семантикой («Low = power, High = vulnerability, Dutch = unease»), «движения камеры» как композиционные ключи одного кадра. Полная формула стилла:

```
[Shot size] + [Angle] + [Movement keyword] of [character description].
[Pose, expression, or action].
[Environment — location, weather, atmosphere].
[Lighting — time of day, source, quality].
[Style — cinematic, film stock, color grade].
```

**E. Вариации и анти-однотипность** (`skills/higgsfield-prompt/SKILL.md`): формула **MCSLA** (Model, Camera, Subject, Look, Action) как пять слоёв чек-листа; и главное — дисциплина вариаций:

> «**Batch-and-Select (Variance-Harvesting) — Not the Same as Stylistic Fan-Out.** …hold the same locked prompt constant, roll N at once… This is the opposite of the stylistic-fan-out exception — that varies N *different looks*; this rolls N *identical* attempts… **fan-out explores, harvest exploits.**»

> «**Change exactly one variable per attempt.** Subject detail, composition, motion behavior, lighting, or style — pick the one that's wrong, change only that, regenerate.» Исключение: «once the prompt is locked and you're varying purely for stylistic exploration (e.g., **five lighting variants of an already-approved scene**), batching changes is fine.»

Отбор из батча — не «самая красивая», а hard-gate по инвариантам (identity, **text legibility**, названные якоря), потом скоринг по оси, ради которой перегенерировали.

### 1.4 Прочие комьюнити-скиллы (кратко)

- `pixelab-ch/higgsfield-skills` (https://github.com/pixelab-ch/higgsfield-skills) — 15 скиллов-«стилей» (cinematic, 3d-cgi, brand-story, ecommerce-ad…), каждый = SKILL.md с model routing + references (`camera.md`, `hooks.md`, `examples.md`); подтверждает паттерн «скилл = стилевой пресет + словарь».
- `AKCodez/higgsfield-claude-skills` (https://github.com/AKCodez/higgsfield-claude-skills) — 19 скиллов, UGC-пайплайн через Playwright; для image-промптинга вторичен.
- `robonuggets/higgsfield-skill` — тонкая обёртка над MCP `https://mcp.higgsfield.ai/mcp`; пример юзкейса дословно: «Generate a poster with GPT Image 2 through my higgsfield sub».

**Чего НЕ нашёл:** сами тексты бэкенд-шаблонов product-photoshoot/marketplace-cards (Higgsfield явно держит их закрытыми: «Backend owns marketplace compliance references and prompt templates»); seed-параметров в CLI-доках нет — вариативность у них строится не на сидах, а на осях preset/lighting/angle/palette и батчах.

---

## Раздел 2. Общие best practices image-промптинга для обложек/постеров с текстом

### 2.1 OpenAI — официальный prompting guide для gpt-image-2

Источник: https://developers.openai.com/cookbook/examples/multimodal/image-gen-models-prompting-guide (+ https://openai.com/academy/image-generation/):

- **Порядок блоков:** «Write prompts in a consistent order (**background/scene → subject → key details → constraints**)» и «include the intended use (**ad, UI mock, infographic**) to set the "mode" and level of polish». Для сложных запросов — «short labeled segments or line breaks instead of one long paragraph».
- **Текст на картинке:** «Put literal text in **quotes or ALL CAPS** and specify typography details (**font style, size, color, placement**) as constraints. For tricky words (brand names, uncommon spellings), **spell them out letter-by-letter**… Use `medium` or `high` quality for small text, dense information panels, and multi-font layouts.» Пример из Academy: `Add the headline "WEEKLY PLAN" in bold sans-serif, white, centered at the top, 72pt. No other text.`
- **Маркетинговые креативы:** «Put the exact copy in quotes, **demand verbatim rendering (no extra characters)**, and describe placement and font style. If text fidelity is imperfect, keep the prompt strict and iterate.»
- **Инварианты против дрейфа:** «clearly separating what should change from what must remain invariant, and **restating those invariants on every iteration** to prevent drift»; «Change only X. Keep everything else exactly the same» — совпадает с higgsfield-паттерном Mode C.
- Промпт лучше писать «like a creative brief rather than a purely technical image spec» — задать концепт, композицию и точный текст, остальное отдать «taste-driven» решениям модели.

### 2.2 Midjourney — консистентность серии и её дозирование

Источники: https://docs.midjourney.com/hc/en-us/articles/32180011136653-Style-Reference, https://docs.midjourney.com/hc/en-us/articles/41308374558221-Style-Creator, разборы практиков (https://promptyze.com/midjourney-style-reference-character-reference-the-real-way-to-build-consistent-brand-assets/, https://tools.inyourleague.net/en/midjourney-style-reference-sref-consistent-brand-visuals-guide-en/):

- `--sref` переносит «color palette, lighting, texture, rendering style, mood», но не сюжет; `--sw` (0–1000, дефолт 100) дозирует силу. Для брендовых серий практики советуют **200–500**: «strong enough to feel cohesive, **loose enough that individual images don't look like clones**» — это точная формулировка баланса «серийность без однотипности».
- Blend двух референсов с весами: `--sref URL1::75 URL2::25` (один даёт палитру, другой свет).
- Seeds официально «unreliable for style consistency» — для стиля серии использовать sref/style-код, не seed.
- Кросс-модельный вывод (styleref.io): вне Midjourney та же задача решается **текстовой стиль-спекой** — сохранённый блок правил (палитра/свет/текстура/гардрейлы), который префиксом вставляется в каждый промпт серии. Это ровно «brand kit / visual system lock» Higgsfield, только в тексте.

### 2.3 Сводка приёмов анти-однотипности (по всем источникам)

1. **Фиксируй систему, варьируй оси.** Один слой констант (палитра бренда, типографика, зоны, стиль-якорь) + явные оси вариаций: **сцена/сюжет, свет (именованная схема), ракурс/крупность, палитра-настроение, пресет-эстетика** (Higgsfield: preset/lighting/angle/palette; Midjourney: sref + sw).
2. **Вариант ≠ парафраз.** Кандидаты серии должны отличаться значением на осях, а не синонимами тех же слов («they will not be paraphrased copies of one another»).
3. **Разводи fan-out и re-roll:** N разных осей — когда ищешь направление; N бросков одного промпта — когда промпт верен, но бросок неудачен.
4. **Один стиль-якорь**, а не стопка прилагательных; «cinematic» и «photorealistic» — мусорные токены (второй ещё и ломает лица).
5. **Отбор по инвариантам:** сначала hard-gate (читаемость текста, айдентика, композиция), потом вкус.

---

## Раздел 3. Что переносимо в наш конвейер (gpt-image-2, обложки TG, русский слоган КАПСОМ, ЦА — фитнес-тренеры)

Наш кейс — ровно «сладкая зона» gpt-image-2 по обеим таксономиям: постер с коротким текстом → **Format A (structured JSON)** или зонная проза. Конкретно:

1. **Собрать свой «backend enhancer» как слой скилла.** Урок №1 от Higgsfield: агент передаёт короткое намерение, а финальный промпт собирает детерминированный шаблон со словарями. У нас это должен быть блок в skill обложек: фикс-слой (бренд-палитра, типографика, зоны, стиль-якорь) + слоты (сюжет, свет, ракурс, палитра-настроение). Не давать модели «freehand» на каждый пост — иначе получим и разнобой качества, и однотипные штампы.

2. **Структура промпта обложки** (синтез Higgsfield Format A + OpenAI order):
   - `type`: "editorial cover image for a Telegram post" (intended use задаёт «mode» полировки — OpenAI);
   - `style`: ОДИН якорь + 1–2 поддержки (например «35mm film photograph, natural grain, muted warm palette» — и никогда «photorealistic», у нас в кадре люди-тренеры → риск пластиковых лиц);
   - `scene/subject`: конкретика уровня «white ribbed tank top», не «спортивная одежда»; конкретные пропсы зала/телефона/планшета тренера;
   - `layout` с зонами в долях высоты: слоган-зона, субъект-зона; **top/bottom 10% свободны** — у Telegram превью тоже подрезает/накладывает элементы, правило переносится как есть;
   - `text`: слоган в кавычках дословно, КАПСОМ, + типографика инлайн: `bold white uppercase sans-serif headline "…", centered in the top text zone` + «no other text, render verbatim, no extra characters»;
   - `background/palette`: из брендовой строки (наш аналог brand kit).
   Держать весь промпт компактным (Higgsfield: ~200 токенов деформация; для JSON-раскладок допустимо больше, но без adjective stacking).

3. **Русский слоган.** Прямых данных о кириллице в этих гайдах нет (OpenAI подтверждает mixed scripts CJK+Latin, комьюнити-скилл — CJK). Переносимые страховки: слоган короткий, КАПС (совпадает с нашим форматом и советом ALL CAPS), quality `high`, verbatim-клауза, и hard-gate «текст читаем и без опечаток» при отборе кандидатов. Если буква ломается системно — приём «по буквам» из OpenAI-гайда.

4. **Анти-однотипность серии обложек — таблица осей вместо сидов.** Прямо скопировать механику Higgsfield: для N кандидатов одного поста и для ленты канала в целом варьировать по 4 осям:
   - **сцена**: зал / телефон в руке / стол тренера / улица-пробежка / раздевалка / скриншот-метафора;
   - **свет**: именованные схемы из style-словаря (golden hour, high-key softbox, practical gym lighting, rim light, blue hour…);
   - **ракурс/крупность**: EWS…CU + eye-level/low/high/overhead/POV (у каждого — эмоциональная семантика: low = сила, overhead = система/порядок);
   - **палитра-настроение**: 3–4 брендовых грейда типа «warm energetic amber», «clean high-key neutral», «deep focused teal».
   Кандидаты одного брифа = разные комбинации осей (fan-out), а не парафразы. Вести реестр использованных комбинаций по последним 10–15 постам — «замок визуальной системы» (бренд-слой) при обязательной смене осей.
5. **Итерация:** если кандидат «почти» — менять одну переменную; если все кандидаты валидны, но слабы по-разному — это стохастика, перебрасывать тем же промптом, не переписывая.

6. **Рубрики = пресеты.** Наши 🔥/✨/💡 рубрики можно оформить как «aesthetic anchors» в духе Soul-пресетов: каждой рубрике — свой стиль-якорь + дефолтная палитра, чтобы рубрика узнавалась в ленте, а оси сцена/свет/ракурс давали разнообразие внутри.

---

## Источники

**Официальные (Higgsfield):**
- https://github.com/higgsfield-ai/skills — официальный репозиторий скиллов (Claude Code plugin)
- https://raw.githubusercontent.com/higgsfield-ai/skills/main/higgsfield-generate/references/prompt-engineering.md — структура промпта, лимит ~200 токенов, позитивные негативы
- https://raw.githubusercontent.com/higgsfield-ai/skills/main/higgsfield-product-photoshoot/SKILL.md — backend enhancer, 10 режимов, оси вариаций preset/lighting/angle/palette, lock визуальной системы
- https://raw.githubusercontent.com/higgsfield-ai/skills/main/higgsfield-generate/references/model-catalog.md — GPT Image 2 как дефолт для typography/on-image text
- https://raw.githubusercontent.com/higgsfield-ai/skills/main/higgsfield-generate/references/marketing-dtc-ads.md — ad formats (headline/bullet-points/us-vs-them), brand kit, batch 1–20
- https://raw.githubusercontent.com/higgsfield-ai/skills/main/higgsfield-marketplace-cards/SKILL.md — закрытые шаблоны на бэкенде
- https://github.com/higgsfield-ai/skills/blob/main/COOKBOOK.md — «5 distinct scenes» в одном промпте, cheap-first iteration
- https://higgsfield.ai/blog/SOUL-2.0-Realistic-AI-Image-Generator-for-Creative-Direction — пресеты как aesthetic anchors, «prompt like a creative brief»

**Комьюнити-скиллы (Higgsfield-экосистема):**
- https://github.com/OSideMedia/higgsfield-ai-prompt-skill — библиотека скиллов; конкретно: `skills/higgsfield-gpt-image-2/SKILL.md` (Format A/B/C), `skills/higgsfield-gpt-image-2/static-ads-workflow.md` (зоны, safe-zone 10%, brand-vs-structure, 3 шаблона), `skills/higgsfield-style/SKILL.md` (словари грейдов/света, One Style Anchor), `skills/higgsfield-image-shots/SKILL.md` (крупности/ракурсы, формула стилла), `skills/higgsfield-prompt/SKILL.md` (MCSLA, fan-out vs harvest, one-variable iteration)
- https://github.com/LumabyteCo/clarifyprompt-mcp/blob/main/packs/higgsfield-creative-handbook.md — каноническая проза-структура из Nano Banana Pro guide
- https://github.com/pixelab-ch/higgsfield-skills, https://github.com/AKCodez/higgsfield-claude-skills, https://github.com/robonuggets/higgsfield-skill — второстепенные обёртки/стилевые паки

**Общие best practices:**
- https://developers.openai.com/cookbook/examples/multimodal/image-gen-models-prompting-guide — официальный OpenAI prompting guide для gpt-image-2 (порядок блоков, текст в кавычках, verbatim, letter-by-letter, quality high)
- https://openai.com/academy/image-generation/ — OpenAI Academy: текст на картинке, invariants при правках
- https://docs.midjourney.com/hc/en-us/articles/32180011136653-Style-Reference и https://docs.midjourney.com/hc/en-us/articles/41308374558221-Style-Creator — sref/style codes
- https://promptyze.com/midjourney-style-reference-character-reference-the-real-way-to-build-consistent-brand-assets/ — sw 200–500 «cohesive, not clones», blend с весами
- https://tools.inyourleague.net/en/midjourney-style-reference-sref-consistent-brand-visuals-guide-en/ — production workflow серии, reject pile
- https://styleref.io/blog/midjourney-style-reference — текстовая стиль-спека как кросс-модельный аналог sref

**Не найдено / ограничения:** тексты бэкенд-шаблонов Higgsfield (product-photoshoot, marketplace-cards, ad-formats) непубличны; seed-стратегий в Higgsfield CLI-доках нет; данных о качестве кириллицы в gpt-image-2 в изученных гайдах нет — только CJK/Latin.
