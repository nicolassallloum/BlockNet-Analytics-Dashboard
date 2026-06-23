Phase 2 — System Architecture Design
BlockNet Analytics Dashboard
1. Step Objective

Design the complete technical architecture for BlockNet Analytics Dashboard based on the approved Phase 1 foundation.

This phase defines how the platform will be structured across:

Laravel REST API backend
Vue.js frontend
PostgreSQL database
Redis cache
Redis queue
Laravel Scheduler
External blockchain APIs
Authentication and authorization
Alerts
Reporting
Docker, Nginx, and Ubuntu VPS deployment
2. Required Files or Components

For Phase 2, the recommended documentation file is:

docs/phase-02-system-architecture-design.md

Recommended future implementation components:

backend/
frontend/
docker/
nginx/
docs/
.env
docker-compose.yml

Main architecture modules:

Authentication Module
User & Role Module
Blockchain Network Module
Metric Sync Module
External API Integration Module
Cache Module
Queue Module
Alert Engine Module
Reporting Module
Dashboard API Module
Deployment Module
1. Full Architecture Explanation

The BlockNet Analytics Dashboard will follow a modular client-server architecture.

The system will have five main layers:

Layer	Responsibility
Frontend Layer	Vue.js dashboard used by Admin, Analyst, and Viewer users
Backend API Layer	Laravel REST API that handles authentication, authorization, dashboard data, alerts, reports, and API responses
Data Processing Layer	Laravel Scheduler, Redis Queue, and Laravel Jobs that fetch and process blockchain metrics
Data Storage Layer	PostgreSQL for persistent storage and Redis for cache, queues, and temporary fast-access data
External Data Layer	CoinGecko, DefiLlama, Etherscan-like APIs, TronGrid, Solana APIs, and Bitcoin APIs

The external APIs provide blockchain and market data. CoinGecko can provide market data such as price, market cap, and volume through its market endpoints. DefiLlama provides DeFi-focused data and separates free and pro APIs, so the integration layer should keep DefiLlama configuration isolated from other data providers. Etherscan API V2 supports a unified multichain approach using a chainid parameter, which is useful for Ethereum and EVM-compatible networks. Solana data can come from Solana RPC, which provides methods to read network state and subscribe to live updates, or from Helius for enhanced parsed transaction and priority fee data. Bitcoin data can come from mempool.space REST APIs or Blockchain.com’s explorer data APIs.

The backend will not call all external APIs directly from dashboard requests. Instead, external data will be collected by scheduled jobs, normalized, saved into PostgreSQL, cached in Redis, and then served to the Vue.js frontend through Laravel API endpoints.

This avoids slow dashboards, reduces external API rate-limit problems, and keeps historical metrics available even if one external provider is temporarily unavailable.

2. Architecture Diagram in Text Format
+---------------------------------------------------------------+
|                        Users                                  |
|---------------------------------------------------------------|
| Admin                 Analyst                    Viewer       |
+-------------------------+-------------------------+-----------+
                          |
                          v
+---------------------------------------------------------------+
|                     Vue.js Frontend                           |
|---------------------------------------------------------------|
| Login Page                                                    |
| Dashboard Overview                                            |
| Network List                                                  |
| Network Detail Pages                                          |
| KPI Cards                                                     |
| Charts                                                        |
| Alerts                                                        |
| Reports                                                       |
| Admin Settings                                                |
+-------------------------+-------------------------------------+
                          |
                          | HTTPS REST API
                          v
+---------------------------------------------------------------+
|                    Nginx Reverse Proxy                        |
+-------------------------+-------------------------------------+
                          |
                          v
+---------------------------------------------------------------+
|                    Laravel REST API                           |
|---------------------------------------------------------------|
| Sanctum Authentication                                        |
| Role Authorization                                            |
| Dashboard Controllers                                         |
| Network Controllers                                           |
| Metrics Controllers                                           |
| Alerts Controllers                                            |
| Reports Controllers                                           |
| Admin Controllers                                             |
+-------------------------+-------------------------------------+
                          |
          +---------------+----------------+
          |                                |
          v                                v
+----------------------+        +-------------------------------+
| PostgreSQL Database  |        | Redis                         |
|----------------------|        |-------------------------------|
| users                |        | Cache                         |
| roles                |        | Queue                         |
| blockchain_networks  |        | Rate-limit counters           |
| network_metrics      |        | Last sync status              |
| market_metrics       |        | Dashboard summaries           |
| defi_metrics         |        +-------------------------------+
| alerts               |
| reports              |
| api_sync_logs        |
+----------------------+
          ^
          |
          v
+---------------------------------------------------------------+
|              Laravel Scheduler + Redis Queue                  |
|---------------------------------------------------------------|
| Fetch Market Data Jobs                                        |
| Fetch Network Data Jobs                                       |
| Fetch DeFi Data Jobs                                          |
| Fetch Stablecoin Data Jobs                                    |
| Evaluate Alerts Jobs                                          |
| Generate Reports Jobs                                         |
+-------------------------+-------------------------------------+
                          |
                          v
+---------------------------------------------------------------+
|                  External Blockchain APIs                     |
|---------------------------------------------------------------|
| CoinGecko                                                     |
| DefiLlama                                                     |
| Etherscan / BscScan / PolygonScan / BaseScan                  |
| Arbiscan / Optimistic Etherscan / Snowtrace                   |
| TronGrid                                                      |
| Helius / Solana RPC                                           |
| mempool.space / Blockchair / Blockchain.com                   |
+---------------------------------------------------------------+
3. Data Flow Diagram in Text Format
3.1 External API to PostgreSQL Flow
1. Laravel Scheduler starts sync command
   |
   v
2. Scheduler dispatches Redis Queue jobs
   |
   v
3. Each job calls one external API provider
   |
   v
4. API response is validated
   |
   v
5. Raw response is optionally logged
   |
   v
6. Data is normalized into internal metric format
   |
   v
7. Normalized metrics are saved into PostgreSQL
   |
   v
8. Latest summary values are saved into Redis cache
   |
   v
9. Alert Engine evaluates new metrics
   |
   v
10. Alerts are saved into PostgreSQL if triggered
3.2 PostgreSQL to Vue.js Dashboard Flow
1. User logs in from Vue.js
   |
   v
2. Laravel Sanctum returns authentication token or session
   |
   v
3. Vue.js calls protected dashboard API endpoints
   |
   v
4. Laravel checks user role and permission
   |
   v
5. Laravel checks Redis cache for dashboard data
   |
   +-----------------------------+
   | Cache hit                   |
   | Return data from Redis      |
   +-----------------------------+
   |
   +-----------------------------+
   | Cache miss                  |
   | Query PostgreSQL            |
   | Store result in Redis       |
   | Return data to frontend     |
   +-----------------------------+
   |
   v
6. Vue.js renders KPI cards, charts, tables, alerts, and reports
4. Backend Architecture

The backend will be a Laravel REST API.

4.1 Backend Responsibilities
Area	Responsibility
Authentication	Login, logout, current user, token/session handling
Authorization	Role-based access for Admin, Analyst, Viewer
Networks	Manage supported blockchain networks
Metrics	Store and serve blockchain, market, DeFi, fee, and stablecoin metrics
Integrations	Connect to external blockchain APIs
Scheduler	Trigger regular sync commands
Queue	Process long-running API sync jobs
Cache	Store latest dashboard summaries
Alerts	Detect abnormal changes and threshold breaches
Reports	Generate summary reports
Admin	Manage users, settings, API providers, and sync status
4.2 Main Backend Modules
Auth Module
User Module
Role Module
Blockchain Network Module
Metric Module
Market Data Module
DeFi Data Module
Network Data Module
Stablecoin Module
External Provider Module
Sync Job Module
Alert Module
Report Module
System Log Module
4.3 Backend Design Pattern

Recommended Laravel structure:

Controller -> Request Validation -> Service -> Repository/Model -> Resource Response

Example:

NetworkMetricController
    -> NetworkMetricRequest
        -> NetworkMetricService
            -> NetworkMetricRepository
                -> PostgreSQL
            -> Redis Cache
        -> NetworkMetricResource

This keeps controllers clean and makes future changes easier.

5. Frontend Architecture

The frontend will be a Vue.js single-page dashboard application.

5.1 Frontend Responsibilities
Area	Responsibility
Authentication UI	Login and logout
Dashboard UI	Main overview cards and charts
Network Pages	Individual blockchain details
Alerts UI	View, create, and manage alerts based on role
Reports UI	View and export reports
Admin UI	User, role, network, and API settings
API Client	Connect Vue.js to Laravel REST API
State Management	Store user session, filters, dashboard state, and network selections
Charting	Render KPIs using ApexCharts or ECharts
5.2 Main Frontend Screens
/login
/dashboard
/networks
/networks/:slug
/alerts
/reports
/admin/users
/admin/roles
/admin/networks
/admin/api-settings
/admin/sync-logs
5.3 Frontend Data Loading Strategy

The frontend should not call external blockchain APIs directly.

Correct flow:

Vue.js -> Laravel API -> Redis/PostgreSQL -> Laravel API Response -> Vue.js

This protects API keys, centralizes business logic, and gives the backend control over caching and rate limits.

6. Database Architecture

The database will use PostgreSQL as the main persistent storage.

6.1 Main Database Tables
Table	Purpose
users	System users
roles	User roles such as Admin, Analyst, Viewer
permissions	Fine-grained permissions
role_user	User-role relationship
blockchain_networks	Supported blockchain networks
api_providers	External API providers
api_provider_networks	Mapping between providers and networks
network_metrics	Latest and historical network metrics
market_metrics	Price, market cap, and volume metrics
defi_metrics	TVL, fees, revenue, DEX volume
stablecoin_metrics	Stablecoin supply and activity
block_metrics	Latest block, block time, block height
fee_metrics	Gas and transaction fee metrics
alert_rules	Alert configuration
alert_events	Triggered alerts
report_definitions	Report setup
generated_reports	Generated report records
api_sync_logs	API sync status and errors
system_logs	Internal system logs
6.2 Database Design Principle

Use separate metric tables by domain instead of storing everything in one large table.

Recommended split:

market_metrics
network_metrics
defi_metrics
stablecoin_metrics
fee_metrics
block_metrics

This makes the data easier to query, index, archive, and scale.

6.3 Metric Storage Strategy

Each metric row should include:

id
blockchain_network_id
metric_date
metric_timestamp
provider_id
value fields
raw_payload optional JSONB
created_at
updated_at

Use JSONB only for flexible raw payloads or provider-specific fields. Important dashboard KPIs should be stored in typed columns for better performance.

7. API Integration Architecture

External APIs should be integrated through provider-specific service classes.

7.1 Provider Groups
Provider Group	Networks / Data
CoinGecko	Price, market cap, 24h volume
DefiLlama	TVL, DeFi data, stablecoins, fees, revenue, DEX volume
Etherscan API V2 / EVM explorers	Ethereum, BNB Chain, Polygon, Base, Arbitrum, Optimism, Avalanche
TronGrid	TRON block, account, and transaction data
Helius / Solana RPC	Solana block, transaction, fee, and network data
mempool.space / Blockchair / Blockchain.com	Bitcoin block, transaction, and fee data

CoinGecko’s markets endpoint is suitable for loading market indicators such as price, market cap, and volume. DefiLlama tracks metrics such as TVL, fees, revenue, volume, and stablecoins across many chains and protocols. Etherscan API V2 is useful for EVM networks because it centralizes multichain access under one API model using chain IDs. TronGrid provides hosted access to the TRON network so the project does not need to run its own TRON node in Version 1.

7.2 API Integration Pattern
ExternalApiClient Interface
    |
    +-- CoinGeckoClient
    +-- DefiLlamaClient
    +-- EtherscanClient
    +-- TronGridClient
    +-- SolanaRpcClient
    +-- HeliusClient
    +-- BitcoinApiClient

Each client should support:

fetchLatestMetrics()
fetchHistoricalMetrics()
validateResponse()
normalizeResponse()
handleRateLimit()
handleError()
7.3 Normalization Layer

Each API returns different response structures. The Laravel backend should convert all responses into internal formats.

Example normalized market metric:

{
  "network": "ethereum",
  "symbol": "ETH",
  "price_usd": 3500.25,
  "market_cap_usd": 420000000000,
  "volume_24h_usd": 18000000000,
  "provider": "coingecko",
  "metric_timestamp": "2026-06-24T10:00:00Z"
}
8. Scheduler and Queue Architecture

The Laravel Scheduler will trigger recurring commands. The commands will dispatch Redis Queue jobs.

8.1 Scheduler Responsibilities
Schedule	Job Type	Purpose
Every 1 minute	Health checks	Check provider availability and system status
Every 5 minutes	Market metrics	Price, market cap, 24h volume
Every 5–10 minutes	Latest block metrics	Latest blocks and fees
Every 15 minutes	Network metrics	Transactions, active addresses where available
Every 30 minutes	DeFi metrics	TVL, fees, revenue, DEX volume
Every 1 hour	Stablecoin metrics	Stablecoin supply and activity
Daily	Reports	Generate daily summaries
Daily	Cleanup	Archive or clean old temporary logs
8.2 Queue Responsibilities

Redis Queue will process:

FetchCoinGeckoMarketMetricsJob
FetchDefiLlamaTvlMetricsJob
FetchEvmNetworkMetricsJob
FetchTronMetricsJob
FetchSolanaMetricsJob
FetchBitcoinMetricsJob
EvaluateAlertRulesJob
GenerateDailyReportJob
8.3 Why Queue Jobs Are Needed

External API calls can be slow or rate-limited. Queue jobs allow the dashboard to stay responsive while sync work runs separately from user requests.

9. Cache Architecture

Redis will be used for cache, queue, and temporary state.

9.1 Cache Usage
Cache Key Type	Example
Dashboard overview	dashboard:overview
Network latest metrics	network:ethereum:latest
Network chart data	network:bitcoin:chart:24h
Provider status	provider:coingecko:status
Sync lock	sync:coingecko:market:lock
User permissions	user:15:permissions
9.2 Cache Strategy

Use PostgreSQL as the source of truth and Redis as the fast-read layer.

Flow:

API request received
    |
    v
Check Redis cache
    |
    +-- Cache hit: return cached data
    |
    +-- Cache miss: query PostgreSQL
                  save result in Redis
                  return response
9.3 Recommended Cache TTL
Data Type	TTL
Dashboard overview	60 seconds
Latest market metrics	60–300 seconds
Network detail metrics	60–300 seconds
Chart data	5–15 minutes
Reports	1–24 hours
User permissions	5–30 minutes
Provider status	1–5 minutes
10. Authentication and Authorization Architecture

Authentication will use Laravel Sanctum.

10.1 Authentication Flow
1. User enters email and password in Vue.js
2. Vue.js sends login request to Laravel API
3. Laravel validates credentials
4. Laravel creates Sanctum token or session
5. Vue.js stores authentication state
6. Vue.js sends token/cookie with future API requests
7. Laravel protects routes using Sanctum middleware
10.2 Authorization Flow
1. Authenticated user requests API endpoint
2. Laravel identifies user role
3. Laravel checks permission
4. If allowed, request continues
5. If denied, API returns 403 Forbidden
10.3 Role Permissions
Module	Admin	Analyst	Viewer
Dashboard	Yes	Yes	Yes
Network pages	Yes	Yes	Yes
Alerts view	Yes	Yes	Yes
Alerts create	Yes	Yes	No
Alerts delete	Yes	No	No
Reports view	Yes	Yes	No
Users management	Yes	No	No
API settings	Yes	No	No
Manual sync	Yes	Yes	No
System logs	Yes	No	No
11. Alert Engine Architecture

The alert engine will detect abnormal or important changes.

11.1 Alert Rule Types
Alert Type	Example
Price movement	BTC price changes more than 5% in 1 hour
Volume spike	ETH 24h volume increases above threshold
Gas fee spike	Ethereum gas fee exceeds configured value
TVL drop	Network TVL drops more than 10%
Stablecoin change	Stablecoin supply changes suddenly
Provider failure	API provider fails multiple sync attempts
Network health warning	Latest block is delayed or unavailable
11.2 Alert Flow
1. New metrics are saved into PostgreSQL
2. Alert evaluation job starts
3. Job loads active alert rules
4. Job compares latest metric values with thresholds
5. If condition is true, alert event is created
6. Alert event appears in dashboard
7. Optional future step: email, Slack, or Telegram notification
11.3 Alert Tables
alert_rules
alert_events
alert_event_logs
12. Reporting Architecture

Reports will summarize blockchain network activity.

12.1 Report Types
Report	Description
Daily Network Summary	Daily KPIs per network
Market Performance Report	Price, market cap, and volume movement
DeFi Report	TVL, fees, revenue, and DEX volume
Stablecoin Report	Stablecoin metrics by network
Alert Report	Triggered alerts over selected period
Provider Health Report	API success/failure status
12.2 Report Flow
1. User selects report type and date range
2. Vue.js sends request to Laravel API
3. Laravel checks authorization
4. Laravel queries PostgreSQL
5. Laravel formats report response
6. Vue.js displays report
7. Future option: export CSV/PDF
12.3 Scheduled Reports

Daily reports can be generated by a queue job:

GenerateDailyNetworkReportJob
GenerateDailyAlertReportJob
GenerateProviderHealthReportJob
13. Deployment Architecture

Deployment will use Docker, Nginx, and Ubuntu VPS.

13.1 Production Components
Ubuntu VPS
    |
    +-- Docker
    |     |
    |     +-- backend-app container
    |     +-- frontend-app container
    |     +-- postgres container
    |     +-- redis container
    |     +-- queue-worker container
    |     +-- scheduler container
    |
    +-- Nginx reverse proxy
    |
    +-- SSL certificate
13.2 Deployment Diagram
Internet
   |
   v
+-------------------+
| Domain / DNS      |
+-------------------+
   |
   v
+-------------------+
| Nginx + SSL       |
+-------------------+
   |
   +--------------------------+
   |                          |
   v                          v
Frontend Container      Backend API Container
Vue.js                  Laravel REST API
   |                          |
   |                          +----------------+
   |                                           |
   v                                           v
Static Assets                             PostgreSQL
                                            |
                                            v
                                          Redis
                                            |
                                            v
                                  Queue + Scheduler Jobs
                                            |
                                            v
                                  External Blockchain APIs
13.3 Recommended Domains
app.blocknet-analytics.com
api.blocknet-analytics.com

For local development:

http://localhost:5173
http://localhost:8000
14. Recommended Folder Structure for Backend
backend/
├── app/
│   ├── Console/
│   │   └── Commands/
│   │       ├── SyncMarketMetricsCommand.php
│   │       ├── SyncNetworkMetricsCommand.php
│   │       ├── SyncDefiMetricsCommand.php
│   │       └── GenerateDailyReportsCommand.php
│   │
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── Auth/
│   │   │   ├── Dashboard/
│   │   │   ├── Networks/
│   │   │   ├── Metrics/
│   │   │   ├── Alerts/
│   │   │   ├── Reports/
│   │   │   └── Admin/
│   │   │
│   │   ├── Requests/
│   │   ├── Resources/
│   │   └── Middleware/
│   │
│   ├── Jobs/
│   │   ├── FetchCoinGeckoMarketMetricsJob.php
│   │   ├── FetchDefiLlamaMetricsJob.php
│   │   ├── FetchEvmNetworkMetricsJob.php
│   │   ├── FetchTronMetricsJob.php
│   │   ├── FetchSolanaMetricsJob.php
│   │   ├── FetchBitcoinMetricsJob.php
│   │   ├── EvaluateAlertRulesJob.php
│   │   └── GenerateReportJob.php
│   │
│   ├── Models/
│   │   ├── User.php
│   │   ├── Role.php
│   │   ├── Permission.php
│   │   ├── BlockchainNetwork.php
│   │   ├── ApiProvider.php
│   │   ├── MarketMetric.php
│   │   ├── NetworkMetric.php
│   │   ├── DefiMetric.php
│   │   ├── StablecoinMetric.php
│   │   ├── FeeMetric.php
│   │   ├── AlertRule.php
│   │   ├── AlertEvent.php
│   │   └── ApiSyncLog.php
│   │
│   ├── Services/
│   │   ├── Auth/
│   │   ├── Dashboard/
│   │   ├── Metrics/
│   │   ├── Alerts/
│   │   ├── Reports/
│   │   └── ExternalApis/
│   │       ├── CoinGeckoClient.php
│   │       ├── DefiLlamaClient.php
│   │       ├── EtherscanClient.php
│   │       ├── TronGridClient.php
│   │       ├── SolanaRpcClient.php
│   │       ├── HeliusClient.php
│   │       └── BitcoinApiClient.php
│   │
│   └── Repositories/
│       ├── NetworkRepository.php
│       ├── MetricRepository.php
│       ├── AlertRepository.php
│       └── ReportRepository.php
│
├── config/
│   ├── blockchain.php
│   ├── external-apis.php
│   └── metrics.php
│
├── database/
│   ├── migrations/
│   ├── seeders/
│   └── factories/
│
├── routes/
│   ├── api.php
│   └── console.php
│
├── tests/
│   ├── Feature/
│   └── Unit/
│
├── Dockerfile
├── composer.json
└── .env.example
15. Recommended Folder Structure for Frontend
frontend/
├── src/
│   ├── api/
│   │   ├── client.js
│   │   ├── authApi.js
│   │   ├── dashboardApi.js
│   │   ├── networksApi.js
│   │   ├── metricsApi.js
│   │   ├── alertsApi.js
│   │   └── reportsApi.js
│   │
│   ├── assets/
│   │   ├── images/
│   │   └── styles/
│   │
│   ├── components/
│   │   ├── layout/
│   │   ├── cards/
│   │   ├── charts/
│   │   ├── tables/
│   │   ├── forms/
│   │   └── alerts/
│   │
│   ├── layouts/
│   │   ├── AuthLayout.vue
│   │   └── DashboardLayout.vue
│   │
│   ├── pages/
│   │   ├── auth/
│   │   │   └── Login.vue
│   │   ├── dashboard/
│   │   │   └── DashboardOverview.vue
│   │   ├── networks/
│   │   │   ├── NetworkList.vue
│   │   │   └── NetworkDetails.vue
│   │   ├── alerts/
│   │   │   └── AlertsPage.vue
│   │   ├── reports/
│   │   │   └── ReportsPage.vue
│   │   └── admin/
│   │       ├── UsersPage.vue
│   │       ├── RolesPage.vue
│   │       ├── NetworksPage.vue
│   │       ├── ApiSettingsPage.vue
│   │       └── SyncLogsPage.vue
│   │
│   ├── router/
│   │   └── index.js
│   │
│   ├── stores/
│   │   ├── authStore.js
│   │   ├── dashboardStore.js
│   │   ├── networkStore.js
│   │   └── alertStore.js
│   │
│   ├── utils/
│   │   ├── formatters.js
│   │   ├── permissions.js
│   │   └── constants.js
│   │
│   ├── App.vue
│   └── main.js
│
├── public/
├── Dockerfile
├── package.json
├── vite.config.js
└── .env.example
16. Environment Variables List
16.1 Backend .env
APP_NAME="BlockNet Analytics Dashboard"
APP_ENV=local
APP_KEY=
APP_DEBUG=true
APP_URL=http://localhost:8000

FRONTEND_URL=http://localhost:5173
SANCTUM_STATEFUL_DOMAINS=localhost:5173
SESSION_DOMAIN=localhost

DB_CONNECTION=pgsql
DB_HOST=postgres
DB_PORT=5432
DB_DATABASE=blocknet_analytics
DB_USERNAME=postgres
DB_PASSWORD=postgres

CACHE_DRIVER=redis
QUEUE_CONNECTION=redis
SESSION_DRIVER=redis

REDIS_HOST=redis
REDIS_PASSWORD=null
REDIS_PORT=6379

COINGECKO_API_BASE_URL=https://api.coingecko.com/api/v3
COINGECKO_API_KEY=

DEFILLAMA_API_BASE_URL=https://api.llama.fi
DEFILLAMA_STABLECOINS_API_BASE_URL=https://stablecoins.llama.fi
DEFILLAMA_FEES_API_BASE_URL=https://api.llama.fi

ETHERSCAN_API_BASE_URL=https://api.etherscan.io/v2/api
ETHERSCAN_API_KEY=

TRONGRID_API_BASE_URL=https://api.trongrid.io
TRONGRID_API_KEY=

SOLANA_RPC_URL=
HELIUS_API_BASE_URL=https://api.helius.xyz
HELIUS_API_KEY=

BITCOIN_API_PROVIDER=mempool
MEMPOOL_API_BASE_URL=https://mempool.space/api
BLOCKCHAIR_API_BASE_URL=https://api.blockchair.com
BLOCKCHAIN_COM_API_BASE_URL=https://blockchain.info

SYNC_MARKET_METRICS_ENABLED=true
SYNC_NETWORK_METRICS_ENABLED=true
SYNC_DEFI_METRICS_ENABLED=true
SYNC_ALERTS_ENABLED=true

LOG_CHANNEL=stack
LOG_LEVEL=debug
16.2 Frontend .env
VITE_APP_NAME="BlockNet Analytics Dashboard"
VITE_API_BASE_URL=http://localhost:8000/api
VITE_APP_ENV=local
17. Security Considerations
17.1 API Key Security

External API keys must be stored only in backend .env files.

Do not expose these keys in Vue.js.

Correct:

Vue.js -> Laravel API -> External API

Incorrect:

Vue.js -> External API directly
17.2 Authentication Security

Use:

Laravel Sanctum
HTTPS in production
Secure cookies or secure token handling
CSRF protection where applicable
Strong password hashing
Login throttling
17.3 Authorization Security

Every sensitive endpoint must check role permissions.

Examples:

/admin/users          Admin only
/admin/api-settings   Admin only
/alerts/create        Admin and Analyst
/reports/export       Admin and Analyst
/dashboard            Admin, Analyst, Viewer
17.4 External API Security

Add:

Timeouts
Retries
Rate limit handling
Provider failure logs
Request validation
Response validation
Fallback provider support where possible
17.5 Database Security

Use:

Strong database password
Private Docker network
No public PostgreSQL port in production
Regular backups
Limited database users
Indexes for performance
JSONB only where needed
17.6 Deployment Security

Use:

HTTPS SSL certificate
Nginx security headers
Firewall rules
Closed unused ports
Production APP_DEBUG=false
Separate production .env
Regular package updates
18. Scalability Considerations
18.1 Application Scalability

The system should be designed so these services can scale separately:

Frontend
Backend API
Queue Workers
Scheduler
PostgreSQL
Redis
18.2 Queue Scaling

If sync jobs become heavy, add more queue workers:

queue-worker-1
queue-worker-2
queue-worker-3
18.3 Database Scaling

Recommended database optimizations:

Indexes on blockchain_network_id
Indexes on metric_timestamp
Indexes on provider_id
Partition large historical metric tables by month
Archive old raw payloads
Use materialized views later for heavy dashboards
18.4 Cache Scaling

Use Redis for:

Dashboard latest summaries
Network latest metrics
Chart datasets
Provider status
Permission cache
Sync locks
18.5 Provider Scaling

Use provider adapters so that future APIs can be added without changing dashboard logic.

Example:

BitcoinApiClient
    -> MempoolSpaceProvider
    -> BlockchairProvider
    -> BlockchainComProvider
18.6 Future Real-Time Upgrade

Version 1 should use scheduled sync.

Future versions can add:

WebSocket updates
Server-sent events
Live mempool monitoring
Direct node integrations
Full blockchain indexing
19. Final Phase 2 Checklist
[x] Full architecture explanation defined
[x] Architecture diagram completed
[x] Data flow diagram completed
[x] Backend architecture defined
[x] Frontend architecture defined
[x] Database architecture defined
[x] API integration architecture defined
[x] Scheduler architecture defined
[x] Queue architecture defined
[x] Cache architecture defined
[x] Authentication architecture defined
[x] Authorization architecture defined
[x] Alert engine architecture defined
[x] Reporting architecture defined
[x] Deployment architecture defined
[x] Backend folder structure defined
[x] Frontend folder structure defined
[x] Environment variables listed
[x] Security considerations defined
[x] Scalability considerations defined
[x] Data movement from external APIs to PostgreSQL explained
[x] Data movement from PostgreSQL to Vue.js dashboard explained
[x] Phase 2 ready for review
