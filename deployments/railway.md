# Railway: Vercel Experience, Fair Pricing

The deployment platform with Vercel's developer experience but without the predatory pricing.

## Why Railway is Loved by Developers

- **Beautiful UI** that actually works
- **Built-in databases** - Postgres, MySQL, Redis, MongoDB
- **Fair pricing** - Pay for what you use
- **No surprises** - Transparent billing
- **Great support** - Responsive team

## Pricing That Makes Sense

- **$5 credit** to start (free)
- **Usage-based** - Typically $5-20/month
- **No seat pricing** - Unlimited team members
- **No bandwidth charges** - Included
- **Databases included** - No separate billing

## Quick Setup (5 Minutes)

### Option 1: Deploy from GitHub

1. Go to [railway.app](https://railway.app)
2. Click "Start a New Project"
3. Select "Deploy from GitHub repo"
4. Choose your repository
5. Railway auto-detects configuration
6. Click "Deploy"

### Option 2: CLI Deployment

```bash
# Install Railway CLI
npm install -g @railway/cli

# Login
railway login

# Initialize project
railway init

# Link to existing project
railway link

# Deploy
railway up
```

## Environment Variables

### Via Dashboard
1. Select your service
2. Variables tab
3. Add variables
4. Auto-restart on save

### Via CLI
```bash
railway variables set NODE_ENV=production
railway variables set DATABASE_URL=postgresql://...
```

### Via .env
```bash
# Pull from Railway
railway variables download > .env

# Push to Railway
railway variables set $(cat .env)
```

## Database Setup

### PostgreSQL (One Click)

1. Add New Service → Database → PostgreSQL
2. Connection string automatically injected
3. Use in your app:

```javascript
import { Pool } from 'pg'

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
})
```

### Redis

```javascript
import Redis from 'ioredis'

const redis = new Redis(process.env.REDIS_URL)
```

### MongoDB

```javascript
import { MongoClient } from 'mongodb'

const client = new MongoClient(process.env.MONGO_URL)
```

## Custom Domain

1. Settings → Domains
2. Add your domain
3. Update DNS:
   ```
   CNAME @ → your-app.up.railway.app
   ```
4. SSL auto-provisioned

## Deployment Configuration

```json
// railway.json
{
  "build": {
    "builder": "nixpacks",
    "buildCommand": "npm run build"
  },
  "deploy": {
    "startCommand": "npm start",
    "restartPolicyType": "on-failure",
    "restartPolicyMaxRetries": 10
  }
}
```

## Nixpacks Configuration

```toml
# nixpacks.toml
[phases.setup]
nixPkgs = ["nodejs-18_x", "yarn"]

[phases.build]
cmds = ["yarn install", "yarn build"]

[start]
cmd = "yarn start"
```

## Monorepo Support

```toml
# nixpacks.toml
[phases.build]
cmds = ["yarn workspace api build"]

[start]
cmd = "yarn workspace api start"
```

## Background Jobs & Cron

```javascript
// cron.js
import { CronJob } from 'cron'

new CronJob('0 * * * *', async () => {
  console.log('Running hourly task')
  // Your task here
}, null, true)
```

Add as separate service in Railway for isolation.

## Private Networking

Services can communicate internally:
```javascript
// Instead of public URL
const apiUrl = 'https://api.example.com'

// Use private networking
const apiUrl = 'http://api.railway.internal:3000'
```

## Monitoring & Logs

### Built-in Monitoring
- CPU/Memory usage graphs
- Network I/O
- Deployment history
- Real-time logs

### Custom Metrics
```javascript
// Integrate with observability tools
import { StatsD } from 'statsd-client'

const metrics = new StatsD({
  host: process.env.STATSD_HOST
})

metrics.increment('api.requests')
metrics.timing('api.response_time', responseTime)
```

## CI/CD Setup

### GitHub Actions
```yaml
name: Deploy to Railway

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: railway/deploy-action@v1
        with:
          service: my-app
          token: ${{ secrets.RAILWAY_TOKEN }}
```

## Migration from Vercel

### 1. Export from Vercel
```bash
vercel env pull .env.production
```

### 2. Create Railway Project
```bash
railway init
```

### 3. Set Environment Variables
```bash
railway variables set $(cat .env.production)
```

### 4. Deploy
```bash
railway up
```

### 5. Add Domain
Update DNS from Vercel to Railway

## Framework Configuration

### Next.js
```javascript
// next.config.js
module.exports = {
  output: 'standalone',
}
```

### SvelteKit
```javascript
// svelte.config.js
import adapter from '@sveltejs/adapter-node'

export default {
  kit: {
    adapter: adapter()
  }
}
```

### Remix
```javascript
// remix.config.js
module.exports = {
  serverBuildTarget: "node-cjs",
}
```

## Cost Examples

### Small App
- 1 service, 1 database
- Cost: ~$5/month
- Vercel equivalent: $200+

### Medium App
- 3 services, 2 databases, Redis
- Cost: ~$20/month
- Vercel equivalent: $500+

### Large App
- 10+ services, multiple databases
- Cost: ~$50-100/month
- Vercel equivalent: $1000+

## Scaling

### Horizontal Scaling
```json
{
  "deploy": {
    "replicas": 3,
    "minReplicas": 1,
    "maxReplicas": 10
  }
}
```

### Resource Limits
```json
{
  "deploy": {
    "memory": "512Mi",
    "cpu": "0.5"
  }
}
```

## Tips & Tricks

1. **Use private networking** for inter-service communication
2. **Enable healthchecks** for better reliability
3. **Set up staging environments** using Railway environments
4. **Use Railway CLI** for local development
5. **Monitor usage** to optimize costs

## Common Issues

**Build failures:**
- Check nixpacks.toml configuration
- Ensure all dependencies are listed
- Check build logs for specific errors

**Connection issues:**
- Verify environment variables
- Check private networking URLs
- Ensure services are in same project

**Performance:**
- Scale horizontally for traffic
- Use Redis for caching
- Optimize database queries

## Why Developers Love Railway

> "It's what Vercel should have been - fair pricing, great DX, no BS" - Startup CTO

> "Railway lets us focus on building, not fighting with configs" - Agency Lead

> "Moved from Vercel, saving $400/month with better performance" - SaaS Founder

## Conclusion

Railway offers:
- Vercel-like developer experience
- 90% cost reduction
- Better database integration
- Transparent, fair pricing
- Excellent support

**Setup time: 5 minutes**
**Monthly cost: $5-20 typical**
**Saved vs Vercel: $200-800/month**
