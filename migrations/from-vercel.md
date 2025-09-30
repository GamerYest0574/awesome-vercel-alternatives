# Migrating from Vercel - Complete Guide

Stop overpaying. Start deploying with freedom. This guide helps you migrate from Vercel to better alternatives in under 30 minutes.

## Pre-Migration Checklist

### 1. Export Everything from Vercel

```bash
# Install Vercel CLI if needed
npm i -g vercel

# Login
vercel login

# List all projects
vercel list

# For each project, export:
vercel env pull .env.production
vercel pull
```

### 2. Document Your Setup

Record from Vercel dashboard:
- [ ] Environment variables (all of them)
- [ ] Custom domains
- [ ] Build settings
- [ ] Function regions
- [ ] Team members
- [ ] Current monthly bill (for comparison)

## Quick Migration Paths

### Option A: Hetzner + Coolify (Best Value)
**Time**: 30 minutes  
**Cost**: €4.51/month  
**Savings**: 99%  

```bash
# 1. Get Hetzner server
# 2. Install Coolify
curl -fsSL https://get.coolify.io | bash
# 3. Import GitHub repo
# 4. Paste env vars
# 5. Deploy!
```
[Full Guide →](./deployments/hetzner-coolify.md)

### Option B: Replit (Fastest)
**Time**: 2 minutes  
**Cost**: $7/month  
**Savings**: 95%  

```bash
# 1. Go to replit.com
# 2. Import from GitHub
# 3. Click Deploy
# Done!
```

### Option C: Cloudflare Pages (Free Bandwidth)
**Time**: 15 minutes  
**Cost**: $0-5/month  
**Savings**: 97%  

```bash
# 1. Install adapter
npm i -D @cloudflare/next-on-pages
# 2. Build
npx @cloudflare/next-on-pages
# 3. Deploy
wrangler pages deploy .vercel/output/static
```

## Framework-Specific Migration

### Next.js

Remove Vercel-specific code:
```javascript
// Remove from next.config.js
- const { withVercel } = require('@vercel/next');

// Update for standalone
module.exports = {
  output: 'standalone',
  // ... rest of config
}
```

### SvelteKit

Change adapter:
```javascript
// From
- import adapter from '@sveltejs/adapter-vercel';
// To
+ import adapter from '@sveltejs/adapter-node';
```

### Remix

Update config:
```javascript
// remix.config.js
module.exports = {
  serverBuildTarget: "node-cjs",
  // Remove Vercel-specific settings
}
```

### Nuxt 3

Change nitro preset:
```javascript
// nuxt.config.ts
export default defineNuxtConfig({
  nitro: {
-   preset: 'vercel',
+   preset: 'node-server'
  }
})
```

## Environment Variables Migration

### Export from Vercel
```bash
vercel env pull .env.production
```

### Import to Alternative

**Coolify**: Dashboard → Environment Variables → Paste

**Replit**: Secrets tab → Add each variable

**Cloudflare**: 
```bash
wrangler pages secret put KEY_NAME
```

**Railway**: Dashboard → Variables → Raw Editor → Paste

## Domain Migration

### 1. Remove from Vercel
- Go to project settings
- Domains → Remove
- Note DNS settings

### 2. Update DNS

| Platform | DNS Setup |
|----------|-----------|
| **Hetzner** | A record → Server IP |
| **Cloudflare** | CNAME → project.pages.dev |
| **Replit** | CNAME → project.repl.co |
| **Railway** | CNAME → project.up.railway.app |

### 3. SSL Certificates
All alternatives provide automatic SSL via Let's Encrypt!

## Database Migration

### From Vercel Postgres

Export:
```bash
pg_dump $POSTGRES_URL > backup.sql
```

Import to alternative:
```bash
psql $NEW_DATABASE_URL < backup.sql
```

### From Vercel KV (Redis)

Export:
```bash
redis-cli --rdb dump.rdb
```

Import:
```bash
redis-cli --rdb dump.rdb
```

## CI/CD Setup

### GitHub Actions

```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      # For Coolify
      - name: Deploy to Coolify
        run: curl -X POST ${{ secrets.COOLIFY_WEBHOOK }}
      
      # For Cloudflare
      - uses: cloudflare/pages-action@v1
        with:
          apiToken: ${{ secrets.CF_API_TOKEN }}
          
      # For Railway
      - uses: railway/deploy-action@v1
        with:
          token: ${{ secrets.RAILWAY_TOKEN }}
```

## Common Issues & Solutions

### Build Failures

```javascript
// Increase memory for builds
NODE_OPTIONS="--max-old-space-size=4096" npm run build
```

### Image Optimization

```javascript
// For self-hosted
const sharp = require('sharp');
// Or use Cloudflare Image Resizing
```

### Edge Functions

- **Cloudflare**: Use Workers
- **Hetzner**: Use regular Node.js
- **Replit**: Use serverless functions

## Post-Migration Checklist

- [ ] All pages load correctly
- [ ] Environment variables working
- [ ] Database connected
- [ ] Custom domains active
- [ ] SSL certificates valid
- [ ] Forms/APIs working
- [ ] Images optimized
- [ ] Analytics tracking
- [ ] Team access configured
- [ ] Monitoring setup

## Rollback Plan

Keep Vercel active for 1 week:
1. Don't delete Vercel project yet
2. Use subdomain for testing
3. Monitor errors
4. Gradual DNS switch
5. Delete Vercel after confirmation

## Cost Comparison After Migration

Document your savings:
- Screenshot Vercel final bill
- Screenshot new platform bill
- Share on social media
- Help others save too!

## Success Stories

> "Migrated 5 Next.js apps from Vercel to Hetzner. Went from $420/mo to €4.51/mo. Took 2 hours total." - @dev

> "Cloudflare Pages handles our 10M hits/month for FREE. Was paying $800 on Vercel." - @startup

> "Replit deployment is so simple, I moved all client projects there. Saving them thousands." - @agency

## Need Help?

- GitHub Issues: [github.com/fadijob-official/awesome-vercel-alternatives](https://github.com/fadijob-official/awesome-vercel-alternatives)
- Discord: [Join Community](https://discord.gg/vercel-alternatives)
- Twitter: [@fadijob-official](https://twitter.com/fadijob-official)

## Share Your Migration

Help others by sharing:
1. Your before/after bills
2. Time taken to migrate
3. Any challenges faced
4. Performance comparisons

Submit PR to add your story!

---

**Remember**: Every migration is a vote for fair pricing and open web. You're not just saving money, you're part of a movement.

#VercelAlternatives #DeploymentFreedom
