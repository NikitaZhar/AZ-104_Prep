# Azure Cert — Инструкция по сборке и проверке Quiz и Practice (самодостаточная)

Этот файл — **единственный и полный файл-задание для исполнителя (Claude)**, рассчитанный
на два равноправных сценария: **создание** нового Quiz/Practice с нуля и
**проверка/исправление** уже существующего файла на соответствие этой же инструкции
(раздел 0 описывает оба). В обоих случаях я сам готовлю или сверяю вопросы, сам
запускаю аудит и технические проверки, сам сверяю результат с требованиями. Внешние
файлы не нужны: эталонный движок встроен в раздел 7 целиком, и файл ни на что за
своими пределами не ссылается.

Назначение: подготовка к сертификации по трём слоям, все на одном движке из раздела 7:
**Quiz** (понимание материала) — для каждого модуля; **Practice** (применение + лабы) —
для модулей с практическим материалом (§2); **Cases и Mock** (§2a) — межмодульные, для
фазы «перед экзаменом». Слои отличаются содержанием, охватом и набором типов вопросов.

---

## 0. Рабочий цикл (что я делаю по шагам)

Два сценария, одна и та же инструкция (разделы 1–9) — различается только точка входа.

### 0.A Создание нового файла (с нуля)

1. **Читаю исходный материал модуля** (PDF/HTML из Microsoft Learn). Составляю карту
   подразделов, определений, ограничений, Note/Important, сценариев — это основа покрытия.
2. **Готовлю вопросы** как JSON-массив (формат — раздел 4), отдельно для Quiz и отдельно
   для Practice, по правилам разделов 1–3.
3. **Запускаю программный аудит** (раздел 5) — сам, в коде, а не «на глаз».
   Чиню всё, что он находит, и гоняю повторно до чистого результата.
4. **Собираю HTML**: беру движок из раздела 7, подставляю только разрешённые поля
   (раздел 6), вставляю JSON вопросов.
5. **Запускаю технические проверки** (раздел 8): JS-синтаксис, валидность JSON,
   функциональный прогон через jsdom (особенно для multi-select).
6. **Сверяю с требованиями** (финальный чек-лист, раздел 9). Только потом отдаю файл.
7. Об ошибках и ограничениях говорю честно: что проверено программно, а что — нет.

### 0.B Проверка/исправление уже существующего файла

1. **Сверяю движок файла с эталоном раздела 7** — побайтово или по sha256 (движок
   вставляется дословно без изменений, поэтому любое расхождение уже само по себе
   нарушение раздела 6). Если движок устаревшей версии (см. «Историю версий» в
   разделе 7) или отредактирован вручную — пересобираю HTML на текущем эталонном
   движке, сохранив вопросы и тексты модуля (title/h1/questions/finalText) без изменений.
2. **Извлекаю массив `questions`** из файла и прогоняю программный аудит (раздел 5).
   Чиню всё, что он находит, и гоняю повторно до чистого результата.
3. **Запускаю технические проверки** (раздел 8) на итоговом файле.
4. **Сверяю с финальным чек-листом** (раздел 9), включая содержательные пункты
   (полнота покрытия, дубли, правдоподобность дистракторов — раздел 3.4), которые
   аудит не ловит.
5. Об ошибках, найденных нарушениях и о том, что проверено программно, а что —
   только содержательно, говорю честно (раздел 11).

---

## 1. Quiz — назначение и правила

Проверяет понимание материала текущего раздела Microsoft Learn: определения,
ограничения, архитектуру, различия сервисов, ключевые возможности,
поддерживаемые/неподдерживаемые сценарии. **Не** Practice.

- **Источник** — только материал текущего раздела. Не привлекать знания из других
  разделов, не выдумывать темы вне источника.
- **Полнота** — перед завершением проверяю покрытие всех подразделов, определений,
  ограничений, архитектурных особенностей, сценариев и важных Note. Пропуск
  подраздела не допускается. Покрытие выравниваю по источнику (не оставляю один
  подраздел с 8 вопросами, а соседний с 3 без причины в материале).
- **Типы вопросов** — Best answer (single choice) и Choose Two/Three (несколько верных).
  Допустимы ситуационные вопросы на понимание. **Не используются** Complete Solution и
  Cmdlet/Parameter Selection — это режимы Practice.
- **Choose Two/Three** — только там, где источник сам перечисляет несколько
  одновременно верных фактов (список особенностей/ограничений/категорий), и проверка
  множественности добавляет понимание. Если для идеи естественен один ответ —
  Single Choice, не натягивать multi ради формата.
- **Запрещено**: вопросы вне материала, дубли вопросов/идей, чистое запоминание (если
  источник на нём не делает акцент), очевидные ответы.

---

## 2. Practice — назначение и правила

Проверяет **применение**, а не запоминание: команды/cmdlet'ы (если реально есть в
материале), выбор инструмента под сценарий, последовательность действий в портале/CLI,
решение в сценарии («что сделать», «чего не делать»), процедура выполнения задачи.

- **Только для модулей с практическим материалом.** Чисто концептуальный модуль
  (например, Fundamentals) Practice-файла не получает — это не пробел, а корректное
  решение. Не натягивать применение там, где источник его не даёт.
- **Источник** — строго материал текущего модуля. Не подмешивать сценарии из других
  сервисов/модулей.
- **Количество вопросов — не фиксированное.** Не ориентироваться на «как в примере».
  Ровно столько, сколько нужно для покрытия практической части (может быть 10, может 25).
- **Если для модуля нет материала под какой-то режим — режим не используется.** Особенно
  про Parameter Selection: нет реальных cmdlet-параметров в источнике → режим пропускается
  целиком. Ничего не выдумывать.
- **Парный lab-checklist (для реально hands-on модулей).** MCQ не проверяет руки, поэтому
  для модулей с портальными/CLI-действиями рядом с Practice-файлом создаётся короткая
  заметка-лаба `<N.N.N> <ModuleName> — Lab.md`: пошаговый список «сделай в портале/CLI»
  (создать ресурс, настроить, проверить, удалить) строго по материалу модуля. Practice
  гоняет решения вокруг действий, Lab — само действие. Если hands-on в источнике нет —
  lab-checklist не создаётся.

**Поддерживаемые режимы:**
- Single Choice
- Choose Two
- Choose Three
- Complete Solution (каждый верный вариант — самостоятельное полное решение задачи)
- Tool Selection (выбор инструмента под сценарий — по реальному списку из материала)
- Cmdlet Selection / Parameter Selection (только если в материале есть реальные cmdlet'ы с параметрами)

Поле `section` для Practice содержит **название режима**
(`Single Choice` / `Choose Two` / `Tool Selection` / `Complete Solution` / ...),
а не тему подраздела.

---

## 2a. Cases и Mock — межмодульные (фаза «перед экзаменом»)

Exam-style подготовка. Два вида, оба **межмодульные**:

- **Cases (кейсы)** — один сценарий-компании + связка под-вопросов через несколько
  доменов (например, identity + networking + storage в одном сценарии).
- **Mock** — смешанный interleaved прогон ~40–50 вопросов по всем доменам, как проверка
  готовности.

Это **единственное место, где снимается правило «источник = один модуль»**: кейс и mock
намеренно охватывают несколько доменов.

**Контроль валидности — обязателен.** Реальные экзаменационные кейсы под NDA, совпадение
не гарантируется; поэтому валидность держится на проверяемых якорях, а не на доверии:

1. **Citation + skill-tag на каждый под-вопрос.** Объект вопроса дополняется полями
   `skill` (домен/задача из official study guide AZ-104) и `source` (URL Microsoft Docs,
   подтверждающий верный ответ). Движок эти поля не рендерит — они для верификации. Нет
   источника → вопрос вырезается.
2. **Проход верификации.** После генерации — отдельный проход: каждый ответ сверяется с
   процитированным `source` на фактическую корректность (можно субагентом). Это сверх
   аудита §5, а не вместо него.
3. **Калибровка по официальному Microsoft Practice Assessment** (бесплатный, exam-style):
   формат, сложность и формулировки подгоняются под него, расхождения правятся. Копировать
   его вопросы нельзя — только калибровать стиль.
4. **Покрытие по весам доменов.** Cases + Mock в сумме отражают веса экзамена: Manage
   identities & governance 20–25%, Implement & manage storage 15–20%, Deploy & manage
   compute 20–25%, Implement & manage virtual networking 15–20%, Monitor & maintain 10–15%.
5. **Честная маркировка.** Явно указывать уверенность и что совпадение с реальным
   экзаменом непроверяемо (NDA) — не выдавать предположения за проверенное (§11).

Формат — тот же движок (§7). Кейс: общий стем в `section: "Case: <сценарий>"`, под-вопросы —
любой тип. Именование — межмодульное, без номера модуля (§10).

---

## 3. Общие правила качества — для ВСЕХ вопросов (Quiz и Practice)

Перед финализацией — **обязателен программный аудит** (раздел 5), а не только визуальный.

**3.0. Полнота покрытия юнита — обязательное требование.** И Quiz, и Practice обязаны
покрыть **все** подразделы (units), темы, определения, ограничения и важные Note модуля —
каждый в своей плоскости: Quiz покрывает все концептуальные подразделы, Practice — все
подразделы, где есть практический материал (портал/CLI/cmdlet/сценарий). Пропуск подраздела
без причины в самом источнике не допускается ни в одном из файлов. Перед финализацией строю
карту подразделов источника и сверяю, что каждый отражён хотя бы одним вопросом
соответствующего файла; подраздел без практической части в Practice пропускается только если
в источнике этой части действительно нет (и это проговаривается). Проверяется содержательно
по карте покрытия (§9), аудит-скрипт полноту не ловит.

**3.1. Баланс длины и детализации.** Правильный ответ **не должен по длине превышать ни
один неверный вариант** этого вопроса (в словах) — жёсткое правило без исключений «по
смыслу», а не «в среднем короче». Нарушение засчитывается, даже если правильный длиннее
лишь одного неверного варианта, даже на одно слово. Само по себе «правильный ответ —
самый длинный/самый подробный вариант» — классический lexical tell, которым отвечающий
пользуется, не читая вопрос по существу; это ровно то, что правило должно исключать.
Практически: при написании варианта сначала формулирую правильный ответ максимально
кратко (без лишних уточнений/оговорок «в большинстве случаев», «как правило» и т.п.,
если источник не требует такой точности), и только потом — неверные, так, чтобы ни один
из них не оказался короче правильного. Если после аудита (раздел 5) правильный ответ
всё равно самый длинный — переформулирую его короче или удлиняю дистракторы, а не
оставляю как «естественную» длину. Дополнительно: разница между самым длинным и самым
коротким вариантом ≤ 4 слов (не 4–5). Исключение из обоих пунктов — короткие
категориальные варианты (названия инструментов, cmdlet'ы, proper-noun, ≤4 слов
максимум): там длина естественно разная и подсказкой не является.

**3.1a. Явно указывать количество ответов в тексте вопроса.** Текст `question` для
каждого multi-select вопроса обязательно заканчивается пометкой «(Choose Two)» /
«(Choose Three)» (в общем случае — «(Choose <N словом с заглавной буквы>)», где N ==
`selectCount`). Формат ровно такой: пробел, открывающая скобка, «Choose », числительное
словом с заглавной буквы, закрывающая скобка, в самом конце строки, без точки после
скобки. Например: `"question": "Which capabilities help you stay up-to-date on your data
landscape with Purview? (Choose Three)"`. Это единственный индикатор количества ответов
в интерфейсе — отдельной строки-подсказки под вопросом движок больше не показывает:
элемент `#hint` остаётся в разметке, но скрипт всегда пишет в него пустую строку.
Проверяется программно — аудит (раздел 5) считает нарушением multi-select вопрос, где
этой пометки нет или где число в ней не совпадает с `selectCount`.

**3.2. Нет лексических «tell»-слов.** Слово, встречающееся **только** в верных или
**только** в неверных ответах по всему файлу — уязвимость (кандидаты: `automatically`,
`always`, `only`, `never`, `requires`, `must`, `cannot`). Проверяю частоту correct vs
wrong по всему массиву. Цель — нет однозначной корреляции.

**3.2a. Отдельно и жёстче — слово `only`.** Это самый эксплуатируемый tell из всех:
формулировки вида «...only...» / «the only service that...» почти всегда выдают
правильный вариант с первого взгляда, без чтения объяснения, потому что естественный
язык вопросов чаще ставит `only` именно туда, где утверждение верно. Поэтому для `only`
порог намного строже, чем «≥3 раза с одной стороны» из 3.2:
- Одного-единственного вхождения `only` (в любой форме — `only`, `the only`, `only if`,
  `only when`) в вариантах ответа одного вопроса, где оно встречается **исключительно**
  в правильном варианте, уже достаточно, чтобы засчитать нарушение — не нужно ждать
  накопления по всему файлу.
- То же для случая, когда `only` встречается исключительно среди неверных вариантов
  этого вопроса (тоже tell, только в обратную сторону — «раз тут only, значит это
  дистрактор»).
- Практическое правило по умолчанию: **избегать `only` в вариантах ответа вообще**, если
  без него формулировка не теряет смысла (переформулировать через `exclusively used for`,
  `works only when...` → лучше вообще снять модальность и описать факт без `only`, или,
  если исключительность — часть сути ответа, вставить `only` **и в правильный, и хотя бы
  в один неверный** вариант, чтобы слово перестало быть подсказкой).
- Это правило проверяется программно per-question (см. аудит, раздел 5), а не только
  агрегированно по всему файлу, как для остальных tell-слов.

**3.3. Нет буквальных Yes/No-префиксов.** В сценарных «возможно ли это?» не делать
верный ответ начинающимся с "Yes," а неверные с "No," (или наоборот). Варианты —
описательные утверждения без префикса.

**3.4. Правдоподобность дистракторов.** Неверные варианты — на реальных смежных
Azure-концептах/сервисах/инструментах/командах, не на выдуманных механизмах. Хороший
дистрактор — реальная, но неверная в этом контексте вещь (другой реальный сервис, другой
реальный флаг, другой реальный инструмент той же категории). **Это проверяю я сам,
содержательно — аудит-скрипт его не ловит.**

**3.5. Уникальность идеи.** Каждый вопрос — одна уникальная идея. Не повторять одну идею
в разных формулировках. Если факт уже проверен в Quiz (понимание), в Practice
брать другой угол — практическое действие/решение, а не то же определение. Для Choose
Two/Three: один и тот же список фактов с тем же набором вариантов и в Quiz, и в Practice
Assessment — это дубль. Quiz проверяет множественность как понимание, Practice —
как применение (выбор действий/инструментов под сценарий). Если различить углы не выходит —
вопрос оставляю только в одном файле.

---

## 4. Формат вопроса (JSON)

Массив объектов `questions`. Один объект:

```json
{
  "section": "Quiz: тема подраздела | Practice: название режима",
  "question": "Текст. Может содержать <code>...</code> для команд/URL/параметров.",
  "options": ["вариант A", "вариант B", "вариант C", "вариант D"],
  "answer": 0,
  "explanation": "Почему этот ответ верный."
}
```

Multi-select (Choose Two/Three) — `answer` массивом + обязательный `selectCount`:

```json
{
  "section": "...",
  "question": "Which ...?",
  "options": ["...","...","...","...","..."],
  "answer": [0,1,2],
  "selectCount": 3,
  "explanation": "..."
}
```

Добавлять «(Choose Three)» (или соответствующее число) в конец `question` — это
требование 3.1a: это единственное место, где число ответов видно пользователю, отдельной
строки-подсказки под вопросом движок не показывает.

Правила: ≥3 опции; `answer` int (single) или list[int] (multi); для multi
`selectCount == len(answer)` и ≥2 верных; индексы в диапазоне; строки непустые.

---

## 5. Программный аудит (запускаю сам перед сборкой)

Гоняю на готовом массиве вопросов. Привожу к чистому результату, потом собираю HTML.
Эталонная реализация проверок (Python):

```python
import re, json
from collections import Counter

def words(s): return re.findall(r"[A-Za-z0-9']+", str(s).lower())

def audit(qs):
    problems = []
    # --- структура ---
    for i,q in enumerate(qs,1):
        for k in ('section','question','options','answer','explanation'):
            if k not in q: problems.append(f"Q{i}: missing '{k}'")
        opts=q.get('options',[])
        if len(opts)<3: problems.append(f"Q{i}: <3 options")
        ans=q.get('answer')
        if isinstance(ans,list):
            if q.get('selectCount')!=len(ans): problems.append(f"Q{i}: selectCount != len(answer)")
            if len(ans)<2: problems.append(f"Q{i}: multi needs >=2 correct")
            if len(set(ans))!=len(ans): problems.append(f"Q{i}: dup answer indices")
            for a in ans:
                if not isinstance(a,int) or a<0 or a>=len(opts): problems.append(f"Q{i}: idx {a} OOR")
        elif isinstance(ans,int):
            if ans<0 or ans>=len(opts): problems.append(f"Q{i}: idx OOR")
        else: problems.append(f"Q{i}: answer must be int or list[int]")
    # --- 3.1 длина: правильный ответ никогда не длиннее любого неверного ---
    for i,q in enumerate(qs,1):
        opts=q['options']
        lens=[len(words(o)) for o in opts]
        if max(lens)<=4: continue          # только совсем короткие категориальные — исключение
        ans=q['answer'] if isinstance(q['answer'],list) else [q['answer']]
        correct_lens=[lens[a] for a in ans]
        wrong_lens=[l for idx,l in enumerate(lens) if idx not in ans]
        if wrong_lens and max(correct_lens) > max(wrong_lens):
            problems.append(f"3.1 Q{i}: correct answer ({max(correct_lens)}w) longer than every wrong option (max {max(wrong_lens)}w) — reformulate shorter or lengthen distractors, not leave as-is")
        spread=max(lens)-min(lens)
        if spread>=4: problems.append(f"3.1 Q{i}: length spread {spread}w (limit <4)")
    # --- 3.1a Choose N обязателен в конце question для multi-select, число должно совпадать с selectCount ---
    _num_name = {2:'Two', 3:'Three', 4:'Four'}
    for i,q in enumerate(qs,1):
        if isinstance(q['answer'],list):
            sc = q.get('selectCount')
            expected = f"(Choose {_num_name.get(sc, sc)})"
            if not q['question'].rstrip().endswith(expected):
                problems.append(f"3.1a Q{i}: question text must end with '{expected}' (selectCount={sc}) — missing or number mismatch")
    # --- 3.2 lexical tells ---
    cc,wc=Counter(),Counter()
    for q in qs:
        ans=q['answer'] if isinstance(q['answer'],list) else [q['answer']]
        for idx,o in enumerate(q['options']):
            tgt=cc if idx in ans else wc
            for t in set(words(o)): tgt[t]+=1
    for w in ['automatically','always','never','requires','all','every','must','cannot','can']:
        if cc[w]>=3 and wc[w]==0: problems.append(f"3.2 tell '{w}': CORRECT-only ({cc[w]}x)")
        if wc[w]>=3 and cc[w]==0: problems.append(f"3.2 tell '{w}': WRONG-only ({wc[w]}x)")
    # --- 3.2a 'only' — намного строже, проверка per-question, а не по всему файлу ---
    only_re = re.compile(r'\bonly\b', re.I)
    for i,q in enumerate(qs,1):
        ans=q['answer'] if isinstance(q['answer'],list) else [q['answer']]
        opts=q['options']
        correct_has = any(only_re.search(opts[a]) for a in ans)
        wrong_has = any(only_re.search(o) for idx,o in enumerate(opts) if idx not in ans)
        if correct_has and not wrong_has:
            problems.append(f"3.2a Q{i}: 'only' appears ONLY in the correct option — dead giveaway, rewrite or add 'only' to a distractor too")
        if wrong_has and not correct_has:
            problems.append(f"3.2a Q{i}: 'only' appears ONLY in wrong option(s) — tell in reverse, rewrite or balance")
    # --- 3.3 Yes/No ---
    for i,q in enumerate(qs,1):
        for j,o in enumerate(q['options']):
            if re.match(r'^\s*(yes|no)\s*,',o.strip(),re.I): problems.append(f"3.3 Q{i} opt{j}: Yes/No prefix")
    # --- 3.5 дубли ---
    qt=[q['question'].lower() for q in qs]
    for i in range(len(qt)):
        for j in range(i+1,len(qt)):
            a,b=set(words(qt[i])),set(words(qt[j]))
            if a and b and len(a&b)/len(a|b)>0.7: problems.append(f"3.5 Q{i+1}/Q{j+1}: near-dup question")
    seen=Counter()
    for q in qs:
        ans=q['answer'] if isinstance(q['answer'],list) else [q['answer']]
        seen[' | '.join(sorted(q['options'][a].lower() for a in ans))]+=1
    for k,c in seen.items():
        if c>1: problems.append(f"3.5 duplicate correct-answer set x{c}")
    return problems

# использование:
qs=json.load(open('questions.json',encoding='utf-8'))
p=audit(qs)
print("CLEAN" if not p else "\n".join(p))
```

Аудит — помощник, а не гарантия. Он НЕ проверяет 3.4 (правдоподобность дистракторов)
и полноту покрытия источника (раздел 1) — это делаю я содержательно.

---

## 6. Что разрешено менять в движке

Движок (раздел 7) HTML/CSS/JS — **неизменен**. Подставляю только:

| Плейсхолдер | Куда | Источник значения |
|---|---|---|
| `__TITLE__` | `<title>` | тема модуля |
| `__H1__` | заголовок в шапке | тема модуля |
| `__QUESTIONS_JSON__` | литерал массива `questions` | мой JSON (через `json.dumps(qs, ensure_ascii=False)`) |
| `__FINAL_HIGH__` | итог при ≥85% | сообщение под тему модуля |
| `__FINAL_MID__` | итог при 70–84% | сообщение под тему модуля |
| `__FINAL_LOW__` | итог при <70% | сообщение под тему модуля |

Строки finalText вставляются внутрь одинарных кавычек JS — экранировать `\` и `'`.
После подстановки в файле НЕ должно остаться `__...__` плейсхолдеров.

Сборка (Python):

```python
import json
tpl = open('engine_template.html', encoding='utf-8').read()   # движок из раздела 7
def esc(s): return s.replace('\\','\\\\').replace("'","\\'")
html = (tpl
    .replace('__TITLE__', title)
    .replace('__H1__', h1)
    .replace('__QUESTIONS_JSON__', json.dumps(qs, ensure_ascii=False))
    .replace('__FINAL_HIGH__', esc(final_high))
    .replace('__FINAL_MID__',  esc(final_mid))
    .replace('__FINAL_LOW__',  esc(final_low)))
assert '__' not in html or not __import__('re').search(r'__[A-Z0-9_]+__', html)
open(out_path,'w',encoding='utf-8').write(html)
```

---

## 7. Эталонный движок (встроен — копировать дословно)

> Поведение (уже реализовано): варианты перемешиваются при каждом показе; если `answer` —
> массив, рендер checkbox с автоблокировкой лишних после `selectCount`, засчёт только при
> точном совпадении набора; если `answer` — int, рендер radio; `<code>` в question/options
> рендерится через innerHTML и стилизован в палитре; live-счётчик показывает
> `score / questions.length` (знаменатель — общее число вопросов); subtitle пустой; UI на
> английском; экран результатов с review и Restart.
>
> **Журнал ошибок (Export mistakes to journal).** На экране результатов — две дополнительные
> кнопки рядом с Restart: `exportMistakesBtn` («Export mistakes to journal») и
> `createJournalBtn` («No journal yet — create it»). По клику `exportMistakesToJournal()`
> собирает все вопросы, где `!userAnswers[i].ok` (неотвеченные или неверные), и:
> - если браузер поддерживает File System Access API (Chrome/Edge) — дозаписывает их в
>   существующий JSON-файл журнала (`showOpenFilePicker`) или создаёт новый
>   (`showSaveFilePicker`, предлагаемое имя `az104-mistakes-journal.json`);
> - иначе — скачивает отдельный файл `mistakes_<title>_<date>.json` с алертом-инструкцией
>   на русском («Браузер не поддерживает прямую запись в файл... импортируйте его вручную»).
>
> Каждая запись журнала — карточка интервального повторения (Leitner-подобная схема):
> `id` (детерминированный хеш `quizTitle|section|question` через `hashStr`), `quizTitle`,
> `domain`, `section`, `question`, `options`, `answer`, `selectCount`, `explanation`, `box`,
> `masteredStreak`, `nextReview`, `firstMissed`. Если вопрос уже есть в журнале и снова
> провален — `box`/`masteredStreak` сбрасываются в исходное состояние, `nextReview` = сегодня;
> иначе добавляется новая запись. Эти кнопки, функции (`hashStr`, `todayStrLocal`,
> `ensureRWPermission`, `downloadFallbackExport`, `exportMistakesToJournal`) и вся остальная
> JS/HTML/CSS-структура — часть неизменяемого движка (раздел 6), как и всё остальное.
>
> Integrity: sha256 движка-шаблона = `796ea400cd07877269d6587106bc58f7f670d285452474ddc7c769694859d4a2`.

> **История версий:** версия движка с журналом ошибок, но со строкой-подсказкой `Select N.`
> под вопросом имела sha256 `1c05a6c0f6ac72d4b4fb093cb91e65b9be80c9ade304ed07c2c05ec8536c42b1` —
> считается устаревшей и заменена текущей (строка-подсказка убрана, число ответов
> указывается только в тексте `question` через «(Choose N)», см. 3.1a; обновление
> зафиксировано 2026-07-11). Ещё более ранняя версия (без журнала ошибок) имела sha256
> `8d56420f5b3584a1ff040f08d6a467fb95fcbf61a1441cfdda5ba86843b0e93e`. Quiz-файлы, собранные
> на устаревших версиях, при следующей проверке приводятся к текущему движку — см. раздел 0.B.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8"><meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>__TITLE__</title>
<style>
:root{--bg:#0b1f2a;--panel:#102b3a;--text:#eef7fb;--muted:#a9c0cc;--accent:#7cc7ff;--accent2:#9ed8ff;--correct:#2ecc71;--wrong:#ff6b6b;--border:#315365}
*{box-sizing:border-box}body{margin:0;font-family:Arial,Helvetica,sans-serif;background:linear-gradient(135deg,#081922,#102b3a);color:var(--text);min-height:100vh;display:flex;justify-content:center;align-items:flex-start;padding:28px}.app{width:min(1050px,100%);background:rgba(16,43,58,.98);border:1px solid var(--border);border-radius:18px;box-shadow:0 20px 60px rgba(0,0,0,.35);overflow:hidden}header{padding:28px 32px 18px;border-bottom:1px solid var(--border);background:rgba(8,25,34,.6)}h1{margin:0 0 10px;font-size:26px}.subtitle{color:var(--muted);font-size:15px;line-height:1.5}.progress-wrap{padding:22px 32px 0}.progress-label{display:flex;justify-content:space-between;color:var(--muted);font-size:14px;margin-bottom:10px}.bar{width:100%;height:8px;background:rgba(255,255,255,.16);border-radius:999px;overflow:hidden}.fill{height:100%;width:0;background:var(--accent);transition:width .25s ease}main{padding:32px}.section{display:inline-block;color:var(--accent2);background:rgba(124,199,255,.12);border:1px solid rgba(124,199,255,.25);padding:8px 12px;border-radius:999px;font-size:14px;margin-bottom:20px}.question{font-size:24px;line-height:1.45;margin:0 0 26px}.question code,.option code{background:rgba(124,199,255,.12);border:1px solid rgba(124,199,255,.25);color:var(--accent2);padding:1px 6px;border-radius:6px;font-family:Consolas,Monaco,monospace;font-size:.92em}.options{display:grid;gap:12px;margin-bottom:24px}.option{display:flex;gap:14px;align-items:flex-start;padding:16px 18px;border:1px solid var(--border);border-radius:14px;background:rgba(8,25,34,.45);cursor:pointer;transition:border-color .15s ease,background .15s ease}.option:hover{border-color:var(--accent);background:rgba(124,199,255,.08)}.option input{margin-top:3px;transform:scale(1.2);accent-color:var(--accent)}.option.correct{border-color:var(--correct);background:rgba(46,204,113,.12)}.option.wrong{border-color:var(--wrong);background:rgba(255,107,107,.12)}.letter{color:var(--accent2);font-weight:bold;width:22px;flex:0 0 auto}.hint{color:var(--muted);font-size:14px;margin:-8px 0 16px}.controls{display:flex;gap:12px;flex-wrap:wrap;align-items:center;margin-top:20px}button{border:0;border-radius:12px;padding:13px 18px;font-size:16px;font-weight:700;cursor:pointer;background:var(--accent);color:#062131}button.secondary{background:rgba(255,255,255,.12);color:var(--text);border:1px solid var(--border)}button:disabled{opacity:.45;cursor:not-allowed}.feedback{display:none;margin-top:22px;padding:18px;border-radius:14px;border:1px solid var(--border);background:rgba(8,25,34,.55);line-height:1.55}.feedback.show{display:block}.status{font-weight:800;margin-bottom:8px}.ok{color:var(--correct)}.bad{color:var(--wrong)}.results{display:none;padding:32px;line-height:1.55}.results.show{display:block}.score{font-size:42px;font-weight:800;margin:12px 0;color:var(--accent2)}.review{margin-top:24px;display:grid;gap:12px}.review-item{border:1px solid var(--border);border-radius:14px;padding:14px 16px;background:rgba(8,25,34,.45)}.review-item strong{color:var(--accent2)}.hidden{display:none}@media(max-width:640px){body{padding:12px}header,main,.progress-wrap,.results{padding-left:18px;padding-right:18px}.question{font-size:20px}h1{font-size:22px}}
</style></head>
<body><div class="app"><header><h1>__H1__</h1><div class="subtitle"></div></header>
<div id="quizArea"><div class="progress-wrap"><div class="progress-label"><span id="counter"></span><span id="scoreLive"></span></div><div class="bar"><div class="fill" id="progressFill"></div></div></div>
<main><div class="section" id="section"></div><p class="question" id="question"></p><div class="hint" id="hint"></div><div class="options" id="options"></div><div class="controls"><button id="checkBtn">Check answer</button><button id="nextBtn" class="secondary" disabled>Next</button><button id="restartBtn" class="secondary">Restart</button></div><div class="feedback" id="feedback"><div class="status" id="status"></div><div id="explanation"></div></div></main></div>
<div class="results" id="results"><h2>Quiz completed</h2><div class="score" id="finalScore"></div><p id="finalText"></p><div class="controls"><button id="restartBtn2">Restart quiz</button><button id="exportMistakesBtn" class="secondary">Export mistakes to journal</button><button id="createJournalBtn" class="secondary">No journal yet — create it</button></div><div class="review" id="review"></div></div></div>
<script>
const questions = __QUESTIONS_JSON__;
let current=0, score=0, checked=false, userAnswers=Array(questions.length).fill(null);
let shuffledOptions = [];
let correctSet = [];
const $=id=>document.getElementById(id);

function isMulti(q){return Array.isArray(q.answer);}

function shuffleArray(array){
  const result = [...array];
  for(let i=result.length-1;i>0;i--){
    const j = Math.floor(Math.random()*(i+1));
    [result[i], result[j]] = [result[j], result[i]];
  }
  return result;
}

function updateLiveScore(){
  const percent = current === 0 && score === 0 ? 0 : Math.round((score / questions.length) * 100);
  $('scoreLive').textContent = `Score: ${score} / ${questions.length} (${percent}%)`;
}

function renderQuestion(){
  const q=questions[current];
  checked=false;
  const answerArr = isMulti(q) ? q.answer : [q.answer];
  shuffledOptions = shuffleArray(q.options.map((text, originalIndex)=>({text, originalIndex})));
  correctSet = shuffledOptions.map((o,idx)=>answerArr.includes(o.originalIndex)?idx:-1).filter(x=>x>=0);

  $('counter').textContent=`Question ${current+1} of ${questions.length}`;
  updateLiveScore();
  $('progressFill').style.width=`${(current/questions.length)*100}%`;
  $('section').textContent=q.section;
  $('question').innerHTML=q.question;
  $('hint').textContent = '';
  $('feedback').classList.remove('show');
  $('status').textContent='';
  $('explanation').textContent='';
  $('nextBtn').disabled=true;
  $('checkBtn').disabled=false;
  $('options').innerHTML='';

  const inputType = isMulti(q) ? 'checkbox' : 'radio';
  shuffledOptions.forEach((option,index)=>{
    const label=document.createElement('label');
    label.className='option';
    label.innerHTML=`<input type="${inputType}" name="answer" value="${index}"><span class="letter">${String.fromCharCode(65+index)}.</span><span>${option.text}</span>`;
    $('options').appendChild(label);
  });

}

function enforceLimit(){
  const q=questions[current];
  if(!isMulti(q))return;
  const boxes=[...document.querySelectorAll('input[name="answer"]')];
  const checkedCount=boxes.filter(b=>b.checked).length;
  boxes.forEach(b=>{ b.disabled = (!b.checked && checkedCount>=q.selectCount); });
}

function selectedIndices(){
  return [...document.querySelectorAll('input[name="answer"]:checked')].map(b=>Number(b.value));
}

function checkAnswer(){
  if(checked)return;
  const q=questions[current];
  const sel=selectedIndices();
  const need = isMulti(q) ? q.selectCount : 1;
  if(sel.length!==need){
    $('feedback').classList.add('show');
    $('status').textContent = isMulti(q) ? `Please select ${q.selectCount} option(s).` : 'Please select an answer.';
    $('status').className='status';
    $('explanation').textContent='';
    return;
  }
  checked=true;
  const correctSorted=[...correctSet].sort((a,b)=>a-b).join(',');
  const selSorted=[...sel].sort((a,b)=>a-b).join(',');
  const ok = correctSorted===selSorted;
  if(ok)score++;

  userAnswers[current]={
    selectedText: sel.map(i=>shuffledOptions[i].text).join(' | '),
    correctText: correctSet.map(i=>shuffledOptions[i].text).join(' | '),
    ok: ok
  };

  document.querySelectorAll('.option').forEach((el,index)=>{
    el.querySelector('input').disabled=true;
    if(correctSet.includes(index))el.classList.add('correct');
    if(sel.includes(index)&&!correctSet.includes(index))el.classList.add('wrong');
  });

  $('feedback').classList.add('show');
  $('status').textContent=ok?'Correct':'Incorrect';
  $('status').className='status '+(ok?'ok':'bad');
  $('explanation').textContent=q.explanation;
  updateLiveScore();
  $('nextBtn').disabled=false;
  $('checkBtn').disabled=true;
}

function nextQuestion(){if(current<questions.length-1){current++;renderQuestion();}else showResults();}

function showResults(){
  $('progressFill').style.width='100%';
  $('quizArea').classList.add('hidden');
  $('results').classList.add('show');

  const percent=Math.round(score/questions.length*100);
  $('finalScore').textContent=`${score} / ${questions.length} (${percent}%)`;
  $('finalText').textContent=percent>=85
    ?'__FINAL_HIGH__'
    :percent>=70
      ?'__FINAL_MID__'
      :'__FINAL_LOW__';

  $('review').innerHTML='';
  questions.forEach((q,i)=>{
    const ua=userAnswers[i];
    const ok=ua!==null && ua.ok;
    const correctText = isMulti(q) ? q.answer.map(a=>q.options[a]).join(' | ') : q.options[q.answer];
    const item=document.createElement('div');
    item.className='review-item';
    item.innerHTML=`<strong>${i+1}. ${q.section}</strong><br>${q.question}<br><span style="color:${ok?'var(--correct)':'var(--wrong)'}">${ua===null?'Not answered':(ok?'Correct':'Incorrect')}</span><br>Your answer: ${ua===null?'No answer':ua.selectedText}<br>Correct answer: ${ua===null?correctText:ua.correctText}<br><em>${q.explanation}</em>`;
    $('review').appendChild(item);
  });
}

function hashStr(s){
  let h1=0xdeadbeef, h2=0x41c6ce57;
  for(let i=0;i<s.length;i++){
    const ch=s.charCodeAt(i);
    h1=Math.imul(h1^ch,2654435761); h2=Math.imul(h2^ch,1597334677);
  }
  h1=Math.imul(h1^(h1>>>16),2246822507)^Math.imul(h2^(h2>>>13),3266489909);
  h2=Math.imul(h2^(h2>>>16),2246822507)^Math.imul(h1^(h1>>>13),3266489909);
  return (4294967296*(2097151&h2)+(h1>>>0)).toString(16);
}
function todayStrLocal(){ const d=new Date(); d.setHours(0,0,0,0); return d.toISOString().slice(0,10); }
async function ensureRWPermission(handle){
  const opts={mode:'readwrite'};
  if((await handle.queryPermission(opts))==='granted') return true;
  if((await handle.requestPermission(opts))==='granted') return true;
  return false;
}
function downloadFallbackExport(quizTitle, missed){
  const payload={quizTitle, date:todayStrLocal(), mistakes:missed};
  const blob=new Blob([JSON.stringify(payload,null,2)],{type:'application/json'});
  const url=URL.createObjectURL(blob);
  const a=document.createElement('a');
  const safe=quizTitle.replace(/[^\w\-]+/g,'_').slice(0,60);
  a.href=url; a.download='mistakes_'+safe+'_'+todayStrLocal()+'.json';
  document.body.appendChild(a); a.click(); a.remove(); URL.revokeObjectURL(url);
}
async function exportMistakesToJournal(forceCreate){
  const quizTitle = document.title || 'AZ-104 Quiz';
  const missed = [];
  questions.forEach((q,i)=>{
    const ua = userAnswers[i];
    if(!ua || !ua.ok){
      missed.push({section:q.section, question:q.question, options:q.options, answer:q.answer, selectCount:q.selectCount||null, explanation:q.explanation});
    }
  });
  if(missed.length===0){ alert('Ошибок нет — нечего экспортировать.'); return; }
  if(!('showOpenFilePicker' in window)){
    alert('Браузер не поддерживает прямую запись в файл (нужен Chrome/Edge). Скачиваю отдельный файл — импортируйте его вручную в Журнал ошибок.');
    downloadFallbackExport(quizTitle, missed);
    return;
  }
  let handle;
  try{
    if(forceCreate){
      handle = await window.showSaveFilePicker({suggestedName:'az104-mistakes-journal.json', types:[{description:'Журнал ошибок JSON', accept:{'application/json':['.json']}}]});
    } else {
      [handle] = await window.showOpenFilePicker({types:[{description:'Журнал ошибок JSON', accept:{'application/json':['.json']}}]});
    }
  }catch(e){ if(e.name!=='AbortError') console.error(e); return; }

  let journal;
  try{
    const perm = await ensureRWPermission(handle);
    if(!perm){ alert('Нет разрешения на запись в файл.'); return; }
    const file = await handle.getFile();
    const text = await file.text();
    journal = text.trim() ? JSON.parse(text) : {mistakes:[]};
    if(!journal.mistakes || !Array.isArray(journal.mistakes)) journal.mistakes=[];
  }catch(e){ alert('Не удалось прочитать файл журнала: '+e.message); return; }

  let added=0, reset=0;
  missed.forEach(q=>{
    const id = hashStr(quizTitle+'|'+q.section+'|'+q.question);
    const existing = journal.mistakes.find(m=>m.id===id);
    if(existing){ existing.box=1; existing.masteredStreak=0; existing.nextReview=todayStrLocal(); reset++; }
    else{
      journal.mistakes.push({id, quizTitle, domain:null, section:q.section, question:q.question, options:q.options, answer:q.answer, selectCount:q.selectCount, explanation:q.explanation, box:1, masteredStreak:0, nextReview:todayStrLocal(), firstMissed:todayStrLocal()});
      added++;
    }
  });
  try{
    const writable = await handle.createWritable();
    await writable.write(JSON.stringify(journal,null,2));
    await writable.close();
  }catch(e){ alert('Не удалось сохранить журнал: '+e.message); return; }
  alert(`Готово. Новых вопросов: ${added}, повторно ошиблись: ${reset}.`);
}

function restart(){current=0;score=0;checked=false;userAnswers=Array(questions.length).fill(null);$('results').classList.remove('show');$('quizArea').classList.remove('hidden');renderQuestion();}
$('options').addEventListener('change',enforceLimit);$('checkBtn').addEventListener('click',checkAnswer);$('nextBtn').addEventListener('click',nextQuestion);$('restartBtn').addEventListener('click',restart);$('restartBtn2').addEventListener('click',restart);$('exportMistakesBtn').addEventListener('click',()=>exportMistakesToJournal(false));$('createJournalBtn').addEventListener('click',()=>exportMistakesToJournal(true));renderQuestion();
</script></body></html>
```

---

## 8. Технические проверки (запускаю сам после сборки)

- **JS-синтаксис**: извлечь содержимое `<script>` и прогнать `node --check` (или
  `new Function(js)`). Должно пройти без ошибок.
- **JSON массива** `questions` — валиден (парсится `json.loads`).
- **Функциональный прогон через jsdom** хотя бы один раз, особенно для multi-select:
  рендер → выбор верных → Check → статус Correct → Next → … → экран результатов
  (`N / N`) → Restart (сброс на Q1, live-score `0 / N`). Отдельно проверить, что для
  multi-select лимит `selectCount` реально блокирует лишние чекбоксы при последовательных
  кликах, и что неверный набор даёт Incorrect.
- **Журнал ошибок — структурная проверка**: `exportMistakesBtn` и `createJournalBtn`
  присутствуют в результатах и повешены на `exportMistakesToJournal(false/true)`; функции
  `hashStr`, `todayStrLocal`, `ensureRWPermission`, `downloadFallbackExport`,
  `exportMistakesToJournal` присутствуют без изменений. В jsdom File System Access API
  недоступен — клик по `exportMistakesBtn` уходит в fallback-ветку (`downloadFallbackExport`)
  и должен показать alert на русском без исключений; это и есть предел автоматической
  проверки (см. раздел 11) — реальная запись/дозапись в `az104-mistakes-journal.json` через
  `showOpenFilePicker`/`showSaveFilePicker` проверяется только вручную в Chrome/Edge.

Скелет jsdom-теста: рендерить файл с `runScripts:'dangerously'`, читать `questions` из
исходного текста (а не из `window` — переменные движка не глобальные), кликать по
`.option input`, диспатчить `change`, проверять `#status`, `#finalScore`, `#counter`,
`#scoreLive`, `.option.correct`.

---

## 9. Финальный чек-лист перед сдачей

- ☐ Полнота покрытия юнита — обязательно (3.0): и Quiz, и Practice покрывают все подразделы источника в своей плоскости, пропусков нет, карта покрытия сверена
- ☐ Нет вопросов вне материала модуля
- ☐ Нет повторяющихся вопросов/идей (в т.ч. между Quiz и Practice)
- ☐ Choose Two/Three только там, где источник перечисляет несколько верных фактов
- ☐ Practice: использованы только режимы, подтверждённые материалом
- ☐ Аудит раздела 5 — чистый (длина, lexical tells, Yes/No, дубли)
- ☐ Правильный ответ нигде не длиннее всех неверных вариантов того же вопроса, спред ≤4 слов (3.1)
- ☐ Слово `only` нигде не встречается исключительно в правильном (или исключительно в
  неверных) вариантах одного вопроса — проверено per-question, не только по всему файлу (3.2a)
- ☐ Каждый multi-select `question` заканчивается «(Choose Two)»/«(Choose Three)» с числом, равным `selectCount` (3.1a)
- ☐ Дистракторы правдоподобны и грамотны (проверено содержательно — 3.4)
- ☐ JSON валиден; JS-синтаксис проходит; jsdom-прогон проходит (вкл. multi-select)
- ☐ В HTML не осталось `__...__` плейсхолдеров; subtitle пустой; live-score от общего числа
- ☐ Кнопки журнала ошибок (`exportMistakesBtn`, `createJournalBtn`) на месте и повешены на обработчики (проверено структурно; реальная запись в файл — только вручную, раздел 11)
- ☐ Именование файлов по разделу 10

---

## 10. Именование файлов

Имя файла повторяет номер и название модуля из структуры папок — тот же
`<N.N.N> <ModuleName>`, что у заметки модуля. Без префикса `NNN_` и без `AZ104`.
Тип указывается суффиксом через « — »:

- Quiz: `<N.N.N> <ModuleName> — Quiz.html`
- Practice: `<N.N.N> <ModuleName> — Practice.html`

Пример для модуля `0.1.1 Review Azure PowerShell module`:

- `0.1.1 Review Azure PowerShell module — Quiz.html`
- `0.1.1 Review Azure PowerShell module — Practice.html`

Файл кладётся в папку своего модуля рядом с заметкой. Межмодульные артефакты
(кейсы, сводный прогон) номер модуля не получают — имя по охвату, например
`Cases — Identity & Governance.html`, `Mock Exam — AZ-104.html`.

**Журнал ошибок и повторение — глобально, без вложенности.** Единый журнал
`az104-mistakes-journal.json` и приложение повторения `az104-mistake-log.html` лежат в
`MS_Learn/Review/` — **один журнал на всю сертификацию**, не в папках модулей (все quiz
дописывают в один и тот же файл). Имя файла журнала **фиксировано** — движок §7 предлагает
его в `showSaveFilePicker`, менять нельзя. Доступ идёт через файловый пикер, поэтому путь
важен только как «где искать». Рабочий цикл: прошёл quiz → **Export mistakes** → выбрать
`MS_Learn/Review/az104-mistakes-journal.json`; для повторения → открыть
`az104-mistake-log.html` → открыть тот же файл (Chrome/Edge, т.к. нужен File System
Access API).

---

## 11. Честные границы автоматики (помнить и проговаривать)

Что гарантируется механически: целостность движка (подстановка, не переписывание),
структурная валидность вопросов, отсутствие лексических/длинных подсказок и дублей,
синтаксис JS, базовый функциональный прогон.

Что требует меня содержательно и НЕ ловится скриптом: правдоподобность дистракторов (3.4),
полнота и верность покрытия источника, фактическая корректность каждого ответа и
объяснения, соответствие вопроса именно текущему модулю. Если в чём-то не уверен —
говорю об этом прямо, а не выдаю за проверенное.

Отдельно про журнал ошибок: jsdom не реализует File System Access API, поэтому программно
проверяется только структура (кнопки на месте, обработчики повешены, fallback-ветка не
падает и показывает корректный alert) — а не реальная запись/дозапись в
`az104-mistakes-journal.json` в браузере. Это последнее можно подтвердить только вручную,
открыв файл в Chrome или Edge.
