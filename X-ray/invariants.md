# Invariant Map

> Midnight | 32 guards | 14 inferred | 4 not enforced on-chain

---

## 1. Enforced Guards (Reference)

Per-call preconditions. Heading IDs below (`G-N`) are anchor targets from x-ray.md attack surfaces.

---

#### G-1

> `require(taker == msg.sender || isAuthorized[taker][msg.sender], TakerUnauthorized());`

**Location** · `src/Midnight.sol:346`

**Purpose** · Prevents third parties from consuming offers against another taker's position without delegated authorization.

---

#### G-2

> `require(_marketState.lossFactor < type(uint128).max, MarketLossFactorMaxedOut());`

**Location** · `src/Midnight.sol:349`

**Purpose** · Blocks new trade settlement once a market has reached terminal bad-debt loss.

---

#### G-3

> `require(UtilsLib.atMostOneNonZero(offer.maxAssets, offer.maxUnits), MultipleNonZero());`

**Location** · `src/Midnight.sol:350`

**Purpose** · Keeps offer consumption accounting in exactly one unit domain.

---

#### G-4

> `require(offer.tick % _marketState.tickSpacing == 0, TickNotAccessible());`

**Location** · `src/Midnight.sol:351`

**Purpose** · Enforces the market's active tick grid.

---

#### G-5

> `require(block.timestamp >= offer.start, OfferNotStarted());`

**Location** · `src/Midnight.sol:352`

**Purpose** · Prevents settlement before the maker's signed offer validity window opens.

---

#### G-6

> `require(block.timestamp <= offer.expiry, OfferExpired());`

**Location** · `src/Midnight.sol:353`

**Purpose** · Prevents settlement after the maker's signed offer validity window closes.

---

#### G-7

> `require(isAuthorized[offer.maker][offer.ratifier], RatifierUnauthorized());`

**Location** · `src/Midnight.sol:355`

**Purpose** · Requires the maker to opt into the ratifier that validates the offer.

---

#### G-8

> `require(IRatifier(offer.ratifier).isRatified(offer, ratifierData) == CALLBACK_SUCCESS, RatifierFail());`

**Location** · `src/Midnight.sol:356`

**Purpose** · Binds successful takes to an external ratifier's exact approval result.

---

#### G-9

> `require(newConsumed <= offer.maxAssets, ConsumedAssets());`

**Location** · `src/Midnight.sol:369`

**Purpose** · Caps assets-based group consumption at the maker's signed limit.

---

#### G-10

> `require(newConsumed <= offer.maxUnits, ConsumedUnits());`

**Location** · `src/Midnight.sol:372`

**Purpose** · Caps units-based group consumption at the maker's signed limit.

---

#### G-11

> `require(block.timestamp <= offer.market.maturity || sellerDebtIncrease == 0, CannotIncreaseDebtPostMaturity());`

**Location** · `src/Midnight.sol:391`

**Purpose** · Prevents post-maturity trades from increasing debt.

---

#### G-12

> `require(!offer.reduceOnly || (offer.buy ? buyerCreditIncrease == 0 : sellerDebtIncrease == 0), MakerCreditOrDebtIncreased());`

**Location** · `src/Midnight.sol:392`

**Purpose** · Enforces maker-signed reduce-only semantics.

---

#### G-13

> `require(offer.market.enterGate == address(0) || buyerCreditIncrease == 0 || IEnterGate(offer.market.enterGate).canIncreaseCredit(buyer), BuyerGatedFromIncreasingCredit());`

**Location** · `src/Midnight.sol:397`

**Purpose** · Lets market-specific gates block credit increases.

---

#### G-14

> `require(offer.market.enterGate == address(0) || sellerDebtIncrease == 0 || IEnterGate(offer.market.enterGate).canIncreaseDebt(seller), SellerGatedFromIncreasingDebt());`

**Location** · `src/Midnight.sol:402`

**Purpose** · Lets market-specific gates block debt increases.

---

#### G-15

> `require(liquidationLocked(id, seller) || isHealthy(offer.market, id, seller), SellerIsLiquidatable());`

**Location** · `src/Midnight.sol:476`

**Purpose** · Ensures a seller is healthy after trade/callback execution unless still in the transient liquidation lock.

---

#### G-16

> `require(onBehalf == msg.sender || isAuthorized[onBehalf][msg.sender], Unauthorized());`

**Location** · `src/Midnight.sol:482`

**Purpose** · Restricts withdraws to the account owner or an authorized delegate.

---

#### G-17

> `require(onBehalf == msg.sender || isAuthorized[onBehalf][msg.sender], Unauthorized());`

**Location** · `src/Midnight.sol:505`

**Purpose** · Restricts repayments on behalf of another user to authorized delegates.

---

#### G-18

> `require(UtilsLib.countBits(newCollateralBitmap) <= MAX_COLLATERALS_PER_BORROWER, TooManyActivatedCollaterals());`

**Location** · `src/Midnight.sol:538`

**Purpose** · Bounds borrower collateral iteration costs.

---

#### G-19

> `require(isHealthy(market, id, onBehalf), UnhealthyBorrower());`

**Location** · `src/Midnight.sol:568`

**Purpose** · Blocks collateral withdrawal that would leave borrower debt undercollateralized.

---

#### G-20

> `require(UtilsLib.atMostOneNonZero(repaidUnits, seizedAssets), InconsistentInput());`

**Location** · `src/Midnight.sol:595`

**Purpose** · Forces liquidation input into either repay-driven or seize-driven mode.

---

#### G-21

> `require(_position.debt > 0, NotBorrower());`

**Location** · `src/Midnight.sol:596`

**Purpose** · Prevents no-op liquidation of accounts without debt.

---

#### G-22

> `require(market.liquidatorGate == address(0) || ILiquidatorGate(market.liquidatorGate).canLiquidate(msg.sender), LiquidatorGatedFromLiquidating());`

**Location** · `src/Midnight.sol:597`

**Purpose** · Lets market-specific gates restrict liquidation callers.

---

#### G-23

> `require(!liquidationLocked(id, borrower) && (postMaturityMode ? block.timestamp > market.maturity : originalDebt > maxDebt), NotLiquidatable());`

**Location** · `src/Midnight.sol:620`

**Purpose** · Restricts liquidation to unlocked, matured or unhealthy borrower states.

---

#### G-24

> `require(repaidUnits <= maxRepaid || _position.collateral[collateralIndex].mulDivDown(liquidatedCollatPrice, ORACLE_PRICE_SCALE).mulDivDown(WAD, lif).zeroFloorSub(maxRepaid) < market.rcfThreshold, RecoveryCloseFactorConditionsViolated());`

**Location** · `src/Midnight.sol:662`

**Purpose** · Enforces the normal-mode recovery close factor unless dust collateral disables it.

---

#### G-25

> `require(amount >= consumed[onBehalf][group], AlreadyConsumed());`

**Location** · `src/Midnight.sol:725`

**Purpose** · Makes direct offer cancellation/consumption monotonic.

---

#### G-26

> `require(market.maturity <= block.timestamp + 100 * 365 days, MaturityTooFar());`

**Location** · `src/Midnight.sol:758`

**Purpose** · Bounds market time-to-maturity at creation.

---

#### G-27

> `require(market.collateralParams.length > 0, NoCollateralParams());`

**Location** · `src/Midnight.sol:759`

**Purpose** · Prevents creation of collateral-less markets.

---

#### G-28

> `require(market.collateralParams.length <= MAX_COLLATERALS, TooManyCollateralParams());`

**Location** · `src/Midnight.sol:760`

**Purpose** · Bounds market collateral loops.

---

#### G-29

> `require(collateralToken > previousCollateralToken, CollateralParamsNotSorted());`

**Location** · `src/Midnight.sol:764`

**Purpose** · Enforces strictly sorted, nonzero, unique collateral tokens.

---

#### G-30

> `require(isLltvAllowed(lltv), LltvNotAllowed());`

**Location** · `src/Midnight.sol:766`

**Purpose** · Restricts market LLTV to predefined tiers.

---

#### G-31

> `require(market.collateralParams[i].maxLif == maxLif(lltv, LIQUIDATION_CURSOR_LOW) || market.collateralParams[i].maxLif == maxLif(lltv, LIQUIDATION_CURSOR_HIGH), InvalidMaxLif());`

**Location** · `src/Midnight.sol:767`

**Purpose** · Binds max LIF to the allowed LLTV/cursor formula.

---

#### G-32

> `require(marketState[id].tickSpacing > 0, MarketNotCreated());`

**Location** · `src/Midnight.sol:825`

**Purpose** · Prevents position updates for markets that have not been created.

---

## 2. Inferred Invariants (Single-Contract)

#### I-1

`Conservation` · On-chain: **Yes**

> A repayment decreases `position[id][onBehalf].debt` and increases `marketState[id].withdrawable` by the same units.

**Derivation** — Δ-pair: `src/Midnight.sol:508` ↔ `src/Midnight.sol:509`.

**If violated** — Repayment would desynchronize debt burn from withdrawable loan-token accounting.

---

#### I-2

`Conservation` · On-chain: **Yes**

> A withdraw decreases user credit, market withdrawable, and total units by the withdrawn units.

**Derivation** — Δ-pair: `src/Midnight.sol:493` ↔ `src/Midnight.sol:494` ↔ `src/Midnight.sol:495`.

**If violated** — Loan-token withdrawals could separate user credit from market liabilities.

---

#### I-3

`Conservation` · On-chain: **Yes**

> A non-bad-debt liquidation repayment decreases borrower debt and increases market withdrawable by the same repaid units.

**Derivation** — Δ-pair: `src/Midnight.sol:675` ↔ `src/Midnight.sol:676`.

**If violated** — Liquidation repayments would not become withdrawable to lenders.

---

#### I-4

`Bound` · On-chain: **Yes**

> `defaultContinuousFee` and per-market `continuousFee` are bounded by `MAX_CONTINUOUS_FEE`.

**Derivation** — guard-lift: `src/Midnight.sol:290`, `src/Midnight.sol:299`; write sites `src/Midnight.sol:293`, `src/Midnight.sol:301`, market initialization `src/Midnight.sol:785`.

**If violated** — Continuous fee accrual could exceed the documented cap.

---

#### I-5

`Bound` · On-chain: **Yes**

> Settlement fee breakpoints are capped by `maxSettlementFee(index)` and stored as cbp multiples.

**Derivation** — guard-lift: `src/Midnight.sol:262-263`, `src/Midnight.sol:280-281`; write sites `src/Midnight.sol:267-273`, `src/Midnight.sol:283`, market initialization `src/Midnight.sol:778-784`.

**If violated** — Piecewise settlement fees could exceed configured per-index maxima or lose cbp precision assumptions.

---

#### I-6

`StateMachine` · On-chain: **Yes**

> A market transitions from uncreated (`tickSpacing == 0`) to created (`tickSpacing = DEFAULT_TICK_SPACING`) once.

**Derivation** — edge: `tickSpacing == 0@src/Midnight.sol:757` → `DEFAULT_TICK_SPACING@src/Midnight.sol:776`; no code path writes it back to zero.

**If violated** — Market identity and SSTORE2-backed reconstruction would become ambiguous.

---

#### I-7

`Bound` · On-chain: **Yes**

> User activated collateral bitmap cardinality is capped at `MAX_COLLATERALS_PER_BORROWER`.

**Derivation** — guard-lift: `src/Midnight.sol:538`; write sites `src/Midnight.sol:537`, `src/Midnight.sol:565`, `src/Midnight.sol:673`.

**If violated** — Health and liquidation loops could exceed the intended borrower collateral bound.

---

#### I-8

`Bound` · On-chain: **Yes**

> `consumed[maker][group]` is monotonic non-decreasing.

**Derivation** — guard-lift: direct setter guard `src/Midnight.sol:725` plus take increments at `src/Midnight.sol:368` and `src/Midnight.sol:371`.

**If violated** — Makers could have signed offer caps reused after consumption is lowered.

---

#### I-9

`Temporal` · On-chain: **Yes**

> New debt cannot be added through `take` after market maturity.

**Derivation** — temporal: `src/Midnight.sol:391`.

**If violated** — Post-maturity debt accounting would not remain settlement-only.

---

#### I-10

`Temporal` · On-chain: **Yes**

> Liquidation post-maturity mode is only available strictly after maturity.

**Derivation** — temporal: `src/Midnight.sol:620-623`.

**If violated** — Pre-maturity liquidations could use the post-maturity LIF path.

---

#### I-11

`Ratio` · On-chain: **Yes**

> Continuous fee accrual is proportional to pending fee over elapsed time until maturity.

**Derivation** — ratio: `src/Midnight.sol:811-815`, then persisted at `src/Midnight.sol:842-846`.

**If violated** — Lender fee accrual would drift from the documented fixed-rate fee schedule.

---

#### I-12

`Conservation` · On-chain: **No**

> Flash loans have no internal fee or accounting delta; repayment depends entirely on token transfer round trip.

**Derivation** — conservation-negative: `src/Midnight.sol:741-750` emits/transfers/callbacks but writes no storage.

**If violated** — The flash-loaned token balance check is delegated to external token transfer semantics only.

---

#### I-13

`Bound` · On-chain: **No**

> Bundler `takes[0]` indexing assumes each bundle call receives a nonempty takes array.

**Derivation** — guard-lift gap: reads at `src/periphery/MidnightBundles.sol:62`, `129`, `193`, `264`; no explicit `takes.length > 0` guard in those entry points.

**If violated** — Bundle calls rely on Solidity bounds checks rather than named protocol errors or documented preconditions.

---

#### I-14

`Bound` · On-chain: **No**

> EIP-712 signatures in ratifiers and authorizer bind to `block.chainid`, while Midnight market ids bind to constructor `INITIAL_CHAIN_ID`.

**Derivation** — NatSpec: `src/Midnight.sol:173-175`, `src/periphery/EcrecoverAuthorizer.sol:14-15`, `src/ratifiers/EcrecoverRatifier.sol:10-11`; code: `src/Midnight.sol:871-872`, `src/periphery/EcrecoverAuthorizer.sol:29`, `src/ratifiers/EcrecoverRatifier.sol:40`.

**If violated** — Hard-fork behavior differs between market identity and off-chain authorization domains.

---

## 3. Inferred Invariants (Cross-Contract)

#### X-1

On-chain: **Yes**

> `Midnight.take` assumes ratifiers return exactly `CALLBACK_SUCCESS` for valid offers and revert/return otherwise.

**Caller side** — `src/Midnight.sol:355-356` — maker authorizes ratifier and `take` checks exact callback success.

**Callee side** — `src/ratifiers/EcrecoverRatifier.sol:33-45`, `src/ratifiers/SetterRatifier.sol:30-36` — both require `msg.sender == MIDNIGHT` and return `CALLBACK_SUCCESS` only after proof/signature or stored-root checks.

**If violated** — Offer validation would no longer be equivalent to maker-authorized ratification.

---

#### X-2

On-chain: **No**

> `Midnight.isHealthy` and `liquidate` assume oracle `price()` returns bounded, current values scaled by `ORACLE_PRICE_SCALE`.

**Caller side** — `src/Midnight.sol:610-615`, `src/Midnight.sol:953-955` — prices feed max-debt, bad-debt, and health math.

**Callee side** — `src/interfaces/IOracle.sol:6` — oracle behavior is an out-of-scope interface with no freshness or bound methods.

**If violated** — Solvency and liquidation calculations inherit the oracle's scale, freshness, and revert behavior.

---

#### X-3

On-chain: **No**

> `Midnight.take` assumes optional enter gates correctly decide credit/debt increases.

**Caller side** — `src/Midnight.sol:397-405` — gate return values decide whether credit or debt can increase.

**Callee side** — `src/interfaces/IGate.sol` — gates are out-of-scope interfaces and can revert or return false by policy.

**If violated** — Market access control becomes the external gate's semantics.

---

#### X-4

On-chain: **No**

> Token movements assume ERC20 transfers move exactly the requested amount and do not re-enter.

**Caller side** — `src/Midnight.sol:455-456`, `src/Midnight.sol:520`, `src/Midnight.sol:545`, `src/Midnight.sol:572`, `src/Midnight.sol:696-717`, `src/Midnight.sol:743-750`.

**Callee side** — `src/libraries/SafeTransferLib.sol:12-33` checks code and boolean return only; exact amount and non-reentrancy are documented token assumptions.

**If violated** — Internal accounting can diverge from real token balances.

---

## 4. Economic Invariants

#### E-1

On-chain: **Yes**

> Loan-token withdrawability follows repayment/liquidation inflows and withdraw/fee-claim outflows.

**Follows from** — `I-1` + `I-2` + `I-3`.

**If violated** — The market's settlement liability would not track realized loan-token inflows.

---

#### E-2

On-chain: **No**

> Solvency depends on internal accounting plus exact ERC20 transfer behavior.

**Follows from** — `I-1` + `I-2` + `I-3` + `X-4`.

**If violated** — Balance-based solvency can fail even when internal deltas are paired.

