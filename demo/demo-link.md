# 🌐 Live Demo

## WealthFolio — Live Application

> 🔗 **[https://wealth.ignatiusharry.my.id](https://wealth.ignatiusharry.my.id)**

---

## What You Can See

> Updated to match the latest WealthFolio feature set (May 2026 portfolio-docs sync).

The live application showcases the full WealthFolio system. Note that the live app uses the owner's personal data — for a portfolio review, the sections below describe what each page demonstrates.

| Page | What It Demonstrates |
|---|---|
| **Dashboard** | Real-time KPIs, Financial Pulse AI widget, Portfolio overview |
| **Transactions** | Multi-currency transaction log with category filtering |
| **Holdings** | Investment portfolio with P&L, current prices |
| **Insights** | AI-generated financial analysis with Recharts visualizations |
| **Activities** | Transaction timeline with search and filter |

---

## Technology Visible in the Demo

- **Glassmorphism UI** — Premium dark-mode design with blur effects
- **Real-time data** — Supabase real-time subscriptions (toast notification on data change)
- **Financial Pulse Widget** — Live AI insight generation with spinning loader
- **Privacy Mode** — Toggle to blur all sensitive numbers
- **Multi-currency** — Switch between IDR, NTD, USD display

---

## Note on Data Privacy

> The live demo runs on the owner's real Supabase instance. Screenshots in the `/screenshots` folder use dummy data to protect privacy. The live site requires authentication for full access.

---

## Local Setup (For Reviewers)

If you'd like to run WealthFolio locally with dummy data:

```bash
# 1. Clone the main repo (separate from this portfolio repo)
git clone https://github.com/IgnatiusHarry/WealthFolio.git
cd WealthFolio

# 2. Install dependencies
npm install

# 3. Set up .env.local with your own Supabase credentials
cp .env.example .env.local
# Edit NEXT_PUBLIC_SUPABASE_URL and NEXT_PUBLIC_SUPABASE_ANON_KEY

# 4. Run development server
npm run dev
```

> **Supabase Schema:** See [data-engineering/schema.md](../data-engineering/schema.md) for the full database schema to set up your own instance.
