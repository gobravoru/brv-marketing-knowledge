# Playbook — Post page template

После того, как пост опубликован и Артём собрал метрики, Hype создаёт страницу `wiki/posts/YYYY-MM-DD-<slug>.md` по этому шаблону.

## Шаблон

```yaml
---
type: post
channel: telegram
published: YYYY-MM-DD
linear-issue: CNT-NN
campaigns: [<campaign-slug>, ...]
hook-formula: pain-story | hard-truth | compare-then-flip | question-lead | other
icp-target: young-trainer | experienced-trainer | both
proof-used: <link или метрика>
metrics:
  views: NN
  reactions: NN
  forwards: NN
  link-clicks: NN
  installs: NN
  trials: NN
  paid: NN
---

## Текст поста

<копия текста как опубликовали>

## Заметки

- Что зашло: ...
- Что не зашло: ...
- Гипотеза для следующего: ...
```
