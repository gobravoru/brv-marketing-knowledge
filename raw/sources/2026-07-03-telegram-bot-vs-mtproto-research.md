# Telegram Bot API vs MTProto для публикации и аналитики канала (2025–2026)

Ресёрч под решение «перестраивать ли публикацию на MTProto» (вопрос Артёма). Первоисточники: core.telegram.org (Bot API docs + changelog 7.6→10.1, API schema, /api/stats), docs Telethon.

## 1. Bot API: аналитики просмотров НЕТ и не появилось

- В объекте `Message` нет поля `views`; методов статистики канала/поста не существует.
- Changelog 2024–2026 (7.6 → 10.1) проверен: ни одна версия не добавила ботам просмотры/статистику. Релизы были про форматы публикации и монетизацию: sendPaidMedia (7.6), платные реакции (7.9), подарки каналам (8.3), stories от бизнес-аккаунтов (9.0), checklists (9.1), suggested posts (9.2).
- Что бот-админ МОЖЕТ: `message_reaction_count` (суммарные анонимные реакции; только в момент события, истории нет — пропустил >24ч, данные потеряны; нужен `allowed_updates`), `getChatMemberCount`, boost-апдейты.
- ⚠️ В выдаче встречается SEO-фейк (puc-telegram.com) с несуществующим методом `getMessageStatistics` — дезинформация.

## 2. MTProto (api_id/api_hash, user-session): аналитика доступна

| Метод | Что даёт | Порог | Админство |
|---|---|---|---|
| `messages.getMessagesViews` | views по списку message_id | нет | не нужно |
| `messages.getHistory` / `iter_messages` | у Message поля `views`, `forwards`, `reactions`, `replies` | нет | не нужно (публичный канал) |
| `stats.getBroadcastStats` | полный дашборд (growth, views_by_source, interactions…) | ~500 подписчиков (`can_view_stats` в channelFull, server-side) | нужно |
| `stats.getMessageStats` | графики одного поста | тот же | нужно |

- Все методы — «only users», ботам BOT_METHOD_INVALID.
- stats.* слать в DC из `channelFull.stats_dc` (Telethon `client.get_stats()` делает сам).
- Для канала <500 подписчиков просмотры/форварды/реакции по постам — через `iter_messages`/`getMessagesViews`, без порогов.

## 3. Риски user-session (read-only, 1×/день)

- Банят в основном за исходящие действия (рассылки, инвайты, массовый скрейпинг); read-only опрос своего канала — минимальный профиль. FloodWaitError обрабатывать, session переиспользовать (не логиниться заново).
- RU-номера банят охотнее (Telethon FAQ); свежереги и VoIP — тоже. Митигация: отдельный «прогретый» реальный аккаунт (не личный), стабильный IP, проверка через @SpamBot.
- **Session-файл = полный доступ к аккаунту**: вне git, права 600/секрет; утечка → revoke sessions с телефона. Отдельный аккаунт с админкой «View Statistics» изолирует риск от основного.

## Вывод

Для доставки Bot API полностью достаточен; метрики постов достаёт только MTProto user-session — без порогов и админки через поля Message. Стандартная связка: **бот публикует + user-session читает метрики по расписанию**. Полная stats.* включится сама при ~500 подписчиков — теми же ключами.
