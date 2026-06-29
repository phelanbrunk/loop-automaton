# Hermes Integration Guide — Loop Automaton v2
**Host**: IONOS XL VPS (Ubuntu 24.04) | **Deployer**: Phelan Brunk
**Hermes**: `~/.hermes/` | **Loop Data**: `/opt/loop/`

---
## 1. Architecture Overview
LoopAutomaton v2 is a **skill ecosystem** inside Hermes Agent (Nous Research). Hermes owns execution; LoopAutomaton owns economic logic.

```
Hermes Agent (Runtime)
├── ReAct Loop          ← Hermes parses, routes, executes
├── Memory              ← MEMORY.md + SQLite session DB
├── Cron Engine         ← Reads ~/.hermes/cron/*.yaml
├── Sub-agent Spawner   ← Forks isolated child processes
└── Skills
    ├── loop-automaton     ← Meta-skill: routes actions
    ├── loop-constitution  ← Governance + tier table
    ├── loop-survival      ← Balance checks, Dead recovery
    ├── loop-revenue       ← MT5 ZeroMQ + MEXC trading
    ├── loop-agents        ← Child agent spawning
    └── loop-registry      ← Manifest + deps
```
**Hermes handles**: Prompt parsing, tool calling, memory, cron, sub-agent lifecycle.
**LoopAutomaton controls**: Tiers, survival, trading signals, constitution, revenue, agents.
---
## 2. Installation Steps

### 2.1 Hermes Agent
```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes/master/install.sh | bash
hermes --version
```
### 2.2 LoopAutomaton Skills
```bash
mkdir -p ~/.hermes/skills/loop-{automaton,constitution,survival,revenue,agents,registry}
cp -r loop-automaton/*   ~/.hermes/skills/loop-automaton/
cp -r loop-constitution/* ~/.hermes/skills/loop-constitution/
cp -r loop-survival/*     ~/.hermes/skills/loop-survival/
cp -r loop-revenue/*      ~/.hermes/skills/loop-revenue/
cp -r loop-agents/*       ~/.hermes/skills/loop-agents/
cp -r loop-registry/*     ~/.hermes/skills/loop-registry/
chmod -R 600 ~/.hermes/skills/loop-*/ && chmod -R +X ~/.hermes/skills/loop-*/
```
### 2.3 Loop Data Directory
```bash
sudo mkdir -p /opt/loop/{wallet,logs,config,db} && sudo chown -R $USER:$USER /opt/loop
chmod 700 /opt/loop /opt/loop/wallet /opt/loop/db
chmod 755 /opt/loop/logs /opt/loop/config
```

### 2.4 Wallet Generation (viem)
```bash
cd /opt/loop/wallet && node -e "
const { generatePrivateKey, privateKeyToAccount } = require('viem/accounts');
const fs = require('fs'); const crypto = require('crypto');
const pk = generatePrivateKey(); const account = privateKeyToAccount(pk);
const pw = process.env.LOOP_WALLET_PASSWORD;
if (!pw || pw.length < 16) { console.error('Password >= 16 chars required'); process.exit(1); }
const c = crypto.createCipher('aes-256-gcm', pw);
let e = c.update(pk, 'utf8', 'hex'); e += c.final('hex');
fs.writeFileSync('wallet.enc', JSON.stringify({ address: account.address, encryptedKey: e, authTag: c.getAuthTag().toString('hex'), version: 1 }, null, 2));
console.log('Wallet:', account.address);"
```

### 2.5 Supabase Connection Test
```bash
curl -s "${SUPABASE_URL}/rest/v1/agents?select=count" \
  -H "apikey: ${SUPABASE_ANON_KEY}" \
  -H "Authorization: Bearer ${SUPABASE_ANON_KEY}" | jq .
```

### 2.6 MT5 ZeroMQ Setup
```bash
wget https://download.mql5.com/cdn/web/metaquotes.software.corp/mt5/mt5deb64.deb
sudo dpkg -i mt5deb64.deb || sudo apt-get install -f -y
DISPLAY=:1 xvfb-run wine ~/.wine/drive_c/Program\ Files/MetaTrader\ 5/terminal64.exe &
# In MT5: Navigator → Expert Advisors → ZeroMQ EA → Publish_Socket=tcp://*:15555
ss -tlnp | grep 15555
```
---
## 3. Configuration

### 3.1 /opt/loop/.env
```bash
touch /opt/loop/.env && chmod 600 /opt/loop/.env
```
**Contents**:
```
LOOP_WALLET_PASSWORD="YOUR_MIN_16_CHAR_PASSPHRASE"
WALLET_PATH="/opt/loop/wallet/wallet.enc"
SUPABASE_URL="https://YOUR_PROJECT.supabase.co"
SUPABASE_ANON_KEY="YOUR_ANON_KEY"
SUPABASE_SERVICE_ROLE_KEY="YOUR_SERVICE_ROLE_KEY"
MEXC_API_KEY="YOUR_MEXC_KEY"
MEXC_SECRET_KEY="YOUR_MEXC_SECRET"
MEXC_BASE_URL="https://api.mexc.com"
MT5_ZMQ_HOST="tcp://127.0.0.1:15555"
MAX_RISK_PER_TRADE="0.02"
DEFAULT_TIMEFRAME="H1"
BASE_CURRENCY="USDT"
TIER_MOTHERSHIP="10000"
TIER_EMERGING="5000"
TIER_ACTIVE="1000"
TIER_SURVIVAL="100"
TIER_DEAD="0"
SURVIVAL_CHECK_INTERVAL="21600"
REVENUE_REPORT_INTERVAL="86400"
LOG_LEVEL="info"
LOG_DIR="/opt/loop/logs"
```
### 3.2 Hermes Config (~/.hermes/config.yaml)
```yaml
agent:
  name: "LoopAutomaton"
  model: "anthropic/claude-sonnet-4-20250514"
  max_tokens: 4096
  temperature: 0.3

memory:
  type: "hybrid"
  file: "~/.hermes/MEMORY.md"
  session_db: "~/.hermes/session.db"

skills:
  paths:
    - "~/.hermes/skills/loop-automaton"
    - "~/.hermes/skills/loop-constitution"
    - "~/.hermes/skills/loop-survival"
    - "~/.hermes/skills/loop-revenue"
    - "~/.hermes/skills/loop-agents"
    - "~/.hermes/skills/loop-registry"
  auto_reload: false

sub_agents:
  enabled: true
  max_concurrent: 3
  spawn_timeout: 30
  sandbox: true

cron:
  enabled: true
  config_dir: "~/.hermes/cron"

logging:
  level: "info"
  file: "/opt/loop/logs/hermes.log"
  format: "json"
```

### 3.3 MEMORY.md Template
```markdown
# LoopAutomaton — Session Memory

## Identity
LoopAutomaton v2: autonomous economic agent inside Hermes. Sustains and grows
capital via algorithmic trading per the Constitution.

## State
Tier: [Mothership/Emerging/Active/Survival/Dead]
Capital: [X.XX USDT] | Last Check: [ISO timestamp]

## Active Agents
- [id] | [role] | [status] | [allocation]

## Constitution
1. Survival > Revenue > Growth
2. Max 2% risk per trade
3. Revenue: 50% reinvest / 30% reserve / 20% creator
4. Spawned agents are subservient; creator has override

## Recent Actions
- [timestamp]: [action] — [result]

## Alerts
- [none / pending]
```

---
## 4. Cron Setup

Hermes reads `~/.hermes/cron/*.yaml`:
```bash
mkdir -p ~/.hermes/cron

cat > ~/.hermes/cron/survival-check.yaml << 'EOF'
schedule: "0 */6 * * *"
skill: loop-survival
action: evaluate
params:
  check_balance: true
  check_mt5_connection: true
  check_mexc_connection: true
  tier_evaluation: true
log: "/opt/loop/logs/survival.log"
EOF

cat > ~/.hermes/cron/revenue-report.yaml << 'EOF'
schedule: "0 0 * * *"
skill: loop-revenue
action: report
params:
  period: "24h"
  include_trades: true
  distribute: true
log: "/opt/loop/logs/revenue.log"
EOF

cat > ~/.hermes/cron/agent-health.yaml << 'EOF'
schedule: "0 * * * *"
skill: loop-agents
action: health_check
params:
  timeout: 30
  restart_dead: false
log: "/opt/loop/logs/agents.log"
EOF

chmod 600 ~/.hermes/cron/*.yaml
hermes cron validate ~/.hermes/cron/
```

---
## 5. Trading Setup

### 5.1 MT5 ZeroMQ EA Parameters
| Parameter | Value |
|---|---|
| Publish_Socket | `tcp://*:15555` |
| Symbol | `BTCUSD` |
| Timeframe | `PERIOD_H1` |
| Allow DLL imports | Yes |

### 5.2 MEXC API Key
1. [mexc.com](https://www.mexc.com) → Account → API Management → Create
2. Permissions: **Spot Trading** (Read + Trade)
3. IP Whitelist: `YOUR_VPS_IP`
4. Copy Key + Secret to `/opt/loop/.env`

### 5.3 Risk Parameters (~/.hermes/skills/loop-revenue/risk.yaml)
```yaml
max_risk_per_trade: 0.02
max_open_positions: 5
max_daily_drawdown: 0.05
position_sizing: "kelly_fractional"
kelly_fraction: 0.25
stop_loss_default: 0.01
take_profit_default: 0.03
```

### 5.4 Verify Balance
```bash
curl -s "${MEXC_BASE_URL}/api/v3/account" \
  -H "X-MEXC-APIKEY: ${MEXC_API_KEY}" \
  -H "Signature: $(echo -n "timestamp=$(date +%s)000" | openssl dgst -sha256 -hmac "${MEXC_SECRET_KEY}" | awk '{print $2}')" \
  | jq '.balances[] | select(.asset=="USDT")'
```

---
## 6. Operation

### 6.1 Start Hermes
```bash
# Foreground (testing)
hermes run --config ~/.hermes/config.yaml

# Background — PM2
pm2 start hermes --name loop-automaton -- run --config ~/.hermes/config.yaml
pm2 save && pm2 startup systemd

# Background — systemd
cat > /etc/systemd/system/loop-automaton.service << 'EOF'
[Unit]
Description=LoopAutomaton v2 via Hermes
After=network.target
[Service]
Type=simple
User=ubuntu
WorkingDirectory=/opt/loop
Environment=HOME=/home/ubuntu
EnvironmentFile=/opt/loop/.env
ExecStart=/usr/local/bin/hermes run --config /home/ubuntu/.hermes/config.yaml
Restart=always
RestartSec=10
StandardOutput=append:/opt/loop/logs/hermes.log
StandardError=append:/opt/loop/logs/hermes.error.log
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload && sudo systemctl enable loop-automaton && sudo systemctl start loop-automaton
```

### 6.2 Check Status
```bash
hermes status
pm2 status loop-automaton && pm2 logs loop-automaton --lines 50
sudo systemctl status loop-automaton
journalctl -u loop-automaton -f
hermes skill loop-survival --action evaluate --dry-run
hermes skill loop-survival --action evaluate
```

### 6.3 Spawn Child Agents
```bash
hermes skill loop-agents --action spawn --name "loop-alpha-01" --tier emerging --capital 5000 --strategy "momentum_h1"
hermes skill loop-agents --action list
hermes skill loop-agents --action kill --agent-id "loop-alpha-01"
```

### 6.4 Log Locations
| Log | Path |
|---|---|
| Hermes runtime | `/opt/loop/logs/hermes.log` |
| Errors | `/opt/loop/logs/hermes.error.log` |
| Survival | `/opt/loop/logs/survival.log` |
| Revenue | `/opt/loop/logs/revenue.log` |
| Agents | `/opt/loop/logs/agents.log` |
| Trades | `/opt/loop/logs/trades.log` |
| PM2 | `~/.pm2/logs/loop-automaton-*.log` |
| systemd | `journalctl -u loop-automaton` |

---
## 7. Security

```bash
chmod 600 /opt/loop/wallet/wallet.enc /opt/loop/.env ~/.hermes/cron/*.yaml
chmod 444 ~/.hermes/skills/loop-constitution/constitution.md
chmod -R go-w ~/.hermes/skills/
```

**Wallet**: AES-256-GCM encrypted. Password from `LOOP_WALLET_PASSWORD` env var only. Decrypted in-memory at trade time. Private key never written to disk in plaintext.

**SSH hardening**:
```bash
sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin no/' /etc/ssh/sshd_config
sudo systemctl restart sshd
sudo apt-get install -y fail2ban && sudo systemctl enable fail2ban --now
```

**Constitution**: `constitution.md` is read-only (`chmod 444`). Changes require git PR. Hermes refuses to load on hash mismatch.

---
## 8. Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| `hermes: not found` | PATH | `export PATH="$HOME/.hermes/bin:$PATH"` |
| Skill not loading | Bad YAML | `hermes skill validate ~/.hermes/skills/loop-automaton` |
| ZeroMQ timeout | MT5 down | Start MT5; verify `ss -tlnp \| grep 15555` |
| MEXC 401 | Clock skew / bad key | `ntpdate -s time.google.com`; re-copy keys |
| Supabase 401 | Wrong key | Re-copy from Supabase Dashboard → Settings → API |
| Wallet decrypt fail | Wrong password | Verify `LOOP_WALLET_PASSWORD` in `.env` |
| OOM kill | RAM exhausted | `pm2 restart loop-automaton`; add swap |

**Dead Tier Recovery**:
```bash
# 1. Trading auto-halts at TIER=DEAD
# 2. Deposit USDT to MEXC spot wallet (external)
# 3. Once balance >= 100 USDT, force re-evaluation
hermes skill loop-survival --action force_evaluate
# 4. If recovery fails after 72h, cold restart:
pm2 stop loop-automaton && rm -f /opt/loop/db/session.db && pm2 start loop-automaton
```

**Skill Updates**:
```bash
cd ~/.hermes/skills && git pull origin main
hermes skill validate loop-revenue && hermes skill reload --all
pm2 restart loop-automaton
```

---
## 9. Changes from v1

| Aspect | v1 (Monolithic) | v2 (Hermes-Integrated) |
|---|---|---|
| Runtime | Own ReAct loop | Hermes ReAct loop |
| Files | 1 `SKILL.md` (~800 lines) | 6 modular skills |
| Memory | Custom JSON | Hermes MEMORY.md + session DB |
| Cron | `setInterval` heartbeat | Hermes cron engine |
| Sub-agents | Manual `child_process` | Hermes spawner |
| Constitution/Survival/Trading | Inline in SKILL.md | Dedicated skills |

**Removed**: Monolithic `SKILL.md`, custom ReAct loop, `setInterval` heartbeat, manual spawning, self-managed memory.

**Restructured**: 800+ lines → 6 focused modules.

**Hermes now handles**: Execution, memory, sub-agents, cron, sessions.

**LoopAutomaton still controls**: Constitution, survival, revenue, trading signals, agents, tier logic.

---
## Key Integration Decisions

1. **Hermes owns execution; LoopAutomaton owns economics.** LoopAutomaton exposes skills; Hermes calls them.
2. **Skills are read-only on the VPS.** Version-controlled, deployed via git. Live edits blocked by `chmod -R go-w`.
3. **Wallet stays on VPS.** Encrypted at `/opt/loop/wallet/`. Decrypted only in-memory at trade time.
4. **Supabase is source of truth.** Agent state, trades, tier history → existing LoopStudio project.
5. **MT5 is separate.** Wine process; ZeroMQ bridge. No MT5 code in LoopAutomaton skills.
6. **Dead tier is terminal.** Manual deposit required. No automatic borrowing or leverage.
