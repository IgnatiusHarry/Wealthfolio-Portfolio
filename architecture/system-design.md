# 🏗️ System Architecture — WealthFolio

> ⚠️ **Note:** All data, metrics, and examples in this document use **fictional dummy data** for portfolio demonstration only.

## Overview

WealthFolio is designed as a **three-layer data platform**:

1. **Ingestion Layer** — Multiple input channels (Discord, web, CSV)
2. **Processing Layer** — AI parsing, FX normalization, daily snapshots
3. **Presentation Layer** — Dashboard, AI insights, alerts

---

## Full System Design

```mermaid
graph TB
    subgraph Users["👤 User Interaction"]
        U1["Web Browser"]
        U2["Discord App"]
        U3["Discord Server"]
    end

    subgraph Ingestion["📥 Ingestion Layer"]
        BOT["Discord Bot<br/>Node.js"]
        WEB["Web Dashboard<br/>Next.js 14"]
        CSV["CSV Importer"]
    end

    subgraph Backend["⚙️ Backend — Next.js API Routes"]
        API_TX["/api/transactions"]
        API_DASH["/api/dashboard-data"]
        API_SNAP["/api/portfolio-snapshot"]
        API_AI["/api/ai-insights"]
        API_SET["/api/settings"]
        CRON["Cron Scheduler<br/>Hourly · Daily · Weekly · Monthly"]
    end

    subgraph Processing["🧠 Processing Layer"]
        VAL["Validation & Sanitization"]
        FX["FX Resolution<br/>fx_history → fx_rates → 1.0"]
        SNAP["Snapshot Engine<br/>compute_portfolio_snapshot()"]
        AICTX["AI Context Builder"]
    end

    subgraph AI["🤖 AI Providers"]
        COP["GitHub Copilot"]
        OR["OpenRouter (fallback)"]
    end

    subgraph DB["🗄️ Supabase PostgreSQL"]
        ACC[(accounts)]
        TXN[(transactions)]
        PORT[(portfolio)]
        IA[(investment_assets)]
        MD[(market_data)]
        FXT[(fx_rates / fx_history)]
        SNAP2[(portfolio_snapshot)]
        INS[(ai_insights)]
    end

    subgraph Output["📤 Presentation Layer"]
        DASH[Dashboard KPIs]
        WIDGET[AI Insights Widget]
        CHARTS[Recharts Visualizations]
    end

    U1 --> WEB
    U2 --> BOT
    U3 --> CSV

    WEB --> API_TX
    BOT --> API_TX
    CSV --> API_TX

    API_TX --> VAL
    VAL --> FX
    FX --> TXN
    FX --> ACC

    API_DASH --> ACC
    API_DASH --> TXN
    API_DASH --> PORT
    API_DASH --> MD

    CRON --> API_SNAP
    API_SNAP --> SNAP
    SNAP --> SNAP2

    CRON --> API_AI
    API_AI --> AICTX
    AICTX --> COP
    AICTX --> OR
    COP --> INS
    OR --> INS

    PORT --> MD
    IA --> MD

    ACC --> DASH
    TXN --> DASH
    PORT --> CHARTS
    SNAP2 --> CHARTS
    INS --> WIDGET

    DASH --> Output
    WIDGET --> Output
    CHARTS --> Output
```

---

## Data Flow Summary

### Transaction Flow
1. User submits transaction (Discord/web/CSV)
2. Server validates input (`safeTable`, `safeOrder`)
3. FX resolution: `fx_history` (historical) → `fx_rates` (latest) → 1.0
4. Server computes `amount_idr = amount * fx_rate`
5. Insert to `transactions` table

### Portfolio Valuation Flow
1. Cron triggers `/api/cron/snapshot-daily` at 15:00 UTC
2. SQL function `compute_portfolio_snapshot()` aggregates:
   - Investment value from `portfolio` + `market_data`
   - Cash value from `accounts` + `fx_rates`
3. Upsert to `portfolio_snapshot` and `wealth_ledger_daily`

### Market Data Flow
1. OpenClaw agent triggers at 15:30 UTC
2. Query active portfolio assets
3. Route by symbol: TradingView (.JK) or Polygon (US)
4. UPSERT to `market_data` with freshness timestamp

### AI Insight Flow
1. Cron triggers `/api/cron/ai-refresh` at 01:00 UTC
2. `getPortfolioContext()` builds context from DB
3. Provider chain: Copilot → OpenRouter → free models
4. Upsert to `ai_insights` (singleton id=1)

---

## Scalability Considerations

| Concern | Current Approach | Scale-up Path |
|---|---|---|
| **Query speed** | Supabase + limit params | Add Redis caching |
| **AI cost** | Cached insights, provider fallback | Rate-limiting + edge caching |
| **Multi-user** | Single user (personal) | Supabase RLS per user |
| **Data volume** | ~3,000 transactions | Partition by month |
| **Market data** | OpenClaw dual-source | Add more providers |

---

## Security Design

- ✅ No API keys in client-side code
- ✅ Supabase Row Level Security (RLS) enabled
- ✅ Cron endpoints validate `CRON_SECRET` header
- ✅ Privacy Mode — blur all sensitive numbers client-side
- ✅ Service role key used only in server-side API routes
- ✅ Environment variables stored securely on VPS
- ✅ UFW firewall + HTTPS only on OpenClaw VPS
