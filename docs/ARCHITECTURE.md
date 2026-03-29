# Architecture — Spreads Leverage

## Overview

Spreads Leverage allows users to build leveraged positions on STRC (Strategy Preferred Stock xToken) using a looping strategy through Morpho Blue, with Privy smart wallets abstracting all complexity into a single user action.

---

## Why a Wrapper Token (wSTRCx)?

STRC pays dividends that are auto-compounded into the token price by xStocks (rebasing mechanism). This makes STRCx **incompatible with standard money markets** like Morpho — they cannot handle rebasing balances correctly.

Solution: wrap STRCx into **wSTRCx**, a non-rebasing ERC-4626-style wrapper:
- Analogous to wstETH (Lido's wrapper for rebasing stETH)
- wSTRCx balance stays constant; exchange rate to STRCx increases over time
- Fully compatible with Morpho, Euler, and any standard DeFi primitive

```
STRCx (rebasing)  →  [WrappedSTRCx contract]  →  wSTRCx (static balance, appreciating)
```

---

## Leverage Loop

```
┌─────────────────────────────────────────────────────────┐
│                    User: 1 click                         │
└────────────────────────┬────────────────────────────────┘
                         │ Privy smart wallet executes all steps
                         ▼
Step 1:  USDC ──[CoW Protocol swap, ~2 min]──► STRCx
Step 2:  STRCx ──[WrappedSTRCx.wrap()]──► wSTRCx
Step 3:  wSTRCx ──[Morpho deposit]──► collateral position
Step 4:  Morpho ──[borrow at 86% LTV]──► USDC
Step 5:  repeat Steps 1–4 until target leverage

Final state: user holds leveraged wSTRCx in Morpho, single debt position in USDC
```

**Each loop iteration:**
| Start | After swap | After wrap | After deposit | After borrow |
|-------|-----------|-----------|--------------|--------------|
| $1,000 USDC | $1,000 STRCx | $1,000 wSTRCx | collateral: $1,000 | +$860 USDC |
| $860 USDC | $860 STRCx | $860 wSTRCx | collateral: $1,860 | +$740 USDC |
| ... | ... | ... | ... | ... |

Achievable leverage: ~3x with 86% LTV after sufficient iterations.

---

## CoW Swap Timing Risk

CoW orders take ~2 minutes to settle. Between swap submission and fill, the user is exposed to:
- STRC price movement
- Potential liquidation if wSTRCx collateral value drops during loop

**Mitigation:** conservative LTV (86% vs. STRC's historical max ~10% drawdown), loop pauses if health factor drops below threshold.

---

## Smart Contract Architecture

```
WrappedSTRCx.sol
  - ERC-4626 vault wrapping STRCx
  - wrap(uint256 strcxAmount) → wSTRCxAmount
  - unwrap(uint256 wStrcxAmount) → strcxAmount
  - exchangeRate() → current STRCx per wSTRCx

LeverageVault.sol
  - enterPosition(uint256 usdcAmount, uint256 targetLeverage)
      → loops: swap → wrap → deposit → borrow
  - exitPosition()
      → unwinds: repay → withdraw → unwrap → swap back to USDC
  - getHealthFactor(address user) → current Morpho health factor
  - pauseIfAtRisk() → internal safety check between loop iterations

interfaces/
  ISTRCx.sol       — rebasing xToken interface
  IMorpho.sol      — Morpho Blue: supply, borrow, repay, withdraw
  ICoWSwap.sol     — CoW Protocol GPv2Settlement order submission
```

---

## Frontend Architecture

```
/ (page.tsx)
  ├── ConnectWallet        — Privy embedded wallet (email/social login, gasless)
  ├── LeverageInput        — USDC amount + target leverage slider (1x–3x)
  ├── PositionPreview      — estimated final position, APR, liquidation price
  ├── EnterButton          — triggers LeverageVault.enterPosition() via Privy
  └── PositionDashboard    — live health factor, collateral value, debt, exit button
```

---

## Open Technical Questions

1. **MetaMorpho vs. Morpho Blue direct?** — MetaMorpho adds a curator layer; Morpho Blue direct is simpler for a hackathon POC. Preference: direct Morpho Blue unless tinkk0 has a MetaMorpho market already deployed on INK.

2. **Atomic bundling of wrap+deposit+borrow?** — Morpho has a Bundler contract that can atomize multiple calls. Need to verify it's deployed on INK mainnet. If yes, Steps 2–4 can be 1 transaction.

3. **Weekend liquidation risk** — xStocks has no price feed on weekends. Morpho can still liquidate based on last known price. Need a health factor buffer or weekend-aware logic.

4. **CoW Protocol on INK** — Need to verify GPv2Settlement is deployed on INK. If not, fall back to 1inch or direct Uniswap swap.
