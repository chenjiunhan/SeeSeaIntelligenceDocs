# SeeSea Intelligence 技術架構文件

**現代化、高性能、高併發的全球航運情報分析系統**

Version: 3.0
Last Updated: 2026-02-07
Domain: https://seesea.ai

## 📦 專案架構

本專案採用 **多倉庫架構（Multi-Repo）**，拆分為四個獨立專案：

| 專案名稱 | 職責 | 技術棧 | Repository |
|---------|------|--------|-----------|
| **SeeSeaIntelligence** | 資料收集與處理 | Python 3.12 (爬蟲、ETL、資料處理) | `/SeeSeaIntelligence` |
| **SeeSeaIntelligenceAgent** | AI 分析與智能代理 | Python 3.12 + LangGraph + FastAPI | `/SeeSeaIntelligenceAgent` |
| **SeeSeaIntelligenceAPI** | 高性能資料查詢 | Go 1.21 + Gin Framework | `/SeeSeaIntelligenceAPI` |
| **SeeSeaIntelligenceWeb** | 前端展示與互動 | Next.js 15 + React 18 + TypeScript | `/SeeSeaIntelligenceWeb` |

---

## 📊 技術堆棧總覽

### 前端技術棧

| 技術 | 版本 | 用途 |
|------|------|------|
| **框架** | Next.js 15 (App Router) | SSR/SSG、路由管理、性能優化 |
| **UI 語言** | React 18 + TypeScript | 組件開發、類型安全 |
| **樣式** | Tailwind CSS + shadcn/ui | 原子化 CSS、高品質組件庫 |
| **狀態管理** | Zustand + TanStack Query | 全局狀態、伺服器狀態快取 |
| **資料可視化** | D3.js v7 + Visx | 主力圖表庫（高度客製化） |
| **地圖可視化** | Mapbox GL JS + Deck.gl | 全球航道地圖、空間資料可視化 |
| **表格** | TanStack Table | 虛擬滾動、大數據表格 |

### 後端技術棧（混合架構）

| 專案 | 技術 | 版本 | 用途 |
|------|------|------|------|
| **SeeSeaIntelligence** | Python + APScheduler | 3.12 | 資料收集爬蟲、資料處理、ETL Pipeline |
| **SeeSeaIntelligenceAPI** | Go + Gin | 1.21 | 高性能資料查詢 API (70% 流量) |
| **SeeSeaIntelligenceAgent** | Python + FastAPI + LangGraph | 3.12 | 複雜分析、AI Agent (30% 流量) |
| **API Gateway** | Nginx | Latest | 反向代理、負載均衡、SSL 終止 |
| **WebSocket** | Go Gorilla WebSocket | Latest | 即時資料推送 |

### 資料層技術棧（雙資料庫架構）

| 技術 | 版本 | 用途 |
|------|------|------|
| **OLTP 資料庫** | PostgreSQL 16 + TimescaleDB | 即時寫入、CRUD、熱數據查詢 |
| **OLAP 資料庫** | ClickHouse 24 | 歷史分析、複雜聚合、全時段數據 |
| **快取層** | Redis 7 | 查詢結果快取、Session 管理、Pub/Sub |
| **資料備份** | CSV/JSON Files | 原始資料備份、可重新處理 |

### 基礎設施與部署

| 技術 | 用途 |
|------|------|
| **前端部署** | Vercel (免費全球 CDN) |
| **後端部署** | AWS EC2 (自有主機) |
| **容器化** | Docker + Docker Compose |
| **域名** | seesea.ai (GoDaddy) |
| **SSL 憑證** | Let's Encrypt (免費自動續約) |
| **監控** | Grafana + Prometheus |
| **日誌** | Loki |
| **CI/CD** | GitHub Actions |

---

## 🏗️ 系統架構圖

### 完整部署架構（多專案整合）

```
┌─────────────────────────────────────────────────────────────────────┐
│                      使用者瀏覽器 (全球)                              │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│              前端：https://seesea.ai                                 │
│              Vercel CDN (全球 300+ 節點)                             │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │         SeeSeaIntelligenceWeb                                  │ │
│  │  • Next.js 15 + React 18 + TypeScript                          │ │
│  │  • D3.js + Visx 圖表                                            │ │
│  │  • Mapbox GL + Deck.gl 地圖                                    │ │
│  │  • TanStack Query (自動快取)                                   │ │
│  └────────────────────────────────────────────────────────────────┘ │
└──────────────────────────────┬──────────────────────────────────────┘
                               │ HTTPS API 請求
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│          GoDaddy DNS: seesea.ai                                      │
│  • seesea.ai         → Vercel (CNAME)                               │
│  • api.seesea.ai     → EC2 IP (A Record)                            │
│  • ws.seesea.ai      → EC2 IP (A Record, WebSocket)                 │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│              後端：AWS EC2 主機                                       │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                    Nginx (Port 80/443)                        │  │
│  │  • 反向代理 + SSL 終止 (Let's Encrypt)                         │  │
│  │  • 路由規則:                                                   │  │
│  │    - /api/v1/vessels/*     → SeeSeaIntelligenceAPI           │  │
│  │    - /api/v1/analytics/*   → SeeSeaIntelligenceAgent         │  │
│  │    - /api/v1/chat/*        → SeeSeaIntelligenceAgent         │  │
│  │    - /ws/*                 → SeeSeaIntelligenceAPI           │  │
│  └──────────────────┬────────────────────────┬───────────────────┘  │
│                     │                        │                      │
│                     ▼                        ▼                      │
│  ┌─────────────────────────────┐  ┌──────────────────────────────┐ │
│  │  SeeSeaIntelligenceAPI      │  │  SeeSeaIntelligenceAgent     │ │
│  │  (Go + Gin)                 │  │  (Python + FastAPI)          │ │
│  │  Port: 8080                 │  │  Port: 8000                  │ │
│  │                             │  │                              │ │
│  │  職責:                       │  │  職責:                        │ │
│  │  • 高性能資料查詢 API        │  │  • 複雜統計分析               │ │
│  │  • WebSocket 即時推送       │  │  • LangGraph AI Agent        │ │
│  │  • 快取管理                 │  │  • 自然語言查詢               │ │
│  │  • 高併發處理               │  │  • 報表生成                  │ │
│  └────────┬────────────────────┘  └────────────┬─────────────────┘ │
│           │                                    │                   │
│           └────────────────┬───────────────────┘                   │
│                            │                                       │
│            ┌───────────────┼────────────────┐                      │
│            │               │                │                      │
│            ▼               ▼                ▼                      │
│   ┌───────────────┐ ┌────────────┐ ┌──────────────┐               │
│   │ PostgreSQL 16 │ │ ClickHouse │ │  Redis 7     │               │
│   │ + TimescaleDB │ │     24     │ │              │               │
│   │ Port: 5432    │ │ Port: 9000 │ │  Port: 6379  │               │
│   │               │ │            │ │              │               │
│   │ • 即時寫入    │ │ • 歷史分析 │ │ • 查詢快取   │               │
│   │ • 熱數據(30天)│ │ • 全時段   │ │ • Pub/Sub    │               │
│   └───────▲───────┘ └──────▲─────┘ └──────────────┘               │
│           │                │                                       │
│           │                │                                       │
│           │      ┌─────────┴──────────┐                            │
│           │      │  ETL Pipeline      │                            │
│           │      │  (每日 2:00)        │                            │
│           │      │  PG → ClickHouse   │                            │
│           │      └────────────────────┘                            │
│           │                                                        │
│           │      ┌─────────────────────────────────────────────┐   │
│           └──────┤   SeeSeaIntelligence                        │   │
│                  │   (Python 資料收集與處理)                    │   │
│                  │                                             │   │
│                  │  • IMF PortWatch 爬蟲                       │   │
│                  │  • 資料清洗與處理                            │   │
│                  │  • CSV 備份與版本控制                       │   │
│                  │  • ETL Pipeline (CSV → PostgreSQL)          │   │
│                  │  • APScheduler 定時任務                     │   │
│                  └─────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

### 資料流程架構（多專案協作）

```
1️⃣ 資料收集層 (SeeSeaIntelligence)
   IMF PortWatch API
        ↓
   Python Scraper (collectors/imf_portwatch.py)
        ↓
   Pickle Files (data/logistics/chokepoints/)
        ↓

2️⃣ 資料處理層 (SeeSeaIntelligence)
   DataProcessor (core/processor.py)
        ↓
   CSV Files (processed/) ← 原始資料備份，可重新處理
        ↓

3️⃣ 資料庫層（雙庫協作）
   ETL Pipeline (SeeSeaIntelligence/etl/)
        ↓
   PostgreSQL (即時寫入、熱數據、CRUD)
        ↓ (每日凌晨 2:00 同步)
   ClickHouse (歷史分析、複雜聚合、全時段)
        ↓

4️⃣ 快取層
   Redis (查詢結果快取 TTL: 5 分鐘)
        ↓

5️⃣ API 層（智能路由）
   ┌─ 簡單查詢 (< 30 天) → SeeSeaIntelligenceAPI (Go) → PostgreSQL
   ├─ 複雜分析 (> 90 天) → SeeSeaIntelligenceAgent (Python) → ClickHouse
   ├─ AI 自然語言查詢 → SeeSeaIntelligenceAgent (LangGraph) → ClickHouse
   └─ 即時推送 → SeeSeaIntelligenceAPI (WebSocket) → Redis Pub/Sub
        ↓

6️⃣ 應用層 (SeeSeaIntelligenceWeb)
   Next.js Frontend (Vercel CDN)
        ↓
   使用者瀏覽器
```

---

## 📁 專案目錄結構（多倉庫架構）

### 專案總覽

```
SeeSea/
│
├── SeeSeaIntelligence/               # 資料收集與處理專案
├── SeeSeaIntelligenceAgent/          # AI 分析與智能代理專案
├── SeeSeaIntelligenceAPI/            # 高性能 API 專案
├── SeeSeaIntelligenceWeb/            # 前端專案
└── SeeSeaIntelligenceDocs/           # 文件專案
```

---

### 1️⃣ SeeSeaIntelligence（資料收集與處理）

```
SeeSeaIntelligence/
│
├── collectors/                       # 資料收集爬蟲
│   ├── __init__.py
│   └── imf_portwatch.py             # IMF PortWatch 爬蟲
│
├── core/                            # 核心功能
│   ├── collector.py                 # 收集器基類
│   ├── processor.py                 # 資料處理器
│   ├── backfill.py                  # 歷史資料回填
│   └── logger.py                    # 日誌管理
│
├── etl/                             # ETL Pipeline
│   ├── jobs/
│   │   ├── csv_to_postgres.py      # CSV → PostgreSQL
│   │   ├── pg_to_clickhouse.py     # PostgreSQL → ClickHouse
│   │   ├── data_aggregation.py     # 資料聚合
│   │   └── data_cleaning.py        # 資料清洗
│   └── scheduler.py                 # APScheduler 任務調度
│
├── logistics/                       # 航道配置
│   └── chokepoints/
│       ├── bab-el-mandeb/
│       ├── suez-canal/
│       ├── strait-of-hormuz/
│       ├── bosporus-strait/
│       ├── panama-canal/
│       └── strait-of-malacca/
│
├── data/                            # 原始 Pickle 資料
│   └── logistics/chokepoints/
│
├── processed/                       # CSV 備份資料
│   └── logistics/chokepoints/
│       └── {chokepoint}/
│           └── vessel_arrivals/
│               └── vessel_arrivals.csv
│
├── config/                          # 配置文件
│   ├── config.yaml
│   └── database.yaml
│
├── tests/                           # 測試
├── requirements.txt
├── .env.example
└── README.md
```

---

### 2️⃣ SeeSeaIntelligenceAgent（AI 分析與智能代理）

```
SeeSeaIntelligenceAgent/
│
├── app/
│   ├── main.py                      # FastAPI 入口
│   │
│   ├── routers/                     # API 路由
│   │   ├── analytics.py            # 分析 API
│   │   ├── chat.py                 # LangGraph Agent 對話
│   │   └── reports.py              # 報表生成
│   │
│   ├── services/                    # 業務邏輯
│   │   ├── analytics_service.py    # 分析服務
│   │   ├── agent_service.py        # Agent 服務
│   │   └── langgraph_agent.py      # LangGraph Agent 實作
│   │
│   ├── models/                      # Pydantic Models
│   │   └── schemas.py
│   │
│   ├── database/                    # 資料庫連接
│   │   ├── postgres.py
│   │   ├── clickhouse.py
│   │   └── redis.py
│   │
│   └── core/
│       ├── config.py
│       └── logger.py
│
├── agents/                          # LangGraph Agents
│   ├── shipping_analyst.py         # 航運分析 Agent
│   ├── trend_analyzer.py           # 趨勢分析 Agent
│   └── report_generator.py         # 報表生成 Agent
│
├── tools/                           # Agent 工具
│   ├── query_tools.py              # 資料查詢工具
│   ├── analysis_tools.py           # 分析工具
│   └── visualization_tools.py      # 可視化工具
│
├── tests/
├── requirements.txt
├── Dockerfile
└── README.md
```

---

### 3️⃣ SeeSeaIntelligenceAPI（高性能 API）

```
SeeSeaIntelligenceAPI/
│
├── cmd/
│   └── server/
│       └── main.go                  # 程式入口
│
├── internal/
│   ├── handlers/                    # HTTP Handlers
│   │   ├── vessels.go              # 船隻資料 API
│   │   ├── realtime.go             # 即時資料 API
│   │   └── websocket.go            # WebSocket Handler
│   │
│   ├── services/                    # 業務邏輯
│   │   ├── vessel_service.go
│   │   └── cache_service.go
│   │
│   ├── models/                      # 資料模型
│   │   └── vessel.go
│   │
│   ├── database/                    # 資料庫連接
│   │   ├── postgres.go
│   │   ├── clickhouse.go
│   │   └── redis.go
│   │
│   └── middleware/                  # 中介軟體
│       ├── cors.go
│       ├── logger.go
│       └── ratelimit.go
│
├── pkg/                             # 公共套件
│   └── utils/
│
├── configs/                         # 配置文件
│   └── config.yaml
│
├── go.mod
├── go.sum
├── Dockerfile
└── README.md
```

---

### 4️⃣ SeeSeaIntelligenceWeb（前端專案）

```
SeeSeaIntelligenceWeb/
│
├── web/                              # 前端專案 (Next.js)
│   ├── app/                          # App Router
│   │   ├── (dashboard)/             # 儀表板頁面群組
│   │   │   ├── page.tsx            # 總覽頁
│   │   │   ├── chokepoints/        # 航道詳情頁
│   │   │   │   └── [id]/page.tsx
│   │   │   ├── analytics/          # 分析頁面
│   │   │   │   └── page.tsx
│   │   │   └── compare/            # 對比分析頁
│   │   │       └── page.tsx
│   │   ├── api/                    # Next.js API Routes (BFF 層)
│   │   └── layout.tsx
│   │
│   ├── components/
│   │   ├── ui/                     # shadcn/ui 基礎組件
│   │   │   ├── button.tsx
│   │   │   ├── card.tsx
│   │   │   └── ...
│   │   ├── charts/                 # D3.js 圖表組件
│   │   │   ├── TimeSeriesChart.tsx
│   │   │   ├── MultiLineChart.tsx
│   │   │   ├── HeatMap.tsx
│   │   │   ├── StackedAreaChart.tsx
│   │   │   └── hooks/
│   │   │       ├── useD3.ts
│   │   │       └── useChartResize.ts
│   │   ├── map/                    # 地圖組件
│   │   │   ├── ChokepointMap.tsx
│   │   │   ├── VesselLayer.tsx
│   │   │   └── RouteLayer.tsx
│   │   ├── data-display/           # 數據展示組件
│   │   │   ├── MetricsCard.tsx
│   │   │   ├── DataTable.tsx
│   │   │   └── TrendIndicator.tsx
│   │   └── layout/                 # 佈局組件
│   │       ├── Sidebar.tsx
│   │       ├── Header.tsx
│   │       └── DashboardLayout.tsx
│   │
│   ├── lib/
│   │   ├── api/                    # API 客戶端
│   │   │   ├── client.ts
│   │   │   ├── vessels.ts
│   │   │   └── analytics.ts
│   │   ├── hooks/                  # 自定義 Hooks
│   │   │   ├── useVessels.ts
│   │   │   └── useWebSocket.ts
│   │   ├── stores/                 # Zustand Stores
│   │   │   ├── useAppStore.ts
│   │   │   └── useFilterStore.ts
│   │   └── utils/                  # 工具函數
│   │       ├── date.ts
│   │       ├── chart-helpers.ts
│   │       └── data-processing.ts
│   │
│   ├── types/                      # TypeScript 類型定義
│   │   ├── vessel.ts
│   │   ├── chokepoint.ts
│   │   └── api.ts
│   │
│   ├── public/                     # 靜態資源
│   ├── package.json
│   ├── tsconfig.json
│   ├── tailwind.config.ts
│   └── next.config.js
│
├── api-go/                          # Go 後端 (高性能 API)
│   ├── cmd/
│   │   └── server/
│   │       └── main.go             # 程式入口
│   ├── internal/
│   │   ├── handlers/               # HTTP Handlers
│   │   │   ├── vessels.go
│   │   │   ├── realtime.go
│   │   │   └── websocket.go
│   │   ├── services/               # 業務邏輯
│   │   │   ├── vessel_service.go
│   │   │   └── cache_service.go
│   │   ├── models/                 # 資料模型
│   │   │   └── vessel.go
│   │   ├── database/               # 資料庫連接
│   │   │   ├── postgres.go
│   │   │   ├── clickhouse.go
│   │   │   └── redis.go
│   │   └── middleware/             # 中介軟體
│   │       ├── cors.go
│   │       ├── logger.go
│   │       └── ratelimit.go
│   ├── pkg/                        # 公共套件
│   │   └── utils/
│   ├── go.mod
│   ├── go.sum
│   └── Dockerfile
│
├── api-python/                      # Python 後端 (分析 API)
│   ├── app/
│   │   ├── main.py                 # FastAPI 入口
│   │   ├── routers/                # API 路由
│   │   │   ├── analytics.py
│   │   │   ├── chat.py             # LangGraph Agent
│   │   │   └── reports.py
│   │   ├── services/               # 業務邏輯
│   │   │   ├── analytics_service.py
│   │   │   ├── agent_service.py
│   │   │   └── etl_service.py
│   │   ├── models/                 # Pydantic Models
│   │   │   └── schemas.py
│   │   ├── database/               # 資料庫連接
│   │   │   ├── postgres.py
│   │   │   ├── clickhouse.py
│   │   │   └── redis.py
│   │   └── core/
│   │       ├── config.py
│   │       └── logger.py
│   ├── requirements.txt
│   └── Dockerfile
│
├── etl/                             # ETL Pipeline
│   ├── jobs/
│   │   ├── csv_to_postgres.py     # CSV → PostgreSQL
│   │   ├── pg_to_clickhouse.py    # 資料庫同步
│   │   ├── data_aggregation.py    # 資料聚合
│   │   └── data_cleaning.py       # 資料清洗
│   ├── scheduler.py                # APScheduler
│   └── Dockerfile
│
├── infrastructure/                  # 基礎設施配置
│   ├── docker/
│   │   ├── docker-compose.yml
│   │   ├── docker-compose.dev.yml
│   │   └── docker-compose.prod.yml
│   ├── nginx/
│   │   ├── nginx.conf
│   │   └── ssl/
│   ├── database/
│   │   ├── postgres/
│   │   │   ├── init.sql
│   │   │   └── migrations/
│   │   └── clickhouse/
│   │       ├── init.sql
│   │       └── schema.sql
│   └── monitoring/
│       ├── prometheus.yml
│       └── grafana/
│           └── dashboards/
│
├── tests/                           # 測試
├── .env.example
└── README.md
```

---

## 🔄 資料流程與查詢路由

### 資料寫入流程

```python
# 1. 爬蟲收集 (src/scheduler.py)
Scraper → IMF PortWatch API → Pickle Files

# 2. 資料處理 (src/core/processor.py)
Pickle → CSV (processed/) [原始備份]

# 3. 資料庫同步 (etl/jobs/)
CSV → PostgreSQL (即時寫入)
PostgreSQL → ClickHouse (每日凌晨 2:00)
```

### API 查詢路由決策

```python
# api-go/internal/handlers/router.go

def route_query(query_params):
    """智能路由：根據查詢特性決定用哪個資料庫"""

    time_range_days = (end_date - start_date).days

    # 決策邏輯
    if time_range_days <= 7:
        # 小範圍查詢 → PostgreSQL (< 10ms)
        return query_postgres(query_params)

    elif time_range_days <= 30 and not needs_aggregation:
        # 中等範圍、簡單查詢 → PostgreSQL
        return query_postgres(query_params)

    elif needs_complex_aggregation:
        # 複雜聚合（AVG、PERCENTILE、窗口函數）→ ClickHouse
        return query_clickhouse(query_params)

    else:
        # 大範圍歷史查詢 → ClickHouse
        return query_clickhouse(query_params)
```

**路由規則總結：**

| 查詢類型 | 路由目標 | 理由 |
|---------|---------|------|
| 時間範圍 < 7 天 | PostgreSQL | 熱數據，極快 |
| 時間範圍 7-30 天 | PostgreSQL | 仍在記憶體，快速 |
| 時間範圍 > 90 天 | ClickHouse | 歷史數據，列式儲存優化 |
| 複雜聚合 (AVG, SUM) | ClickHouse | 聚合查詢優化 |
| 多維度分析 | ClickHouse | OLAP 專用 |
| 單筆 CRUD | PostgreSQL | 行式儲存適合 |
| 即時寫入 | PostgreSQL | ACID 保證 |

---

## 🎨 前端技術實作

### D3.js 圖表組件範例

```typescript
// web/components/charts/TimeSeriesChart.tsx

import { useD3 } from '@/lib/hooks/useD3';
import { scaleTime, scaleLinear } from 'd3-scale';
import { line } from 'd3-shape';
import { axisBottom, axisLeft } from 'd3-axis';
import { extent, max } from 'd3-array';

interface VesselData {
  date: string;
  vessel_count: number;
}

interface TimeSeriesChartProps {
  data: VesselData[];
  width?: number;
  height?: number;
}

export function TimeSeriesChart({
  data,
  width = 800,
  height = 400
}: TimeSeriesChartProps) {
  const ref = useD3(
    (svg) => {
      const margin = { top: 20, right: 30, bottom: 30, left: 40 };
      const innerWidth = width - margin.left - margin.right;
      const innerHeight = height - margin.top - margin.bottom;

      // Scales
      const xScale = scaleTime()
        .domain(extent(data, d => new Date(d.date)) as [Date, Date])
        .range([0, innerWidth]);

      const yScale = scaleLinear()
        .domain([0, max(data, d => d.vessel_count) || 0])
        .range([innerHeight, 0]);

      // Line generator
      const lineGenerator = line<VesselData>()
        .x(d => xScale(new Date(d.date)))
        .y(d => yScale(d.vessel_count));

      // Clear previous content
      svg.selectAll('*').remove();

      const g = svg
        .append('g')
        .attr('transform', `translate(${margin.left},${margin.top})`);

      // Render path
      g.append('path')
        .datum(data)
        .attr('fill', 'none')
        .attr('stroke', 'steelblue')
        .attr('stroke-width', 2)
        .attr('d', lineGenerator);

      // X Axis
      g.append('g')
        .attr('transform', `translate(0,${innerHeight})`)
        .call(axisBottom(xScale));

      // Y Axis
      g.append('g')
        .call(axisLeft(yScale));
    },
    [data, width, height]
  );

  return (
    <svg
      ref={ref}
      width={width}
      height={height}
      className="border rounded-lg bg-white shadow-sm"
    />
  );
}
```

### 地圖可視化組件範例

```typescript
// web/components/map/ChokepointMap.tsx

import Map from 'react-map-gl';
import DeckGL from '@deck.gl/react';
import { ScatterplotLayer, ArcLayer } from '@deck.gl/layers';
import 'mapbox-gl/dist/mapbox-gl.css';

const CHOKEPOINTS = [
  { name: 'Suez Canal', coordinates: [32.3, 30.5], vessels: 1250 },
  { name: 'Strait of Hormuz', coordinates: [56.3, 26.5], vessels: 980 },
  { name: 'Strait of Malacca', coordinates: [100.4, 2.5], vessels: 1580 },
  { name: 'Panama Canal', coordinates: [-79.9, 9.0], vessels: 750 },
  { name: 'Bosporus Strait', coordinates: [29.0, 41.1], vessels: 650 },
  { name: 'Bab el-Mandeb', coordinates: [43.3, 12.6], vessels: 520 }
];

export function ChokepointMap() {
  const layers = [
    new ScatterplotLayer({
      id: 'chokepoints',
      data: CHOKEPOINTS,
      getPosition: d => d.coordinates,
      getRadius: d => d.vessels * 50,
      getFillColor: [255, 140, 0, 200],
      pickable: true,
      onHover: info => {
        if (info.object) {
          console.log(`${info.object.name}: ${info.object.vessels} vessels/day`);
        }
      }
    })
  ];

  return (
    <DeckGL
      initialViewState={{
        longitude: 30,
        latitude: 20,
        zoom: 3,
        pitch: 0,
        bearing: 0
      }}
      controller={true}
      layers={layers}
      style={{ width: '100%', height: '600px' }}
    >
      <Map
        mapboxAccessToken={process.env.NEXT_PUBLIC_MAPBOX_TOKEN}
        mapStyle="mapbox://styles/mapbox/dark-v11"
      />
    </DeckGL>
  );
}
```

---

## ⚙️ 後端技術實作

### Go API 範例

```go
// api-go/internal/handlers/vessels.go

package handlers

import (
    "fmt"
    "net/http"
    "time"
    "github.com/gin-gonic/gin"
    "github.com/go-redis/redis/v8"
    "database/sql"
)

type VesselHandler struct {
    cache   *redis.Client
    db      *sql.DB
}

// GetVessels - 高性能查詢 API
func (h *VesselHandler) GetVessels(c *gin.Context) {
    chokepoint := c.Param("chokepoint")
    startDate := c.Query("start_date")
    endDate := c.Query("end_date")

    // 1. 先查 Redis 快取
    cacheKey := fmt.Sprintf("vessels:%s:%s:%s", chokepoint, startDate, endDate)

    if cached, err := h.cache.Get(c.Request.Context(), cacheKey).Result(); err == nil {
        c.Header("X-Cache", "HIT")
        c.JSON(http.StatusOK, gin.H{"data": cached, "source": "cache"})
        return
    }

    // 2. 快取未命中，查詢資料庫
    query := `
        SELECT date, vessel_count, container, dry_bulk, general_cargo, roro, tanker
        FROM vessel_arrivals
        WHERE chokepoint = $1
          AND date >= $2
          AND date <= $3
        ORDER BY date ASC
    `

    rows, err := h.db.Query(query, chokepoint, startDate, endDate)
    if err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
        return
    }
    defer rows.Close()

    // 3. 組裝結果
    var vessels []map[string]interface{}
    for rows.Next() {
        var v map[string]interface{}
        // ... scan rows
        vessels = append(vessels, v)
    }

    // 4. 寫入快取 (TTL: 5 分鐘)
    h.cache.Set(c.Request.Context(), cacheKey, vessels, 5*time.Minute)

    c.Header("X-Cache", "MISS")
    c.JSON(http.StatusOK, gin.H{"data": vessels, "source": "database"})
}

// StreamVessels - WebSocket 即時推送
func (h *VesselHandler) StreamVessels(c *gin.Context) {
    ws, err := upgrader.Upgrade(c.Writer, c.Request, nil)
    if err != nil {
        return
    }
    defer ws.Close()

    ticker := time.NewTicker(10 * time.Second)
    defer ticker.Stop()

    for {
        select {
        case <-ticker.C:
            // 從 Redis Pub/Sub 獲取最新數據
            data := h.getLatestData()
            if err := ws.WriteJSON(data); err != nil {
                return
            }
        }
    }
}
```

### Python FastAPI 範例

```python
# api-python/app/routers/analytics.py

from fastapi import APIRouter, Depends, HTTPException
from clickhouse_driver import Client
from typing import List, Dict
import pandas as pd

router = APIRouter()

@router.get("/analytics/trend")
async def get_trend(
    chokepoint: str,
    years: int = 5,
    ch: Client = Depends(get_clickhouse_client)
):
    """
    多年趨勢分析 - 使用 ClickHouse

    查詢 2019-2024 年的每月平均船隻數
    """
    query = """
        SELECT
            toStartOfMonth(date) as month,
            avg(vessel_count) as avg_vessels,
            quantile(0.5)(vessel_count) as median_vessels,
            quantile(0.95)(vessel_count) as p95_vessels,
            sum(container + dry_bulk + tanker) as total_cargo
        FROM vessel_arrivals_analytics
        WHERE chokepoint = %(chokepoint)s
          AND date >= today() - INTERVAL %(years)s YEAR
        GROUP BY month
        ORDER BY month
    """

    result = ch.execute(
        query,
        {'chokepoint': chokepoint, 'years': years},
        with_column_types=True
    )

    # 轉換為 DataFrame
    columns = [col[0] for col in result[1]]
    df = pd.DataFrame(result[0], columns=columns)

    return {
        "chokepoint": chokepoint,
        "period": f"last_{years}_years",
        "data": df.to_dict("records")
    }

@router.post("/analytics/compare")
async def compare_chokepoints(
    chokepoints: List[str],
    metric: str = "vessel_count",
    ch: Client = Depends(get_clickhouse_client)
):
    """多航道對比分析"""

    query = """
        SELECT
            chokepoint,
            avg({metric}) as avg_value,
            stddevPop({metric}) as stddev_value,
            quantiles(0.25, 0.5, 0.75)({metric}) as quartiles
        FROM vessel_arrivals_analytics
        WHERE chokepoint IN %(chokepoints)s
          AND date >= today() - INTERVAL 1 YEAR
        GROUP BY chokepoint
    """.format(metric=metric)

    result = ch.execute(query, {'chokepoints': chokepoints})

    return {
        "metric": metric,
        "comparison": result
    }
```

---

## 🗄️ 資料庫設計

### PostgreSQL Schema (OLTP)

```sql
-- 即時資料表
CREATE TABLE vessel_arrivals (
    id BIGSERIAL PRIMARY KEY,
    date DATE NOT NULL,
    chokepoint VARCHAR(50) NOT NULL,
    vessel_count INTEGER NOT NULL,
    container INTEGER DEFAULT 0,
    dry_bulk INTEGER DEFAULT 0,
    general_cargo INTEGER DEFAULT 0,
    roro INTEGER DEFAULT 0,
    tanker INTEGER DEFAULT 0,
    collected_at TIMESTAMPTZ DEFAULT NOW(),
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),

    CONSTRAINT unique_date_chokepoint UNIQUE(date, chokepoint)
);

-- TimescaleDB 時間序列優化
SELECT create_hypertable('vessel_arrivals', 'date',
    chunk_time_interval => INTERVAL '1 month'
);

-- 索引優化
CREATE INDEX idx_chokepoint_date ON vessel_arrivals(chokepoint, date DESC);
CREATE INDEX idx_date ON vessel_arrivals(date DESC);
CREATE INDEX idx_collected_at ON vessel_arrivals(collected_at DESC);

-- 物化視圖（預聚合，加速常見查詢）
CREATE MATERIALIZED VIEW daily_summary AS
SELECT
    date,
    chokepoint,
    vessel_count,
    (container + dry_bulk + tanker) AS total_cargo_vessels,
    ROUND(vessel_count::numeric / NULLIF((container + dry_bulk + general_cargo + roro + tanker), 0) * 100, 2) as data_completeness
FROM vessel_arrivals
WITH DATA;

CREATE INDEX ON daily_summary(chokepoint, date DESC);

-- 自動刷新物化視圖（每小時）
CREATE OR REPLACE FUNCTION refresh_daily_summary()
RETURNS void AS $$
BEGIN
    REFRESH MATERIALIZED VIEW CONCURRENTLY daily_summary;
END;
$$ LANGUAGE plpgsql;

-- 航道元數據表
CREATE TABLE chokepoints (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL,
    display_name VARCHAR(100),
    latitude DECIMAL(10, 7),
    longitude DECIMAL(10, 7),
    description TEXT,
    importance_level INTEGER,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

INSERT INTO chokepoints (name, display_name, latitude, longitude, importance_level) VALUES
('suez-canal', 'Suez Canal', 30.5, 32.3, 5),
('strait-of-hormuz', 'Strait of Hormuz', 26.5, 56.3, 5),
('strait-of-malacca', 'Strait of Malacca', 2.5, 100.4, 5),
('panama-canal', 'Panama Canal', 9.0, -79.9, 4),
('bosporus-strait', 'Bosporus Strait', 41.1, 29.0, 3),
('bab-el-mandeb', 'Bab el-Mandeb', 12.6, 43.3, 4);
```

### ClickHouse Schema (OLAP)

```sql
-- 分析資料表（列式儲存）
CREATE TABLE vessel_arrivals_analytics (
    date Date,
    chokepoint LowCardinality(String),
    vessel_count UInt32,
    container UInt16,
    dry_bulk UInt16,
    general_cargo UInt16,
    roro UInt16,
    tanker UInt16,
    collected_at DateTime
) ENGINE = MergeTree()
PARTITION BY toYYYYMM(date)
ORDER BY (chokepoint, date)
SETTINGS index_granularity = 8192;

-- 預聚合表（月度統計）
CREATE MATERIALIZED VIEW monthly_summary_mv
ENGINE = SummingMergeTree()
PARTITION BY toYear(month)
ORDER BY (chokepoint, month)
AS SELECT
    toStartOfMonth(date) as month,
    chokepoint,
    sum(vessel_count) as total_vessels,
    avg(vessel_count) as avg_vessels,
    max(vessel_count) as peak_vessels,
    min(vessel_count) as min_vessels
FROM vessel_arrivals_analytics
GROUP BY month, chokepoint;

-- 週度統計視圖
CREATE MATERIALIZED VIEW weekly_summary_mv
ENGINE = SummingMergeTree()
ORDER BY (chokepoint, week)
AS SELECT
    toStartOfWeek(date) as week,
    chokepoint,
    sum(vessel_count) as total_vessels,
    avg(vessel_count) as avg_vessels,
    sumIf(container, container > 0) as total_containers,
    sumIf(tanker, tanker > 0) as total_tankers
FROM vessel_arrivals_analytics
GROUP BY week, chokepoint;

-- 字典（加速查詢）
CREATE DICTIONARY chokepoints_dict (
    name String,
    display_name String,
    latitude Float64,
    longitude Float64
)
PRIMARY KEY name
SOURCE(POSTGRESQL(
    host 'postgres'
    port 5432
    user 'admin'
    password 'password'
    db 'seesea'
    table 'chokepoints'
))
LIFETIME(MIN 3600 MAX 7200)
LAYOUT(FLAT());
```

---

## 🐳 Docker 部署配置

### Docker Compose (完整配置)

```yaml
# infrastructure/docker/docker-compose.yml

version: '3.9'

services:
  # Nginx 反向代理
  nginx:
    image: nginx:alpine
    container_name: seesea-nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./infrastructure/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - /etc/letsencrypt:/etc/letsencrypt:ro
    depends_on:
      - api-go
      - api-python
    restart: unless-stopped
    networks:
      - seesea-network

  # SeeSeaIntelligenceAPI (Go - 高性能查詢)
  api-go:
    build:
      context: ../SeeSeaIntelligenceAPI
      dockerfile: Dockerfile
    container_name: seesea-api-go
    ports:
      - "8080:8080"
    environment:
      - GIN_MODE=release
      - DATABASE_URL=postgresql://admin:${POSTGRES_PASSWORD}@postgres:5432/seesea
      - CLICKHOUSE_URL=http://clickhouse:8123
      - REDIS_URL=redis://redis:6379
    depends_on:
      - postgres
      - clickhouse
      - redis
    restart: unless-stopped
    networks:
      - seesea-network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # SeeSeaIntelligenceAgent (Python - AI 分析服務)
  api-python:
    build:
      context: ../SeeSeaIntelligenceAgent
      dockerfile: Dockerfile
    container_name: seesea-api-python
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://admin:${POSTGRES_PASSWORD}@postgres:5432/seesea
      - CLICKHOUSE_URL=http://clickhouse:8123
      - REDIS_URL=redis://redis:6379
      - GEMINI_API_KEY=${GEMINI_API_KEY}
    depends_on:
      - postgres
      - clickhouse
      - redis
    restart: unless-stopped
    networks:
      - seesea-network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # SeeSeaIntelligence (資料收集與 ETL)
  data-collector:
    build:
      context: ../SeeSeaIntelligence
      dockerfile: Dockerfile
    container_name: seesea-data-collector
    environment:
      - POSTGRES_URL=postgresql://admin:${POSTGRES_PASSWORD}@postgres:5432/seesea
      - CLICKHOUSE_URL=http://clickhouse:8123
    volumes:
      - ../SeeSeaIntelligence/data:/app/data
      - ../SeeSeaIntelligence/processed:/app/processed
    depends_on:
      - postgres
      - clickhouse
    restart: unless-stopped
    networks:
      - seesea-network

  # PostgreSQL + TimescaleDB
  postgres:
    image: timescale/timescaledb:latest-pg16
    container_name: seesea-postgres
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_DB=seesea
      - POSTGRES_USER=admin
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - PGDATA=/var/lib/postgresql/data/pgdata
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./infrastructure/database/postgres/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    restart: unless-stopped
    networks:
      - seesea-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U admin -d seesea"]
      interval: 10s
      timeout: 5s
      retries: 5

  # ClickHouse
  clickhouse:
    image: clickhouse/clickhouse-server:latest
    container_name: seesea-clickhouse
    ports:
      - "8123:8123"
      - "9000:9000"
    environment:
      - CLICKHOUSE_DB=seesea_analytics
      - CLICKHOUSE_USER=admin
      - CLICKHOUSE_PASSWORD=${CLICKHOUSE_PASSWORD}
    volumes:
      - clickhouse_data:/var/lib/clickhouse
      - ./infrastructure/database/clickhouse/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    restart: unless-stopped
    networks:
      - seesea-network
    ulimits:
      nofile:
        soft: 262144
        hard: 262144

  # Redis
  redis:
    image: redis:7-alpine
    container_name: seesea-redis
    ports:
      - "6379:6379"
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis_data:/data
    restart: unless-stopped
    networks:
      - seesea-network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3


  # Prometheus 監控
  prometheus:
    image: prom/prometheus:latest
    container_name: seesea-prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./infrastructure/monitoring/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
    restart: unless-stopped
    networks:
      - seesea-network

  # Grafana 視覺化
  grafana:
    image: grafana/grafana:latest
    container_name: seesea-grafana
    ports:
      - "3001:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD}
      - GF_INSTALL_PLUGINS=redis-datasource,clickhouse-datasource
    volumes:
      - grafana_data:/var/lib/grafana
      - ./infrastructure/monitoring/grafana/dashboards:/etc/grafana/provisioning/dashboards:ro
    depends_on:
      - prometheus
    restart: unless-stopped
    networks:
      - seesea-network

volumes:
  postgres_data:
  clickhouse_data:
  redis_data:
  prometheus_data:
  grafana_data:

networks:
  seesea-network:
    driver: bridge
```

### Nginx 配置

```nginx
# infrastructure/nginx/nginx.conf

upstream go_api {
    server api-go:8080;
    keepalive 32;
}

upstream python_api {
    server api-python:8000;
    keepalive 16;
}

# HTTP → HTTPS 重定向
server {
    listen 80;
    server_name api.seesea.ai ws.seesea.ai;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://$server_name$request_uri;
    }
}

# HTTPS 主配置
server {
    listen 443 ssl http2;
    server_name api.seesea.ai;

    # SSL 憑證 (Let's Encrypt)
    ssl_certificate /etc/letsencrypt/live/api.seesea.ai/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.seesea.ai/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    # 安全標頭
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Gzip 壓縮
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml;

    # Go API (資料查詢)
    location /api/v1/vessels {
        proxy_pass http://go_api;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 快取配置
        add_header X-Cache-Status $upstream_cache_status;
    }

    # Python API (分析)
    location /api/v1/analytics {
        proxy_pass http://python_api;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 較長的超時（分析查詢可能較慢）
        proxy_read_timeout 300s;
        proxy_connect_timeout 75s;
    }

    # LangGraph Agent
    location /api/v1/chat {
        proxy_pass http://python_api;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_read_timeout 300s;
    }

    # 健康檢查
    location /health {
        access_log off;
        return 200 "OK\n";
        add_header Content-Type text/plain;
    }

    # 速率限制
    limit_req_zone $binary_remote_addr zone=api_limit:10m rate=100r/m;
    limit_req zone=api_limit burst=20 nodelay;
}

# WebSocket 配置
server {
    listen 443 ssl http2;
    server_name ws.seesea.ai;

    ssl_certificate /etc/letsencrypt/live/ws.seesea.ai/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/ws.seesea.ai/privkey.pem;

    location /ws {
        proxy_pass http://go_api;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;

        # WebSocket 超時
        proxy_read_timeout 86400s;
        proxy_send_timeout 86400s;
    }
}
```

---

## 🌐 GoDaddy DNS 設定

### DNS 記錄配置

**假設您的 EC2 IP: `54.123.45.67`**

登入 GoDaddy DNS 管理，新增以下記錄：

| Type | Name | Value | TTL |
|------|------|-------|-----|
| **CNAME** | @ | cname.vercel-dns.com | 600 |
| **CNAME** | www | cname.vercel-dns.com | 600 |
| **A** | api | 54.123.45.67 | 600 |
| **A** | ws | 54.123.45.67 | 600 |

**結果：**
- `https://seesea.ai` → Vercel 前端 (全球 CDN)
- `https://www.seesea.ai` → Vercel 前端
- `https://api.seesea.ai` → EC2 後端 API
- `wss://ws.seesea.ai` → EC2 WebSocket

---

## 🚀 部署步驟

### Step 1: 設定 GoDaddy DNS

1. 登入 GoDaddy
2. 進入 DNS 管理
3. 新增上述 DNS 記錄
4. 儲存

### Step 2: 設定 EC2 主機

```bash
# SSH 連接到 EC2
ssh -i your-key.pem ubuntu@54.123.45.67

# 更新系統
sudo apt update && sudo apt upgrade -y

# 安裝 Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER

# 安裝 Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# 安裝 Certbot (SSL)
sudo apt install certbot -y

# 安裝基本工具
sudo apt install git curl wget nano -y

# 重新登入（使 Docker 權限生效）
exit
ssh -i your-key.pem ubuntu@54.123.45.67
```

### Step 3: Clone 專案並設定環境

```bash
# 建立專案目錄
mkdir -p ~/SeeSea
cd ~/SeeSea

# Clone 所有專案
git clone https://github.com/your-org/SeeSeaIntelligence.git
git clone https://github.com/your-org/SeeSeaIntelligenceAgent.git
git clone https://github.com/your-org/SeeSeaIntelligenceAPI.git
git clone https://github.com/your-org/SeeSeaIntelligenceWeb.git
git clone https://github.com/your-org/SeeSeaIntelligenceDocs.git

# 設定環境變數（每個專案都需要）
cd SeeSeaIntelligence && cp .env.example .env && cd ..
cd SeeSeaIntelligenceAgent && cp .env.example .env && cd ..
cd SeeSeaIntelligenceAPI && cp .env.example .env && cd ..
cd SeeSeaIntelligenceWeb && cp .env.example .env && cd ..

# 編輯共用環境變數
nano ~/SeeSea/.env.shared

# 填入以下內容:
# POSTGRES_PASSWORD=your_secure_password_here
# CLICKHOUSE_PASSWORD=your_secure_password_here
# REDIS_PASSWORD=your_secure_password_here
# GRAFANA_PASSWORD=your_secure_password_here
# GEMINI_API_KEY=your_gemini_api_key_here
# MAPBOX_TOKEN=your_mapbox_token_here
```

### Step 4: 取得 SSL 憑證

```bash
# 停止 Docker（避免 Port 80 衝突）
docker-compose down

# 取得憑證
sudo certbot certonly --standalone \
  -d api.seesea.ai \
  -d ws.seesea.ai \
  --email your-email@example.com \
  --agree-tos \
  --non-interactive

# 憑證會儲存在 /etc/letsencrypt/live/

# 設定自動續約
sudo crontab -e
# 加入此行（每天凌晨 2 點檢查）:
0 2 * * * certbot renew --quiet --post-hook "cd /home/ubuntu/SeeSeaAgent && docker-compose restart nginx"
```

### Step 5: 啟動後端服務

```bash
cd ~/SeeSea/SeeSeaIntelligenceDocs/infrastructure/docker

# 啟動所有服務（會自動建置所有專案）
docker-compose up -d

# 查看狀態
docker-compose ps

# 查看日誌
docker-compose logs -f

# 檢查健康狀態
curl http://localhost:8080/health  # SeeSeaIntelligenceAPI (Go)
curl http://localhost:8000/health  # SeeSeaIntelligenceAgent (Python)
```

### Step 6: 初始化資料庫

```bash
# 導入歷史 CSV 資料到 PostgreSQL（在 SeeSeaIntelligence 容器中執行）
docker-compose exec data-collector python /app/etl/jobs/csv_to_postgres.py --full

# 同步到 ClickHouse
docker-compose exec data-collector python /app/etl/jobs/pg_to_clickhouse.py --full

# 驗證資料
docker-compose exec postgres psql -U admin -d seesea -c "SELECT COUNT(*) FROM vessel_arrivals;"
docker-compose exec clickhouse clickhouse-client --query "SELECT COUNT(*) FROM vessel_arrivals_analytics;"
```

### Step 7: 部署前端到 Vercel

```bash
# 在本地電腦（不是 EC2）

cd SeeSeaIntelligenceWeb

# 設定生產環境變數
cat > .env.production << EOF
NEXT_PUBLIC_API_URL=https://api.seesea.ai
NEXT_PUBLIC_WS_URL=wss://ws.seesea.ai
NEXT_PUBLIC_MAPBOX_TOKEN=your_mapbox_token_here
EOF

# 安裝 Vercel CLI
npm install -g vercel

# 登入 Vercel
vercel login

# 部署
vercel --prod

# 設定自訂域名
# 1. 在 Vercel Dashboard 進入專案
# 2. Settings → Domains
# 3. 加入: seesea.ai 和 www.seesea.ai
# 4. Vercel 會自動驗證 DNS
# 5. 等待 5-30 分鐘生效
```

### Step 8: 驗證部署

```bash
# 測試後端 API
curl https://api.seesea.ai/health
curl https://api.seesea.ai/api/v1/vessels/suez-canal?start_date=2024-01-01&end_date=2024-01-31

# 測試前端
curl -I https://seesea.ai

# 測試 WebSocket（使用 wscat）
npm install -g wscat
wscat -c wss://ws.seesea.ai/ws
```

---

## 💰 成本分析

### 總成本預估

```yaml
AWS EC2 (t3.medium):
  vCPU: 2
  記憶體: 4 GB
  儲存: 50 GB SSD
  費用: ~$30/月

GoDaddy 域名 (seesea.ai):
  費用: ~$10/年

Vercel 前端:
  費用: 免費（Hobby Plan）
  包含: 100GB 頻寬、全球 CDN、自動 HTTPS

Let's Encrypt SSL:
  費用: 免費

總計: ~$30/月 + $10/年 ≈ $31/月
```

### 擴展後成本（正式環境）

```yaml
EC2 (t3.large):
  vCPU: 2
  記憶體: 8 GB
  費用: ~$60/月

Vercel Pro:
  費用: $20/月
  包含: 1TB 頻寬、優先支援

總計: ~$80/月
```

---

## 📊 性能指標目標

### 前端性能

```yaml
Core Web Vitals:
  LCP (Largest Contentful Paint): < 1.5s
  FID (First Input Delay): < 100ms
  CLS (Cumulative Layout Shift): < 0.1

圖表渲染:
  10,000 點: < 200ms
  100,000 點: < 1s (使用降採樣)

頁面載入:
  首屏: < 1.5s
  互動就緒: < 2s
```

### 後端性能

```yaml
API 響應時間 (P95):
  簡單查詢 (Go): < 10ms
  複雜查詢 (Python): < 100ms

資料庫查詢:
  PostgreSQL: < 20ms
  ClickHouse 聚合: < 50ms

Redis 快取:
  命中率: > 80%
  延遲: < 1ms

併發支援:
  1,000 req/s (正常)
  5,000 req/s (峰值)

WebSocket:
  延遲: < 50ms
  併發連接: 10,000+
```

---

## 🔐 安全性設計

### API 安全

```yaml
認證授權:
  - JWT Token 驗證
  - API Key 認證
  - OAuth 2.0 (未來)

速率限制:
  - 每分鐘 100 次請求
  - 每小時 1,000 次請求
  - IP 黑名單

輸入驗證:
  - 參數化查詢（防 SQL Injection）
  - 輸入清理
  - CORS 配置
```

### 資料安全

```yaml
資料庫:
  - SSL 連接加密
  - 密碼強度要求
  - 定期備份（每日）

敏感資料:
  - 環境變數管理
  - 不記錄敏感資訊
  - API Key 輪換
```

### 網路安全

```yaml
HTTPS:
  - 強制 HTTPS
  - TLS 1.2+
  - HSTS 標頭

防護:
  - DDoS 防護（Cloudflare 可選）
  - 防火牆規則（EC2 Security Group）
  - 定期安全更新
```

---

## 📈 監控與維運

### Grafana 監控指標

```yaml
系統指標:
  - CPU 使用率
  - 記憶體使用率
  - 磁碟 I/O
  - 網路流量

應用指標:
  - API 請求數 (per second)
  - API 響應時間 (P50, P95, P99)
  - 錯誤率
  - 資料庫連接數
  - Redis 快取命中率

業務指標:
  - 活躍使用者數
  - 熱門查詢
  - 資料更新延遲
```

### 告警規則

```yaml
嚴重告警:
  - API 錯誤率 > 5%
  - 資料庫連接失敗
  - 磁碟空間 < 10%
  - 服務無回應

警告告警:
  - API P95 延遲 > 100ms
  - CPU 使用率 > 80%
  - 記憶體使用率 > 85%
  - Redis 快取命中率 < 70%
```

---

## 🎯 階段式部署計劃

### Phase 1: MVP（Week 1-4）

```yaml
目標: 快速上線，驗證產品

實作內容:
  後端:
    ✅ FastAPI (單一後端)
    ✅ PostgreSQL + TimescaleDB
    ✅ 基本 CRUD API

  前端:
    ✅ Next.js 基礎框架
    ✅ D3.js 基礎圖表（折線圖、柱狀圖）
    ✅ 基本資料展示

  資料:
    ✅ CSV → PostgreSQL ETL
    ✅ 歷史資料導入

部署:
  ✅ 前端 Vercel
  ✅ 後端 EC2
  ✅ PostgreSQL

成本: ~$30/月
```

### Phase 2: 性能優化（Week 5-8）

```yaml
目標: 提升性能，支援更多使用者

實作內容:
  後端:
    ✅ 加入 Go API (高頻查詢)
    ✅ 加入 ClickHouse (歷史分析)
    ✅ Redis 快取層
    ✅ Nginx 反向代理

  前端:
    ✅ 圖表性能優化（降採樣、Canvas 渲染）
    ✅ TanStack Query 快取
    ✅ 虛擬滾動表格

  資料:
    ✅ PostgreSQL → ClickHouse 同步
    ✅ 資料分區優化

效能提升:
  - API 響應時間: 100ms → 10ms
  - 複雜查詢: 2s → 100ms
  - 併發支援: 100 → 1,000 req/s
```

### Phase 3: 完整功能（Week 9-12）

```yaml
目標: 完整產品功能

實作內容:
  可視化:
    ✅ Mapbox GL 地圖
    ✅ Deck.gl 航道可視化
    ✅ 複雜互動圖表（熱力圖、Sankey 圖）

  即時功能:
    ✅ WebSocket 即時推送
    ✅ 即時儀表板更新

  AI 功能:
    ✅ LangGraph Agent 整合
    ✅ 自然語言查詢

  維運:
    ✅ Grafana 監控
    ✅ Prometheus 指標
    ✅ 自動告警

完成指標:
  - ✅ 全球 CDN 前端
  - ✅ 高性能混合後端
  - ✅ 雙資料庫架構
  - ✅ 即時監控系統
```

---

## 📚 參考文件

### 官方文件

- [Next.js Documentation](https://nextjs.org/docs)
- [React Documentation](https://react.dev)
- [D3.js Documentation](https://d3js.org/)
- [Visx Documentation](https://airbnb.io/visx/)
- [Mapbox GL JS](https://docs.mapbox.com/mapbox-gl-js/)
- [Deck.gl](https://deck.gl/)
- [Gin Web Framework](https://gin-gonic.com/docs/)
- [FastAPI](https://fastapi.tiangolo.com/)
- [PostgreSQL](https://www.postgresql.org/docs/)
- [TimescaleDB](https://docs.timescale.com/)
- [ClickHouse](https://clickhouse.com/docs/)
- [Redis](https://redis.io/docs/)
- [Docker](https://docs.docker.com/)
- [Nginx](https://nginx.org/en/docs/)

### 部署相關

- [Vercel Documentation](https://vercel.com/docs)
- [AWS EC2](https://docs.aws.amazon.com/ec2/)
- [Let's Encrypt](https://letsencrypt.org/docs/)
- [GoDaddy DNS](https://www.godaddy.com/help/dns-management-19179)

---

## 🛠️ 快速命令參考

### 本地開發

```bash
# 啟動後端 (Docker) - 在 SeeSeaIntelligenceDocs 專案中
cd ~/SeeSea/SeeSeaIntelligenceDocs/infrastructure/docker
docker-compose up -d

# 啟動前端 - 在 SeeSeaIntelligenceWeb 專案中
cd ~/SeeSea/SeeSeaIntelligenceWeb
npm run dev

# 查看日誌
cd ~/SeeSea/SeeSeaIntelligenceDocs/infrastructure/docker
docker-compose logs -f api-go           # SeeSeaIntelligenceAPI
docker-compose logs -f api-python       # SeeSeaIntelligenceAgent
docker-compose logs -f data-collector   # SeeSeaIntelligence

# 停止服務
docker-compose down
```

### 生產環境

```bash
# SSH 到 EC2
ssh -i your-key.pem ubuntu@54.123.45.67

# 更新所有專案程式碼
cd ~/SeeSea
git -C SeeSeaIntelligence pull origin main
git -C SeeSeaIntelligenceAgent pull origin main
git -C SeeSeaIntelligenceAPI pull origin main
git -C SeeSeaIntelligenceDocs pull origin main

# 重啟服務
cd ~/SeeSea/SeeSeaIntelligenceDocs/infrastructure/docker
docker-compose down
docker-compose up -d --build

# 查看狀態
docker-compose ps

# 備份資料庫
docker-compose exec postgres pg_dump -U admin seesea > ~/backups/backup_$(date +%Y%m%d).sql
```

### SSL 續約

```bash
# 手動續約
sudo certbot renew

# 重啟 Nginx
docker-compose restart nginx

# 查看憑證有效期
sudo certbot certificates
```

---

## ✅ 檢查清單

### 部署前檢查

- [ ] EC2 主機已啟動並可 SSH 連接
- [ ] GoDaddy DNS 已設定
- [ ] .env 環境變數已配置
- [ ] SSL 憑證已取得
- [ ] Docker 服務正常運行
- [ ] 資料庫已初始化
- [ ] Vercel 前端已部署
- [ ] 防火牆規則已設定（Port 80, 443, 22）

### 上線後檢查

- [ ] https://seesea.ai 可正常訪問
- [ ] https://api.seesea.ai/health 回應 OK
- [ ] WebSocket 連接正常
- [ ] 監控系統運作中
- [ ] 自動備份已設定
- [ ] SSL 自動續約已設定

---

## 👥 團隊與維護

**架構設計:** 2026-02-07
**技術選型:** 高性能、可擴展、成本優化
**預期用戶:** 全球航運情報分析師、研究人員

**維護計劃:**
- 每日自動備份
- 每週安全更新
- 每月性能檢視
- 季度架構審查

---

**Version:** 3.0.0 (Multi-Repo Architecture)
**Last Updated:** 2026-02-07
**Domain:** https://seesea.ai
**License:** Proprietary

**專案倉庫:**
- SeeSeaIntelligence: 資料收集與處理
- SeeSeaIntelligenceAgent: AI 分析與智能代理
- SeeSeaIntelligenceAPI: 高性能資料查詢 API
- SeeSeaIntelligenceWeb: 前端應用
- SeeSeaIntelligenceDocs: 文件與基礎設施配置

---

**🚀 準備好開始建構了嗎？請參考各專案的 README.md 開始開發！**
