<div align="center">

# 💰 WealthFolio — Personal Finance Intelligence System

**An end-to-end data platform powering automated financial tracking, AI-driven insights, and real-time analytics for an expat living in Taiwan.**

[![Live Demo](https://img.shields.io/badge/🌐_Live_Demo-wealth.ignatiusharry.my.id-6366f1?style=for-the-badge)](https://wealth.ignatiusharry.my.id)
[![Next.js](https://img.shields.io/badge/Next.js-14-black?style=for-the-badge&logo=next.js)](https://nextjs.org)
[![Supabase](https://img.shields.io/badge/Supabase-PostgreSQL-3ECF8E?style=for-the-badge&logo=supabase)](https://supabase.com)
[![Vercel](https://img.shields.io/badge/Deployed-Vercel-black?style=for-the-badge&logo=vercel)](https://vercel.com)
[![OpenAI](https://img.shields.io/badge/AI-GPT_Powered-412991?style=for-the-badge&logo=openai)](https://openai.com)

</div>

---

## 🎯 Problem → Solution → Impact

| | |
|---|---|
| **Problem** | Managing finances across multiple currencies (IDR, NTD, USD) with no unified visibility into spending, investments, and savings trends — resulting in poor cashflow control. |
| **Solution** | Built a full-stack personal finance platform with automated data ingestion via Telegram bot, AI-powered categorization, multi-currency normalization, and real-time dashboard analytics. |
| **Impact** | Reduced manual tracking time by **90%**, achieved **NT$1,200+ cumulative monthly buffer**, and built awareness of food spending trends (avg NT$245/day vs NT$300 limit). |

---

## 🏆 Key Achievements

- ⚡ **Processes 3,000+ transactions** with sub-second query performance via optimized Supabase views
- 🤖 **AI auto-categorizes** transactions from natural language (Telegram: "makan siang 65 NTD")
- 🌏 **Multi-currency normalization** — IDR, NTD, USD, SGD all converted in real-time
- 📊 **Portfolio tracking** with P&L, unrealized gains, and daily snapshots
- 🔔 **Automated alerts** when daily food budget exceeds NT$300 threshold
- 🧠 **Financial Pulse AI** generates fresh contextual insights every session

---

## 🛠️ What Makes This Project Unique

> Most personal finance apps are generic. WealthFolio is engineered specifically for an **expat's financial complexity** — multiple currencies, cross-border investments, and the need for AI that understands behavioral patterns, not just numbers.

1. **Custom AI Prompt Engineering** — "Quantfolio Intelligence" analytical lens rotates daily to prevent repetitive insights
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
        NLP[NLP Parser<br/>GPT-5 / Copilot]
        CLASS[Auto-Classifier<br/>Food · Transport · Rent]
        PULSE[Financial Pulse<br/>Quantfolio Intelligence]
    end

    subgraph BE["⚙️ Backend API (Next.js)"]
        API[REST API Routes]
        SNAP[Snapshot Engine]
        CRON[Cron Scheduler]
    end

    subgraph DB["🗄️ Supabase PostgreSQL"]
        ACC[(accounts)]
        TXN[(transactions)]
        PORT[(portfolio)]
        SNAP2[(snapshots)]
        VIEW[(v_transactions_idr)]
    end

    subgraph OUT["📤 Output"]
        DASH[Dashboard KPIs]
        INSIGHT[AI Insights Widget]
        NOTIF[Telegram Alerts]
    end

    TG --> NLP
    WEB --> API
    CSV --> API
    NLP --> CLASS
    CLASS --> API
    API --> ACC & TXN & PORT
    TXN --> VIEW
    VIEW --> SNAP
    SNAP --> SNAP2
    SNAP2 --> DASH
    VIEW --> PULSE
    PULSE --> INSIGHT
    CRON --> SNAP & NOTIF
    DASH --> WEB
```

---

## 🗂️ Repository Structure

```
Wealthfolio-Portfolio/
├── README.md                    ← You are here
├── architecture/
│   ├── system-design.md         ← Full system overview
│   ├── data-pipeline.md         ← ETL pipeline details
│   └── ai-workflow.md           ← AI integration flow
├── data-engineering/
│   ├── schema.md                ← Database schema & ERD
│   └── pipeline.md              ← Data transformation logic
├── analytics/
│   ├── dashboard.md             ← KPIs & metrics design
│   └── insights.md              ← Sample AI insights
├── ai-pipeline/
│   ├── ai-flow.md               ← AI prompt engineering
│   └── automation.md            ← Cron jobs & automation
├── code-structure/
│   ├── backend.md               ← API architecture
│   └── frontend.md              ← Component structure
├── screenshots/                 ← Visual showcase
└── demo/
    └── demo-link.md             ← Live demo information
```

---

## 🎲 Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| **Frontend** | Next.js 14, React | Dashboard UI, routing |
| **Styling** | Vanilla CSS, Glassmorphism | Premium dark-mode UI |
| **Backend** | Next.js API Routes | REST endpoints, auth |
| **Database** | Supabase (PostgreSQL) | Data storage, real-time |
| **AI** | GitHub Copilot API / GPT-5 | Insights, classification |
| **Bot** | Telegram Bot API | Transaction input |
| **Auth** | Supabase Auth | User management |
| **Deploy** | Vercel | CI/CD, edge functions |
| **Monitoring** | Vercel Analytics | Performance tracking |

---

## 📊 Sample Dashboard Metrics (Dummy Data)

| Metric | Value | Trend |
|---|---|---|
| Monthly Income | NT$ 85,000 | — |
| Total Expenses | NT$ 62,300 | ↗️ +4.2% |
| Savings Rate | **26.7%** | ✅ On track |
| Food Budget | NT$ 245/day avg | ✅ Under NT$300 limit |
| Investment Portfolio | NT$ 340,000 | ↗️ +12.4% YTD |
| Cumulative Buffer | NT$ +1,840 | ✅ Healthy |

---

## 🔗 Quick Links

- 🌐 **Live App**: [wealth.ignatiusharry.my.id](https://wealth.ignatiusharry.my.id)
- 📖 **Data Engineering**: [data-engineering/schema.md](./data-engineering/schema.md)
- 🤖 **AI Pipeline**: [ai-pipeline/ai-flow.md](./ai-pipeline/ai-flow.md)
- 📊 **Analytics**: [analytics/dashboard.md](./analytics/dashboard.md)
- 🏗️ **Architecture**: [architecture/system-design.md](./architecture/system-design.md)

---

## 👨‍💻 About This Project

Built by **Ignatius Harry** — Data Analyst & Data Engineer based in Taiwan.

> This repository is a **portfolio showcase** version. Sensitive data, API keys, and full production code are excluded. It demonstrates technical thinking, system design, and end-to-end data engineering capability.

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0077B5?style=flat&logo=linkedin)](https://linkedin.com/in/ignatiusharry)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-181717?style=flat&logo=github)](https://github.com/IgnatiusHarry)
