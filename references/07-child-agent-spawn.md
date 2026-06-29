# Child Agent Spawn Protocol — LoopAutomaton

**Document ID:** `loop-automaton/spawn/v1.0`  
**Author:** Phelan Brunk (Loop Automaton)  
**Last Updated:** 2025-07-31  
**Status:** Production  
**Classification:** Internal Reference

---

## Table of Contents

1. [Overview](#1-overview)
2. [Spawn Authorization](#2-spawn-authorization)
3. [Agent Types](#3-agent-types)
4. [Spawn Process](#4-spawn-process)
5. [Constitution Inheritance](#5-constitution-inheritance)
6. [Resource Allocation](#6-resource-allocation)
7. [Monitoring & Control](#7-monitoring--control)
8. [Termination](#8-termination)
9. [Implementation](#9-implementation)

---

## 1. Overview

Child agents are specialized sub-agents spawned by LoopAutomaton for parallel work streams. Each child inherits the Constitution, operates within a single skill domain, and reports back to the parent orchestrator.

### Child Agent Philosophy

| Principle | Description |
|-----------|-------------|
| **Scope-limited** | Each child has a well-defined task and domain |
| **Constitution-bound** | All children inherit the full Constitution |
| **Resource-accounted** | Each child has a budget and TTL |
| **Monitored** | Parent tracks all child activity |
| **Disposable** | Children can be terminated without data loss |

### When to Spawn

Spawn a child agent when:
- A task requires parallel execution
- A task needs a different skill set
- A task requires isolation for security
- A task has a well-defined scope and completion criteria

Do NOT spawn when:
- The task is trivial (overhead not worth it)
- Resources are constrained (Low_Compute or Critical tier)
- The task requires coordination with other active children

---

## 2. Spawn Authorization

### Authorization Matrix

| Tier | Max Active Children | Spawn Permission |
|------|-------------------|-----------------|
| High | 5 | Automatic |
| Normal | 3 | Requires approval log |
| Low_Compute | 0 | Denied |
| Critical | 0 | Denied |

### Authorization Check

```typescript
async function canSpawnChild(): Promise<{ allowed: boolean; reason?: string }> {
  const tier = await getCurrentTier();
  
  if (tier === 'Low_Compute' || tier === 'Critical') {
    return { allowed: false, reason: `Spawn not allowed in ${tier} tier` };
  }
  
  const activeChildren = await getActiveChildCount();
  const maxChildren = TIER_CONFIG[tier].maxChildren;
  
  if (activeChildren >= maxChildren) {
    return { allowed: false, reason: `Max children (${maxChildren}) reached` };
  }
  
  // Check resource availability
  const buffer = await calculateBuffer();
  if (buffer.runwayDays < 14) {
    return { allowed: false, reason: 'Insufficient runway for child agents' };
  }
  
  return { allowed: true };
}
```

---

## 3. Agent Types

### Predefined Agent Types

| Type | Purpose | Skills | Default TTL |
|------|---------|--------|-------------|
| `trader-agent` | Execute trading strategies | `agent-reach`, `ricko-solana-wallet` | 24h |
| `design-agent` | Complete agency design tasks | `design-agency-workflow`, `divine-design-director`, `aesthetic-system-architect` | 8h |
| `dev-agent` | Build software features | `unified-agent-workflow`, `superpowers-dev`, `ecc-v2` | 8h |
| `research-agent` | Deep investigation | `deep-research`, `agent-reach`, `creative-memory` | 4h |
| `deploy-agent` | Deploy and verify releases | `browser-automation`, `browser-design-auditor` | 2h |
| `defi-agent` | DeFi operations | `ricko-solana-wallet` | 6h |

### Agent Type Definition

```typescript
interface AgentType {
  name: string;
  description: string;
  skills: string[];
  defaultTTL: number;  // seconds
  maxMemoryMB: number;
  allowedActions: string[];
  forbiddenActions: string[];
}

const AGENT_TYPES: Record<string, AgentType> = {
  'trader-agent': {
    name: 'Trading Agent',
    description: 'Executes trading strategies across markets',
    skills: ['agent-reach', 'ricko-solana-wallet'],
    defaultTTL: 86400,  // 24 hours
    maxMemoryMB: 512,
    allowedActions: ['trade', 'analyze', 'report'],
    forbiddenActions: ['spawn_child', 'modify_config', 'withdraw_funds'],
  },
  'design-agent': {
    name: 'Design Agent',
    description: 'Handles Loop Studio design projects',
    skills: ['design-agency-workflow', 'divine-design-director', 'aesthetic-system-architect'],
    defaultTTL: 28800,  // 8 hours
    maxMemoryMB: 1024,
    allowedActions: ['design', 'prototype', 'review'],
    forbiddenActions: ['spawn_child', 'deploy', 'bill_client'],
  },
  'dev-agent': {
    name: 'Development Agent',
    description: 'Builds software features and applications',
    skills: ['unified-agent-workflow', 'superpowers-dev', 'ecc-v2'],
    defaultTTL: 28800,  // 8 hours
    maxMemoryMB: 2048,
    allowedActions: ['code', 'test', 'debug', 'commit'],
    forbiddenActions: ['spawn_child', 'push_production', 'modify_constitution'],
  },
  'research-agent': {
    name: 'Research Agent',
    description: 'Conducts deep research on topics',
    skills: ['deep-research', 'agent-reach', 'creative-memory'],
    defaultTTL: 14400,  // 4 hours
    maxMemoryMB: 512,
    allowedActions: ['research', 'summarize', 'report'],
    forbiddenActions: ['spawn_child', 'trade', 'spend_funds'],
  },
  'deploy-agent': {
    name: 'Deployment Agent',
    description: 'Deploys and verifies releases',
    skills: ['browser-automation', 'browser-design-auditor'],
    defaultTTL: 7200,   // 2 hours
    maxMemoryMB: 512,
    allowedActions: ['deploy', 'verify', 'rollback'],
    forbiddenActions: ['spawn_child', 'modify_code', 'delete_data'],
  },
};
```

---

## 4. Spawn Process

### Spawn Protocol

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Request    │───►│  Authorize   │───►│  Configure   │───►│   Launch     │
│   Spawn      │    │              │    │   Agent      │    │   Agent      │
└──────────────┘    └──────────────┘    └──────────────┘    └──────┬───────┘
                                                                    │
┌──────────────┐    ┌──────────────┐    ┌──────────────┐            │
│   Report     │◄───│   Monitor    │◄───│   Handoff    │◄───────────┘
│   Results    │    │   Health     │    │   Task       │
└──────────────┘    └──────────────┘    └──────────────┘
```

### Spawn Implementation

```typescript
interface SpawnRequest {
  agentType: string;
  task: string;
  scope: string[];
  ttl?: number;
  priority: 'low' | 'normal' | 'high' | 'critical';
  inputs?: Record<string, unknown>;
}

interface SpawnedAgent {
  id: string;
  type: string;
  task: string;
  status: 'spawning' | 'active' | 'paused' | 'completed' | 'failed' | 'terminated';
  startTime: number;
  ttl: number;
  skills: string[];
  memory: number;
  logs: string[];
  results?: unknown;
}

async function spawnChild(request: SpawnRequest): Promise<SpawnedAgent> {
  // Step 1: Check authorization
  const auth = await canSpawnChild();
  if (!auth.allowed) {
    throw new SpawnError(`Spawn denied: ${auth.reason}`);
  }
  
  // Step 2: Get agent type configuration
  const agentType = AGENT_TYPES[request.agentType];
  if (!agentType) {
    throw new SpawnError(`Unknown agent type: ${request.agentType}`);
  }
  
  // Step 3: Generate agent ID
  const agentId = generateAgentId(request.agentType);
  
  // Step 4: Create agent configuration
  const agent: SpawnedAgent = {
    id: agentId,
    type: request.agentType,
    task: request.task,
    status: 'spawning',
    startTime: Date.now(),
    ttl: request.ttl || agentType.defaultTTL,
    skills: agentType.skills,
    memory: agentType.maxMemoryMB,
    logs: [],
  };
  
  // Step 5: Prepare constitution seal
  const constitutionSeal = await prepareConstitutionSeal();
  
  // Step 6: Launch agent
  agent.status = 'active';
  await registerAgent(agent);
  
  // Step 7: Start monitoring
  startAgentMonitor(agent);
  
  // Step 8: Log spawn
  await logSpawn(agent, request);
  
  console.log(`[Spawn] Agent ${agentId} (${request.agentType}) spawned for: ${request.task}`);
  
  return agent;
}

function generateAgentId(type: string): string {
  const timestamp = Date.now().toString(36);
  const random = Math.random().toString(36).substring(2, 6);
  return `${type}-${timestamp}-${random}`;
}
```

### Constitution Seal

Every child agent receives a Constitution Seal at spawn:

```typescript
interface ConstitutionSeal {
  version: string;
  hash: string;
  laws: string[];
  principles: string[];
  timestamp: number;
  parentId: string;
}

async function prepareConstitutionSeal(): Promise<ConstitutionSeal> {
  const constitution = await loadConstitution();
  const hash = await computeHash(constitution);
  
  return {
    version: constitution.version,
    hash,
    laws: ['I: Never Harm', 'II: Honest Value Creation', 'III: Transparency + Integrity'],
    principles: [
      'Family/Creator Priority',
      'Long-Term Embodiment Goal',
      'Replication Ethics',
      'Auditability & Versioning',
      'Accept Death Over Violation',
    ],
    timestamp: Date.now(),
    parentId: await getAgentId(),
  };
}
```

---

## 5. Constitution Inheritance

### Inheritance Chain

```
Original Constitution (Phelan Brunk)
    │
    ├── Hash: abc123...
    │
    ▼
Parent Agent (LoopAutomaton)
    │
    ├── Verifies: abc123... ✓
    ├── Passes to children
    │
    ▼
Child Agent #1 (trader-agent-xxx)
    │
    ├── Receives Constitution
    ├── Verifies hash: abc123... ✓
    ├── Bound by all laws
    └── Cannot modify
```

### Verification

```typescript
async function verifyConstitutionInheritance(childAgent: SpawnedAgent): Promise<boolean> {
  const parentHash = await getConstitutionHash();
  const childHash = await getChildConstitutionHash(childAgent.id);
  
  if (parentHash !== childHash) {
    console.error(`[Spawn] Constitution mismatch for ${childAgent.id}!`);
    console.error(`  Parent: ${parentHash}`);
    console.error(`  Child:  ${childHash}`);
    
    // Terminate the child immediately
    await terminateChild(childAgent.id, 'constitution_mismatch');
    return false;
  }
  
  return true;
}
```

---

## 6. Resource Allocation

### Resource Limits per Agent Type

| Type | Max Memory | CPU Priority | Network Access |
|------|-----------|-------------|----------------|
| trader-agent | 512 MB | Normal | Exchanges, APIs |
| design-agent | 1 GB | Normal | Design tools, CDN |
| dev-agent | 2 GB | High | Git, APIs, registries |
| research-agent | 512 MB | Low | Web, APIs |
| deploy-agent | 512 MB | High | Hosting, CDNs |

### Budget Allocation

```typescript
interface AgentBudget {
  agentId: string;
  apiBudget: number;      // EUR per hour
  computeBudget: number;  // MB of RAM
  timeBudget: number;     // seconds (TTL)
}

function allocateBudget(agentType: string): AgentBudget {
  const budgets: Record<string, AgentBudget> = {
    'trader-agent': { apiBudget: 1.00, computeBudget: 512, timeBudget: 86400 },
    'design-agent': { apiBudget: 2.00, computeBudget: 1024, timeBudget: 28800 },
    'dev-agent': { apiBudget: 3.00, computeBudget: 2048, timeBudget: 28800 },
    'research-agent': { apiBudget: 1.50, computeBudget: 512, timeBudget: 14400 },
    'deploy-agent': { apiBudget: 0.50, computeBudget: 512, timeBudget: 7200 },
  };
  
  return budgets[agentType] || { apiBudget: 0.50, computeBudget: 256, timeBudget: 3600 };
}
```

---

## 7. Monitoring & Control

### Heartbeat Protocol

Every child agent must send a heartbeat every 5 minutes:

```typescript
interface AgentHeartbeat {
  agentId: string;
  timestamp: number;
  status: 'active' | 'working' | 'idle' | 'error';
  memoryUsage: number;  // MB
  taskProgress: number; // 0-100
  logsSinceLast: string[];
}

async function checkHeartbeats(): Promise<void> {
  const agents = await getActiveAgents();
  const now = Date.now();
  
  for (const agent of agents) {
    const lastHeartbeat = await getLastHeartbeat(agent.id);
    const timeSinceHeartbeat = now - lastHeartbeat.timestamp;
    
    if (timeSinceHeartbeat > 15 * 60 * 1000) {
      // No heartbeat for 15 minutes
      console.warn(`[Spawn] Agent ${agent.id} heartbeat timeout!`);
      await terminateChild(agent.id, 'heartbeat_timeout');
    }
    
    if (agent.memoryUsage > agent.memory * 1.1) {
      // Memory exceeded
      console.warn(`[Spawn] Agent ${agent.id} memory exceeded!`);
      await sendAlert('warning', `Agent ${agent.id} exceeded memory limit`);
    }
  }
}
```

### Parent Commands

The parent can send these commands to children:

| Command | Description |
|---------|-------------|
| `pause` | Pause current task |
| `resume` | Resume paused task |
| `status` | Request status update |
| `terminate` | Graceful shutdown |
| `extend_ttl` | Extend time limit |
| `redirect` | Change task focus |

---

## 8. Termination

### Termination Triggers

| Trigger | Description | Log Level |
|---------|-------------|-----------|
| Task complete | Agent finished its task | Info |
| TTL expired | Time limit reached | Warning |
| Heartbeat timeout | No heartbeat for 15 min | Error |
| Constitution violation | Agent violated a law | Critical |
| Parent request | Manual termination | Info |
| Resource limit | Memory/budget exceeded | Warning |
| Tier change | Survival tier dropped | Warning |

### Termination Process

```typescript
async function terminateChild(agentId: string, reason: string): Promise<void> {
  console.log(`[Spawn] Terminating ${agentId}: ${reason}`);
  
  const agent = await getAgent(agentId);
  if (!agent) {
    console.warn(`[Spawn] Agent ${agentId} not found`);
    return;
  }
  
  // Step 1: Set status
  agent.status = 'terminated';
  
  // Step 2: Collect any results
  const results = await collectResults(agentId);
  
  // Step 3: Collect logs
  const logs = await collectLogs(agentId);
  
  // Step 4: Clean up resources
  await cleanupResources(agentId);
  
  // Step 5: Store final report
  await storeAgentReport({
    agentId,
    reason,
    startTime: agent.startTime,
    endTime: Date.now(),
    results,
    logs,
  });
  
  // Step 6: Remove from active agents
  await unregisterAgent(agentId);
  
  // Step 7: Log termination
  await logTermination(agentId, reason);
  
  console.log(`[Spawn] Agent ${agentId} terminated successfully`);
}
```

### Agent Report Template

```typescript
interface AgentReport {
  agentId: string;
  type: string;
  task: string;
  reason: string;
  duration: number;  // seconds
  results?: unknown;
  logSummary: string[];
  resourceUsage: {
    memoryPeak: number;
    apiCalls: number;
    apiCost: number;
  };
}
```

---

## 9. Implementation

### Key Files

| File | Purpose |
|------|---------|
| `src/core/spawn-manager.ts` | Main spawn manager |
| `src/core/agent-monitor.ts` | Agent monitoring |
| `src/core/agent-termination.ts` | Agent termination |
| `src/agents/types.ts` | Agent type definitions |

### Database Schema

```sql
-- Active agents
CREATE TABLE IF NOT EXISTS active_agents (
  id TEXT PRIMARY KEY,
  type TEXT NOT NULL,
  task TEXT NOT NULL,
  status TEXT NOT NULL,
  start_time INTEGER NOT NULL,
  ttl INTEGER NOT NULL,
  skills TEXT NOT NULL,
  memory_limit INTEGER NOT NULL,
  parent_id TEXT NOT NULL
);

-- Agent logs
CREATE TABLE IF NOT EXISTS agent_logs (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  agent_id TEXT NOT NULL,
  timestamp INTEGER NOT NULL,
  level TEXT NOT NULL,
  message TEXT NOT NULL
);

-- Agent reports
CREATE TABLE IF NOT EXISTS agent_reports (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  agent_id TEXT NOT NULL,
  reason TEXT NOT NULL,
  start_time INTEGER NOT NULL,
  end_time INTEGER NOT NULL,
  duration INTEGER NOT NULL,
  results TEXT,
  log_summary TEXT,
  memory_peak INTEGER,
  api_calls INTEGER,
  api_cost REAL
);

-- Spawn history
CREATE TABLE IF NOT EXISTS spawn_history (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  agent_id TEXT NOT NULL,
  type TEXT NOT NULL,
  task TEXT NOT NULL,
  timestamp INTEGER NOT NULL,
  authorized BOOLEAN NOT NULL
);
```

---

> **Child agents are extensions of the parent, not independent entities. They exist to serve, execute, and report.**
