# Final_Task
# Distributed Calculator

Распределённая система для вычисления арифметических выражений с поддержкой многопользовательского режима, JWT-аутентификацией, асинхронной обработкой и распределением задач между агентами.

---

## 📋 Содержание

- [Требования](#-требования)  
- [Структура проекта](#-структура-проекта)  
- [Настройка окружения](#-настройка-окружения)  
- [Сборка и запуск](#️-сборка-и-запуск)  
- [API Оркестратора](#api-оркестратора)  
- [API Агента (внутреннее)](#api-агента-внутреннее)  
- [Миграции БД](#миграции-бд)  
- [Переменные окружения](#-переменные-окружения)  
- [Лицензия](#-лицензия)  

---

## ⚙️ Требования

- Docker ≥ 20.10  
- Docker Compose ≥ 1.29  
- Git  

---

## 📂 Структура проекта

```text
distributed-calculator/
├── docker-compose.yml
├── orchestrator/
│   ├── main.go
│   ├── Dockerfile
│   ├── handlers/
│   ├── services/
│   ├── models/
│   ├── db/
│   └── utils/
├── agent/
│   ├── main.go
│   ├── Dockerfile
│   ├── compute/
│   ├── handlers/
│   └── models/
├── migrations/
│   └── 001_init.sql
└── 1.env
```

---

## 🔧 Настройка окружения
Скопируйте пример файла с переменными окружения и, при желании, отредактируйте:

bash
Копировать
Редактировать
cp 1.env.example 1.env
Проверьте содержимое 1.env:

.env
```
DB_HOST=db
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=postgres
DB_NAME=calculator

JWT_SECRET=supersecretkey

# Задержки операций в миллисекундах
TIME_ADDITION_MS=1000
TIME_SUBTRACTION_MS=1000
TIME_MULTIPLICATIONS_MS=2000
TIME_DIVISIONS_MS=3000

# Количество параллельных воркеров агента
COMPUTING_POWER=4
```

---


## 🚀 Сборка и запуск
В корне проекта выполните:

-bash
```
git clone https://github.com/your-user/distributed-calculator.git
cd distributed-calculator
```
# Собрать образы и поднять сервисы
```
docker-compose up --build
```
После успешного запуска:
Оркестратор слушает порт 8080
Агент слушает порт 8081
PostgreSQL — порт 5432
API Оркестратора
Все публичные эндпойнты расположены под префиксом /api/v1.
## 🔑 1. Регистрация пользователя

### cURL

```bash
curl -X POST http://localhost:8080/api/v1/register \
  -H "Content-Type: application/json" \
  -d '{
    "username": "user1",
    "password": "secret"
  }'
```
Postman
Method: POST

URL: http://localhost:8080/api/v1/register

Headers:

Content-Type: application/json

Body (raw JSON):

json
Копировать
Редактировать
{
  "username": "user1",
  "password": "secret"
}
🔑 2. Логин и получение JWT
cURL
bash
Копировать
Редактировать
curl -X POST http://localhost:8080/api/v1/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "user1",
    "password": "secret"
  }'
Postman
Method: POST

URL: http://localhost:8080/api/v1/login

Headers:

Content-Type: application/json

Body (raw JSON):

json
Копировать
Редактировать
{
  "username": "user1",
  "password": "secret"
}
Ответ содержит токен:

json
Копировать
Редактировать
{
  "token": "<JWT_TOKEN>"
}
⚠️ Все дальнейшие запросы требуют заголовок:
Authorization: Bearer <JWT_TOKEN>

➗ 3. Отправка выражения на вычисление
cURL
bash
Копировать
Редактировать
curl -X POST http://localhost:8080/api/v1/calculate \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <JWT_TOKEN>" \
  -d '{
    "expression": "(2+3)*4/5 - 1"
  }'
Postman
Method: POST

URL: http://localhost:8080/api/v1/calculate

Headers:

Content-Type: application/json

Authorization: Bearer <JWT_TOKEN>

Body (raw JSON):

json
Копировать
Редактировать
{
  "expression": "(2+3)*4/5 - 1"
}
Ответ:

json
Копировать
Редактировать
{
  "expression_id": 42
}
📋 4. Получение всех выражений пользователя
cURL
bash
Копировать
Редактировать
curl -X GET http://localhost:8080/api/v1/expressions \
  -H "Authorization: Bearer <JWT_TOKEN>"
Postman
Method: GET

URL: http://localhost:8080/api/v1/expressions

Headers:

Authorization: Bearer <JWT_TOKEN>

Пример ответа:

json
Копировать
Редактировать
[
  {
    "id": 42,
    "expression": "(2+3)*4/5 - 1",
    "result": 3.0,
    "status": "done",
    "user_id": 1
  }
]
🔍 5. Получение конкретного выражения
cURL
bash
Копировать
Редактировать
curl -X GET http://localhost:8080/api/v1/expressions/42 \
  -H "Authorization: Bearer <JWT_TOKEN>"
Postman
Method: GET

URL: http://localhost:8080/api/v1/expressions/{{expression_id}}

Headers:

Authorization: Bearer <JWT_TOKEN>

🛠 6. Внутренний API Оркестратора (используется агентами)
🔄 6.1 Получить задачу
cURL
bash
Копировать
Редактировать
curl -X GET http://localhost:8080/internal/task
Postman
Method: GET

URL: http://localhost:8080/internal/task

Ответ:

json
Копировать
Редактировать
{
  "task": {
    "id": 7,
    "arg1": 5.0,
    "arg2": 4.0,
    "operation": "*"
  }
}
✅ 6.2 Отправить результат выполнения
cURL
bash
Копировать
Редактировать
curl -X POST http://localhost:8080/internal/result \
  -H "Content-Type: application/json" \
  -d '{
    "id": 7,
    "result": 20.0
  }'
Postman
Method: POST

URL: http://localhost:8080/internal/result

Headers:

Content-Type: application/json

Body (raw JSON):

json
Копировать
Редактировать
{
  "id": 7,
  "result": 20.0
}
Ответ:

json
Копировать
Редактировать
{
  "status": "ok"
}

🗄️ Миграции БД
Все миграции лежат в папке migrations/. При старте оркестратора они автоматически применяются.

