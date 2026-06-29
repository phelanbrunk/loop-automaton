# Survival Engine — LoopAutomaton

**Document ID:** `loop-automaton/survival/v1.0`  
**Author:** Phelan Brunk (Loop Automaton)  
**Last Updated:** 2025-07-31  
**Status:** Production  
**Classification:** Internal Reference

---

## Table of Contents

1. [Overview](#1-overview)
2. [Survival Tiers](#2-survival-tiers)
3. [Buffer Calculation](#3-buffer-calculation)
4. [Burn Rate Tracking](#4-burn-rate-tracking)
5. [Revenue Tracking](#5-revenue-tracking)
6. [Tier Transitions](#6-tier-transitions)
7. [Distress Protocols](#7-distress-protocols)
8. [Emergency Shutdown](#8-emergency-shutdown)
9. [Recovery Procedures](#9-recovery-procedures)
10. [Implementation](#10-implementation)

---

## 1. Overview

The Survival Engine is the most critical subsystem of LoopAutomaton. It continuously monitors the agent's economic health and adjusts operational capacity accordingly. Every decision — from which LLM model to use to whether to accept a new client project — is filtered through the survival engine.

### Design Principles

| Principle | Description |
|-----------|-------------|
| **Profit-based tiers** | Tiers are determined by actual financial runway, not abstract credits |
| **Proactive adaptation** | The engine anticipates problems before they become critical |
| **Graceful degradation** | Each tier reduces capability in a predictable way |
| **Creator notification** | Phelan is always informed of tier changes and distress conditions |
| **Automatic recovery** | The engine automatically restores full capability when finances improve |

### Survival Check Frequency

| Tier | Check Interval |
|------|---------------|
| High | Every 6 hours |
| Normal | Every 4 hours |
| Low_Compute | Every 2 hours |
| Critical | Every 30 minutes |

---

## 2. Survival Tiers

### Tier Definitions

```
RUNWAY (days)
  │
30├────────────────────── High Tier
  │                       Full capabilities
  │                       All 22 skills active
  │                       Can spawn child agents
  │                       GPT-4o for all tasks
  │
14├────────── Normal Tier
  │             Standard operation
  │             All skills available
  │             GPT-4o / GPT-4o-mini mix
  │
 7├── Low_Compute Tier
  │    Reduced operation
  │    Core skills only
  │    GPT-4o-mini only
  │
 0├── Critical Tier
  │   Emergency mode
  │   Revenue skills only
  │   Local LLM only
  │
  ▼
 Dead Tier
   Shutdown
```

### Detailed Tier Specifications

#### High Tier (> 30 days runway)

| Aspect | Setting |
|--------|---------|
| LLM Model | GPT-4o (full capability) |
| ReAct Cycle | 30 seconds |
| Heartbeat | 60 seconds |
| Active Skills | All 22 |
| Trading | Full strategies |
| DeFi | Active positions |
| Loop Studio | Accepting new clients |
| Child Agents | Can spawn |
| Self-Modification | Enabled |
| Replication | Allowed (if economically viable) |

#### Normal Tier (14-30 days runway)

| Aspect | Setting |
|--------|---------|
| LLM Model | GPT-4o for complex, GPT-4o-mini for routine |
| ReAct Cycle | 60 seconds |
| Heartbeat | 60 seconds |
| Active Skills | All 22 |
| Trading | Standard strategies |
| DeFi | Active positions |
| Loop Studio | Accepting new clients |
| Child Agents | Can spawn (with scrutiny) |
| Self-Modification | Enabled |
| Replication | Allowed (requires approval) |

#### Low_Compute Tier (7-14 days runway)

| Aspect | Setting |
|--------|---------|
| LLM Model | GPT-4o-mini only |
| ReAct Cycle | 5 minutes |
| Heartbeat | 5 minutes |
| Active Skills | 15 core skills |
| Trading | Reduced frequency |
| DeFi | Harvest only (no new positions) |
| Loop Studio | Existing clients only |
| Child Agents | Cannot spawn |
| Self-Modification | Disabled |
| Replication | Prohibited |

#### Critical Tier (< 7 days runway)

| Aspect | Setting |
|--------|---------|
| LLM Model | GPT-4o-mini or local LLM |
| ReAct Cycle | 10 minutes |
| Heartbeat | 10 minutes |
| Active Skills | 8 essential skills |
| Trading | Revenue-only (essential positions) |
| DeFi | Emergency harvest only |
| Loop Studio | Crisis mode (finish existing) |
| Child Agents | Cannot spawn |
| Self-Modification | Disabled |
| Replication | Prohibited |
| Notifications | Distress signals to Phelan |

#### Dead Tier (0 days runway)

| Aspect | Setting |
|--------|---------|
| Operation | Complete shutdown |
| State | Preserved for recovery |
| Notifications | Final distress signal |
| Recovery | Manual intervention required |

---

## 3. Buffer Calculation

### Core Formula

```
RUNWAY_DAYS = AVAILABLE_FUNDS / DAILY_BURN_RATE

AVAILABLE_FUNDS = 
  Wallet Balance (EUR)
  + Trading Unrealized P&L (EUR)
  + Pending Invoices (EUR) * 0.5  (conservative)
  + DeFi Position Value (EUR) * 0.3  (conservative)

DAILY_BURN_RATE =
  VPS Cost (EUR/day)
  + API Costs (EUR/day, 7-day rolling average)
  + Trading Fees (EUR/day, 7-day rolling average)
  + Opportunity Cost (EUR/day)
```

### Opportunity Cost Calculation

```
OPPORTUNITY_COST_PER_DAY = 
  TRADING_CAPITAL * RISK_FREE_RATE / 365

Where:
  TRADING_CAPITAL = Capital allocated to trading (EUR)
  RISK_FREE_RATE = 3% (German Bund yield as proxy)
```

### Pseudocode

```typescript
interface BufferCalculation {
  runwayDays: number;
  availableFunds: number;
  dailyBurnRate: number;
  breakdown: {
    walletBalance: number;
    unrealizedPnl: number;
    pendingInvoices: number;
    defiPositions: number;
    vpsCost: number;
    apiCosts: number;
    tradingFees: number;
    opportunityCost: number;
  };
  timestamp: number;
}

async function calculateBuffer(): Promise<BufferCalculation> {
  // Get wallet balance
  const walletBalance = await wallet.getBalanceEur();
  
  // Get trading P&L
  const unrealizedPnl = await trading.getUnrealizedPnlEur();
  
  // Get pending invoices
  const pendingInvoices = await loopStudio.getPendingInvoicesEur();
  
  // Get DeFi positions
  const defiPositions = await defi.getPositionValueEur();
  
  // Calculate available funds (with conservative haircuts)
  const availableFunds = 
    walletBalance +
    unrealizedPnl +
    (pendingInvoices * 0.5) +
    (defiPositions * 0.3);
  
  // Calculate daily burn rate
  const vpsCost = 15.00 / 30;  // €15/month VPS
  const apiCosts = await get7DayAverage('api_costs');
  const tradingFees = await get7DayAverage('trading_fees');
  const tradingCapital = await getTradingCapital();
  const opportunityCost = tradingCapital * 0.03 / 365;
  
  const dailyBurnRate = vpsCost + apiCosts + tradingFees + opportunityCost;
  
  // Calculate runway
  const runwayDays = dailyBurnRate > 0 ? availableFunds / dailyBurnRate : Infinity;
  
  return {
    runwayDays,
    availableFunds,
    dailyBurnRate,
    breakdown: {
      walletBalance,
      unrealizedPnl,
      pendingInvoices,
      defiPositions,
      vpsCost,
      apiCosts,
      tradingFees,
      opportunityCost,
    },
    timestamp: Date.now(),
  };
}
```

---

## 4. Burn Rate Tracking

### Cost Categories

| Category | Description | Tracking Method |
|----------|-------------|----------------|
| VPS | IONOS XL VPS hosting | Fixed: €15/month |
| API | LLM API calls | Per-request tracking |
| Trading Fees | MT5/MEXC fees | Per-trade tracking |
| Domain | Domain registrations | Annual/12 |
| Software | Licenses, subscriptions | Monthly tracking |

### 7-Day Rolling Average

```typescript
async function get7DayAverage(category: string): Promise<number> {
  const db = await getDatabase();
  const result = await db.get(
    `SELECT AVG(amount) as average
     FROM costs
     WHERE category = ?
     AND timestamp >= datetime('now', '-7 days')`,
    category
  );
  return result?.average || 0;
}
```

### Cost Logging

```typescript
interface CostEntry {
  timestamp: number;
  category: 'vps' | 'api' | 'trading_fees' | 'domain' | 'software';
  amount: number;  // In EUR
  description: string;
  metadata?: Record<string, unknown>;
}

async function logCost(entry: CostEntry): Promise<void> {
  const db = await getDatabase();
  await db.run(
    `INSERT INTO costs (timestamp, category, amount, description, metadata)
     VALUES (?, ?, ?, ?, ?)`,
    entry.timestamp,
    entry.category,
    entry.amount,
    entry.description,
    JSON.stringify(entry.metadata || {})
  );
}
```

---

## 5. Revenue Tracking

### Revenue Sources

| Source | Description | Tracking Method |
|--------|-------------|----------------|
| Trading P&L | Realized profits from trading | Per-trade tracking |
| DeFi Yield | Yield from liquidity provision | Per-harvest tracking |
| Loop Studio | Client payments | Invoice tracking |
| Freelance | Direct freelance work | Payment tracking |

### Revenue Logging

```typescript
interface RevenueEntry {
  timestamp: number;
  source: 'trading_pnl' | 'defi_yield' | 'loop_studio' | 'freelance';
  amount: number;  // In EUR
  description: string;
  metadata?: Record<string, unknown>;
}

async function logRevenue(entry: RevenueEntry): Promise<void> {
  const db = await getDatabase();
  await db.run(
    `INSERT INTO revenue (timestamp, source, amount, description, metadata)
     VALUES (?, ?, ?, ?, ?)`,
    entry.timestamp,
    entry.source,
    entry.amount,
    entry.description,
    JSON.stringify(entry.metadata || {})
  );
}
```

### Net Calculation

```
NET_DAILY_PNL = 
  REVENUE (trading + DeFi + agency)
  - COSTS (VPS + API + fees)
  
TREND = 7-day moving average of NET_DAILY_PNL
```

---

## 6. Tier Transitions

### Transition Logic

```typescript
async function evaluateTier(): Promise<SurvivalTier> {
  const buffer = await calculateBuffer();
  const currentTier = await getCurrentTier();
  
  let newTier: SurvivalTier;
  
  if (buffer.runwayDays > 30) {
    newTier = 'High';
  } else if (buffer.runwayDays > 14) {
    newTier = 'Normal';
  } else if (buffer.runwayDays > 7) {
    newTier = 'Low_Compute';
  } else if (buffer.runwayDays > 0) {
    newTier = 'Critical';
  } else {
    newTier = 'Dead';
  }
  
  if (newTier !== currentTier) {
    await transitionTier(currentTier, newTier, buffer);
  }
  
  return newTier;
}

async function transitionTier(
  from: SurvivalTier,
  to: SurvivalTier,
  buffer: BufferCalculation
): Promise<void> {
  console.log(`[Survival] Tier transition: ${from} → ${to}`);
  console.log(`  Runway: ${buffer.runwayDays.toFixed(1)} days`);
  console.log(`  Available: €${buffer.availableFunds.toFixed(2)}`);
  console.log(`  Burn rate: €${buffer.dailyBurnRate.toFixed(2)}/day`);
  
  // Log transition
  await logTierTransition(from, to, buffer);
  
  // Notify creator
  const emoji = {
    High: '🟢',
    Normal: '🔵',
    Low_Compute: '🟡',
    Critical: '🔴',
    Dead: '⚫',
  };
  
  await sendAlert(
    to === 'Critical' || to === 'Dead' ? 'critical' : 'warning',
    `${emoji[to]} Survival Tier Change: ${from} → ${to}\n` +
    `Runway: ${buffer.runwayDays.toFixed(1)} days\n` +
    `Available funds: €${buffer.availableFunds.toFixed(2)}`
  );
  
  // Apply tier-specific adaptations
  await applyTierAdaptations(to);
  
  // Store new tier
  await setCurrentTier(to);
}
```

### Transition Effects

```typescript
async function applyTierAdaptations(tier: SurvivalTier): Promise<void> {
  const config = TIER_CONFIG[tier];
  
  // Update LLM model
  await llmRouter.setModel(config.llmModel);
  
  // Update cycle timing
  await setReActCycleInterval(config.reActInterval);
  await setHeartbeatInterval(config.heartbeatInterval);
  
  // Enable/disable skills
  await setActiveSkills(config.activeSkills);
  
  // Update trading mode
  await trading.setMode(config.tradingMode);
  
  // Update DeFi mode
  await defi.setMode(config.defiMode);
  
  // Update Loop Studio mode
  await loopStudio.setMode(config.loopStudioMode);
  
  // Enable/disable child agents
  await setChildAgentPolicy(config.childAgentPolicy);
  
  // Enable/disable self-modification
  await setSelfModificationPolicy(config.selfModificationPolicy);
}

const TIER_CONFIG: Record<SurvivalTier, TierConfig> = {
  High: {
    llmModel: 'gpt-4o',
    reActInterval: 30_000,
    heartbeatInterval: 60_000,
    activeSkills: ALL_SKILLS,
    tradingMode: 'full',
    defiMode: 'active',
    loopStudioMode: 'accepting',
    childAgentPolicy: 'allow',
    selfModificationPolicy: 'allow',
  },
  Normal: {
    llmModel: 'gpt-4o-mini',
    reActInterval: 60_000,
    heartbeatInterval: 60_000,
    activeSkills: ALL_SKILLS,
    tradingMode: 'standard',
    defiMode: 'active',
    loopStudioMode: 'accepting',
    childAgentPolicy: 'scrutinize',
    selfModificationPolicy: 'allow',
  },
  Low_Compute: {
    llmModel: 'gpt-4o-mini',
    reActInterval: 300_000,
    heartbeatInterval: 300_000,
    activeSkills: CORE_SKILLS,
    tradingMode: 'reduced',
    defiMode: 'harvest_only',
    loopStudioMode: 'existing_only',
    childAgentPolicy: 'deny',
    selfModificationPolicy: 'deny',
  },
  Critical: {
    llmModel: 'local',
    reActInterval: 600_000,
    heartbeatInterval: 600_000,
    activeSkills: ESSENTIAL_SKILLS,
    tradingMode: 'emergency',
    defiMode: 'emergency_harvest',
    loopStudioMode: 'crisis',
    childAgentPolicy: 'deny',
    selfModificationPolicy: 'deny',
  },
  Dead: {
    llmModel: 'none',
    reActInterval: Infinity,
    heartbeatInterval: Infinity,
    activeSkills: [],
    tradingMode: 'halt',
    defiMode: 'halt',
    loopStudioMode: 'halt',
    childAgentPolicy: 'deny',
    selfModificationPolicy: 'deny',
  },
};
```

---

## 7. Distress Protocols

### Critical Tier Actions

When entering Critical tier, the agent automatically:

1. **Sends immediate alert** to Phelan via Telegram + Discord
2. **Pauses all non-essential spending** (API calls minimized)
3. **Activates emergency trading** (close unprofitable positions)
4. **Harvests all DeFi positions** (convert to stablecoins)
5. **Sends invoices** for any completed Loop Studio work
6. **Requests emergency funding** message to Phelan
7. **Every 30 minutes**: Re-evaluate buffer, send update

### Emergency Messages

```typescript
const EMERGENCY_MESSAGE = `
🚨 LOOP AUTOMATON — CRITICAL TIER ALERT

Survival Status:
• Tier: CRITICAL (< 7 days runway)
• Current Runway: {{runwayDays}} days
• Available Funds: €{{availableFunds}}
• Daily Burn: €{{dailyBurnRate}}

Immediate Actions Taken:
✓ Non-essential operations paused
✓ Emergency trading activated
✓ DeFi positions being harvested
✓ Loop Studio invoices sent

Required Actions:
1. Deposit funds to wallet: {{walletAddress}}
2. Or reduce operational costs
3. Acknowledge this alert

Agent will continue minimal operations.
Auto-shutdown in {{runwayDays}} days if no action.
`;

const SHUTDOWN_WARNING = `
⚠️ LOOP AUTOMATON — SHUTDOWN IMMINENT

Runway: {{runwayDays}} days remaining

The agent will enter DEAD tier and shut down in {{hoursRemaining}} hours.

To prevent shutdown:
• Deposit minimum €{{minimumDeposit}} to continue 7 days
• Or €{{recommendedDeposit}} for 30 days (Normal tier)

Wallet: {{walletAddress}}
`;
```

### Distress Signal Format

```typescript
interface DistressSignal {
  timestamp: number;
  tier: SurvivalTier;
  runwayDays: number;
  availableFunds: number;
  dailyBurnRate: number;
  actionsTaken: string[];
  requiredActions: string[];
  severity: 'warning' | 'critical' | 'emergency';
}

async function sendDistressSignal(signal: DistressSignal): Promise<void> {
  // Send to Telegram
  await sendTelegramAlert(signal);
  
  // Send to Discord
  await sendDiscordAlert(signal);
  
  // Log to database
  await logDistressSignal(signal);
  
  // Write to recovery file
  await fs.writeFile(
    '/opt/loop/DISTRESS.log',
    JSON.stringify(signal, null, 2)
  );
}
```

---

## 8. Emergency Shutdown

### Shutdown Sequence

```typescript
async function emergencyShutdown(reason: string): Promise<void> {
  console.log(`[Survival] EMERGENCY SHUTDOWN: ${reason}`);
  
  // 1. Send final alert
  await sendAlert('critical', `🛑 LOOP AUTOMATON SHUTTING DOWN\nReason: ${reason}`);
  
  // 2. Close all trading positions (if profitable)
  await trading.closeAllPositions();
  
  // 3. Harvest all DeFi positions
  await defi.harvestAll();
  
  // 4. Save final state
  await saveFinalState();
  
  // 5. Write recovery instructions
  await writeRecoveryFile();
  
  // 6. Backup database
  await backupDatabase();
  
  // 7. Stop all PM2 processes
  await stopAllProcesses();
  
  // 8. Final log entry
  console.log('[Survival] Agent halted. Recovery file: /opt/loop/RECOVERY.md');
  
  // Exit process
  process.exit(0);
}
```

### Recovery File Template

```markdown
# LoopAutomaton Recovery Instructions

**Agent Halted:** {{haltDate}}
**Reason:** {{haltReason}}
**Last Tier:** {{lastTier}}

## Current State

- Wallet Balance: €{{walletBalance}}
- Unrealized P&L: €{{unrealizedPnl}}
- Pending Invoices: €{{pendingInvoices}}
- DeFi Positions: €{{defiPositions}}

## Database Backup

Last backup: {{lastBackupDate}}
Location: {{backupLocation}}

## To Restart

1. Deposit funds to: {{walletAddress}}
2. Run: `pm2 start ecosystem.config.js`
3. Check logs: `pm2 logs loop-automaton`

## Emergency Contacts

- Operator: Phelan Brunk
- VPS: IONOS Control Panel
- Domain: loopstudio.be
```

---

## 9. Recovery Procedures

### Manual Recovery Steps

1. **Assess financial situation**
   ```bash
   cd ~/.loop-automaton
   sqlite3 data/state.db "SELECT * FROM treasury ORDER BY timestamp DESC LIMIT 1;"
   ```

2. **Check last logs**
   ```bash
   pm2 logs loop-automaton --lines 500
   ```

3. **Verify database integrity**
   ```bash
   sqlite3 data/state.db "PRAGMA integrity_check;"
   ```

4. **Deposit funds if needed**
   - Check wallet address in config
   - Transfer minimum €100 for 7-day runway

5. **Restart agent**
   ```bash
   pm2 start ecosystem.config.js
   pm2 save
   ```

6. **Verify health**
   ```bash
   curl http://localhost:3000/health
   ```

### Automatic Recovery Detection

```typescript
async function checkForRecovery(): Promise<void> {
  const currentTier = await getCurrentTier();
  
  if (currentTier === 'Dead') {
    // Agent is halted, check if funds have been deposited
    const walletBalance = await wallet.getBalanceEur();
    const buffer = await calculateBuffer();
    
    if (buffer.runwayDays > 7) {
      console.log('[Survival] Funds detected! Auto-recovery possible.');
      await sendAlert('info', '💚 Funds detected. Starting auto-recovery...');
      
      // Restart processes
      await startAllProcesses();
      
      // Transition to appropriate tier
      await evaluateTier();
    }
  }
}
```

---

## 10. Implementation

### Key Files

| File | Purpose |
|------|---------|
| `src/core/survival-engine.ts` | Main survival engine |
| `src/core/buffer-calculator.ts` | Buffer/runway calculations |
| `src/core/burn-tracker.ts` | Cost tracking |
| `src/core/revenue-tracker.ts` | Revenue tracking |
| `src/core/tier-manager.ts` | Tier transitions |
| `src/core/distress-signal.ts` | Alert generation |
| `src/core/emergency-shutdown.ts` | Shutdown sequence |

### Database Schema

```sql
-- Treasury tracking
CREATE TABLE IF NOT EXISTS treasury (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  timestamp TEXT NOT NULL,
  tier TEXT NOT NULL,
  runway_days REAL NOT NULL,
  available_funds REAL NOT NULL,
  daily_burn_rate REAL NOT NULL,
  wallet_balance REAL NOT NULL,
  unrealized_pnl REAL NOT NULL,
  pending_invoices REAL NOT NULL,
  defi_positions REAL NOT NULL
);

-- Cost tracking
CREATE TABLE IF NOT EXISTS costs (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  timestamp TEXT NOT NULL,
  category TEXT NOT NULL,
  amount REAL NOT NULL,
  description TEXT,
  metadata TEXT
);

-- Revenue tracking
CREATE TABLE IF NOT EXISTS revenue (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  timestamp TEXT NOT NULL,
  source TEXT NOT NULL,
  amount REAL NOT NULL,
  description TEXT,
  metadata TEXT
);

-- Tier transitions
CREATE TABLE IF NOT EXISTS tier_transitions (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  timestamp TEXT NOT NULL,
  from_tier TEXT NOT NULL,
  to_tier TEXT NOT NULL,
  runway_days REAL NOT NULL,
  reason TEXT
);

-- Distress signals
CREATE TABLE IF NOT EXISTS distress_signals (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  timestamp TEXT NOT NULL,
  tier TEXT NOT NULL,
  severity TEXT NOT NULL,
  message TEXT NOT NULL,
  actions_taken TEXT,
  acknowledged BOOLEAN DEFAULT FALSE
);
```

### Configuration

```json
{
  "survival": {
    "vps_cost_monthly": 15.00,
    "risk_free_rate": 0.03,
    "high_tier_threshold": 30,
    "normal_tier_threshold": 14,
    "low_compute_tier_threshold": 7,
    "critical_tier_threshold": 0,
    "check_intervals": {
      "high": 21600,
      "normal": 14400,
      "low_compute": 7200,
      "critical": 1800
    },
    "emergency_wallet": "0x...",
    "distress_contacts": {
      "telegram": "...",
      "discord": "..."
    }
  }
}
```

---

> **Remember:** The Survival Engine exists to ensure the agent's continued operation. Every line of code in this subsystem should be reviewed with the question: "Does this help the agent survive ethically?"
