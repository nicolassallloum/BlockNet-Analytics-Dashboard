# Phase 3 — PostgreSQL Database Design

# BlockNet Analytics Dashboard

## 1. Step Objective

Design the full PostgreSQL database structure for **BlockNet Analytics Dashboard**.

The database must support:

- Latest dashboard KPI cards
- Historical charts
- Network comparison table
- Network detail page
- Alerts
- Reports
- API sync logs
- Users and roles
- Audit logs
- API provider settings

---

## 2. Database Information

| Item | Value |
|---|---|
| Database Name | `blocknet_dashboard` |
| Schema Name | `blockchain_dashboard` |
| Main Database Engine | PostgreSQL |
| Main Application | BlockNet Analytics Dashboard |
| Backend Framework | Laravel REST API |
| Frontend Framework | Vue.js |

---

## 3. Required Tables

The Phase 3 database design includes the following tables:

| # | Table Name | Purpose |
|---:|---|---|
| 1 | `blockchain_networks` | Stores supported blockchain networks |
| 2 | `network_metrics_latest` | Stores latest network metrics |
| 3 | `market_metrics_latest` | Stores latest price, market cap, and volume data |
| 4 | `defi_metrics_latest` | Stores latest TVL, DEX volume, fees, and revenue data |
| 5 | `stablecoin_metrics_latest` | Stores latest stablecoin supply and activity metrics |
| 6 | `network_metric_history` | Stores historical time-series metrics |
| 7 | `alerts` | Stores triggered alerts |
| 8 | `users` | Stores application users |
| 9 | `roles` | Stores user roles |
| 10 | `user_roles` | Stores user-role relationships |
| 11 | `audit_logs` | Stores user and system audit actions |
| 12 | `api_sync_logs` | Stores external API sync logs |
| 13 | `report_exports` | Stores generated report export records |
| 14 | `api_provider_settings` | Stores external API provider configuration |

---

## 4. Database Design Overview

The database is designed around two metric layers:

| Layer | Purpose |
|---|---|
| Latest Metrics Tables | Store the most recent values for fast dashboard cards and network comparison tables |
| Historical Metrics Table | Store time-series values for charts, reports, trends, and historical analysis |

The design separates market, network, DeFi, and stablecoin metrics into different latest tables.

This gives the dashboard better performance because the main cards and comparison screens can read from small latest tables instead of scanning large historical tables.

Main data movement:

```text
External APIs
    -> Laravel Scheduler
        -> Redis Queue Jobs
            -> API Client Services
                -> Normalize Response
                    -> Save Latest Metrics
                    -> Save Historical Metrics
                        -> Serve Laravel API
                            -> Display in Vue.js Dashboard
```

---

## 5. Entity Relationship Explanation

### 5.1 Core Blockchain Relationships

```text
blockchain_networks
    1 -> 1 network_metrics_latest
    1 -> 1 market_metrics_latest
    1 -> 1 defi_metrics_latest
    1 -> 1 stablecoin_metrics_latest
    1 -> many network_metric_history
    1 -> many alerts
    1 -> many api_sync_logs
```

### 5.2 User and Role Relationships

```text
users
    many -> many roles through user_roles

users
    1 -> many audit_logs
    1 -> many report_exports
    1 -> many alerts created
```

### 5.3 API Provider Relationships

```text
api_provider_settings
    stores provider configuration

api_sync_logs
    records each sync attempt by provider and network
```

---

## 6. Full CREATE DATABASE Command

Run this command using a PostgreSQL admin user:

```sql
CREATE DATABASE blocknet_dashboard
WITH
    ENCODING = 'UTF8'
    TEMPLATE = template0;
```

Connect to the database:

```bash
psql -U postgres -d blocknet_dashboard
```

---

## 7. Full CREATE SCHEMA Command

```sql
CREATE SCHEMA IF NOT EXISTS blockchain_dashboard;

SET search_path TO blockchain_dashboard, public;
```

---

## 8. Full CREATE TABLE Scripts

Save the following SQL into:

```text
database/init/01_create_blocknet_dashboard_schema.sql
```

```sql
-- ============================================================
-- BlockNet Analytics Dashboard
-- Phase 3: PostgreSQL Database Design
-- Database: blocknet_dashboard
-- Schema: blockchain_dashboard
-- ============================================================

CREATE SCHEMA IF NOT EXISTS blockchain_dashboard;

SET search_path TO blockchain_dashboard, public;

-- ============================================================
-- 1. Roles
-- ============================================================

CREATE TABLE IF NOT EXISTS roles (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(100) NOT NULL,
    description TEXT,
    is_system BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT uq_roles_name UNIQUE (name),
    CONSTRAINT uq_roles_slug UNIQUE (slug)
);

-- ============================================================
-- 2. Users
-- ============================================================

CREATE TABLE IF NOT EXISTS users (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(150) NOT NULL,
    email VARCHAR(255) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    status VARCHAR(30) NOT NULL DEFAULT 'active',
    last_login_at TIMESTAMPTZ,
    email_verified_at TIMESTAMPTZ,
    remember_token VARCHAR(100),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT uq_users_email UNIQUE (email),
    CONSTRAINT chk_users_status CHECK (status IN ('active', 'inactive', 'blocked'))
);

-- ============================================================
-- 3. User Roles
-- ============================================================

CREATE TABLE IF NOT EXISTS user_roles (
    user_id BIGINT NOT NULL,
    role_id BIGINT NOT NULL,
    assigned_by BIGINT,
    assigned_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT pk_user_roles PRIMARY KEY (user_id, role_id),

    CONSTRAINT fk_user_roles_user
        FOREIGN KEY (user_id)
        REFERENCES users (id)
        ON DELETE CASCADE,

    CONSTRAINT fk_user_roles_role
        FOREIGN KEY (role_id)
        REFERENCES roles (id)
        ON DELETE CASCADE,

    CONSTRAINT fk_user_roles_assigned_by
        FOREIGN KEY (assigned_by)
        REFERENCES users (id)
        ON DELETE SET NULL
);

-- ============================================================
-- 4. Blockchain Networks
-- ============================================================

CREATE TABLE IF NOT EXISTS blockchain_networks (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(150) NOT NULL,
    slug VARCHAR(150) NOT NULL,
    symbol VARCHAR(30) NOT NULL,
    native_asset VARCHAR(30) NOT NULL,
    category VARCHAR(100) NOT NULL,
    chain_family VARCHAR(50) NOT NULL,
    chain_id BIGINT,
    is_evm BOOLEAN NOT NULL DEFAULT FALSE,
    explorer_url TEXT,
    rpc_url TEXT,
    logo_url TEXT,
    sort_order INTEGER NOT NULL DEFAULT 0,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    metadata JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT uq_blockchain_networks_name UNIQUE (name),
    CONSTRAINT uq_blockchain_networks_slug UNIQUE (slug),
    CONSTRAINT chk_blockchain_networks_chain_family
        CHECK (chain_family IN ('bitcoin', 'evm', 'solana', 'tron', 'other'))
);

-- ============================================================
-- 5. API Provider Settings
-- ============================================================

CREATE TABLE IF NOT EXISTS api_provider_settings (
    id BIGSERIAL PRIMARY KEY,
    provider_name VARCHAR(150) NOT NULL,
    provider_slug VARCHAR(150) NOT NULL,
    base_url TEXT NOT NULL,
    api_key_env_name VARCHAR(150),
    rate_limit_per_minute INTEGER,
    timeout_seconds INTEGER NOT NULL DEFAULT 30,
    retry_count INTEGER NOT NULL DEFAULT 3,
    priority INTEGER NOT NULL DEFAULT 1,
    is_enabled BOOLEAN NOT NULL DEFAULT TRUE,
    config JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT uq_api_provider_settings_slug UNIQUE (provider_slug)
);

-- ============================================================
-- 6. Network Metrics Latest
-- Purpose:
-- Latest block, transactions, active addresses, gas/fees,
-- and health status per blockchain network.
-- ============================================================

CREATE TABLE IF NOT EXISTS network_metrics_latest (
    id BIGSERIAL PRIMARY KEY,
    blockchain_network_id BIGINT NOT NULL,
    latest_block_number NUMERIC(40, 0),
    latest_block_hash VARCHAR(150),
    block_time_seconds NUMERIC(18, 6),
    transactions_24h NUMERIC(30, 0),
    active_addresses_24h NUMERIC(30, 0),
    avg_transaction_fee_usd NUMERIC(30, 10),
    avg_gas_price_gwei NUMERIC(30, 10),
    health_status VARCHAR(30) NOT NULL DEFAULT 'unknown',
    health_message TEXT,
    data_source VARCHAR(150),
    metric_timestamp TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    raw_payload JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT uq_network_metrics_latest_network UNIQUE (blockchain_network_id),

    CONSTRAINT fk_network_metrics_latest_network
        FOREIGN KEY (blockchain_network_id)
        REFERENCES blockchain_networks (id)
        ON DELETE CASCADE,

    CONSTRAINT chk_network_metrics_latest_health_status
        CHECK (health_status IN ('healthy', 'warning', 'critical', 'unknown'))
);

-- ============================================================
-- 7. Market Metrics Latest
-- Purpose:
-- Price, market cap, volume, and supply metrics.
-- ============================================================

CREATE TABLE IF NOT EXISTS market_metrics_latest (
    id BIGSERIAL PRIMARY KEY,
    blockchain_network_id BIGINT NOT NULL,
    price_usd NUMERIC(30, 10),
    market_cap_usd NUMERIC(38, 2),
    volume_24h_usd NUMERIC(38, 2),
    price_change_24h_pct NUMERIC(18, 8),
    market_cap_change_24h_pct NUMERIC(18, 8),
    circulating_supply NUMERIC(38, 8),
    total_supply NUMERIC(38, 8),
    max_supply NUMERIC(38, 8),
    market_rank INTEGER,
    data_source VARCHAR(150),
    metric_timestamp TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    raw_payload JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT uq_market_metrics_latest_network UNIQUE (blockchain_network_id),

    CONSTRAINT fk_market_metrics_latest_network
        FOREIGN KEY (blockchain_network_id)
        REFERENCES blockchain_networks (id)
        ON DELETE CASCADE
);

-- ============================================================
-- 8. DeFi Metrics Latest
-- Purpose:
-- TVL, DEX volume, fees, revenue, and protocol count.
-- ============================================================

CREATE TABLE IF NOT EXISTS defi_metrics_latest (
    id BIGSERIAL PRIMARY KEY,
    blockchain_network_id BIGINT NOT NULL,
    tvl_usd NUMERIC(38, 2),
    tvl_change_24h_pct NUMERIC(18, 8),
    dex_volume_24h_usd NUMERIC(38, 2),
    fees_24h_usd NUMERIC(38, 2),
    revenue_24h_usd NUMERIC(38, 2),
    protocol_count INTEGER,
    data_source VARCHAR(150),
    metric_timestamp TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    raw_payload JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT uq_defi_metrics_latest_network UNIQUE (blockchain_network_id),

    CONSTRAINT fk_defi_metrics_latest_network
        FOREIGN KEY (blockchain_network_id)
        REFERENCES blockchain_networks (id)
        ON DELETE CASCADE
);

-- ============================================================
-- 9. Stablecoin Metrics Latest
-- Purpose:
-- Stablecoin supply and stablecoin activity.
-- ============================================================

CREATE TABLE IF NOT EXISTS stablecoin_metrics_latest (
    id BIGSERIAL PRIMARY KEY,
    blockchain_network_id BIGINT NOT NULL,
    stablecoin_supply_usd NUMERIC(38, 2),
    stablecoin_volume_24h_usd NUMERIC(38, 2),
    usdt_supply_usd NUMERIC(38, 2),
    usdc_supply_usd NUMERIC(38, 2),
    stablecoin_transactions_24h NUMERIC(30, 0),
    data_source VARCHAR(150),
    metric_timestamp TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    raw_payload JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT uq_stablecoin_metrics_latest_network UNIQUE (blockchain_network_id),

    CONSTRAINT fk_stablecoin_metrics_latest_network
        FOREIGN KEY (blockchain_network_id)
        REFERENCES blockchain_networks (id)
        ON DELETE CASCADE
);

-- ============================================================
-- 10. Network Metric History
-- Purpose:
-- Append-only historical time-series data for charts,
-- reports, and trend analysis.
-- ============================================================

CREATE TABLE IF NOT EXISTS network_metric_history (
    id BIGSERIAL PRIMARY KEY,
    blockchain_network_id BIGINT NOT NULL,
    metric_group VARCHAR(50) NOT NULL,
    metric_name VARCHAR(100) NOT NULL,
    metric_value NUMERIC(38, 10),
    metric_unit VARCHAR(50),
    provider_slug VARCHAR(150) NOT NULL DEFAULT 'unknown',
    metric_timestamp TIMESTAMPTZ NOT NULL,
    raw_payload JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT fk_network_metric_history_network
        FOREIGN KEY (blockchain_network_id)
        REFERENCES blockchain_networks (id)
        ON DELETE CASCADE,

    CONSTRAINT chk_network_metric_history_group
        CHECK (metric_group IN ('market', 'network', 'defi', 'stablecoin', 'fees', 'health')),

    CONSTRAINT uq_network_metric_history_unique_metric
        UNIQUE (
            blockchain_network_id,
            metric_group,
            metric_name,
            metric_timestamp,
            provider_slug
        )
);

-- ============================================================
-- 11. Alerts
-- Purpose:
-- Triggered alert events and alert state.
-- ============================================================

CREATE TABLE IF NOT EXISTS alerts (
    id BIGSERIAL PRIMARY KEY,
    blockchain_network_id BIGINT,
    alert_type VARCHAR(100) NOT NULL,
    severity VARCHAR(30) NOT NULL DEFAULT 'info',
    title VARCHAR(255) NOT NULL,
    message TEXT NOT NULL,
    metric_group VARCHAR(50),
    metric_name VARCHAR(100),
    threshold_value NUMERIC(38, 10),
    current_value NUMERIC(38, 10),
    comparison_operator VARCHAR(20),
    status VARCHAR(30) NOT NULL DEFAULT 'active',
    triggered_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    resolved_at TIMESTAMPTZ,
    created_by BIGINT,
    acknowledged_by BIGINT,
    acknowledged_at TIMESTAMPTZ,
    metadata JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT fk_alerts_network
        FOREIGN KEY (blockchain_network_id)
        REFERENCES blockchain_networks (id)
        ON DELETE SET NULL,

    CONSTRAINT fk_alerts_created_by
        FOREIGN KEY (created_by)
        REFERENCES users (id)
        ON DELETE SET NULL,

    CONSTRAINT fk_alerts_acknowledged_by
        FOREIGN KEY (acknowledged_by)
        REFERENCES users (id)
        ON DELETE SET NULL,

    CONSTRAINT chk_alerts_severity
        CHECK (severity IN ('info', 'warning', 'critical')),

    CONSTRAINT chk_alerts_status
        CHECK (status IN ('active', 'resolved', 'dismissed')),

    CONSTRAINT chk_alerts_comparison_operator
        CHECK (
            comparison_operator IS NULL
            OR comparison_operator IN ('>', '>=', '<', '<=', '=', '!=', 'change_pct')
        )
);

-- ============================================================
-- 12. Audit Logs
-- Purpose:
-- Track important user/system actions.
-- ============================================================

CREATE TABLE IF NOT EXISTS audit_logs (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT,
    action VARCHAR(150) NOT NULL,
    entity_type VARCHAR(150),
    entity_id BIGINT,
    old_values JSONB,
    new_values JSONB,
    ip_address INET,
    user_agent TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT fk_audit_logs_user
        FOREIGN KEY (user_id)
        REFERENCES users (id)
        ON DELETE SET NULL
);

-- ============================================================
-- 13. API Sync Logs
-- Purpose:
-- Track external API sync status, errors, duration,
-- and provider health.
-- ============================================================

CREATE TABLE IF NOT EXISTS api_sync_logs (
    id BIGSERIAL PRIMARY KEY,
    provider_slug VARCHAR(150) NOT NULL,
    blockchain_network_id BIGINT,
    sync_type VARCHAR(100) NOT NULL,
    status VARCHAR(30) NOT NULL,
    started_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    finished_at TIMESTAMPTZ,
    duration_ms INTEGER,
    records_received INTEGER NOT NULL DEFAULT 0,
    records_inserted INTEGER NOT NULL DEFAULT 0,
    http_status_code INTEGER,
    error_message TEXT,
    request_url TEXT,
    response_summary JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT fk_api_sync_logs_network
        FOREIGN KEY (blockchain_network_id)
        REFERENCES blockchain_networks (id)
        ON DELETE SET NULL,

    CONSTRAINT chk_api_sync_logs_status
        CHECK (status IN ('started', 'success', 'failed', 'skipped', 'partial'))
);

-- ============================================================
-- 14. Report Exports
-- Purpose:
-- Store generated report export records.
-- ============================================================

CREATE TABLE IF NOT EXISTS report_exports (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT,
    report_type VARCHAR(100) NOT NULL,
    report_name VARCHAR(255) NOT NULL,
    date_from DATE,
    date_to DATE,
    file_format VARCHAR(20) NOT NULL,
    file_path TEXT,
    status VARCHAR(30) NOT NULL DEFAULT 'queued',
    generated_at TIMESTAMPTZ,
    error_message TEXT,
    filters JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT fk_report_exports_user
        FOREIGN KEY (user_id)
        REFERENCES users (id)
        ON DELETE SET NULL,

    CONSTRAINT chk_report_exports_format
        CHECK (file_format IN ('csv', 'pdf', 'xlsx', 'json')),

    CONSTRAINT chk_report_exports_status
        CHECK (status IN ('queued', 'processing', 'completed', 'failed'))
);
```

---

## 9. Primary Keys

| Table | Primary Key |
|---|---|
| `roles` | `id` |
| `users` | `id` |
| `user_roles` | `user_id`, `role_id` |
| `blockchain_networks` | `id` |
| `api_provider_settings` | `id` |
| `network_metrics_latest` | `id` |
| `market_metrics_latest` | `id` |
| `defi_metrics_latest` | `id` |
| `stablecoin_metrics_latest` | `id` |
| `network_metric_history` | `id` |
| `alerts` | `id` |
| `audit_logs` | `id` |
| `api_sync_logs` | `id` |
| `report_exports` | `id` |

---

## 10. Foreign Keys

| Table | Foreign Key | References |
|---|---|---|
| `user_roles` | `user_id` | `users.id` |
| `user_roles` | `role_id` | `roles.id` |
| `user_roles` | `assigned_by` | `users.id` |
| `network_metrics_latest` | `blockchain_network_id` | `blockchain_networks.id` |
| `market_metrics_latest` | `blockchain_network_id` | `blockchain_networks.id` |
| `defi_metrics_latest` | `blockchain_network_id` | `blockchain_networks.id` |
| `stablecoin_metrics_latest` | `blockchain_network_id` | `blockchain_networks.id` |
| `network_metric_history` | `blockchain_network_id` | `blockchain_networks.id` |
| `alerts` | `blockchain_network_id` | `blockchain_networks.id` |
| `alerts` | `created_by` | `users.id` |
| `alerts` | `acknowledged_by` | `users.id` |
| `audit_logs` | `user_id` | `users.id` |
| `api_sync_logs` | `blockchain_network_id` | `blockchain_networks.id` |
| `report_exports` | `user_id` | `users.id` |

---

## 11. Unique Constraints

| Table | Unique Constraint |
|---|---|
| `roles` | `name` |
| `roles` | `slug` |
| `users` | `email` |
| `user_roles` | `user_id`, `role_id` |
| `blockchain_networks` | `name` |
| `blockchain_networks` | `slug` |
| `api_provider_settings` | `provider_slug` |
| `network_metrics_latest` | `blockchain_network_id` |
| `market_metrics_latest` | `blockchain_network_id` |
| `defi_metrics_latest` | `blockchain_network_id` |
| `stablecoin_metrics_latest` | `blockchain_network_id` |
| `network_metric_history` | `blockchain_network_id`, `metric_group`, `metric_name`, `metric_timestamp`, `provider_slug` |

---

## 12. Recommended Indexes

```sql
SET search_path TO blockchain_dashboard, public;

-- Users and roles
CREATE INDEX IF NOT EXISTS idx_users_status
ON users (status);

CREATE INDEX IF NOT EXISTS idx_user_roles_role_id
ON user_roles (role_id);

CREATE INDEX IF NOT EXISTS idx_user_roles_assigned_by
ON user_roles (assigned_by);

-- Blockchain networks
CREATE INDEX IF NOT EXISTS idx_blockchain_networks_active_sort
ON blockchain_networks (is_active, sort_order);

CREATE INDEX IF NOT EXISTS idx_blockchain_networks_chain_family
ON blockchain_networks (chain_family);

-- Latest metrics
CREATE INDEX IF NOT EXISTS idx_network_metrics_latest_timestamp
ON network_metrics_latest (metric_timestamp DESC);

CREATE INDEX IF NOT EXISTS idx_market_metrics_latest_timestamp
ON market_metrics_latest (metric_timestamp DESC);

CREATE INDEX IF NOT EXISTS idx_defi_metrics_latest_timestamp
ON defi_metrics_latest (metric_timestamp DESC);

CREATE INDEX IF NOT EXISTS idx_stablecoin_metrics_latest_timestamp
ON stablecoin_metrics_latest (metric_timestamp DESC);

-- Historical metrics
CREATE INDEX IF NOT EXISTS idx_metric_history_network_group_name_time
ON network_metric_history (
    blockchain_network_id,
    metric_group,
    metric_name,
    metric_timestamp DESC
);

CREATE INDEX IF NOT EXISTS idx_metric_history_group_name_time
ON network_metric_history (
    metric_group,
    metric_name,
    metric_timestamp DESC
);

CREATE INDEX IF NOT EXISTS idx_metric_history_provider_time
ON network_metric_history (
    provider_slug,
    metric_timestamp DESC
);

CREATE INDEX IF NOT EXISTS idx_metric_history_time_brin
ON network_metric_history
USING BRIN (metric_timestamp);

-- Alerts
CREATE INDEX IF NOT EXISTS idx_alerts_status_severity
ON alerts (status, severity);

CREATE INDEX IF NOT EXISTS idx_alerts_network_status
ON alerts (blockchain_network_id, status);

CREATE INDEX IF NOT EXISTS idx_alerts_triggered_at
ON alerts (triggered_at DESC);

-- Audit logs
CREATE INDEX IF NOT EXISTS idx_audit_logs_user_time
ON audit_logs (user_id, created_at DESC);

CREATE INDEX IF NOT EXISTS idx_audit_logs_entity
ON audit_logs (entity_type, entity_id);

-- API sync logs
CREATE INDEX IF NOT EXISTS idx_api_sync_logs_provider_status_time
ON api_sync_logs (provider_slug, status, started_at DESC);

CREATE INDEX IF NOT EXISTS idx_api_sync_logs_network_time
ON api_sync_logs (blockchain_network_id, started_at DESC);

CREATE INDEX IF NOT EXISTS idx_api_sync_logs_sync_type_time
ON api_sync_logs (sync_type, started_at DESC);

-- Report exports
CREATE INDEX IF NOT EXISTS idx_report_exports_user_time
ON report_exports (user_id, created_at DESC);

CREATE INDEX IF NOT EXISTS idx_report_exports_status
ON report_exports (status);
```

---

## 13. Latest Metrics Table Design

Latest metrics tables are used for fast dashboard loading.

They should contain one row per blockchain network.

Latest tables:

```text
network_metrics_latest
market_metrics_latest
defi_metrics_latest
stablecoin_metrics_latest
```

| Table | Dashboard Usage |
|---|---|
| `network_metrics_latest` | Latest block, transactions, active addresses, gas, fees, health |
| `market_metrics_latest` | Price, market cap, 24h volume |
| `defi_metrics_latest` | TVL, fees, revenue, DEX volume |
| `stablecoin_metrics_latest` | Stablecoin supply and stablecoin activity |

### 13.1 Latest Metrics UPSERT Example

```sql
INSERT INTO blockchain_dashboard.market_metrics_latest (
    blockchain_network_id,
    price_usd,
    market_cap_usd,
    volume_24h_usd,
    price_change_24h_pct,
    data_source,
    metric_timestamp,
    updated_at
)
VALUES (
    2,
    3500.25,
    420000000000,
    18000000000,
    2.45,
    'coingecko',
    NOW(),
    NOW()
)
ON CONFLICT (blockchain_network_id)
DO UPDATE SET
    price_usd = EXCLUDED.price_usd,
    market_cap_usd = EXCLUDED.market_cap_usd,
    volume_24h_usd = EXCLUDED.volume_24h_usd,
    price_change_24h_pct = EXCLUDED.price_change_24h_pct,
    data_source = EXCLUDED.data_source,
    metric_timestamp = EXCLUDED.metric_timestamp,
    updated_at = NOW();
```

---

## 14. Historical Metrics Table Design

Historical chart data is stored in:

```text
network_metric_history
```

This table is append-only and supports:

- Price history
- Market cap history
- Volume history
- TVL history
- Fee history
- Block height history
- Active address history
- Stablecoin supply history
- Network health history

Example history rows:

| Metric Group | Metric Name | Metric Unit |
|---|---|---|
| `market` | `price_usd` | `USD` |
| `market` | `market_cap_usd` | `USD` |
| `market` | `volume_24h_usd` | `USD` |
| `network` | `latest_block_number` | `block` |
| `network` | `transactions_24h` | `count` |
| `network` | `active_addresses_24h` | `count` |
| `fees` | `avg_gas_price_gwei` | `gwei` |
| `defi` | `tvl_usd` | `USD` |
| `stablecoin` | `stablecoin_supply_usd` | `USD` |

Example insert:

```sql
INSERT INTO blockchain_dashboard.network_metric_history (
    blockchain_network_id,
    metric_group,
    metric_name,
    metric_value,
    metric_unit,
    provider_slug,
    metric_timestamp
)
VALUES (
    2,
    'market',
    'price_usd',
    3500.25,
    'USD',
    'coingecko',
    NOW()
);
```

---

## 15. Alert Table Design

Alerts are stored in:

```text
alerts
```

The `alerts` table supports:

- Price movement alerts
- Gas fee alerts
- TVL change alerts
- Stablecoin supply alerts
- API provider failure alerts
- Network health alerts

Example alert:

```sql
INSERT INTO blockchain_dashboard.alerts (
    blockchain_network_id,
    alert_type,
    severity,
    title,
    message,
    metric_group,
    metric_name,
    threshold_value,
    current_value,
    comparison_operator,
    status
)
VALUES (
    1,
    'price_movement',
    'warning',
    'Bitcoin price movement alert',
    'Bitcoin price changed more than configured threshold.',
    'market',
    'price_change_24h_pct',
    5,
    6.2,
    'change_pct',
    'active'
);
```

---

## 16. User and Role Table Design

Users and roles are managed through three tables:

```text
users
roles
user_roles
```

Default roles:

| Role | Purpose |
|---|---|
| Admin | Full system control |
| Analyst | View dashboards, create alerts, export reports |
| Viewer | Read-only dashboard access |

The `user_roles` table supports many-to-many relationships.

This allows one user to have more than one role if needed.

---

## 17. API Sync Logging Table Design

API sync logs are stored in:

```text
api_sync_logs
```

This table tracks:

- Provider name
- Network
- Sync type
- Sync status
- Start time
- End time
- Duration
- Records received
- Records inserted
- HTTP status code
- Error message

Example sync log:

```sql
INSERT INTO blockchain_dashboard.api_sync_logs (
    provider_slug,
    blockchain_network_id,
    sync_type,
    status,
    started_at,
    finished_at,
    duration_ms,
    records_received,
    records_inserted,
    http_status_code
)
VALUES (
    'coingecko',
    1,
    'market_metrics',
    'success',
    NOW() - INTERVAL '5 seconds',
    NOW(),
    5000,
    1,
    1,
    200
);
```

---

## 18. Seed Data for the First 10 Networks

```sql
SET search_path TO blockchain_dashboard, public;

INSERT INTO blockchain_networks (
    name,
    slug,
    symbol,
    native_asset,
    category,
    chain_family,
    chain_id,
    is_evm,
    explorer_url,
    sort_order,
    is_active
)
VALUES
(
    'Bitcoin',
    'bitcoin',
    'BTC',
    'BTC',
    'Layer 1',
    'bitcoin',
    NULL,
    FALSE,
    'https://mempool.space',
    1,
    TRUE
),
(
    'Ethereum',
    'ethereum',
    'ETH',
    'ETH',
    'Layer 1 / Smart Contract',
    'evm',
    1,
    TRUE,
    'https://etherscan.io',
    2,
    TRUE
),
(
    'BNB Chain',
    'bnb-chain',
    'BNB',
    'BNB',
    'Layer 1 / EVM',
    'evm',
    56,
    TRUE,
    'https://bscscan.com',
    3,
    TRUE
),
(
    'Solana',
    'solana',
    'SOL',
    'SOL',
    'Layer 1',
    'solana',
    NULL,
    FALSE,
    'https://solscan.io',
    4,
    TRUE
),
(
    'TRON',
    'tron',
    'TRX',
    'TRX',
    'Layer 1',
    'tron',
    NULL,
    FALSE,
    'https://tronscan.org',
    5,
    TRUE
),
(
    'Polygon',
    'polygon',
    'POL',
    'POL',
    'Ethereum Scaling',
    'evm',
    137,
    TRUE,
    'https://polygonscan.com',
    6,
    TRUE
),
(
    'Base',
    'base',
    'ETH',
    'ETH',
    'Ethereum Layer 2',
    'evm',
    8453,
    TRUE,
    'https://basescan.org',
    7,
    TRUE
),
(
    'Arbitrum',
    'arbitrum',
    'ARB',
    'ETH',
    'Ethereum Layer 2',
    'evm',
    42161,
    TRUE,
    'https://arbiscan.io',
    8,
    TRUE
),
(
    'Optimism',
    'optimism',
    'OP',
    'ETH',
    'Ethereum Layer 2',
    'evm',
    10,
    TRUE,
    'https://optimistic.etherscan.io',
    9,
    TRUE
),
(
    'Avalanche',
    'avalanche',
    'AVAX',
    'AVAX',
    'Layer 1 / EVM',
    'evm',
    43114,
    TRUE,
    'https://snowtrace.io',
    10,
    TRUE
)
ON CONFLICT (slug) DO NOTHING;
```

---

## 19. Seed Data for Default Roles

```sql
SET search_path TO blockchain_dashboard, public;

INSERT INTO roles (
    name,
    slug,
    description,
    is_system
)
VALUES
(
    'Admin',
    'admin',
    'Full access to users, roles, networks, API settings, alerts, reports, and system logs.',
    TRUE
),
(
    'Analyst',
    'analyst',
    'Can view dashboards, analyze network metrics, create alerts, run manual syncs, and export reports.',
    TRUE
),
(
    'Viewer',
    'viewer',
    'Read-only access to dashboard, blockchain networks, KPIs, charts, and alerts.',
    TRUE
)
ON CONFLICT (slug) DO NOTHING;
```

---

## 20. Seed Data for API Provider Settings

```sql
SET search_path TO blockchain_dashboard, public;

INSERT INTO api_provider_settings (
    provider_name,
    provider_slug,
    base_url,
    api_key_env_name,
    rate_limit_per_minute,
    timeout_seconds,
    retry_count,
    priority,
    is_enabled
)
VALUES
(
    'CoinGecko',
    'coingecko',
    'https://api.coingecko.com/api/v3',
    'COINGECKO_API_KEY',
    50,
    30,
    3,
    1,
    TRUE
),
(
    'DefiLlama',
    'defillama',
    'https://api.llama.fi',
    NULL,
    60,
    30,
    3,
    1,
    TRUE
),
(
    'Etherscan',
    'etherscan',
    'https://api.etherscan.io/v2/api',
    'ETHERSCAN_API_KEY',
    5,
    30,
    3,
    1,
    TRUE
),
(
    'TronGrid',
    'trongrid',
    'https://api.trongrid.io',
    'TRONGRID_API_KEY',
    20,
    30,
    3,
    1,
    TRUE
),
(
    'Helius',
    'helius',
    'https://api.helius.xyz',
    'HELIUS_API_KEY',
    20,
    30,
    3,
    1,
    TRUE
),
(
    'Mempool Space',
    'mempool-space',
    'https://mempool.space/api',
    NULL,
    60,
    30,
    3,
    1,
    TRUE
)
ON CONFLICT (provider_slug) DO NOTHING;
```

---

## 21. Sample SELECT Validation Queries

### 21.1 Check Schema Tables

```sql
SELECT table_name
FROM information_schema.tables
WHERE table_schema = 'blockchain_dashboard'
ORDER BY table_name;
```

Expected tables:

```text
alerts
api_provider_settings
api_sync_logs
audit_logs
blockchain_networks
defi_metrics_latest
market_metrics_latest
network_metric_history
network_metrics_latest
report_exports
roles
stablecoin_metrics_latest
user_roles
users
```

### 21.2 Count Initial Networks

```sql
SELECT COUNT(*) AS total_networks
FROM blockchain_dashboard.blockchain_networks;
```

Expected result:

```text
10
```

### 21.3 Count Default Roles

```sql
SELECT COUNT(*) AS total_roles
FROM blockchain_dashboard.roles;
```

Expected result:

```text
3
```

### 21.4 Count API Providers

```sql
SELECT COUNT(*) AS total_api_providers
FROM blockchain_dashboard.api_provider_settings;
```

Expected result:

```text
6
```

### 21.5 View Active Networks

```sql
SELECT
    id,
    name,
    slug,
    symbol,
    native_asset,
    category,
    chain_family,
    chain_id,
    is_evm,
    sort_order
FROM blockchain_dashboard.blockchain_networks
WHERE is_active = TRUE
ORDER BY sort_order;
```

### 21.6 Check Missing Latest Metrics

```sql
SELECT
    n.name,
    n.slug,
    CASE WHEN nm.id IS NULL THEN 'missing' ELSE 'available' END AS network_metrics,
    CASE WHEN mm.id IS NULL THEN 'missing' ELSE 'available' END AS market_metrics,
    CASE WHEN dm.id IS NULL THEN 'missing' ELSE 'available' END AS defi_metrics,
    CASE WHEN sm.id IS NULL THEN 'missing' ELSE 'available' END AS stablecoin_metrics
FROM blockchain_dashboard.blockchain_networks n
LEFT JOIN blockchain_dashboard.network_metrics_latest nm
    ON nm.blockchain_network_id = n.id
LEFT JOIN blockchain_dashboard.market_metrics_latest mm
    ON mm.blockchain_network_id = n.id
LEFT JOIN blockchain_dashboard.defi_metrics_latest dm
    ON dm.blockchain_network_id = n.id
LEFT JOIN blockchain_dashboard.stablecoin_metrics_latest sm
    ON sm.blockchain_network_id = n.id
ORDER BY n.sort_order;
```

---

## 22. Sample Dashboard Queries

### 22.1 Main Dashboard Network Comparison Query

```sql
SELECT
    n.id,
    n.name,
    n.slug,
    n.symbol,
    n.native_asset,
    n.category,
    mm.price_usd,
    mm.market_cap_usd,
    mm.volume_24h_usd,
    mm.price_change_24h_pct,
    nm.latest_block_number,
    nm.transactions_24h,
    nm.active_addresses_24h,
    nm.avg_transaction_fee_usd,
    nm.avg_gas_price_gwei,
    dm.tvl_usd,
    dm.dex_volume_24h_usd,
    sm.stablecoin_supply_usd,
    nm.health_status,
    GREATEST(
        COALESCE(mm.metric_timestamp, '1970-01-01'::timestamptz),
        COALESCE(nm.metric_timestamp, '1970-01-01'::timestamptz),
        COALESCE(dm.metric_timestamp, '1970-01-01'::timestamptz),
        COALESCE(sm.metric_timestamp, '1970-01-01'::timestamptz)
    ) AS latest_metric_timestamp
FROM blockchain_dashboard.blockchain_networks n
LEFT JOIN blockchain_dashboard.market_metrics_latest mm
    ON mm.blockchain_network_id = n.id
LEFT JOIN blockchain_dashboard.network_metrics_latest nm
    ON nm.blockchain_network_id = n.id
LEFT JOIN blockchain_dashboard.defi_metrics_latest dm
    ON dm.blockchain_network_id = n.id
LEFT JOIN blockchain_dashboard.stablecoin_metrics_latest sm
    ON sm.blockchain_network_id = n.id
WHERE n.is_active = TRUE
ORDER BY n.sort_order;
```

### 22.2 Dashboard Summary Cards Query

```sql
SELECT
    COUNT(DISTINCT n.id) AS total_networks,
    SUM(mm.market_cap_usd) AS total_market_cap_usd,
    SUM(mm.volume_24h_usd) AS total_volume_24h_usd,
    SUM(dm.tvl_usd) AS total_tvl_usd,
    SUM(sm.stablecoin_supply_usd) AS total_stablecoin_supply_usd,
    COUNT(a.id) FILTER (WHERE a.status = 'active') AS active_alerts,
    COUNT(a.id) FILTER (WHERE a.status = 'active' AND a.severity = 'critical') AS critical_alerts
FROM blockchain_dashboard.blockchain_networks n
LEFT JOIN blockchain_dashboard.market_metrics_latest mm
    ON mm.blockchain_network_id = n.id
LEFT JOIN blockchain_dashboard.defi_metrics_latest dm
    ON dm.blockchain_network_id = n.id
LEFT JOIN blockchain_dashboard.stablecoin_metrics_latest sm
    ON sm.blockchain_network_id = n.id
LEFT JOIN blockchain_dashboard.alerts a
    ON a.blockchain_network_id = n.id
WHERE n.is_active = TRUE;
```

### 22.3 Network Detail Page Query

```sql
SELECT
    n.id,
    n.name,
    n.slug,
    n.symbol,
    n.native_asset,
    n.category,
    n.chain_family,
    n.chain_id,
    n.explorer_url,

    mm.price_usd,
    mm.market_cap_usd,
    mm.volume_24h_usd,
    mm.price_change_24h_pct,

    nm.latest_block_number,
    nm.latest_block_hash,
    nm.block_time_seconds,
    nm.transactions_24h,
    nm.active_addresses_24h,
    nm.avg_transaction_fee_usd,
    nm.avg_gas_price_gwei,
    nm.health_status,
    nm.health_message,

    dm.tvl_usd,
    dm.tvl_change_24h_pct,
    dm.dex_volume_24h_usd,
    dm.fees_24h_usd,
    dm.revenue_24h_usd,
    dm.protocol_count,

    sm.stablecoin_supply_usd,
    sm.stablecoin_volume_24h_usd,
    sm.usdt_supply_usd,
    sm.usdc_supply_usd,
    sm.stablecoin_transactions_24h

FROM blockchain_dashboard.blockchain_networks n
LEFT JOIN blockchain_dashboard.market_metrics_latest mm
    ON mm.blockchain_network_id = n.id
LEFT JOIN blockchain_dashboard.network_metrics_latest nm
    ON nm.blockchain_network_id = n.id
LEFT JOIN blockchain_dashboard.defi_metrics_latest dm
    ON dm.blockchain_network_id = n.id
LEFT JOIN blockchain_dashboard.stablecoin_metrics_latest sm
    ON sm.blockchain_network_id = n.id
WHERE n.slug = 'ethereum';
```

### 22.4 Historical Chart Query

Example: Ethereum price chart for the last 30 days.

```sql
SELECT
    h.metric_timestamp,
    h.metric_value AS price_usd
FROM blockchain_dashboard.network_metric_history h
JOIN blockchain_dashboard.blockchain_networks n
    ON n.id = h.blockchain_network_id
WHERE n.slug = 'ethereum'
  AND h.metric_group = 'market'
  AND h.metric_name = 'price_usd'
  AND h.metric_timestamp >= NOW() - INTERVAL '30 days'
ORDER BY h.metric_timestamp ASC;
```

### 22.5 Active Alerts Query

```sql
SELECT
    a.id,
    n.name AS network_name,
    n.slug AS network_slug,
    a.alert_type,
    a.severity,
    a.title,
    a.message,
    a.metric_group,
    a.metric_name,
    a.threshold_value,
    a.current_value,
    a.status,
    a.triggered_at
FROM blockchain_dashboard.alerts a
LEFT JOIN blockchain_dashboard.blockchain_networks n
    ON n.id = a.blockchain_network_id
WHERE a.status = 'active'
ORDER BY
    CASE a.severity
        WHEN 'critical' THEN 1
        WHEN 'warning' THEN 2
        ELSE 3
    END,
    a.triggered_at DESC;
```

### 22.6 API Provider Health Query

```sql
SELECT DISTINCT ON (provider_slug, blockchain_network_id, sync_type)
    provider_slug,
    blockchain_network_id,
    sync_type,
    status,
    started_at,
    finished_at,
    duration_ms,
    http_status_code,
    error_message
FROM blockchain_dashboard.api_sync_logs
ORDER BY
    provider_slug,
    blockchain_network_id,
    sync_type,
    started_at DESC;
```

### 22.7 Report Export History Query

```sql
SELECT
    r.id,
    u.name AS user_name,
    r.report_type,
    r.report_name,
    r.date_from,
    r.date_to,
    r.file_format,
    r.status,
    r.generated_at,
    r.created_at
FROM blockchain_dashboard.report_exports r
LEFT JOIN blockchain_dashboard.users u
    ON u.id = r.user_id
ORDER BY r.created_at DESC;
```

---

## 23. Performance Recommendations

### 23.1 Latest Metrics

Use latest metrics tables for dashboard cards.

Correct:

```text
Dashboard cards -> latest metrics tables
```

Avoid:

```text
Dashboard cards -> heavy aggregation from network_metric_history
```

### 23.2 Historical Metrics

Use `network_metric_history` for charts and reports.

Recommended retention strategy:

| Data Age | Strategy |
|---|---|
| Last 30 days | Keep high-frequency data |
| 30–180 days | Keep hourly snapshots |
| 180+ days | Keep daily snapshots or archive |

### 23.3 Indexing

Important indexes:

```text
network_metric_history(blockchain_network_id, metric_group, metric_name, metric_timestamp DESC)
alerts(status, severity)
api_sync_logs(provider_slug, status, started_at DESC)
blockchain_networks(is_active, sort_order)
```

### 23.4 Partitioning

When `network_metric_history` becomes large, partition by month.

Future partition example:

```text
network_metric_history_2026_06
network_metric_history_2026_07
network_metric_history_2026_08
```

### 23.5 JSONB Usage

Use `JSONB` for raw provider payloads only.

Important KPIs should have normal typed columns.

Do not store important KPIs only inside JSONB.

### 23.6 API Sync Logs Cleanup

Recommended retention:

| Log Type | Retention |
|---|---:|
| Successful sync logs | 30–90 days |
| Failed sync logs | 180 days |
| Critical errors | 1 year |

### 23.7 Dashboard Query Optimization

For high-traffic dashboards, later create a materialized view:

```text
dashboard_network_overview_mv
```

This can pre-join latest metrics from all latest tables.

---

## 24. Validation Commands

### 24.1 Create Database

```bash
createdb -U postgres blocknet_dashboard
```

Alternative:

```bash
psql -U postgres -c "CREATE DATABASE blocknet_dashboard WITH ENCODING = 'UTF8' TEMPLATE = template0;"
```

### 24.2 Run Schema SQL File

```bash
psql -U postgres -d blocknet_dashboard -f database/init/01_create_blocknet_dashboard_schema.sql
```

### 24.3 Connect to Database

```bash
psql -U postgres -d blocknet_dashboard
```

### 24.4 Validate Tables

```sql
SELECT table_name
FROM information_schema.tables
WHERE table_schema = 'blockchain_dashboard'
ORDER BY table_name;
```

### 24.5 Validate Seed Data

```sql
SELECT COUNT(*) FROM blockchain_dashboard.blockchain_networks;
SELECT COUNT(*) FROM blockchain_dashboard.roles;
SELECT COUNT(*) FROM blockchain_dashboard.api_provider_settings;
```

Expected output:

```text
10 networks
3 roles
6 API providers
```

---

## 25. GitHub Commit Commands

Save this document as:

```text
docs/phase-03-postgresql-database-design.md
```

Then run:

```bash
git add docs/phase-03-postgresql-database-design.md
git commit -m "docs: add phase 3 postgresql database design"
git push
```

If you still have unwanted Windows metadata files, remove them:

```bash
rm -f "docs/phase-01-business-project-foundation.md:Zone.Identifier"
rm -f "docs/phase-02-system-architecture-design.md:Zone.Identifier"
rm -f "docs/phase-03-postgresql-database-design.md:Zone.Identifier"
```

Recommended `.gitignore` rule:

```bash
echo "*:Zone.Identifier" >> .gitignore
git add .gitignore
git commit -m "chore: ignore Windows zone identifier files"
git push
```

---

## 26. Final Phase 3 Checklist

```text
[x] Database name defined: blocknet_dashboard
[x] Schema name defined: blockchain_dashboard
[x] Database design overview completed
[x] Entity relationship explanation completed
[x] CREATE DATABASE command provided
[x] CREATE SCHEMA command provided
[x] Full CREATE TABLE scripts provided
[x] Primary keys defined
[x] Foreign keys defined
[x] Unique constraints defined
[x] Recommended indexes provided
[x] Latest metrics table design completed
[x] Historical metrics table design completed
[x] Alert table design completed
[x] User and role table design completed
[x] API sync logging table design completed
[x] Seed data for 10 blockchain networks provided
[x] Seed data for default roles provided
[x] Seed data for API provider settings provided
[x] Sample SELECT validation queries provided
[x] Sample dashboard queries provided
[x] Performance recommendations provided
[x] Database supports latest dashboard cards
[x] Database supports historical charts
[x] Database supports network comparison table
[x] Database supports network detail page
[x] Database supports alerts
[x] Database supports reports
[x] Database supports API sync logs
[x] Phase 3 ready for review
```

---

## 27. Expected Output

```text
BlockNet Analytics Dashboard now has a complete PostgreSQL database design for networks, latest metrics, historical charts, alerts, users, roles, audit logs, API sync logs, report exports, and API provider settings.
```

---

## 28. Next Recommended Step

The next phase is:

```text
Phase 4 — Laravel Backend Project Setup
```

Phase 4 should include:

- Laravel installation
- `.env` database configuration
- PostgreSQL connection
- Laravel Sanctum setup
- Initial migrations
- Models
- Seeders
- API route structure
- Health check endpoint

---

## Phase Status

```text
Phase 3 completed.
Waiting for approval before starting Phase 4.
```
