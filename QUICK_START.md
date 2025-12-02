# Быстрый старт

## Установка и запуск

1. **Установите зависимости:**
   ```bash
   npm install
   ```

2. **Создайте файл `.env`** (опционально, если backend на другом адресе):
   ```env
   VITE_API_BASE_URL=http://localhost:8000/api
   ```

3. **Запустите dev-сервер:**
   ```bash
   npm run dev
   ```

4. Откройте браузер по адресу `http://localhost:3000`

## Структура приложения

### Основные страницы:
- `/dashboard` - Главный дашборд с метриками
- `/calls` - Список звонков
- `/calls/:id` - Детальная карточка звонка
- `/analytics/employees` - Аналитика сотрудников
- `/analytics/trends` - Аналитика трендов
- `/chats` - Список чатов
- `/chats/:id` - Детальная карточка чата
- `/reports` - Отчёты
- `/settings` - Настройки

### Компоненты:

**Layout:**
- `Sidebar` - Боковое меню навигации
- `Header` - Верхняя панель с фильтрами

**UI Components:**
- `MetricCard` - Карточки метрик
- `RiskBadge` - Бейдж риска
- `ScoreBadge` - Бейдж оценки
- `AudioPlayer` - Плеер для аудио
- `TranscriptViewer` - Просмотр транскрипций
- `EmotionTimeline` - Таймлайн эмоций

**Charts:**
- `ChartLine` - Линейный график
- `ChartBar` - Столбчатый график
- `ChartPie` - Круговой график

## Интеграция с Backend

Приложение использует REST API. Все запросы идут через `apiClient` из `src/api/client.ts`.

Примеры использования хуков:

```typescript
import { useCalls } from '@/hooks/useCalls'

const { data: calls, isLoading } = useCalls({ period: 'week' })
```

## Тёмная тема

Переключение темы доступно через кнопку в Header (иконка солнца/луны).

## Следующие шаги

1. Подключите реальный backend API
2. Замените mock данные на реальные запросы
3. Настройте аутентификацию
4. Добавьте обработку ошибок
5. Настройте Desktop версию (Tauri/Electron)


