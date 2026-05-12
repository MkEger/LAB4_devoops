# LAB4 DevOps - Docker & AWS Deployment 🚀

Повний проект з Docker контейнеризацією та розгортанням на AWS з детальними інструкціями.

---

## 📚 Документація

### 🐳 Docker Setup
- **[DOCKER_CHANGES.md](./DOCKER_CHANGES.md)** - Всі Docker налаштування та поправки
- **[docker-compose.yml](./docker-compose.yml)** - Конфігурація для локального запуску
- **[.dockerignore](./.dockerignore)** - Глобальні фільтри для Docker build

### ☁️ AWS Deployment
- **[AWS_DEPLOYMENT.md](./AWS_DEPLOYMENT.md)** - Короткий огляд AWS розгортання
- **[AWS_DEPLOYMENT_DETAILED.md](./AWS_DEPLOYMENT_DETAILED.md)** - 📌 **МЕГА ДЕТАЛЬНА ІНСТРУКЦІЯ** (почніть звідси!)

### 🔧 Конфігурація
- **[.env](./.env)** - Змінні оточення
- **[src/MapClient/nginx.conf](./src/MapClient/nginx.conf)** - Nginx конфіг для React routing
- **[src/MapClient/.dockerignore](./src/MapClient/.dockerignore)** - Frontend Docker фільтри
- **[src/CommunicationControl/.dockerignore](./src/CommunicationControl/.dockerignore)** - Backend Docker фільтри

### 🐋 Dockerfiles
- **[src/MapClient/Dockerfile](./src/MapClient/Dockerfile)** - React + Nginx (multi-stage build)
- **[src/CommunicationControl/Dockerfile](./src/CommunicationControl/Dockerfile)** - .NET 8 API

---

## 🚀 Швидкий Старт

### Локально

```bash
# Запуск всіх контейнерів
docker-compose up -d

# Перевірка статусу
docker-compose ps

# Логи в реальному часі
docker-compose logs -f
```

**Доступ:**
- 🌐 **Frontend:** http://localhost
- 📡 **API:** http://localhost:8080
- 🗄️ **Redis:** localhost:6379

### На AWS

👉 **Читайте [AWS_DEPLOYMENT_DETAILED.md](./AWS_DEPLOYMENT_DETAILED.md)** - там є ВСІ команди крок за кроком!

**Основні кроки:**
1. Створити AWS акаунт
2. Запустити EC2 інстанс (t2.micro)
3. SSH підключення
4. Установити Docker & Docker Compose
5. Git clone проекту
6. `docker-compose up -d`
7. Готово! ✅

---

## 📊 Архітектура

```
┌──────────────────────────────────────────────────────────────┐
│                    AWS EC2 Instance                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │         Docker Network: app-network (bridge)          │ │
│  ├────────────────────────────────────────────────────────┤ │
│  │                                                        │ │
│  │  ┌─────────────────────────────────────────┐          │ │
│  │  │ Nginx (Port 80/443)                     │          │ │
│  │  │ - React Frontend (MapClient)            │          │ │
│  │  │ - Container: map-client-nginx           │          │ │
│  │  │ - Status: Running                       │          │ │
│  │  └──────────────┬──────────────────────────┘          │ │
│  │                 │ /api/* → proxy                       │ │
│  │                 ↓                                      │ │
│  │  ┌─────────────────────────────────────────┐          │ │
│  │  │ .NET Core 8 (Port 8080)                 │          │ │
│  │  │ - Communication Control API             │          │ │
│  │  │ - Container: communication-api          │          │ │
│  │  │ - Health: /health                       │          │ │
│  │  └──────────────┬──────────────────────────┘          │ │
│  │                 │                                      │ │
│  │                 ↓                                      │ │
│  │  ┌─────────────────────────────────────────┐          │ │
│  │  │ Redis Cache (Port 6379)                 │          │ │
│  │  │ - Data: redis-data volume               │          │ │
│  │  │ - Container: redis-cache                │          │ │
│  │  │ - Health: redis-cli ping                │          │ │
│  │  └─────────────────────────────────────────┘          │ │
│  │                                                        │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Структура Проекту

```
LAB4_devoops/
│
├── 📄 docker-compose.yml           ← Оркестрація контейнерів
├── 📄 .dockerignore                ← Глобальні Docker фільтри
├── 📄 .env                         ← Змінні оточення
│
├── 📁 src/
│   │
│   ├── 📁 MapClient/               ← React Frontend
│   │   ├── 🐋 Dockerfile           ← Multi-stage build
│   │   ├── 📄 nginx.conf           ← Nginx конфіг
│   │   ├── 📄 .dockerignore        ← Node modules фільтр
│   │   ├── 📄 package.json
│   │   ├── 📄 vite.config.js
│   │   └── src/                    ← React код
│   │
│   └── 📁 CommunicationControl/    ← .NET Backend
│       ├── 🐋 Dockerfile           ← .NET build
│       ├── 📄 .dockerignore        ← bin/obj фільтр
│       ├── DevOpsProject/          ← Проект API
│       ├── DevOpsProject.CommunicationControl.Logic/
│       └── DevOpsProject.Shared/
│
├── 📚 README.md                    ← Цей файл
├── 📘 DOCKER_CHANGES.md            ← Що змінено в Docker
├── 📗 AWS_DEPLOYMENT.md            ← Короткий гайд AWS
└── 📕 AWS_DEPLOYMENT_DETAILED.md   ← 🌟 ПОВНИЙ ГАЙД AWS
```

---

## 📋 Компоненти

### 1️⃣ Frontend (React + Nginx)
| Параметр | Значення |
|----------|----------|
| **Порти** | 80 (HTTP), 443 (HTTPS) |
| **Framework** | React 18.3 + Vite |
| **Сервер** | Nginx Alpine |
| **Build** | Multi-stage (node:20 → nginx:alpine) |
| **Розмір образу** | ~50MB |
| **Health Check** | GET http://localhost/index.html |

### 2️⃣ Backend (.NET 8 API)
| Параметр | Значення |
|----------|----------|
| **Порт** | 8080 |
| **Framework** | ASP.NET Core 8 |
| **Runtime** | .NET 8 Alpine |
| **Build** | Multi-stage (sdk:8.0 → aspnet:8.0) |
| **Розмір образу** | ~200MB |
| **Health Check** | GET http://localhost:8080/health |

### 3️⃣ Cache (Redis)
| Параметр | Значення |
|----------|----------|
| **Порт** | 6379 |
| **Image** | redis:7-alpine |
| **Persistence** | redis-data volume |
| **Memory** | ~5-10MB |
| **Health Check** | redis-cli ping |

---

## ⚙️ Основні Команди

### Docker Compose

```bash
# ▶️  Запуск всіх сервісів
docker-compose up -d

# ⏸️  Зупинення
docker-compose stop

# ▶️  Перезапуск
docker-compose restart

# 🛑 Видалення контейнерів
docker-compose down

# 🗑️  Видалення контейнерів + volumes
docker-compose down -v

# 📋 Статус всіх контейнерів
docker-compose ps

# 📝 Логи
docker-compose logs              # Всі логи
docker-compose logs -f           # Live логи
docker-compose logs service-name # Конкретний сервіс
```

### Docker Утиліти

```bash
# 📊 Статистика
docker stats
docker system df

# 🧹 Очистка
docker system prune -a -f        # Видалити все невикористане
docker image prune -a -f         # Видалити старі образи
docker volume prune -f           # Видалити неживі volumes
```

---

## 🔧 Налаштування

### Environment Variables (.env)

```env
# Redis Configuration
REDIS_HOST=redis
REDIS_PORT=6379

# Backend API Configuration
ASPNETCORE_ENVIRONMENT=Production
ASPNETCORE_URLS=http://+:8080

# Frontend Configuration
REACT_APP_API_URL=http://communication-control:8080

# Docker Compose
COMPOSE_PROJECT_NAME=devops-map-project
```

### Security Group (AWS)

| Порт | Сервіс | Протокол | Джерело |
|------|--------|----------|---------|
| 22 | SSH | TCP | 0.0.0.0/0 |
| 80 | HTTP | TCP | 0.0.0.0/0 |
| 443 | HTTPS | TCP | 0.0.0.0/0 |
| 8080 | API | TCP | 0.0.0.0/0 |
| 6379 | Redis | TCP | 0.0.0.0/0 |

---

## 🆘 Розв'язування Проблем

### ❌ Контейнер не запускається
```bash
# Перевіримо логи
docker-compose logs service-name

# Перестартуємо
docker-compose restart service-name

# Перебудуємо
docker-compose down
docker-compose build
docker-compose up -d
```

### ❌ Port вже в використанні
```bash
# Знайти який процес використовує порт
sudo netstat -tlnp | grep :80

# Або змінити порт у docker-compose.yml
# Змініть: "80:80" на "8000:80"
```

### ❌ Мало місця на диску
```bash
# Перевірити місце
df -h

# Очистити все
docker system prune -a -f
```

### ❌ Redis не підключається
```bash
# Перевірити статус
docker-compose ps redis-cache

# Тест
docker-compose exec redis-cache redis-cli ping
# Повинно: PONG

# Перезапустити
docker-compose restart redis-cache
```

### ❌ Nginx повертає 502 Bad Gateway
```bash
# Перевірити що API запущено
docker-compose ps communication-api

# Перевірити логи nginx
docker-compose logs map-client-nginx

# Перевірити логи API
docker-compose logs communication-api
```

---

## 📞 Посилання

- 🌐 **GitHub:** https://github.com/MkEger/LAB4_devoops
- ☁️ **AWS Free Tier:** https://aws.amazon.com/free
- 🐳 **Docker Docs:** https://docs.docker.com
- 🐢 **Docker Compose:** https://docs.docker.com/compose
- ⚙️ **nginx:** https://nginx.org

---

## ✅ Чеклист Розгортання

### Локальне тестування
- [ ] Docker установлений
- [ ] Docker Compose установлений
- [ ] `docker-compose up -d` успішно
- [ ] `docker-compose ps` показує 3 контейнери (running)
- [ ] Frontend доступний на http://localhost
- [ ] API доступний на http://localhost:8080
- [ ] Redis доступний на localhost:6379
- [ ] Логи не показують помилок

### AWS Розгортання
- [ ] AWS акаунт створено
- [ ] EC2 інстанс запущений (t2.micro)
- [ ] SSH ключ налаштований
- [ ] SSH підключення працює
- [ ] Docker установлений на сервері
- [ ] Docker Compose установлений
- [ ] Проект клонований
- [ ] `docker-compose build` завершено
- [ ] `docker-compose up -d` запущено
- [ ] Всі контейнери running
- [ ] Frontend доступний на http://IP
- [ ] API доступний на http://IP:8080
- [ ] Логи не показують помилок

---

## 🎉 Успіх!

Ваш DevOps проект готовий до розгортання! 

### 👉 Почніть з цього:

**Для локального запуску:**
```bash
docker-compose up -d
```

**Для розгортання на AWS:**
👉 Читайте **[AWS_DEPLOYMENT_DETAILED.md](./AWS_DEPLOYMENT_DETAILED.md)** - там всі команди крок за кроком!

---

## 📞 Питання?

Дивіться документацію:
- 📘 [DOCKER_CHANGES.md](./DOCKER_CHANGES.md) - Docker деталі
- 📗 [AWS_DEPLOYMENT.md](./AWS_DEPLOYMENT.md) - AWS огляд
- 📕 [AWS_DEPLOYMENT_DETAILED.md](./AWS_DEPLOYMENT_DETAILED.md) - AWS повний гайд

💬 Або пишіть issue на GitHub!

---

**Last Updated:** 2024
**License:** University DevOps Course
#   T e s t   w o r k f l o w  
 