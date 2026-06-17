---
type: source
status: active
updated: 2026-06-17
---

# AI-копирайтинг: мастерство, не патчи — deep-research (2026-06-17)

Полный харнесс: 105 агентов, 23 источника, 112 claims → 25 проверено adversarial → **17 подтверждено, 8 убито**. Под пересборку [[hype]].

## Подтверждено (adversarial 2-1 / 3-0)

**Плоскость ИИ правилами не лечится — нужен ПРОЦЕСС:**
- RLHF (за Claude/GPT) даёт measurable mode collapse / гомогенность письма (3-0).
- Prompt-engineering и param-tweaks **НЕ устраняют** это (3-0). → Чеклист запретов не чинит корень. (Подтверждает интуицию Артёма: whack-a-mole не работает.)

**Петля writer→critic→editor (Self-Refine):**
- Один LLM может итеративно улучшать свой выход generate→feedback→refine (3-0).
- Reward hacking сильнее, когда критик = генератор → **критик должен быть ОТДЕЛЬНОЙ персоной** (2-1).

**Процесс мастера:**
- **Volume-then-cull**: набрать объём вариантов, потом отбраковать — не «угадать с первого раза» (2-1).
- Master copy «собирается» из заранее собранного материала (3-0).
- Качество — из правки/редактуры, не первого прохода (2-1).
- Rubrics: **Ogilvy Big Idea** (5-point тест), **Sugarman slippery slide** (каждая фраза тянет к следующей) (3-0).

**Голос:** brand voice = explicit operational rules + эталоны (3-0).

## Убито (НЕ доверять, 0-3 / 1-2)

- «Саморефлексия даёт прирост везде» (0-3) — петля **НЕ магия**; работает только с отдельным критиком + конкретным rubric, не общее «улучши».
- «Perplexity/burstiness как measurable targets голоса» (0-3) — не гнаться за метриками burstiness.
- «4-компонентная prompt-архитектура» (1-2), «few-shot 5-15» (1-2), «research 80-90% работы» (1-2) — спорно, не догма.

## Caveats

Петля — про **стиль/голос, не факты**. Proof points остаются внешними (proof-points). Validators **дополняют** петлю, не заменяют.

## Вывод-архитектура для hype

Из «пиши по чеклисту» → «мастер + самокритика»:
1. **Персона-промпт:** мастер-копирайтер (школа Ogilvy / Sugarman / Schwartz) + голос Bravo.
2. **Voice card:** эталоны из brand-voice как few-shot.
3. **Процесс:** N углов → Big Idea тест → черновик → **отдельный критик-проход** (rubric: Big Idea / slippery slide / голос / якорь конкретики / read-aloud) → правка → validators.
4. **Volume-then-cull**, не один проход.

## Источники

arxiv: Self-Refine (2303.17651), Reflexion (2303.11366), Self-correction survey (2407.04549), Kirk RLHF-diversity (2310.06452); Ogilvy process (chasereeves.co), Schwartz (medium/the-shortform), Sugarman slippery slide (whiskeyandstardust adweek handbook); brand-voice prompting (searchengineland, copyhackers, hubspot, dotdigital).
