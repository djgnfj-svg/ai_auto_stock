# AI Auto Stock 📈

> AI 기반 종합 주식 분석 플랫폼 - MLOps 파이프라인을 활용한 실시간 주식 분석 서비스

[![Python](https://img.shields.io/badge/Python-3.11-blue.svg)](https://python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.104.1-green.svg)](https://fastapi.tiangolo.com)
[![React](https://img.shields.io/badge/React-18.2.0-blue.svg)](https://reactjs.org)
[![Docker](https://img.shields.io/badge/Docker-Ready-blue.svg)](https://docker.com)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

## 🎯 프로젝트 개요

개인 투자자들을 위한 AI 기반 주식 분석 플랫폼입니다. MLOps 파이프라인을 활용하여 실시간 데이터 수집, AI 분석, 백테스팅을 제공합니다.

### 핵심 기능

- 🤖 **AI 기반 분석**: 기술적 지표 + 감성 분석 + 패턴 인식
- 📊 **실시간 데이터**: 주가, 뉴스, 공시 정보 실시간 수집
- 🔄 **MLOps 파이프라인**: Airflow + MLflow를 활용한 자동화된 ML 워크플로우
- 📈 **백테스팅**: 투자 전략 검증 및 성과 분석
- 🌐 **실시간 API**: 고성능 분석 결과 제공

### 기술 스택

**Backend**
- FastAPI 0.104.1 (Python 3.11)
- PostgreSQL 15 + Redis 7.2
- Apache Airflow 2.7.3 (MLOps)
- MLflow 2.8.1 (ML 라이프사이클)

**Frontend**
- React 18.2.0 + TypeScript
- Material-UI + TradingView Charts
- Redux Toolkit (상태 관리)

**ML/Data**
- pandas, numpy, scikit-learn
- TensorFlow 2.14.0 (LSTM)
- TA-Lib (기술적 분석)
- Transformers (KoBERT 감성분석)

**Infrastructure**
- Docker + Kubernetes
- AWS (ECS, RDS, ElastiCache, S3)
- GitHub Actions (CI/CD)

## 🚀 빠른 시작

### 요구사항

- Python 3.11+
- Node.js 20+
- Docker & Docker Compose
- PostgreSQL 15
- Redis 7.2

### 로컬 개발 환경 설정

```bash
# 1. 저장소 클론
git clone https://github.com/djgnfj-svg/ai_auto_stock.git
cd ai_auto_stock

# 2. 환경 변수 설정
cp .env.example .env
# .env 파일에 API 키 등 설정

# 3. Docker로 개발 환경 실행
docker-compose up -d

# 4. 백엔드 의존성 설치 및 실행
cd backend
pip install -r requirements.txt
uvicorn main:app --reload --host 0.0.0.0 --port 8000

# 5. 프론트엔드 실행
cd ../frontend
npm install
npm run dev
```

### 서비스 접속

- **Frontend**: http://localhost:3000
- **Backend API**: http://localhost:8000
- **API Docs**: http://localhost:8000/docs
- **Airflow**: http://localhost:8080
- **MLflow**: http://localhost:5000

## 📁 프로젝트 구조

```
ai_auto_stock/
├── backend/                 # FastAPI 백엔드
│   ├── app/
│   │   ├── api/            # API 라우터
│   │   ├── core/           # 설정, 보안
│   │   ├── models/         # 데이터베이스 모델
│   │   ├── services/       # 비즈니스 로직
│   │   └── ml/             # ML 모델 및 파이프라인
│   ├── airflow/            # Airflow DAGs
│   ├── tests/              # 테스트 코드
│   └── requirements.txt
├── frontend/               # React 프론트엔드
│   ├── src/
│   │   ├── components/     # React 컴포넌트
│   │   ├── pages/          # 페이지 컴포넌트
│   │   ├── services/       # API 서비스
│   │   └── utils/          # 유틸리티
│   └── package.json
├── data/                   # 데이터 저장소
├── docs/                   # 프로젝트 문서
├── docker/                 # Docker 설정
├── scripts/                # 유틸리티 스크립트
└── README.md
```

## 📚 문서

자세한 문서는 `docs/` 폴더를 참고하세요:

- [아키텍처 설계](docs/ARCHITECTURE.md)
- [개발 가이드](docs/DEVELOPMENT-GUIDE.md)
- [환경 설정](docs/SETUP.md)
- [API 레퍼런스](docs/API-REFERENCE.md)

## 🧪 테스트

```bash
# 백엔드 테스트
cd backend
pytest

# 프론트엔드 테스트
cd frontend
npm test

# E2E 테스트
npm run test:e2e
```

## 🚀 배포

### Docker로 프로덕션 배포

```bash
# 프로덕션 빌드
docker-compose -f docker-compose.prod.yml up -d

# AWS ECS 배포 (Terraform)
cd infrastructure
terraform init
terraform plan
terraform apply
```

## 📊 모니터링

- **시스템 메트릭**: Prometheus + Grafana
- **애플리케이션 로그**: ELK Stack
- **에러 추적**: Sentry
- **API 모니터링**: AWS CloudWatch

## 🤝 기여하기

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📝 라이센스

이 프로젝트는 MIT 라이센스 하에 배포됩니다. 자세한 내용은 [LICENSE](LICENSE) 파일을 참고하세요.

## 📞 연락처

프로젝트 관련 문의: djgnfj.svg@gmail.com

---

⭐ 이 프로젝트가 도움이 되었다면 Star를 눌러주세요!
