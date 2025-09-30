# Fly.io: True Global Deployment Platform

Deploy your app close to users worldwide, not just CDN endpoints. Real servers in 30+ regions.

## Why Fly.io is Unique

- **Real global presence** - Actual servers, not just CDN
- **WebSocket native** - First-class support
- **Scales to zero** - Don't pay when idle
- **Multi-region databases** - Data close to users
- **Phoenix/Elixir first-class** - But great for Node.js too

## Pricing Model

- **Pay for what you use** - No minimums
- **Free allowances** - Generous for small apps
- **Predictable** - No surprise bills
- **Scale to zero** - Pause when not in use

Free allowances:
- 3 shared-cpu-1x VMs
- 3GB persistent storage
- 160GB bandwidth

## Quick Setup (20 Minutes)

### Prerequisites

```bash
# Install flyctl
curl -L https://fly.io/install.sh | sh

# Sign up/login
fly auth signup
# or
fly auth login
```

### Deploy Your App

```bash
# In your project directory
fly launch

# Follow prompts:
# - Choose app name
# - Select region(s)
# - Setup PostgreSQL? (optional)
# - Deploy now? Yes
```

This creates `fly.toml` configuration file.

## Configuration (fly.toml)

```toml
app = "my-app"
primary_region = "iad"  # Ashburn, Virginia

[http_service]
  internal_port = 3000
  force_https = true
  auto_stop_machines = true
  auto_start_machines = true
  min_machines_running = 0

[env]
  NODE_ENV = "production"

[[vm]]
  cpu_kind = "shared"
  cpus = 1
  memory_mb = 256
```

## Dockerfile Setup

```dockerfile
FROM node:18-alpine AS base

# Dependencies
FROM base AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Build
FROM base AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Runner
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production

COPY --from=deps /app/node_modules ./node_modules
COPY --from=builder /app/build ./build
COPY --from=builder /app/public ./public
COPY --from=builder /app/package*.json ./

EXPOSE 3000
CMD ["npm", "start"]
```

## Environment Variables

```bash
# Set secrets (encrypted)
fly secrets set API_KEY=secret123
fly secrets set DATABASE_URL=postgresql://...

# List secrets
fly secrets list

# Remove secret
fly secrets unset API_KEY
```

## Database Setup

### PostgreSQL (Managed)

```bash
# Create Postgres cluster
fly postgres create

# Attach to app
fly postgres attach --app my-app

# Connection string auto-added as DATABASE_URL
```

### Redis (via Upstash)

```bash
fly redis create
```

### SQLite with LiteFS

```toml
# fly.toml
[mounts]
  source = "data"
  destination = "/data"

[experimental]
  enable_litefs = true
```

## Multi-Region Deployment

```bash
# Add regions
fly regions add sin hkg syd  # Asia-Pacific
fly regions add fra ams cdg  # Europe
fly regions add gru eze scl  # South America

# List regions
fly regions list

# Set backup regions
fly regions backup fra ams
```

## Scaling

### Horizontal Scaling

```bash
# Scale to multiple instances
fly scale count 3

# Scale by region
fly scale count 2 --region iad
fly scale count 1 --region fra
```

### Vertical Scaling

```bash
# Change VM size
fly scale vm shared-cpu-2x --memory 512

# Available sizes:
# shared-cpu-1x: 256MB
# shared-cpu-2x: 512MB
# shared-cpu-4x: 1GB
# shared-cpu-8x: 2GB
# dedicated-cpu-1x: 2GB
# dedicated-cpu-2x: 4GB
# dedicated-cpu-4x: 8GB
# dedicated-cpu-8x: 16GB
```

### Auto-scaling

```toml
# fly.toml
[http_service]
  min_machines_running = 1
  max_machines_running = 10
  
[http_service.concurrency]
  type = "requests"
  hard_limit = 250
  soft_limit = 200
```

## WebSocket Support

```javascript
// Native WebSocket support
import { WebSocketServer } from 'ws'

const wss = new WebSocketServer({ 
  port: process.env.PORT || 3000 
})

wss.on('connection', (ws) => {
  ws.on('message', (data) => {
    // Broadcast to all clients
    wss.clients.forEach((client) => {
      client.send(data)
    })
  })
})
```

## Custom Domains

```bash
# Add custom domain
fly certs create example.com

# Show DNS instructions
fly certs show example.com

# Add A record pointing to Fly IP
# Add AAAA record for IPv6
```

## Monitoring

### Built-in Metrics
```bash
# View logs
fly logs

# Monitor resources
fly status

# View metrics dashboard
fly dashboard
```

### Prometheus Metrics
```javascript
import prometheus from 'prom-client'

const register = new prometheus.Registry()
prometheus.collectDefaultMetrics({ register })

app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType)
  res.end(await register.metrics())
})
```

## Migration from Vercel

### 1. Prepare Your App

```javascript
// Ensure PORT is configurable
const PORT = process.env.PORT || 3000
app.listen(PORT)
```

### 2. Export from Vercel
```bash
vercel env pull .env.production
```

### 3. Deploy to Fly
```bash
# Initialize
fly launch --no-deploy

# Set secrets
fly secrets import < .env.production

# Deploy
fly deploy
```

### 4. Migrate Domain
Update DNS to point to Fly.io IPs

## Framework Examples

### Next.js
```dockerfile
FROM node:18-alpine AS base

FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

FROM base AS runner
WORKDIR /app
ENV NODE_ENV production
COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static

EXPOSE 3000
CMD ["node", "server.js"]
```

### SvelteKit
```javascript
// svelte.config.js
import adapter from '@sveltejs/adapter-node'

export default {
  kit: {
    adapter: adapter({
      out: 'build',
      precompress: false
    })
  }
}
```

## Advanced Features

### Blue-Green Deployments
```bash
fly deploy --strategy bluegreen
```

### Canary Deployments
```bash
fly deploy --strategy canary
```

### Volume Storage
```bash
# Create volume
fly volumes create data --size 10

# Mount in fly.toml
[mounts]
  source = "data"
  destination = "/data"
```

### Scheduled Jobs
```javascript
// Use node-cron or similar
import cron from 'node-cron'

cron.schedule('0 * * * *', () => {
  console.log('Running hourly task')
})
```

## Cost Examples

### Small App (Free)
- 1 VM (shared-cpu-1x)
- 256MB RAM
- 1GB storage
- **Cost: $0**

### Medium App
- 3 VMs across regions
- 512MB RAM each
- PostgreSQL database
- **Cost: ~$30/month**
- Vercel equivalent: $500+

### Large App
- 10 VMs globally
- 2GB RAM each
- Multi-region Postgres
- Redis cache
- **Cost: ~$150/month**
- Vercel equivalent: $2000+

## Performance Tips

1. **Deploy close to users** - Use multiple regions
2. **Use persistent connections** - For databases
3. **Enable HTTP/3** - Better performance
4. **Cache aggressively** - Use Redis
5. **Optimize images** - Use CDN or proxy

## Why Choose Fly.io

> "Real global deployment, not just CDN endpoints" - DevOps Lead

> "WebSocket support that actually works globally" - Real-time App Developer

> "Scales to zero saved us hundreds per month" - Startup Founder

## Conclusion

Fly.io excels at:
- True global presence
- Real-time applications
- WebSocket-heavy apps
- Multi-region databases
- Scale-to-zero workloads

**Setup time: 20 minutes**
**Monthly cost: $0-50 typical**
**Saved vs Vercel: $200-1500/month**
