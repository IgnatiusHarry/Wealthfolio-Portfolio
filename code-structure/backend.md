# ⚙️ Backend Architecture — WealthFolio

> ⚠️ **Code samples are simplified/pseudocode for illustration. No sensitive logic or credentials are included.**

## API Structure (Next.js API Routes)

```
pages/api/
├── dashboard-data.js          ← Universal data endpoint (accounts, portfolio, transactions)
├── settings.js                ← User settings GET/PATCH
├── portfolio-snapshot.js      ← Daily portfolio value calculation (legacy)
├── cron/
│   ├── ai-refresh.js          ← AI insight generation (01:00 UTC)
│   ├── snapshot-daily.js      ← Daily snapshot (15:00 UTC)
│   └── snapshot-reconcile.js  ← Snapshot reconciliation (15:45 UTC)
└── data.js                    ← Legacy data endpoint
```

---

## Key API Pattern: `/api/dashboard-data`

All dashboard data flows through a single parameterized endpoint:

```javascript
// Simplified pattern — no credentials shown
const ALLOWED_TABLES = [
  'accounts', 'portfolio', 'investment_assets', 
  'transactions', 'investment_transactions', 'goals',
  'market_data', 'fx_rates', 'portfolio_snapshot', 'ai_insights'
];

const ALLOWED_COLUMNS = {
  accounts: ['id', 'name', 'type', 'currency', 'balance', 'created_at'],
  portfolio: ['id', 'asset_id', 'quantity', 'avg_price', 'total_invested_idr', 'current_value_idr', 'unrealized_pnl_idr'],
  // ... other tables
};

export default async function handler(req, res) {
  const { table, limit = 100, order = 'created_at', asc = false } = req.query;

  // Allowlist validation to prevent SQL injection
  if (!ALLOWED_TABLES.includes(table)) {
    return res.status(400).json({ error: 'Invalid table' });
  }

  const { data, error } = await supabase
    .from(table)
    .select(ALLOWED_COLUMNS[table]?.join(',') || '*')
    .order(order, { ascending: asc === 'true' })
    .limit(Number(limit));

  if (error) return res.status(500).json({ error: error.message });
  return res.status(200).json({ data });
}
```

---

## FX Resolution: `resolveFxRate()`

Used in transaction processing to normalize all amounts to IDR:

```javascript
// lib/fx.js — Simplified
export async function resolveFxRate(db, date, currency) {
  if (currency === 'IDR') return 1;
  
  // 1. Try historical fx_history for transaction date
  const historical = await db
    .from('fx_history')
    .select('rate')
    .eq('currency', currency)
    .eq('date', date)
    .single();
  
  if (historical.data) return historical.data.rate;
  
  // 2. Fallback to latest fx_rates
  const latest = await db
    .from('fx_rates')
    .select('rate')
    .eq('from_ccy', currency)
    .single();
  
  if (latest.data) return latest.data.rate;
  
  // 3. Fallback to 1.0 with warning
  console.warn(`No FX rate found for ${currency} on ${date}, using 1.0`);
  return 1.0;
}

// Usage in transaction insert
const fxRate = await resolveFxRate(db, transaction.date, transaction.currency);
const amountIdr = transaction.amount * fxRate;
```

---

## Daily Snapshot: `compute_portfolio_snapshot()`

SQL function called by cron job:

```sql
-- SQL function (created by migration)
CREATE OR REPLACE FUNCTION compute_portfolio_snapshot(p_date DATE)
RETURNS TABLE (
  snapshot_date DATE,
  total_value NUMERIC,
  total_invested NUMERIC,
  investment_value NUMERIC,
  cash_value NUMERIC,
  gains NUMERIC,
  losses NUMERIC
) AS $$
BEGIN
  RETURN QUERY
  SELECT 
    p_date AS snapshot_date,
    COALESCE(SUM(p.current_value_idr), 0) + COALESCE(SUM(a.balance * COALESCE(f.rate, 1)), 0) AS total_value,
    COALESCE(SUM(p.total_invested_idr), 0) AS total_invested,
    COALESCE(SUM(p.current_value_idr), 0) AS investment_value,
    COALESCE(SUM(a.balance * COALESCE(f.rate, 1)), 0) AS cash_value,
    COALESCE(SUM(CASE WHEN p.unrealized_pnl_idr > 0 THEN p.unrealized_pnl_idr ELSE 0 END), 0) AS gains,
    COALESCE(SUM(CASE WHEN p.unrealized_pnl_idr < 0 THEN ABS(p.unrealized_pnl_idr) ELSE 0 END), 0) AS losses
  FROM portfolio p
  LEFT JOIN investment_assets ia ON p.asset_id = ia.id
  CROSS JOIN LATERAL (
    SELECT rate FROM fx_rates 
    WHERE from_ccy = a.currency AND to_ccy = 'IDR'
    ORDER BY updated_at DESC LIMIT 1
  ) f
  LEFT JOIN accounts a ON true;
END;
$$ LANGUAGE plpgsql;
```

---

## Settings Sync Pattern (Optimistic Update)

```javascript
// Optimistic update — UI responds immediately, server syncs in background
const patchSetting = async (key, value) => {
  // 1. Immediate local state update (no flicker)
  setSettings(prev => ({ ...prev, [key]: value }));
  localStorage.setItem('settings', JSON.stringify({ ...settings, [key]: value }));

  // 2. Background server sync (non-blocking)
  try {
    await fetch('/api/settings', {
      method: 'PATCH',
      body: JSON.stringify({ [key]: value })
    });
  } catch (e) {
    // Log warning but don't rollback — prevents flicker bug
    console.warn('Settings sync failed, keeping local state:', e.message);
  }
};
```

---

## Cron Job Pattern

```javascript
// pages/api/cron/ai-refresh.js — Simplified
export default async function handler(req, res) {
  // Validate CRON_SECRET
  const cronSecret = process.env.CRON_SECRET || '';
  const authHeader = req.headers['authorization'] || '';
  
  if (authHeader !== `Bearer ${cronSecret}`) {
    return res.status(401).json({ error: 'Unauthorized' });
  }

  try {
    // 1. Get portfolio context
    const context = await getPortfolioContext();
    
    // 2. Call AI with provider chain
    const { content, model } = await callAI(context);
    
    // 3. Upsert to ai_insights (singleton)
    await db.from('ai_insights').upsert({
      id: 1,
      content,
      model,
      generated_at: new Date().toISOString()
    });
    
    return res.json({ success: true, model });
  } catch (err) {
    console.error('AI refresh failed:', err.message);
    return res.status(500).json({ error: err.message });
  }
}
```

---

## Data Fetching Strategy

| Endpoint | Cache | Strategy |
|---|---|---|
| `accounts` | 5 min LocalStorage | Stale-while-revalidate |
| `transactions` | Per session | Fetch on mount + RT subscription |
| `portfolio` | Per session | Fetch on mount |
| `investment_assets` | 24h LocalStorage | Long TTL |
| `market_data` | 5 min | Price changes rarely |
| `portfolio_snapshot` | 1 hour | Daily data, slow change |
| `ai_insights` | 1 hour | Daily refresh |
| `settings` | Immediate + server | Optimistic update |

---

## Error Handling Patterns

| Pattern | Implementation |
|---|---|
| Input validation | `safeTable()`, `safeOrder()` allowlists |
| Null safety | Optional chaining (`?.`) + default values |
| API timeouts | 50s timeout for AI, 10s for DB |
| Rate limiting | Exponential backoff for retries |
| Graceful degradation | Cache fallback, skip failed assets |
