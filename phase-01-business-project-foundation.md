# Phase 1 — Business & Project Foundation

# BlockNet Analytics Dashboard

## 1. Project Overview

**BlockNet Analytics Dashboard** is an online blockchain monitoring platform designed to provide a centralized view of major blockchain networks.

The dashboard will allow users to monitor blockchain activity, financial indicators, network health, DeFi metrics, stablecoin activity, and alerts from one place.

The platform will be built using:

| Layer | Technology |
|---|---|
| Backend | Laravel REST API |
| Frontend | Vue.js |
| Database | PostgreSQL |
| Cache / Queue | Redis |
| Authentication | Laravel Sanctum |
| Charts | ApexCharts or ECharts |
| Deployment | Docker, Nginx, Ubuntu VPS |
| Data Sources | CoinGecko, DefiLlama, Etherscan-like APIs, TronGrid, Solana API, Bitcoin API |

---

## 2. Business Objective

The business objective of **BlockNet Analytics Dashboard** is to provide a reliable online platform for monitoring major blockchain networks in one unified dashboard.

The dashboard should help users:

- Track blockchain market performance
- Monitor network activity
- Compare multiple blockchain ecosystems
- Detect abnormal changes
- View DeFi and stablecoin indicators
- Support analysis and decision-making
- Reduce manual checking across multiple external platforms

---

## 3. Target Users and Permissions

### 3.1 Initial User Roles

| Role | Description |
|---|---|
| Admin | Manages users, roles, networks, API settings, alerts, and system configuration |
| Analyst | Reviews blockchain data, KPIs, charts, reports, alerts, and trends |
| Viewer | Views dashboards and basic blockchain metrics only |

### 3.2 Permission Matrix

| Permission / Feature | Admin | Analyst | Viewer |
|---|---:|---:|---:|
| Login to dashboard | Yes | Yes | Yes |
| View dashboard overview | Yes | Yes | Yes |
| View blockchain network pages | Yes | Yes | Yes |
| View charts and KPIs | Yes | Yes | Yes |
| View alerts | Yes | Yes | Yes |
| Create alerts | Yes | Yes | No |
| Edit alerts | Yes | Yes | No |
| Delete alerts | Yes | No | No |
| Manage users | Yes | No | No |
| Manage roles | Yes | No | No |
| Manage API settings | Yes | No | No |
| Manage supported networks | Yes | No | No |
| Run manual data sync | Yes | Yes | No |
| Export reports | Yes | Yes | No |
| View system logs | Yes | No | No |

### 3.3 Future User Roles

| Future Role | Purpose |
|---|---|
| Compliance Officer | Monitors risky activity, stablecoin movements, suspicious patterns, and compliance alerts |
| Developer | Manages API integrations, sync jobs, logs, technical diagnostics, and developer tools |
| Bank Manager | Views executive-level blockchain risk, adoption, market, and network summaries |

---

## 4. Blockchain Networks List

The first version of the dashboard will support the following blockchain networks:

| # | Network | Symbol | Category | Version 1 Status |
|---:|---|---|---|---|
| 1 | Bitcoin | BTC | Layer 1 | Included |
| 2 | Ethereum | ETH | Layer 1 / Smart Contract | Included |
| 3 | BNB Chain | BNB | Layer 1 / EVM | Included |
| 4 | Solana | SOL | Layer 1 | Included |
| 5 | TRON | TRX | Layer 1 | Included |
| 6 | Polygon | POL / MATIC | Ethereum Scaling | Included |
| 7 | Base | ETH | Ethereum Layer 2 | Included |
| 8 | Arbitrum | ARB / ETH | Ethereum Layer 2 | Included |
| 9 | Optimism | OP / ETH | Ethereum Layer 2 | Included |
| 10 | Avalanche | AVAX | Layer 1 | Included |

---

## 5. Dashboard KPIs

The dashboard will track the following key performance indicators.

| KPI Group | KPI | Description |
|---|---|---|
| Market | Price | Current asset price |
| Market | Market Cap | Total market capitalization |
| Market | 24h Volume | Trading volume during the last 24 hours |
| Blockchain | Latest Block | Latest block height or block number |
| Blockchain | Transactions | Number of transactions |
| Blockchain | Active Addresses | Active wallet addresses |
| Fees | Gas / Fees | Gas price or average transaction fee |
| DeFi | TVL | Total value locked in DeFi protocols |
| Stablecoins | Stablecoin Metrics | Stablecoin supply or stablecoin activity |
| Monitoring | Health Status | Healthy, warning, or critical status |
| Monitoring | Alerts | Notifications for abnormal changes or threshold breaches |

---

## 6. MVP Scope

The MVP will focus on creating a usable dashboard that tracks the most important blockchain indicators.

### 6.1 Authentication and Access

- Laravel Sanctum login and logout
- Admin, Analyst, and Viewer roles
- Protected API routes
- Protected frontend pages
- Role-based access control

### 6.2 Blockchain Network Dashboard

- Dashboard overview page
- Network list page
- Individual network details page
- KPIs per blockchain network
- Charts and visual comparisons
- Health status indicator per network

### 6.3 Data Collection

- CoinGecko integration for price, market cap, and volume
- DefiLlama integration for TVL
- Etherscan-like APIs for EVM network data
- TronGrid integration for TRON data
- Solana API integration for Solana data
- Bitcoin API integration for Bitcoin block and network data

### 6.4 Backend

- Laravel REST API
- PostgreSQL database
- Redis cache
- Redis queue jobs
- Scheduled API sync commands
- Error logging
- API response structure
- Data validation

### 6.5 Frontend

- Vue.js dashboard
- KPI cards
- Network comparison charts
- Alert display
- Network details screen
- Role-based navigation
- Responsive layout

### 6.6 Deployment

- Docker containers
- Nginx reverse proxy
- Ubuntu VPS hosting
- Environment-based configuration
- Production-ready `.env` structure
- Basic deployment documentation

---

## 7. Out-of-Scope Features for Version 1

The following features are not part of Version 1.

| Feature | Version 1 Decision |
|---|---|
| Running full blockchain nodes | Excluded |
| Full blockchain indexing | Excluded |
| Wallet investigation module | Excluded |
| AML / compliance scoring | Excluded |
| AI prediction engine | Excluded |
| Trading features | Excluded |
| User wallet connection | Excluded |
| Portfolio tracking | Excluded |
| NFT analytics | Excluded |
| Smart contract audit engine | Excluded |
| Mobile app | Excluded |
| Multi-tenant SaaS billing | Excluded |
| Advanced permission builder | Excluded |
| Real-time WebSocket streaming for all data | Excluded initially |

---

## 8. Success Criteria

Version 1 will be considered successful when:

| # | Success Criteria |
|---:|---|
| 1 | Users can log in securely using Laravel Sanctum |
| 2 | Admin can manage users and roles |
| 3 | Dashboard displays the 10 initial blockchain networks |
| 4 | Each network has a detail page |
| 5 | Dashboard displays price, market cap, and 24h volume |
| 6 | Dashboard displays latest block and transaction metrics |
| 7 | Dashboard displays gas or fee metrics where available |
| 8 | Dashboard displays TVL and stablecoin indicators where available |
| 9 | Redis is used for cache and queue jobs |
| 10 | PostgreSQL stores networks, metrics, alerts, and users |
| 11 | Charts are available for important KPIs |
| 12 | Alerts can be created and viewed |
| 13 | Docker deployment works on Ubuntu VPS |
| 14 | The MVP is stable enough for future compliance and advanced analytics modules |

---

## 9. Phase 1 Validation Test

Use the following checklist to validate that Phase 1 is complete.

```text
[ ] Project name confirmed
[ ] Business objective confirmed
[ ] Initial users confirmed
[ ] Future users identified
[ ] Initial blockchain networks confirmed
[ ] Main KPIs confirmed
[ ] MVP scope confirmed
[ ] Out-of-scope features confirmed
[ ] Success criteria confirmed
[ ] Ready to move to Phase 2
```

---

## 10. Final Phase 1 Checklist

```text
[x] Project name defined: BlockNet Analytics Dashboard
[x] Business objective defined
[x] Dashboard purpose defined
[x] Initial users defined: Admin, Analyst, Viewer
[x] Future users defined: Compliance Officer, Developer, Bank Manager
[x] Initial blockchain networks defined
[x] Main KPIs defined
[x] MVP scope defined
[x] Version 1 out-of-scope features defined
[x] Success criteria defined
[x] Phase 1 ready for review
```

---

## 11. Expected Output

At the end of Phase 1, the project should have a clear foundation document.

```text
BlockNet Analytics Dashboard has a confirmed business objective, user roles, supported networks, KPIs, MVP scope, version 1 boundaries, and success criteria.
```

---

## 12. Next Recommended Step

The next recommended step is:

```text
Phase 2 — System Architecture & Technical Design
```

Phase 2 should define:

- High-level architecture
- Backend structure
- Frontend structure
- PostgreSQL schema direction
- Redis usage
- API integration flow
- Queue and scheduler design
- Docker deployment architecture

---

## Phase Status

```text
Phase 1 completed.
Waiting for approval before starting Phase 2.
```
