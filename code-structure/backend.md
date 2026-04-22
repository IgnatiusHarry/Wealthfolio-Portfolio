# ⚙️ Backend Architecture — WealthFolio

> ⚠️ **Code samples are simplified/pseudocode for illustration. No sensitive logic or credentials are included.**

## API Structure (Next.js API Routes)

```
pages/api/
├── dashboard-data.js      ← Universal data endpoint (accounts, portfolio, transactions)
├── settings.js            ← User settings GET/PATCH
├── portfolio-snapshot.js  ← Daily portfolio value calculation
├── portfolio-snapshot-backfill.js  ← Historical gap-filling
└── data.js                ← Legacy data endpoint
```

---

## Key API Pattern: `/api/dashboard-data`

All dashboard data flows through a single parameterized endpoint:

```javascript
// Simplified pattern — no credentials shown
export default async function handler(req, res) {
  const { table, limit = 100, order = 'created_at', asc = false } = req.query;

  // Allowlist to prevent SQL injection
  const ALLOWED_TABLES = ['accounts', 'portfolio', 'investment_assets', 'v_transactions_idr'];
  if (!ALLOWED_TABLES.includes(table)) {
    return res.status(400).json({ error: 'Invalid table' });
  }

  const { data, error } = await supabase
    .from(table)
    .select('*')
    .order(order, { ascending: asc === 'true' })
    .limit(Number(limit));

  if (error) return res.status(500).json({ error: error.message });
  return res.status(200).json({ data });
}
```

---

## Portfolio Snapshot Engine

Calculates and stores daily portfolio value:

```javascript
// Pseudocode — simplified
async function calculateSnapshot(date) {
  // 1. Fetch all holdings
  const holdings = await getPortfolio();

  // 2. Get current prices (from investment_assets)
  const prices = await getCurrentPrices();

  // 3. Calculate total value
  const totalValue = holdings.reduce((sum, h) => {
    const price = prices[h.asset_id]?.current_price || h.avg_buy_price;
    return sum + (h.quantity * price * FX_RATE_TO_IDR);
  }, 0);

  // 4. Calculate P&L
  const totalCost = holdings.reduce((sum, h) => sum + (h.quantity * h.avg_buy_price * FX_RATE_TO_IDR), 0);
  const unrealizedPnL = totalValue - totalCost;

  // 5. Store snapshot
  await supabase.from('portfolio_snapshots').upsert({
    snapshot_date: date,
    total_value_idr: totalValue,
    unrealized_pnl: unrealizedPnL,
    holdings_breakdown: JSON.stringify(holdings)
  });
}
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

## Data Fetching Strategy

| Endpoint | Cache | Strategy |
|---|---|---|
| `accounts` | 5 min LocalStorage | Stale-while-revalidate |
| `v_transactions_idr` | Per session | Fetch on mount + RT subscription |
| `portfolio` | Per session | Fetch on mount |
| `investment_assets` | 24h LocalStorage | Long TTL (prices change rarely) |
| `settings` | Immediate + server | Optimistic update |
