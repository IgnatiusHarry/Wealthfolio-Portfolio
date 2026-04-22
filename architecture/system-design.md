# 🏗️ System Architecture — WealthFolio

> ⚠️ **Note:** All data, metrics, and examples in this document use **fictional dummy data** for portfolio demonstration only.

## Overview

WealthFolio is designed as a **three-layer data platform**:

1. **Ingestion Layer** — Multiple input channels (Telegram, web, CSV)
2. **Processing Layer** — AI parsing, FX normalization, daily snapshots
3. **Presentation Layer** — Dashboard, AI insights, alerts

---

## Full System Design

```mermaid
graph TB
    subgraph Users["👤 User Interaction"]
        U1[Web Browser]
        U2[Telegram App]
        U3[CSV Import]
    end

    subgraph Ingestion["📥 Ingestion Layer"]
        WEB[Web Dashboard<br/>Next.js 14]
        BOT[Telegram Bot]
        CSV[CSV Importer]
    end

    subgraph Processing["⚙️ Processing Layer"]
        FX[FX Resolution<br/>resolveFxRate()]
        VAL[Validation<br/>safeTable, safeOrder]
        SNAP[Snapshot Engine<br/>compute_portfolio_snapshot()]
        AI[AI Pipeline<br/>getPortfolioContext()]
    end

    subgraph AILayer["🤖 AI Layer"]
        PARSE[Transaction Parser<br/>GPT-5-mini]
        PULSE[Financial Pulse<br/>Quantfolio AI]
    end

    subgraph Backend["⚙️ Backend — Next.js API Routes"]
        API_TX[/api/dashboard-data]
        API_SNAP[/api/cron/snapshot-daily]
        API_AI[/api/cron/ai-refresh]
        API_MD[/api/cron/market-data]
    end

    subgraph DB["🗄️ Supabase PostgreSQL"]
        T1[(accounts)]
        T2[(transactions)]
        T3[(portfolio)]
        T4[(investment_assets)]
        T5[(market_data)]
        T6[(fx_rates)]
        T7[(fx_history)]
        T8[(portfolio_snapshot)]
        T9[(wealth_ledger_daily)]
        T10[(ai_insights)]
        T11[(goals)]
    end

    subgraph External["🌐 External Services"]
        TV[TradingView<br/>IDX .JK symbols]
        Poly[Polygon<br/>US symbols]
        Copilot[GitHub Copilot<br/>gpt-5-mini]
        OR[OpenRouter<br/>gemini-2.0-flash]
    end

    subgraph Output["📤 Presentation Layer"]
        DASH[Dashboard KPIs]
        WIDGET[AI Insights Widget]
        CHARTS[Recharts Visualizations]
    end

    U1 --> WEB
    U2 --> BOT
    U3 --> CSV

    WEB --> VAL
    BOT --> VAL
    CSV --> VAL
    VAL --> FX
    FX --> API_TX
    
    API_TX --> T1
    API_TX --> T2
    API_TX --> T3
    API_TX --> T4
    T2 --> T7
    T1 --> T6
    
    API_SNAP --> SNAP
    SNAP --> T8
    T8 --> T9
    
    API_AI --> AI
    AI --> PARSE
    PARSE --> PULSE
    PULSE --> T10
    
    T4 --> T5
    TV --> T5
    Poly --> T5
    
    T1 --> AI
    T2 --> AI
    T3 --> AI
    T4 --> AI
    AI --> Copilot
    AI --> OR
    
    T8 --> DASH
    T10 --> WIDGET
    T5 --> CHARTS
    T8 --> CHARTS
    
    DASH --> Output
    WIDGET --> Output
    CHARTS --> Output
```

---

## Data Flow Summary

### Transaction Flow
1. User submits transaction (Telegram/web/CSV)
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
