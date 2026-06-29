---
name: loop-registry
description: >
  Load when: (1) skill discovery needed, (2) skill composition planning,
  (3) which skill to use for a task, (4) skill dependencies,
  (5) user asks "which skill", "what skills", "skill list", or needs to
  know what capabilities exist. Complete registry of all 22 Loop Automaton
  child skills with triggers, dependencies, and composition patterns.
triggers:
  - "which skill"
  - "what skills"
  - "skill list"
  - "skill discovery"
  - "skill composition"
  - "what can you do"
  - "capabilities"
  - "skill registry"
  - "skill map"
  - "how to"
  - "which tool"
  - "what tool"
  - skill selection
  - skill planning
  - multi-skill workflow
  - orchestration plan
  - skill dispatch
  - dependency chain
  - capability query
  - feature matrix
version: "2.0.0"
standard: "agentskills.io"
author: "loop-automaton"
dependencies: []
---

# Skill Registry — Loop Automaton v2

> Hermes-compatible dispatch reference. 22 skills · 5 domains · 4 tiers.

---

## 1. Skill Dispatch Matrix

| Intent | Primary Skill | Backup Skill |
|---|---|---|
| Build a website / landing page / portfolio | divine-design-director | frontend-design |
| Research a topic / market / trend | deep-research | agent-reach |
| Fix a bug / debug code | unified-agent-workflow | superpowers-dev |
| Check SOL balance / swap tokens | ricko-solana-wallet | — |
| Design a dashboard / admin panel / SaaS UI | product-ui-design | ui-ux-pro-max |
| Create a design system / tokens / palette | aesthetic-system-architect | divine-design-director |
| Add animations / scroll effects / transitions | gsap-animation | motion-principles-master |
| 3D scene / WebGL / Spline / Lottie | visual-effects-orchestrator | — |
| Automate browser / scrape data / test | browser-automation | agent-reach |
| Client project / agency workflow / handoff | design-agency-workflow | divine-design-director |
| Code review / TDD / CI workflow | unified-agent-workflow | ecc-v2 |
| Optimize agent / token budget / harness | ecc-agent-harness | claude-code-mastery |
| Write a skill / context engineering | claude-code-mastery | ecc-v2 |
| Session memory / context compression | claude-mem | creative-memory |
| VPS automation / CLI approval config | kimi-auto-approval | — |
| Visual QA / responsive check / a11y audit | browser-design-auditor | ui-ux-pro-max |
| Competitive analysis / due diligence | deep-research | agent-reach |
| Social media research / video transcripts | agent-reach | deep-research |
| Feature implementation / systematic build | superpowers-dev | unified-agent-workflow |
| Security audit / hardening | ecc-v2 | unified-agent-workflow |
| NFT operations / minting | ricko-solana-wallet | — |
| Micro-interactions / motion polish | motion-principles-master | gsap-animation |

---

## 2. Composition Patterns

### A. Client Website (Full Pipeline)
```
design-agency-workflow → divine-design-director → aesthetic-system-architect
  → frontend-design → browser-design-auditor
```

### B. Research + Trade (DeFi Intelligence)
```
agent-reach → deep-research → ricko-solana-wallet
```

### C. Debug Production Incident
```
unified-agent-workflow → superpowers-dev → ecc-v2
```

### D. Fullstack SaaS App
```
unified-agent-workflow → product-ui-design → frontend-design → superpowers-dev
```

### E. Immersive Experience (3D + Motion)
```
divine-design-director → visual-effects-orchestrator → motion-principles-master
  → frontend-design → browser-design-auditor
```

### F. Agent Optimization Sprint
```
ecc-agent-harness → claude-code-mastery → ecc-v2 → claude-mem
```

### G. Research Report (Multi-Source)
```
agent-reach (web+social+video) → deep-research (analysis) → browser-automation (export)
```

### H. Design System Build
```
aesthetic-system-architect → divine-design-director → product-ui-design
  → browser-design-auditor
```

### I. Trading Bot Setup
```
kimi-auto-approval → ricko-solana-wallet → agent-reach (price feeds)
```

### J. Landing Page (Quick)
```
divine-design-director → frontend-design → gsap-animation → browser-design-auditor
```

---

## 3. Per-Skill Summary

| # | Skill | Domain | Trigger | Priority | Dependencies |
|---|-------|--------|---------|----------|--------------|
| 1 | agent-reach | Research & Intelligence | web search, social media, stock info, video transcripts | high | — |
| 2 | deep-research | Research & Intelligence | research, trend analysis, competitive analysis | high | — |
| 3 | browser-automation | Research & Intelligence | web automation, data extraction, testing | medium | — |
| 4 | divine-design-director | Design & Creative | any webdesign request | critical | — |
| 5 | aesthetic-system-architect | Design & Creative | colors, fonts, design tokens | medium | divine-design-director |
| 6 | motion-principles-master | Design & Creative | scroll effects, transitions, micro-interactions | medium | — |
| 7 | visual-effects-orchestrator | Design & Creative | 3D, WebGL, Spline, Lottie | low | — |
| 8 | design-agency-workflow | Design & Creative | project intake, client presentation, handoff | medium | divine-design-director |
| 9 | creative-memory | Design & Creative | session continuity, context search | low | — |
| 10 | ui-ux-pro-max | UI/UX & Frontend | UI design, component design, UX review | high | — |
| 11 | product-ui-design | UI/UX & Frontend | dashboards, admin panels, SaaS apps | high | ui-ux-pro-max |
| 12 | frontend-design | UI/UX & Frontend | landing pages, portfolios, marketing sites | high | divine-design-director |
| 13 | gsap-animation | UI/UX & Frontend | JS animations, scroll-triggered effects | medium | frontend-design |
| 14 | unified-agent-workflow | Development & Engineering | TDD, code review, debugging | critical | — |
| 15 | ecc-v2 | Development & Engineering | agent optimization, security audits | high | unified-agent-workflow |
| 16 | ecc-agent-harness | Development & Engineering | token optimization, subagent selection | medium | ecc-v2 |
| 17 | superpowers-dev | Development & Engineering | feature implementation, systematic debugging | high | unified-agent-workflow |
| 18 | claude-code-mastery | Development & Engineering | skill creation, context engineering | medium | ecc-v2 |
| 19 | claude-mem | Development & Engineering | context management, session persistence | low | — |
| 20 | kimi-auto-approval | Automation & Blockchain | VPS automation, approval config | medium | — |
| 21 | ricko-solana-wallet | Automation & Blockchain | SOL transfers, Jupiter swaps, NFTs | critical | — |
| 22 | browser-design-auditor | Automation & Blockchain | responsive check, accessibility audit | medium | frontend-design |

---

## 4. Tier-Based Availability

| Tier | Active Skills | Notes |
|------|--------------|-------|
| **High** | All 22 + loop-revenue (trading execution) | Full power. All domains, all compositions. |
| **Normal** | All 22 | Standard operations. No trading execution skill. |
| **Low_Compute** | 15 core (excludes visual-effects-orchestrator, motion-principles-master, creative-memory, browser-automation, ecc-agent-harness, claude-mem, kimi-auto-approval) | Reduced creative + automation. Research, design, dev, trading core intact. |
| **Critical** | 8 essential: ricko-solana-wallet, unified-agent-workflow, divine-design-director, product-ui-design, frontend-design, deep-research, ui-ux-pro-max, superpowers-dev | Survival mode. Trading + build + research only. |

---

## 5. Quick Reference

### Domain Map
- **Research & Intelligence**: `agent-reach` · `deep-research` · `browser-automation`
- **Design & Creative**: `divine-design-director` · `aesthetic-system-architect` · `motion-principles-master` · `visual-effects-orchestrator` · `design-agency-workflow` · `creative-memory`
- **UI/UX & Frontend**: `ui-ux-pro-max` · `product-ui-design` · `frontend-design` · `gsap-animation`
- **Development & Engineering**: `unified-agent-workflow` · `ecc-v2` · `ecc-agent-harness` · `superpowers-dev` · `claude-code-mastery` · `claude-mem`
- **Automation & Blockchain**: `kimi-auto-approval` · `ricko-solana-wallet` · `browser-design-auditor`

### Singleton Skills (no backup)
| Skill | Why no backup |
|-------|--------------|
| ricko-solana-wallet | Sole blockchain operations skill |
| visual-effects-orchestrator | No other 3D/WebGL skill |
| kimi-auto-approval | Sole CLI automation skill |

---

*Registry version 2.0.0 · 22 skills · 5 domains · agentskills.io standard*
