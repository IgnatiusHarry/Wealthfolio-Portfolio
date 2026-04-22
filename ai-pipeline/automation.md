# 🤖 Automation & Cron Jobs — WealthFolio

> ⚠️ **Disclaimer:** All sample outputs and data shown are fictional dummy data for portfolio demonstration only.

## Automation Philosophy

WealthFolio operates on a **"set and forget"** model — once configured, all tracking, summarization, and alerts run automatically without user intervention.

---

## Cron Job Registry

All cron jobs are configured in `vercel.json` and run on Vercel's serverless platform.

```javascript
// Vercel cron configuration
{
  "crons": [
    {
      "path": "/api/cron/ai-refresh",
      "schedule": "0 1 * * *"  // Daily at 01:00 UTC
    },
    {
      "path": "/api/cron/snapshot-daily",
      "schedule": "0 15 * * *"  // Daily at 15:00 UTC
    },
    {
      "path": "/api/cron/snapshot-reconcile",
      "schedule": "45 15 * * *"  // Daily at 15:45 UTC
    }
  ]
}
```

### Job Details

| Job | Schedule (UTC) | Schedule (WIB) | Action |
|---|---|---|---|
| `ai-refresh` | 01:00 UTC | 08:00 WIB | Generate AI portfolio insights |
| `snapshot-daily` | 15:00 UTC | 22:00 WIB | Compute and store daily portfolio snapshot |
| `snapshot-reconcile` | 15:45 UTC | 22:45 WIB | Reconcile snapshot to audit ledger |

---

## External Automation (OpenClaw)

### Market Data Ingestion

**Job:** `market_data_daily_refresh`
**Schedule:** 15:30 UTC (22:30 WIB)
**Platform:** OpenClaw agent on VPS

**Responsibilities:**
- Fetch current prices for all portfolio assets
- Dual-source routing: TradingView (IDX .JK symbols), Polygon (US symbols)
- UPSERT to `market_data` table
- Graceful degradation on API failures

### FX Rate Refresh

**Schedule:** Daily after US market close
**Platform:** OpenClaw agent

**Responsibilities:**
- Fetch latest FX rates for IDR pairs
- Upsert to `fx_rates` table
- Maintain rate freshness for live valuation

---

## Cron Authentication

All cron endpoints validate the `CRON_SECRET` header:

```javascript
// Cron endpoint pattern
export default async function handler(req, res) {
  const cronSecret = process.env.CRON_SECRET || '';
  const authHeader = req.headers['authorization'] || '';
  
  const isAuthorized = authHeader === `Bearer ${cronSecret}`;
  
  if (!isAuthorized) {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  
  // Process cron job...
}
```

---

## Health Monitoring

Each cron job includes health check patterns:

```javascript
// Logging pattern
console.log('🤖 Starting daily AI insight generation...');
try {
  // Job logic
  console.log(`✅ Daily AI insight updated (${provider} / ${model})`);
} catch (err) {
  console.error('❌ Daily AI generation failed:', err.message);
}
```

---

## Sample Automated Outputs (Dummy Data)

### Daily Portfolio Snapshot (Computed)

```
📊 WealthFolio Snapshot — April 22, 2024

💰 Total Net Worth: Rp 1,245,000,000
├── Cash: Rp 185,000,000 (14.9%)
└── Investments: Rp 1,060,000,000 (85.1%)

📈 Portfolio Performance:
├── Total Invested: Rp 890,000,000
├── Current Value: Rp 1,060,000,000
├── Unrealized P&L: Rp +170,000,000 (+19.1%)
├── Gains: Rp 195,000,000
└── Losses: Rp 25,000,000

🏦 Accounts:
├── BCA Indonesia: Rp 125,000,000 (IDR)
├── Taiwan Bank: Rp 42,500,000 (NTD)
└── IBKR Investment: Rp 17,500,000 (USD)

Generated: 2024-04-22 15:00 UTC
```

### AI Insight Report (Generated at 01:00 UTC)

```
## 📊 Portfolio Health Score: 7.5/10
*Strong performance with moderate risk in tech allocation*

## 🎯 Top 3 Risks
1. **Concentration Risk** — NVDA represents 35% of portfolio
2. **Currency Exposure** — 40% in USD assets, IDR weakness risk
3. **Liquidity Gap** — Only 15% in cash, below 6-month buffer

## 💡 Top 3 Opportunities
1. **Rebalance to ETFs** — Shift 10% from NVDA to IHSG ETF
2. **DCA US Index** — Add monthly Rp 5M to US index fund
3. **Emergency Fund** — Build 3-month expense buffer

## ⚖️ Rebalancing Targets
- Reduce NVDA: 35% → 25%
- Increase IHSG ETF: 10% → 20%
- Cash buffer: 15% → 20%

## 💰 Cashflow Tips
1. Automate savings transfer on salary day
2. Food variance +12% this month — review spending
3. 3 unused subscriptions detected

**🎯 Priority Action This Week:** 
Reduce NVDA by 10% and reallocate to IHSG ETF.
```

---

## Monitoring Dashboard

Query for cron job health:

```sql
-- Check latest snapshot
SELECT * FROM portfolio_snapshot ORDER BY date DESC LIMIT 1;

-- Check market data freshness
SELECT 
  source, 
  COUNT(*) AS count,
  MAX(last_updated_at) AS freshest,
  MIN(last_updated_at) AS stalest
FROM market_data 
GROUP BY source;

-- Check AI insights freshness
SELECT * FROM ai_insights WHERE id = 1;
```

---

## Alert Conditions

| Condition | Action |
|---|---|
| `portfolio_snapshot.date` not today/yesterday | Alert ops team |
| `market_data.last_updated_at` > 24h old | Check OpenClaw status |
| AI refresh fails | Log error, retain previous |
| Snapshot reconcile mismatch | Flag for investigation |
