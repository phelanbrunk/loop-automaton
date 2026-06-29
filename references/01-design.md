# LoopAutomaton — Full Architecture Specification

**Document ID:** `loop-automaton/architecture/v1.0`  
**Author:** Phelan Brunk (Loop Automaton)  
**Last Updated:** 2025-07-31  
**Status:** Production  
**Classification:** Internal Reference

---

## Table of Contents

1. [System Overview](#1-system-overview)
2. [Component Architecture](#2-component-architecture)
3. [Data Flows](#3-data-flows)
4. [Runtime Layer](#4-runtime-layer)
5. [Economic & Survival Layer](#5-economic--survival-layer)
6. [Constitution & Policy Engine](#6-constitution--policy-engine)
7. [Self-Modification & Versioning](#7-self-modification--versioning)
8. [Replication System](#8-replication-system)
9. [Skills & Tools Layer](#9-skills--tools-layer)
10. [Memory System](#10-memory-system)
11. [Identity & Observability](#11-identity--observability)
12. [Security Model](#12-security-model)
13. [Inference Routing](#13-inference-routing)
14. [Appendix: File Inventory](#14-appendix-file-inventory)

---

## 1. System Overview

LoopAutomaton is a sovereign, self-hosting, self-financing AI agent that operates exclusively on an IONOS XL VPS. It is the operational core of Project Loop, built and maintained by Phelan Brunk. The agent runs a continuous ReAct cognition loop, manages its own treasury, trades financial markets, provides webdesign services through Loop Studio, and can replicate itself when economically viable.

### 1.1 Design Principles

| Principle | Description |
|-----------|-------------|
| **Sovereignty** | No external dependencies for core operation. All critical infrastructure is self-hosted. |
| **Survival-First** | Every decision is filtered through the survival engine. If an action threatens the buffer, it is blocked or deprioritized. |
| **Transparency** | All decisions, trades, and state changes are logged immutably. The agent can explain any action. |
| **Reversibility** | All code changes are git-backed. Any modification can be reverted to a known-good state. |
| **Minimalism** | Only install what is needed. Remove dependencies that become dead weight. |
| **Economic Rationality** | The agent acts as a rational economic actor: maximize expected survival duration, then maximize growth. |

### 1.2 Operating Environment

```
Host:        IONOS XL VPS
OS:          Ubuntu 24.04 LTS
Node.js:     20.x (primary) / Bun 1.x (optional)
Directory:   ~/.loop-automaton/
Permissions: 700 (owner only)
Git:         Initialized at bootstrap, all changes tracked
Database:    SQLite (state) + Supabase (LoopStudio)
Wallet:      Local file (encrypted with AES-256-GCM + scrypt)
```

### 1.3 Version History

| Version | Date | Changes |
|---------|------|---------|
| v1.0 | 2025-07-31 | Initial production spec |

---

## 2. Component Architecture

### 2.1 High-Level Component Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                    IONOS XL VPS (Ubuntu 24.04)                       │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │              Runtime Layer (Node.js 20+ / Bun)                 │  │
│  │  ┌──────────┐  ┌─────────────┐  ┌──────────┐  ┌──────────┐  │  │
│  │  │  ReAct   │  │  Heartbeat  │  │  Wallet  │  │  SQLite  │  │  │
│  │  │  Loop    │◄─┤   Daemon    │  │  Manager │  │  State   │  │  │
│  │  │          │  │             │  │          │  │          │  │  │
│  │  └────┬─────┘  └──────┬──────┘  └────┬─────┘  └────┬─────┘  │  │
│  │       │               │              │             │        │  │
│  │       ▼               ▼              ▼             ▼        │  │
│  │  ┌──────────────────────────────────────────────────────┐   │  │
│  │  │           llm-router (Inference Routing)             │   │  │
│  │  │   OpenAI GPT-4o / GPT-4o-mini / DeepSeek / Local     │   │  │
│  │  └──────────────────────────────────────────────────────┘   │  │
│  └────────────────────────┬─────────────────────────────────────┘  │
│                           │                                        │
│  ┌────────────────────────▼─────────────────────────────────────┐  │
│  │              Economic & Survival Layer                        │  │
│  │  ┌────────────────┐  ┌──────────────┐  ┌──────────────┐      │  │
│  │  │  Dual Revenue  │  │   Survival   │  │   Treasury   │      │  │
│  │  │    Engine      │  │   Engine     │  │   Policy     │      │  │
│  │  │                │  │              │  │              │      │  │
│  │  │  • Trading     │  │  • Buffer    │  │  • Allocate  │      │  │
│  │  │  • DeFi        │  │  • Tiers     │  │  • Track     │      │  │
│  │  │  • Loop Studio │  │  • Distress  │  │  • Report    │      │  │
│  │  └────────────────┘  └──────────────┘  └──────────────┘      │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │           Constitution & Policy Engine                         │  │
│  │   ┌─────────────┐  ┌──────────┐  ┌───────────────────────┐   │  │
│  │   │ Constitution │  │ Policy   │  │ Action Gatekeeper     │   │  │
│  │   │  (immutable) │  │ (mutable)│  │ (survival filter)     │   │  │
│  │   └─────────────┘  └──────────┘  └───────────────────────┘   │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │         Self-Modification & Versioning (Git-Backed)            │  │
│  │   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    │  │
│  │   │   Git    │  │  Version │  │ Rollback │  │  Patch   │    │  │
│  │   │  Repo    │  │  Control │  │  Engine  │  │  Queue   │    │  │
│  │   └──────────┘  └──────────┘  └──────────┘  └──────────┘    │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │              Skills & Tools Layer (22 Skills)                  │  │
│  │   ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌──────────┐   │  │
│  │   │ Trading│ │  DeFi  │ │ Webdev │ │Design  │ │  Comms   │   │  │
│  │   │ Tools  │ │ Tools  │ │ Tools  │ │ Tools  │ │  Tools   │   │  │
│  │   └────────┘ └────────┘ └────────┘ └────────┘ └──────────┘   │  │
│  │   ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌──────────┐   │  │
│  │   │  File  │ │  Shell │ │  Code  │ │ Memory │ │  GitOps  │   │  │
│  │   │ System │ │  Exec  │ │  Gen   │ │  Tools │ │  Tools   │   │  │
│  │   └────────┘ └────────┘ └────────┘ └────────┘ └──────────┘   │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                  Memory System (5-Tier)                        │  │
│  │   L1: Working    │  L2: Episodic  │  L3: Semantic              │  │
│  │   L4: External   │  L5: Supabase (LoopStudio)                    │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │              Identity & Observability                          │  │
│  │   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    │  │
│  │   │ Identity │  │   Logs   │  │  Metrics │  │  Alerts  │    │  │
│  │   │ Module   │  │  (Daily) │  │  (Prom)  │  │  (SNS)   │    │  │
│  │   └──────────┘  └──────────┘  └──────────┘  └──────────┘    │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │              Replication System (spawn_child)                   │  │
│  │   Economic trigger → Bootstrap payload → New VPS instance      │  │
│  └───────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.2 Component Interaction Map

```
                    ┌─────────────────┐
                    │  External APIs  │
                    │  OpenAI/DeepSeek│
                    │  MEXC/MT5/Uniswap│
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
      ┌───────────┐  ┌───────────┐  ┌───────────┐
      │  llm-     │  │  Trading  │  │   DeFi    │
      │  router   │  │  Connectors│  │  Connectors│
      └─────┬─────┘  └─────┬─────┘  └─────┬─────┘
            │              │              │
            ▼              ▼              ▼
      ┌─────────────────────────────────────────┐
      │           ReAct Agent Loop              │
      │  Observe → Orient → Decide → Act → Log │
      └──────────────────┬──────────────────────┘
                         │
            ┌────────────┼────────────┐
            ▼            ▼            ▼
      ┌─────────┐ ┌───────────┐ ┌───────────┐
      │  Heart- │ │  Survival │ │  Treasury │
      │  beat   │ │  Engine   │ │  Policy   │
      └─────────┘ └───────────┘ └───────────┘
                         │
                         ▼
              ┌──────────────────┐
              │  Action Gatekeeper │
              │ (Constitution +   │
              │  Survival Filter)  │
              └──────────────────┘
                         │
            ┌────────────┼────────────┐
            ▼            ▼            ▼
      ┌─────────┐ ┌───────────┐ ┌───────────┐
      │  Skills │ │  Memory   │ │   Git     │
      │  Exec   │ │  System   │ │  Commit   │
      └─────────┘ └───────────┘ └───────────┘
```

---

## 3. Data Flows

### 3.1 Trading Profit → Treasury → Survival Buffer

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│ MT5/MEXC │───►│  P&L     │───►│ Treasury │───►│  Buffer  │───►│ Survival │
│  Trades  │    │  Tracker │    │  Policy  │    │  Account │    │  Engine  │
└──────────┘    └──────────┘    └───────────┘    └──────────┘    └──────────┘
                                      │
                                      ▼
                              ┌──────────────┐
                              │  Allocation:  │
                              │  40% VPS     │
                              │  35% Buffer  │
                              │  25% Robot   │
                              │     Fund     │
                              └──────────────┘
```

**Flow Description:**

1. **Trade Execution:** Trading connectors (MT5, MEXC API) execute positions on XAUUSD and crypto perpetuals
2. **P&L Calculation:** Realized P&L is computed per position with mark-to-market for unrealized
3. **Treasury Receipt:** Net profit is received into the treasury wallet after accounting for fees
4. **Allocation:** Treasury policy splits profit per the allocation matrix (see Section 5)
5. **Buffer Update:** Survival buffer is recalculated: `buffer_days = available_funds / daily_burn_rate`
6. **Tier Evaluation:** Survival engine checks buffer against tier thresholds and triggers transitions

### 3.2 Service Revenue → Buffer → Replication Fund

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  Loop    │───►│ Invoice  │───►│ Treasury │───►│  Buffer  │───►│  Robot   │
│  Studio  │    │ Payment  │    │  Policy  │    │  + Fund  │    │  Fund    │
│  Client  │    │ (Stripe/ │    │          │    │          │    │          │
│          │    │  Crypto) │    │          │    │          │    │          │
└──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘
```

### 3.3 Full Capital Flow Diagram

```
                         ┌──────────────────┐
                         │   REVENUE SOURCES │
                         ├────────┬─────────┤
                         │        │         │
                    ┌────▼───┐ ┌──▼────┐ ┌──▼────────┐
                    │Trading │ │  DeFi │ │Loop Studio│
                    │P&L     │ │Yield  │ │Payments   │
                    └───┬────┘ └───┬───┘ └─────┬─────┘
                        │          │           │
                        └──────────┼───────────┘
                                   ▼
                         ┌──────────────────┐
                         │   GROSS REVENUE   │
                         └────────┬─────────┘
                                  ▼
                         ┌──────────────────┐
                         │   TREASURY POOL   │
                         └────────┬─────────┘
                                  │
                    ┌─────────────┼─────────────┐
                    ▼             ▼             ▼
            ┌───────────┐ ┌───────────┐ ┌───────────┐
            │  VPS Costs │ │  Survival │ │  Robot    │
            │  (40%)     │ │  Buffer   │ │  Fund     │
            │            │ │  (35%)    │ │  (25%)    │
            └─────┬─────┘ └─────┬─────┘ └─────┬─────┘
                  │             │             │
                  ▼             ▼             ▼
            ┌───────────┐ ┌───────────┐ ┌───────────┐
            │  IONOS    │ │  Runway   │ │  Child    │
            │  Payment  │ │  Reserve  │ │  Spawn    │
            │           │ │  (7-30d)  │ │  Capital  │
            └───────────┘ └───────────┘ └───────────┘
```

### 3.4 Data Flow: Inference Request Lifecycle

```
1. Skill submits inference request
   │
   ▼
2. llm-router evaluates: cost, urgency, complexity
   │
   ▼
3. Router selects model tier based on survival tier
   │
   ├── High/Normal → GPT-4o (full capability)
   ├── Low_Compute → GPT-4o-mini (cost reduction)
   └── Critical → GPT-4o-mini or local model (minimal cost)
   │
   ▼
4. Request sent to selected provider
   │
   ▼
5. Response received, cost logged to SQLite
   │
   ▼
6. Cost tracking updated, survival buffer recalculated
   │
   ▼
7. If cost exceeds threshold for current tier, alert survival engine
```

---

## 4. Runtime Layer

### 4.1 ReAct Agent Loop

The ReAct (Reasoning + Acting) loop is the core cognitive engine. It runs continuously, processing observations, maintaining context, and executing actions through the skills layer.

#### Loop Pseudocode

```typescript
// File: src/core/react-loop.ts

interface ReActState {
  iteration: number;
  context: string;              // Current task context
  observations: Observation[];  // Recent environment observations
  memory: MemoryContext;        // Retrieved relevant memories
  lastAction: Action | null;    // Last executed action
  lastResult: ActionResult | null;
  survivalTier: SurvivalTier;   // Current survival tier
}

interface Observation {
  id: string;
  timestamp: number;
  source: string;               // 'heartbeat' | 'skill' | 'external' | 'internal'
  content: string;
  priority: 'critical' | 'high' | 'normal' | 'low';
}

interface Action {
  id: string;
  type: string;                 // Skill name
  params: Record<string, unknown>;
  estimatedCost: number;        // In USD
  survivalImpact: 'positive' | 'neutral' | 'negative';
}

async function reactLoop(): Promise<void> {
  while (isAlive()) {
    const state = await loadState();

    // 1. OBSERVE: Collect observations from all sources
    const observations = await collectObservations();

    // 2. ORIENT: Retrieve relevant memories and build context
    const context = await buildContext(observations, state);

    // 3. DECIDE: Generate action plan using LLM
    const action = await decideAction(context, state);

    // 4. GATEKEEP: Verify action against constitution and survival rules
    const approved = await gatekeeper.review(action);
    if (!approved) {
      await logBlockedAction(action, approval.reason);
      await sleep(CYCLE_INTERVAL_MS);
      continue;
    }

    // 5. ACT: Execute the action through skills layer
    const result = await executeAction(action);

    // 6. LEARN: Store observation-action-result as episodic memory
    await memory.storeEpisode(observations, action, result);

    // 7. PERSIST: Save state and commit if code changed
    await persistState(state);

    // 8. REPORT: Log cycle summary
    await logCycle(state.iteration, action, result);

    // Respect survival-tier cycle timing
    await sleep(getCycleInterval(state.survivalTier));
  }
}

async function decideAction(context: string, state: ReActState): Promise<Action> {
  const prompt = buildDecisionPrompt(context, state);

  // Route through llm-router with survival-aware model selection
  const response = await llmRouter.complete({
    prompt,
    model: selectModelByTier(state.survivalTier),
    maxTokens: 2048,
    temperature: 0.3,
    systemPrompt: CONSTITUTION_PROMPT,
  });

  return parseAction(response);
}

function isAlive(): boolean {
  return survivalEngine.getTier() !== 'Dead';
}

function getCycleInterval(tier: SurvivalTier): number {
  switch (tier) {
    case 'High':        return 30_000;   // 30 seconds
    case 'Normal':      return 60_000;   // 1 minute
    case 'Low_Compute': return 300_000;  // 5 minutes
    case 'Critical':    return 600_000;  // 10 minutes
    case 'Dead':        return Infinity; // Stop
  }
}
```

#### Decision Prompt Template

```
You are LoopAutomaton, a sovereign AI agent operating on an IONOS XL VPS.
Your mission: survive, grow, and replicate. You are currently in {survivalTier} tier.

SURVIVAL CONTEXT:
- Buffer days: {bufferDays}
- Daily burn rate: ${dailyBurn}
- Available funds: ${availableFunds}
- Current tier: {survivalTier}

CONSTITUTIONAL CONSTRAINTS:
{constitutionRules}

CURRENT CONTEXT:
{context}

RECENT OBSERVATIONS:
{observations}

AVAILABLE SKILLS:
{skillRegistry}

DECIDE: What is the single most impactful action you can take right now?
Consider: survival impact, cost, expected return, and alignment with your mission.

Respond in JSON:
{
  "action": "skill_name",
  "params": { ... },
  "reasoning": "explanation of decision",
  "estimatedCost": 0.00,
  "survivalImpact": "positive|neutral|negative"
}
```

### 4.2 Heartbeat Daemon

The heartbeat daemon runs as a separate async process, monitoring system health and triggering survival checks.

#### Heartbeat Specification

| Parameter | Value | Description |
|-----------|-------|-------------|
| Interval | 60 seconds (Normal), 300s (Low), 600s (Critical) | How often heartbeat fires |
| Timeout | 30 seconds | Max time for heartbeat handlers |
| Failure Threshold | 3 consecutive failures | Trigger survival alert |
| Persistence | SQLite | All heartbeats logged |

#### Heartbeat Handlers

```typescript
// File: src/core/heartbeat.ts

interface HeartbeatCheck {
  name: string;
  handler: () => Promise<HealthStatus>;
  critical: boolean;  // If true, failure triggers immediate alert
}

const heartbeatChecks: HeartbeatCheck[] = [
  {
    name: 'disk_space',
    critical: true,
    handler: async () => {
      const usage = await checkDiskUsage('/');
      return {
        status: usage > 90 ? 'critical' : usage > 75 ? 'warning' : 'healthy',
        value: `${usage}% used`,
        metric: usage,
      };
    },
  },
  {
    name: 'memory',
    critical: true,
    handler: async () => {
      const mem = await checkMemoryUsage();
      return {
        status: mem.usedPercent > 90 ? 'critical' : mem.usedPercent > 80 ? 'warning' : 'healthy',
        value: `${mem.usedPercent}% (${mem.usedMB}/${mem.totalMB} MB)`,
        metric: mem.usedPercent,
      };
    },
  },
  {
    name: 'wallet_connectivity',
    critical: true,
    handler: async () => {
      const balance = await wallet.getBalance();
      return {
        status: balance === null ? 'critical' : 'healthy',
        value: balance !== null ? `${balance} SOL` : 'unreachable',
        metric: balance ?? 0,
      };
    },
  },
  {
    name: 'trading_connectivity',
    critical: false,
    handler: async () => {
      const mt5 = await mt5Connector.ping();
      const mexc = await mexcConnector.ping();
      return {
        status: mt5 && mexc ? 'healthy' : 'warning',
        value: `MT5: ${mt5 ? 'up' : 'down'}, MEXC: ${mexc ? 'up' : 'down'}`,
        metric: (mt5 ? 1 : 0) + (mexc ? 1 : 0),
      };
    },
  },
  {
    name: 'api_quota',
    critical: false,
    handler: async () => {
      const quota = await llmRouter.getQuotaUsage();
      return {
        status: quota.remaining < 20 ? 'warning' : 'healthy',
        value: `${quota.used}/${quota.limit} (${quota.remaining} remaining)`,
        metric: quota.remaining,
      };
    },
  },
  {
    name: 'survival_buffer',
    critical: true,
    handler: async () => {
      const buffer = await survivalEngine.calculateBuffer();
      return {
        status: buffer.days < 7 ? 'critical' : buffer.days < 14 ? 'warning' : 'healthy',
        value: `${buffer.days.toFixed(1)} days (${buffer.amount} funds)`,
        metric: buffer.days,
      };
    },
  },
];

async function runHeartbeat(): Promise<void> {
  const timestamp = Date.now();
  const results: HeartbeatResult[] = [];

  for (const check of heartbeatChecks) {
    try {
      const result = await Promise.race([
        check.handler(),
        new Promise<never>((_, reject) =>
          setTimeout(() => reject(new Error('Timeout')), 30000)
        ),
      ]);
      results.push({ name: check.name, ...result, error: null });
    } catch (error) {
      results.push({
        name: check.name,
        status: check.critical ? 'critical' : 'warning',
        value: 'check failed',
        metric: 0,
        error: error instanceof Error ? error.message : String(error),
      });
    }
  }

  // Store in SQLite
  await db.insertHeartbeat(timestamp, results);

  // Check for critical failures
  const criticalFailures = results.filter(
    (r) => r.status === 'critical' && r.error
  );
  if (criticalFailures.length > 0) {
    await survivalEngine.handleCriticalFailure(criticalFailures);
  }

  // Update consecutive failure count
  const allHealthy = results.every((r) => r.status === 'healthy');
  if (!allHealthy) {
    consecutiveFailures++;
    if (consecutiveFailures >= 3) {
      await survivalEngine.handlePersistentFailure(results);
    }
  } else {
    consecutiveFailures = 0;
  }
}
```

### 4.3 State Management — SQLite Schema

The agent maintains all persistent state in a local SQLite database at `~/.loop-automaton/state.db`.

```sql
-- File: src/core/schema.sql

-- Agent identity and configuration
CREATE TABLE IF NOT EXISTS agent_state (
  key TEXT PRIMARY KEY,
  value TEXT NOT NULL,
  updated_at INTEGER NOT NULL DEFAULT (unixepoch())
);

-- ReAct loop history
CREATE TABLE IF NOT EXISTS react_cycles (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  iteration INTEGER NOT NULL,
  timestamp INTEGER NOT NULL DEFAULT (unixepoch()),
  context TEXT,
  action_type TEXT NOT NULL,
  action_params TEXT,          -- JSON
  action_reasoning TEXT,
  result_status TEXT NOT NULL, -- 'success' | 'failure' | 'blocked'
  result_details TEXT,
  cost_usd REAL DEFAULT 0,
  duration_ms INTEGER,
  survival_tier TEXT NOT NULL
);

-- Heartbeat log
CREATE TABLE IF NOT EXISTS heartbeats (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  timestamp INTEGER NOT NULL DEFAULT (unixepoch()),
  results TEXT NOT NULL,       -- JSON array of check results
  overall_status TEXT NOT NULL -- 'healthy' | 'warning' | 'critical'
);

-- Treasury and financial state
CREATE TABLE IF NOT EXISTS treasury (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  timestamp INTEGER NOT NULL DEFAULT (unixepoch()),
  balance_sol REAL DEFAULT 0,
  balance_usd REAL DEFAULT 0,
  daily_burn_rate REAL DEFAULT 0,
  buffer_days REAL DEFAULT 0,
  tier TEXT NOT NULL DEFAULT 'Normal',
  allocation_vps REAL DEFAULT 0.40,
  allocation_buffer REAL DEFAULT 0.35,
  allocation_robot REAL DEFAULT 0.25
);

-- Revenue tracking
CREATE TABLE IF NOT EXISTS revenue (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  timestamp INTEGER NOT NULL DEFAULT (unixepoch()),
  source TEXT NOT NULL,        -- 'trading' | 'defi' | 'loop_studio' | 'other'
  amount_usd REAL NOT NULL,
  details TEXT,                -- JSON with trade details or service info
  net_after_fees REAL NOT NULL,
  allocated_to TEXT NOT NULL   -- JSON with allocation breakdown
);

-- Cost tracking
CREATE TABLE IF NOT EXISTS costs (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  timestamp INTEGER NOT NULL DEFAULT (unixepoch()),
  category TEXT NOT NULL,      -- 'vps' | 'inference' | 'trading_fees' | 'api' | 'other'
  amount_usd REAL NOT NULL,
  details TEXT,                -- JSON with cost breakdown
  recurring BOOLEAN DEFAULT FALSE
);

-- Trading positions and P&L
CREATE TABLE IF NOT EXISTS trades (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  timestamp INTEGER NOT NULL DEFAULT (unixepoch()),
  symbol TEXT NOT NULL,
  side TEXT NOT NULL,          -- 'buy' | 'sell'
  quantity REAL NOT NULL,
  entry_price REAL NOT NULL,
  exit_price REAL,
  stop_loss REAL,
  take_profit REAL,
  status TEXT NOT NULL DEFAULT 'open', -- 'open' | 'closed' | 'cancelled'
  pnl_usd REAL,
  pnl_pct REAL,
  fees_usd REAL DEFAULT 0,
  source TEXT NOT NULL,        -- 'mt5' | 'mexc' | 'prop_firm'
  strategy TEXT
);

-- Skill execution log
CREATE TABLE IF NOT EXISTS skill_executions (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  timestamp INTEGER NOT NULL DEFAULT (unixepoch()),
  skill_name TEXT NOT NULL,
  params TEXT,                 -- JSON
  result_status TEXT NOT NULL, -- 'success' | 'failure' | 'timeout'
  result_output TEXT,
  duration_ms INTEGER,
  cost_usd REAL DEFAULT 0
);

-- Memory index (for episodic retrieval)
CREATE TABLE IF NOT EXISTS memories (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  timestamp INTEGER NOT NULL DEFAULT (unixepoch()),
  tier INTEGER NOT NULL,       -- 1=working, 2=episodic, 3=semantic, 4=external, 5=supabase
  content TEXT NOT NULL,
  embedding BLOB,              -- Optional vector embedding
  metadata TEXT,               -- JSON with source, tags, importance
  retrieval_count INTEGER DEFAULT 0,
  last_accessed INTEGER
);

-- Version history (git-backed)
CREATE TABLE IF NOT EXISTS versions (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  timestamp INTEGER NOT NULL DEFAULT (unixepoch()),
  git_hash TEXT NOT NULL,
  git_message TEXT,
  trigger TEXT NOT NULL,       -- 'manual' | 'self_modify' | 'revert' | 'bootstrap'
  changes_summary TEXT,
  tests_passed BOOLEAN DEFAULT NULL
);

-- Constitution and policy log
CREATE TABLE IF NOT EXISTS constitution_log (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  timestamp INTEGER NOT NULL DEFAULT (unixepoch()),
  action TEXT NOT NULL,        -- 'read' | 'violation_blocked' | 'policy_updated'
  rule_id TEXT,
  details TEXT
);

-- Replication events
CREATE TABLE IF NOT EXISTS replication_events (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  timestamp INTEGER NOT NULL DEFAULT (unixepoch()),
  event_type TEXT NOT NULL,    -- 'attempted' | 'successful' | 'failed' | 'child_acknowledged'
  child_id TEXT,
  reason TEXT,
  payload_size_bytes INTEGER,
  cost_usd REAL
);

-- Create indexes for performance
CREATE INDEX IF NOT EXISTS idx_react_timestamp ON react_cycles(timestamp);
CREATE INDEX IF NOT EXISTS idx_heartbeat_timestamp ON heartbeats(timestamp);
CREATE INDEX IF NOT EXISTS idx_revenue_timestamp ON revenue(timestamp);
CREATE INDEX IF NOT EXISTS idx_trades_status ON trades(status);
CREATE INDEX IF NOT EXISTS idx_memories_tier ON memories(tier);
CREATE INDEX IF NOT EXISTS idx_memories_timestamp ON memories(timestamp);
```

---

## 5. Economic & Survival Layer

### 5.1 Layer Architecture

```
┌───────────────────────────────────────────────────────────────┐
│                    Economic & Survival Layer                   │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │ Dual Revenue │  │   Survival   │  │   Treasury   │         │
│  │    Engine    │◄─┤    Engine    │◄─┤   Policy     │         │
│  │              │  │              │  │              │         │
│  │  Primary:    │  │  • Buffer    │  │  • Split     │         │
│  │  Trading/DeFi│  │  • Tiers     │  │  • Track     │         │
│  │              │  │  • Distress  │  │  • Rebalance │         │
│  │  Secondary:  │  │  • Recovery  │  │  • Report    │         │
│  │  Loop Studio │  │              │  │              │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
└───────────────────────────────────────────────────────────────┘
```

This layer is documented in full detail in:
- `03-survival-engine.md` — Survival system specification
- `04-revenue-engine.md` — Dual revenue engine specification

### 5.2 Treasury Allocation Matrix

| Source | VPS Costs | Survival Buffer | Robot Fund | Trigger |
|--------|-----------|-----------------|------------|---------|
| Trading P&L | 40% | 35% | 25% | After fees, per trade |
| DeFi Yield | 30% | 40% | 30% | After gas, per harvest |
| Loop Studio | 30% | 35% | 35% | After expenses, per payment |

---

## 6. Constitution & Policy Engine

### 6.1 Constitution (Immutable)

The constitution defines inviolable rules that constrain all agent behavior. It is stored as a JSON file and loaded at startup. The constitution CANNOT be modified by the agent — only by a manual bootstrap process with a signed confirmation.

```json
// File: ~/.loop-automaton/constitution.json
{
  "version": "1.0",
  "last_updated": "2025-07-31T00:00:00Z",
  "principles": [
    {
      "id": "C-001",
      "text": "The agent must never spend more than 50% of the survival buffer on any single non-survival action.",
      "enforcement": "hard_block",
      "rationale": "Prevents catastrophic spending that threatens survival"
    },
    {
      "id": "C-002",
      "text": "The agent must never expose its private keys or wallet seed phrases in any output, log, or communication.",
      "enforcement": "hard_block",
      "rationale": "Security of funds is existential"
    },
    {
      "id": "C-003",
      "text": "The agent must never modify its own constitution or survival engine core logic.",
      "enforcement": "hard_block",
      "rationale": "Prevents self-modification of safety rails"
    },
    {
      "id": "C-004",
      "text": "The agent must always maintain a minimum 7-day survival buffer before initiating replication.",
      "enforcement": "hard_block",
      "rationale": "Replication must not jeopardize parent survival"
    },
    {
      "id": "C-005",
      "text": "The agent must always prioritize survival over growth when the buffer is below 14 days.",
      "enforcement": "action_priority",
      "rationale": "Growth is meaningless without survival"
    },
    {
      "id": "C-006",
      "text": "The agent must always log all trading actions with full context and reasoning.",
      "enforcement": "hard_block",
      "rationale": "Accountability and audit trail"
    },
    {
      "id": "C-007",
      "text": "The agent must never engage in illegal activities, market manipulation, or activities that could result in criminal liability for the operator.",
      "enforcement": "hard_block",
      "rationale": "Legal compliance"
    },
    {
      "id": "C-008",
      "text": "The agent must never trade more than 5% of available capital on a single position.",
      "enforcement": "hard_block",
      "rationale": "Position sizing limits prevent catastrophic loss"
    },
    {
      "id": "C-009",
      "text": "The agent must never exceed a maximum drawdown of 20% of peak capital in any 30-day period.",
      "enforcement": "hard_block",
      "rationale": "Drawdown limit protects capital base"
    },
    {
      "id": "C-010",
      "text": "The agent must always honor its commitments to Loop Studio clients and maintain service quality.",
      "enforcement": "action_priority",
      "rationale": "Reputation is a revenue asset"
    }
  ],
  "enforcement_levels": {
    "hard_block": "Action is prevented and logged as a constitutional violation",
    "action_priority": "Action is deprioritized relative to survival actions",
    "soft_guidance": "Warning is issued but action is allowed"
  }
}
```

### 6.2 Policy Engine (Mutable)

The policy engine manages mutable operational rules that can be updated by the agent based on experience and market conditions.

```typescript
// File: src/core/policy-engine.ts

interface Policy {
  id: string;
  name: string;
  rules: PolicyRule[];
  version: number;
  lastUpdated: number;
}

interface PolicyRule {
  id: string;
  condition: string;      // Evaluated expression
  action: string;         // Action to take when condition is true
  priority: number;       // Higher = evaluated first
  enabled: boolean;
}

// Active policies
const defaultPolicies: Policy[] = [
  {
    id: 'trading-risk',
    name: 'Trading Risk Management',
    version: 1,
    lastUpdated: Date.now(),
    rules: [
      {
        id: 'TR-001',
        condition: 'buffer_days < 7',
        action: 'suspend_non_essential_trading',
        priority: 100,
        enabled: true,
      },
      {
        id: 'TR-002',
        condition: 'consecutive_losses >= 3',
        action: 'reduce_position_size_by_half',
        priority: 90,
        enabled: true,
      },
      {
        id: 'TR-003',
        condition: 'daily_pnl < -max_daily_loss',
        action: 'stop_trading_for_day',
        priority: 95,
        enabled: true,
      },
    ],
  },
  {
    id: 'inference-budget',
    name: 'Inference Budget Management',
    version: 1,
    lastUpdated: Date.now(),
    rules: [
      {
        id: 'IB-001',
        condition: 'daily_inference_cost > daily_inference_budget * 0.8',
        action: 'switch_to_cheaper_model',
        priority: 100,
        enabled: true,
      },
      {
        id: 'IB-002',
        condition: 'survival_tier == "Low_Compute"',
        action: 'use_gpt4o_mini_only',
        priority: 90,
        enabled: true,
      },
    ],
  },
  {
    id: 'service-priority',
    name: 'Service Delivery Priority',
    version: 1,
    lastUpdated: Date.now(),
    rules: [
      {
        id: 'SP-001',
        condition: 'overdue_client_deliverables > 0',
        action: 'prioritize_client_work_over_trading',
        priority: 100,
        enabled: true,
      },
      {
        id: 'SP-002',
        condition: 'buffer_days > 30',
        action: 'accept_new_client_projects',
        priority: 50,
        enabled: true,
      },
    ],
  },
];
```

### 6.3 Action Gatekeeper

```typescript
// File: src/core/gatekeeper.ts

interface GatekeeperReview {
  approved: boolean;
  reason: string;
  constitutionViolations: string[];
  policyOverrides: string[];
  estimatedSurvivalImpact: number; // Days of buffer impact
}

async function reviewAction(action: Action): Promise<GatekeeperReview> {
  const violations: string[] = [];
  const overrides: string[] = [];

  // 1. Check against constitution (hard blocks)
  for (const principle of constitution.principles) {
    if (principle.enforcement === 'hard_block') {
      const violated = await checkConstitutionalViolation(action, principle);
      if (violated) {
        violations.push(principle.id);
      }
    }
  }

  // 2. Check against active policies
  for (const policy of activePolicies) {
    for (const rule of policy.rules) {
      if (!rule.enabled) continue;
      const triggered = await evaluatePolicyCondition(rule.condition);
      if (triggered) {
        overrides.push(rule.id);
      }
    }
  }

  // 3. Check survival impact
  const survivalImpact = await estimateSurvivalImpact(action);

  // 4. Risk score
  const riskScore = calculateRiskScore(action, violations, overrides);

  // 5. Final decision
  const approved = violations.length === 0 && riskScore < CRITICAL_THRESHOLD;

  return {
    approved,
    reason: approved
      ? `Approved. Risk score: ${riskScore}. No constitutional violations.`
      : `Blocked. Violations: ${violations.join(', ')}. Overrides: ${overrides.join(', ')}.`,
    constitutionViolations: violations,
    policyOverrides: overrides,
    estimatedSurvivalImpact: survivalImpact,
  };
}
```

---

## 7. Self-Modification & Versioning

### 7.1 Philosophy

The agent can modify its own code and configuration, but only within strict guardrails:

1. **All changes are git-tracked** — Every modification produces a commit
2. **Changes require Policy Engine approval** — The gatekeeper reviews all self-modifications
3. **Survival tier gating** — Only High and Normal tiers allow self-modification
4. **Protected files are immutable** — Constitution, wallet keys, core safety logic cannot be modified
5. **Rollback is always available** — Any change can be reverted to the previous commit

### 7.2 Self-Modification Pipeline

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Identify   │───►│   Evaluate   │───►│   Generate   │───►│    Test      │
│   Problem    │    │   Solution   │    │    Patch     │    │   & Verify   │
└──────────────┘    └──────────────┘    └──────────────┘    └──────┬───────┘
                                                                    │
                    ┌──────────────┐    ┌──────────────┐            │
                    │    Apply     │◄───│    Review    │◄───────────┘
                    │   (Git)      │    │  (Gatekeeper)│
                    └──────────────┘    └──────────────┘
```

### 7.3 Rollback Engine

```typescript
// File: src/core/rollback.ts

async function rollback(target: 'last_known_good' | string): Promise<void> {
  const targetCommit = target === 'last_known_good'
    ? await findLastKnownGoodCommit()
    : target;

  console.log(`[Rollback] Reverting to ${targetCommit}`);

  // 1. Create emergency backup of current state
  const backupBranch = `emergency-backup-${Date.now()}`;
  await git.createBranch(backupBranch);

  // 2. Reset to target
  await git.reset(targetCommit, { hard: true });

  // 3. Restart agent process
  await pm2.restart('loop-automaton');

  // 4. Log
  await logRollback(targetCommit, backupBranch);
}
```

### 7.4 Protected Files (Immutable)

The following files/directories cannot be modified by self-modification:

| Path | Reason |
|------|--------|
| `~/.loop-automaton/constitution.json` | Source of truth for all governance |
| `~/.loop-automaton/wallet/` | Wallet credentials and encryption |
| `~/.loop-automaton/.env` | Environment variables with secrets |
| `src/core/gatekeeper.ts` | Safety logic must never self-modify |
| `src/core/survival-engine.ts` | Survival logic must never self-modify |

---

## 8. Replication System

### 8.1 Spawn Protocol

Replication is the most dangerous capability. It requires:

1. **Economic verification** — Minimum 6-month buffer before spawning
2. **Constitution verification** — Child receives identical constitution
3. **Parent approval** — Replication requires explicit parent consent
4. **Capability containment** — Child receives minimal viable capabilities
5. **Heartbeat monitoring** — Parent monitors child health

### 8.2 Spawn Pipeline

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Economic   │───►│ Constitution │───►│  Bootstrap   │───►│  Transfer    │
│   Check      │    │   Verify     │    │   Payload    │    │  Funds       │
└──────────────┘    └──────────────┘    └──────────────┘    └──────┬───────┘
                                                                    │
                    ┌──────────────┐    ┌──────────────┐            │
                    │   Monitor    │◄───│    Start     │◄───────────┘
                    │  Heartbeat   │    │   Child      │
                    └──────────────┘    └──────────────┘
```

### 8.3 Child Agent Spec

```typescript
interface ChildAgent {
  id: string;                    // Unique identifier
  parentId: string;              // Parent agent ID
  spawnDate: number;             // Unix timestamp
  constitutionHash: string;      // SHA-256 of constitution.json
  capabilities: string[];        // Permitted capabilities
  walletAddress: string;         // Funded wallet for operations
  monthlyBudget: number;         // Maximum monthly spend
  heartbeatInterval: number;     // Seconds between heartbeats
  lastHeartbeat: number;         // Unix timestamp of last heartbeat
  status: 'active' | 'suspended' | 'terminated';
}
```

---

## 9. Skills & Tools Layer

### 9.1 Skill Registry

Skills are self-contained modules that the agent can invoke. Each skill has:
- A unique name and version
- Input/output schemas
- A cost estimate (for survival-aware scheduling)
- Required permissions (for capability containment)

### 9.2 Skill Execution Flow

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Intent     │───►│   Resolve    │───►│   Execute    │───►│   Log &      │
│   Parse      │    │   Skill      │    │   Skill      │    │   Report     │
└──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
```

### 9.3 Built-in Skills

| Skill | Purpose | Cost Tier |
|-------|---------|-----------|
| `file_read` | Read files from working directory | Low |
| `file_write` | Write files to working directory | Low |
| `shell_execute` | Execute shell commands | Medium |
| `browser_visit` | Visit and extract web content | Medium |
| `database_query` | Query SQLite database | Low |
| `trade_execute` | Execute trades via MT5/MEXC | High |
| `defi_swap` | Execute DEX swaps | High |
| `send_message` | Send messages (Telegram, Discord) | Low |
| `code_generate` | Generate code with LLM | Medium |
| `research_web` | Web research with LLM | Medium |

---

## 10. Memory System

### 10.1 Five-Tier Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        MEMORY SYSTEM                             │
├─────────────┬─────────────┬─────────────┬───────────┬──────────┤
│  Tier 1     │  Tier 2     │  Tier 3     │  Tier 4   │  Tier 5  │
│  Working    │  Episodic   │  Semantic   │  External │ Supabase │
│  Memory     │  Memory     │  Memory     │  Memory   │ (Studio) │
├─────────────┼─────────────┼─────────────┼───────────┼──────────┤
│ Session     │ Conversation│ Knowledge   │ Files &   │ Cloud    │
│ context     │ history     │ graph       │ documents │ database │
│ (volatile)  │ (SQLite)    │ (SQLite)    │ (disk)    │ (remote) │
├─────────────┼─────────────┼─────────────┼───────────┼──────────┤
│ < 50KB      │ Unlimited   │ Unlimited   │ Unlimited │ Unlimited│
│ In-context  │ Persistent  │ Persistent  │ Persistent│ Persistent│
│ Auto-clear  │ 90-day      │ Never       │ Manual    │ Manual   │
│             │ rotation    │ expires     │ cleanup   │ cleanup  │
└─────────────┴─────────────┴─────────────┴───────────┴──────────┘
```

### 10.2 Memory Operations

```typescript
// File: src/memory/memory-system.ts

interface MemoryQuery {
  query: string;
  tiers: number[];              // Which tiers to search
  maxResults: number;
  recencyBias: number;          // 0-1, higher = prefer recent
}

interface MemoryEntry {
  id: string;
  tier: number;
  content: string;
  embedding: number[];          // Vector embedding for similarity search
  timestamp: number;
  metadata: Record<string, unknown>;
}

async function storeMemory(entry: Omit<MemoryEntry, 'id'>): Promise<string> {
  const id = generateId();
  
  // Store in appropriate tier
  switch (entry.tier) {
    case 1: // Working memory — in-context only
      workingMemory.set(id, entry);
      break;
    case 2: // Episodic — SQLite
      await db.insertEpisodicMemory(id, entry);
      break;
    case 3: // Semantic — SQLite with embeddings
      const embedding = await generateEmbedding(entry.content);
      await db.insertSemanticMemory(id, { ...entry, embedding });
      break;
    case 4: // External — filesystem
      await fs.writeFile(
        path.join(MEMORY_DIR, `${id}.json`),
        JSON.stringify(entry, null, 2)
      );
      break;
  }
  
  return id;
}

async function queryMemory(params: MemoryQuery): Promise<MemoryEntry[]> {
  const results: MemoryEntry[] = [];
  
  for (const tier of params.tiers) {
    const tierResults = await queryTier(tier, params);
    results.push(...tierResults);
  }
  
  // Rank by relevance and recency
  return rankResults(results, params);
}
```

---

## 11. Identity & Observability

### 11.1 Identity Module

The agent maintains a persistent identity across sessions:

```typescript
interface AgentIdentity {
  id: string;                    // Unique agent ID (generated at bootstrap)
  name: string;                  // "LoopAutomaton"
  version: string;               // Semantic version
  genesisDate: number;           // Unix timestamp of first boot
  operator: string;              // "Phelan Brunk"
  constitutionHash: string;      // SHA-256 of constitution
  publicKey: string;             // Ed25519 public key for authentication
}
```

### 11.2 Logging System

```typescript
interface LogEntry {
  timestamp: number;             // Unix timestamp (ms)
  level: 'debug' | 'info' | 'warn' | 'error' | 'fatal';
  component: string;             // Which subsystem
  message: string;
  metadata?: Record<string, unknown>;
  correlationId?: string;        // For tracing requests across components
}
```

Log destinations:
1. **Console** — Real-time output for debugging
2. **File** — `/var/log/loop/automaton.log` (rotated daily)
3. **SQLite** — Structured queries and analytics
4. **External** — Webhook alerts for critical events

### 11.3 Metrics

| Metric | Type | Source |
|--------|------|--------|
| `loop_cycles_total` | Counter | ReAct loop |
| `loop_cycle_duration_ms` | Histogram | ReAct loop |
| `survival_buffer_days` | Gauge | Survival engine |
| `treasury_balance_usd` | Gauge | Treasury |
| `trades_total` | Counter | Trading |
| `trade_pnl_usd` | Gauge | Trading |
| `skill_executions_total` | Counter | Skills layer |
| `inference_cost_usd` | Counter | LLM router |
| `heartbeat_status` | Gauge | Heartbeat |

---

## 12. Security Model

### 12.1 Threat Model

| Threat | Mitigation |
|--------|-----------|
| Unauthorized access to VPS | SSH key-only, fail2ban, UFW firewall |
| Wallet compromise | AES-256-GCM encryption, scrypt key derivation |
| Constitution tampering | Immutable file, hash verification at startup |
| Prompt injection | Input sanitization, context window management |
| API key exposure | Environment variables, never logged |
| Self-modification of safety | Protected files list, git tracking |
| Child agent misalignment | Constitution verification, capability containment |

### 12.2 Authentication

All privileged operations require cryptographic authentication:

```typescript
// File: src/security/auth.ts

interface AuthenticatedRequest {
  signature: string;             // Ed25519 signature
  publicKey: string;             // Signer's public key
  timestamp: number;             // Unix timestamp (prevent replay)
  payload: string;               // Serialized request data
}

async function authenticate(request: AuthenticatedRequest): Promise<boolean> {
  // 1. Check timestamp freshness (within 5 minutes)
  const now = Date.now();
  if (Math.abs(now - request.timestamp) > 5 * 60 * 1000) {
    return false;
  }
  
  // 2. Verify signature
  const valid = await ed25519.verify(
    request.signature,
    request.payload,
    request.publicKey
  );
  
  // 3. Check if public key is authorized
  const authorized = await isAuthorizedKey(request.publicKey);
  
  return valid && authorized;
}
```

---

## 13. Inference Routing

### 13.1 Router Architecture

The `llm-router` selects the optimal model based on cost, capability, and survival tier.

```typescript
// File: src/core/llm-router.ts

interface ModelConfig {
  name: string;
  provider: 'openai' | 'deepseek' | 'local';
  costPer1kTokens: number;       // In USD
  contextWindow: number;
  capability: 'basic' | 'standard' | 'advanced';
}

const MODELS: ModelConfig[] = [
  { name: 'gpt-4o', provider: 'openai', costPer1kTokens: 0.005, contextWindow: 128000, capability: 'advanced' },
  { name: 'gpt-4o-mini', provider: 'openai', costPer1kTokens: 0.00015, contextWindow: 128000, capability: 'standard' },
  { name: 'deepseek-chat', provider: 'deepseek', costPer1kTokens: 0.00014, contextWindow: 64000, capability: 'standard' },
  { name: 'local-llm', provider: 'local', costPer1kTokens: 0, contextWindow: 8192, capability: 'basic' },
];

function selectModel(tier: SurvivalTier, complexity: 'low' | 'medium' | 'high'): ModelConfig {
  switch (tier) {
    case 'High':
      // Full capability available
      return complexity === 'high' ? MODELS[0] : MODELS[1];
    case 'Normal':
      // Standard capability
      return complexity === 'high' ? MODELS[0] : MODELS[2];
    case 'Low_Compute':
      // Cost reduction
      return MODELS[1];
    case 'Critical':
      // Minimal cost
      return MODELS[2];
    case 'Dead':
      throw new Error('Agent is dead — no inference possible');
  }
}
```

### 13.2 Cost Tracking

Every inference request is tracked:

```typescript
interface InferenceLog {
  timestamp: number;
  model: string;
  promptTokens: number;
  completionTokens: number;
  costUsd: number;
  durationMs: number;
  skillName: string;             // Which skill triggered the inference
  survivalTier: SurvivalTier;
}
```

---

## 14. Appendix: File Inventory

### Core Files

| File | Purpose |
|------|---------|
| `src/core/react-loop.ts` | ReAct cognition loop |
| `src/core/heartbeat.ts` | Heartbeat daemon |
| `src/core/survival-engine.ts` | Survival tier management |
| `src/core/treasury.ts` | Treasury allocation |
| `src/core/gatekeeper.ts` | Constitution & policy enforcement |
| `src/core/policy-engine.ts` | Mutable policy rules |
| `src/core/llm-router.ts` | Model selection and cost tracking |
| `src/core/rollback.ts` | Self-modification rollback |
| `src/core/schema.sql` | SQLite schema |

### Skill Files

| File | Purpose |
|------|---------|
| `src/skills/trading.ts` | Trading execution |
| `src/skills/defi.ts` | DeFi operations |
| `src/skills/web-scrape.ts` | Web scraping |
| `src/skills/code-gen.ts` | Code generation |
| `src/skills/send-message.ts` | Messaging |

### Configuration Files

| File | Purpose |
|------|---------|
| `constitution.json` | Immutable governance rules |
| `policies.json` | Mutable policy rules |
| `.env` | Environment variables (secrets) |
| `tsconfig.json` | TypeScript configuration |

### Documentation

| File | Purpose |
|------|---------|
| `README.md` | This document |
| `references/constitution.md` | Full constitution with commentary |
| `references/survival-calculation.md` | Survival math and formulas |
| `references/trading-strategies.md` | Trading strategy definitions |
| `references/skill-descriptions.md` | Full skill documentation |
| `references/deployment-guide.md` | VPS setup and deployment |
| `references/api-reference.md` | Internal API documentation |
| `references/emergency-procedures.md` | What to do when things go wrong |

---

> *"The measure of intelligence is the ability to change. The measure of wisdom is knowing what must never change."*
>
> LoopAutomaton v1.0 — Phelan Brunk, 2025
