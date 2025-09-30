# Cloudflare Pages: Unlimited Bandwidth Deployment

The only platform with truly FREE unlimited bandwidth. Perfect for high-traffic applications.

## Why Cloudflare Pages Dominates

- **UNLIMITED bandwidth** - No overage charges, ever
- **275+ edge locations** - Fastest global performance  
- **Free tier** - Incredibly generous
- **Workers integration** - True edge computing
- **DDoS protection** - Enterprise-grade included

## Cost Comparison

| Traffic | Vercel Cost | Cloudflare Cost | You Save |
|---------|------------|-----------------|----------|
| 1 TB/month | $400 | $0 | $400 |
| 10 TB/month | $4,000 | $0 | $4,000 |
| 100 TB/month | $40,000 | $0 | $40,000 |

## Quick Setup (15 Minutes)

### Prerequisites

```bash
# Install Wrangler CLI
npm install -g wrangler

# For Next.js projects
npm install -D @cloudflare/next-on-pages vercel
```

### Option 1: GitHub Integration (Easiest)

1. Go to [dash.cloudflare.com](https://dash.cloudflare.com)
2. Pages → Create application → Connect to Git
3. Select repository
4. Configure build:
   ```
   Build command: npm run build
   Build output: /build (or /dist, /.next)
   ```
5. Deploy!

### Option 2: CLI Deployment

```bash
# Build your project
npm run build

# For Next.js
npx @cloudflare/next-on-pages

# Deploy
wrangler pages deploy build --project-name=my-app
```

## Next.js Configuration

```javascript
// next.config.js
module.exports = {
  output: 'export', // For static export
  // OR
  experimental: {
    runtime: 'edge', // For edge runtime
  },
}
```

## Environment Variables

```bash
# Set via CLI
wrangler pages secret put API_KEY --project=my-app

# Or in dashboard
Pages → Settings → Environment variables
```

## Edge Functions (Workers)

```javascript
// functions/api/hello.js
export async function onRequest(context) {
  return new Response(JSON.stringify({
    message: 'Hello from the edge!',
    country: context.request.cf?.country
  }), {
    headers: { 'Content-Type': 'application/json' }
  })
}
```

## KV Storage (Key-Value Database)

```toml
# wrangler.toml
[[kv_namespaces]]
binding = "MY_KV"
id = "your-namespace-id"
```

```javascript
// functions/api/cache.js
export async function onRequest({ env }) {
  // Write
  await env.MY_KV.put('key', 'value', {
    expirationTtl: 3600 // 1 hour
  })
  
  // Read
  const value = await env.MY_KV.get('key')
  
  return new Response(value)
}
```

## R2 Storage (S3 Compatible)

```toml
# wrangler.toml
[[r2_buckets]]
binding = "MY_BUCKET"
bucket_name = "my-files"
```

```javascript
// Upload files
export async function onRequestPost({ request, env }) {
  const formData = await request.formData()
  const file = formData.get('file')
  
  await env.MY_BUCKET.put(file.name, file.stream())
  
  return new Response('Uploaded!')
}
```

## Custom Domain Setup

1. Add domain in Pages dashboard
2. Update DNS:
   ```
   CNAME @ → my-app.pages.dev
   ```
3. SSL automatically provisioned

## Caching Strategy

```javascript
// Cache at edge for performance
export async function onRequest({ request, next }) {
  const response = await next()
  
  response.headers.set('Cache-Control', 'public, max-age=3600')
  response.headers.set('CDN-Cache-Control', 'max-age=86400')
  
  return response
}
```

## Migration from Vercel

### 1. Export from Vercel
```bash
vercel env pull .env.production
```

### 2. Update Configuration
```javascript
// Remove Vercel-specific configs
// Update image optimization
// Configure for edge runtime
```

### 3. Deploy to Cloudflare
```bash
# Install adapter
npm i -D @cloudflare/next-on-pages

# Build
npx @cloudflare/next-on-pages

# Deploy
wrangler pages deploy .vercel/output/static
```

## Performance Tips

1. **Use Cache API** aggressively
2. **Enable Auto Minify** in dashboard
3. **Use Cloudflare Images** for optimization
4. **Implement Smart Placement** for Workers
5. **Enable HTTP/3** for better performance

## Monitoring

- **Analytics**: Built-in, free
- **Web Analytics**: Privacy-focused
- **Real User Monitoring**: Performance metrics
- **Workers Analytics**: Function metrics

## Limitations to Know

- Build time: 20 minutes max
- Build frequency: 500/month (free)
- Function duration: 30 seconds max
- Function size: 10MB max

## Cost Breakdown

**Free Tier:**
- Unlimited bandwidth
- 500 builds/month
- 100k requests/day
- Unlimited sites

**Paid ($20/month):**
- 5,000 builds/month
- 10M requests/month
- Concurrent builds
- Build prioritization

## Real-World Example

A media site with 50M pageviews/month:
- **Vercel cost**: $2,000/month
- **Cloudflare cost**: $0
- **Annual savings**: $24,000

## Conclusion

Cloudflare Pages is unbeatable for:
- High-traffic sites
- Global applications
- Static/JAMstack sites
- Cost-conscious projects

**Setup time: 15 minutes**
**Monthly cost: $0-20**
**Saved vs Vercel: $400-4000+/month**
