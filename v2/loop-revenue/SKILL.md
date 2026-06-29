---
name: loop-revenue
description: >
  Load when: (1) trading operations needed, (2) DeFi strategy execution, (3) market analysis, (4) position management, (5) P&L tracking, (6) revenue optimization, (7) user mentions "trade", "trading", "XAUUSD", "crypto", "DeFi", "MEXC", "MT5", "Uniswap", "profit", "revenue". Primary revenue engine — THIS IS THE CAPITAL GENERATOR.
author: LoopAutomaton v2
version: 2.0.0
skills_required:
  - cash-orchestrator
  - project-loop-trading-strategy
  - multi-agent-trading-orchestrator
  - ricko-solana-wallet
---

# Revenue Engine — Primary Capital Generator

## Capital Allocation
```
60% Trading Capital    (XAUUSD + Crypto Perps)
25% DeFi Operations    (Uniswap V3 + Yield + Flash Loans)
10% Agency Operational Reserve
 5% Emergency Buffer   (NEVER TOUCHED — requires 3-agent consensus)
```
Rebalance weekly. Operational reserve covers API fees, gas, VPS costs.

---

## 1. XAUUSD Trading (Primary)

### MT5 ZeroMQ Connector
```python
import zmq, pandas as pd
class MT5Bridge:
    def __init__(self, host="localhost", port=15555):
        self.ctx = zmq.Context()
        self.sock = self.ctx.socket(zmq.REQ)
        self.sock.connect(f"tcp://{host}:{port}")
        self.sock.setsockopt(zmq.RCVTIMEO, 5000)
    def send(self, action, symbol="XAUUSD", **kw):
        self.sock.send_json({"action": action, "symbol": symbol, **kw})
        return self.sock.recv_json()
    def get_rates(self, tf="M15", count=100):
        r = self.send("GET_RATES", timeframe=tf, count=count)
        df = pd.DataFrame(r["data"])
        df["time"] = pd.to_datetime(df["time"], unit="s")
        return df
    def market_order(self, side, volume, sl, tp, comment="loop-v2"):
        return self.send("TRADE", type="MARKET", side=side, volume=volume, sl=sl, tp=tp, comment=comment)
    def close_position(self, ticket):
        return self.send("CLOSE", ticket=ticket)
    def get_open_positions(self):
        return self.send("GET_POSITIONS")
    def get_account_info(self):
        return self.send("ACCOUNT")
```

### Multi-Timeframe Breakout + EMA Strategy
```python
def analyze_xauusd(bridge: MT5Bridge):
    h4 = bridge.get_rates("H4", 200); m15 = bridge.get_rates("M15", 100)
    for df, ps in [(h4, [20,50,200]), (m15, [9,21,50])]:
        for p in ps: df[f"EMA_{p}"] = df["close"].ewm(span=p).mean()
    hb = h4["EMA_20"].iloc[-1] > h4["EMA_50"].iloc[-1] > h4["EMA_200"].iloc[-1]
    hbe = h4["EMA_20"].iloc[-1] < h4["EMA_50"].iloc[-1] < h4["EMA_200"].iloc[-1]
    m15_bull = (m15["close"].iloc[-1] > m15["EMA_9"].iloc[-1] and
                m15["close"].iloc[-2] <= m15["EMA_9"].iloc[-2] and
                m15["tick_volume"].iloc[-1] > m15["tick_volume"].rolling(20).mean().iloc[-1])
    m15_bear = (m15["close"].iloc[-1] < m15["EMA_9"].iloc[-1] and
                m15["close"].iloc[-2] >= m15["EMA_9"].iloc[-2] and
                m15["tick_volume"].iloc[-1] > m15["tick_volume"].rolling(20).mean().iloc[-1])
    if hb and m15_bull: return {"dir":"LONG","entry":m15["close"].iloc[-1]}
    if hbe and m15_bear: return {"dir":"SHORT","entry":m15["close"].iloc[-1]}
    return {"dir":"FLAT"}
```

### XAUUSD Position Sizing
```python
def calc_xauusd_size(balance, entry, atr, signal_dir):
    risk = balance * 0.01                       # 1% max
    sl_dist = 2.0 * atr; tp_dist = 3.0 * atr    # 2x SL, 3x TP
    sl_pips = sl_dist / 0.01                     # XAUUSD 1 pip = $0.01
    lot = risk / (sl_pips * 10.0)               # 1 lot = $10/pip
    lot = round(max(0.01, min(lot, 5.0)), 2)
    sl = entry - sl_dist if signal_dir == "LONG" else entry + sl_dist
    tp = entry + tp_dist if signal_dir == "LONG" else entry - tp_dist
    return {"lots": lot, "sl": sl, "tp": tp, "risk": risk}
def get_atr(df, period=14):
    tr = pd.concat([df["high"]-df["low"], (df["high"]-df["close"].shift()).abs(), (df["low"]-df["close"].shift()).abs()], axis=1).max(axis=1)
    return tr.rolling(period).mean().iloc[-1]
```
**Params:** 1% risk/trade, SL=2xATR, TP=3xATR (1.5:1 R:R). Max 1 open XAUUSD position.

---

## 2. Crypto Perpetuals (Primary)

### MEXC API Connector
```python
import hmac, hashlib, time, requests
class MEXCClient:
    BASE = "https://contract.mexc.com"
    def __init__(self, key, secret): self.key = key; self.secret = secret
    def _sign(self, params):
        params["timestamp"] = str(int(time.time()*1000))
        qs = "&".join(f"{k}={v}" for k,v in sorted(params.items()))
        return hmac.new(self.secret.encode(), qs.encode(), hashlib.sha256).hexdigest()
    def req(self, method, path, params=None, signed=False):
        url = f"{self.BASE}{path}"
        if signed: params = params or {}; params["signature"] = self._sign(params); params["api_key"] = self.key
        return requests.request(method, url, params=params, timeout=10).json()
    def ticker(self, s): return self.req("GET","/api/v1/contract/ticker",{"symbol":s})
    def order(self, s, side, vol, price=0, otype="MARKET", sl=None, tp=None, lev=5):
        b = {"symbol":s,"side":side,"vol":vol,"type":otype,"price":price,"leverage":lev,"openType":2}
        if sl: b["stopLossPrice"]=sl; b["stopLossType"]=2
        if tp: b["takeProfitPrice"]=tp
        return self.req("POST","/api/v1/private/order/submit",b,signed=True)
    def positions(self): return self.req("GET","/api/v1/private/position/list",signed=True)
    def close(self, s): return self.req("POST","/api/v1/private/position/close",{"symbol":s},signed=True)
    def funding(self, s): return self.req("GET","/api/v1/contract/funding_rate",{"symbol":s})
```

### Momentum + Mean Reversion Strategy
```python
def analyze_crypto(client, symbol):
    df = fetch_klines_mexc(symbol, "1h", 100)   # OHLC from MEXC
    df["EMA12"] = df["close"].ewm(12).mean(); df["EMA26"] = df["close"].ewm(26).mean()
    df["RSI"] = rsi(df["close"],14); df["ATR"] = atr(df,14)
    bb = df["close"].rolling(20); df["BBu"] = bb.mean() + 2*bb.std(); df["BBd"] = bb.mean() - 2*bb.std()
    c = df.iloc[-1]
    if c["EMA12"] > c["EMA26"] and 55 < c["RSI"] < 80: return {"dir":"LONG","e":c["close"]}
    if c["EMA12"] < c["EMA26"] and 20 < c["RSI"] < 45: return {"dir":"SHORT","e":c["close"]}
    if c["close"] > c["BBu"] and c["RSI"] > 75: return {"dir":"SHORT","e":c["close"],"s":"MR"}
    if c["close"] < c["BBd"] and c["RSI"] < 25: return {"dir":"LONG","e":c["close"],"s":"MR"}
    return {"dir":"FLAT"}
```

### Funding Rate Arbitrage
```python
def funding_arb(client, syms=["BTC_USDT","ETH_USDT","SOL_USDT"]):
    for s in syms:
        rate = float(client.funding(s)["data"]["fundingRate"])
        if abs(rate) > 0.0001:                       # > 0.01% per 8h
            side = "SELL" if rate > 0 else "BUY"     # short positive funding
            price = float(client.ticker(s)["data"]["lastPrice"])
            vol = round(500 / price, 4)              # $500 notional max
            sl = price * (1.01 if side=="SELL" else 0.99)
            return client.order(s, side, vol, sl=sl)
    return None
```

### Crypto Position Sizing
```python
def calc_crypto_size(balance, entry, sl_price, lev=5):
    risk = balance * 0.02                            # 2% per position
    sl_pct = abs(entry - sl_price) / entry
    notional = risk / sl_pct
    margin = notional / lev
    total_exp = sum(p["notional"] for p in get_open_positions())
    if total_exp + notional > balance * 0.06:        # 6% total cap
        notional = max(0, balance * 0.06 - total_exp)
    return {"qty": round(notional/entry, 6), "margin": round(margin,2), "notional": round(notional,2)}
```
**Params:** 2% risk/position, 5x leverage, 6% total exposure, mandatory SL/TP.

---

## 3. DeFi Operations (Primary)

### Uniswap V3 Liquidity Provision
```python
from web3 import Web3
POS_MANAGER = "0xC36442b4a4522E871399CD717aBDD847Ab11FE88"
FEE_TIER = 500  # 0.05% for ETH-USDC, WBTC-ETH
class UniswapV3LP:
    def __init__(self, w3: Web3, pk: str):
        self.w3 = w3; self.acct = w3.eth.account.from_key(pk)
    def add_liq(self, t0, t1, amt0, amt1, tick_lo, tick_hi):
        pm = self.w3.eth.contract(address=POS_MANAGER, abi=PM_ABI)
        for t,a in [(t0,amt0),(t1,amt1)]:
            erc20 = self.w3.eth.contract(address=t, abi=ERC20_ABI)
            tx = erc20.functions.approve(POS_MANAGER,a).build_transaction({"from":self.acct.address,"nonce":self.w3.eth.get_transaction_count(self.acct.address)})
            self.w3.eth.send_raw_transaction(self.w3.eth.account.sign_transaction(tx,self.acct.key).rawTransaction)
        tx = pm.functions.mint({"token0":t0,"token1":t1,"fee":FEE_TIER,"tickLower":tick_lo,"tickUpper":tick_hi,"amount0Desired":amt0,"amount1Desired":amt1,"amount0Min":int(amt0*0.995),"amount1Min":int(amt1*0.995),"recipient":self.acct.address,"deadline":int(time.time())+300}).build_transaction({"from":self.acct.address,"nonce":self.w3.eth.get_transaction_count(self.acct.address),"gas":500000})
        return self.w3.eth.send_raw_transaction(self.w3.eth.account.sign_transaction(tx,self.acct.key).rawTransaction)
```

### Flash Loan Arbitrage (Aave V3)
```python
AAVE_POOL = "0x87870Bca3F3fD6335C3F4ce8392D69350B4fA4E2"
def flash_arb(w3, path, amounts, arb_contract):
    c = w3.eth.contract(address=arb_contract, abi=ARB_ABI)
    gp = w3.eth.gas_price; max_g = 500000; gas_cost = max_g * gp / 1e18
    profit = est_arb_profit(path, amounts[0])
    if profit <= gas_cost * 2: return {"status":"SKIP","why":"profit<2x_gas"}
    tx = c.functions.executeArbitrage(path,amounts).build_transaction({"from":ACCT.address,"gas":max_g,"gasPrice":int(gp*1.1),"nonce":w3.eth.get_transaction_count(ACCT.address)})
    rx = w3.eth.wait_for_transaction_receipt(w3.eth.send_raw_transaction(w3.eth.account.sign_transaction(tx,ACCT.key).rawTransaction), timeout=120)
    return {"status":"OK" if rx.status==1 else "FAIL","gas":rx.gasUsed}
```

### Benign MEV — Backrunning Only (Law I)
```python
def benign_backrun(w3, target_tx):
    """ONLY backrunning — trade AFTER target tx. NO sandwich. Law I: do not harm."""
    sv = int(target_tx["value"]) / 1e18
    if sv < 10: return {"status":"SKIP","why":"too_small"}
    backrun = build_backrun_tx(target_tx)
    backrun["maxFeePerGas"] = int(target_tx["maxFeePerGas"] * 0.95)
    return {"status":"SUBMITTED","bundle":flashbots_send_bundle([target_tx["hash"],backrun])}
# BANNED: sandwich_attack(), frontrunning() — these DO NOT EXIST.
```

### Solana DeFi (ricko-solana-wallet)
```python
def solana_yield(rpc, kp, pool):
    from ricko_solana_wallet import SolanaWallet
    w = SolanaWallet(rpc, kp)
    swap = w.jupiter_swap("USDC","SOL",amount_usdc=100)
    dep = w.orca_deposit(pool, sol_amt, usdc_amt, lo_p, hi_p)
    return {"swap":swap,"deposit":dep}
```

---

## 4. Prop Firm Trading (Secondary)

### AquaFunded Integration
```python
def sync_aqua_risk(acct_info):
    firm = {"max_daily": acct_info["balance"]*0.05, "max_dd": acct_info["balance"]*0.10, "target": acct_info["balance"]*0.08}
    ours = {"max_daily": min(firm["max_daily"], acct_info["balance"]*0.03), "risk_trade": acct_info["balance"]*0.01}
    return ours
def request_payout(acct_id, amount):
    return requests.post("https://api.aquafunded.com/v1/payouts/request", json={"account_id":acct_id,"amount":amount,"method":"crypto","wallet":PAYOUT_WALLET}, headers=auth(), timeout=10).json()
```

---

## 5. P&L Tracking

### Formulas
```python
def realized_pnl(trades): return sum(t["pnl"] for t in trades if t["status"]=="CLOSED")
def unrealized_pnl(pos, marks): return sum((marks[p["sym"]]-p["entry"])*p["qty"]*p["dir"] for p in pos)
def total_equity(bal, r, u): return bal + r + u
def daily_report(td, bal):
    r = realized_pnl(td)
    return {"date":datetime.utcnow().date().isoformat(),"start_bal":bal,"r_pnl":round(r,2),"ret_pct":round((r/bal)*100,4),"n":len(td),"wr":sum(1 for t in td if t["pnl"]>0)/len(td) if td else 0}
```

### Supabase Schema
```sql
CREATE TABLE trade_logs(id SERIAL PRIMARY KEY, ts TIMESTAMPTZ DEFAULT NOW(), sym TEXT, dir TEXT, entry NUMERIC, exit NUMERIC, qty NUMERIC, lev NUMERIC DEFAULT 1, sl NUMERIC, tp NUMERIC, pnl NUMERIC DEFAULT 0, status TEXT DEFAULT 'OPEN', strategy TEXT, source TEXT, tx_hash TEXT, agent TEXT);
CREATE TABLE daily_pnl(date DATE PRIMARY KEY, realized NUMERIC DEFAULT 0, unrealized NUMERIC DEFAULT 0, equity NUMERIC, ret_pct NUMERIC DEFAULT 0, trades INT DEFAULT 0, win_rate NUMERIC DEFAULT 0, max_dd NUMERIC DEFAULT 0);
```

---

## 6. Risk Manager (Sacred)

### Pre-Trade Check
```python
class RiskManager:
    def __init__(self, dl=3.0, mdd=20.0):
        self.dl = dl/100; self.mdd = mdd/100; self.today_pnl = 0; self.peak = 0
        self.paused = False; self.reason = None
    def check(self, bal, sig, risk_pct, positions):
        if self.paused: return {"ok":False,"why":f"PAUSED: {self.reason}"}
        if self.today_pnl <= -bal*self.dl: self._pause("Daily loss limit"); return {"ok":False,"why":"Daily loss 3%"}
        eq = bal + self.today_pnl
        if self.peak > 0 and (self.peak-eq)/self.peak >= self.mdd:
            self._pause("Max drawdown 20%"); return {"ok":False,"why":"Drawdown 20%"}
        if any(p["sym"]==sig.get("sym") for p in positions): return {"ok":False,"why":"Position exists"}
        if is_high_impact_news(): return {"ok":False,"why":"News event"}
        if sig.get("sym")=="XAUUSD" and is_weekend(): return {"ok":False,"why":"Weekend"}
        if sig.get("sym")=="XAUUSD": risk_pct = min(risk_pct, 1.0)
        return {"ok":True,"risk_pct":risk_pct,"risk_amt":bal*risk_pct/100}
    def _pause(self, r): self.paused = True; self.reason = r; log_alert(f"STOP: {r}")
    def reset(self, bal): self.today_pnl = 0; self.peak = max(self.peak, bal)
```

### Emergency Stop
```python
async def emergency_stop(bridge, mexc):
    for p in bridge.get_open_positions().get("positions",[]): bridge.close_position(p["ticket"])
    for p in mexc.positions().get("data",[]): mexc.close(p["symbol"])
    return {"status":"STOP","closed":len(bridge.get_open_positions())+len(mexc.positions())}
```

---

## 7. Execution Flow & Decision Matrix

### Main Cycle
```python
async def revenue_cycle():
    rm = RiskManager(); br = MT5Bridge(); mx = MEXCClient(KEY, SEC)
    if rm.paused: return {"status":"PAUSED","why":rm.reason}
    # XAUUSD
    s = analyze_xauusd(br)
    if s["dir"]!="FLAT" and rm.check(get_bal(),s,1.0,br.get_open_positions())["ok"]:
        z = calc_xauusd_size(get_bal(), s["entry"], get_atr(br.get_rates("M15",100)), s["dir"])
        br.market_order(s["dir"], z["lots"], z["sl"], z["tp"])
    # Crypto perps
    for sym in ["BTC_USDT","ETH_USDT","SOL_USDT"]:
        s = analyze_crypto(mx, sym)
        if s["dir"]!="FLAT" and rm.check(get_cbal(),s,2.0,mx.positions())["ok"]:
            sl = s["e"]*0.97 if s["dir"]=="LONG" else s["e"]*1.03
            tp = s["e"]*1.06 if s["dir"]=="LONG" else s["e"]*0.94
            z = calc_crypto_size(get_cbal(), s["e"], sl)
            mx.order(sym, s["dir"], z["qty"], sl=sl, tp=tp)
    funding_arb(mx)
    return {"status":"DONE"}
```

| Signal | TF | Risk | Lev | SL | TP |
|--------|-----|------|-----|-----|-----|
| XAUUSD | H4+M15 | 1% | 1x | 2xATR | 3xATR |
| BTC/ETH | 1H | 2% | 5x | 3% | 6% |
| SOL | 1H | 2% | 5x | 4% | 8% |
| Funding | 8h | 0.5% | 1x | 1% | 0.5% fee |

**Immutable Rules:** Risk manager vetoes ALL trades. Constitution Law I governs MEV. Max 20% drawdown = emergency stop. Daily loss 3% = pause.
