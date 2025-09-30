# Replit: 2-Minute Deployment Guide

The fastest way to deploy any JavaScript framework. Period.

## Why Replit is Perfect for Quick Deployment

- **2 minutes** from GitHub to production (literally)
- **$7/month** for production hosting
- **Built-in IDE** - code from anywhere
- **Instant previews** - see changes immediately
- **Multiplayer** - code with your team in real-time

## Quick Start (2 Minutes)

### Option 1: Import from GitHub

1. Go to [replit.com](https://replit.com)
2. Click "Create Repl" → "Import from GitHub"
3. Paste your repo URL
4. Replit auto-detects framework
5. Click "Import and Run"
6. Your app is running!
7. Click "Deploy" for production

### Option 2: Start from Template

1. Visit [replit.com/@templates](https://replit.com/@templates)
2. Choose your framework (Next.js, SvelteKit, etc.)
3. Click "Use Template"
4. Start coding immediately

## Production Deployment

### Basic Setup

1. In your Repl, click the "Deploy" button
2. Choose deployment type:
   - **Reserved VM**: Always-on ($7/month)
   - **Autoscale**: Pay per request

### Configuration

```toml
# .replit file
run = "npm run start"
entrypoint = "index.js"

[env]
NODE_ENV = "production"

[nix]
channel = "stable-23_05"

[[ports]]
localPort = 3000
externalPort = 80
```

## Environment Variables

1. Click "Secrets" icon (🔒)
2. Add your variables:
```
DATABASE_URL=postgresql://...
REDIS_URL=redis://...
API_KEY=your-key
```

## Custom Domain

1. Go to Deployments → Settings
2. Add your domain
3. Update DNS:
```
CNAME @ → your-app.repl.co
```

## Database Options

### Built-in Replit DB (NoSQL)
```javascript
import Database from "@replit/database"
const db = new Database()

await db.set("users:1", { name: "John" })
const user = await db.get("users:1")
```

### PostgreSQL
```bash
# In Shell tab
npm install pg

# Use external service like Supabase
DATABASE_URL=postgresql://...
```

## Monitoring

Replit provides:
- CPU/Memory graphs
- Request logs
- Error tracking
- Uptime monitoring

All accessible in the Deployments dashboard.

## Cost Optimization

- **Development**: Free
- **Production**: $7/month (Reserved VM)
- **Autoscale**: $0.000024/request + $0.024/GB

For most apps, $7/month covers everything.

## Migration from Vercel

```bash
# 1. Export from Vercel
vercel env pull .env

# 2. Push to GitHub
git push

# 3. Import to Replit
# Click "Import from GitHub"

# 4. Add env vars to Secrets

# 5. Deploy!
```

## Framework-Specific Tips

### Next.js
```json
// package.json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start"
  }
}
```

### SvelteKit
```javascript
// Use adapter-node
import adapter from '@sveltejs/adapter-node';

export default {
  kit: {
    adapter: adapter()
  }
};
```

## Troubleshooting

**Port issues:**
```javascript
const PORT = process.env.PORT || 3000
app.listen(PORT)
```

**Build failures:**
```bash
# Clear cache
rm -rf .next node_modules
npm install
```

**Memory issues:**
Upgrade to larger Reserved VM in deployment settings.

## Pro Tips

1. Use Replit's AI assistant for debugging
2. Enable "Always On" for production
3. Use GitHub integration for CI/CD
4. Leverage multiplayer for team coding
5. Set up monitoring alerts

## Conclusion

Replit offers the fastest path from code to production. Perfect for:
- MVPs and prototypes
- Small to medium production apps
- Teams wanting zero DevOps
- Learning and experimentation

**Setup time: 2 minutes**
**Monthly cost: $7**
**Saved vs Vercel: $833/month**
