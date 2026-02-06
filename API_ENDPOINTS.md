# API Endpoints

**Проект:** Pre-flight валидатор тендерной заявки
**Фреймворк:** Next.js 15 API Routes
**Дата:** 2026-02-06

---

## Overview

| Метод | Endpoint | Описание |
|---|---|---|
| POST | `/api/upload` | Загрузка файлов |
| POST | `/api/validate` | Запуск проверки |
| GET | `/api/report/:id` | Получение отчета |
| GET | `/api/checks` | Список проверок пользователя |
| GET | `/api/check/:id` | Детали проверки |
| DELETE | `/api/check/:id` | Удаление проверки |

---

## POST /api/upload

**Назначение:** Загрузка файлов для проверки

**Request:**
```http
POST /api/upload
Content-Type: multipart/form-data
```

**Body (FormData):**
```
files: File[]
```

**Response (200 OK):**
```json
{
  "fileIds": ["cuid1", "cuid2", "cuid3"],
  "checkId": "cuid_check_id"
}
```

**Error (400 Bad Request):**
```json
{
  "error": "Invalid file format",
  "details": "Only PDF, DOC, DOCX, XLS, XLSX, SIG, XML allowed"
}
```

**Error (413 Payload Too Large):**
```json
{
  "error": "File too large",
  "details": "Maximum file size is 100 MB"
}
```

---

## POST /api/validate

**Назначение:** Запуск проверки заявки

**Request:**
```http
POST /api/validate
Content-Type: application/json
```

**Body:**
```json
{
  "checkId": "cuid_check_id",
  "type": "44-fz",
  "purchaseType": "services"
}
```

**Response (202 Accepted):**
```json
{
  "checkId": "cuid_check_id",
  "status": "processing"
}
```

**Error (404 Not Found):**
```json
{
  "error": "Check not found"
}
```

---

## GET /api/report/:id

**Назначение:** Получение отчета по проверке

**Request:**
```http
GET /api/report/:id
```

**Response (200 OK):**
```json
{
  "status": "completed",
  "summary": {
    "blockCount": 2,
    "warnCount": 5
  },
  "errors": [
    {
      "id": "cuid_error_id",
      "severity": "BLOCK",
      "category": "anonymity",
      "ruleId": "ANON-001",
      "message": "В документе найден логотип компании",
      "fix": "Удалите логотип или замените документ",
      "fileId": "cuid_file_id",
      "fileName": "document.pdf"
    }
  ],
  "checklists": [
    {
      "id": "cuid_checklist_id",
      "name": "Перед загрузкой",
      "completed": false,
      "items": [
        {
          "title": "Документы анонимны?",
          "checked": false
        },
        {
          "title": "Форматы верны?",
          "checked": true
        }
      ]
    }
  ]
}
```

**Error (404 Not Found):**
```json
{
  "error": "Check not found"
}
```

**Error (409 Conflict):**
```json
{
  "error": "Check not completed",
  "status": "processing"
}
```

---

## GET /api/checks

**Назначение:** Список проверок пользователя

**Request:**
```http
GET /api/checks?limit=10&offset=0
```

**Query Parameters:**
- `limit` (optional) — количество проверок (default: 20)
- `offset` (optional) — смещение (default: 0)
- `status` (optional) — фильтр по статусу ('pending', 'processing', 'completed', 'failed')

**Response (200 OK):**
```json
{
  "checks": [
    {
      "id": "cuid_check_id",
      "status": "completed",
      "type": "44-fz",
      "purchaseType": "services",
      "createdAt": "2026-02-06T10:00:00Z",
      "completedAt": "2026-02-06T10:05:00Z",
      "summary": {
        "blockCount": 2,
        "warnCount": 5
      }
    }
  ],
  "total": 42
}
```

---

## GET /api/check/:id

**Назначение:** Детали проверки

**Request:**
```http
GET /api/check/:id
```

**Response (200 OK):**
```json
{
  "id": "cuid_check_id",
  "status": "completed",
  "type": "44-fz",
  "purchaseType": "services",
  "createdAt": "2026-02-06T10:00:00Z",
  "completedAt": "2026-02-06T10:05:00Z",
  "summary": {
    "blockCount": 2,
    "warnCount": 5
  },
  "files": [
    {
      "id": "cuid_file_id",
      "name": "document.pdf",
      "type": "application/pdf",
      "size": 1024000,
      "createdAt": "2026-02-06T10:00:00Z"
    }
  ]
}
```

**Error (404 Not Found):**
```json
{
  "error": "Check not found"
}
```

---

## DELETE /api/check/:id

**Назначение:** Удаление проверки

**Request:**
```http
DELETE /api/check/:id
```

**Response (204 No Content):**

**Error (404 Not Found):**
```json
{
  "error": "Check not found"
}
```

---

## Error Responses

Все ошибки возвращаются в формате:

```json
{
  "error": "Error message",
  "details": "Detailed error message (optional)"
}
```

**HTTP Status Codes:**
- `200 OK` — успешный запрос
- `202 Accepted` — запрос принят, обработка в процессе
- `400 Bad Request` — невалидные данные
- `404 Not Found` — ресурс не найден
- `409 Conflict` — конфликт состояния
- `413 Payload Too Large` — слишком большой файл
- `500 Internal Server Error` — внутренняя ошибка сервера

---

## Authentication & Authorization

**Текущее состояние:** Без аутентификации (MVP)

**План:**
- V1.1: Simple email/password (NextAuth.js)
- V1.2: OAuth (Google, GitHub)
- V2.0: SSO (Enterprise)

---

## Rate Limiting

**Текущее состояние:** Нет (MVP)

**План:**
- Vercel Rate Limiting (100 req/min)
- Redis-based rate limiting (продакшн)

---

## File Upload

**Поддерживаемые форматы:**
- PDF (`.pdf`)
- DOC (`.doc`)
- DOCX (`.docx`)
- XLS (`.xls`)
- XLSX (`.xlsx`)
- SIG (`.sig`)
- XML (`.xml`)

**Ограничения:**
- Максимальный размер файла: 100 МБ
- Максимальное количество файлов: 50

**Storage:** Vercel Blob (development) / S3 (production)

---

## Validation Pipeline

```
1. POST /api/upload
   ├─ Валидация форматов
   ├─ Валидация размеров
   ├─ Загрузка в storage
   └─ Создание Check в DB

2. POST /api/validate
   ├─ Валидация типа закупки
   ├─ Изменение статуса на 'processing'
   └─ Запуск background job

3. Background job (в отдельном потоке)
   ├─ Парсинг файлов
   ├─ Проверка по правилам (rule-engine)
   ├─ Генерация отчета
   ├─ Сохранение ошибок в DB
   └─ Изменение статуса на 'completed'

4. GET /api/report/:id
   ├─ Проверка статуса
   ├─ Загрузка ошибок из DB
   └─ Возврат отчета
```

---

## Background Jobs

**Текущее состояние:** Прямой вызов (MVP)

**План:**
- V1.1: BullMQ (Redis)
- V2.0: AWS SQS / RabbitMQ

---

## Next Steps

1. ✅ API endpoints спроектированы
2. ⏸️ Реализовать POST /api/upload
3. ⏸️ Реализовать POST /api/validate
4. ⏸️ Реализовать GET /api/report/:id
5. ⏸️ Реализовать остальные endpoints
6. ⏸️ Написать unit тесты
7. ⏸️ Написать интеграционные тесты
