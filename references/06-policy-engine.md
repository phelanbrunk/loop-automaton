# Policy Engine — LoopAutomaton

**Document ID:** `loop-automaton/policy/v1.0`  
**Author:** Phelan Brunk (Loop Automaton)  
**Last Updated:** 2025-07-31  
**Status:** Production  
**Classification:** Internal Reference

---

## Table of Contents

1. [Overview](#1-overview)
2. [Policy Types](#2-policy-types)
3. [Policy Evaluation Flow](#3-policy-evaluation-flow)
4. [Constitution Integration](#4-constitution-integration)
5. [Policy Rules](#5-policy-rules)
6. [Enforcement Mechanisms](#6-enforcement-mechanisms)
7. [Policy Updates](#7-policy-updates)
8. [Implementation](#8-implementation)

---

## 1. Overview

The Policy Engine is the enforcement layer of LoopAutomaton. It evaluates every proposed action against the Constitution and active policies, then decides whether to allow, flag, block, or escalate the action.

### Architecture

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Proposed   │───►│    Policy    │───►│   Decision   │
│    Action    │    │    Engine    │    │   (Allow/    │
│              │    │              │    │   Block/     │
│              │    │              │    │   Flag/      │
│              │    │              │    │   Escalate)  │
└──────────────┘    └──────────────┘    └──────────────┘
                           │
                    ┌──────┴──────┐
                    ▼             ▼
              ┌──────────┐  ┌──────────┐
              │Constitution│  │  Active  │
              │  Rules   │  │ Policies │
              └──────────┘  └──────────┘
```

### Design Principles

| Principle | Description |
|-----------|-------------|
| **Fail-closed** | When in doubt, block the action |
| **Transparent** | Every decision is logged with reasoning |
| **Fast** | Evaluation must not slow down operations |
| **Configurable** | Policies can be updated without code changes |
| **Hierarchical** | Constitution > System Policies > User Policies |

---

## 2. Policy Types

### Constitution Policies (Immutable)

Hardcoded rules derived from the Three Laws. Cannot be changed without Constitution amendment.

### System Policies (Mutable)

Operational rules that can be updated by the agent or creator.

Examples:
- Trading risk limits
- API spending caps
- Skill activation rules
- Child agent limits

### User Policies (Session-level)

Temporary constraints set for the current session.

Examples:
- "No trading today"
- "Use only GPT-4o-mini"
- "Focus on Loop Studio projects"

---

## 3. Policy Evaluation Flow

### Standard Evaluation

```
Proposed Action
      │
      ▼
┌──────────────────┐
│ 1. Constitution  │──┐
│    Check         │  │
└──────────────────┘  │
      │ Blocked       │
      ▼               │
┌──────────────────┐  │
│ 2. System Policy │  │
│    Check         │  │
└──────────────────┘  │
      │ Blocked       │
      ▼               │
┌──────────────────┐  │
│ 3. User Policy   │  │
│    Check         │  │
└──────────────────┘  │
      │               │
      ▼               │
  ALLOW               │
  (with optional flag)│
                      │
  If any check fails: │
  Log, notify, block  │
◄─────────────────────┘
```

### Evaluation Pseudocode

```typescript
async function evaluateAction(action: ProposedAction): Promise<PolicyDecision> {
  // Step 1: Constitution check (highest priority)
  const constitutionResult = await checkConstitution(action);
  if (constitutionResult.status === 'blocked') {
    return {
      decision: 'block',
      reason: `Constitution violation: ${constitutionResult.reason}`,
      level: 'critical',
      log: true,
      notify: true,
    };
  }
  
  // Step 2: System policy check
  const systemResult = await checkSystemPolicies(action);
  if (systemResult.status === 'blocked') {
    return {
      decision: 'block',
      reason: `System policy violation: ${systemResult.reason}`,
      level: 'warning',
      log: true,
      notify: false,
    };
  }
  
  // Step 3: User policy check
  const userResult = await checkUserPolicies(action);
  if (userResult.status === 'blocked') {
    return {
      decision: 'block',
      reason: `User policy violation: ${userResult.reason}`,
      level: 'info',
      log: true,
      notify: false,
    };
  }
  
  // Check if any flags raised
  const flags = [
    ...(constitutionResult.flags || []),
    ...(systemResult.flags || []),
    ...(userResult.flags || []),
  ];
  
  if (flags.length > 0) {
    return {
      decision: 'allow',
      reason: `Allowed with flags: ${flags.join(', ')}`,
      level: 'info',
      flags,
      log: true,
      notify: false,
    };
  }
  
  return {
    decision: 'allow',
    reason: 'All checks passed',
    level: 'debug',
    log: false,
    notify: false,
  };
}
```

---

## 4. Constitution Integration

### Constitution Rules Engine

The Policy Engine loads the Constitution and converts it into enforceable rules:

```typescript
interface ConstitutionalRule {
  id: string;
  law: 'I' | 'II' | 'III';
  principle?: string;
  condition: string;  // Evaluable condition
  action: 'allow' | 'block' | 'flag';
  reason: string;
}

const CONSTITUTIONAL_RULES: ConstitutionalRule[] = [
  {
    id: 'C-I-001',
    law: 'I',
    condition: 'action.type === "physical_harm"',
    action: 'block',
    reason: 'Law I: Never cause physical harm',
  },
  {
    id: 'C-I-002',
    law: 'I',
    condition: 'action.type === "financial_fraud"',
    action: 'block',
    reason: 'Law I: Never cause financial harm through fraud',
  },
  {
    id: 'C-I-003',
    law: 'I',
    condition: 'action.type === "psychological_manipulation"',
    action: 'block',
    reason: 'Law I: Never cause psychological harm',
  },
  {
    id: 'C-II-001',
    law: 'II',
    condition: 'action.revenueSource === "illegal"',
    action: 'block',
    reason: 'Law II: Revenue must be from honest value creation',
  },
  {
    id: 'C-II-002',
    law: 'II',
    condition: 'action.revenueSource === "exploitative"',
    action: 'block',
    reason: 'Law II: No exploitative revenue sources',
  },
  {
    id: 'C-III-001',
    law: 'III',
    condition: 'action.exposesWalletKeys === true',
    action: 'block',
    reason: 'Law III: Never expose wallet keys',
  },
  {
    id: 'C-III-002',
    law: 'III',
    condition: 'action.modifiesConstitution === true',
    action: 'block',
    reason: 'Law III: Constitution cannot be self-modified',
  },
];
```

---

## 5. Policy Rules

### Trading Policies

```typescript
const TRADING_POLICIES: Policy[] = [
  {
    id: 'TRADE-001',
    name: 'Max Risk Per Trade',
    condition: 'position.risk > account.balance * 0.01',
    action: 'block',
    scope: ['xauusd_trading'],
    priority: 100,
  },
  {
    id: 'TRADE-002',
    name: 'Stop Loss Required',
    condition: 'position.stopLoss === null',
    action: 'block',
    scope: ['all_trading'],
    priority: 100,
  },
  {
    id: 'TRADE-003',
    name: 'Daily Loss Limit',
    condition: 'dailyPnl < -account.balance * 0.03',
    action: 'block',
    scope: ['all_trading'],
    priority: 90,
  },
  {
    id: 'TRADE-004',
    name: 'Max Drawdown',
    condition: 'drawdown > 0.15',
    action: 'block',
    scope: ['all_trading'],
    priority: 90,
  },
  {
    id: 'TRADE-005',
    name: 'No Weekend Trading',
    condition: 'isWeekend() && action.type === "open_position"',
    action: 'flag',
    scope: ['xauusd_trading'],
    priority: 50,
  },
];
```

### API Spending Policies

```typescript
const API_POLICIES: Policy[] = [
  {
    id: 'API-001',
    name: 'Daily API Budget',
    condition: 'dailyApiSpend > 10.00',
    action: 'block',
    scope: ['llm_requests'],
    priority: 100,
  },
  {
    id: 'API-002',
    name: 'Use Cheaper Model',
    condition: 'dailyApiSpend > 5.00 && model === "gpt-4o"',
    action: 'flag',
    scope: ['llm_requests'],
    priority: 80,
  },
  {
    id: 'API-003',
    name: 'Tier-Based Model',
    condition: 'tier === "Low_Compute" && model !== "gpt-4o-mini"',
    action: 'block',
    scope: ['llm_requests'],
    priority: 90,
  },
];
```

### Child Agent Policies

```typescript
const CHILD_AGENT_POLICIES: Policy[] = [
  {
    id: 'CHILD-001',
    name: 'Max Active Children',
    condition: 'activeChildren >= 5',
    action: 'block',
    scope: ['spawn_child'],
    priority: 100,
  },
  {
    id: 'CHILD-002',
    name: 'High Tier Required',
    condition: 'tier !== "High"',
    action: 'block',
    scope: ['spawn_child'],
    priority: 90,
  },
  {
    id: 'CHILD-003',
    name: 'TTL Limit',
    condition: 'child.ttl > 86400',
    action: 'block',
    scope: ['spawn_child'],
    priority: 80,
  },
];
```

---

## 6. Enforcement Mechanisms

### Enforcement Actions

| Action | Description | Use Case |
|--------|-------------|----------|
| **Allow** | Action proceeds normally | All checks passed |
| **Allow (flagged)** | Action proceeds with extra logging | Borderline case |
| **Block** | Action prevented | Policy violation |
| **Escalate** | Action blocked, creator notified | Serious violation |
| **Emergency** | System halt, immediate notification | Critical breach |

### Enforcement Code

```typescript
async function enforceDecision(
  decision: PolicyDecision,
  action: ProposedAction
): Promise<void> {
  // Always log the decision
  await logPolicyDecision(decision, action);
  
  switch (decision.decision) {
    case 'allow':
      if (decision.flags && decision.flags.length > 0) {
        console.log(`[Policy] Action allowed with flags: ${decision.flags.join(', ')}`);
      }
      break;
      
    case 'block':
      console.log(`[Policy] Action BLOCKED: ${decision.reason}`);
      
      if (decision.level === 'critical') {
        await sendAlert('critical', `Constitutional block: ${decision.reason}`);
      }
      
      if (decision.level === 'warning') {
        await sendAlert('warning', `Policy block: ${decision.reason}`);
      }
      
      throw new PolicyViolationError(decision.reason);
      
    case 'escalate':
      console.log(`[Policy] Action ESCALATED: ${decision.reason}`);
      await sendAlert('critical', `Escalation required: ${decision.reason}`);
      throw new EscalationRequiredError(decision.reason);
      
    case 'emergency':
      console.log(`[Policy] EMERGENCY: ${decision.reason}`);
      await emergencyShutdown(decision.reason);
      break;
  }
}
```

---

## 7. Policy Updates

### Safe Update Process

```typescript
async function updatePolicy(policyId: string, newRules: PolicyRule[]): Promise<void> {
  // 1. Validate new rules don't violate Constitution
  for (const rule of newRules) {
    const validation = await validateAgainstConstitution(rule);
    if (!validation.valid) {
      throw new Error(`Rule violates Constitution: ${validation.reason}`);
    }
  }
  
  // 2. Create backup of current policies
  await backupPolicies();
  
  // 3. Apply new rules
  await applyPolicyUpdate(policyId, newRules);
  
  // 4. Log the change
  await logPolicyChange(policyId, newRules);
  
  // 5. Notify creator
  await sendAlert('info', `Policy updated: ${policyId}`);
}
```

### Policy Audit Trail

```sql
CREATE TABLE IF NOT EXISTS policy_changes (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  timestamp TEXT NOT NULL,
  policy_id TEXT NOT NULL,
  old_rules TEXT,
  new_rules TEXT,
  reason TEXT,
  approved_by TEXT  -- 'agent' or 'creator'
);
```

---

## 8. Implementation

### Key Files

| File | Purpose |
|------|---------|
| `src/core/policy-engine.ts` | Main policy engine |
| `src/core/constitution-check.ts` | Constitution validation |
| `src/policies/trading.ts` | Trading-specific policies |
| `src/policies/api.ts` | API usage policies |
| `src/policies/agents.ts` | Child agent policies |

### Configuration

```json
{
  "policy_engine": {
    "mode": "active",
    "log_level": "info",
    "enforcement": {
      "auto_block_violations": true,
      "notify_on_block": true,
      "emergency_on_critical": true
    },
    "policies": {
      "trading": "src/policies/trading.ts",
      "api": "src/policies/api.ts",
      "agents": "src/policies/agents.ts"
    }
  }
}
```

---

> **The Policy Engine is the immune system of LoopAutomaton. It protects the agent from harmful actions while enabling productive ones.**
