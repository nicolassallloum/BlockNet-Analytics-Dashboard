# BlockNet Analytics Dashboard

**Project:** Blockchain Networks Analytics Dashboard  
**Database:** `blocknet_dashboard`  
**Schema:** `blockchain_dashboard`  
**Stack:** PostgreSQL, Laravel, Vue, Docker, Nginx, Ubuntu VPS  
**Initial Networks:** Bitcoin, Ethereum, BNB Chain, Solana, TRON, Polygon, Base, Arbitrum

---

# Phase 10 — Vue Frontend Project Setup

## 1. Phase Objective

Create the Vue frontend application for the BlockNet Analytics Dashboard.

This phase prepares the frontend foundation:

- Vue project setup
- Routing
- API client
- Authentication store
- Layout structure
- Dark dashboard theme
- Base components
- Environment configuration

---

## 2. Create Vue Project

From repository root:

```bash
npm create vite@latest frontend -- --template vue
cd frontend
npm install
```

Install dependencies:

```bash
npm install axios pinia vue-router chart.js vue-chartjs lucide-vue-next
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

---

## 3. Frontend Folder Structure

```text
frontend/src/
├── assets/
├── components/
│   ├── common/
│   ├── dashboard/
│   ├── charts/
│   └── layout/
├── layouts/
│   └── DashboardLayout.vue
├── pages/
│   ├── LoginPage.vue
│   ├── DashboardPage.vue
│   ├── NetworksPage.vue
│   ├── NetworkDetailsPage.vue
│   ├── AlertsPage.vue
│   ├── ReportsPage.vue
│   └── AdminPage.vue
├── router/
│   └── index.js
├── services/
│   └── api.js
├── stores/
│   ├── authStore.js
│   └── dashboardStore.js
├── utils/
└── main.js
```

---

## 4. Environment File

Create `frontend/.env`:

```env
VITE_APP_NAME="BlockNet Analytics Dashboard"
VITE_API_BASE_URL=http://127.0.0.1:8000/api/v1
```

Create `frontend/.env.example` with the same keys and empty production values.

---

## 5. Axios API Client

Create `src/services/api.js`:

```js
import axios from 'axios'

const api = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  headers: {
    'Content-Type': 'application/json',
    Accept: 'application/json'
  }
})

api.interceptors.request.use((config) => {
  const token = localStorage.getItem('blocknet_token')
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})

api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem('blocknet_token')
      window.location.href = '/login'
    }
    return Promise.reject(error)
  }
)

export default api
```

---

## 6. Pinia Setup

Update `src/main.js`:

```js
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'
import router from './router'
import './style.css'

createApp(App)
  .use(createPinia())
  .use(router)
  .mount('#app')
```

---

## 7. Auth Store

Create `src/stores/authStore.js`:

```js
import { defineStore } from 'pinia'
import api from '../services/api'

export const useAuthStore = defineStore('auth', {
  state: () => ({
    user: null,
    token: localStorage.getItem('blocknet_token'),
    loading: false,
    error: null
  }),

  getters: {
    isAuthenticated: (state) => Boolean(state.token),
    roles: (state) => state.user?.roles || []
  },

  actions: {
    async login(email, password) {
      this.loading = true
      this.error = null
      try {
        const response = await api.post('/auth/login', { email, password })
        this.token = response.data.token
        this.user = response.data.user
        localStorage.setItem('blocknet_token', this.token)
      } catch (error) {
        this.error = error.response?.data?.message || 'Login failed'
        throw error
      } finally {
        this.loading = false
      }
    },

    async fetchMe() {
      const response = await api.get('/auth/me')
      this.user = response.data.user
    },

    async logout() {
      await api.post('/auth/logout')
      this.token = null
      this.user = null
      localStorage.removeItem('blocknet_token')
    }
  }
})
```

---

## 8. Router Setup

Create `src/router/index.js`:

```js
import { createRouter, createWebHistory } from 'vue-router'
import { useAuthStore } from '../stores/authStore'
import LoginPage from '../pages/LoginPage.vue'
import DashboardPage from '../pages/DashboardPage.vue'
import NetworksPage from '../pages/NetworksPage.vue'
import NetworkDetailsPage from '../pages/NetworkDetailsPage.vue'
import AlertsPage from '../pages/AlertsPage.vue'
import ReportsPage from '../pages/ReportsPage.vue'
import AdminPage from '../pages/AdminPage.vue'

const routes = [
  { path: '/login', component: LoginPage },
  { path: '/', component: DashboardPage, meta: { requiresAuth: true } },
  { path: '/networks', component: NetworksPage, meta: { requiresAuth: true } },
  { path: '/networks/:networkKey', component: NetworkDetailsPage, meta: { requiresAuth: true } },
  { path: '/alerts', component: AlertsPage, meta: { requiresAuth: true } },
  { path: '/reports', component: ReportsPage, meta: { requiresAuth: true } },
  { path: '/admin', component: AdminPage, meta: { requiresAuth: true, role: 'admin' } }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

router.beforeEach((to) => {
  const auth = useAuthStore()
  if (to.meta.requiresAuth && !auth.isAuthenticated) {
    return '/login'
  }
})

export default router
```

---

## 9. Tailwind Configuration

Update `tailwind.config.js`:

```js
export default {
  content: ['./index.html', './src/**/*.{vue,js,ts,jsx,tsx}'],
  theme: {
    extend: {}
  },
  plugins: []
}
```

Update `src/style.css`:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

body {
  margin: 0;
  background: #020617;
  color: #e5e7eb;
  font-family: Inter, system-ui, sans-serif;
}
```

---

## 10. Dashboard Layout

Create:

```text
src/layouts/DashboardLayout.vue
```

Layout sections:

- Sidebar navigation
- Top header
- Main content area
- User menu
- Logout button

Navigation items:

- Dashboard
- Networks
- Alerts
- Reports
- Admin

---

## 11. Run Frontend

```bash
npm run dev
```

Expected URL:

```text
http://localhost:5173
```

---

## 12. Phase Deliverables

- Vue project created
- Dependencies installed
- Axios API client created
- Pinia configured
- Auth store created
- Router configured
- Base layout prepared
- Tailwind configured
- Dark dashboard base theme applied

---

## 13. GitHub Commit

```bash
git add frontend docs/PHASE_10_VUE_FRONTEND_PROJECT_SETUP.md
git commit -m "Phase 10: setup Vue frontend project"
git push
```
