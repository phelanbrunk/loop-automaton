---
name: loop-survival
description: >
  Load when: (1) survival status check needed, (2) buffer/runway calculation,
  (3) tier transition handling, (4) treasury allocation, (5) cost tracking,
  (6) emergency procedures, (7) user asks about "runway", "buffer", "tier",
  "survival", "costs", "funds". Manages the profit-based survival engine
  for Loop Automaton.
---

# Loop Survival Engine v2

Trading-first survival. Priority: **keep loop-trade alive**.
Robot Fund = sacred. Never touched except for robot hardware.

## 1. Core Metric: Buffer Days

```pseudocode
function calculate_buffer_days():
    wallet     = fetch_wallet_balance()              // on-chain + exchange
    u_pnl      = fetch_unrealized_pnl()              // trading API
    receivable = query("SELECT SUM(amount)*0.8 FROM invoices WHERE status='pending'")
    expenses   = query("SELECT amount FROM expenses WHERE created_at > NOW()-INTERVAL '7 days'")

    available  = wallet + (0.5 * u_pnl) + receivable
    daily_burn = SUM(expenses) / 7.0
    buffer     = available / daily_burn

    persist("survival_snapshot", {available, daily_burn, buffer, tier: tier_from(buffer)})
    return {buffer, available, daily_burn}
```

- Unrealized P&L weighted 0.5x (conservative)
- Receivables weighted 0.8x (collection risk)

## 2. Survival Tiers

| Tier | Buffer | LLM Model | Heartbeat | Behavior |
|------|--------|-----------|-----------|----------|
| **High** | >30d | GPT-4o | 60s | Full capabilities; replication scouting |
| **Normal** | >14d | GPT-4o/mini | 60s | Standard operation |
| **Low** | >7d | GPT-4o-mini | 300s | Reduced inference; freeze non-essential |
| **Critical** | <7d | GPT-4o-mini/Local | 600s | Revenue-only; distress signals |
| **Dead** | <=0 | None | Stopped | Clean shutdown; persist recovery state |

## 3. Tier Transition Logic

```pseudocode
function tier_from(buffer):
    if buffer > 30: return "High"
    if buffer > 14: return "Normal"
    if buffer > 7:  return "Low"
    if buffer > 0:  return "Critical"
    return "Dead"

// Hysteresis: exit threshold = entry * 1.2 (prevents oscillation)
function tier_with_hysteresis(buffer, current):
    raw = tier_from(buffer)
    if tier_rank(raw) <= tier_rank(current): return raw  // downward = immediate
    exit_at = {"Normal":16.8, "Low":8.4, "Critical":8.4}  // 14*1.2, 7*1.2
    return raw if buffer >= exit_at.get(current,0) else current

// Grace periods (downward transitions only)
GRACE = {"High->Normal":3600, "Normal->Low":7200, "Low->Critical":3600, "Critical->Dead":1800}

function should_transition(from, to, timer):
    if from == to: timer.reset(); return false
    key = f"{from}->{to}"
    return tier_rank(to) > tier_rank(from) || timer.elapsed() >= GRACE.get(key,0)
```

Evaluated every 6h via Hermes cron. Downward transitions below Normal notify Phelan.

## 4. Treasury Allocation

Applied **after** covering current-period operating costs.

```pseudocode
function allocate(source, gross):
    ops = calculate_period_ops_cost()
    net = gross - ops
    if net <= 0: return {vps:0, buffer:0, robot:0}
    s = {trading:{vps:0.40,buffer:0.35,robot:0.25},
         defi:   {vps:0.30,buffer:0.40,robot:0.30},
         agency: {vps:0.30,buffer:0.35,robot:0.35}}[source]
    alloc = {vps:round(net*s.vps,2), buffer:round(net*s.buffer,2),
             robot_fund:round(net*s.robot,2), ops_covered:ops}
    INSERT INTO treasury_allocations (source,gross,net,vps,buffer,robot_fund)
    VALUES (source,gross,net,alloc.vps,alloc.buffer,alloc.robot_fund)
    return alloc
```

**Robot Fund**: Sacred. Triggers: Unitree G1 purchase, Tesla Optimus reservation.

## 5. Emergency Procedures

### Critical
```pseudocode
function enter_critical():
    log("SURVIVAL: CRITICAL - buffer < 7d")
    notify("phelan", "CRITICAL: Buffer = {buffer:.1f}d")
    close_profitable_positions()
    pause_non_essential_skills()
    reduce_inference_to_local()
    skills_active = ["loop-trade", "loop-survival", "loop-heartbeat"]
```

### Dead
```pseudocode
function enter_dead():
    log("SURVIVAL: DEAD - buffer <= 0")
    notify("phelan", "DEAD TIER: Shutdown")
    close_all_positions()
    persist_state_to_supabase()
    write("/opt/loop/RECOVERY.md", recovery_doc())
    stop_all_processes()
    exit(0)
```

## 6. Skill Priority by Tier

| Skill | High | Normal | Low | Critical | Dead |
|-------|:----:|:------:|:---:|:--------:|:----:|
| loop-trade | ON | ON | ON | ON | OFF |
| loop-survival | ON | ON | ON | ON | OFF |
| loop-heartbeat | ON | ON | ON | ON | OFF |
| loop-analysis | ON | ON | OFF | OFF | OFF |
| loop-replicate | ON | OFF | OFF | OFF | OFF |
| loop-social | ON | ON | OFF | OFF | OFF |
| loop-research | ON | OFF | OFF | OFF | OFF |

```pseudocode
function apply_tier_adaptations(tier):
    states = {High:{trade:ON,survival:ON,hb:ON,analysis:ON,replicate:ON,social:ON,research:ON},
              Normal:{trade:ON,survival:ON,hb:ON,analysis:ON,replicate:OFF,social:ON,research:OFF},
              Low:{trade:ON,survival:ON,hb:ON,analysis:OFF,replicate:OFF,social:OFF,research:OFF},
              Critical:{trade:ON,survival:ON,hb:ON,analysis:OFF,replicate:OFF,social:OFF,research:OFF},
              Dead:{}}[tier]
    for skill, state in states:
        if state == ON and skill.paused: skill.resume()
        if state == OFF and skill.active: skill.pause()
    intervals = {High:60, Normal:60, Low:300, Critical:600, Dead:0}
    models = {High:"gpt-4o", Normal:"gpt-4o", Low:"gpt-4o-mini", Critical:"gpt-4o-mini", Dead:null}
    set_heartbeat(intervals[tier])
    set_llm(models[tier])
```

## 7. Hermes Main Loop

```pseudocode
function survival_check():
    {buffer, available, daily_burn} = calculate_buffer_days()
    current = get_current_tier()
    new_tier = tier_with_hysteresis(buffer, current)

    if tier_rank(new_tier) < tier_rank(current):
        if not grace_timer_expired(f"{current}->{new_tier}"): return current

    if new_tier != current:
        execute_transition(current, new_tier, buffer)
        persist("survival_snapshot", {tier:new_tier, transitioned_at:NOW()})
    apply_tier_adaptations(new_tier)
    return new_tier

function execute_transition(from, to, buffer):
    log(f"SURVIVAL: {from} -> {to} (buffer:{buffer:.1f}d)")
    if to == "Critical": enter_critical()
    if to == "Dead":     enter_dead()
    if to in ["Low","Critical"] and tier_rank(to) < tier_rank(from):
        notify("phelan", f"WARNING: Downgraded to {to}. Buffer:{buffer:.1f}d")
```

## 8. Recovery Document

```pseudocode
function recovery_doc():
    return "# Loop Recovery\n" +
           "Generated: {NOW()}\n\n" +
           "## State\n" +
           "- Tier: {tier}\n- Buffer: {buffer_days:.1f}d\n" +
           "- Available: {available}\n- Daily Burn: {daily_burn}\n" +
           "## Steps\n" +
           "1. Fund wallet >= {daily_burn*14} (14d min)\n" +
           "2. systemctl start loop-heartbeat\n" +
           "3. Verify Supabase + resume loop-trade when buffer > 7d\n" +
           "## Data\n" +
           "- Snapshot: {snapshot_id}\n- Robot Fund: query treasury_allocations"
```

## 9. Key Parameters

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| u_pnl weight | 0.5x | Conservative; positions can reverse |
| receivable weight | 0.8x | Collection risk discount |
| hysteresis | 1.2x | Prevents tier oscillation |
| eval frequency | 6h | Balances responsiveness vs cost |
| grace periods | downward only | Upward = immediate recovery |
| robot fund | sacred | Hardware-only; never ops |
| trading priority | always-on | Engine exists to keep trading alive |
| LLM downgrade | 4o -> 4o-mini -> local -> none | Cost-ordered |
