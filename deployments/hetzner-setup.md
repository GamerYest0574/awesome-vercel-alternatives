# Hetzner + Coolify: The Ultimate Next.js Hosting Setup

**Deploy unlimited Next.js apps for €4.51/month** with performance that rivals $500/mo enterprise hosting.

## Why Hetzner is the Best Value

- **€4.51/mo** gets you 4GB RAM, 2 vCPU, 40GB NVMe SSD
- **20TB bandwidth** included (worth $8,000 on Vercel!)
- **German engineering** = reliable, fast, secure
- **EU/US locations** available
- **No vendor lock-in** - you own everything

## Complete Setup Guide (30 minutes)

### Step 1: Create Hetzner Server

1. Go to [console.hetzner.cloud](https://console.hetzner.cloud)
2. Create account (no credit card required for signup)
3. Create new project
4. Click "Add Server"

**Recommended Configuration:**
```
Location: Nuremberg (EU) or Ashburn (US)
Image: Ubuntu 22.04
Type: CX21 (€4.51/mo)
  - 2 vCPU (AMD EPYC)
  - 4 GB RAM
  - 40 GB NVMe SSD
  - 20 TB Traffic
Networking: IPv4 + IPv6
SSH Keys: Add your public key
```

### Step 2: Initial Server Setup

```bash
# SSH into your server
ssh root@your-server-ip

# Update system
apt update && apt upgrade -y

# Set timezone
timedatectl set-timezone Europe/Berlin  # or your timezone

# Enable firewall
ufw allow OpenSSH
ufw allow 80
ufw allow 443
ufw allow 8000  # Coolify UI
ufw --force enable

# Install essential packages
apt install -y curl wget git htop ncdu
```

### Step 3: Install Coolify

```bash
# One-command installation
curl -fsSL https://get.coolify.io | bash

# Installation takes ~5 minutes
# Note the credentials shown at the end!
```

### Step 4: Configure Coolify

1. Navigate to `http://your-server-ip:8000`
2. Login with credentials from installation
3. Complete initial setup:
   - Set admin email
   - Configure SMTP (optional)
   - Enable auto-updates

### Step 5: Add Custom Domain

```bash
# Point your domain to server IP
# A Record: @ -> your-server-ip
# A Record: * -> your-server-ip (for subdomains)
```

### Step 6: Deploy Your First Next.js App

**In Coolify Dashboard:**

1. Click "New Project"
2. Select "Public Repository" or connect GitHub
3. Choose "Next.js" as build pack
4. Configure:

```yaml
Build Command: npm run build
Start Command: npm start
Port: 3000
Health Check Path: /api/health
```

5. Add environment variables
6. Click "Deploy"

### Advanced Configuration

#### Optimize for Next.js

```bash
# Create swap file for build process
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab

# Increase file watchers
echo 'fs.inotify.max_user_watches=524288' >> /etc/sysctl.conf
sysctl -p
```

#### Setup PM2 for Direct Deployment (Alternative to Coolify)

```bash
# Install Node.js 20
curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
apt install -y nodejs

# Install PM2
npm install -g pm2

# Clone your app
git clone https://github.com/fadijob.official/your-app.git
cd your-app

# Install dependencies
npm install

# Build
npm run build

# Start with PM2
pm2 start npm --name "nextjs-app" -- start
pm2 save
pm2 startup
```

#### Configure Nginx as Reverse Proxy

```nginx
# /etc/nginx/sites-available/nextjs
server {
    listen 80;
    server_name yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

#### SSL with Certbot

```bash
# Install Certbot
apt install -y certbot python3-certbot-nginx

# Get SSL certificate
certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Auto-renewal is configured automatically
```

## Database Setup Options

### PostgreSQL

```bash
# Via Coolify
# 1. Add new service -> PostgreSQL
# 2. Auto-configured with your app

# Manual installation
apt install -y postgresql postgresql-contrib
sudo -u postgres createuser --interactive
sudo -u postgres createdb your_database
```

### Redis

```bash
# Via Coolify
# 1. Add new service -> Redis
# 2. Connect using provided credentials

# Manual installation
apt install -y redis-server
systemctl enable redis-server
```

## Performance Optimization

### 1. Enable Compression

```javascript
// next.config.js
module.exports = {
  compress: true,
  poweredByHeader: false,
  generateEtags: false,
}
```

### 2. Setup CDN (Cloudflare)

1. Add site to Cloudflare (free)
2. Update nameservers
3. Enable:
   - Auto Minify
   - Brotli compression
   - HTTP/3
   - Automatic HTTPS Rewrites

### 3. Optimize Images

```javascript
// next.config.js
module.exports = {
  images: {
    domains: ['yourdomain.com'],
    deviceSizes: [640, 750, 828, 1080, 1200],
    imageSizes: [16, 32, 48, 64, 96],
    formats: ['image/webp'],
  },
}
```

### 4. Server Tuning

```bash
# /etc/sysctl.conf
net.core.somaxconn = 65535
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_fin_timeout = 30
net.core.netdev_max_backlog = 5000
net.ipv4.tcp_max_syn_backlog = 8192

# Apply
sysctl -p
```

## Monitoring Setup

### 1. Uptime Monitoring

```bash
# Install Uptime Kuma via Coolify
# Or use external service like:
# - UptimeRobot (free)
# - Pingdom
# - StatusCake
```

### 2. Server Monitoring

```bash
# Install Netdata
wget -O /tmp/netdata-kickstart.sh https://my-netdata.io/kickstart.sh
sh /tmp/netdata-kickstart.sh

# Access at http://your-server-ip:19999
```

### 3. Application Monitoring

```javascript
// Install analytics
npm install @vercel/analytics

// app/layout.tsx
import { Analytics } from '@vercel/analytics/react'

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Analytics />
      </body>
    </html>
  )
}
```

## Backup Strategy

### Automated Backups

```bash
#!/bin/bash
# /root/backup.sh

# Backup database
pg_dump your_database > /backup/db_$(date +%Y%m%d).sql

# Backup application files
tar -czf /backup/app_$(date +%Y%m%d).tar.gz /var/www/your-app

# Sync to external storage (Backblaze B2)
rclone sync /backup b2:your-bucket

# Keep only last 7 days locally
find /backup -type f -mtime +7 -delete

# Add to crontab:
# 0 3 * * * /root/backup.sh
```

### Hetzner Snapshots

```bash
# Via Hetzner Console
# 1. Select your server
# 2. Click "Snapshots"
# 3. Create snapshot (€0.012/GB/month)
# Recommended: Weekly snapshots, keep 2
```

## Scaling Strategies

### Vertical Scaling

| Type | vCPU | RAM | Storage | Price | Use Case |
|------|------|-----|---------|-------|----------|
| CX11 | 1 | 2GB | 20GB | €3.29 | Dev/Test |
| **CX21** | 2 | 4GB | 40GB | €4.51 | **Recommended** |
| CX31 | 2 | 8GB | 80GB | €7.59 | Growing apps |
| CX41 | 4 | 16GB | 160GB | €13.57 | High traffic |
| CX51 | 8 | 32GB | 240GB | €26.29 | Enterprise |

### Horizontal Scaling with Load Balancer

```nginx
# /etc/nginx/nginx.conf
upstream nextjs_cluster {
    least_conn;
    server app1.local:3000;
    server app2.local:3000;
    server app3.local:3000;
}

server {
    listen 80;
    location / {
        proxy_pass http://nextjs_cluster;
    }
}
```

## Cost Breakdown

### Single Server Setup
- **Server**: €4.51/mo
- **Backup Snapshots**: ~€1/mo
- **Domain**: €10/year
- **Total**: ~€6/mo ($7)

### High-Availability Setup
- **2x CX21 Servers**: €9.02/mo
- **Load Balancer**: €5.39/mo
- **Backups**: €2/mo
- **Total**: ~€16/mo ($19)

### Enterprise Setup
- **3x CX41 Servers**: €40.71/mo
- **Load Balancer**: €5.39/mo
- **Managed Database**: €15/mo
- **Backups**: €5/mo
- **Total**: ~€66/mo ($75)

## Troubleshooting

### High Memory Usage

```bash
# Check memory
free -h
htop

# Clear cache
sync && echo 3 > /proc/sys/vm/drop_caches

# Check Node.js memory
pm2 monit
```

### Slow Build Times

```bash
# Increase Node memory
NODE_OPTIONS="--max-old-space-size=4096" npm run build

# Use swap file (already configured above)
swapon -s  # Check swap
```

### SSL Issues

```bash
# Test SSL
certbot renew --dry-run

# Force renewal
certbot renew --force-renewal

# Check certificate
openssl s_client -connect yourdomain.com:443
```

## Security Hardening

```bash
# Install fail2ban
apt install -y fail2ban

# Configure SSH
sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
sed -i 's/#Port 22/Port 2222/' /etc/ssh/sshd_config
systemctl restart sshd

# Auto updates
apt install -y unattended-upgrades
dpkg-reconfigure --priority=low unattended-upgrades
```

## Migration from Vercel

```bash
# 1. Export environment variables from Vercel
vercel env pull .env.production

# 2. Push to GitHub
git add .
git commit -m "Ready for Hetzner"
git push

# 3. In Coolify
# - Import GitHub repo
# - Paste environment variables
# - Deploy

# 4. Update DNS
# Change from Vercel's IP to Hetzner's IP
```

## Performance Comparison

| Metric | Vercel | Hetzner + Coolify |
|--------|--------|-------------------|
| **Monthly Cost** | $200+ | €4.51 |
| **Bandwidth** | 1TB | 20TB |
| **TTFB** | 45ms | 25ms |
| **Build Speed** | Fast | Fast |
| **Control** | Limited | Full |
| **Scaling** | Auto | Manual |

## Pro Tips

1. **Use Cloudflare** for CDN (free)
2. **Enable HTTP/2** in Nginx
3. **Compress assets** with Brotli
4. **Cache aggressively** with Redis
5. **Monitor everything** with Netdata
6. **Backup daily** to B2/S3
7. **Update regularly** with auto-updates
8. **Use PM2** for process management

## Common Commands

```bash
# Coolify
coolify logs
coolify restart
coolify update

# PM2
pm2 list
pm2 logs
pm2 restart all
pm2 monit

# System
systemctl status nginx
systemctl restart nginx
journalctl -u nginx -f

# Docker (if using Coolify)
docker ps
docker logs container-name
docker system prune -a
```

## Support Resources

- [Hetzner Docs](https://docs.hetzner.com)
- [Coolify Docs](https://coolify.io/docs)
- [Coolify Discord](https://discord.gg/coolify)
- [Hetzner Community](https://community.hetzner.com)

## Conclusion

For €4.51/month, you get:
- ✅ Unlimited Next.js apps
- ✅ 20TB bandwidth
- ✅ Full control
- ✅ No vendor lock-in
- ✅ Better performance than Vercel
- ✅ 95% cost reduction

**This is the way.**
