# ⚙️ Backend Architecture — WealthFolio

> ⚠️ **Code samples are simplified/pseudocode for illustration. No sensitive logic or credentials are included.**

## API Structure (Next.js API Routes)

```text
pages/api/
├── ai-insights.js                ← Fetch latest AI insight payload
├── ai-proxy.js                   ← Provider proxy / fallback wrapper
├── chat.js                       ← Assistant chat endpoint
├── transactions.js               ← Transaction list/filter endpoint
├── insert-transaction.js         ← Write transaction endpoint
├── goals.js                      ← Goals CRUD endpoint
├── fee-summary.js                ← Fee analytics endpoint
├── trade-matches.js              ← Buy/sell matching engine endpoint
├── sql.js                        ← SQL runner for diagnostics (restricted)
├── prices.js                     ← Market price endpoint
├── stocks.js                     ← Equity metadata endpoint
├── fx-live.js                    ← Live FX endpoint
├── snapshots.js                  ← Snapshot read endpoint
├── portfolio-snapshot.js         ← Daily portfolio value calculation
├── portfolio-snapshot-backfill.js← Backfill historical snapshots
├── trigger-snapshot.js           ← Manual snapshot trigger (protected)
└── [debug/admin endpoints]       ← Internal troubleshooting endpoints
```

---

## Core Backend Flows

- **Transaction Ingestion**
  - Input channels: Discord/web form/import
  - Validation: allowlist + numeric/date checks
  - FX normalization: `fx_history` → `fx_rates` → fallback `1.0`
  - Persist to `transactions` with computed `amount_idr`

- **Portfolio Valuation**
  - Snapshot routes aggregate portfolio + cash + FX into daily totals
  - Uses SQL function pattern `compute_portfolio_snapshot()`
  - Stores results in `portfolio_snapshot` (+ optional mirrored ledger)

- **AI Insights Refresh**
  - Cron-triggered endpoint builds context from portfolio + cashflow + trends
  - Provider chain fallback (Copilot/OpenRouter/free model)
  - Upsert singleton-style insight row in `ai_insights`

---

## Security Patterns

- Allowlist-based query/table guards (`safeTable`, `safeOrder` style)
- `CRON_SECRET` check for scheduled endpoints
- Sensitive keys server-side only (`SUPABASE_SERVICE_ROLE_KEY`, provider keys)
- Restricted SQL endpoint for diagnostics only (admin/internal)

---

## Reliability Patterns

- Graceful degradation for market-data/AI provider failures
- Retry with capped backoff for transient external errors
- Keep-last-good insight/snapshot if refresh fails
- Optional cache-first UI consumption with stale-while-revalidate

---

## Example: Cron Auth Guard (Simplified)

```javascript
export default async function handler(req, res) {
  const expected = process.env.CRON_SECRET || '';
  const auth = req.headers.authorization || '';

  if (auth !== `Bearer ${expected}`) {
    return res.status(401).json({ error: 'Unauthorized' });
  }

  // run job...
  return res.status(200).json({ ok: true });
}
```

---

## Example: FX Normalization (Simplified)

```javascript
export async function resolveFxRate(db, date, ccy) {
  if (ccy === 'IDR') return 1;

  const historical = await db
    .from('fx_history')
    .select('rate')
    .eq('currency', ccy)
    .eq('date', date)
    .maybeSingle();

  if (historical.data?.rate) return historical.data.rate;

  const latest = await db
    .from('fx_rates')
    .select('rate')
    .eq('from_ccy', ccy)
    .maybeSingle();

  return latest.data?.rate ?? 1.0;
}
```

---

## Notes

- This portfolio repo documents architecture and engineering decisions.
- Production secrets, internal infra coordinates, and private business logic are intentionally excluded.
