# Database Schema

**Проект:** Pre-flight валидатор тендерной заявки
**База данных:** PostgreSQL 16+
**ORM:** Prisma 5+
**Дата:** 2026-02-06

---

## Prisma Schema

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  organization String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  checks    Check[]
}

model Check {
  id          String   @id @default(cuid())
  userId      String
  user        User     @relation(fields: [userId], references: [id])
  status      String   // 'pending', 'processing', 'completed', 'failed'
  type        String   // '44-fz'
  purchaseType String  // 'services'
  createdAt   DateTime @default(now())
  completedAt DateTime?
  errors      Error[]
  files       File[]
  checklists  Checklist[]
  summary     Json?    // { blockCount: number, warnCount: number }
}

model File {
  id        String   @id @default(cuid())
  checkId   String
  check     Check    @relation(fields: [checkId], references: [id])
  name      String
  type      String   // MIME type (application/pdf, etc.)
  size      Int
  url       String
  parsed    Json?    // Пarsed content (text, metadata, etc.)
  createdAt DateTime @default(now())
}

model Error {
  id          String   @id @default(cuid())
  checkId     String
  check       Check    @relation(fields: [checkId], references: [id])
  severity    String   // 'BLOCK', 'WARN'
  category    String   // 'anonymity', 'signature', 'docs', 'form2', 'deadline'
  ruleId      String   // Ссылка на правило из rule-engine
  message     String
  fix         String
  fileId      String?
  file        File?    @relation("ErrorFile", fields: [fileId], references: [id])
  createdAt   DateTime @default(now())
}

model Checklist {
  id          String   @id @default(cuid())
  checkId     String
  check       Check    @relation(fields: [checkId], references: [id])
  name        String   // 'Перед загрузкой', 'После Формы 2', 'Перед подачей'
  items       Json     // [{ title: string, checked: boolean }]
  completed   Boolean  @default(false)
  createdAt   DateTime @default(now())
}
```

---

## ER Diagram

```
User (1) ───── (N) Check
                    │
                    ├─ (N) File
                    ├─ (N) Error
                    └─ (N) Checklist
```

---

## Model Descriptions

### User
**Назначение:** Пользователи системы

**Поля:**
- `id` — уникальный идентификатор (cuid)
- `email` — email пользователя (уникальный)
- `name` — имя пользователя (опционально)
- `organization` — название организации (опционально)
- `createdAt` — дата создания
- `updatedAt` — дата последнего обновления

**Индексы:**
- `email` (unique)

---

### Check
**Назначение:** Проверки тендерных заявок

**Поля:**
- `id` — уникальный идентификатор (cuid)
- `userId` — ссылка на пользователя
- `status` — статус проверки ('pending', 'processing', 'completed', 'failed')
- `type` — тип закупки ('44-fz')
- `purchaseType` — тип закупки ('services', 'goods', 'construction', 'medical')
- `createdAt` — дата создания
- `completedAt` — дата завершения
- `summary` — сводка { blockCount: number, warnCount: number }

**Индексы:**
- `userId`
- `status`
- `createdAt`

---

### File
**Назначение:** Загруженные файлы

**Поля:**
- `id` — уникальный идентификатор (cuid)
- `checkId` — ссылка на проверку
- `name` — имя файла
- `type` — MIME type (application/pdf, application/vnd.openxmlformats-officedocument.wordprocessingml.document, etc.)
- `size` — размер файла в байтах
- `url` — URL файла в storage
- `parsed` — распарсенный контент (текст, метаданные, etc.)
- `createdAt` — дата загрузки

**Индексы:**
- `checkId`

---

### Error
**Назначение:** Ошибки и предупреждения

**Поля:**
- `id` — уникальный идентификатор (cuid)
- `checkId` — ссылка на проверку
- `severity` — критичность ('BLOCK', 'WARN')
- `category` — категория ошибки ('anonymity', 'signature', 'docs', 'form2', 'deadline')
- `ruleId` — ссылка на правило из rule-engine
- `message` — описание ошибки
- `fix` — как исправить
- `fileId` — ссылка на файл (опционально)
- `createdAt` — дата создания

**Индексы:**
- `checkId`
- `severity`
- `category`

---

### Checklist
**Назначение:** Чек-листы готовности к подаче

**Поля:**
- `id` — уникальный идентификатор (cuid)
- `checkId` — ссылка на проверку
- `name` — название чек-листа ('Перед загрузкой', 'После Формы 2', 'Перед подачей')
- `items` — элементы чек-листа [{ title: string, checked: boolean }]
- `completed` — выполнен ли чек-лист
- `createdAt` — дата создания

**Индексы:**
- `checkId`

---

## Check Statuses

| Статус | Описание |
|---|---|
| **pending** | Проверка создана, но не запущена |
| **processing** | Проверка в процессе |
| **completed** | Проверка завершена |
| **failed** | Проверка завершилась с ошибкой |

---

## Error Severities

| Severity | Описание | Действие |
|---|---|---|
| **BLOCK** | Критичная ошибка, блокирует подачу | Обязательно исправить |
| **WARN** | Предупреждение, рекомендация | Рекомендуется исправить |

---

## Error Categories

| Категория | Описание | Примеры |
|---|---|---|
| **anonymity** | Нарушение анонимности | Логотипы, названия, ИНН |
| **signature** | Проблемы с подписями | Нет КЭП, сертификат истек |
| **docs** | Проблемы с документами | Нет обязательного документа |
| **form2** | Проблемы с Формой 2 | Не заполнено поле, противоречия |
| **deadline** | Проблемы с дедлайном | Мало времени до дедлайна |

---

## Purchase Types

| Тип | Описание |
|---|---|
| **services** | Услуги |
| **goods** | Поставка товаров |
| **construction** | Строительство |
| **medical** | Медицинские изделия |

---

## Migration

```bash
# Создать миграцию
npx prisma migrate dev --name init

# Применить миграцию
npx prisma migrate deploy

# Сбросить базу данных (для dev)
npx prisma migrate reset

# Генерировать Prisma Client
npx prisma generate
```

---

## Environment Variables

```env
DATABASE_URL="postgresql://user:password@host:5432/database"
```

---

## Next Steps

1. ✅ Database schema спроектирована
2. ⏸️ Создать миграцию `npx prisma migrate dev --name init`
3. ⏸️ Создать Prisma Client
4. ⏸️ Написать seed данные (для dev)
5. ⏸️ Написать unit тесты для моделей
