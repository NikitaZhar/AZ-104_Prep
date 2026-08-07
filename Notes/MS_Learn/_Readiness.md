Tags: #AZ-104 #meta

# Готовность материалов (readiness)

Обновлено: 2026-08-07. Источник статусов — поле `status` во frontmatter (см. [[_Unit Processing Contract]] §1).

## Схема Properties (актуальная)

- `type`: `LP` (официальный learning path AZ-104, треки 2–6) | `Additional` (prereq/extra, треки 0–1). Роль module/unit/lab в `type` **не** кодируется — она видна из нумерации и имени файла.
- `status`: `draft` → `unit-processed` → `tests-built` → `complete`.
- Поле `lp` удалено; удалены также legacy-поля `chapter` и `section`.
- Порядок полей: `created, type, status, part, module, mslearn, tags, prev, next`.

Сейчас все 80 заметок трека 0–1 имеют `type: Additional` (это prerequisites и tools, вне LP). `type: LP` появится с треками 2–6.

## Статус по модулям

| # | Модуль | Юнитов | Quiz | Practice | Lab | Блок «Тесты» | status |
|---|--------|:-----:|:----:|:--------:|:---:|:-----------:|--------|
| 0.1 | PowerShell (контейнер пути) | — | — | — | — | — | unit-processed |
| 0.1.1 | Review Azure PowerShell module | 7 | ✓ | ✓ | ✓ | есть | **tests-built** |
| 0.1.2 | Review features & tools for Azure Cloud Shell | 5 | ✓ | ✓ | ✓ | есть | **complete** |
| 0.1.3 | Manage Azure resources with Windows PowerShell | 6 | ✓ | ✓ | ✓ | есть | **complete** |
| 1.1 | Describe cloud computing | 7 | — | — | — | есть | unit-processed |
| 1.2 | Describe the benefits of using cloud services | 7 | — | — | — | нет | unit-processed |
| 1.3 | Describe cloud service types | 5 | — | — | — | нет | unit-processed |
| 1.4 | Describe the core architectural components of Azure | 6 | — | — | — | нет | unit-processed |
| 1.5 | Describe Azure compute services | 8 | — | — | — | нет | unit-processed |
| 1.6 | Describe Azure networking services | 6 | — | — | — | нет | unit-processed |
| 1.7 | Describe Azure storage services | 7 | — | — | — | нет | unit-processed |
| 1.8 | Describe Azure identity, access, and security | 11 | — | — | — | нет | unit-processed |

Standalone-заметки трека 0 (`0.2 Azure CLI`, `0.3 The Azure portal`, `0.4 ARM templates and Bicep files`) — `unit-processed`.

## Что осталось до `complete`

- **Собрать тесты.** Собраны для `0.1.1`, `0.1.2`, `0.1.3`. Для модулей 1.2–1.8 нужны Quiz (и Lab/Practice, где применимо) по [[_Quiz Builder Contract]].
- **Дописать блок «Тесты»** в индексы модулей — по контракту он обязателен всегда (даже с плейсхолдерами). Сейчас есть у `1.1`, `0.1.1`, `0.1.2`, `0.1.3`; у 1.2–1.8 отсутствует.
- **Перепроверить глоссарий** по трекам 1.x (линковка при миграции не пересобиралась — только Properties).
- `complete` ставится вручную после всех проверок модуля. `0.1.2` и `0.1.3`: Quiz/Practice/Lab фактически собраны и разложены по папкам 2026-08-07 (в статусе `complete` от 2026-08-06 файлы отсутствовали на диске). Верификация: аудит §5 чистый для всех четырёх наборов (Quiz 0.1.2 — 18 вопросов, Practice 0.1.2 — 13; Quiz 0.1.3 — 20, Practice 0.1.3 — 15); `node --check` по JS; jsdom-прогон (полный проход с верными ответами → N/N, лимит multi-select, ветка Incorrect, Restart) — PASS; движок в размещённых файлах дословно совпал с эталонным sha256 `796ea400…`.

## Трек 2 (LP) — не начат

`2 Manage identities & governance/2.1 Manage Microsoft Entra users and groups.md` — пока черновой список ссылок/квизов, ещё не разбит на модули/юниты и без frontmatter. Это первый настоящий `type: LP` контент.
