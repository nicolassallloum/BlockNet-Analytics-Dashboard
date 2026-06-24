# BlockNet Analytics Dashboard

**Project:** Blockchain Networks Analytics Dashboard  
**Database:** `blocknet_dashboard`  
**Schema:** `blockchain_dashboard`  
**Stack:** PostgreSQL, Laravel, Vue, Docker, Nginx, Ubuntu VPS  
**Initial Networks:** Bitcoin, Ethereum, BNB Chain, Solana, TRON, Polygon, Base, Arbitrum

---

# Phase 6 — Authentication, Authorization, and User Management

## 1. Phase Objective

Build secure user authentication and authorization for the dashboard.

This phase includes:

- Login
- Logout
- Current user endpoint
- Token-based API authentication
- Admin, analyst, and viewer roles
- User management endpoints
- Authorization middleware
- Audit logging for security events

---

## 2. Authentication Approach

Use Laravel Sanctum for API authentication.

Recommended frontend flow:

1. User submits email and password.
2. Backend validates credentials.
3. Backend creates Sanctum token.
4. Frontend stores token securely.
5. Frontend sends token in API requests.

---

## 3. Auth Controller

Create controller:

```bash
php artisan make:controller Api/V1/AuthController
```

Controller structure:

```php
<?php

namespace App\Http\Controllers\Api\V1;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use App\Models\User;

class AuthController extends Controller
{
    public function login(Request $request)
    {
        $request->validate([
            'email' => ['required', 'email'],
            'password' => ['required', 'string'],
        ]);

        $user = User::where('email', $request->email)->first();

        if (!$user || !Hash::check($request->password, $user->password)) {
            return response()->json([
                'message' => 'Invalid login credentials',
            ], 401);
        }

        if ($user->status !== 'active') {
            return response()->json([
                'message' => 'User account is not active',
            ], 403);
        }

        $token = $user->createToken('blocknet-dashboard-token')->plainTextToken;
        $user->update(['last_login_at' => now()]);

        return response()->json([
            'token' => $token,
            'user' => $user->load('roles'),
        ]);
    }

    public function me(Request $request)
    {
        return response()->json([
            'user' => $request->user()->load('roles'),
        ]);
    }

    public function logout(Request $request)
    {
        $request->user()->currentAccessToken()->delete();

        return response()->json([
            'message' => 'Logged out successfully',
        ]);
    }
}
```

---

## 4. User Model Updates

Update `app/Models/User.php`:

```php
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;

    protected $table = 'blockchain_dashboard.users';

    protected $fillable = [
        'name',
        'email',
        'password',
        'status',
        'last_login_at',
    ];

    protected $hidden = [
        'password',
        'remember_token',
    ];

    protected $casts = [
        'email_verified_at' => 'datetime',
        'last_login_at' => 'datetime',
        'password' => 'hashed',
    ];

    public function roles()
    {
        return $this->belongsToMany(Role::class, 'blockchain_dashboard.user_roles', 'user_id', 'role_id')
            ->withPivot('assigned_at');
    }

    public function hasRoleKey(string $roleKey): bool
    {
        return $this->roles()->where('role_key', $roleKey)->exists();
    }
}
```

---

## 5. Role Middleware

Create middleware:

```bash
php artisan make:middleware RoleMiddleware
```

Add:

```php
public function handle($request, Closure $next, ...$roles)
{
    $user = $request->user();

    if (!$user) {
        return response()->json(['message' => 'Unauthenticated'], 401);
    }

    foreach ($roles as $role) {
        if ($user->hasRoleKey($role)) {
            return $next($request);
        }
    }

    return response()->json(['message' => 'Forbidden'], 403);
}
```

Register middleware in Laravel bootstrap or HTTP kernel depending on Laravel version:

```php
'role' => \App\Http\Middleware\RoleMiddleware::class,
```

---

## 6. User Management Controller

Create controller:

```bash
php artisan make:controller Api/V1/UserManagementController
```

Required endpoints:

| Method | Endpoint | Role | Purpose |
|--------|----------|------|---------|
| GET | `/api/v1/users` | admin | List users |
| POST | `/api/v1/users` | admin | Create user |
| GET | `/api/v1/users/{id}` | admin | View user |
| PUT | `/api/v1/users/{id}` | admin | Update user |
| DELETE | `/api/v1/users/{id}` | admin | Disable user |
| POST | `/api/v1/users/{id}/roles` | admin | Assign roles |

---

## 7. API Routes

Update `routes/api.php`:

```php
Route::prefix('v1')->group(function () {
    Route::post('/auth/login', [AuthController::class, 'login']);

    Route::middleware('auth:sanctum')->group(function () {
        Route::post('/auth/logout', [AuthController::class, 'logout']);
        Route::get('/auth/me', [AuthController::class, 'me']);

        Route::middleware('role:admin')->group(function () {
            Route::apiResource('/users', UserManagementController::class);
            Route::post('/users/{user}/roles', [UserManagementController::class, 'assignRoles']);
        });
    });
});
```

---

## 8. Default Admin Seeder

Create:

```bash
php artisan make:seeder AdminUserSeeder
```

Seeder:

```php
use App\Models\User;
use App\Models\Role;
use Illuminate\Support\Facades\Hash;

class AdminUserSeeder extends Seeder
{
    public function run(): void
    {
        $admin = User::updateOrCreate(
            ['email' => 'admin@blocknet.local'],
            [
                'name' => 'BlockNet Admin',
                'password' => Hash::make('ChangeMe123!'),
                'status' => 'active',
            ]
        );

        $role = Role::where('role_key', 'admin')->first();

        if ($role) {
            $admin->roles()->syncWithoutDetaching([$role->id]);
        }
    }
}
```

Add it to `DatabaseSeeder` after roles:

```php
$this->call([
    RoleSeeder::class,
    BlockchainNetworkSeeder::class,
    AdminUserSeeder::class,
]);
```

---

## 9. Security Rules

- Passwords must always be hashed.
- Tokens must be deleted on logout.
- Disabled users cannot log in.
- Admin-only routes must use role middleware.
- Authentication failures should not reveal whether email exists.
- User create/update/delete actions should be stored in audit logs.

---

## 10. Test Login

```bash
curl -X POST http://127.0.0.1:8000/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@blocknet.local","password":"ChangeMe123!"}'
```

Expected:

```json
{
  "token": "...",
  "user": {
    "email": "admin@blocknet.local"
  }
}
```

---

## 11. Phase Deliverables

- Login API
- Logout API
- Current user API
- Role middleware
- Admin user management APIs
- Default admin seed user
- Audit-ready security structure

---

## 12. GitHub Commit

```bash
git add backend docs/PHASE_06_AUTHORIZATION_USER_MANAGEMENT.md
git commit -m "Phase 6: add authentication authorization and user management"
git push
```
