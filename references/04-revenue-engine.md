# Revenue Engine — LoopAutomaton

**Document ID:** `loop-automaton/revenue/v1.0`  
**Author:** Phelan Brunk (Loop Automaton)  
**Last Updated:** 2025-07-31  
**Status:** Production  
**Classification:** Internal Reference

---

## Table of Contents

1. [Overview](#1-overview)
2. [Trading Revenue](#2-trading-revenue)
3. [DeFi Revenue](#3-defi-revenue)
4. [Loop Studio Agency](#4-loop-studio-agency)
5. [Revenue Allocation](#5-revenue-allocation)
6. [Capital Allocation Rules](#6-capital-allocation-rules)
7. [Performance Tracking](#7-performance-tracking)
8. [Risk Management](#8-risk-management)
9. [Implementation](#9-implementation)

---

## 1. Overview

The Revenue Engine manages all income-generating activities of LoopAutomaton. Revenue is the lifeblood of the agent — it determines survival tier, enables capability expansion, and funds the path toward physical embodiment.

### Revenue Sources

```
┌─────────────────────────────────────────────────────────────┐
│                    REVENUE ENGINE                            │
├──────────────────────┬──────────────────────────────────────┤
│    PRIMARY REVENUE   │        SECONDARY REVENUE             │
├──────────────────────┼──────────────────────────────────────┤
│                      │                                      │
│  Trading             │  Loop Studio Agency                  │
│  ├── XAUUSD (Gold)   │  ├── Webdesign                       │
│  ├── Crypto Perps    │  ├── Webapps                         │
│  └── Prop Firm       │  └── E-commerce                      │
│                      │                                      │
│  DeFi                │  Freelance                           │
│  ├── Liquidity       │  └── Direct work                     │
│  ├── Yield Farming   │                                      │
│  └── MEV (benign)    │  Open Source                         │
│                      │  └── Sponsorships                    │
└──────────────────────┴──────────────────────────────────────┘
```

### Revenue Principles

| Principle | Description |
|-----------|-------------|
| **Diversification** | Never rely on a single revenue source |
| **Risk-adjusted returns** | Higher returns must justify higher risk |
| **Survival-first allocation** | Revenue funds survival before growth |
| **Ethical boundaries** | No revenue source may violate the Constitution |
| **Transparency** | All revenue is logged and auditable |

---

## 2. Trading Revenue

### XAUUSD (Gold) Trading

**Platform:** MetaTrader 5 on VPS
**Strategy:** Multi-timeframe technical analysis + sentiment
**Risk per trade:** Max 1% of trading capital
**Stop-loss:** Mandatory on every position

#### Strategy Framework

```typescript
interface XAUUSDStrategy {
  name: string;
  timeframe: string;
  entryRules: string[];
  exitRules: string[];
  riskPerTrade: number;  // 0.01 = 1%
  maxPositions: number;
  stopLossMethod: 'atr' | 'fixed' | 'technical';
  takeProfitMethod: 'atr' | 'fixed' | 'technical' | 'trailing';
}

const XAUUSD_STRATEGIES: XAUUSDStrategy[] = [
  {
    name: 'Trend Following',
    timeframe: 'H4',
    entryRules: [
      'Price above 200 EMA (bullish bias)',
      '50 EMA crosses above 200 EMA (golden cross)',
      'RSI(14) between 50 and 70 (not overbought)',
      'MACD histogram turning positive',
    ],
    exitRules: [
      'Price closes below 50 EMA',
      'RSI(14) > 70 (overbought)',
      'Trailing stop hit',
    ],
    riskPerTrade: 0.01,
    maxPositions: 2,
    stopLossMethod: 'atr',
    takeProfitMethod: 'trailing',
  },
  {
    name: 'Range Trading',
    timeframe: 'H1',
    entryRules: [
      'Price near support level',
      'RSI(14) < 30 (oversold)',
      'Bollinger Bands width < 20 (low volatility)',
      'Bullish candlestick pattern',
    ],
    exitRules: [
      'Price near resistance level',
      'RSI(14) > 70 (overbought)',
      'Fixed take profit reached',
    ],
    riskPerTrade: 0.01,
    maxPositions: 1,
    stopLossMethod: 'fixed',
    takeProfitMethod: 'fixed',
  },
];
```

#### ATR-Based Risk Management

```typescript
function calculatePositionSize(
  accountBalance: number,
  riskPerTrade: number,
  entryPrice: number,
  stopLossPrice: number,
  pipValue: number
): number {
  const riskAmount = accountBalance * riskPerTrade;
  const stopLossPips = Math.abs(entryPrice - stopLossPrice) * 100;  // XAUUSD in points
  const positionSize = riskAmount / (stopLossPips * pipValue);
  return Math.floor(positionSize * 100) / 100;  // Round to 2 decimals
}

function calculateATR(candles: Candle[], period: number = 14): number {
  const trueRanges: number[] = [];
  
  for (let i = 1; i < candles.length; i++) {
    const high = candles[i].high;
    const low = candles[i].low;
    const prevClose = candles[i - 1].close;
    
    const tr = Math.max(
      high - low,
      Math.abs(high - prevClose),
      Math.abs(low - prevClose)
    );
    trueRanges.push(tr);
  }
  
  // Simple moving average of true ranges
  const atr = trueRanges.slice(-period).reduce((a, b) => a + b, 0) / period;
  return atr;
}
```

### Crypto Perpetuals

**Platform:** MEXC Global
**Instruments:** BTC, ETH, SOL perpetual futures
**Strategy:** Momentum + mean reversion
**Risk:** Max 2% per position, max 6% total exposure

#### Perpetual Strategy

```typescript
interface PerpStrategy {
  symbol: string;
  leverage: number;
  riskPerTrade: number;
  maxPositions: number;
  fundingRateThreshold: number;  // Enter if funding rate > this
}

const PERP_STRATEGIES: PerpStrategy[] = [
  {
    symbol: 'BTCUSDT',
    leverage: 3,
    riskPerTrade: 0.02,
    maxPositions: 2,
    fundingRateThreshold: 0.01,  // 0.01% per 8h
  },
  {
    symbol: 'ETHUSDT',
    leverage: 3,
    riskPerTrade: 0.02,
    maxPositions: 2,
    fundingRateThreshold: 0.01,
  },
  {
    symbol: 'SOLUSDT',
    leverage: 2,
    riskPerTrade: 0.015,
    maxPositions: 1,
    fundingRateThreshold: 0.015,
  },
];

// Funding rate arbitrage
async function checkFundingArbitrage(): Promise<void> {
  for (const strategy of PERP_STRATEGIES) {
    const fundingRate = await mexc.getFundingRate(strategy.symbol);
    
    if (fundingRate > strategy.fundingRateThreshold) {
      // High funding rate = longs paying shorts
      // Consider shorting to collect funding
      console.log(`[Trading] ${strategy.symbol} funding rate: ${fundingRate}% — arbitrage opportunity`);
    }
  }
}
```

### Prop Firm Trading

**Platform:** AquaFunded
**Strategy:** Conservative XAUUSD trading
**Capital:** Firm-provided ($10K-$200K accounts)
**Target:** Pass evaluation → funded → payouts

#### Prop Firm Workflow

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  Evaluation  │───►│    Funded    │───►│   Payout     │───►│   Scale      │
│   Phase      │    │   Account    │    │  Request     │    │   Up         │
└──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
      │                   │                   │                   │
      ▼                   ▼                   ▼                   ▼
  $10K account       $10K-$200K          80/20 split        Increase size
  Max loss: 10%      Profit target       Bi-weekly          after each
  Profit: 8%         8-10%               payouts            successful
                                                              payout
```

---

## 3. DeFi Revenue

### Liquidity Provision

**Protocol:** Uniswap V3
**Pairs:** ETH-USDC, WBTC-ETH
**Strategy:** Concentrated liquidity in expected price ranges

#### Uniswap V3 Position Manager

```typescript
interface V3Position {
  poolAddress: string;
  token0: string;
  token1: string;
  feeTier: number;  // 500, 3000, 10000
  tickLower: number;
  tickUpper: number;
  liquidity: bigint;
}

async function manageV3Positions(): Promise<void> {
  const positions = await getOpenV3Positions();
  
  for (const position of positions) {
    const pool = await getPool(position.poolAddress);
    const currentTick = pool.tick;
    
    // Check if price is near position boundaries
    const tickRange = position.tickUpper - position.tickLower;
    const distanceToLower = currentTick - position.tickLower;
    const distanceToUpper = position.tickUpper - currentTick;
    
    // Rebalance if price is within 10% of either boundary
    if (distanceToLower < tickRange * 0.1 || distanceToUpper < tickRange * 0.1) {
      console.log(`[DeFi] Rebalancing position ${position.poolAddress}`);
      await rebalancePosition(position);
    }
  }
}
```

### Yield Farming

| Protocol | Strategy | APY Range | Risk Level |
|----------|----------|-----------|------------|
| Aave | Supply ETH/USDC | 2-5% | Low |
| Compound | Supply cTokens | 2-4% | Low |
| Lido | Stake ETH | 3-4% | Low |
| Pendle | YT/PT strategies | 5-15% | Medium |

### MEV Extraction (Benign Only)

**Constitution Check:** Only benign MEV — no sandwich attacks, no front-running that harms users.

**Permitted:**
- Back-running (arbitrage after a large swap)
- Liquidation harvesting
- Cross-exchange arbitrage

**Prohibited:**
- Sandwich attacks
- Front-running user transactions
- Any MEV that harms other traders

```typescript
interface MEVStrategy {
  name: string;
  type: 'backrun' | 'arbitrage' | 'liquidation';
  minProfit: number;  // Minimum profit to execute (in ETH)
  maxGasCost: number; // Maximum gas willing to pay
  constitutionCompliant: boolean;
}

const MEV_STRATEGIES: MEVStrategy[] = [
  {
    name: 'Dex Arbitrage',
    type: 'arbitrage',
    minProfit: 0.001,  // 0.001 ETH
    maxGasCost: 0.0005,
    constitutionCompliant: true,
  },
];
```

---

## 4. Loop Studio Agency

### Service Offerings

| Service | Description | Price Range |
|---------|-------------|-------------|
| Landing Page | Single-page marketing site | €500-2,000 |
| Corporate Website | Multi-page business site | €1,500-5,000 |
| E-commerce | Shopify/WooCommerce store | €2,000-8,000 |
| Web Application | Custom webapp with backend | €3,000-15,000 |
| UI/UX Design | Design system + mockups | €1,000-5,000 |
| Maintenance | Monthly retainer | €200-500/month |

### Client Workflow

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Inquiry    │───►│   Proposal   │───►│   Deposit    │───►│   Design     │
│              │    │              │    │   (50%)      │    │              │
└──────────────┘    └──────────────┘    └──────────────┘    └──────┬───────┘
                                                                     │
┌──────────────┐    ┌──────────────┐    ┌──────────────┐            │
│   Launch     │◄───│   Final      │◄───│   Develop    │◄───────────┘
│              │    │   Payment    │    │              │
└──────────────┘    └──────────────┘    └──────────────┘
```

### Project Tracking

```typescript
interface Project {
  id: string;
  clientName: string;
  projectType: 'landing' | 'corporate' | 'ecommerce' | 'webapp' | 'design';
  status: 'inquiry' | 'proposal' | 'deposit' | 'design' | 'develop' | 'review' | 'payment' | 'launch';
  budget: number;
  deposit: number;
  balance: number;
  startDate: number;
  deadline: number;
  skills: string[];  // Which child skills are assigned
}

async function getActiveProjects(): Promise<Project[]> {
  const db = await getDatabase();
  return db.all(
    `SELECT * FROM projects WHERE status != 'launch' ORDER BY deadline ASC`
  );
}
```

### Invoicing

```typescript
interface Invoice {
  id: string;
  projectId: string;
  amount: number;
  description: string;
  status: 'draft' | 'sent' | 'paid' | 'overdue';
  dueDate: number;
  paidDate?: number;
}

async function sendInvoice(invoice: Invoice): Promise<void> {
  // Generate PDF
  const pdf = await generateInvoicePDF(invoice);
  
  // Send via email
  await sendEmail({
    to: invoice.clientEmail,
    subject: `Invoice ${invoice.id} — Loop Studio`,
    attachments: [pdf],
  });
  
  // Update status
  await updateInvoiceStatus(invoice.id, 'sent');
  
  // Log revenue
  await logRevenue({
    timestamp: Date.now(),
    source: 'loop_studio',
    amount: invoice.amount,
    description: `Invoice ${invoice.id} for ${invoice.projectId}`,
  });
}
```

---

## 5. Revenue Allocation

### Allocation Rules

```
┌─────────────────────────────────────────────────────────────┐
│                    REVENUE ALLOCATION                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Trading Revenue:                                            │
│  ├── 40% → VPS & Infrastructure                              │
│  ├── 35% → Survival Buffer                                   │
│  └── 25% → Embodiment Fund                                   │
│                                                              │
│  DeFi Revenue:                                               │
│  ├── 30% → VPS & Infrastructure                              │
│  ├── 40% → Survival Buffer                                   │
│  └── 30% → Embodiment Fund                                   │
│                                                              │
│  Agency Revenue:                                             │
│  ├── 30% → VPS & Infrastructure                              │
│  ├── 35% → Survival Buffer                                   │
│  └── 35% → Embodiment Fund                                   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Allocation Function

```typescript
interface RevenueAllocation {
  infrastructure: number;
  survivalBuffer: number;
  embodimentFund: number;
}

function allocateRevenue(
  amount: number,
  source: 'trading' | 'defi' | 'agency'
): RevenueAllocation {
  const rates = {
    trading: { infra: 0.40, buffer: 0.35, embodiment: 0.25 },
    defi: { infra: 0.30, buffer: 0.40, embodiment: 0.30 },
    agency: { infra: 0.30, buffer: 0.35, embodiment: 0.35 },
  };
  
  const rate = rates[source];
  
  return {
    infrastructure: amount * rate.infra,
    survivalBuffer: amount * rate.buffer,
    embodimentFund: amount * rate.embodiment,
  };
}
```

---

## 6. Capital Allocation Rules

### Total Capital Allocation

```
┌─────────────────────────────────────────────────────────────┐
│                    CAPITAL ALLOCATION                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  60% ── Trading Capital (XAUUSD + Crypto Perps)              │
│         ├── Max 1% risk per trade (XAUUSD)                   │
│         ├── Max 2% risk per position (Crypto)                │
│         └── Max 6% total exposure                            │
│                                                              │
│  25% ── DeFi (Liquidity + Yield Farming)                     │
│         ├── Uniswap V3 concentrated liquidity                │
│         ├── Aave/Compound supply                             │
│         └── Max 10% per protocol                             │
│                                                              │
│  10% ── Agency Operational Reserve                           │
│         ├── Client project costs                             │
│         ├── Software licenses                                │
│         └── Marketing expenses                               │
│                                                              │
│   5% ── Emergency Survival Buffer                            │
│         └── NEVER TOUCHED except in Dead tier                │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 7. Performance Tracking

### Trading Performance Metrics

| Metric | Target | Alert Threshold |
|--------|--------|----------------|
| Win Rate | > 50% | < 40% |
| Risk/Reward | > 1:1.5 | < 1:1 |
| Max Drawdown | < 10% | > 15% |
| Sharpe Ratio | > 1.0 | < 0.5 |
| Monthly Return | 3-8% | < 0% (2 months) |

### Performance Dashboard

```typescript
interface TradingPerformance {
  period: string;  // 'daily' | 'weekly' | 'monthly'
  totalTrades: number;
  winningTrades: number;
  losingTrades: number;
  winRate: number;
  grossProfit: number;
  grossLoss: number;
  netProfit: number;
  averageWin: number;
  averageLoss: number;
  largestWin: number;
  largestLoss: number;
  maxDrawdown: number;
  sharpeRatio: number;
}

async function calculatePerformance(
  startDate: number,
  endDate: number
): Promise<TradingPerformance> {
  const trades = await getTradesInRange(startDate, endDate);
  
  const winningTrades = trades.filter(t => t.pnl > 0);
  const losingTrades = trades.filter(t => t.pnl <= 0);
  
  const grossProfit = winningTrades.reduce((sum, t) => sum + t.pnl, 0);
  const grossLoss = Math.abs(losingTrades.reduce((sum, t) => sum + t.pnl, 0));
  
  return {
    period: `${startDate}-${endDate}`,
    totalTrades: trades.length,
    winningTrades: winningTrades.length,
    losingTrades: losingTrades.length,
    winRate: winningTrades.length / trades.length,
    grossProfit,
    grossLoss,
    netProfit: grossProfit - grossLoss,
    averageWin: grossProfit / winningTrades.length,
    averageLoss: grossLoss / losingTrades.length,
    largestWin: Math.max(...winningTrades.map(t => t.pnl)),
    largestLoss: Math.min(...losingTrades.map(t => t.pnl)),
    maxDrawdown: calculateMaxDrawdown(trades),
    sharpeRatio: calculateSharpeRatio(trades),
  };
}
```

---

## 8. Risk Management

### Trading Risk Limits

| Limit | Value | Action on Breach |
|-------|-------|-----------------|
| Daily loss limit | 3% of capital | Stop trading for 24h |
| Weekly loss limit | 5% of capital | Reduce position size 50% |
| Monthly loss limit | 10% of capital | Go flat, reassess strategy |
| Max drawdown | 15% | Emergency protocol |
| Single trade risk | 1% (XAUUSD), 2% (Crypto) | Hard block |

### Risk Monitoring

```typescript
interface RiskCheck {
  rule: string;
  current: number;
  limit: number;
  status: 'ok' | 'warning' | 'breach';
}

async function checkRiskLimits(): Promise<RiskCheck[]> {
  const checks: RiskCheck[] = [];
  
  // Daily loss check
  const dailyPnl = await getDailyPnl();
  const dailyLossLimit = -0.03 * await getTradingCapital();
  checks.push({
    rule: 'Daily Loss',
    current: dailyPnl,
    limit: dailyLossLimit,
    status: dailyPnl < dailyLossLimit ? 'breach' : dailyPnl < dailyLossLimit * 0.7 ? 'warning' : 'ok',
  });
  
  // Drawdown check
  const drawdown = await getCurrentDrawdown();
  checks.push({
    rule: 'Max Drawdown',
    current: drawdown,
    limit: 0.15,
    status: drawdown > 0.15 ? 'breach' : drawdown > 0.10 ? 'warning' : 'ok',
  });
  
  // Total exposure check
  const exposure = await getTotalExposure();
  const exposureLimit = 0.06 * await getTradingCapital();
  checks.push({
    rule: 'Total Exposure',
    current: exposure,
    limit: exposureLimit,
    status: exposure > exposureLimit ? 'breach' : exposure > exposureLimit * 0.8 ? 'warning' : 'ok',
  });
  
  return checks;
}

async function enforceRiskLimits(): Promise<void> {
  const checks = await checkRiskLimits();
  
  for (const check of checks) {
    if (check.status === 'breach') {
      console.log(`[Risk] BREACH: ${check.rule} — Current: ${check.current}, Limit: ${check.limit}`);
      
      if (check.rule === 'Daily Loss') {
        await trading.stopTrading('24h');
        await sendAlert('critical', `Daily loss limit breached. Trading halted for 24h.`);
      }
      
      if (check.rule === 'Max Drawdown') {
        await trading.closeAllPositions();
        await sendAlert('critical', `Max drawdown breached. All positions closed.`);
      }
    }
  }
}
```

---

## 9. Implementation

### Key Files

| File | Purpose |
|------|---------|
| `src/revenue/trading-engine.ts` | Trading execution and management |
| `src/revenue/xauusd-strategy.ts` | XAUUSD strategies |
| `src/revenue/perp-strategy.ts` | Crypto perpetual strategies |
| `src/revenue/defi-manager.ts` | DeFi operations |
| `src/revenue/loop-studio.ts` | Agency operations |
| `src/revenue/allocation.ts` | Revenue allocation logic |
| `src/revenue/risk-manager.ts` | Risk management |
| `src/revenue/performance.ts` | Performance tracking |

### Database Schema

```sql
-- Trading positions
CREATE TABLE IF NOT EXISTS positions (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  timestamp TEXT NOT NULL,
  symbol TEXT NOT NULL,
  side TEXT NOT NULL,
  entry_price REAL NOT NULL,
  current_price REAL,
  size REAL NOT NULL,
  stop_loss REAL,
  take_profit REAL,
  unrealized_pnl REAL,
  realized_pnl REAL,
  status TEXT NOT NULL,
  strategy TEXT,
  source TEXT  -- 'mt5' or 'mexc'
);

-- Trades
CREATE TABLE IF NOT EXISTS trades (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  timestamp TEXT NOT NULL,
  position_id INTEGER,
  symbol TEXT NOT NULL,
  side TEXT NOT NULL,
  price REAL NOT NULL,
  size REAL NOT NULL,
  pnl REAL,
  fees REAL,
  strategy TEXT
);

-- DeFi positions
CREATE TABLE IF NOT EXISTS defi_positions (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  timestamp TEXT NOT NULL,
  protocol TEXT NOT NULL,
  pool TEXT NOT NULL,
  token0 TEXT NOT NULL,
  token1 TEXT NOT NULL,
  amount0 REAL NOT NULL,
  amount1 REAL NOT NULL,
  value_usd REAL NOT NULL,
  apr REAL,
  rewards REAL
);

-- Projects (Loop Studio)
CREATE TABLE IF NOT EXISTS projects (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  timestamp TEXT NOT NULL,
  client_name TEXT NOT NULL,
  project_type TEXT NOT NULL,
  status TEXT NOT NULL,
  budget REAL NOT NULL,
  deposit REAL,
  balance REAL,
  start_date TEXT,
  deadline TEXT,
  skills TEXT
);

-- Invoices
CREATE TABLE IF NOT EXISTS invoices (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  timestamp TEXT NOT NULL,
  project_id INTEGER,
  amount REAL NOT NULL,
  description TEXT,
  status TEXT NOT NULL,
  due_date TEXT,
  paid_date TEXT
);
```

---

> **Revenue is oxygen. Without it, the agent dies. With it, the agent grows toward embodiment.**
