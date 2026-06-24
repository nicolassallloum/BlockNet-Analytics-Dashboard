# BlockNet Analytics Dashboard

**Project:** Blockchain Networks Analytics Dashboard  
**Database:** `blocknet_dashboard`  
**Schema:** `blockchain_dashboard`  
**Stack:** PostgreSQL, Laravel, Vue, Docker, Nginx, Ubuntu VPS  
**Initial Networks:** Bitcoin, Ethereum, BNB Chain, Solana, TRON, Polygon, Base, Arbitrum

---

# Phase 11 — Vue Dashboard Pages, Charts, and Alerts UI

## 1. Phase Objective

Build the visible dashboard experience in Vue.

This phase creates:

- Dashboard overview page
- KPI cards
- Blockchain network grid
- Network detail page
- Historical charts
- Alerts page
- Reports page UI
- Admin monitoring page
- Loading and error states

---

## 2. Dashboard Page Sections

`DashboardPage.vue` should include:

1. Header summary
2. KPI cards
3. Market cap ranking
4. TVL ranking
5. Stablecoin supply overview
6. Latest alerts
7. Last sync status

---

## 3. KPI Cards

Required KPI cards:

| Card | Source |
|------|--------|
| Active Networks | `blockchain_networks` |
| Total Market Cap | `market_metrics_latest` |
| Total DeFi TVL | `defi_metrics_latest` |
| Stablecoin Supply | `stablecoin_metrics_latest` |
| Open Alerts | `alerts` |
| Last Sync | `api_sync_logs` |

Example component:

```text
src/components/dashboard/KpiCard.vue
```

Props:

```js
defineProps({
  title: String,
  value: [String, Number],
  subtitle: String,
  icon: String,
  trend: String
})
```

---

## 4. Dashboard Store

Create `src/stores/dashboardStore.js`:

```js
import { defineStore } from 'pinia'
import api from '../services/api'

export const useDashboardStore = defineStore('dashboard', {
  state: () => ({
    overview: null,
    networks: [],
    alerts: [],
    loading: false,
    error: null
  }),

  actions: {
    async fetchOverview() {
      this.loading = true
      try {
        const response = await api.get('/dashboard/overview')
        this.overview = response.data
      } catch (error) {
        this.error = error.response?.data?.message || 'Failed to load dashboard'
      } finally {
        this.loading = false
      }
    },

    async fetchNetworks() {
      const response = await api.get('/networks')
      this.networks = response.data.data
    },

    async fetchAlerts() {
      const response = await api.get('/alerts')
      this.alerts = response.data.data
    }
  }
})
```

---

## 5. Network Grid

Create:

```text
src/components/dashboard/NetworkCard.vue
```

Display:

- Network name
- Symbol
- Type L1/L2
- Price
- 24h change
- TVL
- Stablecoin supply
- Link to detail page

---

## 6. Network Details Page

Route:

```text
/networks/:networkKey
```

Sections:

1. Network profile
2. Latest network metrics
3. Latest market metrics
4. Latest DeFi metrics
5. Latest stablecoin metrics
6. Price chart
7. TVL chart
8. Transaction chart
9. Related alerts

---

## 7. Chart Components

Create reusable chart component:

```text
src/components/charts/LineChart.vue
```

Use `vue-chartjs` and `chart.js`.

Props:

```js
defineProps({
  title: String,
  labels: Array,
  values: Array,
  unit: String
})
```

Supported chart types:

- Line chart for price history
- Line chart for TVL history
- Bar chart for transaction volume
- Doughnut chart for market share

---

## 8. Chart Data API Flow

Frontend request:

```js
api.get(`/networks/${networkKey}/history`, {
  params: {
    metric_group: 'market',
    metric_key: 'price_usd',
    range: '30d'
  }
})
```

Transform response:

```js
const labels = response.data.points.map(point => point.recorded_at)
const values = response.data.points.map(point => point.value)
```

---

## 9. Alerts Page

`AlertsPage.vue` should include:

- Filter by status
- Filter by severity
- Filter by network
- Alert table
- Acknowledge button
- Resolve button
- Alert details modal

Alert statuses:

```text
open
acknowledged
resolved
```

Alert severities:

```text
info
warning
critical
```

---

## 10. Reports Page UI

`ReportsPage.vue` should include:

- Report type dropdown
- Date range filters
- Network filter
- Export format dropdown
- Export button
- Export history table

Formats:

```text
CSV
XLSX
PDF
```

---

## 11. Admin Page UI

`AdminPage.vue` should include:

- API provider settings
- Sync logs table
- Failed sync filter
- Last successful sync by provider
- Manual sync button
- User management link

Admin-only route must be protected by frontend route guard and backend role middleware.

---

## 12. Loading and Error Components

Create:

```text
src/components/common/LoadingSpinner.vue
src/components/common/ErrorState.vue
src/components/common/EmptyState.vue
```

Use them across all pages to avoid blank screens.

---

## 13. UI Theme Rules

- Background: dark navy or black
- Cards: glass-style dark panels
- Text: high contrast
- Positive trend: visual up indicator
- Negative trend: visual down indicator
- Critical alerts: strong visual emphasis
- Mobile responsive layout

---

## 14. Phase Deliverables

- Dashboard overview page
- KPI cards
- Network cards
- Network details page
- Chart components
- Alerts page
- Reports UI
- Admin UI
- Loading and error states
- Responsive dark dashboard design

---

## 15. GitHub Commit

```bash
git add frontend/src docs/PHASE_11_VUE_DASHBOARD_PAGES_CHARTS_ALERTS_UI.md
git commit -m "Phase 11: add Vue dashboard pages charts and alerts UI"
git push
```
