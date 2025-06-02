# 🚀 Система Регистрации Клиентов GeminiVoice AI

## 📋 Обзор

Комплексная система регистрации клиентов с автоматическими AI звонками, анализом потребностей через Gemini API и управлением API ключами для модемов.

## 🎯 Основные Возможности

### 1. Регистрация Клиентов
- ✅ Регистрация с email верификацией
- ✅ Подтверждение номера телефона
- ✅ Автоматическое назначение номера компании
- ✅ Многоэтапный процесс регистрации

### 2. AI Звонки
- ✅ Автоматические AI звонки через Gemini API
- ✅ Интеллектуальные диалоги с клиентами
- ✅ Анализ потребностей в реальном времени
- ✅ Генерация персональных рекомендаций

### 3. Анализ и Рекомендации
- ✅ AI анализ разговоров с клиентами
- ✅ Генерация готовых промптов
- ✅ Расчет экономии времени и денег
- ✅ Персональные предложения автоматизации

### 4. Админ Панель
- ✅ Управление номерами компании
- ✅ Назначение API ключей Gemini
- ✅ Конфигурация модемов
- ✅ Статистика и аналитика

## 🏗️ Архитектура

### Backend Компоненты

#### Контроллеры
- `registrationController.js` - Регистрация и аутентификация клиентов
- `callController.js` - Управление AI звонками
- `adminController.js` - Админ панель и управление системой

#### Сервисы
- `emailService.js` - Отправка email уведомлений
- `companyNumberService.js` - Управление номерами компании
- `aiCallService.js` - AI звонки и анализ через Gemini

#### Маршруты
- `/api/registration` - Регистрация клиентов
- `/api/ai-calls` - AI звонки
- `/api/admin` - Админ панель

### База Данных

#### Новые Модели
```prisma
model CompanyNumber {
  id           String  @id @default(cuid())
  number       String  @unique
  isActive     Boolean @default(true)
  geminiApiKey String?
  modemId      String?
  assignedUser User?
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt
}

model CallSession {
  id            String   @id @default(cuid())
  userId        String
  companyNumber String
  clientPhone   String
  status        String   // 'in_progress', 'completed', 'failed'
  startTime     DateTime
  endTime       DateTime?
  duration      Int?     // в секундах
  transcript    String?  // JSON
  user          User     @relation(fields: [userId], references: [id])
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt
}

model AIAnalysis {
  id                    String   @id @default(cuid())
  userId                String
  callSessionId         String
  clientNeeds           String
  recommendedFeatures   String   // JSON
  generatedPrompts      String   // JSON
  automationSuggestions String  // JSON
  geminiResponse        String
  user                  User     @relation(fields: [userId], references: [id])
  callSession          CallSession @relation(fields: [callSessionId], references: [id])
  createdAt            DateTime @default(now())
  updatedAt            DateTime @updatedAt
}

model SavingsCalculation {
  id                    String   @id @default(cuid())
  userId                String   @unique
  estimatedTimeSavings  Int      // часов в месяц
  estimatedMoneySavings Float    // долларов в месяц
  currentCosts          Float    // текущие расходы
  automationLevel       String   // 'low', 'medium', 'high'
  calculationData       String   // JSON с деталями
  user                  User     @relation(fields: [userId], references: [id])
  createdAt             DateTime @default(now())
  updatedAt             DateTime @updatedAt
}

model ModemConfig {
  id                  String   @id @default(cuid())
  modemId             String   @unique
  geminiApiKey        String?
  isActive            Boolean  @default(true)
  assignedNumber      String?
  maxConcurrentCalls  Int      @default(1)
  createdAt           DateTime @default(now())
  updatedAt           DateTime @updatedAt
}
```

#### Обновленная Модель User
```prisma
model User {
  // ... существующие поля ...
  emailVerified     Boolean   @default(false)
  emailVerifyToken  String?
  phone             String?
  assignedNumber    String?
  registrationStep  String    @default("email_verification")
  // 'email_verification', 'phone_verification', 'call_scheduled', 'call_completed', 'analysis_done'
}
```

## 🔄 Процесс Регистрации

### Этапы Регистрации

1. **Email Verification** (`email_verification`)
   - Пользователь регистрируется с email и паролем
   - Отправляется email с токеном верификации
   - После подтверждения переход к следующему этапу

2. **Phone Verification** (`phone_verification`)
   - Пользователь указывает номер телефона
   - Автоматически назначается номер компании
   - Отправляется приветственный email с номером

3. **Call Scheduled** (`call_scheduled`)
   - Пользователь может позвонить на назначенный номер
   - Система готова принять AI звонок
   - Ожидание звонка от клиента

4. **Call Completed** (`call_completed`)
   - AI звонок завершен
   - Транскрипт сохранен в базе данных
   - Запущен процесс анализа

5. **Analysis Done** (`analysis_done`)
   - AI анализ завершен
   - Рекомендации сгенерированы
   - Email с результатами отправлен

## 📞 AI Звонки

### Возможности AI Консультанта

- **Персонализированное приветствие** для каждого клиента
- **Интеллектуальный диалог** на основе Gemini API
- **Анализ потребностей** в реальном времени
- **Генерация рекомендаций** по автоматизации
- **Расчет экономии** времени и денег

### Пример Диалога

```
AI: Здравствуйте, Алексей! Добро пожаловать в GeminiVoice AI Call Center!
    Меня зовут Алиса, я ваш персональный AI-консультант.

Клиент: Здравствуйте! Я хочу узнать о ваших услугах автоматизации.

AI: Отлично! Расскажите о вашем бизнесе и задачах, которые хотите автоматизировать?

Клиент: У меня интернет-магазин, много времени уходит на обработку заказов.

AI: Понимаю! Для интернет-магазинов мы можем автоматизировать:
    - Прием заказов через AI чат-бота
    - Консультации клиентов 24/7
    - Обработку возвратов и жалоб
    
    Сколько заказов вы обрабатываете в день?
```

## 🎯 Анализ и Рекомендации

### Что Анализирует AI

- **Тип бизнеса** клиента
- **Текущие проблемы** и задачи
- **Потенциал автоматизации**
- **Экономическую выгоду**

### Что Генерирует

- **Готовые промпты** для AI инструментов
- **Конкретные решения** автоматизации
- **Расчет ROI** и экономии
- **План внедрения**

### Пример Результата

```json
{
  "clientNeeds": "Автоматизация обработки заказов и клиентской поддержки",
  "recommendedFeatures": [
    {
      "feature": "AI Чат-бот для Telegram",
      "description": "Автоматическая обработка заказов через Telegram",
      "benefit": "Экономия 15 часов в неделю",
      "implementation": "Интеграция с вашей CRM системой"
    }
  ],
  "generatedPrompts": [
    {
      "type": "telegram_bot",
      "prompt": "Ты - помощник интернет-магазина...",
      "useCase": "Обработка заказов в Telegram"
    }
  ],
  "estimatedSavings": {
    "timePerMonth": "40 часов",
    "moneyPerMonth": "2000 долларов",
    "automationLevel": "high"
  }
}
```

## 🔧 API Эндпоинты

### Регистрация (`/api/registration`)

```javascript
POST /register
POST /verify-email
POST /update-phone
GET  /status/:userId
POST /resend-verification
POST /login
```

### AI Звонки (`/api/ai-calls`)

```javascript
POST /start        // Начать звонок
POST /message      // Отправить сообщение
POST /end          // Завершить звонок
GET  /active       // Активные звонки
GET  /history/:userId
GET  /details/:callSessionId
GET  /stats
POST /simulate     // Тестирование
```

### Админ Панель (`/api/admin`)

```javascript
GET  /stats                    // Статистика системы
GET  /numbers                  // Номера компании
POST /numbers/assign-api-key   // Назначить API ключ
GET  /modems                   // Конфигурация модемов
POST /modems                   // Обновить модем
GET  /users                    // Все пользователи
GET  /users/:userId            // Детали пользователя
GET  /analyses                 // Все анализы
GET  /savings                  // Статистика экономии
GET  /export                   // Экспорт данных
```

## 🚀 Запуск и Тестирование

### 1. Установка Зависимостей

```bash
cd backend
npm install nodemailer @google/generative-ai
```

### 2. Обновление Базы Данных

```bash
npx prisma generate
npx prisma db push
```

### 3. Создание Тестовых Данных

```bash
node src/scripts/seedRegistrationData.js
```

### 4. Настройка Environment

```env
# Email Service
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password

# Gemini API (по умолчанию)
DEFAULT_GEMINI_API_KEY=your-gemini-api-key

# Frontend URL
FRONTEND_URL=https://work-2-ccybwhtepolsidzo.prod-runtime.all-hands.dev
```

## 📊 Мониторинг и Аналитика

### Метрики Системы

- **Общее количество пользователей**
- **Процент верификации email**
- **Активные пользователи**
- **Статистика по этапам регистрации**
- **Количество завершенных звонков**
- **Средняя продолжительность звонков**
- **Процент успешных анализов**

### Экспорт Данных

- **JSON** формат для интеграций
- **CSV** формат для анализа
- **Фильтрация** по типам данных
- **Автоматические отчеты**

## 🔐 Безопасность

- **JWT токены** для аутентификации
- **Хеширование паролей** с bcrypt
- **Rate limiting** для API
- **Валидация входных данных**
- **Защищенные маршруты**

## 🎨 Интеграция с Frontend

Система готова для интеграции с React frontend:

- **Компоненты регистрации**
- **Интерфейс AI звонков**
- **Админ панель**
- **Дашборд аналитики**

## 📈 Масштабирование

- **Горизонтальное масштабирование** AI звонков
- **Балансировка нагрузки** между модемами
- **Кеширование** частых запросов
- **Асинхронная обработка** анализов

---

## 🎯 Следующие Шаги

1. **Создание Frontend компонентов** для регистрации
2. **Интеграция с реальными Gemini API ключами**
3. **Настройка email сервиса**
4. **Тестирование полного цикла**
5. **Развертывание в продакшн**

Система готова к использованию и дальнейшему развитию! 🚀