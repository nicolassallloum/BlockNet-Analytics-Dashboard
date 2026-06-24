# BlockNet Analytics Dashboard

**Project:** Blockchain Networks Analytics Dashboard  
**Database:** `blocknet_dashboard`  
**Schema:** `blockchain_dashboard`  
**Stack:** PostgreSQL, Laravel, Vue, Docker, Nginx, Ubuntu VPS  
**Initial Networks:** Bitcoin, Ethereum, BNB Chain, Solana, TRON, Polygon, Base, Arbitrum

---

# Phase 13 — Docker, Nginx, and Ubuntu VPS Deployment

## 1. Phase Objective

Prepare the BlockNet Analytics Dashboard for production deployment using Docker, Nginx, PostgreSQL, Laravel queue workers, and an Ubuntu VPS.

This phase covers:

- Docker configuration
- Docker Compose services
- Laravel production setup
- Vue production build
- Nginx reverse proxy
- SSL preparation
- Queue worker setup
- Scheduler cron setup
- Deployment checklist

---

## 2. Production Architecture

```text
User Browser
    ↓
Nginx HTTPS Reverse Proxy
    ↓
Vue Frontend Static Build
    ↓
Laravel API Backend
    ↓
PostgreSQL Database
    ↓
Queue Worker + Scheduler
```

---

## 3. Docker Services

Required services:

| Service | Purpose |
|---------|---------|
| `nginx` | Reverse proxy and frontend serving |
| `backend` | Laravel API |
| `frontend` | Vue build container or static files |
| `postgres` | PostgreSQL database |
| `queue` | Laravel queue worker |
| `scheduler` | Laravel scheduler runner |

---

## 4. Docker Compose File

Create:

```text
docker-compose.yml
```

Example:

```yaml
services:
  postgres:
    image: postgres:16
    container_name: blocknet_postgres
    restart: unless-stopped
    environment:
      POSTGRES_DB: blocknet_dashboard
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: blocknet_backend
    restart: unless-stopped
    env_file:
      - ./backend/.env
    depends_on:
      - postgres
    volumes:
      - ./backend:/var/www/html

  queue:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: blocknet_queue
    restart: unless-stopped
    env_file:
      - ./backend/.env
    depends_on:
      - backend
    command: php artisan queue:work --sleep=3 --tries=3 --timeout=120
    volumes:
      - ./backend:/var/www/html

  scheduler:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: blocknet_scheduler
    restart: unless-stopped
    env_file:
      - ./backend/.env
    depends_on:
      - backend
    command: sh -c "while true; do php artisan schedule:run; sleep 60; done"
    volumes:
      - ./backend:/var/www/html

  nginx:
    image: nginx:stable-alpine
    container_name: blocknet_nginx
    restart: unless-stopped
    depends_on:
      - backend
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./docker/nginx/default.conf:/etc/nginx/conf.d/default.conf
      - ./frontend/dist:/usr/share/nginx/html
      - ./backend:/var/www/html

volumes:
  postgres_data:
```

---

## 5. Backend Dockerfile

Create:

```text
backend/Dockerfile
```

```dockerfile
FROM php:8.3-fpm

RUN apt-get update && apt-get install -y \
    git \
    unzip \
    libpq-dev \
    libzip-dev \
    zip \
    && docker-php-ext-install pdo pdo_pgsql zip

COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

WORKDIR /var/www/html

COPY . .

RUN composer install --no-interaction --prefer-dist --optimize-autoloader

RUN chown -R www-data:www-data storage bootstrap/cache

CMD ["php-fpm"]
```

---

## 6. Nginx Configuration

Create:

```text
docker/nginx/default.conf
```

```nginx
server {
    listen 80;
    server_name _;

    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api {
        proxy_pass http://backend:9000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Alternative production setup can use Nginx + PHP-FPM upstream configuration if Laravel is served directly by PHP-FPM.

---

## 7. Vue Production Build

```bash
cd frontend
npm install
npm run build
```

Output:

```text
frontend/dist/
```

---

## 8. Laravel Production Commands

```bash
cd backend
composer install --no-dev --optimize-autoloader
php artisan key:generate
php artisan migrate --force
php artisan db:seed --force
php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan storage:link
```

---

## 9. Ubuntu VPS Setup

Install packages:

```bash
sudo apt update
sudo apt install -y git curl unzip nginx docker.io docker-compose-plugin
sudo systemctl enable docker
sudo systemctl start docker
```

Clone project:

```bash
cd /var/www
sudo git clone <repository-url> blocknet
sudo chown -R $USER:$USER /var/www/blocknet
cd /var/www/blocknet
```

---

## 10. Environment Production Rules

Production `.env` must use:

```env
APP_ENV=production
APP_DEBUG=false
APP_URL=https://your-domain.com
DB_HOST=postgres
CACHE_STORE=database
QUEUE_CONNECTION=database
```

Never commit real API keys or production passwords.

---

## 11. Start Docker Stack

```bash
docker compose up -d --build
```

Check containers:

```bash
docker compose ps
```

View logs:

```bash
docker compose logs -f backend
docker compose logs -f queue
docker compose logs -f scheduler
```

---

## 12. SSL with Certbot

If Nginx is installed directly on host:

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d your-domain.com
```

If Nginx is containerized, use a reverse proxy SSL container or mount certificates into the Nginx container.

---

## 13. Deployment Checklist

- `.env` configured
- Database password changed
- API keys added securely
- `APP_DEBUG=false`
- Migrations executed
- Seeders executed
- Frontend build generated
- Nginx route working
- HTTPS working
- Queue worker running
- Scheduler running
- Logs writable
- Backups configured

---

## 14. Phase Deliverables

- Docker Compose file
- Backend Dockerfile
- Nginx config
- Production environment checklist
- VPS deployment steps
- Queue worker setup
- Scheduler setup
- SSL preparation

---

## 15. GitHub Commit

```bash
git add docker-compose.yml backend/Dockerfile docker/nginx docs/PHASE_13_DOCKER_NGINX_UBUNTU_VPS_DEPLOYMENT.md
git commit -m "Phase 13: add Docker Nginx and VPS deployment setup"
git push
```
