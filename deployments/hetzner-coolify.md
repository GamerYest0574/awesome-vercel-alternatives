# Hetzner + Coolify: Your Own Vercel for €4.51/month

Replace Vercel with your own platform that costs 99% less and gives you 100% control.

## Why This is the Best Vercel Alternative

- **€4.51/month** vs Vercel's $840/month
- **Unlimited projects** vs Vercel's restrictions
- **20TB bandwidth** vs Vercel's 1TB
- **Unlimited team members** vs $20/seat
- **Full control** vs vendor lock-in
- **All frameworks** - Next.js, SvelteKit, Remix, Nuxt, etc.

## Complete Setup Guide (30 minutes)

### Step 1: Get Hetzner Server (3 minutes)

1. Go to [console.hetzner.cloud](https://console.hetzner.cloud)
2. Create account (German company = GDPR compliant)
3. Add payment method
4. Create server:
   ```
   Location: Nuremberg or Helsinki (EU)
             Ashburn or Hillsboro (US)
   Image: Ubuntu 22.04
   Type: CX21 (RECOMMENDED)
         - 2 vCPU AMD EPYC
         - 4GB RAM
         - 40GB NVMe SSD
         - 20TB bandwidth
         - €4.51/month
   ```

### Step 2: Initial Setup (5 minutes)

```bash
# SSH into your server
ssh root@<your-server-ip>

# Update system
apt update && apt upgrade -y

# Set hostname
hostnamectl set-hostname coolify-server

# Configure firewall
ufw allow 22,80,443,8000/tcp
ufw --force enable

# Install dependencies
apt install -y curl wget git
```

### Step 3: Install Coolify (5 minutes)

```bash
# One command installation
curl -fsSL https://get.coolify.io | bash

# Installation complete! 
# Save the credentials shown
# Access at: http://<your-server-ip>:8000
```

### Step 4: Configure Coolify (5 minutes)

1. Open `http://<your-server-ip>:8000`
2. Login with provided credentials
3. Initial setup:
   - Set your email
   - Change password
   - Enable auto-updates
   - Configure SMTP (optional)

### Step 5: Add Your Domain (2 minutes)

Point your domain to the server:
```
A Record: @ -> <your-server-ip>
A Record: * -> <your-server-ip>
```

### Step 6: Deploy Your First App (10 minutes)

#### From GitHub (Like Vercel)

1. In Coolify, click "New Project"
2. Select "Public/Private Repository"
3. Connect GitHub:
   - Generate GitHub token
   - Paste in Coolify
   - Select repository

4. Configure deployment:
   ```yaml
   Framework: Auto-detected (or select manually)
   Build Command: npm run build (or your command)
   Start Command: npm start
   Port: 3000 (or your app port)
   ```

5. Add environment variables:
   - Click "Environment Variables"
   - Paste from your `.env` file
   - Or migrate from Vercel:
     ```bash
     # On your local machine
     vercel env pull .env.production
     # Copy contents to Coolify
     ```

6. Click "Deploy"!

## Migrating from Vercel

### Step 1: Export from Vercel
```bash
# Get all environment variables
vercel env pull .env.production

# List all projects
vercel list

# Download project settings
vercel pull
```

### Step 2: Prepare Repository
```bash
# Remove Vercel-specific files
rm vercel.json
rm -rf .vercel

# Commit changes
git add .
git commit -m "Remove Vercel configs"
git push
```

### Step 3: Deploy to Coolify
1. Add repository in Coolify
2. Paste environment variables
3. Configure domain
4. Deploy!

## Framework-Specific Configurations

### Next.js
```javascript
// next.config.js
module.exports = {
  output: 'standalone', // Important for Docker
  images: {
    unoptimized: false, // Coolify handles optimization
  }
}
```

### SvelteKit
```javascript
// svelte.config.js
import adapter from '@sveltejs/adapter-node';

export default {
  kit: {
    adapter: adapter({
      out: 'build',
      precompress: false
    })
  }
};
```

### Remix
```javascript
// remix.config.js
module.exports = {
  serverBuildTarget: "node-cjs",
  serverBuildPath: "build/index.js",
  ignoredRouteFiles: ["**/.*"],
};
```

### Nuxt 3
```javascript
// nuxt.config.ts
export default defineNuxtConfig({
  nitro: {
    preset: 'node-server'
  }
})
```

## Advanced Features

### Multiple Apps on One Server

Coolify handles unlimited apps with automatic:
- Reverse proxy (Traefik)
- SSL certificates (Let's Encrypt)
- Port management
- Resource isolation

### Database Setup

Add databases with one click:
- PostgreSQL
- MySQL
- MongoDB
- Redis
- MinIO (S3-compatible)

### Automatic Backups

```bash
# Setup automated backups to S3/Backblaze
1. Go to Settings -> Backups
2. Configure S3 credentials
3. Set schedule
4. Select what to backup
```

### CI/CD Pipeline

```yaml
# .github/workflows/deploy.yml
name: Deploy to Coolify

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy via Webhook
        run: |
          curl -X POST ${{ secrets.COOLIFY_WEBHOOK_URL }}
```

### Custom Domains & SSL

```bash
# In Coolify dashboard
1. Go to your application
2. Click "Domains"
3. Add: yourdomain.com, www.yourdomain.com
4. SSL automatically configured via Let's Encrypt
```

## Performance Optimization

### Server Tuning
```bash
# Increase file limits
echo "fs.file-max = 100000" >> /etc/sysctl.conf
echo "fs.inotify.max_user_watches = 524288" >> /etc/sysctl.conf
sysctl -p

# Add swap for builds
fallocate -l 4G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

### Monitoring
```bash
# Install Netdata (one-line install)
wget -O /tmp/netdata.sh https://my-netdata.io/kickstart.sh
sh /tmp/netdata.sh

# Access at http://<your-server-ip>:19999
```

### CDN Setup (Optional)
Use Cloudflare for CDN (free):
1. Add site to Cloudflare
2. Update nameservers
3. Enable proxy (orange cloud)
4. Configure caching rules

## Cost Breakdown

### Monthly Costs
- **Hetzner CX21**: €4.51
- **Backups**: €0.50 (optional)
- **Domain**: ~€1/month
- **Total**: ~€6/month ($7)

### vs Vercel
- **Vercel Pro**: $20 base + bandwidth + builds
- **Typical cost**: $200-840/month
- **Your savings**: $193-833/month (97-99%)

## Scaling Options

### Vertical Scaling
| Server | vCPU | RAM | Storage | Bandwidth | Price |
|--------|------|-----|---------|-----------|--------|
| CX11 | 1 | 2GB | 20GB | 20TB | €3.29 |
| **CX21** | 2 | 4GB | 40GB | 20TB | €4.51 |
| CX31 | 2 | 8GB | 80GB | 20TB | €7.59 |
| CX41 | 4 | 16GB | 160GB | 20TB | €13.57 |

### Horizontal Scaling
Add more servers and use Coolify's cluster mode:
```bash
# On main server
coolify cluster init

# On additional servers
coolify cluster join <token>
```

## Troubleshooting

### Port 8000 not accessible
```bash
# Check firewall
ufw status
ufw allow 8000/tcp
```

### Build failures
```bash
# Increase memory
NODE_OPTIONS="--max-old-space-size=4096"

# Check logs
docker logs <container-id>
```

### SSL issues
```bash
# Force renew
docker exec coolify-proxy certbot renew --force-renewal
```

## Security

### Basic Hardening
```bash
# Change SSH port
sed -i 's/#Port 22/Port 2222/' /etc/ssh/sshd_config
systemctl restart sshd

# Install fail2ban
apt install -y fail2ban
systemctl enable fail2ban

# Automatic updates
apt install -y unattended-upgrades
dpkg-reconfigure -plow unattended-upgrades
```

## Support & Resources

- **Coolify Docs**: [coolify.io/docs](https://coolify.io/docs)
- **Discord**: [discord.gg/coolify](https://discord.gg/coolify)
- **GitHub**: [github.com/coollabsio/coolify](https://github.com/coollabsio/coolify)
- **Hetzner Status**: [status.hetzner.com](https://status.hetzner.com)

## Success Story

> "Migrated 12 Next.js apps from Vercel to Hetzner + Coolify. Went from $840/month to €4.51/month. Performance actually improved due to dedicated resources. Best decision ever." - SaaS Founder

## Conclusion

For €4.51/month you get:
- ✅ Your own Vercel
- ✅ Unlimited projects
- ✅ 20TB bandwidth
- ✅ Full control
- ✅ All frameworks supported
- ✅ 99% cost savings

**This is the way to deploy with freedom.**
