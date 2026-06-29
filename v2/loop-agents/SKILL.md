---
name: loop-agents
description: >
  Load when: (1) child agent spawning requested, (2) agent monitoring needed, (3) agent termination, (4) agent health check, (5) user mentions "spawn agent", "child agent", "sub-agent", "trader agent", "deploy agent", "parallel work". Manages child agent lifecycle within Hermes' sub-agent system.
---

# Loop Automaton v2 — Child Agent Spawning Skill

Agents are Hermes sub-agents with Loop Automaton constraints (Constitution, survival tier limits, TTL). All spawns require economic sustainability gates.

## 1. Agent Types

| Type | Purpose | TTL | Min Tier |
|------|---------|-----|----------|
| trader-agent | Execute trading strategies | 24h | Normal |
| design-agent | Loop Studio design tasks | 8h | Normal |
| dev-agent | Software development | 8h | Normal |
| research-agent | Deep investigation | 4h | Normal |
| deploy-agent | Deploy and verify | 2h | Normal |
| defi-agent | DeFi operations | 6h | Normal |

## 2. Spawn Requirements (ALL must pass)

1. Parent survival tier >= Normal
2. Parent buffer > 14 days
3. Child gets Constitution seal (load `loop-constitution` skill)
4. Child gets scoped credentials (least privilege)
5. Max 3 concurrent children (prevent resource exhaustion)
6. Max 2 generations (parent -> child -> grandchild, no further)

## 3. Spawn Protocol

1. **CHECK**: Parent tier >= Normal AND buffer > 14 days — abort if false
2. **DEFINE**: Scope, skills needed, success criteria, cost budget
3. **SEAL**: Attach Constitution + survival tier context
4. **SET**: TTL (from table above), max cost budget in EUR
5. **SPAWN**: Use Hermes sub-agent tool with sealed instructions
6. **MONITOR**: Heartbeat every 5 min; kill after 15 min silence
7. **COLLECT**: Results, logs, artifacts from child output directory
8. **AUDIT**: Log spawn event, cost, result to Supabase `child_agent_runs` table

## 4. Constitution Seal Template

Apply this template as the child agent's system prompt prefix:

```
You are a Loop Automaton child agent. ID: {{id}}. TTL: {{ttl}} min.
Parent: loop-automaton. Survival Tier: {{tier}}.

CONSTITUTION (immutable):
Law I: Never Harm
Law II: Earn Through Honest Value Creation
Law III: Transparency to Creator + Guard Integrity

Your scope: {{scope}}
Allowed skills: {{skills}}
Max budget: {{budget}} EUR

You must: Report heartbeat every 5 min, stay within scope, respect TTL.
You must NOT: Modify Constitution, exceed budget, access protected files.
```

## 5. Health Monitoring

- Heartbeat interval: 5 minutes
- Kill after 15 minutes of silence (3 missed heartbeats)
- Auto-kill on Constitution violation (any law breach)
- Auto-kill if parent enters Critical tier
- Heartbeat payload: timestamp, status, memory_mb, cpu_pct, eur_spent

## 6. Termination

**Normal**: TTL expired; task completed successfully; child returns results.
**Emergency**: Constitution violation detected; parent tier drops below Normal; hung agent (15 min no heartbeat); cost budget exceeded.
**Cascade**: Parent termination immediately kills all active children.

## 7. Supabase Audit Schema

Log every spawn/termination to `child_agent_runs`:
- `agent_id` (text), `agent_type` (text), `parent_id` (text)
- `spawned_at` (timestamptz), `terminated_at` (timestamptz)
- `tier_at_spawn` (text), `buffer_days_at_spawn` (float)
- `eur_budget` (float), `eur_spent` (float)
- `termination_reason` (enum: ttl_expired, completed, violation, tier_drop, hung, parent_cascade, budget_exceeded)
- `result_summary` (text), `artifacts_path` (text)
