# 🚀 МЕГА ДЕТАЛЬНА ІНСТРУКЦІЯ РОЗГОРТАННЯ НА AWS

---

## 📊 ЗАГАЛЬНА СХЕМА

```
1. AWS Console → Створити EC2 інстанс
2. SSH підключення до серверу
3. Установка Docker і Docker Compose
4. Git clone вашого проекту
5. Запуск контейнерів
6. Налаштування доменів (опціонально)
7. HTTPS/SSL сертифікат (опціонально)
```

---

# 🌐 КРОК 1: СТВОРЕННЯ AWS АКАУНТУ

## 1.1 Якщо у вас ще немає AWS акаунту:

```
1. Зайдіть на https://aws.amazon.com
2. Натисніть "Create an AWS Account"
3. Виберіть "Personal" або "Business"
4. Вкажіть email, пароль, AWS Account Name
5. Додайте спосіб оплати (буде лише для верифікації)
6. Виберіть підтримку: "Basic Support (Free)"
7. Завершіть реєстрацію
8. Перевірте email і активуйте акаунт
```

**Вартість:** 0 грн! (12 місяців Free Tier)

---

# 🖥️ КРОК 2: СТВОРЕННЯ EC2 ІНСТАНСУ

## 2.1 Увійдіть в AWS Console

```
1. Зайдіть на https://console.aws.amazon.com
2. Виберіть регіон: N. Virginia (us-east-1) - найдешевший
3. У пошуку введіть "EC2"
4. Натисніть "EC2"
```

## 2.2 Запуск нового інстансу

```
1. На лівій панелі: Instances → Instances
2. Натисніть "Launch instances" (помаранчева кнопка)
```

## 2.3 Налаштування инстансу

### Назва та вибір ОС:
```
Name: devops-map-server
OS: Amazon Linux 2 (Free tier eligible)
Architecture: 64-bit (x86)
```

### Тип інстансу:
```
Instance Type: t2.micro (Free tier eligible)
```

### Keypair (дуже важливо!):
```
1. Create new key pair
2. Key pair name: devops-map-key
3. Key pair type: RSA
4. Private key file format: .pem
5. Натисніть "Create key pair"
6. ЗБЕРІЖІТЬ ФАЙЛ В БЕЗПЕЧНОМУ МІСЦІ! 
   (Приклад: C:\Users\YourName\AWS\devops-map-key.pem)
```

### Network settings:
```
VPC: (залиште за замовчуванням)
Subnet: (залиште за замовчуванням)
Auto-assign public IP: Enable
Firewall (security group): Create security group
```

### Security Group налаштування:

**Назва:** `devops-map-sg`

**Правила дозволу (Inbound rules):**

```
╔════════════════════════════════════════════════════════════════╗
║ Type          │ Protocol │ Port  │ Source      │ Description  ║
╠════════════════════════════════════════════════════════════════╣
║ SSH           │ TCP      │ 22    │ 0.0.0.0/0   │ SSH Access   ║
║ HTTP          │ TCP      │ 80    │ 0.0.0.0/0   │ Web App      ║
║ HTTPS         │ TCP      │ 443   │ 0.0.0.0/0   │ Secure Web   ║
║ Custom TCP    │ TCP      │ 8080  │ 0.0.0.0/0   │ API Backend  ║
║ Custom TCP    │ TCP      │ 6379  │ 0.0.0.0/0   │ Redis (опц)  ║
╚════════════════════════════════════════════════════════════════╝
```

Якщо ви не бачите поле для додавання правил:
```
1. Натисніть "Add security group rule"
2. Тип: SSH, Порт: 22, Джерело: Anywhere (0.0.0.0/0)
3. Натисніть "Add security group rule"
4. Тип: HTTP, Порт: 80, Джерело: Anywhere (0.0.0.0/0)
5. Натисніть "Add security group rule"
6. Тип: Custom TCP, Порт: 8080, Джерело: Anywhere (0.0.0.0/0)
```

### Storage:
```
Root volume size: 30 GB (залиште за замовчуванням)
Volume type: gp3
Delete on termination: checked
```

### Запуск:
```
Натисніть "Launch instance"
Чекайте 1-2 хвилини, поки інстанс стартує
```

---

## 2.4 Отримання публічної IP адреси

```
1. Перейдіть на вкладку "Instances"
2. Виберіть ваш новий інстанс (devops-map-server)
3. У деталях знайдіть "Public IPv4 address"
   Приклад: 54.123.45.67
4. ЗАПАМ'ЯТАЙТЕ ЦЮ IP!
```

---

# 🔑 КРОК 3: SSH ПІДКЛЮЧЕННЯ

## 3.1 Підготовка ключа (Windows)

### Якщо у вас Windows з PowerShell:

```powershell
# Переходимо в папку з ключем
cd C:\Users\YourName\AWS

# Правим дозволи на файл ключа (обов'язково!)
icacls "devops-map-key.pem" /inheritance:r
icacls "devops-map-key.pem" /grant:r "$env:username:(F)"

# Перевіряємо що все OK
ls -la devops-map-key.pem
```

### Якщо у вас Windows Subsystem for Linux (WSL):

```bash
# Переходимо в папку з ключем
cd /mnt/c/Users/YourName/AWS

# Правим дозволи
chmod 400 devops-map-key.pem

# Перевіряємо
ls -la devops-map-key.pem
```

### Якщо у вас macOS або Linux:

```bash
# Переходимо в папку з ключем
cd ~/AWS

# Правим дозволи
chmod 400 devops-map-key.pem

# Перевіряємо
ls -la devops-map-key.pem
```

## 3.2 SSH підключення

### За допомогою PowerShell (Windows):

```powershell
# Замініть YOUR_IP на вашу IP адресу з AWS
ssh -i C:\Users\YourName\AWS\devops-map-key.pem ec2-user@54.123.45.67

# Приклад:
ssh -i C:\Users\Lenovo\AWS\devops-map-key.pem ec2-user@54.123.45.67
```

### За допомогою WSL або Terminal (macOS/Linux):

```bash
ssh -i ~/AWS/devops-map-key.pem ec2-user@54.123.45.67
```

## 3.3 Перша залогінювання

Коли ви вперше логінюєтесь:
```
The authenticity of host '54.123.45.67 (54.123.45.67)' can't be established.
ECDSA key fingerprint is SHA256:xxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no)?
```

Напишіть: `yes` і натисніть Enter

```
Welcome to Amazon Linux 2
[ec2-user@ip-xxx ~]$
```

✅ **Ви в серверові!**

---

# 🐳 КРОК 4: УСТАНОВКА DOCKER

## 4.1 Оновлення системи

```bash
# Виконайте ці команди по одній
sudo yum update -y
```

Чекайте 2-5 хвилин, поки системні пакети оновлюються.

## 4.2 Установка Docker

```bash
# Встановлення Docker
sudo yum install docker -y

# Додавання вашого користувача в docker групу
sudo usermod -aG docker ec2-user

# Запуск Docker демона
sudo systemctl start docker

# Перевірка, що Docker запущений
sudo systemctl status docker
```

Повинно вивести:
```
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: active (running) since ...
```

Натисніть `q` для виходу.

## 4.3 Увімкнення автозапуску Docker

```bash
# Docker буде автоматично стартувати при перезавантаженні
sudo systemctl enable docker
```

## 4.4 Перевірка Docker

```bash
# Вийміть з поточної сесії та зайдіть знову
exit
```

Потім знову залогініться:
```bash
ssh -i C:\Users\YourName\AWS\devops-map-key.pem ec2-user@54.123.45.67
```

Тепер перевірте Docker без `sudo`:
```bash
docker --version
# Повинно вивести: Docker version 20.10.21, build ...

docker run hello-world
# Повинно вивести: Hello from Docker!
```

---

# 🐳 КРОК 5: УСТАНОВКА DOCKER COMPOSE

## 5.1 Завантаження Docker Compose

```bash
# Завантажуємо версію 2.20.0 (або найновішу)
sudo curl -L "https://github.com/docker/compose/releases/download/v2.20.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Видаємо дозволи на виконання
sudo chmod +x /usr/local/bin/docker-compose

# Перевіряємо установку
docker-compose --version
# Повинно вивести: Docker Compose version 2.20.0, build ...
```

---

# 📥 КРОК 6: ЗАВАНТАЖЕННЯ ВАШОГО ПРОЕКТУ

## 6.1 Установка Git

```bash
sudo yum install git -y

# Перевірка
git --version
```

## 6.2 Клонування вашого репозиторію

### Якщо ваш репозиторій публічний:

```bash
# Переходимо в home директорію
cd ~

# Клонуємо проект (замініть YOUR_REPO_URL на вашу URL)
git clone https://github.com/MkEger/LAB4_devoops.git

# Переходимо в директорію проекту
cd LAB4_devoops

# Перевіряємо, що файли там
ls -la
```

Повинні бачити:
```
docker-compose.yml
.dockerignore
.env
src/
AWS_DEPLOYMENT_DETAILED.md
DOCKER_CHANGES.md
```

### Якщо ваш репозиторій приватний (Потребує SSH ключа):

```bash
# Генеруємо SSH ключ для GitHub
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""

# Виводимо публічний ключ
cat ~/.ssh/id_rsa.pub
```

Потім:
```
1. Зайдіть на GitHub → Settings → SSH and GPG keys
2. Натисніть "New SSH key"
3. Назва: "AWS EC2 Server"
4. Ключ: Вставте текст з cat ~/.ssh/id_rsa.pub
5. Натисніть "Add SSH key"
6. Повернітесь на сервер
```

```bash
# Налаштування Git для SSH
git config --global url."git@github.com:".insteadOf "https://github.com/"

# Клонування
cd ~
git clone git@github.com:MkEger/LAB4_devoops.git
cd LAB4_devoops
```

---

# 🚀 КРОК 7: НАЛАШТУВАННЯ ФАЙЛІВ

## 7.1 Перевірка файлів

```bash
# Ви маєте бути в папці проекту
pwd
# Повинно вивести: /home/ec2-user/LAB4_devoops

# Перевіряємо наявність docker-compose.yml
ls -la docker-compose.yml
```

## 7.2 Редагування .env файлу (якщо потрібно)

```bash
# Перегляд вмісту .env
cat .env

# Редагування (якщо потрібно)
nano .env
```

Помощь в nano:
```
- Редагуйте текст звичайно
- Ctrl+X для виходу
- Y для збереження
- Enter для підтвердження імені файлу
```

Для AWS вам може знадобитися змінити:
```env
# Якщо у вас є власний домен
REACT_APP_API_URL=https://api.your-domain.com

# Або просто залиште для IP доступу
REACT_APP_API_URL=http://54.123.45.67:8080
```

---

# 🐳 КРОК 8: ЗАПУСК DOCKER CONTAINERS

## 8.1 Побудова образів

```bash
# Переконайтесь, що ви в папці проекту
pwd
# Повинно бути: /home/ec2-user/LAB4_devoops

# Побудова образів (перший раз буде довго!)
docker-compose build

# Це займе 10-20 хвилин
# Ви побачите багато текста - це нормально
```

Коли закінчиться, повинна бути успішна інформація:
```
[+] Building 234.5s (15/15) FINISHED
```

## 8.2 Запуск контейнерів

```bash
# Запуск всіх контейнерів у фоновому режимі
docker-compose up -d

# Трохи чекайте...
```

Повинно вивести:
```
[+] Running 4/4
 ✔ Network devops-map-project_app-network  Created
 ✔ Container redis-cache  Created
 ✔ Container communication-api  Started
 ✔ Container map-client-nginx  Started
```

## 8.3 Перевірка статусу контейнерів

```bash
# Показує статус всіх контейнерів
docker-compose ps

# Повинно показувати:
# NAME              COMMAND             STATUS          PORTS
# redis-cache      redis-server        Up 20s          0.0.0.0:6379->6379/tcp
# communication-api   dotnet             Up 15s          0.0.0.0:8080->8080/tcp
# map-client-nginx    nginx              Up 10s          0.0.0.0:80->80/tcp
```

---

# 📋 КРОК 9: ЛОГУВАННЯ ТА ТЕСТУВАННЯ

## 9.1 Перевірка логів

```bash
# Логи всіх контейнерів
docker-compose logs

# Логи конкретного контейнера (Redis)
docker-compose logs redis-cache

# Логи API
docker-compose logs communication-api

# Логи Frontend
docker-compose logs map-client-nginx

# Живі логи (натисніть Ctrl+C для вихода)
docker-compose logs -f map-client-nginx
```

## 9.2 Тестування доступу

### Тестування Frontend (React):

Відкрийте в браузері:
```
http://54.123.45.67
```

Повинні бачити вашу React карту.

### Тестування API:

```bash
# З вашої локальної машини - відкрийте PowerShell
curl http://54.123.45.67:8080/health

# Або замість curl можна в браузері
http://54.123.45.67:8080/health
```

### Тестування Redis:

```bash
# З сервера
docker-compose exec redis-cache redis-cli ping
# Повинно вивести: PONG

# Перевірка кількості ключів
docker-compose exec redis-cache redis-cli dbsize
```

### Тестування з вашої машини (PowerShell):

```powershell
# Тест Frontend
$response = Invoke-WebRequest -Uri "http://54.123.45.67"
$response.StatusCode
# Повинно: 200

# Тест API
$response = Invoke-WebRequest -Uri "http://54.123.45.67:8080/health"
$response.Content
```

---

# 📊 КРОК 10: МОНІТОРИНГ ТА УТРИМАННЯ

## 10.1 Перегляд використання ресурсів

```bash
# Статистика контейнерів (оновлюється в реальному часі)
docker stats

# Приклад виводу:
# CONTAINER ID    NAME               CPU %   MEM USAGE / LIMIT
# abc123...       map-client-nginx   0.1%    45MB / 1.9GB
# def456...       communication-api  0.2%    120MB / 1.9GB
# ghi789...       redis-cache        0.0%    5MB / 1.9GB
```

## 10.2 Автоматичне перезавантаження контейнерів

Контейнери налаштовані на `restart: unless-stopped`, це означає, що вони автоматично перезавантажуються при падінні.

```bash
# Перевірка що вони запущені
docker-compose ps

# Перезавантаження всіх контейнерів
docker-compose restart

# Перезавантаження конкретного контейнера
docker-compose restart communication-api
```

## 10.3 Перегляд дискового простору

```bash
# Вся інформація про диск
df -h

# Приклад:
# Filesystem       Size  Used Avail Use%
# /dev/xvda1       30G   3.2G   26.8G  11%

# Місце займане Docker
docker system df

# Очистка невикористовуваних образів
docker system prune -a -f
```

---

# 🔄 КРОК 11: ОНОВЛЕННЯ КОДУ

## 11.1 Коли ви хочете оновити код:

```bash
# Переходимо в папку проекту
cd ~/LAB4_devoops

# Завантажуємо останні зміни
git pull origin main

# Перебудовуємо образи
docker-compose build

# Перезапускаємо контейнери
docker-compose up -d

# Перевіряємо
docker-compose ps
```

---

# 🔐 КРОК 12: НАЛАШТУВАННЯ ДОМЕННОЇ ІМЕНІ (ОПЦІОНАЛЬНО)

## 12.1 Якщо у вас є свій домен:

### На сайті реєстратора домену (GoDaddy, NameCheap тощо):

```
1. Перейдіть в DNS налаштування
2. Додайте A record:
   - Name: @ або www
   - Type: A
   - Value: 54.123.45.67 (ваша AWS IP)
   - TTL: 3600
3. Натисніть Save
4. Чекайте 5-30 хвилин для розповсюджування DNS
```

### Перевірка:

```bash
# З вашої локальної машини
nslookup your-domain.com
# Повинно вивести вашу IP

# Тестування
curl http://your-domain.com
```

---

# 🔐 КРОК 13: HTTPS/SSL СЕРТИФІКАТ (РЕКОМЕНДУЄТЬСЯ)

## 13.1 Установка Certbot

```bash
# На AWS сервері
ssh -i C:\Users\YourName\AWS\devops-map-key.pem ec2-user@54.123.45.67

# Установка
sudo yum install certbot python3-certbot-nginx -y

# Отримання сертифіката (замініть на свій домен)
sudo certbot certonly --standalone -d your-domain.com -d www.your-domain.com --non-interactive --agree-tos --email your-email@example.com
```

## 13.2 Оновлення nginx.conf

```bash
# Редагування nginx.conf
sudo nano src/MapClient/nginx.conf
```

Додайте до конфіга:
```nginx
server {
    listen 443 ssl http2;
    server_name your-domain.com www.your-domain.com;

    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;

    # ... решта конфіга
}

# Редирект з HTTP на HTTPS
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;
    return 301 https://$server_name$request_uri;
}
```

## 13.3 Перестройка контейнера

```bash
# Скопіюємо сертифікати в проект (опціонально)
docker-compose down
docker-compose build
docker-compose up -d
```

---

# 🆘 КРОК 14: РОЗВ'ЯЗУВАННЯ ПРОБЛЕМ

## 14.1 Контейнер не запускається

```bash
# Перевіріть логи
docker-compose logs communication-api

# Перестартуйте контейнер
docker-compose restart communication-api

# Очистіть і перебудуйте
docker-compose down
docker-compose build
docker-compose up -d
```

## 14.2 Port вже використовується

```bash
# Знайдіть який контейнер використовує порт 80
docker-compose ps

# Перевіріть, чи немає іншого nginx на машині
sudo netstat -tlnp | grep 80

# Або використайте інший порт в docker-compose.yml
# Змініть: "80:80" на "8000:80"
```

## 14.3 Мало місця на диску

```bash
# Перевірте місце
df -h

# Видаліть старі образи
docker image prune -a -f

# Видаліть неживі контейнери
docker container prune -f

# Видаліть неживі volumes
docker volume prune -f
```

## 14.4 Не можу підключитися SSH

```bash
# Перевірте дозволи ключа
ls -la ~/AWS/devops-map-key.pem
# Повинно бути: -r-------- 1 ...

# Перевірте IP адресу
# AWS Console → EC2 → Instances → (виберіть інстанс)
```

## 14.5 Redis не підключається

```bash
# Перевіріть що Redis запущений
docker-compose ps redis-cache

# Тест підключення
docker-compose exec redis-cache redis-cli ping
# Повинно: PONG

# Якщо не працює:
docker-compose restart redis-cache
```

---

# 📞 КОРИСНІ КОМАНДИ

## Моніторинг

```bash
# Загальна інформація про контейнери
docker ps -a

# Всі образи
docker images

# Всі networks
docker network ls

# Всі volumes
docker volume ls

# Статус Docker демона
sudo systemctl status docker
```

## Управління

```bash
# Зупинити контейнери
docker-compose stop

# Видалити контейнери
docker-compose down

# Видалити контейнери і volumes
docker-compose down -v

# Перезапустити сервіс
docker-compose restart

# Обновити образи
docker-compose pull
docker-compose build
docker-compose up -d
```

## Очистка

```bash
# Видалити невикористовувані образи
docker image prune -a -f

# Видалити невикористовувані volumes
docker volume prune -f

# Видалити все (будьте обережні!)
docker system prune -a -f
```

---

# 📝 ЧЕКЛИСТ РОЗГОРТАННЯ

```
□ 1. Створено AWS акаунт
□ 2. Створено EC2 інстанс (t2.micro)
□ 3. Завантажено SSH ключ (devops-map-key.pem)
□ 4. Налаштовано Security Group (порти 22, 80, 8080)
□ 5. SSH підключення працює
□ 6. Docker установлений
□ 7. Docker Compose установлений
□ 8. Git проект клонований
□ 9. docker-compose build виконаний
□ 10. docker-compose up -d запущений
□ 11. docker-compose ps показує 3 контейнери (running)
□ 12. Frontend доступний на http://IP
□ 13. API доступний на http://IP:8080
□ 14. Redis доступний на IP:6379
□ 15. Логи не показують помилок
```

---

# 🎉 УСПІХ!

Ваш додаток тепер запущений на AWS! 

**Доступи:**
- **Frontend:** http://54.123.45.67 (замініть на вашу IP)
- **API:** http://54.123.45.67:8080
- **SSH:** ssh -i ~/AWS/devops-map-key.pem ec2-user@54.123.45.67

---

# 📞 ЯКЩО ЩОСЬ НЕ ПРАЦЮЄ

Напишіть мені:
1. Вихід команди `docker-compose ps`
2. Вихід команди `docker-compose logs`
3. Яка саме проблема

Я допоможу розв'язати! 🚀
