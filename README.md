# МАС «Снежинка» API

Базовая URL: https://snowapi.github.io/

API предоставляет программный доступ к вычислительным модулям проекта МАС «Снежинка» на платформе VK Play. Все запросы требуют аутентификации по API-ключу, выданному платформой.

---

🔑 Аутентификация

API-ключ передаётся в HTTP-заголовке:

```
Authorization: Bearer <ваш_ключ>
```

Ключ можно получить в личном кабинете разработчика на VK Play.

---

📡 Эндпоинты

1. Биоинформатика

POST /bio/analyze

Анализ биологических последовательностей (нуклеотидных или аминокислотных).

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

Работа с химическими соединениями.

Параметры запроса (JSON):

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

Параметры запроса (JSON):

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

Получение списка доступных шаблонов атомарных и молекулярных анимаций.

Пример ответа:

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

Запрос рендера анимации с параметрами.

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

Ответ: задача ставится в очередь, возвращается job_id.

---

5. Фолдинг белков и ДНК

POST /folding/predict

Предсказание трёхмерной структуры по последовательности.

Параметры запроса (JSON):

```json
{
  "sequence": "MVLSPADKTNVKAAW...",
  "molecule_type": "protein | dna",
  "engine": "deepfold-plm | trRosetta | alphafold",
  "options": {
    "return_pdb": true,
    "return_confidence": true
  }
}
```

Пример ответа (асинхронный):

```json
{
  "job_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "pending"
}
```

GET /folding/result/{job_id}

Получение результата по завершении.

Пример ответа (готовый):

```json
{
  "pdb": "ATOM      1  N   MET A   1     -5.123  2.456  1.789  1.00 15.23           N\n...",
  "confidence": [0.95, 0.87, ...],
  "visualization_url": "https://snowapi.github.io/view/550e8400"
}
```

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

📊 Лимиты и квоты

· Бесплатный тариф: 100 запросов в день, 1 запрос в секунду.
· Платные тарифы: см. VK Play Developer Console.

Лимиты рассчитываются отдельно для каждого эндпоинта (кроме асинхронных задач, где учитывается только запуск).

---

📦 Пример использования (cURL)

```bash
# Получение списка шаблонов анимаций
curl -H "Authorization: Bearer YOUR_API_KEY" \
     https://snowapi.github.io/animations/templates

# Запуск фолдинга белка
curl -X POST -H "Authorization: Bearer YOUR_API_KEY" \
     -H "Content-Type: application/json" \
     -d '{"sequence":"MVLSPADKTNVKAAW...","molecule_type":"protein"}' \
     https://snowapi.github.io/folding/predict
```

---

📞 Поддержка

По вопросам работы API обращайтесь в службу поддержки VK Play или создавайте issue в репозитории snowapi/snowapi.

---

© 2026 МАС «Снежинка». Все права защищены.
