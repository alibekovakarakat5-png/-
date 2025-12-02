# 🚀 Руководство по тестированию AI CRM Analytics

## ✅ Статус системы

- ✅ Backend запущен на **http://localhost:8080**
- ✅ Frontend запущен на **http://localhost:3000**
- ✅ База данных SQLite настроена
- ✅ Переменные окружения загружены
- ✅ Синхронизация с Bitrix работает

## 📋 Тестирование функционала

### 1. Проверка работы backend

```powershell
# Health check
Invoke-WebRequest -Uri http://localhost:8080/api/health

# Получение звонков
Invoke-WebRequest -Uri http://localhost:8080/api/calls

# Синхронизация из Bitrix
Invoke-WebRequest -Uri http://localhost:8080/api/calls/sync -Method POST
```

### 2. Создание тестового звонка

**Вариант A: Через API endpoint**

```powershell
# Создать тестовый звонок
$body = @{
    managerName = "Иван Иванов"
    clientPhone = "+7 (999) 123-45-67"
} | ConvertTo-Json

Invoke-WebRequest -Uri http://localhost:8080/api/test-call/create -Method POST -Body $body -ContentType "application/json"
```

**Вариант B: Через webhook (для реальных данных)**

```powershell
# Получить токен webhook
$settings = Invoke-WebRequest -Uri http://localhost:8080/api/settings | ConvertFrom-Json
$token = $settings.incomingWebhookUrl -replace '.*/incoming/', ''

# Создать звонок через webhook
$body = @{
    type = "call"
    clientPhone = "+7 (999) 123-45-67"
    managerName = "Петр Петров"
    recordUrl = "https://example.com/audio.mp3"
    autoProcess = $true
} | ConvertTo-Json

Invoke-WebRequest -Uri "http://localhost:8080/api/webhooks/incoming/$token" -Method POST -Body $body -ContentType "application/json"
```

### 3. Обработка звонка (транскрибация + анализ)

После создания звонка получите `callId` из ответа, затем:

```powershell
# Обработать звонок (транскрибация + анализ)
Invoke-WebRequest -Uri http://localhost:8080/api/calls/1/process -Method POST

# Или для тестового звонка
Invoke-WebRequest -Uri http://localhost:8080/api/test-call/1/process -Method POST
```

**Что происходит:**
1. Скачивание аудио (если есть `recordUrl`)
2. Транскрибация через faster-whisper (Python)
3. Анализ через OpenAI GPT-4o
4. Сохранение результатов в БД
5. Отправка алерта в Telegram (если высокий риск)

### 4. Просмотр результатов

**Через API:**
```powershell
# Детальная информация о звонке
Invoke-WebRequest -Uri http://localhost:8080/api/calls/1

# Список всех звонков
Invoke-WebRequest -Uri http://localhost:8080/api/calls
```

**Через Frontend:**
1. Откройте http://localhost:3000
2. Перейдите в раздел **"Звонки"**
3. Нажмите на звонок для просмотра деталей
4. Увидите:
   - Транскрипцию
   - Анализ по блокам
   - Оценки и рекомендации
   - Таймлайн эмоций

### 5. Аналитика

**Сотрудники:**
```powershell
Invoke-WebRequest -Uri http://localhost:8080/api/analytics/employees?period=week
```

**Тренды:**
```powershell
Invoke-WebRequest -Uri http://localhost:8080/api/analytics/trends?period=week
```

**На Frontend:**
- Раздел **"Аналитика сотрудников"** - рейтинги, графики улучшений
- Раздел **"Аналитика трендов"** - топ товаров, причины отказов

### 6. Генерация и отправка отчётов в Telegram

```powershell
# Отчёт по сотрудникам
Invoke-WebRequest -Uri http://localhost:8080/api/reports/generate/employees -Method POST

# Проблемные звонки
Invoke-WebRequest -Uri http://localhost:8080/api/reports/generate/problematic-calls -Method POST

# Тренды
Invoke-WebRequest -Uri http://localhost:8080/api/reports/generate/trends -Method POST

# Тестовая отправка
Invoke-WebRequest -Uri http://localhost:8080/api/reports/test-telegram -Method POST
```

Отчёты будут отправлены в Telegram чат (ID: 975281619)

### 7. Тестирование транскрибации

```powershell
# Тест транскрибации (нужен реальный MP3 файл)
Invoke-WebRequest -Uri "http://localhost:8080/api/test/transcription?file=C:/path/to/audio.mp3"
```

## 🎯 Полный цикл тестирования

### Шаг 1: Создать тестовый звонок
```powershell
$call = Invoke-WebRequest -Uri http://localhost:8080/api/test-call/create -Method POST -Body (@{managerName="Тест"; clientPhone="+79991234567"} | ConvertTo-Json) -ContentType "application/json" | ConvertFrom-Json
$callId = $call.callId
```

### Шаг 2: Обработать звонок
```powershell
Invoke-WebRequest -Uri "http://localhost:8080/api/test-call/$callId/process" -Method POST
```

### Шаг 3: Просмотреть результаты
- Откройте http://localhost:3000
- Перейдите в "Звонки"
- Найдите созданный звонок
- Откройте детали

### Шаг 4: Проверить аналитику
- Раздел "Аналитика сотрудников"
- Раздел "Аналитика трендов"

### Шаг 5: Скачать отчёт в Telegram
- Раздел "Отчёты"
- Выберите тип отчёта
- Нажмите "Скачать" или используйте API

## 🔧 Troubleshooting

### Backend не отвечает
```powershell
# Проверить процессы
Get-Process node

# Перезапустить backend
cd backend
npm run dev
```

### Нет данных на фронтенде
1. Проверьте консоль браузера (F12)
2. Убедитесь что backend запущен
3. Проверьте CORS настройки

### Ошибка транскрибации
1. Проверьте что Python путь правильный в `.env`
2. Проверьте что faster-whisper установлен
3. Проверьте логи backend

### Ошибка анализа OpenAI
1. Проверьте API ключ в `.env`
2. Проверьте что есть кредиты на аккаунте
3. Проверьте логи backend

## 📊 API Endpoints

- `GET /api/health` - проверка работы
- `GET /api/calls` - список звонков
- `GET /api/calls/:id` - детали звонка
- `POST /api/calls/sync` - синхронизация из Bitrix
- `POST /api/calls/:id/process` - обработка звонка
- `POST /api/test-call/create` - создать тестовый звонок
- `POST /api/test-call/:id/process` - обработать тестовый звонок
- `GET /api/analytics/employees` - аналитика сотрудников
- `GET /api/analytics/trends` - аналитика трендов
- `POST /api/reports/generate/:type` - генерация отчёта
- `POST /api/reports/test-telegram` - тест Telegram

## ✅ Готово к тестированию!

Все системы запущены и готовы к работе. Можете создавать тестовые звонки, получать транскрипции, анализировать их и скачивать отчёты в Telegram.

