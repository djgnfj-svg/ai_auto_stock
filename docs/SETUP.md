# 환경 설정 및 배포 가이드

## 1. 로컬 개발 환경 설정

### 1.1 시스템 요구사항

**최소 시스템 요구사항**
- CPU: 4 코어 이상
- RAM: 8GB 이상 (16GB 권장)
- Storage: 50GB 이상의 여유 공간
- OS: macOS, Linux, Windows (WSL2)

**필수 소프트웨어**
- Python 3.11+
- Node.js 20+
- Docker & Docker Compose
- PostgreSQL 15 (선택적 - Docker 사용 시 불필요)
- Redis 7.2 (선택적 - Docker 사용 시 불필요)

### 1.2 환경 변수 설정

**환경 변수 파일 생성**
```bash
# 백엔드 환경 변수
cp backend/.env.example backend/.env

# 프론트엔드 환경 변수
cp frontend/.env.example frontend/.env
```

**backend/.env 파일 설정**
```bash
# Database
DATABASE_URL=postgresql://username:password@localhost:5432/ai_auto_stock
REDIS_URL=redis://localhost:6379/0

# Security
SECRET_KEY=your-super-secret-key-here
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# API Keys
ALPHA_VANTAGE_API_KEY=your_alpha_vantage_key
YAHOO_FINANCE_API_KEY=your_yahoo_finance_key
NEWS_API_KEY=your_news_api_key

# MLflow
MLFLOW_TRACKING_URI=http://localhost:5000

# Airflow
AIRFLOW__CORE__FERNET_KEY=your_fernet_key
AIRFLOW__CORE__SQL_ALCHEMY_CONN=postgresql://airflow:airflow@localhost:5432/airflow

# Logging
LOG_LEVEL=INFO

# Email (선택적)
SMTP_SERVER=smtp.gmail.com
SMTP_PORT=587
SMTP_USERNAME=your_email@gmail.com
SMTP_PASSWORD=your_app_password
```

**frontend/.env 파일 설정**
```bash
# API Endpoints
NEXT_PUBLIC_API_URL=http://localhost:8000
NEXT_PUBLIC_WS_URL=ws://localhost:8000

# Analytics (선택적)
NEXT_PUBLIC_GA_ID=your_google_analytics_id

# Feature Flags
NEXT_PUBLIC_ENABLE_CHAT=false
NEXT_PUBLIC_ENABLE_NOTIFICATIONS=true
```

### 1.3 Docker 개발 환경

**docker-compose.dev.yml 파일**
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: ai_auto_stock
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./scripts/init.sql:/docker-entrypoint-initdb.d/init.sql

  redis:
    image: redis:7.2-alpine
    ports:
      - "6379:6379"
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data

  airflow-webserver:
    image: apache/airflow:2.7.3
    environment:
      - AIRFLOW__CORE__EXECUTOR=LocalExecutor
      - AIRFLOW__CORE__SQL_ALCHEMY_CONN=postgresql+psycopg2://airflow:airflow@postgres:5432/airflow
      - AIRFLOW__CORE__FERNET_KEY=your_fernet_key
    ports:
      - "8080:8080"
    volumes:
      - ./airflow/dags:/opt/airflow/dags
      - ./airflow/logs:/opt/airflow/logs
      - ./airflow/plugins:/opt/airflow/plugins
    depends_on:
      - postgres
      - redis

  mlflow:
    image: python:3.11-slim
    command: >
      sh -c "pip install mlflow psycopg2-binary &&
             mlflow server --host 0.0.0.0 --port 5000 --backend-store-uri postgresql://mlflow:mlflow@postgres:5432/mlflow --default-artifact-root ./mlruns"
    ports:
      - "5000:5000"
    volumes:
      - mlflow_data:/mlflow
    depends_on:
      - postgres

volumes:
  postgres_data:
  redis_data:
  mlflow_data:
```

**개발 환경 실행**
```bash
# 1. Docker 서비스 시작
docker-compose -f docker/docker-compose.dev.yml up -d

# 2. 데이터베이스 초기화
cd backend
alembic upgrade head
python scripts/create_initial_data.py

# 3. 백엔드 서버 실행
poetry install
poetry shell
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# 4. 프론트엔드 서버 실행 (새 터미널)
cd frontend
npm install
npm run dev
```

### 1.4 API 키 설정 가이드

**Alpha Vantage API 키 발급**
1. [Alpha Vantage 웹사이트](https://www.alphavantage.co/) 방문
2. 무료 계정 생성
3. API 키 발급 (무료: 5 API calls/min, 500 calls/day)
4. `.env` 파일에 추가

**Yahoo Finance API 설정**
```python
# Yahoo Finance는 비공식 API이므로 RapidAPI 사용 권장
# RapidAPI에서 Yahoo Finance API 구독
YAHOO_FINANCE_API_KEY=your_rapidapi_key
YAHOO_FINANCE_API_HOST=yahoo-finance15.p.rapidapi.com
```

**News API 키 발급**
1. [NewsAPI.org](https://newsapi.org/) 방문
2. 계정 생성 및 API 키 발급
3. 무료 플랜: 1,000 requests/month
4. `.env` 파일에 추가

## 2. 프로덕션 배포

### 2.1 AWS 인프라 설정

**AWS 계정 및 IAM 설정**
```bash
# AWS CLI 설치 및 설정
aws configure
# Access Key ID: your_access_key
# Secret Access Key: your_secret_key
# Default region: ap-northeast-2 (서울)
# Default output format: json

# IAM 정책 예시 (최소 권한)
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecs:*",
        "rds:*",
        "elasticache:*",
        "s3:*",
        "cloudformation:*",
        "iam:PassRole"
      ],
      "Resource": "*"
    }
  ]
}
```

**Terraform 인프라스트럭처 코드**
```hcl
# infrastructure/main.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "ap-northeast-2"
}

# VPC 설정
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "ai-auto-stock-vpc"
  }
}

# RDS PostgreSQL
resource "aws_db_instance" "postgres" {
  identifier     = "ai-auto-stock-db"
  engine         = "postgres"
  engine_version = "15.4"
  instance_class = "db.t3.micro"
  
  allocated_storage     = 20
  max_allocated_storage = 100
  storage_encrypted     = true
  
  db_name  = "ai_auto_stock"
  username = "postgres"
  password = var.db_password
  
  vpc_security_group_ids = [aws_security_group.rds.id]
  db_subnet_group_name   = aws_db_subnet_group.main.name
  
  backup_retention_period = 7
  backup_window          = "03:00-04:00"
  maintenance_window     = "sun:04:00-sun:05:00"
  
  skip_final_snapshot = true
  
  tags = {
    Name = "ai-auto-stock-db"
  }
}

# ElastiCache Redis
resource "aws_elasticache_replication_group" "redis" {
  replication_group_id       = "ai-auto-stock-redis"
  description                = "Redis cache for AI Auto Stock"
  
  num_cache_clusters         = 2
  node_type                  = "cache.t3.micro"
  port                       = 6379
  parameter_group_name       = "default.redis7"
  
  subnet_group_name          = aws_elasticache_subnet_group.main.name
  security_group_ids         = [aws_security_group.redis.id]
  
  at_rest_encryption_enabled = true
  transit_encryption_enabled = true
  
  tags = {
    Name = "ai-auto-stock-redis"
  }
}

# ECS Cluster
resource "aws_ecs_cluster" "main" {
  name = "ai-auto-stock-cluster"
  
  setting {
    name  = "containerInsights"
    value = "enabled"
  }
  
  tags = {
    Name = "ai-auto-stock-cluster"
  }
}
```

### 2.2 Docker 이미지 빌드 및 배포

**Dockerfile.backend**
```dockerfile
FROM python:3.11-slim

WORKDIR /app

# 시스템 패키지 설치
RUN apt-get update && apt-get install -y \
    gcc \
    g++ \
    && rm -rf /var/lib/apt/lists/*

# Python 의존성 설치
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 애플리케이션 코드 복사
COPY . .

# 포트 노출
EXPOSE 8000

# 헬스체크 추가
HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1

# 애플리케이션 실행
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**Dockerfile.frontend**
```dockerfile
# 빌드 스테이지
FROM node:20-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

# 프로덕션 스테이지
FROM nginx:alpine

# Nginx 설정 파일 복사
COPY nginx.conf /etc/nginx/nginx.conf

# 빌드된 파일 복사
COPY --from=builder /app/build /usr/share/nginx/html

# 포트 노출
EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

**GitHub Actions CI/CD**
```yaml
# .github/workflows/deploy.yml
name: Deploy to AWS

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: 3.11
    
    - name: Install dependencies
      run: |
        cd backend
        pip install -r requirements.txt
    
    - name: Run tests
      run: |
        cd backend
        pytest
    
    - name: Set up Node.js
      uses: actions/setup-node@v3
      with:
        node-version: 20
    
    - name: Install and test frontend
      run: |
        cd frontend
        npm install
        npm run test

  build-and-deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ap-northeast-2
    
    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1
    
    - name: Build and push backend image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        ECR_REPOSITORY: ai-auto-stock-backend
        IMAGE_TAG: ${{ github.sha }}
      run: |
        cd backend
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
    
    - name: Build and push frontend image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        ECR_REPOSITORY: ai-auto-stock-frontend
        IMAGE_TAG: ${{ github.sha }}
      run: |
        cd frontend
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
    
    - name: Update ECS service
      run: |
        aws ecs update-service \
          --cluster ai-auto-stock-cluster \
          --service ai-auto-stock-backend \
          --force-new-deployment
        
        aws ecs update-service \
          --cluster ai-auto-stock-cluster \
          --service ai-auto-stock-frontend \
          --force-new-deployment
```

### 2.3 모니터링 및 로깅 설정

**CloudWatch 로그 그룹 생성**
```bash
# 백엔드 로그 그룹
aws logs create-log-group --log-group-name /ecs/ai-auto-stock-backend

# 프론트엔드 로그 그룹
aws logs create-log-group --log-group-name /ecs/ai-auto-stock-frontend

# Airflow 로그 그룹
aws logs create-log-group --log-group-name /ecs/ai-auto-stock-airflow
```

**Prometheus 설정**
```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'fastapi-app'
    static_configs:
      - targets: ['localhost:8000']
    metrics_path: '/metrics'
    scrape_interval: 5s

  - job_name: 'postgres-exporter'
    static_configs:
      - targets: ['localhost:9187']

  - job_name: 'redis-exporter'
    static_configs:
      - targets: ['localhost:9121']
```

**Grafana 대시보드 설정**
```json
{
  "dashboard": {
    "title": "AI Auto Stock Monitoring",
    "panels": [
      {
        "title": "API Response Time",
        "type": "graph",
        "targets": [
          {
            "expr": "http_request_duration_seconds_bucket",
            "legendFormat": "{{method}} {{endpoint}}"
          }
        ]
      },
      {
        "title": "Database Connections",
        "type": "stat",
        "targets": [
          {
            "expr": "pg_stat_database_numbackends",
            "legendFormat": "Active Connections"
          }
        ]
      }
    ]
  }
}
```

## 3. 보안 설정

### 3.1 SSL/TLS 인증서 설정

**AWS Certificate Manager 사용**
```bash
# SSL 인증서 요청
aws acm request-certificate \
  --domain-name yourdomain.com \
  --subject-alternative-names "*.yourdomain.com" \
  --validation-method DNS \
  --region ap-northeast-2
```

### 3.2 환경 변수 보안 관리

**AWS Secrets Manager 사용**
```bash
# 시크릿 생성
aws secretsmanager create-secret \
  --name "ai-auto-stock/prod" \
  --description "Production secrets for AI Auto Stock" \
  --secret-string '{
    "DATABASE_URL": "postgresql://user:pass@rds-endpoint:5432/db",
    "SECRET_KEY": "your-super-secret-key",
    "API_KEYS": {
      "ALPHA_VANTAGE": "your-api-key"
    }
  }'

# ECS 태스크 정의에서 시크릿 사용
{
  "secrets": [
    {
      "name": "DATABASE_URL",
      "valueFrom": "arn:aws:secretsmanager:region:account:secret:ai-auto-stock/prod:DATABASE_URL::"
    }
  ]
}
```

### 3.3 방화벽 및 보안 그룹 설정

**AWS 보안 그룹 규칙**
```bash
# 웹 서버 보안 그룹 (ALB용)
aws ec2 create-security-group \
  --group-name ai-auto-stock-alb \
  --description "Security group for ALB"

aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxxxxxx \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxxxxxx \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

# 애플리케이션 보안 그룹 (ECS용)
aws ec2 create-security-group \
  --group-name ai-auto-stock-app \
  --description "Security group for application"

# ALB에서만 접근 허용
aws ec2 authorize-security-group-ingress \
  --group-id sg-yyyyyyyyy \
  --protocol tcp \
  --port 8000 \
  --source-group sg-xxxxxxxxx
```

## 4. 백업 및 복구

### 4.1 데이터베이스 백업

**RDS 자동 백업 설정**
```bash
# 자동 백업 활성화 (Terraform에서 이미 설정됨)
aws rds modify-db-instance \
  --db-instance-identifier ai-auto-stock-db \
  --backup-retention-period 7 \
  --apply-immediately
```

**수동 스냅샷 생성**
```bash
# 수동 스냅샷 생성
aws rds create-db-snapshot \
  --db-instance-identifier ai-auto-stock-db \
  --db-snapshot-identifier ai-auto-stock-db-snapshot-$(date +%Y%m%d)

# 스냅샷 복원
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier ai-auto-stock-db-restored \
  --db-snapshot-identifier ai-auto-stock-db-snapshot-20241201
```

### 4.2 애플리케이션 데이터 백업

**S3 백업 스크립트**
```bash
#!/bin/bash
# scripts/backup.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/tmp/backup_$DATE"

# 로그 파일 백업
mkdir -p $BACKUP_DIR/logs
cp -r logs/* $BACKUP_DIR/logs/

# ML 모델 백업
mkdir -p $BACKUP_DIR/models
cp -r models/* $BACKUP_DIR/models/

# S3에 업로드
aws s3 sync $BACKUP_DIR s3://ai-auto-stock-backups/app-backup-$DATE/

# 로컬 백업 파일 정리
rm -rf $BACKUP_DIR

echo "Backup completed: s3://ai-auto-stock-backups/app-backup-$DATE/"
```

이 환경 설정 가이드를 따라하면 로컬 개발부터 프로덕션 배포까지 안정적으로 환경을 구축할 수 있습니다.
