# 🚀 Запуск Backend и получение данных из Bitrix

## ✅ Текущий статус

- ✅ Backend запущен на **http://localhost:8080**
- ✅ Health endpoint работает: `GET http://localhost:8080/api/health`
- ❌ База данных не настроена (нужно настроить PostgreSQL)

## 📋 Что нужно сделать

### Шаг 1: Настройте базу данных

**Вариант A: PostgreSQL (рекомендуется)**

1. Убедитесь, что PostgreSQL установлен:
   ```powershell
   psql --version
   ```

2. Создайте базу данных:
   ```powershell
   psql -U postgres
   # В psql:
   CREATE DATABASE crm_analytics;
   \q
   ```

3. Обновите `backend/.env`:
   ```
   DATABASE_URL=postgresql://postgres:ВАШ_ПАРОЛЬ@localhost:5432/crm_analytics
   ```

4. Примените миграции:
   ```powershell
   cd backend
   npx prisma migrate dev
   ```

**Вариант B: SQLite (для быстрого теста)**

1. Измените `backend/prisma/schema.prisma`:
   ```prisma
   datasource db {
     provider = "sqlite"
     url      = "file:./dev.db"
   }
   ```

2. Обновите `backend/.env`:
   ```
   DATABASE_URL="file:./dev.db"
   ```

3. Примените миграции:
   ```powershell
   cd backend
   npx prisma generate
   npx prisma migrate dev
   ```

### Шаг 2: Перезапустите backend

```powershell
cd backend
npm run dev
```

### Шаг 3: Синхронизируйте данные из Bitrix

```powershell
Invoke-WebRequest -Uri http://localhost:8080/api/calls/sync -Method POST -ContentType "application/json"
```

Или через curl (если установлен):
```bash
curl -X POST http://localhost:8080/api/calls/sync
```

### Шаг 4: Проверьте данные

**Через API:**
```powershell
curl http://localhost:8080/api/calls
```

**Через фронтенд:**
1. Убедитесь, что фронтенд запущен: `npm run dev` (в корне проекта)
2. Откройте http://localhost:3000
3. Перейдите в раздел "Звонки"
4. Должны отображаться данные из Bitrix

## 🔍 Проверка работы

### 1. Health check
```powershell
curl http://localhost:8080/api/health
```
Ожидаемый ответ: `{"status":"ok"}`

### 2. Настройки
```powershell
curl http://localhost:8080/api/settings
```

### 3. Список звонков
```powershell
curl http://localhost:8080/api/calls
```

### 4. Синхронизация
```powershell
Invoke-WebRequest -Uri http://localhost:8080/api/calls/sync -Method POST
```

## 📝 Документация

- `backend/QUICK_DB_SETUP.md` - быстрая настройка БД
- `backend/SETUP_DATABASE.md` - подробная инструкция
- `backend/README.md` - полная документация

## ⚠️ Важные замечания

1. **DATABASE_URL** в `.env` должен быть правильным
2. **BITRIX_WEBHOOK_URL** должен быть активным и иметь права на чтение звонков
3. После настройки БД обязательно примените миграции: `npx prisma migrate dev`

## 🎯 Следующие шаги после настройки БД

1. Синхронизируйте звонки из Bitrix
2. Обработайте звонки (транскрибация + анализ)
3. Просмотрите данные на фронтенде
4. Проверьте аналитику сотрудников и трендов

