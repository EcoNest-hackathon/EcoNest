# 🚀 Інструкції з деплою EcoNest

## 📋 Огляд деплою

Проєкт EcoNest деплоїться на **Vercel** - платформі, оптимізованій для Next.js додатків. Вибір Vercel обумовлений:

- ✅ **Нативна підтримка Next.js** - автоматична оптимізація та SSR
- ✅ **Безкоштовний план** - достатній для демонстрації та тестування
- ✅ **Автоматичні деплої** - з GitHub репозиторію ??
- ✅ **CDN та Edge функції** - швидкість завантаження по всьому світу
- ✅ **SSL сертифікати** - автоматичне HTTPS

## 🌐 Production посилання

- **Основний сайт**: [https://econest-ukraine.vercel.app](https://eco-house-zeta.vercel.app/)
- **Preview деплою**: автоматично для кожної нової версії

## ⚙️ Налаштування деплою

### 1. Підключення до Vercel

```bash
# Встановлення Vercel CLI (опціонально)
npm i -g vercel

# Підключення репозиторію
vercel --prod
```

### 2. Змінні середовища на Vercel

В **Vercel Dashboard** → **Settings** → **Environment Variables** додано:

```env
# Analytics
NEXT_PUBLIC_GA_MEASUREMENT_ID=G-XXXXXXXXXX
NEXT_PUBLIC_GTM_ID=GTM-XXXXXXX

# Contact Forms  
CONTACT_EMAIL=contact@econest.ua
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=econest.team@gmail.com
SMTP_PASS=[SECURE_APP_PASSWORD]

# External APIs
DONATION_API_URL=https://donate.savelife.in.ua/api
DONATION_REDIRECT_URL=https://savelife.in.ua/donate/

# Build optimization
NEXT_TELEMETRY_DISABLED=1
```

### 3. Build команди

```json
{
  "scripts": {
    "build": "next build",
    "start": "next start",
    "export": "next export"
  }
}
```

## 🔄 Процес автоматичного деплою

### Vercel конфігурація (`vercel.json`)

```json
{
  "version": 2,
  "builds": [
    {
      "src": "package.json",
      "use": "@vercel/next"
    }
  ],
  "routes": [
    {
      "src": "/(.*)",
      "dest": "/$1"
    }
  ],
  "functions": {
    "pages/api/contact.ts": {
      "maxDuration": 30
    }
  }
}
```

## 📊 Оптимізації для продакшену

### 1. Next.js конфігурація (`next.config.js`)

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Оптимізація зображень
  images: {
    domains: ['econest.ua', 'images.unsplash.com'],
    formats: ['image/webp', 'image/avif'],
  },
  
  // Стиснення
  compress: true,
  
  // PWA налаштування
  pwa: {
    dest: 'public',
    disable: process.env.NODE_ENV === 'development',
  },
  
  // SEO оптимізація
  trailingSlash: false,
  
  // Аналітика
  experimental: {
    instrumentationHook: true,
  }
}

module.exports = nextConfig
```

### 2. Оптимізація Bundle Size

```bash
# Аналіз розміру bundle
npm run build
npm run analyze

# Результати:
Page                                Size     First Load JS
┌ ○ /                              2.1 kB          87.2 kB
├   └ css/styles.css               1.2 kB
├ ○ /404                           182 B           85.3 kB  
├ ○ /contacts                      1.8 kB          87.0 kB
├ ○ /faq                          1.5 kB          86.7 kB
├ ● /house/[id]                   2.3 kB          87.4 kB
└ ○ /models                       2.0 kB          87.1 kB
```

### 3. Performance метрики

- **First Contentful Paint**: < 1.5s
- **Largest Contentful Paint**: < 2.5s  
- **Cumulative Layout Shift**: < 0.1
- **Time to Interactive**: < 3.0s

## 🔒 Безпека

### HTTPS та SSL
- **Автоматичний SSL** через Vercel
- **HSTS заголовки** налаштовано
- **Content Security Policy** для захисту від XSS

### Захист API ключів
```javascript
// Перевірка змінних середовища
if (!process.env.NEXT_PUBLIC_GA_MEASUREMENT_ID) {
  console.warn('Analytics ID не налаштовано')
}

// Приховування чутливих даних в логах  
const sanitizedConfig = {
  ...config,
  smtp: '[HIDDEN]'
}
```

## 📈 Моніторинг та аналітика

### 1. Vercel Analytics
- **Web Vitals** - автоматичне відстеження
- **Real User Monitoring** - реальна продуктивність
- **Geographic distribution** - звідки користувачі

### 2. Google Analytics 4
```javascript
// Налаштування GA4
gtag('config', 'G-XXXXXXXXXX', {
  page_title: 'EcoNest - Модульні будинки',
  page_location: window.location.href,
  custom_map: {
    'custom_parameter': 'house_type'
  }
});
```

### 3. Error tracking
```javascript
// Sentry для відстеження помилок (опціонально)
if (process.env.NODE_ENV === 'production') {
  Sentry.init({
    dsn: process.env.SENTRY_DSN,
    environment: 'production'
  });
}
```

## 🚨 Troubleshooting

### Поширені проблеми

**1. Build fails з TypeScript помилками**
```bash
# Рішення
npm run type-check
npm run lint:fix
```

**2. Зображення не завантажуються**
```javascript
// next.config.js - додати домен
images: {
  domains: ['your-image-domain.com']
}
```

**3. API routes не працюють**
```javascript
// Перевірити структуру папок
pages/api/contact.ts // ✅ Правильно
pages/contact/api.ts // ❌ Неправильно  
```

### Логи деплою
```bash
# Перегляд логів через Vercel CLI
vercel logs [deployment-url]

# Або в Vercel Dashboard → Deployments → View Logs
```

## 📱 Mobile оптимізація

### Responsive Images
```javascript
import Image from 'next/image'

<Image
  src="/images/house-1.jpg"
  alt="Модульний будинок"
  width={800}
  height={600}
  responsive
  priority // для головного зображення
/>
```

### Service Worker (PWA)
```javascript
// Кешування для offline роботи
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js')
}
```

## 🔮 Майбутні покращення

### 1. CDN оптимізація
- Використання Vercel Edge Network
- Кешування статичних ресурсів
- Географічно розподілені сервери

### 2. Database інтеграція
```bash
# Потенційні варіанти
- Vercel Postgres (для контактів)
- Supabase (для аналітики)  
- MongoDB Atlas (для контенту)
```

### 3. CI/CD Pipeline
```yaml
# .github/workflows/deploy.yml
name: Deploy to Vercel
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Install Vercel CLI
        run: npm install -g vercel@latest
      - name: Deploy to Vercel
        run: vercel --prod --token=${{ secrets.VERCEL_TOKEN }}
```
---
**Статус сервісу**: [https://status.vercel.com](https://status.vercel.com)
