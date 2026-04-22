# 🎨 Frontend Architecture — WealthFolio

> ⚠️ **Code samples are simplified for illustration. No sensitive logic is included.**

## Component Hierarchy

```
pages/
└── index.js                    ← Root layout, routing, settings context

components/
├── AINotificationWidget.js     ← Financial Pulse AI widget (star component)
├── InsightsPage.js             ← Full AI analysis page with charts
├── HoldingsPage.js             ← Investment portfolio tracking
├── ActivitiesPage.js           ← Transaction timeline & search
├── CashByAccount.js            ← Account balance breakdown
├── DashboardPage.js            ← Main KPI overview
└── [various chart components]  ← Recharts-based visualizations

lib/
├── settings.js                 ← Global settings context (privacy, currency, AI config)
├── aiEngineConfig.js           ← AI provider configuration & client factory
└── supabase.js                 ← Supabase client initialization
```

---

## Key Design Patterns

### 1. Settings Context (Global State)
```javascript
// lib/settings.js — Simplified
export function SettingsProvider({ children }) {
  const [settings, setSettings] = useState(DEFAULT_SETTINGS);

  // Load in priority order: Server → LocalStorage → Default
  useEffect(() => {
    const local = JSON.parse(localStorage.getItem('hwt_settings') || '{}');
    setSettings(prev => ({ ...prev, ...local }));

    fetch('/api/settings').then(r => r.json()).then(({ settings: server }) => {
      setSettings(prev => ({ ...prev, ...server }));
    });
  }, []);

  // Optimistic update: UI first, server second
  const patchSetting = async (key, value) => {
    setSettings(prev => ({ ...prev, [key]: value }));      // Immediate
    localStorage.setItem('hwt_settings', JSON.stringify({ ...settings, [key]: value }));
    await fetch('/api/settings', { method: 'PATCH', body: JSON.stringify({ [key]: value }) });
  };

  return <SettingsContext.Provider value={{ settings, patchSetting }}>{children}</SettingsContext.Provider>;
}
```

### 2. Privacy Mode (Global Number Masking)
```javascript
// Pattern used across all components
const mask = (value) => {
  if (!isPrivacyMode) return value;
  return String(value).replace(/\d[\d,.]*/g, '****');
};

// Usage in JSX
<span>{mask('NT$85,000')}</span>
// Privacy OFF: NT$85,000
// Privacy ON:  NT$****
```

### 3. Data Fetching with Cache
```javascript
// Pattern: Serve cache immediately, refresh in background
const [data, setData] = useState(() => readLocalCache() || { accounts: [], transactions: [] });

useEffect(() => {
  // Fresh data loads without blocking UI
  Promise.all([
    fetch('/api/dashboard-data?table=accounts&limit=100'),
    fetch('/api/dashboard-data?table=v_transactions_idr&limit=3000'),
  ]).then(async ([accs, txs]) => {
    const fresh = { accounts: await accs.json().data, transactions: await txs.json().data };
    setData(fresh);
    writeLocalCache(fresh);
  });
}, []);
```

---

## UI/UX Design System

| Token | Value | Usage |
|---|---|---|
| `--accent` | `#6366f1` (Indigo) | Primary interactive elements |
| `--bg-card` | `rgba(255,255,255,0.03)` | Glassmorphism card background |
| `--text-primary` | `#f1f5f9` | Main text |
| `--text-muted` | `rgba(255,255,255,0.4)` | Secondary labels |
| Border radius | `14px–20px` | All card components |
| Font | System default + monospace for numbers | Clean, fast rendering |

### Glassmorphism Card Pattern
```css
.card {
  background: rgba(255, 255, 255, 0.03);
  border: 1px solid rgba(255, 255, 255, 0.07);
  border-radius: 16px;
  backdrop-filter: blur(10px);
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
}
```

---

## Performance Considerations

| Strategy | Implementation |
|---|---|
| **LocalStorage caching** | All API responses cached, served instantly on next visit |
| **Optimistic updates** | Settings changes reflect immediately, no loading state |
| **Lazy data loading** | Heavy data (3,000 txns) loads after initial paint |
| **Memoized calculations** | `useMemo` for financial aggregations (only recalc on data change) |
| **Real-time subscriptions** | Supabase channel updates only trigger relevant components |
