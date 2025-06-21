# 개발 가이드

## 1. 개발 환경 준비

### 1.1 필수 도구 설치

**Python 개발 환경**
```bash
# Python 3.11 설치 (pyenv 권장)
pyenv install 3.11.7
pyenv global 3.11.7

# Poetry 설치 (의존성 관리)
curl -sSL https://install.python-poetry.org | python3 -
poetry --version

# 가상환경 생성
cd ai_auto_stock/backend
poetry install
poetry shell
```

**Node.js 개발 환경**
```bash
# Node.js 20 설치 (nvm 권장)
nvm install 20
nvm use 20

# 프론트엔드 의존성 설치
cd ai_auto_stock/frontend
npm install

# 개발 서버 실행
npm run dev
```

**데이터베이스 설정**
```bash
# Docker로 개발 DB 실행
docker-compose -f docker/docker-compose.dev.yml up -d postgres redis

# 데이터베이스 마이그레이션
cd backend
alembic upgrade head

# 초기 데이터 로드
python scripts/load_initial_data.py
```

### 1.2 IDE 설정

**VSCode 추천 확장**
```json
{
  "recommendations": [
    "ms-python.python",
    "ms-python.black-formatter",
    "ms-python.isort",
    "ms-python.flake8",
    "ms-vscode.vscode-typescript-next",
    "bradlc.vscode-tailwindcss",
    "ms-vscode.vscode-json",
    "redhat.vscode-yaml",
    "ms-vscode-remote.remote-containers"
  ]
}
```

**설정 파일 (.vscode/settings.json)**
```json
{
  "python.defaultInterpreterPath": "./backend/.venv/bin/python",
  "python.formatting.provider": "black",
  "python.linting.enabled": true,
  "python.linting.flake8Enabled": true,
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.organizeImports": true
  },
  "typescript.preferences.importModuleSpecifier": "relative"
}
```

## 2. 프로젝트 구조 및 컨벤션

### 2.1 백엔드 구조 (FastAPI)

```
backend/
├── app/
│   ├── __init__.py
│   ├── main.py                 # FastAPI 앱 진입점
│   ├── api/                    # API 라우터
│   │   ├── __init__.py
│   │   ├── v1/
│   │   │   ├── __init__.py
│   │   │   ├── auth.py         # 인증 관련 API
│   │   │   ├── stocks.py       # 주식 데이터 API
│   │   │   ├── portfolios.py   # 포트폴리오 API
│   │   │   └── backtests.py    # 백테스팅 API
│   │   └── deps.py             # 의존성 주입
│   ├── core/                   # 핵심 설정
│   │   ├── __init__.py
│   │   ├── config.py           # 설정 관리
│   │   ├── security.py         # 보안 유틸리티
│   │   └── database.py         # DB 연결
│   ├── models/                 # 데이터베이스 모델
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── stock.py
│   │   └── portfolio.py
│   ├── schemas/                # Pydantic 스키마
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── stock.py
│   │   └── portfolio.py
│   ├── services/               # 비즈니스 로직
│   │   ├── __init__.py
│   │   ├── stock_analysis.py
│   │   ├── ml_prediction.py
│   │   └── backtesting.py
│   ├── ml/                     # ML 관련 코드
│   │   ├── __init__.py
│   │   ├── models/
│   │   ├── features/
│   │   └── utils/
│   └── utils/                  # 유틸리티 함수
│       ├── __init__.py
│       ├── data_collectors.py
│       └── technical_analysis.py
├── airflow/                    # Airflow DAGs
│   ├── dags/
│   │   ├── daily_data_collection.py
│   │   ├── realtime_processing.py
│   │   └── ml_training.py
│   └── plugins/
├── tests/                      # 테스트 코드
│   ├── conftest.py
│   ├── test_api/
│   ├── test_services/
│   └── test_ml/
├── alembic/                    # DB 마이그레이션
├── pyproject.toml              # Python 의존성
└── requirements.txt
```

### 2.2 프론트엔드 구조 (React + TypeScript)

```
frontend/
├── public/
├── src/
│   ├── components/             # 재사용 가능한 컴포넌트
│   │   ├── common/
│   │   │   ├── Header.tsx
│   │   │   ├── Sidebar.tsx
│   │   │   └── Loading.tsx
│   │   ├── charts/
│   │   │   ├── StockChart.tsx
│   │   │   └── PortfolioChart.tsx
│   │   └── forms/
│   ├── pages/                  # 페이지 컴포넌트
│   │   ├── Dashboard.tsx
│   │   ├── StockAnalysis.tsx
│   │   ├── Portfolio.tsx
│   │   └── Backtesting.tsx
│   ├── hooks/                  # 커스텀 Hook
│   │   ├── useStockData.ts
│   │   └── useWebSocket.ts
│   ├── services/               # API 서비스
│   │   ├── api.ts
│   │   ├── stockService.ts
│   │   └── authService.ts
│   ├── store/                  # 상태 관리 (Redux Toolkit)
│   │   ├── index.ts
│   │   ├── authSlice.ts
│   │   └── stockSlice.ts
│   ├── types/                  # TypeScript 타입 정의
│   │   ├── stock.ts
│   │   ├── user.ts
│   │   └── api.ts
│   ├── utils/                  # 유틸리티 함수
│   │   ├── formatters.ts
│   │   └── validators.ts
│   └── styles/                 # 스타일링
│       ├── globals.css
│       └── components/
├── package.json
└── tsconfig.json
```

### 2.3 코딩 컨벤션

**Python 코드 스타일**
```python
# 1. 함수명: snake_case
def calculate_rsi(prices: List[float], period: int = 14) -> List[float]:
    """RSI 계산 함수
    
    Args:
        prices: 가격 리스트
        period: 계산 기간 (기본값: 14)
    
    Returns:
        RSI 값 리스트
    """
    pass

# 2. 클래스명: PascalCase
class StockAnalyzer:
    def __init__(self, symbol: str):
        self.symbol = symbol
        self._cache = {}  # private 속성은 underscore

# 3. 상수: UPPER_SNAKE_CASE
MAX_RETRY_COUNT = 3
DEFAULT_TIMEOUT = 30

# 4. 타입 힌트 사용
from typing import List, Dict, Optional, Union

def get_stock_data(
    symbol: str, 
    start_date: Optional[str] = None
) -> Dict[str, Union[float, str]]:
    pass
```

**TypeScript 코드 스타일**
```typescript
// 1. 인터페이스명: PascalCase
interface StockData {
  symbol: string;
  price: number;
  volume: number;
  timestamp: string;
}

// 2. 함수명: camelCase
const calculateMovingAverage = (
  prices: number[], 
  period: number = 20
): number[] => {
  // 구현
};

// 3. 컴포넌트명: PascalCase
const StockChart: React.FC<StockChartProps> = ({ data, height = 400 }) => {
  return <div>Chart Component</div>;
};

// 4. 훅 함수명: use로 시작
const useStockData = (symbol: string) => {
  const [data, setData] = useState<StockData | null>(null);
  // 구현
  return { data, loading, error };
};
```

## 3. 개발 워크플로우

### 3.1 Git 브랜치 전략

**GitFlow 기반 브랜치 전략**
```
main                 # 프로덕션 브랜치
├── develop          # 개발 통합 브랜치
├── feature/         # 기능 개발 브랜치
│   ├── feature/stock-analysis-api
│   ├── feature/ml-prediction-model
│   └── feature/portfolio-management
├── release/         # 릴리즈 준비 브랜치
│   └── release/v1.0.0
└── hotfix/          # 긴급 수정 브랜치
    └── hotfix/critical-bug-fix
```

**브랜치 생성 및 작업 흐름**
```bash
# 1. 새 기능 개발 시작
git checkout develop
git pull origin develop
git checkout -b feature/stock-analysis-api

# 2. 개발 완료 후 PR 생성
git add .
git commit -m "feat: Add stock analysis API endpoints"
git push origin feature/stock-analysis-api

# 3. PR 리뷰 및 병합 후 브랜치 정리
git checkout develop
git pull origin develop
git branch -d feature/stock-analysis-api
```

**커밋 메시지 컨벤션**
```
<type>(<scope>): <subject>

<body>

<footer>
```

**Type 종류**
- `feat`: 새로운 기능
- `fix`: 버그 수정
- `docs`: 문서 수정
- `style`: 코드 포맷팅
- `refactor`: 코드 리팩토링
- `test`: 테스트 추가/수정
- `chore`: 빌드 과정 또는 보조 도구 변경

**예시**
```
feat(api): Add stock price prediction endpoint

- Implement LSTM model for price prediction
- Add API endpoint /api/v1/stocks/{symbol}/predict
- Include confidence intervals in response

Closes #123
```

### 3.2 코드 리뷰 가이드라인

**리뷰 체크리스트**
```markdown
## 기능성
- [ ] 요구사항을 올바르게 구현했는가?
- [ ] 에지 케이스를 고려했는가?
- [ ] 에러 처리가 적절한가?

## 코드 품질
- [ ] 코드가 읽기 쉽고 이해하기 쉬운가?
- [ ] 함수/클래스가 단일 책임 원칙을 따르는가?
- [ ] 중복 코드가 없는가?

## 성능
- [ ] 불필요한 연산이나 메모리 사용이 없는가?
- [ ] 데이터베이스 쿼리가 최적화되었는가?
- [ ] 캐싱을 적절히 활용했는가?

## 보안
- [ ] 사용자 입력 검증이 적절한가?
- [ ] 인증/인가가 올바르게 구현되었는가?
- [ ] 민감한 정보가 노출되지 않는가?

## 테스트
- [ ] 단위 테스트가 작성되었는가?
- [ ] 테스트 커버리지가 충분한가?
- [ ] 통합 테스트가 필요한 경우 작성되었는가?
```

### 3.3 테스트 전략

**백엔드 테스트 (pytest)**
```python
# tests/conftest.py
import pytest
from fastapi.testclient import TestClient
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

from app.main import app
from app.core.database import get_db
from app.core.config import settings

# 테스트 DB 설정
SQLALCHEMY_DATABASE_URL = "sqlite:///./test.db"
engine = create_engine(SQLALCHEMY_DATABASE_URL, connect_args={"check_same_thread": False})
TestingSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

def override_get_db():
    try:
        db = TestingSessionLocal()
        yield db
    finally:
        db.close()

app.dependency_overrides[get_db] = override_get_db

@pytest.fixture
def client():
    return TestClient(app)

@pytest.fixture
def test_user():
    return {
        "email": "test@example.com",
        "password": "testpass123"
    }
```

**단위 테스트 예시**
```python
# tests/test_services/test_stock_analysis.py
import pytest
from app.services.stock_analysis import StockAnalyzer

class TestStockAnalyzer:
    def setup_method(self):
        self.analyzer = StockAnalyzer("AAPL")
    
    def test_calculate_rsi(self):
        prices = [100, 102, 101, 103, 105, 104, 106, 108, 107, 109]
        rsi_values = self.analyzer.calculate_rsi(prices, period=5)
        
        assert len(rsi_values) == len(prices)
        assert all(0 <= rsi <= 100 for rsi in rsi_values if rsi is not None)
    
    @pytest.mark.asyncio
    async def test_get_real_time_data(self):
        data = await self.analyzer.get_real_time_data()
        
        assert "symbol" in data
        assert "price" in data
        assert "timestamp" in data
        assert data["symbol"] == "AAPL"
```

**API 테스트 예시**
```python
# tests/test_api/test_stocks.py
def test_get_stock_list(client):
    response = client.get("/api/v1/stocks")
    assert response.status_code == 200
    
    data = response.json()
    assert "stocks" in data
    assert isinstance(data["stocks"], list)

def test_get_stock_detail(client):
    response = client.get("/api/v1/stocks/AAPL")
    assert response.status_code == 200
    
    data = response.json()
    assert data["symbol"] == "AAPL"
    assert "current_price" in data
    assert "volume" in data
```

**프론트엔드 테스트 (Jest + React Testing Library)**
```typescript
// src/components/__tests__/StockChart.test.tsx
import { render, screen } from '@testing-library/react';
import { StockChart } from '../StockChart';

const mockData = [
  { timestamp: '2024-01-01', price: 100 },
  { timestamp: '2024-01-02', price: 102 },
];

describe('StockChart', () => {
  it('renders chart with data', () => {
    render(<StockChart data={mockData} />);
    
    expect(screen.getByTestId('stock-chart')).toBeInTheDocument();
  });
  
  it('shows loading state when data is empty', () => {
    render(<StockChart data={[]} />);
    
    expect(screen.getByText('Loading...')).toBeInTheDocument();
  });
});
```

## 4. 디버깅 및 트러블슈팅

### 4.1 로깅 설정

**백엔드 로깅**
```python
# app/core/logging.py
import logging
from loguru import logger
import sys

def setup_logging(log_level: str = "INFO"):
    # 기본 로거 제거
    logger.remove()
    
    # 콘솔 로깅
    logger.add(
        sys.stdout,
        level=log_level,
        format="<green>{time:YYYY-MM-DD HH:mm:ss}</green> | <level>{level: <8}</level> | <cyan>{name}</cyan>:<cyan>{function}</cyan>:<cyan>{line}</cyan> - <level>{message}</level>"
    )
    
    # 파일 로깅
    logger.add(
        "logs/app.log",
        level=log_level,
        format="{time:YYYY-MM-DD HH:mm:ss} | {level: <8} | {name}:{function}:{line} - {message}",
        rotation="10 MB",
        retention="30 days",
        compression="zip"
    )

# 사용 예시
from loguru import logger

@logger.catch
async def risky_function():
    # 예외가 발생하면 자동으로 로그 기록
    result = await some_async_operation()
    logger.info(f"Operation completed: {result}")
    return result
```

### 4.2 성능 모니터링

**프로파일링**
```python
# 성능 측정 데코레이터
import time
import functools
from loguru import logger

def measure_time(func):
    @functools.wraps(func)
    async def wrapper(*args, **kwargs):
        start_time = time.time()
        try:
            result = await func(*args, **kwargs)
            return result
        finally:
            elapsed_time = time.time() - start_time
            logger.info(f"{func.__name__} took {elapsed_time:.2f} seconds")
    return wrapper

# 사용 예시
@measure_time
async def expensive_calculation():
    # 시간이 오래 걸리는 작업
    pass
```

### 4.3 일반적인 문제 해결

**데이터베이스 연결 문제**
```python
# 연결 풀 설정 확인
from sqlalchemy import create_engine

engine = create_engine(
    DATABASE_URL,
    pool_size=20,          # 기본 연결 풀 크기
    max_overflow=30,       # 추가 연결 수
    pool_pre_ping=True,    # 연결 상태 확인
    pool_recycle=3600,     # 1시간마다 연결 재생성
)
```

**메모리 누수 방지**
```python
# 대용량 데이터 처리 시 배치 처리
def process_large_dataset(data_iterator, batch_size=1000):
    batch = []
    for item in data_iterator:
        batch.append(item)
        if len(batch) >= batch_size:
            process_batch(batch)
            batch.clear()  # 메모리 해제
    
    # 남은 데이터 처리
    if batch:
        process_batch(batch)
```

이 개발 가이드를 따라 일관되고 고품질의 코드를 작성할 수 있습니다.
