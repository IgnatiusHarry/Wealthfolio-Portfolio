<div align="center">

# 💰 WealthFolio — Personal Finance Intelligence System

**An end-to-end data platform powering automated financial tracking, AI-driven insights, and real-time analytics for personal wealth management.**

[![Live Demo](https://img.shields.io/badge/🌐_Live_Demo-wealth.ignatiusharry.my.id-6366f1?style=for-the-badge)](https://wealth.ignatiusharry.my.id)
[![Next.js](https://img.shields.io/badge/Next.js-14-black?style=for-the-badge&logo=next.js)](https://nextjs.org)
[![Supabase](https://img.shields.io/badge/Supabase-PostgreSQL-3ECF8E?style=for-the-badge&logo=supabase)](https://supabase.com)
[![Vercel](https://img.shields.io/badge/Deployed-Vercel-black?style=for-the-badge&logo=vercel)](https://vercel.com)
[![AI-Powered](https://img.shields.io/badge/AI-Powered-412991?style=for-the-badge&logo=openai)](https://openai.com)

</div>

---

> ⚠️ **Disclaimer:** This repository is a **portfolio showcase** version. Sensitive data, API keys, and full production code are excluded. It demonstrates technical thinking, system design, and end-to-end data engineering capability.

---

## 🎯 Problem → Solution → Impact

| | |
|---|---|
| **Problem** | Managing finances across multiple currencies (IDR, NTD, USD) with no unified visibility into spending, investments, and savings trends — resulting in poor cashflow control. |
| **Solution** | Built a full-stack personal finance platform with automated data ingestion via Telegram bot, AI-powered categorization, multi-currency normalization, and real-time dashboard analytics. |
| **Impact** | Reduced manual tracking time by **90%**, achieved **cumulative monthly buffer**, and built awareness of spending patterns. |

---

## 🏆 Key Achievements

- ⚡ **Processes 3,000+ transactions** with sub-second query performance via optimized Supabase queries
- 🤖 **AI auto-categorizes** transactions from natural language (Telegram: "makan siang 65 NTD")
- 🌏 **Multi-currency normalization** — IDR, NTD, USD, SGD all converted in real-time via `fx_history` + `fx_rates`
- 📊 **Portfolio tracking** with P&L, unrealized gains, and daily snapshots via `compute_portfolio_snapshot()`
- 🔔 **Automated alerts** when daily spending exceeds budget threshold
- 🧠 **Quantfolio AI** generates fresh contextual insights daily via provider chain (Copilot → OpenRouter → free models)

---

## 🛠️ What Makes This Project Unique

> Most personal finance apps are generic. WealthFolio is engineered specifically for **financial complexity** — multiple currencies, cross-border investments, and the need for AI that understands behavioral patterns, not just numbers.

1. **Custom AI Prompt Engineering** — "Quantfolio AI" analytical lens rotates daily to prevent repetitive insights
2. **Optimistic UI Updates** — Privacy toggle responds instantly, syncs to server in background (no flicker)
3. **Smart Data Normalization** — All monetary values stored in IDR base, converted on-the-fly with live FX rates
4. **Graceful Degradation** — Widget serves cached data instantly while fresh data loads in background

---

## 📐 System Architecture

```mermaid
graph TB
    subgraph Input["📥 Input Channels"]
        TG[Telegram Bot]
        WEB[Web Dashboard]
        CSV[Bank CSV Import]
    end

    subgraph AI["🤖 AI Processing Layer"]
        NLP[NLP Parser<br/>GPT-5-mini]
        CLASS[Auto-Classifier<br/>Food · Transport · Rent]
        PULSE[Financial Pulse<br/>Quantfolio AI]
    end

    subgraph BE["⚙️ Backend API (Next.js)"]
        API[REST API Routes]
        SNAP[Snapshot Engine<br/>compute_portfolio_snapshot()]
        CRON[Cron Scheduler<br/>01:00, 15:00, 15:45 UTC]
    end

    subgraph DB["🗄️ Supabase PostgreSQL"]
        ACC[(accounts)]
        TXN[(transactions)]
        PORT[(portfolio)]
        IA[(investment_assets)]
        MD[(market_data)]
        FX[(fx_rates<br/>fx_history)]
        SNAP2[(portfolio_snapshot<br/>wealth_ledger_daily)]
        AI[(ai_insights)]
    end

    subgraph OUT["📤 Output"]
        DASH[Dashboard KPIs]
        INSIGHT[AI Insights Widget]
        CHARTS[Portfolio Charts]
    end

    TG --> NLP
    WEB --> API
    CSV --> API
    NLP --> CLASS
    CLASS --> API
    API --> ACC
    API --> TXN
    API --> PORT
    API --> IA
    TXN --> FX
    ACC --> FX
    PORT --> MD
    IA --> MD
    TXN --> SNAP
    PORT --> SNAP
    ACC --> SNAP
    SNAP --> SNAP2
    SNAP2 --> DASH
    ACC --> PULSE
    PORT --> PULSE
    TXN --> PULSE
    PULSE --> INSIGHT
    CRON --> SNAP
    CRON --> PULSE
    MD --> CHARTS
    DASH --> OUT
    INSIGHT --> OUT
    CHARTS --> OUT
```

---

## 🗂️ Repository Structure

```
Wealthfolio-Portfolio/
├── README.md                    ← You are here
├── architecture/
│   ├── system-design.md         ← Full system overview
│   ├── openclaw-vps.md          ← AI orchestration & market data
│   └── data-pipeline.md         ← ETL pipeline details
├── data-engineering/
│   ├── schema.md                ← Database schema & ERD
│   └── pipeline.md              ← Data transformation logic
├── analytics/
│   └── dashboard.md             ← KPIs & metrics design
├── ai-pipeline/
│   ├── ai-flow.md               ← AI prompt engineering & provider chain
│   └── automation.md            ← Cron jobs & automation
├── code-structure/
│   ├── backend.md               ← API architecture & patterns
│   └── frontend.md              ← Component structure & design system
├── screenshots/                 ← Visual showcase
└── demo/
    └── index.html               ← Interactive demo
```

---

## 🎲 Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| **Frontend** | Next.js 14, React | Dashboard UI, routing |
| **Styling** | Vanilla CSS, Glassmorphism | Premium dark-mode UI |
| **Backend** | Next.js API Routes | REST endpoints, cron jobs |
| **Database** | Supabase (PostgreSQL) | Data storage, real-time |
| **AI** | GitHub Copilot (GPT-4o), OpenRouter (gemini-2.0-flash) | Insights, classification |
| **Market Data** | TradingView (IDX), Polygon (US) | Price feeds |
| **Bot** | Telegram Bot API | Transaction input |
| **Orchestration** | OpenClaw (VPS) | AI routing, market data jobs |
| **Deploy** | Vercel | CI/CD, cron jobs, edge functions |

---

## 📊 Sample Dashboard Metrics (Dummy Data)

| Metric | Value | Trend |
|---|---|---|
| Net Worth | Rp 1,245,000,000 | ↗️ +8.2% this month |
| Total Cash | Rp 185,000,000 | — |
| Investment Value | Rp 1,060,000,000 | ↗️ +12.4% YTD |
| Unrealized P&L | +Rp 170,000,000 | ✅ +19.1% |
| Savings Rate | **26.7%** | ✅ On track |

---

## 🔗 Quick Links

- 🌐 **Live App**: [wealth.ignatiusharry.my.id](https://wealth.ignatiusharry.my.id)
- 📖 **Data Engineering**: [data-engineering/schema.md](./data-engineering/schema.md)
- 🤖 **AI Pipeline**: [ai-pipeline/ai-flow.md](./ai-pipeline/ai-flow.md)
- 📊 **Analytics**: [analytics/dashboard.md](./analytics/dashboard.md)
- 🏗️ **Architecture**: [architecture/system-design.md](./architecture/system-design.md)

---

## 👨‍💻 About This Project

Built by **Ignatius Harry** — Data Analyst & Data Engineer.

> This repository demonstrates end-to-end data engineering capability: database design, ETL pipelines, AI integration, and real-time analytics.

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0077B5?style=flat&logo=linkedin)](https://linkedin.com/in/ignatiusharry)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-181717?style=flat&logo=github)](https://github.com/IgnatiusHarry)
