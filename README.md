# Limit Break AMM — Audit Journey

> First real security contest. Hardest format. Full documentation.
> This repository does not disclose vulnerabilities or confidential 
> contest information. It documents personal audit methodology and 
> learning outcomes only.

---

## Contest

| | |
|---|---|
| **Protocol** | Limit Break AMM |
| **Format** | Guardian Defender |
| **Prize Pool** | $150,000 |
| **Period** | Feb 23 – Apr 9, 2026 |
| **My Hours** | ~45 hours |
| **Confirmed Findings** | 0 |
| **Candidates Investigated** | 35+ |
| **False Positives Submitted** | 0 |

---

## What is Guardian Defender?

Guardian Defender is the hardest contest format that exists.

Before the public contest opens, **5 senior auditors spend 3 months doing a paid engagement**. They produce a full report with Critical, High, Medium, Low findings across 3 audit rounds. Then the contest opens — and participants are asked to find what Guardian **missed**.

This was my first contest. I picked the hardest format on purpose.

---

## Why 0 Findings ≠ Failure

Most first-contest auditors do this:

- Hunt 2-3 days
- Submit 3-5 candidates hoping something sticks
- 0-1 valid, rest are false positives
- Learn nothing systematic

I did this:

- 45 hours structured investigation
- 35+ candidates with documented hypothesis and kill reason
- 0 submitted because discipline > spray and pray
- 98% codebase covered
- Every discard has a documented reasoning

The discipline to **not** submit a false positive is harder than finding bugs. That's what this contest taught me.

---

## What I Investigated

### Files Exhausted (✅=clean, ❌=permanent blacklist)

```
✅ SecureProxy.sol
✅ CLOBTransferHandler.sol
✅ AMMStandardHook.sol
✅ PermitTransferHandler.sol
✅ AMMModule.sol — swap, fee, hook paths
✅ ModuleLiquidity.sol
✅ ModuleFeeCollection.sol
✅ ModuleAdmin.sol
✅ FeeHelper.sol
✅ DynamicPoolType.sol
✅ DynamicHelper.sol — computeSwap, modifyPosition, _crossTick
✅ FixedPoolType.sol (entry point)
✅ FixedHelper.sol — addLiquidity path, getFeeGrowthInside
✅ SingleProviderPoolType.sol
✅ SingleProviderHelper.sol
✅ CreatorHookSettingsRegistry.sol
✅ PoolDecoder.sol
✅ tm-core-lib (Signatures, TstorishReentrancyGuard)
✅ Swap settlement + reentrancy architecture
✅ Malicious hook operator scenarios
✅ CLOB + multi-hop interaction
❌ FixedHelper.sol swap path — BLACKLISTED (Guardian covered exhaustively)
```

---

## Skills Built

### Technical

| Skill | What I Can Do Now |
|---|---|
| Architectural Reading | Read complex multi-module codebases in hours |
| Hook System Design | Understand supportedFlags, execution ordering, hook flag boundaries |
| Uniswap V3 Internals | tick crossing, feeGrowthOutside init, modifyPosition ordering |
| Fee Accounting | 3 isolated key spaces, delta-based collection, underflow protection |
| Multi-hop Architecture | Shared swapCache design, partial fill guard patterns |
| Assembly Safety | Validate inline assembly in hook execution paths |
| Audit Report Reading | Extract scope, status, root cause, fix recommendation professionally |

### Methodology

- **Fail fast discipline** — 90 minutes per candidate max, then park
- **Guardian-aware auditing** — check known issues before deep-diving  
- **Cross-module thinking** — bugs emerge from interactions, not individual functions
- **Economic attack mindset** — beyond code correctness, into "can this be abused?"
- **Hypothesis-driven** — specific question first, then code lookup
- **False positive filter** — if 5 senior auditors missed it and it looks obvious → false positive

---

## The Mindset Shift That Matters

> **Guardian defends with:** *"Is this function correct?"*
>
> **Attackers win with:** *"Is this system still correct after 7 valid steps?"*

Bugs in well-audited codebases are almost never "forgot a require."  
They are emergent behaviors from the interaction of multiple individually-correct components.

Real examples:
- **Nomad Bridge ($190M):** `process()` correct. `proveAndProcess()` correct. Call `process()` without `proveAndProcess()` first → bypass validation.
- **Qubit Finance ($80M):** Deposit correct. Mint correct. Deposit to zero address → mint free collateral.

This is what I'm now trained to hunt.

---

## Key Architectural Insights (Limit Break AMM)

For anyone building on or auditing this protocol later:

1. **Pool type executes BEFORE hooks** in all liquidity operations — by design
2. **`LBAMM__CannotPartialFillAfterFirstHop`** at AMMModule.sol line 1565 kills partial fill mid-route
3. **`_supportedHookFlags` is an explicit whitelist** — `REMOVE_LIQUIDITY` and `COLLECT_FEES` flags are NOT supported in AMMStandardHook. LP can always exit.
4. **Three isolated fee key spaces** — cannot cross-claim between hook-managed, token-managed, and general
5. **DynamicPoolType = Uniswap V3 clone** — computeSwap atomic commit, afterSwap reads final settled state
6. **CLOB and AMM multi-hop are architecturally separate** — no intersection via `_directSwap` vs `_poolSwapByInput`

---

## Documentation

Full audit notes including all 35+ candidates with verdict and reasoning:  
→ [`LimitBreakAMM_Audit_Documentation.pdf`](./LimitBreakAMM_Audit_Documentation.pdf)

---

## What's Next

This contest was training. Not for this codebase — for every codebase after.

Next target: normal contest format (Code4rena / Sherlock), no prior Guardian engagement, apply cross-module emergent bug methodology from day one.

---  
*Follow the journey: [@gandalfbuilder](https://x.com/gandalfbuilder)*
