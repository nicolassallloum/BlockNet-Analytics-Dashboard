# BlockNet Analytics Dashboard

**Project:** Blockchain Networks Analytics Dashboard  
**Database:** `blocknet_dashboard`  
**Schema:** `blockchain_dashboard`  
**Stack:** PostgreSQL, Laravel, Vue, Docker, Nginx, Ubuntu VPS  
**Initial Networks:** Bitcoin, Ethereum, BNB Chain, Solana, TRON, Polygon, Base, Arbitrum

---

# Phase 9 — Dashboard API Endpoints and Controllers

## 1. Phase Objective

Create secure REST API endpoints for the Vue dashboard.

This phase exposes data from PostgreSQL to the frontend using clean controllers, resources, filters, pagination, and role-protected routes.

---

## 2. Required API Modules

| Module | Purpose |
|--------|---------|
| Dashboard Overview | KPI cards and summary metrics |
| Networks | List and detail views |
| Latest Metrics | Current network, market, DeFi, stablecoin metrics |
| Historical Charts | Time-series data |
| Alerts | Open/resolved alerts |
| Reports | Export requests and history |
| API Sync Logs | Admin monitoring |
| Provider Settings | Admin provider configuration |

---

## 3. Controllers to Create

```bash
php artisan make:controller Api/V1/DashboardController
php artisan make:controller Api/V1/BlockchainNetworkController
php artisan make:controller Api/V1/MetricController
php artisan make:controller Api/V1/ChartController
php artisan make:controller Api/V1/AlertController
php artisan make:controller Api/V1/ReportController
php artisan make:controller Api/V1/ApiSyncLogController
php artisan make:controller Api/V1/ApiProviderSettingController
```

---

## 4. API Route Map

| Method | Endpoint | Role | Purpose |
|--------|----------|------|---------|
| GET | `/api/v1/dashboard/overview` | viewer+ | Dashboard KPIs |
| GET | `/api/v1/networks` | viewer+ | Active networks |
| GET | `/api/v1/networks/{network}` | viewer+ | Network profile |
| GET | `/api/v1/networks/{network}/latest` | viewer+ | Latest metrics |
| GET | `/api/v1/networks/{network}/history` | viewer+ | Chart history |
| GET | `/api/v1/alerts` | viewer+ | Alerts list |
| POST | `/api/v1/alerts/{alert}/acknowledge` | analyst+ | Acknowledge alert |
| POST | `/api/v1/alerts/{alert}/resolve` | analyst+ | Resolve alert |
| GET | `/api/v1/reports/exports` | analyst+ | Export history |
| POST | `/api/v1/reports/export` | analyst+ | Create export |
| GET | `/api/v1/admin/sync-logs` | admin | API sync logs |
| GET | `/api/v1/admin/provider-settings` | admin | Provider list |
| PUT | `/api/v1/admin/provider-settings/{id}` | admin | Update provider |

---

## 5. Route Definition

Update `routes/api.php`:

```php
Route::middleware('auth:sanctum')->prefix('v1')->group(function () {
    Route::middleware('role:admin,analyst,viewer')->group(function () {
        Route::get('/dashboard/overview', [DashboardController::class, 'overview']);
        Route::get('/networks', [BlockchainNetworkController::class, 'index']);
        Route::get('/networks/{network}', [BlockchainNetworkController::class, 'show']);
        Route::get('/networks/{network}/latest', [MetricController::class, 'latest']);
        Route::get('/networks/{network}/history', [ChartController::class, 'history']);
        Route::get('/alerts', [AlertController::class, 'index']);
    });

    Route::middleware('role:admin,analyst')->group(function () {
        Route::post('/alerts/{alert}/acknowledge', [AlertController::class, 'acknowledge']);
        Route::post('/alerts/{alert}/resolve', [AlertController::class, 'resolve']);
        Route::get('/reports/exports', [ReportController::class, 'index']);
        Route::post('/reports/export', [ReportController::class, 'export']);
    });

    Route::middleware('role:admin')->prefix('admin')->group(function () {
        Route::get('/sync-logs', [ApiSyncLogController::class, 'index']);
        Route::get('/provider-settings', [ApiProviderSettingController::class, 'index']);
        Route::put('/provider-settings/{setting}', [ApiProviderSettingController::class, 'update']);
    });
});
```

---

## 6. Dashboard Overview Response

Example response:

```json
{
  "networks_count": 8,
  "total_market_cap_usd": 2450000000000,
  "total_tvl_usd": 95000000000,
  "total_stablecoin_supply_usd": 160000000000,
  "open_alerts_count": 3,
  "last_sync_at": "2026-01-01T12:00:00Z",
  "top_networks_by_market_cap": [],
  "top_networks_by_tvl": []
}
```

---

## 7. Network List Response

```json
{
  "data": [
    {
      "id": 1,
      "network_key": "bitcoin",
      "name": "Bitcoin",
      "native_symbol": "BTC",
      "network_type": "L1",
      "is_active": true
    }
  ]
}
```

---

## 8. History Query Parameters

Endpoint:

```text
GET /api/v1/networks/{network}/history
```

Supported query parameters:

| Parameter | Example | Purpose |
|----------|---------|---------|
| `metric_group` | `market` | Filter group |
| `metric_key` | `price_usd` | Filter metric |
| `range` | `24h`, `7d`, `30d`, `90d`, `1y` | Chart period |
| `interval` | `hour`, `day` | Aggregation interval |

Example:

```bash
curl "http://127.0.0.1:8000/api/v1/networks/bitcoin/history?metric_group=market&metric_key=price_usd&range=30d"
```

---

## 9. Chart Response Format

```json
{
  "network": "bitcoin",
  "metric_group": "market",
  "metric_key": "price_usd",
  "range": "30d",
  "points": [
    {
      "recorded_at": "2026-01-01T00:00:00Z",
      "value": 100000.55
    }
  ]
}
```

---

## 10. Pagination Standard

Use Laravel pagination for logs, alerts, and exports.

```text
?page=1&per_page=25
```

Maximum page size:

```text
100
```

---

## 11. API Resource Classes

Create resources:

```bash
php artisan make:resource BlockchainNetworkResource
php artisan make:resource LatestMetricsResource
php artisan make:resource AlertResource
php artisan make:resource ApiSyncLogResource
php artisan make:resource ReportExportResource
```

Benefits:

- Consistent JSON
- Field hiding
- Cleaner controllers
- Safer frontend contracts

---

## 12. Response Rules

- All successful responses must be JSON.
- All validation errors must return HTTP 422.
- Unauthorized requests must return HTTP 401.
- Forbidden requests must return HTTP 403.
- Missing records must return HTTP 404.
- Server errors must be logged and return HTTP 500 with generic message.

---

## 13. Phase Deliverables

- Dashboard overview API
- Network list and detail APIs
- Latest metrics API
- Historical chart API
- Alert APIs
- Report APIs
- Admin sync log APIs
- Provider settings APIs
- Resource classes and pagination standards

---

## 14. GitHub Commit

```bash
git add backend/app/Http/Controllers/Api/V1 backend/app/Http/Resources/Api/V1 backend/routes/api.php docs/PHASE_09_DASHBOARD_API_ENDPOINTS_CONTROLLERS.md
git commit -m "Phase 9: add dashboard API endpoints and controllers"
git push
```
