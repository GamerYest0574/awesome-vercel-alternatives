# Cloudflare Pages: Unlimited Bandwidth Next.js Hosting

**The only platform with truly FREE unlimited bandwidth.** Perfect for viral apps and high-traffic sites.

## Why Cloudflare Pages Dominates

- **UNLIMITED bandwidth** - No overage charges ever
- **275+ edge locations** - Fastest global performance
- **Free SSL** - Automatic HTTPS everywhere
- **DDoS protection** - Enterprise-grade included
- **Workers integration** - True edge computing
- **R2 storage** - S3-compatible object storage
- **D1 database** - SQLite at the edge

## Real Cost Comparison

| Traffic | Vercel | Cloudflare |
|---------|--------|------------|
| 1 TB/month | $400 | **$0** |
| 10 TB/month | $4,000 | **$0** |
| 100 TB/month | $40,000 | **$0** |

## Complete Setup Guide

### Prerequisites

```bash
# Install required packages
npm install -D @cloudflare/next-on-pages vercel wrangler

# Wrangler CLI (Cloudflare's CLI)
npm install -g wrangler
```

### Step 1: Prepare Next.js for Edge

```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    // Enable edge runtime
    runtime: 'edge',
    serverActions: true,
  },
  
  // Required for Cloudflare
  output: 'standalone',
  
  // Image optimization
  images: {
    loader: 'custom',
    loaderFile: './image-loader.js',
    remotePatterns: [
      { protocol: 'https', hostname: '**' }
    ],
  },
  
  // Disable x-powered-by header
  poweredByHeader: false,
}

module.exports = nextConfig
```

### Step 2: Create Image Loader

```javascript
// image-loader.js
export default function cloudflareLoader({ src, width, quality }) {
  // Cloudflare Image Resizing
  const params = [
    `w=${width}`,
    `q=${quality || 75}`,
    'f=auto', // Auto format (WebP/AVIF)
  ]
  
  if (src.startsWith('http')) {
    return `/cdn-cgi/image/${params.join(',')}/${src}`
  }
  
  return `/cdn-cgi/image/${params.join(',')}/https://${process.env.NEXT_PUBLIC_DOMAIN}${src}`
}
```

### Step 3: Configure Build

```json
// package.json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "build:cf": "npx @cloudflare/next-on-pages",
    "preview": "wrangler pages dev .vercel/output/static",
    "deploy": "npm run build && npm run build:cf && wrangler pages deploy .vercel/output/static"
  }
}
```

### Step 4: Deploy Options

#### Option A: GitHub Integration (Easiest)

1. Go to [dash.cloudflare.com](https://dash.cloudflare.com)
2. Workers & Pages → Create application → Pages
3. Connect GitHub account
4. Select repository
5. Configure build:
   ```yaml
   Build command: npx @cloudflare/next-on-pages@1
   Build output: .vercel/output/static
   Root directory: /
   ```
6. Add environment variables
7. Deploy!

#### Option B: Direct Upload

```bash
# Build locally
npm run build
npx @cloudflare/next-on-pages

# Deploy
wrangler pages deploy .vercel/output/static --project-name=my-app

# Get deployment URL
# https://my-app.pages.dev
```

#### Option C: CLI with Wrangler

```bash
# Login to Cloudflare
wrangler login

# Create project
wrangler pages project create my-app

# Deploy
wrangler pages deploy .vercel/output/static --project-name=my-app
```

### Step 5: Environment Variables

```bash
# Set via dashboard or CLI
wrangler pages secret put DATABASE_URL --project=my-app
wrangler pages secret put API_KEY --project=my-app

# Local development (.dev.vars)
DATABASE_URL=postgresql://localhost:5432/mydb
API_KEY=dev-key-123
NEXT_PUBLIC_API_URL=http://localhost:3001
```

### Step 6: Edge Functions Setup

```typescript
// app/api/edge/route.ts
import { NextRequest } from 'next/server'

export const runtime = 'edge' // Important!

export async function GET(request: NextRequest) {
  // Access Cloudflare bindings
  const env = process.env as any
  
  // Geolocation data
  const country = request.headers.get('CF-IPCountry')
  const city = request.headers.get('CF-IPCity')
  const timezone = request.headers.get('CF-Timezone')
  
  return Response.json({
    country,
    city,
    timezone,
    timestamp: Date.now()
  })
}
```

## Advanced Features

### KV Storage (Key-Value)

```toml
# wrangler.toml
name = "my-nextjs-app"
compatibility_date = "2024-01-01"

[[kv_namespaces]]
binding = "CACHE"
id = "your-kv-namespace-id"
```

```typescript
// app/api/kv/route.ts
export const runtime = 'edge'

export async function GET() {
  const env = process.env as any
  
  // Read from KV
  const value = await env.CACHE.get('my-key')
  
  return Response.json({ value })
}

export async function POST(request: NextRequest) {
  const env = process.env as any
  const { key, value } = await request.json()
  
  // Write to KV with TTL
  await env.CACHE.put(key, value, {
    expirationTtl: 3600 // 1 hour
  })
  
  return Response.json({ success: true })
}
```

### D1 Database (SQLite at Edge)

```bash
# Create database
wrangler d1 create my-database

# Create tables
wrangler d1 execute my-database --file=./schema.sql
```

```sql
-- schema.sql
CREATE TABLE users (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  email TEXT UNIQUE NOT NULL,
  name TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE posts (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER REFERENCES users(id),
  title TEXT NOT NULL,
  content TEXT,
  published BOOLEAN DEFAULT 0
);
```

```typescript
// app/api/db/route.ts
export const runtime = 'edge'

export async function GET() {
  const env = process.env as any
  const db = env.DB
  
  const { results } = await db.prepare(
    'SELECT * FROM posts WHERE published = 1'
  ).all()
  
  return Response.json({ posts: results })
}

export async function POST(request: NextRequest) {
  const env = process.env as any
  const db = env.DB
  const { title, content } = await request.json()
  
  const result = await db.prepare(
    'INSERT INTO posts (title, content) VALUES (?, ?)'
  ).bind(title, content).run()
  
  return Response.json({ id: result.meta.last_row_id })
}
```

### R2 Storage (Object Storage)

```toml
# wrangler.toml
[[r2_buckets]]
binding = "BUCKET"
bucket_name = "my-bucket"
```

```typescript
// app/api/upload/route.ts
export const runtime = 'edge'

export async function POST(request: NextRequest) {
  const env = process.env as any
  const formData = await request.formData()
  const file = formData.get('file') as File
  
  if (!file) {
    return Response.json({ error: 'No file' }, { status: 400 })
  }
  
  // Upload to R2
  const arrayBuffer = await file.arrayBuffer()
  await env.BUCKET.put(file.name, arrayBuffer, {
    httpMetadata: {
      contentType: file.type,
    }
  })
  
  return Response.json({ 
    url: `/api/files/${file.name}`,
    size: file.size
  })
}

export async function GET(request: NextRequest) {
  const env = process.env as any
  const url = new URL(request.url)
  const key = url.pathname.split('/').pop()
  
  const object = await env.BUCKET.get(key)
  
  if (!object) {
    return new Response('Not found', { status: 404 })
  }
  
  return new Response(object.body, {
    headers: {
      'Content-Type': object.httpMetadata?.contentType || 'application/octet-stream'
    }
  })
}
```

### Durable Objects (Stateful Edge)

```javascript
// durable-objects/counter.js
export class Counter {
  constructor(state) {
    this.state = state
    this.value = 0
  }
  
  async fetch(request) {
    const url = new URL(request.url)
    
    switch(url.pathname) {
      case '/increment':
        this.value++
        await this.state.storage.put('value', this.value)
        break
      case '/get':
        this.value = await this.state.storage.get('value') || 0
        break
    }
    
    return Response.json({ value: this.value })
  }
}
```

### Caching Strategy

```typescript
// app/api/cached/route.ts
export const runtime = 'edge'

export async function GET(request: NextRequest) {
  const cache = caches.default
  const cacheKey = new Request(request.url, request)
  
  // Check cache
  let response = await cache.match(cacheKey)
  
  if (!response) {
    // Generate fresh response
    const data = await fetchExpensiveData()
    
    response = Response.json(data, {
      headers: {
        'Cache-Control': 'public, max-age=3600, s-maxage=86400',
        'CDN-Cache-Control': 'max-age=86400',
      }
    })
    
    // Store in cache
    await cache.put(cacheKey, response.clone())
  }
  
  return response
}
```

### Custom Domain Setup

```bash
# Add domain via dashboard
# Pages project → Custom domains → Add domain

# Or via CLI
wrangler pages domain create my-app.com --project=my-app

# DNS configuration (if using external DNS)
# CNAME: @ → my-app.pages.dev
# or
# CNAME: www → my-app.pages.dev
```

## Performance Optimization

### 1. Smart Caching Headers

```typescript
// middleware.ts
import { NextResponse } from 'next/server'

export function middleware(request: NextRequest) {
  const response = NextResponse.next()
  
  // Cache static assets aggressively
  if (request.nextUrl.pathname.startsWith('/_next/static')) {
    response.headers.set('Cache-Control', 'public, max-age=31536000, immutable')
  }
  
  // Cache images
  if (request.nextUrl.pathname.match(/\.(jpg|jpeg|png|gif|webp|svg|ico)$/)) {
    response.headers.set('Cache-Control', 'public, max-age=86400, s-maxage=31536000')
  }
  
  return response
}
```

### 2. Image Optimization

```html
<!-- Automatic format selection -->
<img src="/cdn-cgi/image/w=800,q=80,f=auto/hero.jpg" />

<!-- Responsive images -->
<picture>
  <source
    srcset="/cdn-cgi/image/w=640,f=avif/hero.jpg"
    media="(max-width: 640px)"
    type="image/avif"
  />
  <source
    srcset="/cdn-cgi/image/w=1920,f=webp/hero.jpg"
    type="image/webp"
  />
  <img src="/cdn-cgi/image/w=1920,q=80/hero.jpg" />
</picture>
```

### 3. Route Caching

```typescript
// app/blog/[slug]/page.tsx
export const runtime = 'edge'
export const revalidate = 3600 // Cache for 1 hour

export async function generateStaticParams() {
  const posts = await getPosts()
  return posts.map(post => ({ slug: post.slug }))
}

export default async function BlogPost({ params }) {
  const post = await getPost(params.slug)
  return <article>{post.content}</article>
}
```

## Monitoring & Analytics

### Cloudflare Analytics (Free)

```html
<!-- Add to layout.tsx -->
<script 
  defer 
  src='https://static.cloudflareinsights.com/beacon.min.js'
  data-cf-beacon='{"token": "your-token"}'
/>
```

### Custom Analytics

```typescript
// lib/analytics.ts
export function trackEvent(name: string, data?: any) {
  if (typeof window !== 'undefined') {
    // Send to Cloudflare Analytics
    fetch('/api/analytics', {
      method: 'POST',
      body: JSON.stringify({ event: name, ...data })
    })
  }
}
```

## Troubleshooting

### Build Timeout

```json
// Increase build memory
{
  "scripts": {
    "build:cf": "NODE_OPTIONS='--max-old-space-size=8192' npx @cloudflare/next-on-pages"
  }
}
```

### Edge Runtime Compatibility

```javascript
// Check for edge runtime
if (typeof EdgeRuntime !== 'undefined') {
  // Edge-specific code
} else {
  // Node.js fallback
}
```

### Large Pages

```javascript
// Split large pages
import dynamic from 'next/dynamic'

const HeavyComponent = dynamic(() => import('./HeavyComponent'), {
  loading: () => <p>Loading...</p>,
  ssr: false
})
```

## CI/CD with GitHub Actions

```yaml
# .github/workflows/deploy.yml
name: Deploy to Cloudflare Pages

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - uses: actions/setup-node@v3
        with:
          node-version: '20'
          cache: 'npm'
      
      - run: npm ci
      - run: npm run build
      - run: npx @cloudflare/next-on-pages
      
      - uses: cloudflare/pages-action@v1
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          projectName: my-app
          directory: .vercel/output/static
          gitHubToken: ${{ secrets.GITHUB_TOKEN }}
```

## Cost Analysis

| Feature | Free Tier | Pro ($20/mo) | Business ($200/mo) |
|---------|-----------|--------------|-------------------|
| **Bandwidth** | ∞ | ∞ | ∞ |
| **Requests** | 10M/mo | 50M/mo | 500M/mo |
| **Builds** | 500/mo | 5,000/mo | 20,000/mo |
| **Concurrent builds** | 1 | 5 | 20 |
| **KV operations** | 100k/day | 10M/mo | 50M/mo |
| **D1 operations** | 5M/mo | 50M/mo | 500M/mo |
| **R2 storage** | 10GB | 100GB | 1TB |

## Performance Benchmarks

| Metric | Cloudflare | Vercel | AWS CloudFront |
|--------|------------|--------|----------------|
| **Global PoPs** | 275+ | 100+ | 450+ |
| **Avg TTFB** | 15ms | 45ms | 35ms |
| **Cache Hit Rate** | 95%+ | 85% | 90% |
| **DDoS Protection** | ✅ Free | ⚠️ Limited | 💰 Paid |
| **Bandwidth Cost** | $0 | $0.40/GB | $0.085/GB |

## Best Practices

1. **Always use edge runtime** for API routes
2. **Cache aggressively** with proper headers
3. **Use KV for sessions** instead of cookies
4. **Implement stale-while-revalidate** patterns
5. **Optimize images** with Cloudflare's resizing
6. **Use R2** instead of external S3
7. **Monitor with Analytics** (free tier generous)

## When to Choose Cloudflare Pages

✅ **Perfect for:**
- High-traffic sites (news, media)
- Global audience
- Static-heavy sites
- E-commerce
- Gaming/streaming platforms
- Cost-sensitive projects

❌ **Not ideal for:**
- Complex backend logic
- Long-running processes (>30s)
- WebSocket-heavy apps
- Sites needing >20min builds

## Conclusion

Cloudflare Pages offers:
- ✅ **Unlimited bandwidth** (worth $1000s)
- ✅ **275+ edge locations**
- ✅ **Free DDoS protection**
- ✅ **Integrated storage & database**
- ✅ **Generous free tier**

**The best choice for scalable, global Next.js apps.**
