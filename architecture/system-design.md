# 🏗️ System Architecture — WealthFolio

> ⚠️ **Note:** All data, metrics, and examples in this document use **fictional dummy data** for portfolio demonstration only.

## Overview

WealthFolio is designed as a **three-layer data platform**:

1. **Ingestion Layer** — Multiple input channels (bot, web, CSV)
2. **Processing Layer** — AI parsing, transformation, normalization
3. **Presentation Layer** — Dashboard, alerts, AI insights

---

## Full System Design

```mermaid
graph TB
    subgraph Users["👤 User Interaction"]
        U1[Web Browser]
        U2[Telegram App]
        U3[Discord Server]
    end

    subgraph Ingestion["📥 Ingestion Layer"]
        BOT[Telegram / Discord Bot<br/>Node.js]
        WEB[Web Dashboard<br/>Next.js 14]
        CSV[CSV Importer]
    end

    subgraph AILayer["🤖 AI Layer (GitHub Copilot / GPT-5)"]
        PARSE[Transaction Parser<br/>Extract: amount, category, currency, date]
        CLASSIFY[Auto-Classifier<br/>Food · Transport · Rent · Entertainment · Investment]
        PULSE[Financial Pulse Engine<br/>Quantfolio Intelligence — Daily Rotation]
        PROMPT[Prompt Engineering<br/>Temperature: 0.75 · Context Window: 3,000 txns]
    end

    subgraph Backend["⚙️ Backend — Next.js API Routes"]
        API_TX[/api/dashboard-data]
        API_SNAP[/api/portfolio-snapshot]
        API_BACK[/api/portfolio-snapshot-backfill]
        API_SET[/api/settings]
        CRON[Cron Scheduler<br/>Hourly · Daily · Weekly · Monthly]
    end

    subgraph DB["🗄️ Supabase PostgreSQL"]
        T1[(accounts)]
        T2[(transactions)]
        T3[(portfolio)]
        T4[(investment_assets)]
        T5[(portfolio_snapshots)]
        V1[(VIEW: v_transactions_idr<br/>Normalized to IDR base)]
    end

    subgraph Output["📤 Presentation Layer"]
        DASH[Dashboard KPIs]
        WIDGET[Financial Pulse Widget]
        ALERTS[Telegram Alerts]
        CHARTS[Recharts Visualizations]
    end

    U1 --> WEB
    U2 --> BOT
    U3 --> BOT

    BOT --> PARSE
    WEB --> API_TX
    CSV --> API_TX

    PARSE --> CLASSIFY
    CLASSIFY --> API_TX

    API_TX --> T1 & T2 & T3 & T4
    T2 --> V1
    V1 --> API_SNAP
    API_SNAP --> T5
    API_BACK --> T5

    T5 --> DASH
    V1 --> PULSE
    PULSE --> PROMPT
    PROMPT --> WIDGET

    CRON --> API_SNAP & ALERTS

    DASH --> CHARTS
    WIDGET --> DASH
    CHARTS --> Output
```

---

## Scalability Considerations

| Concern | Current Approach | Scale-up Path |
|---|---|---|
| **Query speed** | Supabase views + limit params | Add Redis caching layer |
| **AI cost** | Cached insights, generate on mount | Rate-limiting + edge caching |
| **Multi-user** | Single user (personal) | Row-level security (Supabase RLS) per user |
| **Data volume** | 3,000 transactions | Partition by month, archive old data |
| **Bot concurrency** | Single instance | Message queue (BullMQ / Redis) |

---

## Security Design

- ✅ No API keys in client-side code
- ✅ Supabase Row Level Security (RLS) enabled
- ✅ Settings synced via server-side PATCH (credentials: include)
- ✅ Privacy Mode — blur all sensitive numbers client-side
- ✅ Bot commands require pre-authorized user ID whitelist
