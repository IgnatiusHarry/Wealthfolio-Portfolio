# 🗄️ Database Schema — WealthFolio

> ⚠️ **Disclaimer:** This document describes the actual production schema. All values shown in examples are for illustration only.

## Entity Relationship Diagram (ERD)

```mermaid
erDiagram
    accounts {
        uuid id PK
        text name
        text type
        text currency
        numeric balance
        timestamp created_at
    }

    investment_assets {
        uuid id PK
        text symbol
        text asset_name
        text asset_type
        text currency
    }

    portfolio {
        uuid id PK
        uuid asset_id FK
        numeric quantity
        numeric avg_price
        numeric total_invested_idr
        numeric current_value_idr
        numeric unrealized_pnl_idr
    }

    transactions {
        uuid id PK
        date date
        text type
        text category
        numeric amount
        text currency
        numeric fx_rate
        numeric amount_idr
        text description
    }

    investment_transactions {
        uuid id PK
        text type
        text symbol
        numeric qty
        numeric price
        numeric amount_idr
        date date
    }

    goals {
        uuid id PK
        text title
        text description
        numeric target_amount
        numeric current_amount
        date target_date
        text category
        text color
        text icon
        text status
        timestamp created_at
        timestamp updated_at
    }

    market_data {
        uuid id PK
        uuid asset_id FK UNIQUE
        numeric price
        numeric price_change
        numeric price_change_pct
        numeric market_cap
        numeric volume
        text currency
        text source
        timestamp last_updated_at
    }

    fx_rates {
        uuid id PK
        text from_ccy
        text to_ccy
        numeric rate
        date rate_date
        timestamp updated_at
    }

    fx_history {
        uuid id PK
        text currency
        numeric rate
        date date
        timestamp created_at
    }

    portfolio_snapshot {
        uuid id PK
        date date UNIQUE
        numeric total_value
        numeric total_invested
        numeric investment_value
        numeric cash_value
        numeric gains
        numeric losses
        timestamp created_at
        timestamp updated_at
    }

    wealth_ledger_daily {
        uuid id PK
        date date UNIQUE
        numeric total_value
        numeric total_invested
        numeric investment_value
        numeric cash_value
        numeric gains
        numeric losses
        timestamp created_at
        timestamp updated_at
    }

    ai_insights {
        uuid id PK
        text content
        text model
        timestamp generated_at
    }

    accounts ||--o{ transactions : "has many"
    investment_assets ||--o{ portfolio : "tracked in"
    portfolio }o--|| market_data : "prices from"
    investment_assets ||--o{ market_data : "has price"
    portfolio }o--|| investment_assets : "references"
    transactions --> fx_history : "fx_rate resolved from"
    accounts --> fx_rates : "current valuation via"
    portfolio_snapshot --> wealth_ledger_daily : "mirrored to"
```

---

## Table Descriptions

### `accounts`
Stores cash accounts and balances by currency.

| Column | Type | Description |
|---|---|---|
| `id` | UUID | Primary key |
| `name` | TEXT | Account label (e.g., "BCA Savings", "Taiwan Bank") |
| `type` | TEXT | `bank`, `ewallet`, `investment`, `cash` |
| `currency` | TEXT | Primary currency (IDR, NTD, USD, SGD) |
| `balance` | NUMERIC | Current balance in account currency |
| `created_at` | TIMESTAMP | Record creation timestamp |

---

### `investment_assets`
Instrument master table for all investment holdings.

| Column | Type | Description |
|---|---|---|
| `id` | UUID | Primary key |
| `symbol` | TEXT | Ticker symbol (e.g., `BBCA.JK`, `AAPL`, `0050.TW`) |
| `asset_name` | TEXT | Full asset name |
| `asset_type` | TEXT | `stock`, `etf`, `us_stock`, `crypto` |
| `currency` | TEXT | Base currency (IDR, USD, SGD, NTD) |

---

### `portfolio`
Position-level holdings with valuation state.

| Column | Type | Description |
|---|---|---|
| `id` | UUID | Primary key |
| `asset_id` | UUID | FK to `investment_assets.id` |
| `quantity` | NUMERIC | Number of units held |
| `avg_price` | NUMERIC | Average purchase price |
| `total_invested_idr` | NUMERIC | Total cost basis in IDR |
| `current_value_idr` | NUMERIC | Current market value in IDR |
| `unrealized_pnl_idr` | NUMERIC | Unrealized P&L in IDR |

---

### `transactions`
Cashflow ledger with server-side FX normalization.

| Column | Type | Description |
|---|---|---|
| `id` | UUID | Primary key |
| `date` | DATE | Transaction date |
| `type` | TEXT | `inflow` or `outflow` |
| `category` | TEXT | Auto-classified category |
| `amount` | NUMERIC | Amount in original currency |
| `currency` | TEXT | Original currency code |
| `fx_rate` | NUMERIC | Resolved historical rate from `fx_history` |
| `amount_idr` | NUMERIC | Server-computed IDR value |
| `description` | TEXT | Transaction description |

---

### `investment_transactions`
Investment buy/sell transaction log.

| Column | Type | Description |
|---|---|---|
| `id` | UUID | Primary key |
| `type` | TEXT | `buy` or `sell` |
| `symbol` | TEXT | Asset symbol |
| `qty` | NUMERIC | Quantity traded |
| `price` | NUMERIC | Price per unit |
| `amount_idr` | NUMERIC | Total value in IDR |
| `date` | DATE | Transaction date |

---

### `goals`
Financial goals tracking with target/progress monitoring.

| Column | Type | Description |
|---|---|---|
| `id` | UUID | Primary key |
| `title` | TEXT | Goal title |
| `description` | TEXT | Goal description |
| `target_amount` | NUMERIC | Target value |
| `current_amount` | NUMERIC | Current progress |
| `target_date` | DATE | Target completion date |
| `category` | TEXT | Goal category |
| `color` | TEXT | Hex color code |
| `icon` | TEXT | Emoji icon |
| `status` | TEXT | `active`, `completed`, `paused`, `cancelled` |
| `created_at` | TIMESTAMP | Creation timestamp |
| `updated_at` | TIMESTAMP | Last update timestamp |

---

### `market_data`
Latest current prices for active portfolio assets. Populated by OpenClaw agent.

| Column | Type | Description |
|---|---|---|
| `id` | UUID | Primary key |
| `asset_id` | UUID | FK to `investment_assets.id` (UNIQUE) |
| `price` | NUMERIC | Latest price in asset currency |
| `price_change` | NUMERIC | Absolute change from previous close |
| `price_change_pct` | NUMERIC | Percent change |
| `market_cap` | NUMERIC | Market capitalization (optional) |
| `volume` | NUMERIC | Trading volume (optional) |
| `currency` | TEXT | Asset base currency |
| `source` | TEXT | `TradingView`, `Polygon`, `CoinGecko` |
| `last_updated_at` | TIMESTAMP | Freshness tracking |

---

### `fx_rates`
Latest FX rates for current valuation (accounts, portfolio live value).

| Column | Type | Description |
|---|---|---|
| `id` | UUID | Primary key |
| `from_ccy` | TEXT | Source currency |
| `to_ccy` | TEXT | Target currency (IDR) |
| `rate` | NUMERIC | Exchange rate |
| `rate_date` | DATE | Rate date |
| `updated_at` | TIMESTAMP | Last update |

---

### `fx_history`
Historical FX records for accurate cost-basis conversion.

| Column | Type | Description |
|---|---|---|
| `id` | UUID | Primary key |
| `currency` | TEXT | Source currency |
| `rate` | NUMERIC | Rate to IDR |
| `date` | DATE | Rate effective date |
| `created_at` | TIMESTAMP | Record creation |

---

### `portfolio_snapshot`
Daily time-series of total wealth and decomposition.

| Column | Type | Description |
|---|---|---|
| `id` | UUID | Primary key |
| `date` | DATE | Snapshot date (UNIQUE) |
| `total_value` | NUMERIC | Investment + cash value |
| `total_invested` | NUMERIC | Sum of cost basis |
| `investment_value` | NUMERIC | Sum of current position values |
| `cash_value` | NUMERIC | Sum of account balances |
| `gains` | NUMERIC | Sum of positive P&L |
| `losses` | NUMERIC | Sum of negative P&L |
| `created_at` | TIMESTAMP | Creation timestamp |
| `updated_at` | TIMESTAMP | Last update |

---

### `wealth_ledger_daily`
Mirror table for `portfolio_snapshot` (audit trail).

| Column | Type | Description |
|---|---|---|
| `id` | UUID | Primary key |
| `date` | DATE | Snapshot date (UNIQUE) |
| `total_value` | NUMERIC | Total portfolio value |
| `total_invested` | NUMERIC | Total cost basis |
| `investment_value` | NUMERIC | Investment component |
| `cash_value` | NUMERIC | Cash component |
| `gains` | NUMERIC | Total gains |
| `losses` | NUMERIC | Total losses |
| `created_at` | TIMESTAMP | Creation timestamp |
| `updated_at` | TIMESTAMP | Last update |

---

### `ai_insights`
Materialized AI portfolio report (singleton pattern).

| Column | Type | Description |
|---|---|---|
| `id` | UUID | Primary key (default 1) |
| `content` | TEXT | Full AI analysis |
| `model` | TEXT | LLM model used |
| `generated_at` | TIMESTAMP | Generation timestamp |

---

## Indexes

| Table | Index Name | Columns | Purpose |
|---|---|---|---|
| `transactions` | `idx_transactions_date` | `date` | Date range queries |
| `transactions` | `idx_transactions_category` | `category` | Category filtering |
| `portfolio_snapshot` | `idx_snapshot_date` | `date` | Daily snapshot queries |
| `market_data` | `idx_market_data_asset` | `asset_id` | Price lookups |
| `fx_history` | `idx_fx_history_date_currency` | `date, currency` | Historical rate lookup |
| `fx_rates` | `idx_fx_rates_pair` | `from_ccy, to_ccy` | Current rate lookup |
| `investment_assets` | `idx_assets_symbol` | `symbol` | Symbol search |

---

## Data Quality Constraints

| Constraint | Table | Rule |
|---|---|---|
| UNIQUE | `market_data.asset_id` | One price row per asset |
| UNIQUE | `portfolio_snapshot.date` | One snapshot per day |
| UNIQUE | `wealth_ledger_daily.date` | One ledger entry per day |
| UNIQUE | `fx_rates.from_ccy, fx_rates.to_ccy` | One rate per currency pair |
| FK | `portfolio.asset_id` → `investment_assets.id` | Position must reference valid asset |
| FK | `market_data.asset_id` → `investment_assets.id` | Price must reference valid asset |
| CHECK | `transactions.amount > 0` | Positive transaction amounts |
| CHECK | `portfolio.quantity > 0` | Positive holdings |
| NOT NULL | `transactions.date`, `transactions.type`, `transactions.amount` | Required fields |

---

## FX Conversion Flow

### Transaction Insert Path

1. User submits transaction with `amount`, `currency`, `date`
2. Server calls `resolveFxRate(db, date, currency)`:
   - If `currency = 'IDR'` → return `1`
   - Else → Query `fx_history` for rate on transaction date + currency pair
   - Fallback → Use latest `fx_rates` if historical missing
   - Fallback → Use `1.0` and log warning if no rate found
3. Compute `amount_idr = amount * resolved_fx_rate`
4. Validate client-supplied `amount_idr` against computed value
5. Insert row into `transactions` with resolved `fx_rate` + computed `amount_idr`

### Account Valuation Path (Current Prices)

1. Query accounts with current balances by currency
2. For each non-IDR account: lookup latest rate in `fx_rates`
3. Convert `balance * latest_rate` to IDR
4. Sum all account values in IDR for `cash_value`

### Cost-Basis Preservation

- `transactions.fx_rate` = historical rate at transaction date
- `fx_rates` table = current rates for live valuation only
- This separation preserves historical P&L accuracy while allowing live revaluation

---

## SQL Functions

### `compute_portfolio_snapshot(p_date DATE)`

Computes daily wealth snapshot from portfolio + accounts.

```sql
SELECT * FROM compute_portfolio_snapshot('2026-04-12');
```

Returns: `snapshot_date, total_value, total_invested, investment_value, cash_value, gains, losses`
