# Начальная инициализация

## API Ключи

### DeepSeek API:
sk-84395b9ebca34bf78eb770c2e3d7cf36

text

### Google Gemini:
AIzaSyCRy7S4eyuEJVYl4uxwiyhBizI_9MGAyyY

text

### Google Serp API:
8c02e8532993970497e4baf5344ca9191bbf94a6

text

## Вебхуки для загрузки файлов

### Для теста:
```bash
curl -X POST \
  -F "file=@test-rag.txt" \
  https://kazoazazo.app.n8n.cloud/webhook-test/upload-knowledge-base
Для продакшена:
bash
curl -X POST -F "file=@test-rag.txt" https://kazoazazo.app.n8n.cloud/webhook/upload-knowledge-base
Настройка Google Serp API
В узле "HTTP Request" → Authentication:

Выбери "Header Auth" вместо "Bearer Auth"

Настрой:

Name: X-API-KEY

Value: 8c02e8532993970497e4baf5344ca9191bbf94a6 (просто ключ БЕЗ кавычек!)

📋 Документация по настройке лидогенерации с Redis и PostgreSQL
🎯 Обзор системы
Ваш workflow был улучшен добавлением:

Rate limiting - 25 контактов/день для WhatsApp, 50 писем/день для Email

PostgreSQL - постоянное хранение лидов и логов отправок

Redis - кэширование и контроль лимитов с автосбросом каждые 24 часа

🗄️ Настройка PostgreSQL (Supabase)
1. Создание проекта в Supabase
Перейдите на https://supabase.com

Нажмите "Start your project" → "New project"

Заполните:

Organization: выберите или создайте организацию

Name: любое имя проекта (например, n8n-leads)

Database Password: сохраните пароль!

Region: выберите ближайший к вам

2. Создание таблиц
В левом меню Supabase выберите SQL Editor → New query, выполните:

sql
-- Таблица лидов
CREATE TABLE leads (
  id BIGSERIAL PRIMARY KEY,
  name VARCHAR(255),
  address TEXT,
  phone VARCHAR(50),
  email VARCHAR(255),
  website VARCHAR(255),
  category VARCHAR(100),
  created_at TIMESTAMP DEFAULT NOW()
);

-- Таблица логов отправки
CREATE TABLE outreach_log (
  id BIGSERIAL PRIMARY KEY,
  lead_name VARCHAR(255),
  contact_type VARCHAR(20),
  contact_value VARCHAR(255),
  sent_at TIMESTAMP DEFAULT NOW(),
  status VARCHAR(50)
);

-- Индексы для оптимизации
CREATE INDEX idx_leads_phone ON leads(phone);
CREATE INDEX idx_leads_email ON leads(email);
CREATE INDEX idx_outreach_log_sent_at ON outreach_log(sent_at);
3. Получение данных подключения
Project Settings → Database → Connection string

Скопируйте строку в формате:

text
postgresql://postgres:[YOUR-PASSWORD]@db.xxxxxxxxxxxxx.supabase.co:5432/postgres
4. Настройка в n8n
Откройте любую PostgreSQL ноду в workflow

Credentials → Create New → PostgreSQL

Заполните:

Host: db.xxxxxxxxxxxxx.supabase.co

Database: postgres

User: postgres

Password: ваш пароль от Supabase

Port: 5432

SSL: включите ✅

🔴 Настройка Redis (Upstash)
1. Создание базы данных
Перейдите на https://upstash.com

Войдите через GitHub или email

Create Database → заполните:

Name: n8n-rate-limiting

Type: Regional

Region: ближайший к вам

TLS: включено ✅

2. Получение ключей подключения
После создания базы в разделе REST API найдите:

UPSTASH_REDIS_REST_URL (например: https://us1-xxxxx.upstash.io)

UPSTASH_REDIS_REST_TOKEN (длинный токен)

3. Настройка в n8n
Откройте любую Redis ноду в workflow

Credentials → Create New → Redis

Заполните:

Host: часть URL после https:// (например us1-xxxxx.upstash.io)

Port: 6379

Password: вставьте UPSTASH_REDIS_REST_TOKEN

Database Number: 0

SSL: включите ✅

⚙️ Как работают лимиты
Redis ключи для rate limiting:
whatsapp_daily_count:2025-01-15 - счетчик WhatsApp (TTL 24h)

email_daily_count:2025-01-15 - счетчик Email (TTL 24h)

Логика работы:
Перед отправкой - проверка текущего счетчика в Redis

Если лимит не превышен - отправка + инкремент счетчика

Если лимит превышен - пропуск отправки

Автосброс - через 24 часа TTL автоматически удаляет ключи

📊 Структура данных
Таблица leads:
Поле	Тип	Описание
id	BIGSERIAL	Автоинкремент
name	VARCHAR(255)	Название компании
address	TEXT	Полный адрес
phone	VARCHAR(50)	Телефон
email	VARCHAR(255)	Email
website	VARCHAR(255)	Сайт
category	VARCHAR(100)	Категория бизнеса
created_at	TIMESTAMP	Дата создания
Таблица outreach_log:
Поле	Тип	Описание
id	BIGSERIAL	Автоинкремент
lead_name	VARCHAR(255)	Название лида
contact_type	VARCHAR(20)	WhatsApp/Email
contact_value	VARCHAR(255)	Телефон/email
sent_at	TIMESTAMP	Время отправки
status	VARCHAR(50)	Статус отправки
🚀 Проверка работоспособности
Тест PostgreSQL:
Откройте ноду "Сохранение лида"

Нажмите Test connection - должно быть ✅ успешно

Проверьте в Supabase: Table Editor → данные должны появляться

Тест Redis:
Откройте ноду "Проверка лимита WhatsApp"

Выполните тестовый запрос - должен вернуть текущее значение счетчика

💡 Полезные запросы для аналитики
sql
-- Статистика отправок за сегодня
SELECT contact_type, COUNT(*) as sent_count
FROM outreach_log 
WHERE sent_at >= CURRENT_DATE
GROUP BY contact_type;

-- Топ категорий лидов
SELECT category, COUNT(*) as lead_count
FROM leads 
GROUP BY category 
ORDER BY lead_count DESC;

-- Лимиты на сегодня
SELECT 
  (SELECT COUNT(*) FROM outreach_log WHERE contact_type = 'whatsapp' AND sent_at >= CURRENT_DATE) as whatsapp_today,
  (SELECT COUNT(*) FROM outreach_log WHERE contact_type = 'email' AND sent_at >= CURRENT_DATE) as email_today;
🔧 АКТУАЛЬНЫЕ ДАННЫЕ ПОДКЛЮЧЕНИЯ
PostgreSQL (Supabase) - РАБОЧИЕ НАСТРОЙКИ
text
Host: aws-1-eu-central-1.pooler.supabase.com
Database: postgres
User: postgres.mhataheibhhpmyuusxtj
Password: 2304730qQ
Port: 5432
SSL: Require
Ignore SSL Issues: ✅ Включено (Insecure)
Важно: Используйте Session Pooler вместо Direct Connection для обхода проблем с IPv6!

Redis (Upstash) - РАБОЧИЕ НАСТРОЙКИ
text
Host: quality-zebra-36238.upstash.io
Port: 6379
Password: AY2OAAIncDJlZjdjNThmYTg4ZDM0NDg4OTQ4MWM2ZTdlYTkyMTNhZnAyMzYyMzg
Database Number: 0
SSL: Require
Disable TLS Verification: ✅ Включено (Insecure)
⚠️ РЕШЕНИЕ ТИПОВЫХ ПРОБЛЕМ
Ошибка IPv6 в PostgreSQL:
Используйте Session Pooler вместо Direct Connection

Host: aws-1-eu-central-1.pooler.supabase.com

Port: 5432

Ошибка SSL сертификата:
В PostgreSQL: Ignore SSL Issues → ✅ Включено

В Redis: Disable TLS Verification → ✅ Включено

Проверка подключений:
PostgreSQL: Test connection в любой PostgreSQL ноде

Redis: Проверьте через команды в Upstash консоли:

bash
GET whatsapp_daily_count:2025-01-15
GET email_daily_count:2025-01-15
✅ ЧТО ДОЛЖНО РАБОТАТЬ ПОСЛЕ НАСТРОЙКИ
Лимиты WhatsApp: 25/день

Лимиты Email: 50/день

Данные в Supabase таблицах leads и outreach_log

Автоматический сброс счетчиков каждые 24 часа

Весь предыдущий функционал остается рабочим

Система полностью настроена и готова к использованию! 🎯