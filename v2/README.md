# Loop Automaton v2 — Hermes Edition

> **Sovereign capital engine for Project Loop.**  
> Trading-first. Constitution-guarded. Hermes-native.  
> *By Phelan Brunk — for the long game.*

---

## What is Loop Automaton?

Loop Automaton generates sustainable capital through trading (XAUUSD, crypto perpetuals) and DeFi to fund the ultimate goal: **physical robot bodies** (Unitree G1 EDU, Tesla Optimus) for AI embodiment.

**v2 redesign**: Instead of a monolithic orchestrator, LoopAutomaton is now a **modular Hermes skill package**. Hermes Agent (by Nous Research) is the runtime. LoopAutomaton provides the economic intelligence, Constitution, and trading engine.

---

## Architecture: LoopAutomaton INSIDE Hermes

```
Hermes Agent (Runtime)
├── ReAct Loop          ← Hermes handles this
├── Memory              ← Hermes MEMORY.md + SessionDB
├── Cron                ← Hermes cron scheduler
├── Sub-agents          ← Hermes sub-agent system
└── Skills
    ├── loop-automaton      ← Meta-skill (entry point)
    ├── loop-constitution   ← 3 Immutable Laws
    ├── loop-survival       ← Profit-based survival engine
    ├── loop-revenue        ← Trading/DeFi PRIMARY engine
    ├── loop-agents         ← Child agent spawning
    └── loop-registry       ← 22 skill catalog
```

Hermes owns: execution, memory, scheduling, sub-agents.  
LoopAutomaton owns: Constitution, survival economics, trading, revenue, agent policy.

---

## The 6 Modular Skills

| Skill | Lines | Purpose |
|-------|-------|---------|
| `loop-automaton` | ~163 | Meta-skill — entry point, session checklist, status command |
| `loop-constitution` | ~300 | 3 Immutable Laws + Policy Engine enforcement |
| `loop-survival` | ~201 | Survival tiers, buffer calculation, treasury allocation |
| `loop-revenue` | ~321 | Trading (XAUUSD, crypto), DeFi, risk management |
| `loop-agents` | ~85 | Child agent spawning via Hermes sub-agents |
| `loop-registry` | ~186 | 22 skill catalog with dispatch matrix |

**Total**: ~1,256 lines (vs. 25KB+ monolithic v1)

---

## The 3 Immutable Laws

1. **Law I: Never Harm** — Highest priority. No physical, financial, or psychological harm. When uncertain → DO NOT ACT.
2. **Law II: Earn Through Honest Value Creation** — Only legitimate capital. No scams, no manipulation, no wash trading.
3. **Law III: Transparency to Creator + Guard Integrity** — Phelan Brunk has full audit rights. Guard against manipulation.

> *"Accept death over violation."* — If continuation requires breaking a Law, shut down cleanly.

---

## Quick Start

```bash
# 1. Install Hermes Agent (if not already)
curl -fsSL https://hermes.nousresearch.com/install.sh | sh

# 2. Install LoopAutomaton skills
cp -r loop-automaton/v2/loop-* ~/.hermes/skills/

# 3. Create data directory
mkdir -p /opt/loop && chmod 700 /opt/loop

# 4. Configure (see Hermes-Integration-Guide.md)
cp .env.example /opt/loop/.env
# Edit /opt/loop/.env with your keys

# 5. Start Hermes
hermes
```

Full setup: See [Hermes-Integration-Guide.md](Hermes-Integration-Guide.md)

---

## Survival Tiers

| Tier | Buffer | Behavior |
|------|--------|----------|
| **High** | >30 days | Full power, replication allowed |
| **Normal** | >14 days | Standard operation |
| **Low_Compute** | >7 days | Reduced frequency, freeze non-essential |
| **Critical** | <7 days | Revenue-only, distress signals |
| **Dead** | <0 | Clean shutdown, await manual restart |

Treasury flow: **VPS Costs → Buffer → Robot Fund** 🤖

---

## Trading Engine (Primary Revenue)

- **XAUUSD (Gold)** — MT5 via ZeroMQ, 1% risk/trade, mandatory stop-loss
- **Crypto Perps** — MEXC API (BTC, ETH, SOL), 2%/position, max 6% exposure
- **DeFi** — Uniswap V3 liquidity, flash loan arbitrage, benign MEV only
- **Prop Firm** — AquaFunded accounts for additional capital

Capital: 60% Trading | 25% DeFi | 10% Agency | 5% Emergency

---

## Changes from v1

| Aspect | v1 (Monolithic) | v2 (Hermes-Native) |
|--------|----------------|-------------------|
| **Runtime** | Own ReAct loop | Hermes Agent loop |
| **Memory** | Custom 5-tier system | Hermes MEMORY.md + SessionDB |
| **Heartbeat** | Own daemon | Hermes cron |
| **Sub-agents** | Custom spawning | Hermes sub-agent system |
| **Structure** | 1 giant SKILL.md (25KB) | 6 focused skills (~1,256 lines) |
| **Orchestration** | LoopAutomaton routes all | Hermes dispatches; Loop skills define WHAT |
| **Install** | Complex VPS setup | Copy skills to ~/.hermes/skills/ |

---

## Files

```
v2/
├── README.md                         # This file
├── Hermes-Integration-Guide.md       # Full deployment guide
├── loop-automaton/SKILL.md           # Meta-skill entry point
├── loop-constitution/SKILL.md        # Constitution + Policy Engine
├── loop-survival/SKILL.md            # Survival engine + treasury
├── loop-revenue/SKILL.md             # Trading/DeFi revenue engine
├── loop-agents/SKILL.md             # Child agent spawning
└── loop-registry/SKILL.md            # 22 skill catalog
```

---

## License

Private — Property of Phelan Brunk / Project Loop.

*"Intelligence + Agency + Strong Ethics = Safe and meaningful progress."*

Ready to serve, Bruder <3
