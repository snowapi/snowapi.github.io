МАС «Снежинка» API

Базовая URL (шлюз): https://christofari-neo.snowapi.ru/
Все запросы первично направляются на суперкомпьютер Christofari Neo (Сбер), который выполняет маршрутизацию, классическую предобработку и оркестрацию гибридных задач.

Публичный эндпоинт документации: https://snowapi.github.io/

API предоставляет программный доступ к вычислительным модулям проекта МАС «Снежинка» на платформе VK Play. Все запросы требуют аутентификации по API-ключу, выданному платформой.

---

🔑 Аутентификация

API-ключ передаётся в HTTP-заголовке:

```
Authorization: Bearer <ваш_ключ>
```

Ключ можно получить в личном кабинете разработчика на VK Play.

---

🖥️ Архитектура: Christofari Neo как Primary Gateway

Суперкомпьютер Christofari Neo (Сбер) выступает центральным узлом для всех операций:

· Маршрутизация запросов к соответствующим эндпоинтам.
· Классическая предобработка (генеративные модели, молекулярная динамика, работа с базами данных).
· Оркестрация гибридных задач — автоматическое распределение нагрузки между классическими GPU-кластерами и квантовыми провайдерами (D-Wave, IonQ и др.).
· Кэширование результатов для снижения затрат на повторные квантовые вычисления.

Таким образом, пользователь взаимодействует только с Christofari Neo, который самостоятельно управляет всей сложностью нижележащей инфраструктуры.

---

📡 Эндпоинты

1. Биоинформатика

POST /bio/analyze

Анализ биологических последовательностей (нуклеотидных или аминокислотных).
Использует классические HPC-ресурсы Christofari Neo (GPU/CPU).

Параметры запроса (JSON):

```json
{
  "sequence": "ATCGATCG...",
  "type": "dna | rna | protein",
  "tasks": ["gc_content", "molecular_weight", "secondary_structure"]
}
```

Пример ответа:

```json
{
  "gc_content": 0.45,
  "molecular_weight": 12345.6,
  "secondary_structure": "hairpin"
}
```

---

2. Хемоинформатика

POST /chemo/process

Работа с химическими соединениями (дескрипторы, фингерпринты).

```json
{
  "smiles": "CCO",
  "descriptors": ["logP", "TPSA", "HBD"],
  "fingerprints": ["morgan", "maccs"]
}
```

Пример ответа:

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

---

3. Параметрические системы ветвления (L-Systems)

POST /lsystem/generate

Генерация L-систем по заданным правилам.

```json
{
  "axiom": "A",
  "rules": {
    "A": "AB",
    "B": "A"
  },
  "iterations": 5,
  "angle": 25.7,
  "parameters": {
    "length": 10,
    "width": 1.5
  }
}
```

Пример ответа:

```json
{
  "lsystem_string": "ABAABABA...",
  "turtle_commands": ["F", "+", "F", "-", ...],
  "svg": "data:image/svg+xml;base64,..."
}
```

---

4. Стандартизированные шаблоны анимаций

GET /animations/templates

Список доступных шаблонов атомарных и молекулярных анимаций.

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

Запрос рендера анимации с параметрами (асинхронно).
Рендеринг выполняется на GPU-кластере Christofari Neo.

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

Ответ:

```json
{
  "job_id": "anim_550e8400",
  "status": "pending"
}
```

GET /animations/result/{job_id} – получение результата (URL на готовый файл).

---

5. Фолдинг белков и ДНК

POST /folding/predict

Предсказание трёхмерной структуры по последовательности (асинхронно).
Christofari Neo автоматически выбирает движок: классический (DeepFold-PLM, AlphaFold) или квантовый (если указан quantum_provider).

Параметры запроса (JSON):

```json
{
  "sequence": "MVLSPADKTNVKAAW...",
  "molecule_type": "protein | dna",
  "engine": "auto | deepfold-plm | alphafold | quantum_annealing | quantum_gate",
  "quantum_provider": "dwave | ionq | rigetti | rosatom | azure_quantum",
  "options": {
    "return_pdb": true,
    "return_confidence": true,
    "num_reads": 1000,           // для квантового отжига
    "annealing_time": 20,         // для квантового отжига
    "hybrid_jobs": true,          // для вентильных процессоров
    "classical_instances": "ml.m5.xlarge"
  }
}
```

· Если engine: "auto", Christofari Neo принимает решение на основе длины последовательности и текущей загрузки.

Ответ (задача поставлена в очередь):

```json
{
  "job_id": "fold_550e8400",
  "status": "pending"
}
```

GET /folding/result/{job_id} – получение результата.

```json
{
  "pdb": "ATOM      1  N   MET A   1     -5.123  2.456  1.789  1.00 15.23           N\n...",
  "confidence": [0.95, 0.87, ...],
  "visualization_url": "https://snowapi.github.io/view/fold_550e8400"
}
```

---

6. Квантовые вычисления

Christofari Neo выступает прокси-сервером для всех квантовых провайдеров. Пользователь указывает желаемого провайдера, а суперкомпьютер берёт на себя аутентификацию, отправку задачи и сбор результатов.

Таблица доступных квантовых провайдеров (актуально на 09.03.2026)

Провайдер Тип Технология / Модель Применение
dwave Аннилер (D-Wave) Квантовый отжиг Оптимизация QUBO, отбор молекул
ionq_aria Вентильный (IonQ Aria) Ионная ловушка, 25 к. Точный докинг, QAOA 
ionq_forte Вентильный (IonQ Forte) Ионная ловушка, 36+ к. Сложная квантовая химия 
rigetti Вентильный (Rigetti Ankaa) Сверхпроводники Гибридные алгоритмы 
iqm Вентильный (IQM Garnet) Сверхпроводники Высокочастотные задачи 
azure_quantum Вентильный (IonQ + Quantinuum) Ионные ловушки Квантовая химия, материаловедение
rosatom Вентильный (Росатом) Ионная ловушка (РФ) Задачи БРИКС+, госзаказ
sandboxaq Large Quantum Models AI + квант. симуляция Дизайн молекул, катализаторов 

---

6.1 Оптимизация (квантовый отжиг)

POST /quantum/optimize

```json
{
  "provider": "dwave",
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

Ответ (асинхронный):

```json
{
  "job_id": "qopt_550e8400",
  "status": "pending"
}
```

GET /quantum/result/{job_id} – получение результата.

---

6.2 Молекулярный докинг (гибридный QAOA)

POST /quantum/docking

```json
{
  "provider": "ionq_aria",
  "molecule_smiles": "CCO",
  "target_pdb": "3ERT",
  "options": {
    "hybrid_jobs": true,
    "classical_instances": "ml.m5.xlarge"
  }
}
```

---

6.3 Квантовая химия (симуляция электронной структуры)

POST /quantum/chemistry

```json
{
  "provider": "azure_quantum",
  "system": "ferrocene",
  "method": "vqe",
  "basis_set": "sto-3g"
}
```

---

💰 Стоимость и лимиты.

Все расценки указаны в долларах США. Оплата списывается с баланса VK Play.

Бесплатный тариф (для ознакомления)

· Классические вычисления: 100 запросов в день (кроме квантовых).
· Квантовые симуляторы: 1 час в месяц на SV1/DM1/TN1 (входит в AWS Free Tier) .
· Квантовые задачи на реальном QPU: не более 10 задач в сутки, одновременно не более 2.

Платные тарифы

Квантовые процессоры (Amazon Braket, цены за задачу + shot) 

Провайдер Задача (фикс) Shot (execution) Минимальное число shots
IonQ Aria $0.30 $0.03000 2500 (при error mitigation)
IonQ Forte $0.30 $0.08000 2500
Rigetti Ankaa $0.30 $0.00090 1
IQM Garnet $0.30 $0.00145 1
QuEra Aquila $0.30 $0.01000 1
AQT (IBEX Q1) $0.30 $0.02350 1

Пример расчёта (IonQ Aria, 10 000 shots):
$0.30 + (10 000 × $0.03) = $300.30 за задачу.

Часовая аренда QPU (Braket Direct) 

Провайдер Резервирование (1 час)
IonQ Aria $7 000.00
IonQ Forte $7 000.00
Rigetti Ankaa $5 750.00
IQM Garnet $3 000.00
QuEra Aquila $2 500.00

Квантовые симуляторы 

Тип Цена за минуту Минимальное время
SV1 (State Vector) $0.075 3 сек
DM1 (Density Matrix) $0.075 3 сек
TN1 (Tensor Network) $0.075 3 сек

Гибридные задания (Hybrid Jobs) 

· Базовый инстанс: ml.m5.xlarge – $0.23/час
· Дополнительно: оплата за QPU/simulator согласно тарифам выше.

SandboxAQ (Large Quantum Models) 

· Enterprise-решения, индивидуальное ценообразование (обычно от $100 000/год).
· Для получения доступа необходимо подписать NDA через VK Play.

---

⚠️ Коды ошибок

Код Описание
400 Неверный запрос (отсутствуют обязательные поля)
401 Недействительный или отсутствующий API-ключ
403 Доступ запрещён (недостаточно прав)
404 Ресурс не найден
429 Превышен лимит запросов
500 Внутренняя ошибка сервера

---

💻 Примеры использования cURL (через Christofari Neo)

```bash
# Получить список шаблонов анимаций
curl -H "Authorization: Bearer YOUR_API_KEY" \
     https://christofari-neo.snowapi.ru/animations/templates

# Запустить фолдинг с квантовым отжигом на D-Wave
curl -X POST -H "Authorization: Bearer YOUR_API_KEY" \
     -H "Content-Type: application/json" \
     -d '{
         "sequence": "MVLSPADKTNVKAAW...",
         "molecule_type": "protein",
         "engine": "quantum_annealing",
         "quantum_provider": "dwave"
     }' \
     https://christofari-neo.snowapi.ru/folding/predict

# Запустить оптимизацию на IonQ Aria (10 000 shots)
curl -X POST -H "Authorization: Bearer YOUR_API_KEY" \
     -H "Content-Type: application/json" \
     -d '{
         "provider": "ionq_aria",
         "problem_type": "qubo",
         "qubo": {"linear": {"0":-1,"1":-2}, "quadratic": {"0,1":3}},
         "params": {"num_reads": 10000}
     }' \
     https://christofari-neo.snowapi.ru/quantum/optimize
```

---

📞 Поддержка

· Технические вопросы по API: support@christofari-neo.snowapi.ru
· Биллинг и квоты VK Play: https://vkplay.ru/support
· GitHub (open source компоненты): https://github.com/snowapi/snowapi

---

© 2026 МАС «Снежинка». Все права защищены.
Суперкомпьютер Christofari Neo предоставлен ПАО «Сбер». Квантовый доступ обеспечен через партнёрство с Госкорпорацией «Росатом».
