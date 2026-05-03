# AWS Deployment Guide

## 🚀 Розгортання на AWS (безкоштовний рівень)

### Варіант 1: AWS Elastic Container Service (ECS) з Fargate

```bash
# 1. Вхід в AWS
aws configure

# 2. Створення ECR репозиторіїв
aws ecr create-repository --repository-name map-client --region us-east-1
aws ecr create-repository --repository-name communication-api --region us-east-1

# 3. Побудова і пуш образів
AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
AWS_REGION=us-east-1
ECR_REGISTRY=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com

# Логін в ECR
aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REGISTRY

# Побудова
docker build -t $ECR_REGISTRY/map-client:latest ./src/MapClient
docker build -t $ECR_REGISTRY/communication-api:latest ./src/CommunicationControl

# Пуш
docker push $ECR_REGISTRY/map-client:latest
docker push $ECR_REGISTRY/communication-api:latest
```

### Варіант 2: AWS Elastic Beanstalk (найпростіший)

```bash
# 1. Установка EB CLI
pip install awsebcli

# 2. Ініціалізація проекту
eb init -p docker devops-map-project --region us-east-1

# 3. Створення Dockerrun.aws.json
cat > Dockerrun.aws.json << 'EOF'
{
  "AWSEBDockerrunVersion": 2,
  "containerDefinitions": [
    {
      "name": "redis",
      "image": "redis:7-alpine",
      "memory": 128,
      "portMappings": [
        {
          "hostPort": 6379,
          "containerPort": 6379
        }
      ]
    },
    {
      "name": "communication-api",
      "image": "YOUR_AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/communication-api:latest",
      "memory": 256,
      "portMappings": [
        {
          "hostPort": 8080,
          "containerPort": 8080
        }
      ],
      "environment": [
        {
          "name": "ASPNETCORE_ENVIRONMENT",
          "value": "Production"
        },
        {
          "name": "RedisHost",
          "value": "localhost:6379"
        }
      ],
      "links": ["redis"]
    },
    {
      "name": "map-client",
      "image": "YOUR_AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/map-client:latest",
      "memory": 256,
      "portMappings": [
        {
          "hostPort": 80,
          "containerPort": 80
        }
      ],
      "links": ["communication-api"]
    }
  ]
}
EOF

# 4. Розгортання
eb create devops-map-env
eb deploy

# 5. Відкриття додатку
eb open
```

### Варіант 3: EC2 + Docker Compose (мінімальна конфігурація)

```bash
# 1. SSH на EC2 інстанс
ssh -i your-key.pem ec2-user@your-instance-ip

# 2. Установка Docker
sudo yum update -y
sudo yum install docker -y
sudo usermod -aG docker ec2-user
sudo systemctl start docker

# 3. Установка Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.20.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# 4. Завантаження проекту
git clone YOUR_REPO_URL
cd YOUR_REPO

# 5. Запуск
docker-compose up -d

# 6. Перевірка статусу
docker-compose ps
docker logs map-client
docker logs communication-control
```

## 📊 Моніторинг на AWS

```bash
# CloudWatch Logs
aws logs describe-log-groups --region us-east-1

# Перевірка ECS tasks
aws ecs list-clusters --region us-east-1
aws ecs list-tasks --cluster YOUR_CLUSTER --region us-east-1

# Scaling
aws application-autoscaling register-scalable-target \
  --service-namespace ecs \
  --scalable-dimension ecs:service:DesiredCount \
  --min-capacity 1 \
  --max-capacity 3 \
  --resource-id service/default/devops-map-service
```

## 💰 Знижки для безкоштовного рівня AWS

- **EC2**: 750 годин t2.micro в місяць
- **ECR**: 500 MB місячно для хранення образів
- **CloudWatch**: 5 ГБ логів в місяць
- **RDS (опціонально)**: 750 годин db.t2.micro

## ⚙️ Рекомендовані налаштування для вашого проекту

**t2.micro інстанс достатньо для:**
- 1x React Frontend (MapClient)
- 1x .NET API (CommunicationControl)
- 1x Redis (кешування)

**Вартість: 0 грн під час пробного періоду!**
