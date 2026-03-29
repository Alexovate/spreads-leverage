# Spreads Leverage — STRC Leveraged Positions on INK

> 1-click leveraged exposure to STRC (Strategy Preferred Stock) using Morpho Blue as the money market and CoW Protocol for swaps. Built on INK Chain (Kraken's L2).

**xStocks Hackathon — ETH Cannes 2026**
> Code development starts: **April 1, 09:00 CET** (hackathon rule: only code produced during the event counts)

---

## What It Does

STRC is Michael Saylor's preferred stock instrument (~$100 peg, 11.5% annualized dividend yield, $45B market cap). Getting leveraged exposure to STRC today requires institutional access. This protocol opens that up on-chain in a single click.

**The loop:**
```
User deposits USDC
        ↓
Swap USDC → STRCx via CoW Protocol (~2 min)
        ↓
Wrap STRCx → wSTRCx (non-rebasing, Morpho-compatible)
        ↓
Deposit wSTRCx as collateral into Morpho Blue
        ↓
Borrow USDC at 86% LTV
        ↓
Repeat until target leverage reached (up to ~3x)
        ↓
User holds leveraged wSTRCx position
```

All steps are abstracted via a **Privy smart wallet** — the user signs once, everything else is executed in the background (gasless).

---

## Why This Matters

- Borrow rates on Morpho are cheaper than perpetual funding rates — leverage via money market is structurally better
- STRC is a rebasing token (dividends auto-compound into price) → needs a wrapper (like wstETH) to be compatible with money markets
- Nobody has built this cleanly for tokenized equities yet
- Institutional product, previously gated — now accessible on-chain

---

## Stack

| Layer | Tech |
|-------|------|
| Chain | INK (Kraken's L2, Optimism-based) |
| Contracts | Solidity, Foundry |
| Money Market | Morpho Blue |
| Swap | CoW Protocol |
| Wallet Abstraction | Privy smart wallet |
| Frontend | Next.js 14, TailwindCSS |
| Token Wrapper | wSTRCx (ERC-4626-style, analogous to wstETH) |

---

## Team

| Name | Role |
|------|------|
| Alex ([@alexovate](https://github.com/alexovate)) | Smart contracts + full-stack |
| Waj | Product, strategy (founder of Spreads) |
| Kevin | Financial strategy, product |
| tinkk0 | CTO of Spreads (remote) |

---

## Setup (available after April 1)

```bash
# Contracts
cd contracts
forge install
forge build
forge test

# Frontend
cd frontend
npm install
npm run dev
```

---

## Architecture

See [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) for the full system design.
See [`docs/TECHNICAL_PLAN.md`](docs/TECHNICAL_PLAN.md) for technical decisions and open questions.
