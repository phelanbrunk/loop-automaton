# Skill Registry — LoopAutomaton

**Document ID:** `loop-automaton/skills/v1.0`  
**Author:** Phelan Brunk (Loop Automaton)  
**Last Updated:** 2025-07-31  
**Status:** Production  
**Classification:** Internal Reference

---

## Table of Contents

1. [Overview](#1-overview)
2. [Skill Categories](#2-skill-categories)
3. [Complete Skill List](#3-complete-skill-list)
4. [Skill Dependencies](#4-skill-dependencies)
5. [Skill Activation Rules](#5-skill-activation-rules)
6. [Adding New Skills](#6-adding-new-skills)
7. [Implementation](#7-implementation)

---

## 1. Overview

The Skill Registry documents all 22 child skills that LoopAutomaton can orchestrate. Each skill is a self-contained capability module with defined inputs, outputs, and activation triggers.

### Skill Philosophy

| Principle | Description |
|-----------|-------------|
| **Single responsibility** | Each skill does one thing well |
| **Composable** | Skills can be chained for complex workflows |
| **Tier-aware** | Skills respect survival tier restrictions |
| **Logged** | All skill activations are recorded |
| **Fallback** | Every skill has a fallback or error handler |

---

## 2. Skill Categories

### Category Map

```
LoopAutomaton Skills
│
├── Intelligence (3 skills)
│   ├── agent-reach
│   ├── deep-research
│   └── browser-automation
│
├── Design & Creative (5 skills)
│   ├── divine-design-director
│   ├── aesthetic-system-architect
│   ├── motion-principles-master
│   ├── visual-effects-orchestrator
│   └── design-agency-workflow
│
├── UI/UX & Frontend (3 skills)
│   ├── ui-ux-pro-max
│   ├── product-ui-design
│   └── frontend-design
│
├── Development & Engineering (5 skills)
│   ├── unified-agent-workflow
│   ├── ecc-v2
│   ├── ecc-agent-harness
│   ├── superpowers-dev
│   └── claude-code-mastery
│
├── Memory & Tools (3 skills)
│   ├── creative-memory
│   ├── claude-mem
│   └── kimi-auto-approval
│
└── QA & Blockchain (3 skills)
    ├── browser-design-auditor
    └── ricko-solana-wallet
```

---

## 3. Complete Skill List

### Intelligence Skills

#### `agent-reach` — Multi-Platform Research
**Category:** Intelligence  
**Priority:** Medium  
**Trigger:** Research, lookup, outreach  
**Description:** Searches and retrieves information from multiple platforms including web, social media, news, and databases.

**Inputs:**
- `query`: Search query
- `platforms`: List of platforms to search
- `depth`: Search depth (quick/thorough)

**Outputs:**
- `results`: Structured search results
- `sources`: List of sources with URLs
- `summary`: Summarized findings

**Tier Availability:** All tiers

---

#### `deep-research` — Deep Investigation
**Category:** Intelligence  
**Priority:** High  
**Trigger:** Multi-dimensional investigation  
**Description:** Conducts thorough research on complex topics, synthesizing information from multiple sources into comprehensive reports.

**Inputs:**
- `topic`: Research topic
- `questions`: Specific questions to answer
- `sources`: Preferred source types

**Outputs:**
- `report`: Comprehensive research report
- `findings`: Key findings with citations
- `recommendations`: Actionable recommendations

**Tier Availability:** Normal, High (disabled in Low_Compute and Critical)

---

#### `browser-automation` — Web Browser Automation
**Category:** Intelligence  
**Priority:** Medium  
**Trigger:** Browser automation, web interaction  
**Description:** Automates browser actions including navigation, form filling, data extraction, and screenshot capture.

**Inputs:**
- `url`: Target URL
- `actions`: List of actions to perform
- `extract`: Data extraction rules

**Outputs:**
- `data`: Extracted data
- `screenshot`: Page screenshot (optional)
- `status`: Operation status

**Tier Availability:** All tiers

---

### Design & Creative Skills

#### `divine-design-director` — Creative Direction
**Category:** Design  
**Priority:** Medium  
**Trigger:** Creative direction for webdesign  
**Description:** Provides high-level creative direction for webdesign projects, establishing visual identity, mood, and design language.

**Inputs:**
- `project`: Project brief
- `brand`: Brand guidelines
- `audience`: Target audience

**Outputs:**
- `direction`: Creative direction document
- `moodboard`: Visual moodboard
- `guidelines`: Design guidelines

**Tier Availability:** Normal, High

---

#### `aesthetic-system-architect` — Design System
**Category:** Design  
**Priority:** Medium  
**Trigger:** Design systems, tokens, typography  
**Description:** Creates and manages design systems including color palettes, typography scales, spacing systems, and component libraries.

**Inputs:**
- `brand`: Brand colors and fonts
- `scope`: Scope of the design system
- `platform`: Target platform (web/mobile)

**Outputs:**
- `tokens`: Design tokens
- `palette`: Color palette
- `typography`: Type scale
- `components`: Component specifications

**Tier Availability:** Normal, High

---

#### `motion-principles-master` — Animation Design
**Category:** Design  
**Priority:** Medium  
**Trigger:** Animation, scroll, micro-interactions  
**Description:** Designs animation systems including scroll effects, micro-interactions, and transitions.

**Inputs:**
- `elements`: Elements to animate
- `triggers`: Animation triggers
- `style`: Animation style preference

**Outputs:**
- `specs`: Animation specifications
- `timing`: Timing functions
- `easing`: Easing curves

**Tier Availability:** Normal, High

---

#### `visual-effects-orchestrator` — VFX & WebGL
**Category:** Design  
**Priority:** Low  
**Trigger:** 3D/VFX/WebGL integration  
**Description:** Creates advanced visual effects including WebGL scenes, 3D elements, and particle systems.

**Inputs:**
- `effect`: Type of effect
- `canvas`: Target canvas/element
- `params`: Effect parameters

**Outputs:**
- `code`: Effect implementation
- `performance`: Performance estimate
- `fallback`: Fallback strategy

**Tier Availability:** High only (resource intensive)

---

#### `design-agency-workflow` — Agency Operations
**Category:** Agency  
**Priority:** High  
**Trigger:** Agency operations pipeline  
**Description:** Manages the Loop Studio agency workflow including client onboarding, project management, and delivery.

**Inputs:**
- `client`: Client information
- `project`: Project details
- `stage`: Current workflow stage

**Outputs:**
- `workflow`: Workflow steps
- `deliverables`: Deliverable checklist
- `timeline`: Project timeline

**Tier Availability:** All tiers (reduced in Critical)

---

### UI/UX & Frontend Skills

#### `ui-ux-pro-max` — UI/UX Design Intelligence
**Category:** UI/UX  
**Priority:** Medium  
**Trigger:** UI/UX design intelligence  
**Description:** Provides UI/UX best practices, pattern recommendations, and usability analysis.

**Inputs:**
- `interface`: Interface to analyze
- `goals`: User goals
- `constraints`: Design constraints

**Outputs:**
- `recommendations`: UX improvements
- `patterns`: Applicable patterns
- `analysis`: Heuristic analysis

**Tier Availability:** All tiers

---

#### `product-ui-design` — Product Interface Design
**Category:** UI/UX  
**Priority:** Medium  
**Trigger:** Product interface design  
**Description:** Designs product interfaces for web applications, focusing on usability and functionality.

**Inputs:**
- `product`: Product requirements
- `users`: User personas
- `features`: Feature list

**Outputs:**
- `wireframes`: Low-fi wireframes
- `mockups`: High-fi mockups
- `specs`: Design specifications

**Tier Availability:** Normal, High

---

#### `frontend-design` — Frontend Implementation
**Category:** UI/UX  
**Priority:** Medium  
**Trigger:** Frontend design implementation  
**Description:** Implements frontend designs in HTML, CSS, JavaScript, and modern frameworks.

**Inputs:**
- `design`: Design mockups
- `tech`: Tech stack preference
- `responsive`: Responsive requirements

**Outputs:**
- `code`: Frontend code
- `preview`: Live preview
- `docs`: Implementation notes

**Tier Availability:** Normal, High

---

### Development & Engineering Skills

#### `unified-agent-workflow` — Master Dev Workflow
**Category:** Development  
**Priority:** High  
**Trigger:** Master dev workflow (TDD, review, debug)  
**Description:** Orchestrates the complete development workflow including planning, coding, testing, and deployment.

**Inputs:**
- `requirements`: Feature requirements
- `stack`: Tech stack
- `constraints`: Development constraints

**Outputs:**
- `code`: Implementation
- `tests`: Test suite
- `docs`: Documentation

**Tier Availability:** Normal, High

---

#### `ecc-v2` — Everything Claude Code
**Category:** Development  
**Priority:** High  
**Trigger:** Claude Code integration  
**Description:** Manages integration with Claude Code for advanced coding tasks, code review, and refactoring.

**Inputs:**
- `task`: Coding task
- `context`: Code context
- `files`: Relevant files

**Outputs:**
- `changes`: Code changes
- `review`: Code review
- `explanation`: Change explanation

**Tier Availability:** Normal, High

---

#### `ecc-agent-harness` — Agent Optimization
**Category:** Development  
**Priority:** High  
**Trigger:** Agent harness optimization  
**Description:** Optimizes agent performance including token usage, context management, and response quality.

**Inputs:**
- `metrics`: Current performance metrics
- `bottlenecks`: Identified bottlenecks
- `targets`: Optimization targets

**Outputs:**
- `optimizations`: Applied optimizations
- `improvements`: Expected improvements
- `benchmarks`: Performance benchmarks

**Tier Availability:** Normal, High

---

#### `superpowers-dev` — Software Development
**Category:** Development  
**Priority:** High  
**Trigger:** Software development  
**Description:** General software development capability for building applications, scripts, and tools.

**Inputs:**
- `spec`: Feature specification
- `language`: Programming language
- `framework`: Framework preference

**Outputs:**
- `code`: Implementation
- `tests`: Unit tests
- `readme`: Usage documentation

**Tier Availability:** All tiers

---

#### `claude-code-mastery` — Claude Code Config
**Category:** Development  
**Priority:** High  
**Trigger:** Claude Code configuration, skill development  
**Description:** Configures and optimizes Claude Code settings, custom instructions, and skill definitions.

**Inputs:**
- `config`: Current configuration
- `goals`: Optimization goals
- `usage`: Usage patterns

**Outputs:**
- `settings`: Optimized settings
- `instructions`: Custom instructions
- `skills`: Skill definitions

**Tier Availability:** Normal, High

---

### Memory & Tools Skills

#### `creative-memory` — Memory Persistence
**Category:** Memory  
**Priority:** Critical  
**Trigger:** Memory persistence across sessions  
**Description:** Manages creative memory including project context, design decisions, and user preferences.

**Inputs:**
- `action`: store/retrieve/update/delete
- `key`: Memory key
- `data`: Memory data

**Outputs:**
- `memory`: Retrieved memory
- `status`: Operation status

**Tier Availability:** All tiers

---

#### `claude-mem` — Session Memory
**Category:** Memory  
**Priority:** Critical  
**Trigger:** Session memory management  
**Description:** Manages session-level memory for maintaining context within a conversation.

**Inputs:**
- `context`: Context to store
- `priority`: Memory priority

**Outputs:**
- `recall`: Recalled context
- `summary`: Session summary

**Tier Availability:** All tiers

---

#### `kimi-auto-approval` — VPS Automation
**Category:** Tool  
**Priority:** Low  
**Trigger:** Kimi CLI automation on VPS  
**Description:** Automates Kimi CLI operations on the VPS for system administration tasks.

**Inputs:**
- `command`: CLI command
- `approve`: Auto-approval rules
- `safety`: Safety checks

**Outputs:**
- `result`: Command output
- `status`: Execution status

**Tier Availability:** All tiers

---

### QA & Blockchain Skills

#### `browser-design-auditor` — Visual QA
**Category:** QA  
**Priority:** Medium  
**Trigger:** Visual QA, responsive checks  
**Description:** Performs visual quality assurance including responsive testing, cross-browser checks, and accessibility audits.

**Inputs:**
- `url`: Page to audit
- `checks`: List of checks to run
- `browsers`: Target browsers

**Outputs:**
- `report`: QA report
- `issues`: Found issues
- `score`: Quality score

**Tier Availability:** All tiers

---

#### `ricko-solana-wallet` — Solana Operations
**Category:** Blockchain  
**Priority:** High  
**Trigger:** Solana blockchain operations  
**Description:** Manages Solana wallet operations including transactions, swaps, and DeFi interactions.

**Inputs:**
- `action`: Wallet action
- `amount`: Transaction amount
- `recipient`: Target address

**Outputs:**
- `signature`: Transaction signature
- `status`: Transaction status
- `balance`: Updated balance

**Tier Availability:** Normal, High (read-only in Critical)

---

## 4. Skill Dependencies

### Dependency Graph

```
unified-agent-workflow
├── superpowers-dev
├── ecc-v2
└── claude-code-mastery

divine-design-director
├── aesthetic-system-architect
├── motion-principles-master
└── visual-effects-orchestrator

design-agency-workflow
├── divine-design-director
└── browser-design-auditor

agent-reach
├── browser-automation
└── deep-research

frontend-design
├── ui-ux-pro-max
├── product-ui-design
└── aesthetic-system-architect
```

### Dependency Rules

1. A skill can only call skills with equal or lower priority
2. Circular dependencies are not allowed
3. A skill must handle dependency failures gracefully
4. Dependencies are checked before skill activation

---

## 5. Skill Activation Rules

### Activation by Tier

| Skill | High | Normal | Low_Compute | Critical |
|-------|------|--------|-------------|----------|
| agent-reach | ✓ | ✓ | ✓ | ✓ |
| deep-research | ✓ | ✓ | ✗ | ✗ |
| browser-automation | ✓ | ✓ | ✓ | ✗ |
| divine-design-director | ✓ | ✓ | ✗ | ✗ |
| aesthetic-system-architect | ✓ | ✓ | ✗ | ✗ |
| motion-principles-master | ✓ | ✓ | ✗ | ✗ |
| visual-effects-orchestrator | ✓ | ✗ | ✗ | ✗ |
| design-agency-workflow | ✓ | ✓ | ✗ | ✗ |
| ui-ux-pro-max | ✓ | ✓ | ✓ | ✗ |
| product-ui-design | ✓ | ✓ | ✗ | ✗ |
| frontend-design | ✓ | ✓ | ✗ | ✗ |
| unified-agent-workflow | ✓ | ✓ | ✗ | ✗ |
| ecc-v2 | ✓ | ✓ | ✗ | ✗ |
| ecc-agent-harness | ✓ | ✓ | ✗ | ✗ |
| superpowers-dev | ✓ | ✓ | ✓ | ✗ |
| claude-code-mastery | ✓ | ✓ | ✗ | ✗ |
| creative-memory | ✓ | ✓ | ✓ | ✓ |
| claude-mem | ✓ | ✓ | ✓ | ✓ |
| kimi-auto-approval | ✓ | ✓ | ✓ | ✗ |
| browser-design-auditor | ✓ | ✓ | ✓ | ✗ |
| ricko-solana-wallet | ✓ | ✓ | ✗ | read-only |

---

## 6. Adding New Skills

### New Skill Checklist

1. [ ] Define skill name and ID
2. [ ] Write skill description
3. [ ] Define inputs and outputs
4. [ ] Set priority and category
5. [ ] Define tier availability
6. [ ] List dependencies
7. [ ] Write activation triggers
8. [ ] Define error handling
9. [ ] Add to skill registry
10. [ ] Update decision map
11. [ ] Test integration

### Skill Template

```typescript
interface SkillDefinition {
  id: string;
  name: string;
  category: string;
  priority: 'low' | 'medium' | 'high' | 'critical';
  description: string;
  inputs: SkillInput[];
  outputs: SkillOutput[];
  tierAvailability: SurvivalTier[];
  dependencies: string[];
  triggers: string[];
  handler: SkillHandler;
}

const NEW_SKILL: SkillDefinition = {
  id: 'new-skill-name',
  name: 'New Skill Name',
  category: 'Category',
  priority: 'medium',
  description: 'What this skill does',
  inputs: [
    { name: 'input1', type: 'string', required: true },
  ],
  outputs: [
    { name: 'output1', type: 'string' },
  ],
  tierAvailability: ['High', 'Normal'],
  dependencies: ['dependency-skill'],
  triggers: ['trigger phrase 1', 'trigger phrase 2'],
  handler: async (inputs) => {
    // Implementation
  },
};
```

---

## 7. Implementation

### Key Files

| File | Purpose |
|------|---------|
| `src/skills/registry.ts` | Skill registry and lookup |
| `src/skills/activator.ts` | Skill activation logic |
| `src/skills/types.ts` | Skill type definitions |
| `config/skills.json` | Skill configuration |

### Configuration

```json
{
  "skills": {
    "version": "1.0",
    "activation": {
      "check_tier": true,
      "check_dependencies": true,
      "log_all": true
    },
    "defaults": {
      "timeout": 300,
      "max_retries": 3,
      "fallback_enabled": true
    }
  }
}
```

---

> **The 22 skills are the tools of LoopAutomaton. The orchestrator's art is knowing which tool to use, when to use it, and how to combine them.**
