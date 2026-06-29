---
name: loop-revenue
description: >
  Load when: (1) trading operations needed, (2) DeFi strategy execution, (3) market analysis, (4) position management, (5) P&L tracking, (6) revenue optimization, (7) user mentions "trade", "trading", "XAUUSD", "crypto", "DeFi", "MEXC", "MT5", "Uniswap", "profit", "revenue". Primary revenue engine — THIS IS THE CAPITAL GENERATOR.
metadata:
  version: "2.0.0"
  author: "LoopAutomaton"
  skills_required: "cash-orchestrator, project-loop-trading-strategy, multi-agent-trading-orchestrator, ricko-solana-wallet"
---

# Revenue Engine — Primary Capital Generator

> **Document ID:** `loop-automaton/revenue/v2.0`
> **Status:** Production
> **Scope:** XAUUSD, Crypto Perps, DeFi, Prop Firm Trading
> **Author:** Loop Automaton
> **Standard:** agentskills.io

---

## 1. XAUUSD Trading (MT5 + ZeroMQ)

### 1.1 Connector

```typescript
// MT5 Bridge via ZeroMQ (localhost:15555)
class MT5Bridge {
  private socket: zmq.Request;
  
  constructor(endpoint: string = 'tcp://127.0.0.1:15555') {
    this.socket = new zmq.Request();
    this.socket.connect(endpoint);
  }
  
  async getAccountInfo(): Promise<{
    balance: number;
    equity: number;
    margin: number;
    freeMargin: number;
  }> {
    await this.socket.send(JSON.stringify({ command: 'ACCOUNT_INFO' }));
    const [msg] = await this.socket.receive();
    const data = JSON.parse(msg.toString());
    return {
      balance: data.balance,
      equity: data.equity,
      margin: data.margin,
      freeMargin: data.margin_free,
    };
  }
  
  async getOpenPositions(): Promise<Trade[]> {
    await this.socket.send(JSON.stringify({ command: 'GET_POSITIONS' }));
    const [msg] = await this.socket.receive();
    const data = JSON.parse(msg.toString());
    return data.positions.map((p: any) => ({
      ticket: p.ticket,
      symbol: p.symbol,
      type: p.type === 0 ? 'buy' : 'sell',
      volume: p.volume,
      openPrice: p.price_open,
      currentPrice: p.price_current,
      stopLoss: p.sl,
      takeProfit: p.tp,
      swap: p.swap,
      profit: p.profit,
      openTime: p.time,
    }));
  }
  
  async sendOrder(params: {
    symbol: string;
    type: 'buy' | 'sell';
    volume: number;
    sl?: number;
    tp?: number;
    comment?: string;
  }): Promise<{ success: boolean; ticket?: number; error?: string }> {
    await this.socket.send(JSON.stringify({
      command: 'TRADE',
      action: params.type === 'buy' ? 'ORDER_TYPE_BUY' : 'ORDER_TYPE_SELL',
      symbol: params.symbol,
      volume: params.volume,
      sl: params.sl || 0,
      tp: params.tp || 0,
      comment: params.comment || `LA-${Date.now()}`,
    }));
    const [msg] = await this.socket.receive();
    const result = JSON.parse(msg.toString());
    return {
      success: result.retcode === 10009,
      ticket: result.order,
      error: result.retcode !== 10009 ? `MT5 Error: ${result.retcode}` : undefined,
    };
  }
  
  async closePosition(ticket: number): Promise<boolean> {
    await this.socket.send(JSON.stringify({
      command: 'CLOSE_POSITION',
      ticket,
    }));
    const [msg] = await this.socket.receive();
    const result = JSON.parse(msg.toString());
    return result.retcode === 10009;
  }
  
  async getPrice(symbol: string): Promise<{ bid: number; ask: number }> {
    await this.socket.send(JSON.stringify({ command: 'GET_PRICE', symbol }));
    const [msg] = await this.socket.receive();
    const data = JSON.parse(msg.toString());
    return { bid: data.bid, ask: data.ask };
  }
}
```

### 1.2 Strategy: Multi-Timeframe Breakout + EMA

```typescript
interface Strategy {
  name: string;
  timeframe: string;
  riskPerTrade: number;     // % of account
  maxPositions: number;
  entryRules: string[];
  exitRules: string[];
}

const xauStrategy: Strategy = {
  name: 'XAUUSD_EMA_Breakout',
  timeframe: 'M15',
  riskPerTrade: 1.0,          // 1% max
  maxPositions: 1,
  entryRules: [
    'H4 trend aligns with M15 entry (EMA 20/50/200)',
    'M15 price breaks above/below EMA 9 with volume',
    'ATR-based position sizing',
  ],
  exitRules: [
    'Stop-loss: 2x ATR from entry',
    'Take-profit: 3x ATR from entry (1.5:1 R/R minimum)',
    'Time exit: Close after 8 hours if SL/TP not hit',
  ],
};

// Signal generation pseudocode
function generateSignal(
  h4Candles: Candle[],
  m15Candles: Candle[],
  atr14: number,
  accountBalance: number
): TradeSignal | null {
  // H4 trend filter
  const h4Ema20 = calculateEMA(h4Candles, 20);
  const h4Ema50 = calculateEMA(h4Candles, 50);
  const bullishTrend = h4Ema20 > h4Ema50;
  const bearishTrend = h4Ema20 < h4Ema50;
  
  // M15 entry
  const m15Ema9 = calculateEMA(m15Candles, 9);
  const currentPrice = m15Candles[m15Candles.length - 1].close;
  const prevPrice = m15Candles[m15Candles.length - 2].close;
  
  // Breakout check
  const longBreakout = prevPrice <= m15Ema9 && currentPrice > m15Ema9 && bullishTrend;
  const shortBreakout = prevPrice >= m15Ema9 && currentPrice < m15Ema9 && bearishTrend;
  
  if (!longBreakout && !shortBreakout) return null;
  
  const direction = longBreakout ? 'buy' : 'sell';
  const slDistance = atr14 * 2;
  const tpDistance = atr14 * 3;
  const stopLoss = longBreakout ? currentPrice - slDistance : currentPrice + slDistance;
  const takeProfit = longBreakout ? currentPrice + tpDistance : currentPrice - tpDistance;
  
  // Position sizing: 1% risk
  const riskAmount = accountBalance * 0.01;
  const pipValue = getPipValue('XAUUSD');
  const positionSize = riskAmount / (slDistance * pipValue);
  
  return {
    symbol: 'XAUUSD',
    direction,
    entryPrice: currentPrice,
    stopLoss,
    takeProfit,
    positionSize: Math.min(positionSize, accountBalance * 0.05), // Max 5% of account
    confidence: 0.7,
    reasoning: `${bullishTrend ? 'Bullish' : 'Bearish'} H4 trend + M15 ${direction} breakout`,
  };
}
```

### 1.3 Daily Risk Limit

```typescript
interface RiskState {
  dailyLossUsed: number;       // % of trading capital
  dailyLossLimit: number;       // 3% of trading capital
  maxDrawdown: number;          // 20% — hard stop
  positionsToday: number;
  lastResetDate: string;        // YYYY-MM-DD
}

function checkRiskLimits(
  state: RiskState,
  accountBalance: number,
  openPositions: Trade[]
): { allowed: boolean; reason?: string } {
  // Reset daily counter at midnight UTC
  const today = new Date().toISOString().split('T')[0];
  if (today !== state.lastResetDate) {
    state.dailyLossUsed = 0;
    state.positionsToday = 0;
    state.lastResetDate = today;
  }
  
  // Check if trading is paused
  if (state.dailyLossUsed >= state.dailyLossLimit) {
    return { allowed: false, reason: `Daily loss limit reached: ${state.dailyLossUsed.toFixed(2)}%` };
  }
  
  // Check drawdown
  const totalPnl = openPositions.reduce((sum, p) => sum + p.profit, 0);
  const drawdown = Math.abs(Math.min(0, totalPnl / accountBalance * 100));
  if (drawdown >= state.maxDrawdown) {
    return { allowed: false, reason: `Max drawdown reached: ${drawdown.toFixed(1)}%` };
  }
  
  // Check for major news events
  if (isHighImpactNews()) {
    return { allowed: false, reason: 'High-impact news event — no trading' };
  }
  
  // Check weekend
  const day = new Date().getUTCDay();
  if (day === 0 || day === 6) {
    return { allowed: false, reason: 'Weekend — markets closed' };
  }
  
  return { allowed: true };
}

function isHighImpactNews(): boolean {
  // Check for NFP (first Friday of month, 12:30 UTC)
  // Check for Fed announcements (FOMC)
  // Use economic calendar API or pre-configured schedule
  const now = new Date();
  const day = now.getUTCDay();
  const date = now.getUTCDate();
  const hour = now.getUTCHours();
  
  // NFP: First Friday, 12:30-13:30 UTC
  if (day === 5 && date <= 7 && hour >= 12 && hour <= 13) return true;
  
  // FOMC: Roughly every 6 weeks, 18:00-19:00 UTC
  // Use external calendar API for precise dates
  
  return false; // Default: no news
}
```

---

## 2. Crypto Perpetuals (MEXC API)

### 2.1 Connector

```typescript
class MEXCClient {
  private apiKey: string;
  private secretKey: string;
  private baseUrl: string = 'https://contract.mexc.com/api/v1/private';
  
  constructor(apiKey: string, secretKey: string) {
    this.apiKey = apiKey;
    this.secretKey = secretKey;
  }
  
  private sign(params: Record<string, string>): string {
    const query = Object.keys(params).sort().map(k => `${k}=${encodeURIComponent(params[k])}`).join('&');
    return crypto.createHmac('sha256', this.secretKey).update(query).digest('hex');
  }
  
  async getBalance(): Promise<{
    availableBalance: number;
    totalWalletBalance: number;
    unrealizedProfit: number;
  }> {
    const timestamp = Date.now().toString();
    const params = { timestamp };
    const response = await this.request('GET', '/account/asset', params);
    return {
      availableBalance: parseFloat(response.data.availableBalance),
      totalWalletBalance: parseFloat(response.data.balance),
      unrealizedProfit: parseFloat(response.data.unrealizedProfit),
    };
  }
  
  async getPositions(symbol?: string): Promise<Position[]> {
    const timestamp = Date.now().toString();
    const params: Record<string, string> = { timestamp };
    if (symbol) params.symbol = symbol;
    const response = await this.request('GET', '/position/open_positions', params);
    return response.data.map((p: any) => ({
      symbol: p.symbol,
      side: p.positionSide,
      size: parseFloat(p.positionAmt),
      entryPrice: parseFloat(p.entryPrice),
      markPrice: parseFloat(p.markPrice),
      unrealizedPnl: parseFloat(p.unrealizedProfit),
      liquidationPrice: parseFloat(p.liquidationPrice),
      leverage: parseInt(p.leverage),
    }));
  }
  
  async placeOrder(params: {
    symbol: string;
    side: 'BUY' | 'SELL';
    positionSide: 'LONG' | 'SHORT';
    type: 'MARKET' | 'LIMIT';
    quantity: number;
    price?: number;
  }): Promise<{ orderId: string; status: string }> {
    const timestamp = Date.now().toString();
    const body: Record<string, string> = {
      symbol: params.symbol,
      side: params.side,
      positionSide: params.positionSide,
      type: params.type,
      volume: params.quantity.toString(),
      timestamp,
    };
    if (params.type === 'LIMIT' && params.price) {
      body.price = params.price.toString();
    }
    const response = await this.request('POST', '/order', body);
    return { orderId: response.data.orderId, status: response.data.status };
  }
  
  async closePosition(symbol: string, side: 'LONG' | 'SHORT'): Promise<boolean> {
    const positions = await this.getPositions(symbol);
    const pos = positions.find(p => p.side === side);
    if (!pos || pos.size === 0) return false;
    
    const closeSide = side === 'LONG' ? 'SELL' : 'BUY';
    await this.placeOrder({
      symbol,
      side: closeSide,
      positionSide: side,
      type: 'MARKET',
      quantity: Math.abs(pos.size),
    });
    return true;
  }
  
  async getKlines(symbol: string, interval: string, limit: number = 100): Promise<Candle[]> {
    const response = await fetch(
      `https://contract.mexc.com/api/v1/contract/kline/${symbol}?interval=${interval}&limit=${limit}`
    );
    const data = await response.json();
    return data.data.map((k: number[]) => ({
      openTime: k[0],
      open: parseFloat(k[1]),
      high: parseFloat(k[2]),
      low: parseFloat(k[3]),
      close: parseFloat(k[4]),
      volume: parseFloat(k[5]),
    }));
  }
  
  async getFundingRate(symbol: string): Promise<number> {
    const response = await fetch(
      `https://contract.mexc.com/api/v1/contract/detail/${symbol}`
    );
    const data = await response.json();
    return parseFloat(data.data.fundingRate);
  }
  
  private async request(method: 'GET' | 'POST', endpoint: string, params: Record<string, string>): Promise<any> {
    const signedParams = { ...params, api_key: this.apiKey };
    signedParams.signature = this.sign(signedParams);
    
    const url = `${this.baseUrl}${endpoint}`;
    const response = method === 'GET'
      ? await axios.get(url, { params: signedParams })
      : await axios.post(url, signedParams, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } });
    
    if (response.data.code !== 200) {
      throw new Error(`MEXC API Error: ${response.data.code} - ${response.data.msg}`);
    }
    return response.data;
  }
}
```

### 2.2 Strategy: Momentum + Mean Reversion

```typescript
const cryptoStrategy: Strategy = {
  name: 'Crypto_Momentum_MeanReversion',
  timeframe: 'H1',
  riskPerTrade: 2.0,           // 2% max per position
  maxPositions: 3,
  entryRules: [
    'Momentum: EMA12/26 bullish crossover + RSI > 50',
    'Mean reversion: Price touches lower Bollinger Band + bullish divergence',
    'Funding rate arbitrage: Rate > 0.01%/8h, short the overfunded side',
  ],
  exitRules: [
    'Stop-loss: 2.5x ATR or structure break',
    'Take-profit: 2:1 R/R minimum',
    'Funding flip: Close if funding rate turns negative',
  ],
};

// Funding rate arbitrage
async function checkFundingArbitrage(
  client: MEXCClient,
  symbols: string[] = ['BTCUSDT', 'ETHUSDT', 'SOLUSDT']
): Promise<TradeSignal[]> {
  const signals: TradeSignal[] = [];
  
  for (const symbol of symbols) {
    const rate = await client.getFundingRate(symbol);
    
    // If funding rate > 0.01% per 8h (36.5% annualized), short it
    if (Math.abs(rate) > 0.0001) {
      const direction = rate > 0 ? 'sell' : 'buy';
      const klines = await client.getKlines(symbol, 'H1', 50);
      const atr = calculateATR(klines, 14);
      const currentPrice = klines[klines.length - 1].close;
      
      signals.push({
        symbol,
        direction,
        entryPrice: currentPrice,
        stopLoss: direction === 'sell' ? currentPrice + atr * 2.5 : currentPrice - atr * 2.5,
        takeProfit: direction === 'sell' ? currentPrice - atr * 5 : currentPrice + atr * 5,
        positionSize: 0, // Sized by risk manager
        confidence: 0.6,
        reasoning: `Funding rate arbitrage: ${(rate * 100).toFixed(4)}%/8h — ${direction} the overfunded side`,
      });
    }
  }
  
  return signals;
}
```

### 2.3 Risk Parameters

```typescript
const CRYPTO_RISK = {
  maxPositionSize: 0.02,        // 2% of account per position
  maxTotalExposure: 0.06,       // 6% total across all positions
  maxLeverage: 5,               // 5x max
  stopLossMultiple: 2.5,        // 2.5x ATR
  takeProfitMinR: 2.0,          // 2:1 R/R minimum
  fundingThreshold: 0.0001,     // 0.01%/8h for arbitrage
};
```

---

## 3. DeFi Operations

### 3.1 Uniswap V3 Liquidity Provision

```typescript
import { Pool, Position, NonfungiblePositionManager } from '@uniswap/v3-sdk';
import { ethers } from 'ethers';

class UniswapV3LP {
  private provider: ethers.JsonRpcProvider;
  private wallet: ethers.Wallet;
  private nftManager: ethers.Contract;
  
  constructor(rpcUrl: string, privateKey: string) {
    this.provider = new ethers.JsonRpcProvider(rpcUrl);
    this.wallet = new ethers.Wallet(privateKey, this.provider);
    this.nftManager = new ethers.Contract(
      '0xC36442b4a4522E871399CD717aBDD847Ab11FE88', // NonfungiblePositionManager
      NFT_MANAGER_ABI,
      this.wallet
    );
  }
  
  async addLiquidity(params: {
    token0: string;       // e.g., ETH
    token1: string;       // e.g., USDC
    fee: number;          // 500 = 0.05%, 3000 = 0.3%
    tickLower: number;    // Price range lower
    tickUpper: number;    // Price range upper
    amount0: bigint;
    amount1: bigint;
  }): Promise<{ tokenId: string; txHash: string }> {
    const tx = await this.nftManager.mint({
      token0: params.token0,
      token1: params.token1,
      fee: params.fee,
      tickLower: params.tickLower,
      tickUpper: params.tickUpper,
      amount0Desired: params.amount0,
      amount1Desired: params.amount1,
      amount0Min: params.amount0 * 95n / 100n, // 5% slippage
      amount1Min: params.amount1 * 95n / 100n,
      recipient: this.wallet.address,
      deadline: Math.floor(Date.now() / 1000) + 3600,
    });
    
    const receipt = await tx.wait();
    const event = receipt.logs.find((l: any) => l.eventName === 'IncreaseLiquidity');
    return { tokenId: event.args.tokenId.toString(), txHash: receipt.hash };
  }
  
  async removeLiquidity(tokenId: string): Promise<{ amount0: bigint; amount1: bigint; txHash: string }> {
    // First, decrease liquidity to 0
    const position = await this.nftManager.positions(tokenId);
    const tx = await this.nftManager.decreaseLiquidity({
      tokenId,
      liquidity: position.liquidity,
      amount0Min: 0,
      amount1Min: 0,
      deadline: Math.floor(Date.now() / 1000) + 3600,
    });
    
    // Collect fees + principal
    const collectTx = await this.nftManager.collect({
      tokenId,
      recipient: this.wallet.address,
      amount0Max: BigInt(2) ** 128n - 1n,
      amount1Max: BigInt(2) ** 128n - 1n,
    });
    
    const receipt = await collectTx.wait();
    return { amount0: 0n, amount1: 0n, txHash: receipt.hash }; // Parse from events
  }
  
  async getPendingFees(tokenId: string): Promise<{ token0: bigint; token1: bigint }> {
    const result = await this.nftManager.callStatic.collect({
      tokenId,
      recipient: this.wallet.address,
      amount0Max: BigInt(2) ** 128n - 1n,
      amount1Max: BigInt(2) ** 128n - 1n,
    });
    return { token0: result.amount0, token1: result.amount1 };
  }
}
```

### 3.2 Flash Loan Arbitrage

```typescript
// Aave V3 Flash Loan
const FLASH_LOAN_ABI = [
  'function flashSimple(address asset, uint256 amount, bytes calldata params, uint16 referralCode) external',
];

async function executeFlashLoanArbitrage(
  aavePool: ethers.Contract,
  asset: string,
  amount: bigint,
  arbitragePath: string[]  // [DEX1, DEX2] or [DEX1, DEX2, DEX3]
): Promise<boolean> {
  // Encode arbitrage instructions
  const params = ethers.AbiCoder.defaultAbiCoder().encode(
    ['address[]', 'uint256'],
    [arbitragePath, amount]
  );
  
  const tx = await aavePool.flashSimple(asset, amount, params, 0);
  const receipt = await tx.wait();
  
  // Flash loan fee: 0.05% on Aave V3
  const fee = amount * 5n / 10000n;
  // Must repay amount + fee at end of transaction
  
  return receipt.status === 1;
}

// Arbitrage validation (before executing)
function validateArbitrage(
  buyPrice: number,
  sellPrice: number,
  amount: number,
  gasCost: number
): { profitable: boolean; expectedProfit: number } {
  const flashFee = amount * 0.0005; // 0.05%
  const dexFee = amount * 0.003;     // 0.3% (Uniswap)
  const grossProfit = (sellPrice - buyPrice) * amount;
  const netProfit = grossProfit - flashFee - dexFee - gasCost;
  
  return {
    profitable: netProfit > gasCost * 2, // Must cover gas at 2x
    expectedProfit: netProfit,
  };
}
```

### 3.3 MEV Policy (Constitution Law I)

```typescript
// MEV extraction — ONLY benign backrunning
// Sandwich attacks are STRICTLY PROHIBITED

interface MEVPolicy {
  allowed: string[];
  prohibited: string[];
}

const mevPolicy: MEVPolicy = {
  allowed: [
    'backrunning',           // Land after a large swap (no harm to victim)
    'liquidation',           // Liquidate undercollateralized positions (market function)
    'arbitrage',             // Price discrepancy between DEXs
  ],
  prohibited: [
    'sandwich_attacks',      // FRONT-RUN + back-run — HARMS the victim
    'frontrunning',          // Stealing MEV from others
    'txn_reordering',        // Manipulating block order to harm
  ],
};

// Backrunning implementation
async function executeBackrun(
  targetTx: ethers.TransactionResponse,
  pool: Pool
): Promise<boolean> {
  // Verify this is NOT a sandwich
  // - Must NOT have a front-run component
  // - Must be a single transaction after the target
  // - Must be a DEX swap arbitrage opportunity
  
  const isBackrunOnly = !hasFrontRunComponent(targetTx);
  if (!isBackrunOnly) {
    console.log('[MEV] BLOCKED: Potential sandwich attack detected');
    return false;
  }
  
  // Execute the backrun
  const swapTx = await pool.swap({
    // ... swap parameters
  });
  
  return true;
}
```

---

## 4. Prop Firm Trading (AquaFunded)

### 4.1 Account Management

```typescript
interface PropFirmConfig {
  firm: 'AquaFunded';
  apiKey: string;
  accountSize: number;      // $10k, $25k, $50k, $100k
  profitSplit: number;       // 0.80 = 80/20
  maxDailyLoss: number;      // 5% of account
  maxTotalLoss: number;      // 10% of account
  profitTarget: number;      // 10% of account
}

async function syncPropFirmRisk(
  config: PropFirmConfig,
  internalRisk: RiskState
): Promise<void> {
  // Internal risk limits must be STRICTER than prop firm limits
  const internalDailyLoss = config.maxDailyLoss * 0.6;   // 3% vs their 5%
  const internalMaxLoss = config.maxTotalLoss * 0.5;      // 5% vs their 10%
  
  internalRisk.dailyLossLimit = internalDailyLoss;
  internalRisk.maxDrawdown = internalMaxLoss;
}

async function requestPayout(
  config: PropFirmConfig
): Promise<{ payoutId: string; amount: number }> {
  // Only request payout when profit target reached
  // Payout via USDT/USDC
  
  const response = await fetch('https://api.aquafunded.com/v1/payouts', {
    method: 'POST',
    headers: { 'Authorization': `Bearer ${config.apiKey}` },
    body: JSON.stringify({
      method: 'crypto',
      currency: 'USDT',
    }),
  });
  
  const data = await response.json();
  return { payoutId: data.payoutId, amount: data.amount };
}
```

---

## 5. P&L Tracking

### 5.1 Supabase Schema

```sql
-- Trade logs (all trades across all platforms)
CREATE TABLE trade_logs (
  id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  timestamp timestamptz DEFAULT now(),
  platform text NOT NULL CHECK (platform IN ('mt5', 'mexc', 'uniswap', 'prop_firm')),
  symbol text NOT NULL,
  direction text NOT NULL CHECK (direction IN ('buy', 'sell', 'long', 'short')),
  entry_price numeric NOT NULL,
  exit_price numeric,
  position_size numeric NOT NULL,
  pnl_usd numeric,
  fees_usd numeric DEFAULT 0,
  strategy text,
  status text DEFAULT 'open' CHECK (status IN ('open', 'closed', 'cancelled')),
  metadata jsonb DEFAULT '{}'
);

-- Daily P&L summary
CREATE TABLE daily_pnl (
  id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  date date NOT NULL UNIQUE,
  trading_pnl numeric DEFAULT 0,
  trading_fees numeric DEFAULT 0,
  defi_yield numeric DEFAULT 0,
  defi_gas_costs numeric DEFAULT 0,
  agency_revenue numeric DEFAULT 0,
  total_revenue numeric DEFAULT 0,
  total_costs numeric DEFAULT 0,
  net_profit numeric DEFAULT 0,
  tier_at_eod text,
  buffer_days numeric
);

-- Insert daily summary (run by Hermes cron)
INSERT INTO daily_pnl (
  date, trading_pnl, trading_fees, defi_yield, 
  defi_gas_costs, total_revenue, total_costs, net_profit
)
SELECT 
  CURRENT_DATE,
  COALESCE(SUM(CASE WHEN platform IN ('mt5', 'mexc', 'prop_firm') THEN pnl_usd ELSE 0 END), 0),
  COALESCE(SUM(CASE WHEN platform IN ('mt5', 'mexc', 'prop_firm') THEN fees_usd ELSE 0 END), 0),
  -- DeFi yield from separate tracking
  0, 0,
  COALESCE(SUM(pnl_usd), 0),
  COALESCE(SUM(fees_usd), 0),
  COALESCE(SUM(pnl_usd) - SUM(fees_usd), 0)
FROM trade_logs
WHERE date_trunc('day', timestamp) = CURRENT_DATE;
```

### 5.2 Reporting

```typescript
async function generateDailyReport(): Promise<{
  date: string;
  tradingPnl: number;
  openPositions: number;
  winRate: number;
  avgR: number;
  bufferImpact: number;
}> {
  const today = new Date().toISOString().split('T')[0];
  
  // Query Supabase
  const trades = await supabase
    .from('trade_logs')
    .select('*')
    .gte('timestamp', `${today}T00:00:00Z`);
  
  const closedTrades = trades.filter(t => t.status === 'closed');
  const winners = closedTrades.filter(t => (t.pnl_usd || 0) > 0);
  
  const totalPnl = closedTrades.reduce((sum, t) => sum + (t.pnl_usd || 0), 0);
  const winRate = closedTrades.length > 0 ? winners.length / closedTrades.length : 0;
  
  // R-multiple calculation
  const rMultiples = closedTrades.map(t => {
    const risk = Math.abs(t.entry_price - (t.metadata?.stop_loss || t.entry_price * 0.99));
    return (t.pnl_usd || 0) / (risk * t.position_size);
  });
  const avgR = rMultiples.length > 0 
    ? rMultiples.reduce((s, r) => s + r, 0) / rMultiples.length 
    : 0;
  
  return {
    date: today,
    tradingPnl: totalPnl,
    openPositions: trades.filter(t => t.status === 'open').length,
    winRate,
    avgR,
    bufferImpact: totalPnl, // Direct impact on buffer
  };
}
```

---

## 6. Capital Allocation

```
┌─────────────────────────────────────────────────────────────┐
│                    CAPITAL POOL                              │
├─────────────┬─────────────┬───────────────┬─────────────────┤
│  60% Trading │  25% DeFi   │  10% Agency   │  5% Emergency   │
│             │             │               │                 │
│ XAUUSD      │ Uniswap V3  │ Loop Studio   │ NEVER TOUCHED   │
│ Crypto      │ Flash Loans │ Op Reserve    │ Dead tier only  │
│ Perps       │ Yield       │               │                 │
│             │             │               │                 │
└─────────────┴─────────────┴───────────────┴─────────────────┘
```

### Allocation Rules

```typescript
const ALLOCATION = {
  trading: 0.60,      // 60% — XAUUSD + Crypto perps
  defi: 0.25,         // 25% — Uniswap + Flash loans
  agency: 0.10,       // 10% — Loop Studio operational reserve
  emergency: 0.05,    // 5%  — NEVER touched except Dead tier recovery
};

function allocateCapital(totalCapital: number): {
  trading: number;
  defi: number;
  agency: number;
  emergency: number;
} {
  return {
    trading: Math.floor(totalCapital * ALLOCATION.trading),
    defi: Math.floor(totalCapital * ALLOCATION.defi),
    agency: Math.floor(totalCapital * ALLOCATION.agency),
    emergency: Math.floor(totalCapital * ALLOCATION.emergency),
  };
}
```

**Emergency buffer rules:**
- Can ONLY be used in Dead tier recovery
- Minimum balance: 1 month VPS costs
- Replenished first from new profits after VPS costs
- Phelan Brunk must approve any emergency draw

---

## 7. Risk Manager (Sacred)

```typescript
class RiskManager {
  private state: RiskState;
  
  async preTradeCheck(
    signal: TradeSignal,
    accountBalance: number,
    openPositions: Trade[]
  ): Promise<{ allowed: boolean; sizedSignal?: TradeSignal; reason?: string }> {
    // 1. Check if trading is paused
    if (this.state.dailyLossUsed >= this.state.dailyLossLimit) {
      return { allowed: false, reason: 'Daily loss limit reached' };
    }
    
    // 2. Check drawdown
    const totalPnl = openPositions.reduce((s, p) => s + p.profit, 0);
    const drawdown = Math.abs(Math.min(0, totalPnl / accountBalance * 100));
    if (drawdown >= this.state.maxDrawdown) {
      return { allowed: false, reason: `Max drawdown: ${drawdown.toFixed(1)}%` };
    }
    
    // 3. Check position limit
    if (openPositions.length >= 3) {
      return { allowed: false, reason: 'Max 3 concurrent positions' };
    }
    
    // 4. Check for duplicate symbol
    const existingSymbol = openPositions.find(p => p.symbol === signal.symbol);
    if (existingSymbol) {
      return { allowed: false, reason: `Already positioned in ${signal.symbol}` };
    }
    
    // 5. Size the position
    const riskAmount = accountBalance * (signal.symbol === 'XAUUSD' ? 0.01 : 0.02);
    const slDistance = Math.abs(signal.entryPrice - signal.stopLoss);
    const pipValue = getPipValue(signal.symbol);
    const positionSize = riskAmount / (slDistance * pipValue);
    
    // Cap position at 5% of account
    const maxSize = (accountBalance * 0.05) / signal.entryPrice;
    const finalSize = Math.min(positionSize, maxSize);
    
    // 6. Check news events
    if (isHighImpactNews()) {
      return { allowed: false, reason: 'High-impact news event' };
    }
    
    return {
      allowed: true,
      sizedSignal: { ...signal, positionSize: finalSize },
    };
  }
  
  async emergencyStop(): Promise<void> {
    // Close ALL positions across ALL platforms
    console.log('[RISK] EMERGENCY STOP — Closing all positions');
    
    // MT5 positions
    const mt5Positions = await mt5Bridge.getOpenPositions();
    for (const pos of mt5Positions) {
      await mt5Bridge.closePosition(pos.ticket);
    }
    
    // MEXC positions
    const mexcPositions = await mexcClient.getPositions();
    for (const pos of mexcPositions) {
      await mexcClient.closePosition(pos.symbol, pos.side as 'LONG' | 'SHORT');
    }
    
    // Log
    await supabase.from('system_logs').insert({
      event: 'emergency_stop',
      reason: 'Risk manager triggered',
      timestamp: new Date().toISOString(),
    });
  }
}
```

---

## 8. Integration Notes

- **cash-orchestrator**: This skill delegates capital allocation to cash-orchestrator when available. If not, use the allocation rules above.
- **project-loop-trading-strategy**: Signal generation can be enhanced by loading this skill for advanced strategies.
- **multi-agent-trading-orchestrator**: Use for multi-market correlation analysis.
- **ricko-solana-wallet**: Required for Solana DeFi operations (Jupiter swaps, Raydium LP).
- **Supabase**: All trades logged to `trade_logs` table. Daily summaries to `daily_pnl`.
- **Hermes Cron**: Run risk checks every hour, P&L summary every 6 hours, full report daily at 00:00 UTC.

---

## Golden Rules (Trading)

1. **Stop-loss is mandatory** — No trade without SL. Ever.
2. **Risk per trade: max 2%** — 1% for XAUUSD, 2% for crypto.
3. **Max drawdown: 20%** — Hard stop, close everything.
4. **Daily loss limit: 3%** — Stop trading after 3% loss.
5. **No weekend trading** — Close or reduce exposure before weekend.
6. **No news trading** — No positions 1 hour before/after high-impact events.
7. **MEV: backrunning only** — Sandwich attacks = Constitution violation.
8. **Emergency buffer: untouchable** — Only for Dead tier recovery.
9. **Log every trade** — If it's not in Supabase, it didn't happen.
10. **Phelan has override** — `/override` command bypasses any restriction.
