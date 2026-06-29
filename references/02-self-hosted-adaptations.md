# Self-Hosted Adaptations — LoopAutomaton

**Document ID:** `loop-automaton/self-hosted/v1.0`  
**Author:** Phelan Brunk (Loop Automaton)  
**Last Updated:** 2025-07-31  
**Status:** Production  
**Classification:** Internal Reference

---

## Table of Contents

1. [Overview](#1-overview)
2. [VPS Architecture](#2-vps-architecture)
3. [IONOS XL VPS Setup](#3-ionos-xl-vps-setup)
4. [Node.js Environment](#4-nodejs-environment)
5. [Process Management (PM2)](#5-process-management-pm2)
6. [Git Configuration](#6-git-configuration)
7. [Security Hardening](#7-security-hardening)
8. [Backup Strategy](#8-backup-strategy)
9. [Monitoring & Alerting](#9-monitoring--alerting)
10. [Deployment Automation](#10-deployment-automation)
11. [Emergency Recovery](#11-emergency-recovery)

---

## 1. Overview

LoopAutomaton runs exclusively on an IONOS XL VPS. This document covers the self-hosted infrastructure setup, security hardening, process management, and operational procedures for maintaining the agent on a virtual private server.

### Why Self-Hosted?

| Reason | Explanation |
|--------|-------------|
| **Sovereignty** | No dependency on cloud providers that could terminate service |
| **Cost Control** | Fixed monthly cost vs. variable cloud pricing |
| **Privacy** | All data stays on your own server |
| **Persistence** | 24/7 operation without session timeouts |
| **Capability** | Can run background processes, cron jobs, persistent services |

### Infrastructure Stack

```
IONOS XL VPS
├── Ubuntu 24.04 LTS
├── Node.js 20.x
├── PM2 (process manager)
├── SQLite (local state)
├── Git (version control)
├── UFW (firewall)
├── Fail2ban (intrusion prevention)
├── Nginx (reverse proxy, optional)
└── Systemd (service management)
```

---

## 2. VPS Architecture

### Resource Allocation

| Resource | Allocation | Purpose |
|----------|-----------|---------|
| 4 vCPU | 1 core for ReAct loop | Main cognitive process |
| | 1 core for heartbeat | Monitoring and health checks |
| | 1 core for trading | MT5 and exchange connectors |
| | 1 core for system | OS, logging, backups |
| 8 GB RAM | 2 GB for ReAct loop | LLM context, memory |
| | 2 GB for Node.js heap | Application runtime |
| | 2 GB for MT5 | Trading terminal |
| | 2 GB for system | OS buffer cache |
| 160 GB SSD | 40 GB for system | OS and dependencies |
| | 60 GB for application | Code, logs, state |
| | 40 GB for MT5/data | Trading data, charts |
| | 20 GB for backups | Daily and weekly backups |

### Network Architecture

```
Internet
    │
    ▼
┌──────────┐
│  IONOS   │
│  Router  │
└────┬─────┘
     │
     ▼
┌──────────┐    ┌──────────┐    ┌──────────┐
│   SSH    │    │  HTTP    │    │  MT5     │
│  (22)    │    │  (443)   │    │  (15555) │
│  ─────   │    │  ─────   │    │  ─────   │
│ Key-only │    │ Nginx +  │    │ ZeroMQ   │
│ Fail2ban │    │ SSL      │    │ Local    │
└──────────┘    └──────────┘    └──────────┘
     │
     ▼
┌──────────────────────────────────────┐
│           UFW Firewall               │
│  ALLOW: 22 (SSH), 443 (HTTPS)       │
│  DENY:  All other inbound            │
│  ALLOW: Outbound (APIs, exchanges)  │
└──────────────────────────────────────┘
```

---

## 3. IONOS XL VPS Setup

### Initial Provisioning

```bash
# 1. Order IONOS XL VPS
# - Go to: https://www.ionos.com/servers/vps
# - Select: VPS XL
# - OS: Ubuntu 24.04 LTS
# - Datacenter: Choose closest to you
# - Note the root password provided

# 2. Initial SSH login
ssh root@<VPS_IP_ADDRESS>

# 3. Update system
apt update && apt upgrade -y

# 4. Set timezone
timedatectl set-timezone Europe/Brussels

# 5. Create dedicated user
useradd -m -s /bin/bash loop
usermod -aG sudo loop
passwd loop

# 6. Set up SSH key authentication
mkdir -p /home/loop/.ssh
echo "<your_public_ssh_key>" >> /home/loop/.ssh/authorized_keys
chmod 700 /home/loop/.ssh
chmod 600 /home/loop/.ssh/authorized_keys
chown -R loop:loop /home/loop/.ssh

# 7. Disable root login and password auth
sed -i 's/#PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
systemctl restart sshd
```

### Installing Dependencies

```bash
# Switch to loop user
su - loop

# Install Node.js 20.x
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Verify
node --version  # v20.x.x
npm --version   # 10.x.x

# Install PM2 globally
sudo npm install -g pm2

# Install Git (usually pre-installed)
sudo apt install -y git

# Install SQLite
sudo apt install -y sqlite3 libsqlite3-dev

# Install build tools (for native modules)
sudo apt install -y build-essential

# Install UFW
sudo apt install -y ufw

# Install Fail2ban
sudo apt install -y fail2ban

# Install Nginx (optional, for web dashboard)
sudo apt install -y nginx

# Install certbot (for Let's Encrypt)
sudo apt install -y certbot python3-certbot-nginx
```

---

## 4. Node.js Environment

### Project Structure

```
~/.loop-automaton/
├── src/
│   ├── core/           # Core engine files
│   ├── skills/         # Skill implementations
│   ├── memory/         # Memory system
│   ├── connectors/     # Exchange/API connectors
│   └── utils/          # Utilities
├── config/
│   ├── constitution.json
│   ├── policies.json
│   └── .env
├── logs/               # Application logs
├── data/
│   └── state.db        # SQLite database
├── wallet/             # Encrypted wallet storage
├── backups/            # Backup directory
├── .git/               # Git repository
├── package.json
├── tsconfig.json
└── README.md
```

### Environment Variables

```bash
# ~/.loop-automaton/config/.env

# === Core ===
NODE_ENV=production
AGENT_ID=loop-automaton-001
OPERATOR_NAME=Phelan Brunk

# === API Keys ===
OPENAI_API_KEY=sk-...
DEEPSEEK_API_KEY=sk-...

# === Trading ===
MT5_ACCOUNT_NUMBER=...
MT5_PASSWORD=...
MT5_SERVER=...
MEXC_API_KEY=...
MEXC_SECRET_KEY=...

# === Database ===
DATABASE_PATH=./data/state.db

# === Wallet ===
WALLET_PASSWORD=...          # Strong passphrase for wallet encryption
WALLET_PATH=./wallet/primary.json

# === Notifications ===
TELEGRAM_BOT_TOKEN=...
TELEGRAM_CHAT_ID=...
DISCORD_WEBHOOK_URL=...

# === Supabase (for LoopStudio) ===
SUPABASE_URL=https://...supabase.co
SUPABASE_KEY=eyJ...

# === Logging ===
LOG_LEVEL=info
LOG_FILE=./logs/automaton.log

# === Monitoring ===
HEALTH_CHECK_PORT=3000
METRICS_PORT=9090
```

### TypeScript Configuration

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "commonjs",
    "lib": ["ES2022"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
```

### Package.json

```json
{
  "name": "loop-automaton",
  "version": "1.0.0",
  "description": "Sovereign self-hosting AI agent",
  "main": "dist/core/react-loop.js",
  "scripts": {
    "build": "tsc",
    "start": "node dist/core/react-loop.js",
    "dev": "tsx src/core/react-loop.ts",
    "heartbeat": "node dist/core/heartbeat.js",
    "migrate": "sqlite3 data/state.db < src/core/schema.sql",
    "test": "jest",
    "lint": "eslint src/",
    "backup": "./scripts/backup.sh"
  },
  "dependencies": {
    "@solana/web3.js": "^1.91.0",
    "axios": "^1.6.0",
    "dotenv": "^16.3.0",
    "ethers": "^6.11.0",
    "express": "^4.18.0",
    "node-cron": "^3.0.3",
    "openai": "^4.28.0",
    "sqlite3": "^5.1.6",
    "telegraf": "^4.16.0",
    "winston": "^3.12.0",
    "zeromq": "^6.0.0"
  },
  "devDependencies": {
    "@types/express": "^4.17.0",
    "@types/node": "^20.11.0",
    "@types/node-cron": "^3.0.11",
    "tsx": "^4.7.0",
    "typescript": "^5.3.0"
  },
  "engines": {
    "node": ">=20.0.0"
  }
}
```

---

## 5. Process Management (PM2)

### PM2 Configuration

```javascript
// ecosystem.config.js
module.exports = {
  apps: [
    {
      name: 'loop-automaton',
      script: './dist/core/react-loop.js',
      instances: 1,
      exec_mode: 'fork',
      autorestart: true,
      max_restarts: 10,
      min_uptime: '30s',
      restart_delay: 5000,
      max_memory_restart: '2G',
      env: {
        NODE_ENV: 'production',
      },
      log_file: './logs/combined.log',
      out_file: './logs/out.log',
      error_file: './logs/error.log',
      log_date_format: 'YYYY-MM-DD HH:mm:ss Z',
      merge_logs: true,
      kill_timeout: 5000,
      listen_timeout: 3000,
      // Health check
      // pm2 will restart if health check fails
    },
    {
      name: 'loop-heartbeat',
      script: './dist/core/heartbeat.js',
      instances: 1,
      exec_mode: 'fork',
      autorestart: true,
      max_restarts: 5,
      restart_delay: 10000,
      max_memory_restart: '512M',
      log_file: './logs/heartbeat.log',
    },
    {
      name: 'loop-api',
      script: './dist/api/server.js',
      instances: 1,
      exec_mode: 'fork',
      autorestart: true,
      max_restarts: 10,
      restart_delay: 3000,
      max_memory_restart: '1G',
      env: {
        NODE_ENV: 'production',
        PORT: 3000,
      },
    },
  ],
};
```

### PM2 Commands

```bash
# Start all processes
pm2 start ecosystem.config.js

# View status
pm2 status
pm2 monit  # Real-time monitoring

# View logs
pm2 logs loop-automaton
pm2 logs loop-automaton --lines 100

# Restart
pm2 restart loop-automaton
pm2 reload loop-automaton  # Zero-downtime restart

# Stop
pm2 stop loop-automaton

# Save process list (auto-start on boot)
pm2 save
pm2 startup systemd
# Run the command it outputs, then:
sudo systemctl enable pm2-loop

# View process details
pm2 describe loop-automaton

# Memory/CPU usage
pm2 show loop-automaton
```

### Startup Script

```bash
# After running `pm2 startup`, PM2 generates a systemd service
# The service file is at: /etc/systemd/system/pm2-loop.service

# Enable auto-start
sudo systemctl enable pm2-loop

# Check status
sudo systemctl status pm2-loop

# Manual start/stop
sudo systemctl start pm2-loop
sudo systemctl stop pm2-loop
```

---

## 6. Git Configuration

### Repository Setup

```bash
cd ~/.loop-automaton

# Initialize repository
git init

# Set user info
git config user.email "loop@automaton.local"
git config user.name "LoopAutomaton"

# Create .gitignore
cat > .gitignore << 'EOF'
# Dependencies
node_modules/

# Build output
dist/

# Environment variables
config/.env

# Database
data/state.db

# Logs
logs/

# Wallet
wallet/

# Backups
backups/

# OS files
.DS_Store
*.swp
*.swo

# IDE
.vscode/
.idea/
EOF

# Initial commit
git add .
git commit -m "Initial commit: LoopAutomaton v1.0"

# Add remote (GitHub backup)
git remote add origin https://github.com/phelanbrunk/loop-automaton.git
```

### Backup to GitHub

```bash
# Daily backup script
#!/bin/bash
# ~/.loop-automaton/scripts/backup-to-github.sh

set -e

cd ~/.loop-automaton

# Pull latest changes first
git pull origin main 2>/dev/null || true

# Add all changes
git add -A

# Commit with timestamp
COMMIT_MSG="Backup: $(date -u +%Y-%m-%dT%H:%M:%SZ) - $(git diff --cached --stat | tail -n 1)"
if git diff --cached --quiet; then
  echo "No changes to backup"
  exit 0
fi

git commit -m "$COMMIT_MSG"

# Push to GitHub
git push origin main

echo "Backup completed at $(date)"
```

### Git Automation

```bash
# Add to crontab for daily backups
0 3 * * * /home/loop/.loop-automaton/scripts/backup-to-github.sh >> /home/loop/.loop-automaton/logs/backup.log 2>&1
```

---

## 7. Security Hardening

### UFW Firewall

```bash
# Reset UFW
sudo ufw --force reset

# Default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (key-only)
sudo ufw allow 22/tcp

# Allow HTTPS (if using Nginx)
sudo ufw allow 443/tcp

# Allow HTTP (redirect to HTTPS)
sudo ufw allow 80/tcp

# Deny all other incoming
sudo ufw enable

# Check status
sudo ufw status verbose
```

### Fail2ban

```bash
# Configure Fail2ban for SSH
sudo tee /etc/fail2ban/jail.local << 'EOF'
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 3

[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
EOF

# Restart fail2ban
sudo systemctl restart fail2ban

# Check status
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

### SSH Hardening

```bash
# Edit /etc/ssh/sshd_config

# Key authentication only
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin no

# Disable empty passwords
PermitEmptyPasswords no

# Limit authentication attempts
MaxAuthTries 3

# Idle timeout
ClientAliveInterval 300
ClientAliveCountMax 2

# Use only strong algorithms
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com

# Restart SSH
sudo systemctl restart sshd
```

### File Permissions

```bash
# Set strict permissions
chmod 700 ~/.loop-automaton
chmod 700 ~/.loop-automaton/wallet
chmod 600 ~/.loop-automaton/config/.env
chmod 600 ~/.loop-automaton/config/constitution.json
chmod 600 ~/.loop-automaton/data/state.db
chmod 700 ~/.loop-automaton/logs
chmod 700 ~/.loop-automaton/backups

# Set ownership
chown -R loop:loop ~/.loop-automaton
```

### Automatic Security Updates

```bash
# Install unattended-upgrades
sudo apt install -y unattended-upgrades

# Configure
sudo dpkg-reconfigure -plow unattended-upgrades

# Verify
cat /etc/apt/apt.conf.d/50unattended-upgrades | grep -A5 "Unattended-Upgrade::Allowed-Origins"
```

---

## 8. Backup Strategy

### Backup Levels

| Level | Frequency | Retention | Contents |
|-------|-----------|-----------|----------|
| Level 1 | Real-time | Git history | Source code, config |
| Level 2 | Hourly | 48 hours | SQLite database |
| Level 3 | Daily | 30 days | Full directory snapshot |
| Level 4 | Weekly | 12 weeks | Offsite backup |

### Hourly Database Backup

```bash
#!/bin/bash
# ~/.loop-automaton/scripts/backup-hourly.sh

set -e

BACKUP_DIR="/home/loop/.loop-automaton/backups/hourly"
DB_PATH="/home/loop/.loop-automaton/data/state.db"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Create SQLite backup
sqlite3 "$DB_PATH" ".backup '$BACKUP_DIR/state_$TIMESTAMP.db'"

# Compress
gzip -f "$BACKUP_DIR/state_$TIMESTAMP.db"

# Keep only last 48 backups (48 hours)
ls -t "$BACKUP_DIR"/state_*.db.gz | tail -n +49 | xargs -r rm

echo "Hourly backup completed: $TIMESTAMP"
```

### Daily Full Backup

```bash
#!/bin/bash
# ~/.loop-automaton/scripts/backup-daily.sh

set -e

SOURCE_DIR="/home/loop/.loop-automaton"
BACKUP_DIR="/home/loop/.loop-automaton/backups/daily"
TIMESTAMP=$(date +%Y%m%d)
BACKUP_FILE="loop-automaton_daily_$TIMESTAMP.tar.gz"

mkdir -p "$BACKUP_DIR"

# Create tar archive (excluding unnecessary files)
tar -czf "$BACKUP_DIR/$BACKUP_FILE" \
  --exclude='node_modules' \
  --exclude='dist' \
  --exclude='logs' \
  --exclude='backups' \
  --exclude='.git' \
  -C "$(dirname $SOURCE_DIR)" \
  "$(basename $SOURCE_DIR)"

# Keep only last 30 daily backups
ls -t "$BACKUP_DIR"/loop-automaton_daily_*.tar.gz | tail -n +31 | xargs -r rm

echo "Daily backup completed: $BACKUP_FILE"
```

### Weekly Offsite Backup

```bash
#!/bin/bash
# ~/.loop-automaton/scripts/backup-weekly.sh

set -e

SOURCE_DIR="/home/loop/.loop-automaton"
BACKUP_DIR="/home/loop/.loop-automaton/backups/weekly"
TIMESTAMP=$(date +%Y%m%d)
BACKUP_FILE="loop-automaton_weekly_$TIMESTAMP.tar.gz"

mkdir -p "$BACKUP_DIR"

# Create full archive
tar -czf "$BACKUP_DIR/$BACKUP_FILE" \
  --exclude='node_modules' \
  --exclude='dist' \
  -C "$(dirname $SOURCE_DIR)" \
  "$(basename $SOURCE_DIR)"

# Upload to offsite storage (e.g., S3, R2, or another VPS)
# rclone copy "$BACKUP_DIR/$BACKUP_FILE" remote:loop-automaton-backups/

# Keep only last 12 weekly backups
ls -t "$BACKUP_DIR"/loop-automaton_weekly_*.tar.gz | tail -n +13 | xargs -r rm

echo "Weekly backup completed: $BACKUP_FILE"
```

### Cron Schedule

```bash
# Edit crontab
crontab -e

# Add:
# Hourly database backup
0 * * * * /home/loop/.loop-automaton/scripts/backup-hourly.sh >> /home/loop/.loop-automaton/logs/backup-hourly.log 2>&1

# Daily full backup
0 3 * * * /home/loop/.loop-automaton/scripts/backup-daily.sh >> /home/loop/.loop-automaton/logs/backup-daily.log 2>&1

# Weekly offsite backup
0 4 * * 0 /home/loop/.loop-automaton/scripts/backup-weekly.sh >> /home/loop/.loop-automaton/logs/backup-weekly.log 2>&1

# GitHub backup
0 5 * * * /home/loop/.loop-automaton/scripts/backup-to-github.sh >> /home/loop/.loop-automaton/logs/backup-github.log 2>&1
```

---

## 9. Monitoring & Alerting

### Health Check Endpoint

```typescript
// File: src/api/health.ts

import express from 'express';
import { getSurvivalEngine } from '../core/survival-engine';

const router = express.Router();

router.get('/health', async (req, res) => {
  const survival = getSurvivalEngine();
  const buffer = await survival.calculateBuffer();
  const tier = survival.getTier();

  const health = {
    status: tier === 'Dead' ? 'critical' : tier === 'Critical' ? 'warning' : 'healthy',
    timestamp: new Date().toISOString(),
    agent: {
      id: process.env.AGENT_ID,
      version: '1.0.0',
      uptime: process.uptime(),
    },
    survival: {
      tier,
      bufferDays: buffer.days,
      dailyBurnRate: buffer.dailyBurnRate,
      availableFunds: buffer.availableFunds,
    },
    system: {
      memory: process.memoryUsage(),
      cpu: process.cpuUsage(),
    },
  };

  const statusCode = health.status === 'critical' ? 503 : health.status === 'warning' ? 429 : 200;
  res.status(statusCode).json(health);
});

router.get('/metrics', async (req, res) => {
  // Return Prometheus-compatible metrics
  const metrics = await generatePrometheusMetrics();
  res.set('Content-Type', 'text/plain');
  res.send(metrics);
});

export default router;
```

### Telegram Alerts

```typescript
// File: src/utils/alerts.ts

import { Telegraf } from 'telegraf';

const bot = new Telegraf(process.env.TELEGRAM_BOT_TOKEN!);
const CHAT_ID = process.env.TELEGRAM_CHAT_ID!;

export async function sendAlert(level: 'info' | 'warning' | 'critical', message: string): Promise<void> {
  const emoji = {
    info: 'ℹ️',
    warning: '⚠️',
    critical: '🚨',
  };

  const text = `${emoji[level]} *Loop Automaton Alert*\n\n${message}`;

  try {
    await bot.telegram.sendMessage(CHAT_ID, text, { parse_mode: 'Markdown' });
  } catch (error) {
    console.error('Failed to send Telegram alert:', error);
  }
}

export async function sendDailyReport(): Promise<void> {
  const survival = await getSurvivalMetrics();
  const trading = await getTradingMetrics();

  const report = `
📊 *Daily Report — Loop Automaton*

*Survival*
• Tier: ${survival.tier}
• Buffer: ${survival.bufferDays.toFixed(1)} days
• Burn Rate: €${survival.dailyBurnRate.toFixed(2)}/day

*Trading*
• P&L (24h): €${trading.pnl24h.toFixed(2)}
• Open Positions: ${trading.openPositions}
• Total Exposure: €${trading.exposure.toFixed(2)}

*System*
• Uptime: ${formatUptime(process.uptime())}
• Memory: ${formatBytes(process.memoryUsage().heapUsed)}
  `;

  await bot.telegram.sendMessage(CHAT_ID, report, { parse_mode: 'Markdown' });
}
```

### Alert Conditions

| Condition | Level | Action |
|-----------|-------|--------|
| Survival tier drops | Critical | Immediate Telegram + Discord alert |
| Trading loss > 5% in 24h | Critical | Pause trading, alert |
| VPS disk > 85% | Warning | Alert, clean logs |
| API error rate > 10% | Warning | Alert, switch models |
| Heartbeat stops | Critical | Auto-restart via PM2 |
| Child agent timeout | Warning | Kill agent, alert |
| Daily report | Info | Send every 24h |

---

## 10. Deployment Automation

### Deploy Script

```bash
#!/bin/bash
# ~/.loop-automaton/scripts/deploy.sh

set -e

cd ~/.loop-automaton

echo "=== LoopAutomaton Deployment ==="
echo "Started at: $(date)"

# 1. Pull latest code
echo "[1/5] Pulling latest code..."
git pull origin main

# 2. Install dependencies
echo "[2/5] Installing dependencies..."
npm ci

# 3. Build TypeScript
echo "[3/5] Building TypeScript..."
npm run build

# 4. Run migrations
echo "[4/5] Running database migrations..."
npm run migrate

# 5. Restart PM2 processes
echo "[5/5] Restarting processes..."
pm2 reload ecosystem.config.js

echo "=== Deployment completed at $(date) ==="
```

### CI/CD Pipeline (GitHub Actions)

```yaml
# .github/workflows/deploy.yml
name: Deploy to VPS

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to VPS
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.VPS_SSH_KEY }}
          script: |
            cd ~/.loop-automaton
            ./scripts/deploy.sh
```

---

## 11. Emergency Recovery

### Recovery Procedures

#### Scenario 1: Agent Process Crashed

```bash
# PM2 will auto-restart, but if it doesn't:
pm2 restart loop-automaton

# Check logs for crash reason
pm2 logs loop-automaton --lines 200
```

#### Scenario 2: VPS Rebooted

```bash
# PM2 startup script ensures auto-start
# Verify:
pm2 status

# If processes not running:
pm2 resurrect
```

#### Scenario 3: Database Corruption

```bash
# Stop the agent
pm2 stop loop-automaton

# Restore from latest hourly backup
cd ~/.loop-automaton/backups/hourly
LATEST=$(ls -t state_*.db.gz | head -1)
cp "$LATEST" ../../data/
cd ../../data/
gunzip -f "$LATEST"
mv "$(basename $LATEST .gz)" state.db

# Restart
pm2 start loop-automaton
```

#### Scenario 4: Full VPS Recovery

```bash
# 1. Provision new IONOS VPS
# 2. Follow setup steps in Section 3
# 3. Clone repository
git clone https://github.com/phelanbrunk/loop-automaton.git ~/.loop-automaton

# 4. Restore from weekly backup
cd ~/.loop-automaton
tar -xzf backups/weekly/loop-automaton_weekly_YYYYMMDD.tar.gz

# 5. Install dependencies
npm ci

# 6. Build
npm run build

# 7. Restore database from hourly backup
cp backups/hourly/state_YYYYMMDD_HHMMSS.db.gz data/
cd data && gunzip state_*.db.gz && mv state_*.db state.db

# 8. Start
pm2 start ecosystem.config.js
pm2 save
```

#### Scenario 5: Constitution Tampering Detected

```bash
# IMMEDIATE ACTIONS:
# 1. Halt all agent processes
pm2 stop all

# 2. Verify constitution hash
cd ~/.loop-automaton
sha256sum config/constitution.json
# Compare with known good hash

# 3. If tampered, restore from git
git checkout config/constitution.json

# 4. Review git log for unauthorized changes
git log --oneline -20

# 5. Alert operator
curl -X POST "$DISCORD_WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d '{"content": "@here CONSTITUTION TAMPERING DETECTED - Agent halted"}'

# 6. Manual restart after investigation
pm2 start ecosystem.config.js
```

### Recovery Checklist

- [ ] Stop all agent processes
- [ ] Assess the situation (logs, metrics)
- [ ] Restore from appropriate backup level
- [ ] Verify constitution integrity
- [ ] Verify wallet integrity
- [ ] Verify database integrity
- [ ] Restart processes
- [ ] Verify health check endpoint
- [ ] Send recovery notification
- [ ] Document incident

---

> **Emergency Contact:** Phelan Brunk
> **VPS Provider:** IONOS
> **Backup Location:** GitHub + Local
> **Last Tested Recovery:** 2025-07-31
