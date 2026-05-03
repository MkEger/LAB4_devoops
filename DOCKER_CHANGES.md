# Docker Configuration Summary

## ✅ Виконані поправки

### 1. **docker-compose.yml** (Оновлено)
   - ✔️ Додано versioning (3.8)
   - ✔️ Redis з health checks
   - ✔️ Named volumes для Redis persistence
   - ✔️ Named networks (app-network)
   - ✔️ depends_on з condition: service_healthy
   - ✔️ Restart policies (unless-stopped)
   - ✔️ Environment variables
   - ✔️ Container names для зручного доступу

### 2. **.dockerignore** (Заповнено)
   - Фільтрує непотрібні файли при контекстному копіюванні

### 3. **src/MapClient/Dockerfile** (Оновлено)
   - ✔️ Додано nginx.conf для правильної маршрутизації
   - ✔️ Health checks
   - ✔️ Оптимізована багатостадійна збірка

### 4. **src/MapClient/nginx.conf** (Новий)
   - ✔️ Правильна маршрутизація для React Router
   - ✔️ API proxy до backend
   - ✔️ Кешування статичних файлів
   - ✔️ Security headers

### 5. **src/MapClient/.dockerignore** (Новий)
   - Фільтрує ненужні файли Node.js

### 6. **src/CommunicationControl/Dockerfile** (Оновлено)
   - ✔️ Health checks добавлено

### 7. **.env** (Новий)
   - Централізовані змінні оточення

### 8. **AWS_DEPLOYMENT.md** (Новий)
   - 3 варіанти развертання на AWS
   - Команди для ECR, ECS, Elastic Beanstalk, EC2

---

## 🚀 Швидкий Старт Локально

```bash
# Почувства, які потрібні:
docker --version  # Docker Desktop
docker-compose --version

# Запуск
docker-compose up -d

# Перевірка
docker-compose ps
docker-compose logs -f map-client
docker-compose logs -f communication-control

# Доступ
# Frontend: http://localhost
# API: http://localhost:8080
# Redis: localhost:6379
```

---

## 🔍 Структура мережі (Network)

```
┌─────────────────────────────────────┐
│         app-network (bridge)        │
├─────────────────────────────────────┤
│  ┌──────────────┐                   │
│  │   nginx:80   │ (map-client)      │
│  └──────┬───────┘                   │
│         │                           │
│         ↓                           │
│  ┌──────────────┐                   │
│  │  .NET:8080   │ (communication)   │
│  └──────┬───────┘                   │
│         │                           │
│         ↓                           │
│  ┌──────────────┐                   │
│  │ Redis:6379   │ (cache)           │
│  └──────────────┘                   │
└─────────────────────────────────────┘
```

---

## 💾 Volumes (Persistence)

- **redis-data**: Зберігає Redis дані між перезавантаженнями
- **./logs**: Логи .NET API на хост-машині

---

## 🔗 Залежності (Dependencies)

```
map-client → communication-control → redis
```

1. **redis** стартує першим
2. Потім **communication-control** чекає redis (healthcheck)
3. Потім **map-client** чекає communication-control

---

## 🛑 Зупинення

```bash
docker-compose down              # Видаляє контейнери, але зберігає volumes
docker-compose down -v           # Видаляє контейнери і volumes
```

---

## 📝 Наступні кроки для AWS

1. Створити AWS EC2 інстанс (t2.micro)
2. SSH на інстанс
3. Установити Docker і Docker Compose
4. Git clone проекту
5. Запустити: `docker-compose up -d`
6. Настроїти Security Group для портів 80, 8080
7. Отримати публічну IP і відкрити у браузері

Детально в **AWS_DEPLOYMENT.md**
