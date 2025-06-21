# API 레퍼런스

## 1. API 개요

### 1.1 기본 정보

**Base URL**
- 개발: `http://localhost:8000`
- 프로덕션: `https://api.ai-auto-stock.com`

**API 버전**
- 현재 버전: `v1`
- 모든 API 엔드포인트는 `/api/v1` prefix를 사용합니다.

**Content-Type**
- 요청: `application/json`
- 응답: `application/json`

### 1.2 인증

**JWT Bearer Token 인증**
```bash
# 헤더에 토큰 포함
Authorization: Bearer <your_jwt_token>
```

**토큰 획득 방법**
```bash
curl -X POST "http://localhost:8000/api/v1/auth/login" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "password123"
  }'
```

### 1.3 응답 형식

**성공 응답**
```json
{
  "success": true,
  "data": {
    // 실제 데이터
  },
  "message": "Success",
  "timestamp": "2024-01-01T12:00:00Z"
}
```

**에러 응답**
```json
{
  "success": false,
  "error": {
    "code": "INVALID_REQUEST",
    "message": "Invalid request parameters",
    "details": {
      "field": "symbol",
      "issue": "Symbol is required"
    }
  },
  "timestamp": "2024-01-01T12:00:00Z"
}
```

### 1.4 에러 코드

| HTTP Status | Error Code | Description |
|-------------|------------|-------------|
| 400 | INVALID_REQUEST | 잘못된 요청 파라미터 |
| 401 | UNAUTHORIZED | 인증 토큰 없음 또는 무효 |
| 403 | FORBIDDEN | 권한 없음 |
| 404 | NOT_FOUND | 리소스를 찾을 수 없음 |
| 422 | VALIDATION_ERROR | 입력 데이터 검증 실패 |
| 429 | RATE_LIMIT_EXCEEDED | API 요청 제한 초과 |
| 500 | INTERNAL_ERROR | 서버 내부 오류 |

## 2. 인증 API

### 2.1 회원가입

**POST** `/api/v1/auth/signup`

```bash
curl -X POST "http://localhost:8000/api/v1/auth/signup" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "newuser@example.com",
    "password": "securepassword123",
    "full_name": "John Doe"
  }'
```

**Request Body**
```json
{
  "email": "string (required)",
  "password": "string (required, min 8 chars)",
  "full_name": "string (required)"
}
```

**Response**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": 1,
      "email": "newuser@example.com",
      "full_name": "John Doe",
      "subscription_plan": "free",
      "created_at": "2024-01-01T12:00:00Z"
    },
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "token_type": "bearer"
  }
}
```

### 2.2 로그인

**POST** `/api/v1/auth/login`

```bash
curl -X POST "http://localhost:8000/api/v1/auth/login" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "password123"
  }'
```

**Request Body**
```json
{
  "email": "string (required)",
  "password": "string (required)"
}
```

### 2.3 토큰 갱신

**POST** `/api/v1/auth/refresh`

```bash
curl -X POST "http://localhost:8000/api/v1/auth/refresh" \
  -H "Authorization: Bearer <refresh_token>"
```

## 3. 주식 데이터 API

### 3.1 주식 목록 조회

**GET** `/api/v1/stocks`

```bash
curl -X GET "http://localhost:8000/api/v1/stocks?market=KOSPI&limit=50&offset=0" \
  -H "Authorization: Bearer <token>"
```

**Query Parameters**
- `market` (optional): 시장 필터 (KOSPI, KOSDAQ, NASDAQ)
- `sector` (optional): 섹터 필터
- `limit` (optional): 결과 개수 (기본값: 20, 최대: 100)
- `offset` (optional): 페이지네이션 오프셋 (기본값: 0)
- `search` (optional): 종목명 또는 심볼 검색

**Response**
```json
{
  "success": true,
  "data": {
    "stocks": [
      {
        "id": 1,
        "symbol": "005930",
        "name": "삼성전자",
        "market": "KOSPI",
        "sector": "반도체",
        "current_price": 68000,
        "change_rate": 1.2,
        "volume": 15000000,
        "market_cap": 400000000000000
      }
    ],
    "total": 2000,
    "limit": 50,
    "offset": 0
  }
}
```

### 3.2 개별 주식 정보 조회

**GET** `/api/v1/stocks/{symbol}`

```bash
curl -X GET "http://localhost:8000/api/v1/stocks/005930" \
  -H "Authorization: Bearer <token>"
```

**Response**
```json
{
  "success": true,
  "data": {
    "symbol": "005930",
    "name": "삼성전자",
    "market": "KOSPI",
    "sector": "반도체",
    "industry": "반도체 제조",
    "current_price": 68000,
    "open_price": 67500,
    "high_price": 68500,
    "low_price": 67000,
    "volume": 15000000,
    "market_cap": 400000000000000,
    "pe_ratio": 12.5,
    "pb_ratio": 1.2,
    "dividend_yield": 2.5,
    "52_week_high": 85000,
    "52_week_low": 55000,
    "change_rate": 1.2,
    "change_amount": 800,
    "last_updated": "2024-01-01T15:30:00Z"
  }
}
```

### 3.3 주가 차트 데이터

**GET** `/api/v1/stocks/{symbol}/chart`

```bash
curl -X GET "http://localhost:8000/api/v1/stocks/005930/chart?timeframe=1d&period=3M" \
  -H "Authorization: Bearer <token>"
```

**Query Parameters**
- `timeframe`: 시간 간격 (1m, 5m, 15m, 30m, 1h, 1d, 1w, 1M)
- `period`: 조회 기간 (1D, 5D, 1M, 3M, 6M, 1Y, 2Y, 5Y, 10Y, max)
- `start_date` (optional): 시작 날짜 (YYYY-MM-DD)
- `end_date` (optional): 종료 날짜 (YYYY-MM-DD)

**Response**
```json
{
  "success": true,
  "data": {
    "symbol": "005930",
    "timeframe": "1d",
    "period": "3M",
    "candles": [
      {
        "timestamp": "2024-01-01T09:00:00Z",
        "open": 67500,
        "high": 68500,
        "low": 67000,
        "close": 68000,
        "volume": 15000000,
        "adjusted_close": 68000
      }
    ],
    "count": 63
  }
}
```

### 3.4 기술적 지표

**GET** `/api/v1/stocks/{symbol}/indicators`

```bash
curl -X GET "http://localhost:8000/api/v1/stocks/005930/indicators?indicators=RSI,MACD,BB&period=14" \
  -H "Authorization: Bearer <token>"
```

**Query Parameters**
- `indicators`: 요청할 지표 (RSI, MACD, BB, SMA, EMA, STOCH 등)
- `period`: 계산 기간 (기본값: 14)
- `timeframe`: 시간 간격 (기본값: 1d)

**Response**
```json
{
  "success": true,
  "data": {
    "symbol": "005930",
    "indicators": {
      "RSI": {
        "current": 65.5,
        "signal": "neutral",
        "values": [
          {
            "timestamp": "2024-01-01T09:00:00Z",
            "value": 65.5
          }
        ]
      },
      "MACD": {
        "macd": 1200,
        "signal": 1150,
        "histogram": 50,
        "trend": "bullish"
      },
      "BB": {
        "upper": 70000,
        "middle": 68000,
        "lower": 66000,
        "position": 0.5
      }
    },
    "last_updated": "2024-01-01T15:30:00Z"
  }
}
```

### 3.5 관련 뉴스

**GET** `/api/v1/stocks/{symbol}/news`

```bash
curl -X GET "http://localhost:8000/api/v1/stocks/005930/news?limit=10&lang=ko" \
  -H "Authorization: Bearer <token>"
```

**Response**
```json
{
  "success": true,
  "data": {
    "symbol": "005930",
    "news": [
      {
        "id": 1,
        "title": "삼성전자, 3분기 영업이익 10.7조원 기록",
        "summary": "반도체 호황으로 실적 개선...",
        "url": "https://news.example.com/article/123",
        "source": "한국경제신문",
        "published_at": "2024-01-01T10:00:00Z",
        "sentiment_score": 0.8,
        "sentiment": "positive",
        "relevance_score": 0.95
      }
    ],
    "total": 150,
    "limit": 10
  }
}
```

## 4. 포트폴리오 API

### 4.1 포트폴리오 목록

**GET** `/api/v1/portfolios`

```bash
curl -X GET "http://localhost:8000/api/v1/portfolios" \
  -H "Authorization: Bearer <token>"
```

**Response**
```json
{
  "success": true,
  "data": {
    "portfolios": [
      {
        "id": 1,
        "name": "My Stock Portfolio",
        "description": "장기 투자 포트폴리오",
        "total_value": 50000000,
        "total_return": 2500000,
        "return_rate": 5.0,
        "created_at": "2024-01-01T12:00:00Z",
        "holdings_count": 8
      }
    ]
  }
}
```

### 4.2 포트폴리오 생성

**POST** `/api/v1/portfolios`

```bash
curl -X POST "http://localhost:8000/api/v1/portfolios" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Tech Stocks Portfolio",
    "description": "IT 기업 중심 포트폴리오",
    "initial_capital": 10000000
  }'
```

### 4.3 포트폴리오 상세 조회

**GET** `/api/v1/portfolios/{portfolio_id}`

```bash
curl -X GET "http://localhost:8000/api/v1/portfolios/1" \
  -H "Authorization: Bearer <token>"
```

**Response**
```json
{
  "success": true,
  "data": {
    "portfolio": {
      "id": 1,
      "name": "My Stock Portfolio",
      "description": "장기 투자 포트폴리오",
      "total_value": 50000000,
      "total_return": 2500000,
      "return_rate": 5.0,
      "holdings": [
        {
          "symbol": "005930",
          "name": "삼성전자",
          "quantity": 100,
          "average_price": 65000,
          "current_price": 68000,
          "total_value": 6800000,
          "unrealized_gain": 300000,
          "unrealized_gain_rate": 4.6,
          "weight": 13.6
        }
      ],
      "performance": {
        "daily_return": 50000,
        "weekly_return": 200000,
        "monthly_return": 800000,
        "ytd_return": 2500000
      },
      "risk_metrics": {
        "volatility": 0.15,
        "sharpe_ratio": 1.2,
        "max_drawdown": 0.08,
        "beta": 1.1
      }
    }
  }
}
```

### 4.4 종목 추가

**POST** `/api/v1/portfolios/{portfolio_id}/holdings`

```bash
curl -X POST "http://localhost:8000/api/v1/portfolios/1/holdings" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "symbol": "000660",
    "quantity": 50,
    "purchase_price": 85000,
    "purchase_date": "2024-01-01"
  }'
```

## 5. 백테스팅 API

### 5.1 백테스트 실행

**POST** `/api/v1/backtests`

```bash
curl -X POST "http://localhost:8000/api/v1/backtests" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "RSI Strategy Test",
    "strategy": {
      "type": "rsi_strategy",
      "parameters": {
        "rsi_period": 14,
        "oversold_threshold": 30,
        "overbought_threshold": 70
      }
    },
    "universe": ["005930", "000660", "035420"],
    "start_date": "2023-01-01",
    "end_date": "2023-12-31",
    "initial_capital": 10000000,
    "commission": 0.0015
  }'
```

**Response**
```json
{
  "success": true,
  "data": {
    "backtest_id": "bt_123456789",
    "status": "running",
    "estimated_completion": "2024-01-01T12:05:00Z"
  }
}
```

### 5.2 백테스트 결과 조회

**GET** `/api/v1/backtests/{backtest_id}`

```bash
curl -X GET "http://localhost:8000/api/v1/backtests/bt_123456789" \
  -H "Authorization: Bearer <token>"
```

**Response**
```json
{
  "success": true,
  "data": {
    "backtest": {
      "id": "bt_123456789",
      "name": "RSI Strategy Test",
      "status": "completed",
      "results": {
        "total_return": 0.15,
        "annual_return": 0.12,
        "sharpe_ratio": 1.45,
        "max_drawdown": 0.08,
        "volatility": 0.18,
        "win_rate": 0.62,
        "profit_factor": 1.35,
        "total_trades": 156,
        "final_portfolio_value": 11500000
      },
      "performance_chart": {
        "dates": ["2023-01-01", "2023-01-02", "..."],
        "portfolio_values": [10000000, 10050000, "..."],
        "benchmark_values": [10000000, 10020000, "..."]
      },
      "trades": [
        {
          "symbol": "005930",
          "side": "buy",
          "quantity": 100,
          "price": 65000,
          "timestamp": "2023-01-15T09:30:00Z",
          "signal": "RSI oversold"
        }
      ]
    }
  }
}
```

## 6. 시장 데이터 API

### 6.1 시장 현황

**GET** `/api/v1/market/summary`

```bash
curl -X GET "http://localhost:8000/api/v1/market/summary" \
  -H "Authorization: Bearer <token>"
```

**Response**
```json
{
  "success": true,
  "data": {
    "markets": {
      "KOSPI": {
        "index": 2650.5,
        "change": 15.2,
        "change_rate": 0.58,
        "volume": 500000000000
      },
      "KOSDAQ": {
        "index": 850.2,
        "change": -5.8,
        "change_rate": -0.68,
        "volume": 200000000000
      }
    },
    "top_gainers": [
      {
        "symbol": "123456",
        "name": "상승종목",
        "change_rate": 15.0,
        "price": 25000
      }
    ],
    "top_losers": [
      {
        "symbol": "654321",
        "name": "하락종목",
        "change_rate": -8.5,
        "price": 15000
      }
    ],
    "last_updated": "2024-01-01T15:30:00Z"
  }
}
```

## 7. WebSocket API

### 7.1 실시간 주가 구독

**연결**
```javascript
const ws = new WebSocket('ws://localhost:8000/ws/stocks/realtime');

// 인증
ws.onopen = function() {
  ws.send(JSON.stringify({
    type: 'auth',
    token: 'your_jwt_token'
  }));
};

// 종목 구독
ws.send(JSON.stringify({
  type: 'subscribe',
  symbols: ['005930', '000660', '035420']
}));

// 실시간 데이터 수신
ws.onmessage = function(event) {
  const data = JSON.parse(event.data);
  console.log(data);
};
```

**수신 데이터 형식**
```json
{
  "type": "price_update",
  "symbol": "005930",
  "price": 68000,
  "change": 800,
  "change_rate": 1.2,
  "volume": 15000000,
  "timestamp": "2024-01-01T15:30:00Z"
}
```

## 8. SDK 사용법

### 8.1 Python SDK

```python
from ai_auto_stock import StockAPI

# 클라이언트 초기화
client = StockAPI(
    base_url="http://localhost:8000",
    api_key="your_api_key"  # 또는 JWT 토큰
)

# 주식 데이터 조회
stock = client.stocks.get("005930")
print(f"{stock.name}: {stock.current_price}")

# 차트 데이터 조회
chart_data = client.stocks.get_chart("005930", timeframe="1d", period="1M")

# 기술적 지표 계산
indicators = client.stocks.get_indicators("005930", indicators=["RSI", "MACD"])

# 포트폴리오 생성
portfolio = client.portfolios.create(
    name="My Portfolio",
    initial_capital=10000000
)

# 백테스트 실행
backtest = client.backtests.run(
    strategy={
        "type": "rsi_strategy",
        "parameters": {"rsi_period": 14}
    },
    universe=["005930", "000660"],
    start_date="2023-01-01",
    end_date="2023-12-31"
)
```

### 8.2 JavaScript SDK

```javascript
import { StockAPI } from 'ai-auto-stock-js';

const client = new StockAPI({
  baseURL: 'http://localhost:8000',
  apiKey: 'your_api_key'
});

// 주식 데이터 조회
const stock = await client.stocks.get('005930');
console.log(`${stock.name}: ${stock.current_price}`);

// 실시간 데이터 구독
const subscription = client.subscribe({
  symbols: ['005930', '000660'],
  onUpdate: (data) => {
    console.log('Price update:', data);
  }
});
```

이 API 레퍼런스를 참고하여 AI Auto Stock 서비스의 모든 기능을 활용할 수 있습니다.
