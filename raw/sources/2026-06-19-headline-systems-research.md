---
type: source
status: active
updated: 2026-06-19
---

# Системы генерации заголовков для LLM — ресёрч (Exa, 2026-06-19)

Под задачу «заголовки слабоваты — нужна система, не один эталон». Источник для [[headline-craft]] и [[headline-swipe]].

## Готовые скиллы (архитектура подтверждена)

- **Headline Lab** (copywriting.ai, Mark Masters) — Claude skill: генерь 10 по 5 механизмам (curiosity-gap, specific-benefit, contrarian, fear/risk, identity-call) → скорь 1–10 (specificity, emotional pull, clarity) → рекомендуй топ-3. Файлы: `headline-formulas.md` (банк), `winning-headlines.md` (swipe-file), `product-context.md`.
- **hook-and-headline-writing** (skill-stack) — 15 формул, 4 U's тест, «generate volume 10+, then select; first option rarely best».
- **ClaudeKit /hook-finder** — angles → variations → score → refine. «Generation is easy; judgment is the skill. Give a rubric and make it cut.»

→ Наша форма (6–8 механизмами → отбор → пачка) верна. Не хватало трёх вещей ниже.

## 1. Override на удар (лекарство от «диагноза»)

setaffiliatebusiness: после скоринга — **пометь заголовки с высоким Curiosity + Differentiation как приоритет, ВНЕ общего балла**. «Заголовок, который удивляет И выделяется, бьёт технически-правильный — он прерывает автопилот (pattern interrupt)». Прямо чинит нашу беду: «диагнозы» корректны, но не удивляют.

## 2. Rubric отбора (строже галочек)

- **4 U's** (Bly/Masterson): Useful · Urgent · Unique · Ultra-specific, балл 1–4; <10/16 = переписать; 14+ давали 3× отклик. Диагностический, работает под любой формат.
- **S.C.O.R.E.** (Dietrich): Specificity, Curiosity, Outcome, Restriction, Emotion + диагностика до генерации.
- **Stop-scroll axis** (toolsforwriting «Four Jobs», robpalmer): «если б увидел среди 10 похожих — остановился бы?». Specificity > cleverness.
- **Pairwise-ранжирование** (discoveredlabs): сравнивать заголовки *попарно* «какой лучше?», турнир — надёжнее абсолютного балла.

## 3. Swipe-file вместо одного эталона

robpalmer / oscom: банк *выигравших* заголовков (winning patterns), растёт от того, что зашло по метрикам. «После 6 мес — карта психологии аудитории». Калибр из многих примеров, не из одного.

## 4. Формат-профиль

hashmeta/jenova/theseoagent: промпт под блог НЕ годится под email/обложку — у каждого свой лимит и драйвер. Форсить РАЗНЫЕ паттерны (2 на 6 механизмов), сравнивать поперёк углов. Для обложки: ≤5 слов, читается за 1 сек, фильтр лексики не-ЦА.

## Источники

timdietrich.me (S.C.O.R.E. meta-prompt), copywriting.ai (Headline Lab skill), promptwritingstudio.com (Headline Lab), setaffiliatebusiness.com (scorer + curiosity×differentiation override), getclaudekit.com (hook-finder), playbooks.com/skills (hook-and-headline-writing), toolsforwriting.com (Four Jobs rubric), robpalmer.com (25–50 variations + swipe-file), themarketingjuice.com + conversion.studio (4 U's), oscom.ai (swipe-file), discoveredlabs.com (pairwise ranking), hashmeta.ai/jenova.ai/theseoagent.ai (format-specific).
