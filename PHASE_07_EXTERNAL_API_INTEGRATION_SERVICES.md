# BlockNet Analytics Dashboard

**Project:** Blockchain Networks Analytics Dashboard  
**Database:** `blocknet_dashboard`  
**Schema:** `blockchain_dashboard`  
**Stack:** PostgreSQL, Laravel, Vue, Docker, Nginx, Ubuntu VPS  
**Initial Networks:** Bitcoin, Ethereum, BNB Chain, Solana, TRON, Polygon, Base, Arbitrum

---

# Phase 7 — External API Integration Services

## 1. Phase Objective

Build backend service classes that fetch blockchain, market, DeFi, and stablecoin metrics from external providers and store them in PostgreSQL.

This phase prepares the integration layer. The scheduler and queue automation will be added in Phase 8.

---

## 2. Integration Categories

| Category | Example Data |
|----------|--------------|
| Network metrics | block height, transactions, active addresses, fees, validators |
| Market metrics | price, market cap, 24h volume, supply, rank |
| DeFi metrics | TVL, protocol count, DEX volume |
| Stablecoin metrics | USDT, USDC, DAI supply and transfer volume |

---

## 3. Service Folder Structure

```text
backend/app/Services/
├── Blockchain/
│   ├── BlockchainMetricsService.php
│   ├── NetworkMetricsMapper.php
│   └── Providers/
│       ├── EtherscanProvider.php
│       ├── TronscanProvider.php
│       ├── SolanaProvider.php
│       └── GenericRpcProvider.php
├── MarketData/
│   ├── MarketMetricsService.php
│   └── Providers/
│       └── CoinGeckoProvider.php
├── Defi/
│   ├── DefiMetricsService.php
│   └── Providers/
│       └── DefiLlamaProvider.php
└── Stablecoins/
    ├── StablecoinMetricsService.php
    └── Providers/
        └── StablecoinProvider.php
```

---

## 4. Environment Variables

Update `backend/.env.example`:

```env
COINGECKO_BASE_URL=https://api.coingecko.com/api/v3
COINGECKO_API_KEY=

DEFILLAMA_BASE_URL=https://api.llama.fi

ETHERSCAN_BASE_URL=https://api.etherscan.io/api
ETHERSCAN_API_KEY=

BSCSCAN_BASE_URL=https://api.bscscan.com/api
BSCSCAN_API_KEY=

POLYGONSCAN_BASE_URL=https://api.polygonscan.com/api
POLYGONSCAN_API_KEY=

BASESCAN_BASE_URL=https://api.basescan.org/api
BASESCAN_API_KEY=

ARBISCAN_BASE_URL=https://api.arbiscan.io/api
ARBISCAN_API_KEY=

TRONSCAN_BASE_URL=https://apilist.tronscanapi.com/api
SOLANA_RPC_URL=https://api.mainnet-beta.solana.com
```

---

## 5. Provider Settings Table Usage

External providers should also be stored in `api_provider_settings`.

Example providers:

| Provider Key | Provider Name | Usage |
|--------------|---------------|-------|
| `coingecko` | CoinGecko | Prices and market metrics |
| `defillama` | DefiLlama | DeFi TVL and protocols |
| `etherscan` | Etherscan | Ethereum network metrics |
| `tronscan` | TRONSCAN | TRON metrics |
| `solana-rpc` | Solana RPC | Solana network data |

---

## 6. API Client Base Class

Create:

```text
app/Services/ExternalApiClient.php
```

```php
<?php

namespace App\Services;

use Illuminate\Support\Facades\Http;
use RuntimeException;

class ExternalApiClient
{
    public function get(string $url, array $query = [], array $headers = []): array
    {
        $response = Http::timeout(30)
            ->retry(3, 500)
            ->withHeaders($headers)
            ->get($url, $query);

        if (!$response->successful()) {
            throw new RuntimeException('External API request failed: ' . $response->body());
        }

        return $response->json() ?? [];
    }
}
```

---

## 7. Market Metrics Service

Create:

```text
app/Services/MarketData/MarketMetricsService.php
```

Responsibilities:

- Fetch price data by coin ID
- Normalize result
- Update `market_metrics_latest`
- Insert history rows into `network_metric_history`
- Create `api_sync_logs` entry

Pseudo-implementation:

```php
class MarketMetricsService
{
    public function syncAll(): void
    {
        $networks = BlockchainNetwork::where('is_active', true)->get();

        foreach ($networks as $network) {
            $this->syncNetwork($network);
        }
    }

    public function syncNetwork(BlockchainNetwork $network): void
    {
        // 1. Resolve provider coin ID
        // 2. Fetch provider response
        // 3. Map fields
        // 4. Upsert latest table
        // 5. Insert history rows
        // 6. Write api_sync_logs
    }
}
```

---

## 8. Network Key to Provider ID Mapping

Create config file:

```text
config/blockchain_providers.php
```

```php
return [
    'market_ids' => [
        'bitcoin' => 'bitcoin',
        'ethereum' => 'ethereum',
        'bnb-chain' => 'binancecoin',
        'solana' => 'solana',
        'tron' => 'tron',
        'polygon' => 'matic-network',
        'base' => 'ethereum',
        'arbitrum' => 'ethereum',
    ],
];
```

---

## 9. API Sync Logging

Every sync attempt must write a row to `api_sync_logs`.

Statuses:

| Status | Meaning |
|--------|---------|
| `running` | Sync started |
| `success` | Sync completed |
| `failed` | Sync failed |
| `partial` | Some networks failed |

Required fields:

- `provider_name`
- `sync_type`
- `network_id`
- `status`
- `started_at`
- `finished_at`
- `records_processed`
- `error_message`
- `request_meta`
- `response_meta`

---

## 10. Upsert Strategy

Latest metrics tables must use upsert logic.

Example:

```php
MarketMetricsLatest::updateOrCreate(
    ['network_id' => $network->id],
    [
        'price_usd' => $data['price_usd'],
        'market_cap_usd' => $data['market_cap_usd'],
        'volume_24h_usd' => $data['volume_24h_usd'],
        'recorded_at' => now(),
    ]
);
```

History rows should be append-only:

```php
NetworkMetricHistory::create([
    'network_id' => $network->id,
    'metric_group' => 'market',
    'metric_key' => 'price_usd',
    'metric_value' => $data['price_usd'],
    'source_provider' => 'coingecko',
    'recorded_at' => now(),
]);
```

---

## 11. Error Handling Rules

- One failed network must not stop the full sync.
- Log failed API calls.
- Store provider error response in `api_sync_logs.error_message`.
- Use retries for temporary HTTP failures.
- Use timeout to avoid long-running locked jobs.
- Never expose API keys in logs.

---

## 12. Manual Sync Command

Create command:

```bash
php artisan make:command SyncBlockchainMetricsCommand
```

Command signature:

```php
protected $signature = 'blocknet:sync-metrics {--network=} {--type=all}';
```

Example usage:

```bash
php artisan blocknet:sync-metrics --type=market
php artisan blocknet:sync-metrics --network=bitcoin --type=all
```

---

## 13. Phase Deliverables

- External API client created
- Provider config created
- Market metrics service created
- Network metrics service structure created
- DeFi metrics service structure created
- Stablecoin metrics service structure created
- API sync logging implemented
- Manual sync command created

---

## 14. GitHub Commit

```bash
git add backend/app/Services backend/app/Console backend/config docs/PHASE_07_EXTERNAL_API_INTEGRATION_SERVICES.md
git commit -m "Phase 7: add external API integration services"
git push
```
