# Log — Bravo Marketing Knowledge

Хронолог изменений в knowledge base. Дописываем строкой при каждом значимом событии (ingest, lint, hard-contract change, кампания запущена/закрыта).

Формат: `YYYY-MM-DD action — описание (автор)`

---

2026-05-24 bootstrap — vault создан, добавлен schema (Steve)
2026-05-24 ingest [bravo-product](../raw/product-docs/bravo-product.md), [target-audience](../raw/product-docs/target-audience.md) → seeded [[young-trainer]], [[experienced-trainer]], [[positioning]] v0, [[brand-voice]] v0 (Steve)
2026-05-24 hard-contract edit — WhatsApp → Telegram в [[positioning]], [[hook-formulas]], [[experienced-trainer]]. Причина: на ru-рынке ЦА тренеров общается с клиентами в Telegram, а не в WhatsApp. PR pr/whatsapp-to-telegram-and-campaign-warmup, approve @arkoval1 (Steve)
2026-05-24 campaign created — [[0001-warmup-content]] (Steve)
2026-05-25 insight — реальная воронка от Артёма: ~5-7 установок/нед → 1-2 профиля → 0 платящих. Горлышко = trial→paid, не трафик/контент. Зафиксировано в [[learnings]] (Steve)
2026-05-25 insight — канал [[telegram]] @bravo_pro уже ведётся (~37 постов, 21 подписчик, просмотры 20-50); TG Ads дал ~240₽/подписчик. Обновлён [[telegram]] (Steve)
2026-05-25 archived — [[0001-warmup-content]]: посылка «канал пустой» неверна, витрина уже есть (Steve)
2026-05-25 linear setup — team Marketing [Bravo]: проекты Activation / ASO / Контент-движок + задачи BMK-10..15; фокус 0→1 = активация (Steve)
2026-05-25 insight — выгрузка базы (28 рег → 9 профиль → 0 платящих; 74% установок из поиска App Store). Лина: free-подписка ≠ использование → барьер не в цене. Зафиксировано в [[learnings]] (Steve)
2026-05-25 linear — заведены 28 подзадач customer dev (BMK-17..44) под BMK-10, по сегментам ACTIVE/TRIAL/EXPIRED±профиль; скрипты outreach (2 док) + ASO-аудит (док) (Steve)
2026-05-25 research → создан [[competitors]] (ближайший аналог TRENERA.PRO) + ASO-семантика (BMK-14): ядро job-запросов «запись клиентов / CRM для тренера / учёт-база» vs низкоцелевые виды спорта. Источники vc.ru/picktech/лендинги (Steve)
2026-05-25 content → первая пачка постов (BMK-45/46/47) написана hype, отревьюирована, скорректирована по фидбэку Артёма. Создан [[demo-profiles]] (9 демо + правило ссылок демо/лендинг). Формат постов дополнен: заголовок-плашка + дата публикации (skills write-telegram-post, brief-content-task) (Steve)
2026-05-25 content → кампания BMK-15 добита: 12 постов (BMK-45..56) написаны hype, все In Review, даты 26.05–17.06 (Steve)
2026-05-25 concept → создан [[content-strategy]]: рубрики (тематические + фича), фича-серии (боль → без нас → с нами + ограничители против перегиба), источники тем = customer dev. По согласованию с Артёмом (Steve)
2026-05-25 concept → [[content-registry]] (реестр тем, антиповтор) + skill plan-content-campaign. Обновлён index. Контент-процесс замкнут: стратегия → план кампании → задачи → hype → метрики → learnings/реестр (Steve)
2026-05-25 review → 1-й раунд приёмки BMK-15: Артём принял 4 поста (#1/2/3/11), 8 на доработку. hard-contract edit [[brand-voice]] — принцип «От боли — к возможности» (approve Артёма). Добавлен раздел «Чего НЕ обещаем» в [[proof-points]] (нет импорта/программ/платежей) + skill write-telegram-post его открывает. Все 8 переписаны hype (v2), снова In Review. Паттерны фидбэка → [[learnings]] (Steve)
2026-05-25 triage → 2-й раунд правок BMK-15 (необработанный фидбэк Артёма по v2): посты #4/#5/#10 переписаны v3 (BMK-48/49/54), #9 новый самостоятельный заголовок (BMK-53), #12 принят заголовок Артёма (BMK-56) — все In Review. Паттерн раунда-2 (смысловые ямы / «иишный» тон / несамостоятельные заголовки) → [[learnings]] + предложена правка skill write-telegram-post (PR) (Steve)
2026-05-25 infra → запущена облачная routine «СМО проверка задач в работе» (Claude Code on web): 3×/день (07/15/21 МСК) автономный обход Linear, ответы/работа в рамках порога автономии, сводка в Pachca. Детекция по префиксу `[Steve]`; правило ack-на-каждый-коммент в skill work-with-artem. Процедура — skill daily-linear-triage; невоспроизводимый из репо веб-конфиг — в agent-памяти. Также фикс: валидатор длины (UTF-8 локаль). (Steve)
2026-05-26 infra → модели на Opus 4.7: hype (контент-субагент) переведён Sonnet → Opus 4.7 + основная сессия routine на Opus 4.7. Решение Артёма — выше качество контента после 2 раундов правок за «иишность». (Steve)
2026-06-06 lint → fixed 17 broken wikilinks (raw-ссылки → md-link, удалены отсылки к несуществующим страницам), resolved 2 orphans ([[content-registry]] и [[goals]] подключены из [[content-strategy]]). Hard contracts ([[positioning]], [[brand-voice]]) содержат `[[raw/...]]`-ссылки на источники — не правил по правилам lint'а, флаганул отдельно. PR в knowledge репо. (Steve)
