# 🎨 Frontend Architecture — WealthFolio

> ⚠️ **Code samples are simplified for illustration. No sensitive logic is included.**

## Component Hierarchy

```
pages/
└── index.js                    ← Root layout, routing, settings context

components/
├── AINotificationWidget.js     ← AI Insights widget (star component)
├── InsightsPage.js             ← Full AI analysis page with charts
├── HoldingsPage.js             ← Investment portfolio tracking
├── ActivitiesPage.js           ← Transaction timeline & search
├── CashByAccount.js            ← Account balance breakdown
├── DashboardPage.js            ← Main KPI overview
└── [various chart components]  ← Recharts-based visualizations

lib/
├── settings.js                 ← Global settings context (privacy, currency, AI config)
├── fx.js                       ← FX resolution utilities
├── supabase.js                 ← Supabase client initialization
└── authMiddleware.js           ← API allowlist validation
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
<span>{mask('Rp 85,000,000')}</span>
// Privacy OFF: Rp 85,000,000
// Privacy ON:  Rp ****,*****,***
```

### 3. Data Fetching with Cache

```javascript
// Pattern: Serve cache immediately, refresh in background
const [data, setData] = useState(() => readLocalCache() || { 
  accounts: [], 
  transactions: [], 
  portfolio: [] 
});

useEffect(() => {
  // Fresh data loads without blocking UI
  Promise.all([
    fetch('/api/dashboard-data?table=accounts&limit=100'),
    fetch('/api/dashboard-data?table=transactions&limit=3000'),
    fetch('/api/dashboard-data?table=portfolio&limit=200'),
  ]).then(async ([accs, txs, port]) => {
    const fresh = { 
      accounts: await accs.json().data, 
      transactions: await txs.json().data,
      portfolio: await port.json().data 
    };
    setData(fresh);
    writeLocalCache(fresh);
  });
}, []);
```

### 4. Error Boundaries

```javascript
// Error boundary component
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return <div className="error-fallback">Something went wrong. Please refresh.</div>;
    }
    return this.props.children;
  }
}
```

### 5. Loading Skeletons

```javascript
// Skeleton component for loading states
function DashboardSkeleton() {
  return (
    <div className="skeleton-grid">
      <div className="skeleton-card skeleton-shimmer" style={{ height: '120px' }} />
      <div className="skeleton-card skeleton-shimmer" style={{ height: '120px' }} />
      <div className="skeleton-card skeleton-shimmer" style={{ height: '200px' }} />
    </div>
  );
}
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
| **Lazy data loading** | Heavy data (3,000+ txns) loads after initial paint |
| **Memoized calculations** | `useMemo` for financial aggregations |
| **Real-time subscriptions** | Supabase channel updates only trigger relevant components |
| **Code splitting** | Dynamic imports for chart components |
| **Image optimization** | Next.js Image component with lazy loading |

---

## Responsive Breakpoints

```css
/* Mobile-first approach */
.container {
  width: 100%;
  padding: 16px;
}

@media (min-width: 768px) {
  .container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 24px;
  }
  
  .grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (min-width: 1024px) {
  .grid {
    grid-template-columns: repeat(3, 1fr);
  }
}
```
