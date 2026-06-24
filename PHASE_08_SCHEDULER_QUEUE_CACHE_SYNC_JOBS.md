# BlockNet Analytics Dashboard

**Project:** Blockchain Networks Analytics Dashboard  
**Database:** `blocknet_dashboard`  
**Schema:** `blockchain_dashboard`  
**Stack:** PostgreSQL, Laravel, Vue, Docker, Nginx, Ubuntu VPS  
**Initial Networks:** Bitcoin, Ethereum, BNB Chain, Solana, TRON, Polygon, Base, Arbitrum

---

# Phase 8 — Scheduler, Queue, Cache, and Sync Jobs

## 1. Phase Objective

Automate external API synchronization using Laravel scheduler, queue jobs, cache, retry logic, and background workers.

This phase converts the manual sync services from Phase 7 into production-ready scheduled jobs.

---

## 2. Required Jobs

| Job | Purpose | Suggested Frequency |
|-----|---------|--------------------|
| `SyncMarketMetricsJob` | Price, market cap, volume | Every 5 to 15 minutes |
| `SyncNetworkMetricsJob` | Network activity metrics | Every 15 minutes |
| `SyncDefiMetricsJob` | TVL and DeFi metrics | Every 30 to 60 minutes |
| `SyncStablecoinMetricsJob` | Stablecoin supply and volume | Every 30 to 60 minutes |
| `EvaluateAlertsJob` | Check thresholds and create alerts | Every 5 minutes |
| `CleanOldSyncLogsJob` | Retention cleanup | Daily |

---

## 3. Queue Configuration

Update `.env`:

```env
QUEUE_CONNECTION=database
CACHE_STORE=database
```

Create queue tables:

```bash
php artisan queue:table
php artisan queue:failed-table
php artisan migrate
```

Run worker locally:

```bash
php artisan queue:work --tries=3 --timeout=120
```

---

## 4. Create Jobs

```bash
php artisan make:job SyncMarketMetricsJob
php artisan make:job SyncNetworkMetricsJob
php artisan make:job SyncDefiMetricsJob
php artisan make:job SyncStablecoinMetricsJob
php artisan make:job EvaluateAlertsJob
php artisan make:job CleanOldSyncLogsJob
```

---

## 5. Example Market Metrics Job

```php
<?php

namespace App\Jobs;

use App\Services\MarketData\MarketMetricsService;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class SyncMarketMetricsJob implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public int $tries = 3;
    public int $timeout = 120;

    public function handle(MarketMetricsService $service): void
    {
        $service->syncAll();
    }
}
```

---

## 6. Scheduler Configuration

For Laravel 11+, update `routes/console.php`:

```php
use Illuminate\Support\Facades\Schedule;
use App\Jobs\SyncMarketMetricsJob;
use App\Jobs\SyncNetworkMetricsJob;
use App\Jobs\SyncDefiMetricsJob;
use App\Jobs\SyncStablecoinMetricsJob;
use App\Jobs\EvaluateAlertsJob;
use App\Jobs\CleanOldSyncLogsJob;

Schedule::job(new SyncMarketMetricsJob())->everyTenMinutes()->withoutOverlapping();
Schedule::job(new SyncNetworkMetricsJob())->everyFifteenMinutes()->withoutOverlapping();
Schedule::job(new SyncDefiMetricsJob())->hourly()->withoutOverlapping();
Schedule::job(new SyncStablecoinMetricsJob())->hourly()->withoutOverlapping();
Schedule::job(new EvaluateAlertsJob())->everyFiveMinutes()->withoutOverlapping();
Schedule::job(new CleanOldSyncLogsJob())->dailyAt('02:00')->withoutOverlapping();
```

For older Laravel versions, configure in `app/Console/Kernel.php`.

---

## 7. Server Cron Entry

On Ubuntu VPS:

```bash
crontab -e
```

Add:

```cron
* * * * * cd /var/www/blocknet/backend && php artisan schedule:run >> /dev/null 2>&1
```

---

## 8. Cache Strategy

Use cache for dashboard APIs and provider responses.

Recommended cache keys:

```text
dashboard:overview
networks:active
network:{network_key}:latest
network:{network_key}:history:{metric_key}:{range}
alerts:open
```

Example:

```php
Cache::remember('dashboard:overview', now()->addMinutes(5), function () {
    return $this->buildOverviewData();
});
```

---

## 9. Cache Invalidation Rules

Invalidate cache after successful sync:

```php
Cache::forget('dashboard:overview');
Cache::forget('networks:active');
Cache::tags(['dashboard'])->flush();
```

If cache tags are not available for the chosen driver, manually forget known keys.

---

## 10. Alert Evaluation Rules

`EvaluateAlertsJob` should check:

- Price drop percentage
- Price increase percentage
- TVL drop
- Stablecoin supply drop
- Transactions spike
- API sync failure
- Network metric missing or stale

Example alert condition:

```text
If market price changes more than 10% in 24 hours, create warning alert.
```

Alert duplicate prevention:

```text
Do not create the same alert for the same network, alert_type, and open status within the configured cooldown window.
```

---

## 11. Queue Worker in Production

Recommended Supervisor config:

```ini
[program:blocknet-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/blocknet/backend/artisan queue:work --sleep=3 --tries=3 --timeout=120
autostart=true
autorestart=true
user=www-data
numprocs=2
redirect_stderr=true
stdout_logfile=/var/www/blocknet/backend/storage/logs/worker.log
```

Apply:

```bash
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start blocknet-worker:*
```

---

## 12. Monitoring Commands

```bash
php artisan schedule:list
php artisan queue:work --once
php artisan queue:failed
php artisan queue:retry all
```

---

## 13. Phase Deliverables

- Queue tables created
- Sync jobs created
- Alert evaluation job created
- Cleanup job created
- Scheduler configured
- Cache strategy implemented
- Production worker config prepared

---

## 14. GitHub Commit

```bash
git add backend/app/Jobs backend/routes/console.php docs/PHASE_08_SCHEDULER_QUEUE_CACHE_SYNC_JOBS.md
git commit -m "Phase 8: add scheduler queue cache and sync jobs"
git push
```
