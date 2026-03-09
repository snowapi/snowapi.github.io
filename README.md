# МАС «Снежинка» API v2.0

**Базовая URL (шлюз):** `https://christofari-neo.snowapi.ru/`  
**Публичная документация:** `https://snowapi.github.io/`

API предоставляет программный доступ к гибридному вычислительному комплексу проекта МАС «Снежинка» на платформе VK Play. Все запросы аутентифицируются через API-ключ, выдаваемый в личном кабинете разработчика VK Play.

---

## 1. Аутентификация

Ключ передаётся в HTTP-заголовке:
```

Authorization: Bearer <ваш_ключ>

```
Получение ключа: [VK Play Developer Console](https://vkplay.ru/dev).

---

## 2. Архитектура и маршрутизация

Все запросы первично обрабатываются суперкомпьютером **Christofari Neo** (Сбер), который выполняет:
- классическую предобработку (генеративные модели, молекулярная динамика);
- кэширование результатов;
- оркестрацию гибридных задач;
- автоматическое перенаправление квантовых запросов в инфраструктуру **Госкорпорации «Росатом»** (ионные ловушки, квантовый отжиг, нейтральные атомы);
- запуск автономных исследовательских агентов (autoresearch) на выделенных GPU.

Пользователь взаимодействует только с единым эндпоинтом, не задумываясь о внутреннем распределении.

---

## 3. Эндпоинты

### 3.1 Биоинформатика

**POST** `/bio/analyze`

Анализ биологических последовательностей (ДНК/РНК/белок). Выполняется на классических узлах Christofari Neo.

**Параметры (JSON):**
```json
{
  "sequence": "ATCGATCG...",
  "type": "dna | rna | protein",
  "tasks": ["gc_content", "molecular_weight", "secondary_structure"]
}
```

Ответ:

```json
{
  "gc_content": 0.45,
  "molecular_weight": 12345.6,
  "secondary_structure": "hairpin"
}
```

3.2 Хемоинформатика

POST /chemo/process

Расчёт молекулярных дескрипторов и фингерпринтов по SMILES.

Параметры:

```json
{
  "smiles": "CCO",
  "descriptors": ["logP", "TPSA", "HBD"],
  "fingerprints": ["morgan", "maccs"]
}
```

Ответ:

```json
{
  "descriptors": {
    "logP": 0.37,
    "TPSA": 20.23,
    "HBD": 1
  },
  "fingerprints": {
    "morgan": "101010...",
    "maccs": "110011..."
  }
}
```

3.3 L-системы

POST /lsystem/generate

Генерация фрактальных структур.

Параметры:

```json
{
  "axiom": "A",
  "rules": { "A": "AB", "B": "A" },
  "iterations": 5,
  "angle": 25.7,
  "parameters": { "length": 10, "width": 1.5 }
}
```

Ответ:

```json
{
  "lsystem_string": "ABAABABA...",
  "turtle_commands": ["F", "+", "F", "-", ...],
  "svg": "data:image/svg+xml;base64,..."
}
```

3.4 Шаблоны анимаций

GET /animations/templates

Список доступных шаблонов.

Ответ:

```json
{
  "templates": [
    {"id": "dna_helix", "name": "ДНК-спираль", "format": "gltf"},
    {"id": "protein_fold", "name": "Сворачивание белка", "format": "usd"},
    {"id": "atom_orbitals", "name": "Атомные орбитали", "format": "gltf"}
  ]
}
```

POST /animations/render

Запуск рендера (асинхронно). Результат сохраняется в облачное хранилище, возвращается ссылка.

Параметры:

```json
{
  "template_id": "dna_helix",
  "parameters": {
    "sequence": "ATCG",
    "rotation_speed": 0.5,
    "color_scheme": "rainbow"
  },
  "output_format": "mp4"
}
```

Ответ (задача поставлена):

```json
{
  "job_id": "anim_550e8400",
  "status": "pending"
}
```

GET /animations/result/{job_id} – получение ссылки на готовый файл.

3.5 Фолдинг белков и ДНК

POST /folding/predict

Предсказание трёхмерной структуры. Может использовать классические движки (DeepFold-PLM, AlphaFold) или квантовые ускорители Росатома.

Параметры:

```json
{
  "sequence": "MVLSPADKTNVKAAW...",
  "molecule_type": "protein",
  "engine": "auto | deepfold-plm | alphafold | quantum_annealing | quantum_gate",
  "quantum_provider": "rosatom_annealer | rosatom_ion | rosatom_neutral",
  "options": {
    "return_pdb": true,
    "return_confidence": true,
    "num_reads": 1000,
    "annealing_time": 20,
    "hybrid_jobs": true,
    "classical_instances": "ml.m5.xlarge"
  }
}
```

· Если engine: "auto", Christofari Neo выбирает оптимальный движок на основе длины последовательности и текущей загрузки.
· Квантовые провайдеры доступны только при указании соответствующего engine.

Ответ (асинхронный):

```json
{
  "job_id": "fold_550e8400",
  "status": "pending"
}
```

GET /folding/result/{job_id}

```json
{
  "pdb": "ATOM      1  N   MET A   1     -5.123  2.456  1.789  1.00 15.23           N\n...",
  "confidence": [0.95, 0.87, ...],
  "visualization_url": "https://christofari-neo.snowapi.ru/view/fold_550e8400"
}
```

3.6 Квантовые вычисления (Росатом)

Доступны через единый шлюз Christofari Neo, который автоматически перенаправляет запросы в Национальную квантовую лабораторию.

Провайдеры Росатома:

Провайдер Тип Технология Применение
rosatom_annealer Квантовый аннилер Российский аналог D‑Wave (отжиг) Оптимизация QUBO, отбор молекул
rosatom_ion Вентильный (ионная ловушка) 50+ физических кубитов, точность 99.7% Точный докинг (QAOA), квантовая химия
rosatom_neutral Вентильный (нейтральные атомы) Массивы атомов (опытные образцы) Материаловедение, симуляция решёток
rosatom_sim Симулятор Классический эмулятор Отладка и тестирование (бесплатно)

3.6.1 Оптимизация (QUBO)

POST /quantum/optimize

```json
{
  "provider": "rosatom_annealer",
  "problem_type": "qubo",
  "qubo": {
    "linear": {"0": -1, "1": -2},
    "quadratic": {"0,1": 3}
  },
  "params": {
    "num_reads": 1000,
    "annealing_time": 20
  }
}
```

3.6.2 Молекулярный докинг (QAOA)

POST /quantum/docking

```json
{
  "provider": "rosatom_ion",
  "molecule_smiles": "CCO",
  "target_pdb": "3ERT",
  "options": {
    "hybrid_jobs": true,
    "classical_instances": "ml.m5.xlarge"
  }
}
```

3.6.3 Квантовая химия (VQE)

POST /quantum/chemistry

```json
{
  "provider": "rosatom_ion",
  "system": "ferrocene",
  "method": "vqe",
  "basis_set": "sto-3g"
}
```

Все квантовые эндпоинты возвращают job_id и статус; результат получается через GET /quantum/result/{job_id}.

3.7 Автономный исследователь (autoresearch)

Реализует идею Андрея Карпати: ИИ-агент самостоятельно проводит серии экспериментов по улучшению моделей или поиску новых молекул/белков.

POST /research/launch

Запускает автономного исследователя с заданной программой.

Параметры:

```json
{
  "program_md": "markdown-инструкция для агента (аналогично program.md в проекте autoresearch)",
  "target": "protein_stability | docking_score | binding_energy",
  "budget_hours": 8,
  "priority": "low | normal | high",
  "notify_url": "https://your-server.com/callback"
}
```

· Агент разворачивается на выделенном GPU-кластере Christofari Neo.
· Он самостоятельно модифицирует код (train.py), запускает 5-минутные эксперименты, оценивает метрику (val_bpb или кастомную), принимает/откатывает изменения.
· По окончании возвращается отчёт со списком проведённых экспериментов, лучшими результатами и кодом лучшей модели.

Ответ:

```json
{
  "research_id": "rsrch_123456",
  "status": "running",
  "estimated_completion": "2026-03-10T08:00:00Z"
}
```

GET /research/status/{research_id} – текущий статус, прогресс.

GET /research/report/{research_id} – итоговый отчёт (доступен после завершения).

Пример отчёта:

```json
{
  "research_id": "rsrch_123456",
  "total_experiments": 96,
  "best_metric": -1.234,
  "best_checkpoint": "gs://bucket/models/rsrch_123456_best.pth",
  "experiments": [
    {
      "timestamp": "2026-03-09T22:15:00Z",
      "changes": "increased depth to 12",
      "metric": -1.187,
      "accepted": true
    },
    ...
  ]
}
```

Примечание: автономный режим может запрашивать квантовые ускорители, если это указано в program.md. В этом случае затраты на квантовые ресурсы будут добавлены к стоимости сессии.

---

4. Лимиты и стоимость

4.1 Бесплатный тариф (для ознакомления)

· 100 классических запросов в день (био, хемо, L-системы, анимации).
· 10 квантовых задач в сутки (суммарно на всех провайдерах Росатома).
· Неограниченное использование симулятора rosatom_sim.
· Автономный исследователь недоступен.

4.2 Платные тарифы

Классические HPC (Christofari Neo):

· Выделенный GPU-инстанс (аналог H100): $4.50/час.
· Массовый скрининг (пакетный режим): от $0.10 за задачу.

Квантовые процессоры Росатома:

· Аннилер (rosatom_annealer): $15 за 1000 задач.
· Ионная ловушка (rosatom_ion): $0.30 + $0.025 за shot (минимально 2500 shots).
· Нейтральные атомы (rosatom_neutral): по запросу (цены формируются индивидуально).

Автономный исследователь:

· Фиксированная ставка: $200 за час работы агента (включает использование одного GPU H100).
· Квантовые вызовы внутри сессии тарифицируются отдельно по ценам выше.

Гибридные задачи (Hybrid Jobs):

· Классический инстанс + квантовый доступ: стоимость суммируется.

Все цены указаны в долларах США, списание с баланса VK Play.

---

5. Коды ошибок

Код Описание
400 Неверный запрос (отсутствуют обязательные поля)
401 Недействительный или отсутствующий API-ключ
403 Доступ запрещён (недостаточно прав)
404 Ресурс не найден
429 Превышен лимит запросов
500 Внутренняя ошибка сервера
503 Квантовый ресурс временно недоступен

---

6. Примеры вызовов (cURL)

```bash
# Получить шаблоны анимаций
curl -H "Authorization: Bearer YOUR_API_KEY" \
     https://christofari-neo.snowapi.ru/animations/templates

# Запустить фолдинг с квантовым отжигом
curl -X POST -H "Authorization: Bearer YOUR_API_KEY" \
     -H "Content-Type: application/json" \
     -d '{
         "sequence": "MVLSPADKTNVKAAW...",
         "molecule_type": "protein",
         "engine": "quantum_annealing",
         "quantum_provider": "rosatom_annealer"
     }' \
     https://christofari-neo.snowapi.ru/folding/predict

# Запустить автономного исследователя на 4 часа
curl -X POST -H "Authorization: Bearer YOUR_API_KEY" \
     -H "Content-Type: application/json" \
     -d '{
         "program_md": "# Цель: улучшить стабильность фермента X\n...",
         "target": "protein_stability",
         "budget_hours": 4,
         "priority": "high"
     }' \
     https://christofari-neo.snowapi.ru/research/launch
```

---

7. Поддержка

· Технические вопросы: support@christofari-neo.snowapi.ru
· Биллинг и квоты VK Play: https://vkplay.ru/support
· GitHub (open source): https://github.com/snowapi/snowapi

---

© 2026 МАС «Снежинка» / Госкорпорация «Росатом» / ПАО «Сбер»
Версия документации: 2.0 (09.03.2026)
