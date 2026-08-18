# Канон промптинга моделей Claude 5-го поколения (Anthropic, официальный)

Дата сбора: 2026-08-18. Собрано по запросу Артёма («антропики рекомендуют меньше зажимать модели 5-го поколения») — гипотеза подтверждена официальными источниками.

## Источники

- Anthropic migration guide → «Prompting Claude Opus 5» / «Migrating to Claude Fable 5» (platform.claude.com/docs, через skill claude-api)
- https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5
- https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices
- Блог Anthropic: «The new rules of context engineering for Claude 5 generation models» (2026-07-24) — https://claude.com/blog/the-new-rules-of-context-engineering-for-claude-5-generation-models
- Вторичные разборы: i-scoop.eu (2026-07-30), codewithseb.com «A De-Prescribing Guide» (2026-07-03), mindstudio.ai (2026-08-12)

## Ключевые тезисы (дословно или близко)

1. **«Prompts written for prior models are often too prescriptive and reduce output quality»** — официальная формулировка migration guide. Anthropic удалили **>80% системного промпта Claude Code** для Opus 5 / Fable 5 без измеримой потери качества на кодинг-эвалах.
2. **Запретительные стены вредят.** «Absolute bans» («default to writing no comments», «never write multi-paragraph docstrings») заменены на локальные адаптивные принципы («write code that reads like the surrounding code»). Конфликтующие наслоения правил заставляют модель тратить внимание на распутывание противоречий. Тревожный промпт → осторожная, зажатая модель.
3. **Пошаговые сценарии — потолок, не опора.** «Prefer general instructions over prescriptive steps»; жёсткие STEP 1..N модель исполняет буквально и останавливается, даже когда план должен меняться. Цель + рамки + критерии готовности > нумерованный чек-лист.
4. **Примеры сужают пространство поиска.** «Giving examples actually constrains them to a certain exploration space» — вместо worked examples вкладываться в выразительность параметров/инструментов. (⚠️ для нас: эталоны brand-voice — камертон вкуса, это «format-pinning» и «taste», их канон разрешает; но они должны покрывать весь спектр регистров — см. фидбэк «бубубу»: три серьёзных эталона утянули весь тон.)
5. **Verification scaffolding — удалить.** Opus 5 «verifies its own work without being told»; инструкции «double-check», «re-verify», отдельные шаги проверки вызывают over-verification. Убирать, не переписывать.
6. **Фильтры серьёзности давят recall.** «Only report high-severity» модель исполняет буквально: находит всё, но молчит о том, что ниже планки. Сначала полнота, фильтр — отдельным шагом.
7. **Правило с обоснованием применяется с суждением, голое правило — буквально.** Выжившие ограничения снабжать «почему».
8. **Один источник правды.** Повторы инструкций (system prompt + tool description) были нужны старым моделям — теперь удалять; инструкция живёт в одном месте, progressive disclosure вместо «всё в одном файле».
9. **Что ОСТАВЛЯТЬ:** контекст, который модель не может вывести сама (вкус, gotchas, бизнес-ограничения, definition of done), стиль-контракты (модель отлично им следует — вкладываться), границы автономии, evidence standards, точные скрипты для хрупких операций. «Cruft ≠ length» — вред от конкретных устаревших инструкций, не от объёма.
10. **Не де-прескрайбить на веру — A/B.** Снапшот промпта → вариант без лесов → 10–15 реальных задач → сравнение качества и токенов. Часть строк вернётся — как ограничения с обоснованием.

## Приложение к системе Bravo (решения 2026-08-18)

- `hype` переведён на **Opus 5 + effort max** (frontmatter + CLAUDE.md).
- Сняты скаффолды под старые модели: «грамотный текст с первого прохода = иишно» (hype.md, write-telegram-post) — Opus 5 пишет сильный первый проход, клеймо заставляло имитировать доработку.
- «Чего НЕ делать»-стены в hype.md и write-telegram-post сжаты до «Рамок»: остались бизнес-ограничения (правда продукта, публикация, hard contracts, негатив ≤ входа) — дубли валидаторов и writing-craft удалены (enforcement уже в коде/каноне; тезис 8).
- **Не** тронуты: валидаторы (executable checks — «настоящая страховка» по канону), hard contracts (banned-words = бизнес-ограничение Артёма), writing-craft (канон-источник; его прореживание — отдельный A/B по тезису 10 после первой пачки на Opus 5).
