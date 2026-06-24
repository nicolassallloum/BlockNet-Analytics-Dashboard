# BlockNet Analytics Dashboard

**Project:** Blockchain Networks Analytics Dashboard  
**Database:** `blocknet_dashboard`  
**Schema:** `blockchain_dashboard`  
**Stack:** PostgreSQL, Laravel, Vue, Docker, Nginx, Ubuntu VPS  
**Initial Networks:** Bitcoin, Ethereum, BNB Chain, Solana, TRON, Polygon, Base, Arbitrum

---

# Phase 4 — Laravel Backend Project Setup

## 1. Phase Objective

Create the Laravel backend application that will expose secure REST APIs for the BlockNet Analytics Dashboard.

This phase prepares the backend foundation only. Database structure was completed in Phase 3. The backend will later connect to the PostgreSQL schema and provide APIs for authentication, dashboard metrics, alerts, reports, sync jobs, and admin settings.

---

## 2. Backend Folder Structure

Recommended repository structure:

```text
blocknet-analytics-dashboard/
├── backend/
├── frontend/
├── database/
│   └── sql/
├── docker/
├── docs/
└── README.md
```

Laravel project location:

```text
backend/
```

---

## 3. Create Laravel Project

```bash
cd blocknet-analytics-dashboard
composer create-project laravel/laravel backend
cd backend
```

Check Laravel version:

```bash
php artisan --version
```

---

## 4. Install Required Backend Packages

```bash
composer require laravel/sanctum
composer require spatie/laravel-permission
composer require maatwebsite/excel
composer require barryvdh/laravel-dompdf
composer require guzzlehttp/guzzle
```

Optional development tools:

```bash
composer require --dev laravel/pint
composer require --dev nunomaduro/collision
```

---

## 5. Environment Configuration

Update `backend/.env`:

```env
APP_NAME="BlockNet Analytics Dashboard"
APP_ENV=local
APP_KEY=
APP_DEBUG=true
APP_URL=http://localhost:8000

LOG_CHANNEL=stack
LOG_LEVEL=debug

DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=blocknet_dashboard
DB_USERNAME=postgres
DB_PASSWORD=postgres

CACHE_STORE=database
QUEUE_CONNECTION=database
SESSION_DRIVER=database
SESSION_LIFETIME=120

SANCTUM_STATEFUL_DOMAINS=localhost,localhost:5173,127.0.0.1,127.0.0.1:5173
FRONTEND_URL=http://localhost:5173
```

Generate application key:

```bash
php artisan key:generate
```

---

## 6. Configure PostgreSQL Schema Search Path

In `config/database.php`, update the PostgreSQL connection:

```php
'pgsql' => [
    'driver' => 'pgsql',
    'url' => env('DB_URL'),
    'host' => env('DB_HOST', '127.0.0.1'),
    'port' => env('DB_PORT', '5432'),
    'database' => env('DB_DATABASE', 'laravel'),
    'username' => env('DB_USERNAME', 'root'),
    'password' => env('DB_PASSWORD', ''),
    'charset' => env('DB_CHARSET', 'utf8'),
    'prefix' => '',
    'prefix_indexes' => true,
    'search_path' => 'blockchain_dashboard,public',
    'sslmode' => 'prefer',
],
```

---

## 7. Publish Sanctum

```bash
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
php artisan migrate
```

---

## 8. Publish Permission Package

```bash
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider"
php artisan migrate
```

---

## 9. Create Main API Folder Structure

```bash
mkdir -p app/Http/Controllers/Api/V1
mkdir -p app/Services/Blockchain
mkdir -p app/Services/MarketData
mkdir -p app/Services/Reporting
mkdir -p app/Services/Alerts
mkdir -p app/Repositories
mkdir -p app/DTO
mkdir -p app/Enums
mkdir -p app/Jobs
mkdir -p app/Console/Commands
mkdir -p app/Http/Requests/Api/V1
mkdir -p app/Http/Resources/Api/V1
```

---

## 10. API Route Prefix

Create `routes/api.php` structure:

```php
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\V1\AuthController;

Route::prefix('v1')->group(function () {
    Route::post('/auth/login', [AuthController::class, 'login']);

    Route::middleware('auth:sanctum')->group(function () {
        Route::post('/auth/logout', [AuthController::class, 'logout']);
        Route::get('/auth/me', [AuthController::class, 'me']);
    });
});
```

---

## 11. Health Check Endpoint

Create controller:

```bash
php artisan make:controller Api/V1/HealthCheckController
```

Add:

```php
<?php

namespace App\Http\Controllers\Api\V1;

use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\DB;

class HealthCheckController extends Controller
{
    public function __invoke()
    {
        DB::select('SELECT 1');

        return response()->json([
            'status' => 'ok',
            'service' => 'blocknet-backend',
            'database' => 'connected',
            'timestamp' => now()->toISOString(),
        ]);
    }
}
```

Route:

```php
use App\Http\Controllers\Api\V1\HealthCheckController;

Route::get('/v1/health', HealthCheckController::class);
```

---

## 12. CORS Configuration

Update `config/cors.php`:

```php
'paths' => ['api/*', 'sanctum/csrf-cookie'],
'allowed_methods' => ['*'],
'allowed_origins' => [env('FRONTEND_URL', 'http://localhost:5173')],
'allowed_headers' => ['*'],
'exposed_headers' => [],
'max_age' => 0,
'supports_credentials' => true,
```

---

## 13. Local Run Commands

```bash
php artisan serve --host=127.0.0.1 --port=8000
```

Test:

```bash
curl http://127.0.0.1:8000/api/v1/health
```

Expected response:

```json
{
  "status": "ok",
  "service": "blocknet-backend",
  "database": "connected"
}
```

---

## 14. Phase Deliverables

- Laravel backend created in `backend/`
- Required Composer packages installed
- `.env` configured for PostgreSQL
- PostgreSQL schema search path configured
- Sanctum installed
- Permission package installed
- API v1 structure prepared
- Health check endpoint created
- CORS configured for Vue frontend

---

## 15. GitHub Commit

```bash
git add backend docs/PHASE_04_LARAVEL_BACKEND_PROJECT_SETUP.md
git commit -m "Phase 4: setup Laravel backend project"
git push
```
