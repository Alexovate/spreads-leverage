# Technical Plan — Spreads Leverage

## Technology Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Chain | INK mainnet | Kraken's L2, xStocks native, Spreads already deployed here |
| Money market | Morpho Blue | Simpler than MetaMorpho for POC, great docs, battle-tested |
| Swap | CoW Protocol | Best execution, MEV protection — verify deployment on INK |
| Local testing | Foundry (fork mode) | `forge test --fork-url <INK_RPC>` to test against live state |
| Wallet abstraction | Privy | Already used by Spreads, gasless UX, 1-click entry |
| Token standard | ERC-4626 for wSTRCx | Standard interface, compatible with all DeFi |

---

## Build Priority (for 48h hackathon)

### Must ship (Day 1 — Tuesday)
1. `WrappedSTRCx.sol` — core primitive, everything else depends on it
2. `LeverageVault.sol` — enter/exit logic (can mock CoW swap with direct call if needed)
3. Morpho market setup (or verify existing INK market for wSTRCx collateral / USDC debt)

### Should ship (Day 1–2)
4. Basic frontend: connect wallet, enter position, show health factor
5. Foundry tests for WrappedSTRCx and LeverageVault

### Nice to have (Day 2 if time allows)
6. Exit position with full unwind
7. Health factor monitoring + warning UI
8. Weekend liquidation protection logic

---

## LTV Analysis

STRC trades around $100 (soft-pegged). Historical worst drawdowns:
- 10 Oct crash: ~$90 (10% down)
- Jan 2026 crash: similar range

**Chosen LTV: 86%** (matches existing xStocks markets)
- At 86% LTV, STRC would need to drop >14% to approach liquidation
- During a 2-min CoW swap window, this is extremely unlikely
- Borrow rate on Morpho expected to be significantly below perp funding rates (~11.5% APR STRC yield)

---

## wSTRCx Exchange Rate

STRC dividends auto-compound into the STRCx price via xStocks' rebasing mechanism.
wSTRCx wraps this: 1 wSTRCx = increasing amount of STRCx over time.

```
exchangeRate = totalSTRCxInVault / totalWSTRCxSupply
```

No manual rate updates needed — rate increases automatically as xStocks compounds dividends into STRCx.

---

## Devpost & Repository Rules

- [x] Register on Devpost (Alex ✓, Kevin + Waj: check before Tuesday)
- [ ] Create project on Devpost: Tuesday April 1, 09:00 CET
- [ ] Link this GitHub repo to Devpost submission
- [ ] **First commit with actual code**: April 1, 09:00 CET (not before!)
- [ ] Commit regularly throughout — judges check git history
- [ ] Submit draft on Devpost early, update before deadline
- [ ] **Deadline**: Thursday April 2, 11:00 CET (xStocks hackathon)

---

## Role Split (proposed)

| Person | Focus |
|--------|-------|
| Alex | `WrappedSTRCx.sol`, `LeverageVault.sol`, Next.js frontend scaffolding |
| tinkk0 | Morpho market setup on INK, Privy integration, bundler research |
| Kevin | LTV analysis, yield calculations, product narrative / pitch |
| Waj | Product decisions, Spreads integration context, judging criteria alignment |

---

## Open Questions for the Team

1. **@tinkk0** — Is there already a Morpho Blue market on INK for wSTRCx collateral? Or do we need to create one?
2. ✅ **CoW Protocol on INK confirmed** — only DEX available, Spreads.fi already routes through it
3. ✅ **Morpho Blue on INK mainnet confirmed** (tinkk0, 2026-03-29)
4. **@tinkk0** — Morpho Bundler deployed on INK? (Needed for atomic wrap+deposit+borrow — status unknown)
5. **@Waj** — Does Spreads already have a Privy app ID we can reuse for the hackathon?
