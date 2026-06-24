# BlockNet Analytics Dashboard

**Project:** Blockchain Networks Analytics Dashboard  
**Database:** `blocknet_dashboard`  
**Schema:** `blockchain_dashboard`  
**Stack:** PostgreSQL, Laravel, Vue, Docker, Nginx, Ubuntu VPS  
**Initial Networks:** Bitcoin, Ethereum, BNB Chain, Solana, TRON, Polygon, Base, Arbitrum

---

# Phase 5 — Laravel Migrations, Models, and Seeders

## 1. Phase Objective

Create Laravel migrations, Eloquent models, model relationships, factories, and seeders for all database entities defined in Phase 3.

This phase connects Laravel to the database design and makes the structure manageable through Laravel commands.

---

## 2. Required Laravel Models

Create the following models:

```bash
php artisan make:model BlockchainNetwork -mfs
php artisan make:model NetworkMetricsLatest -mfs
php artisan make:model MarketMetricsLatest -mfs
php artisan make:model DefiMetricsLatest -mfs
php artisan make:model StablecoinMetricsLatest -mfs
php artisan make:model NetworkMetricHistory -mfs
php artisan make:model Alert -mfs
php artisan make:model Role -mfs
php artisan make:model UserRole -mfs
php artisan make:model AuditLog -mfs
php artisan make:model ApiSyncLog -mfs
php artisan make:model ReportExport -mfs
php artisan make:model ApiProviderSetting -mfs
```

The Laravel default `User` model should be reused and customized.

---

## 3. Model Table Configuration

Because the project uses the PostgreSQL schema `blockchain_dashboard`, each model should define the schema-qualified table name.

Example for `app/Models/BlockchainNetwork.php`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class BlockchainNetwork extends Model
{
    use HasFactory;

    protected $table = 'blockchain_dashboard.blockchain_networks';

    protected $fillable = [
        'network_key',
        'name',
        'native_symbol',
        'network_type',
        'chain_id',
        'explorer_url',
        'official_website',
        'logo_url',
        'is_active',
        'display_order',
    ];

    protected $casts = [
        'is_active' => 'boolean',
        'display_order' => 'integer',
    ];

    public function networkMetricsLatest()
    {
        return $this->hasOne(NetworkMetricsLatest::class, 'network_id');
    }

    public function marketMetricsLatest()
    {
        return $this->hasOne(MarketMetricsLatest::class, 'network_id');
    }

    public function defiMetricsLatest()
    {
        return $this->hasOne(DefiMetricsLatest::class, 'network_id');
    }

    public function stablecoinMetricsLatest()
    {
        return $this->hasOne(StablecoinMetricsLatest::class, 'network_id');
    }

    public function history()
    {
        return $this->hasMany(NetworkMetricHistory::class, 'network_id');
    }

    public function alerts()
    {
        return $this->hasMany(Alert::class, 'network_id');
    }
}
```

---

## 4. Latest Metrics Models

Example `NetworkMetricsLatest`:

```php
class NetworkMetricsLatest extends Model
{
    use HasFactory;

    protected $table = 'blockchain_dashboard.network_metrics_latest';

    protected $fillable = [
        'network_id',
        'block_height',
        'transactions_24h',
        'active_addresses_24h',
        'avg_transaction_fee_usd',
        'hash_rate',
        'validator_count',
        'gas_price_gwei',
        'tps',
        'finality_seconds',
        'recorded_at',
    ];

    protected $casts = [
        'recorded_at' => 'datetime',
    ];

    public function network()
    {
        return $this->belongsTo(BlockchainNetwork::class, 'network_id');
    }
}
```

Repeat the same structure for:

- `MarketMetricsLatest`
- `DefiMetricsLatest`
- `StablecoinMetricsLatest`

---

## 5. Migration Naming

Recommended migration files:

```text
2026_01_01_000001_create_blockchain_networks_table.php
2026_01_01_000002_create_network_metrics_latest_table.php
2026_01_01_000003_create_market_metrics_latest_table.php
2026_01_01_000004_create_defi_metrics_latest_table.php
2026_01_01_000005_create_stablecoin_metrics_latest_table.php
2026_01_01_000006_create_network_metric_history_table.php
2026_01_01_000007_create_roles_table.php
2026_01_01_000008_update_users_table.php
2026_01_01_000009_create_user_roles_table.php
2026_01_01_000010_create_alerts_table.php
2026_01_01_000011_create_audit_logs_table.php
2026_01_01_000012_create_api_sync_logs_table.php
2026_01_01_000013_create_report_exports_table.php
2026_01_01_000014_create_api_provider_settings_table.php
```

---

## 6. Migration Example

Example migration for `blockchain_networks`:

```php
public function up(): void
{
    Schema::create('blockchain_dashboard.blockchain_networks', function (Blueprint $table) {
        $table->id();
        $table->string('network_key', 80)->unique();
        $table->string('name', 120);
        $table->string('native_symbol', 20);
        $table->string('network_type', 50)->default('L1');
        $table->string('chain_id', 80)->nullable();
        $table->text('explorer_url')->nullable();
        $table->text('official_website')->nullable();
        $table->text('logo_url')->nullable();
        $table->boolean('is_active')->default(true);
        $table->integer('display_order')->default(0);
        $table->timestampsTz();
    });
}

public function down(): void
{
    Schema::dropIfExists('blockchain_dashboard.blockchain_networks');
}
```

---

## 7. Network Seeder

Create:

```bash
php artisan make:seeder BlockchainNetworkSeeder
```

Seeder content:

```php
use App\Models\BlockchainNetwork;

class BlockchainNetworkSeeder extends Seeder
{
    public function run(): void
    {
        $networks = [
            ['network_key' => 'bitcoin', 'name' => 'Bitcoin', 'native_symbol' => 'BTC', 'network_type' => 'L1', 'display_order' => 1],
            ['network_key' => 'ethereum', 'name' => 'Ethereum', 'native_symbol' => 'ETH', 'network_type' => 'L1', 'chain_id' => '1', 'display_order' => 2],
            ['network_key' => 'bnb-chain', 'name' => 'BNB Chain', 'native_symbol' => 'BNB', 'network_type' => 'L1', 'chain_id' => '56', 'display_order' => 3],
            ['network_key' => 'solana', 'name' => 'Solana', 'native_symbol' => 'SOL', 'network_type' => 'L1', 'display_order' => 4],
            ['network_key' => 'tron', 'name' => 'TRON', 'native_symbol' => 'TRX', 'network_type' => 'L1', 'display_order' => 5],
            ['network_key' => 'polygon', 'name' => 'Polygon', 'native_symbol' => 'POL', 'network_type' => 'L2', 'chain_id' => '137', 'display_order' => 6],
            ['network_key' => 'base', 'name' => 'Base', 'native_symbol' => 'ETH', 'network_type' => 'L2', 'chain_id' => '8453', 'display_order' => 7],
            ['network_key' => 'arbitrum', 'name' => 'Arbitrum', 'native_symbol' => 'ETH', 'network_type' => 'L2', 'chain_id' => '42161', 'display_order' => 8],
        ];

        foreach ($networks as $network) {
            BlockchainNetwork::updateOrCreate(
                ['network_key' => $network['network_key']],
                $network
            );
        }
    }
}
```

---

## 8. Role Seeder

Create:

```bash
php artisan make:seeder RoleSeeder
```

Seeder content:

```php
use App\Models\Role;

class RoleSeeder extends Seeder
{
    public function run(): void
    {
        $roles = [
            ['role_key' => 'admin', 'name' => 'Administrator', 'description' => 'Full system access'],
            ['role_key' => 'analyst', 'name' => 'Analyst', 'description' => 'Analytics and reporting access'],
            ['role_key' => 'viewer', 'name' => 'Viewer', 'description' => 'Read-only access'],
        ];

        foreach ($roles as $role) {
            Role::updateOrCreate(['role_key' => $role['role_key']], $role);
        }
    }
}
```

---

## 9. DatabaseSeeder

Update `database/seeders/DatabaseSeeder.php`:

```php
public function run(): void
{
    $this->call([
        RoleSeeder::class,
        BlockchainNetworkSeeder::class,
    ]);
}
```

---

## 10. Migration and Seed Commands

```bash
php artisan migrate:fresh --seed
```

Validate:

```bash
php artisan tinker
```

```php
App\Models\BlockchainNetwork::count();
App\Models\Role::count();
```

Expected:

```text
8 networks
3 roles
```

---

## 11. Phase Deliverables

- Models created
- Migrations created
- Relationships defined
- Seeders created
- Initial networks seeded
- Initial roles seeded
- Database validated through Laravel

---

## 12. GitHub Commit

```bash
git add backend/app/Models backend/database docs/PHASE_05_LARAVEL_MIGRATIONS_MODELS_SEEDERS.md
git commit -m "Phase 5: add Laravel migrations models and seeders"
git push
```
