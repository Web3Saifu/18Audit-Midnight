# Entry Point Map

> Midnight | 31 entry points | 21 permissionless/self-authorized | 6 role-gated | 4 admin-only

---

## Protocol Flow Paths

### Setup

`constructor()` → `setFeeSetter()` / `setFeeClaimer()` / `setTickSpacingSetter()`

`[role setup above]` → `setDefaultSettlementFee()` / `setDefaultContinuousFee()` → first `touchMarket()`

### Market Creation

`touchMarket()` ◄── first interaction with market params creates immutable market data

`touchMarket()` → `setMarketSettlementFee()` / `setMarketContinuousFee()` / `setMarketTickSpacing()`

### Offer Taking

`setIsAuthorized(maker, ratifier)` → `EcrecoverRatifier.cancelRoot()` or `SetterRatifier.setIsRootRatified()` → `take()`

`take()` → `IRatifier.isRatified()` → buyer/seller callbacks → ERC20 transfers → seller health check

### Position Lifecycle

`supplyCollateral()` → `take()` ◄── debt or credit created from signed offer settlement

`repay()` → `withdraw()` ◄── repay increases withdrawable units

`withdrawCollateral()` ◄── borrower must remain healthy unless debt is zero

`updatePosition()` ◄── market already created; slashes/accrues fees lazily

### Liquidation & Fees

`take()` / `supplyCollateral()` → unhealthy or matured borrower → `liquidate()` ◄── optional liquidator gate may restrict caller

`take()` → `claimSettlementFee()`; `updatePosition()` → `claimContinuousFee()`

### Periphery

`setIsAuthorized(taker, MidnightBundles)` → bundle function → `touchMarket()` → `take()` / `supplyCollateral()` / `withdrawCollateral()` / `repay()`

---

## Permissionless

### `Midnight.multicall()`

| Aspect | Detail |
|--------|--------|
| Visibility | external |
| Caller | Any caller batching calls as itself |
| Parameters | calls (user-controlled) |
| Call chain | `→ Midnight.delegatecall(self)` |
| State modified | Any called Midnight function |
| Value flow | Depends on subcall |
| Reentrancy guard | no |

### `Midnight.take()`

| Aspect | Detail |
|--------|--------|
| Visibility | external |
| Caller | Taker or authorized delegate |
| Parameters | offer (maker-signed/user-provided), ratifierData (user-provided), units/taker/receiver/callbacks (user-controlled) |
| Call chain | `→ touchMarket()` → `IRatifier.isRatified()` → callbacks → ERC20 transfers → `isHealthy()` |
| State modified | `consumed`, buyer/seller `position`, `marketState.totalUnits`, `claimableSettlementFee`, transient liquidation lock |
| Value flow | Loan token from payer to Midnight and seller receiver |
| Reentrancy guard | transient liquidation lock only |

### `Midnight.withdraw()`

| Aspect | Detail |
|--------|--------|
| Visibility | external |
| Caller | Account owner or authorized delegate |
| Parameters | market/units/onBehalf/receiver (user-controlled) |
| Call chain | `→ touchMarket()` → `_updatePosition()` → ERC20 transfer |
| State modified | `position.credit`, `position.pendingFee`, `marketState.withdrawable`, `marketState.totalUnits`, `continuousFeeCredit` via update |
| Value flow | Loan token out to receiver |
| Reentrancy guard | no |

### `Midnight.repay()`

| Aspect | Detail |
|--------|--------|
| Visibility | external |
| Caller | Account owner or authorized delegate |
| Parameters | market/units/onBehalf/callback/data (user-controlled) |
| Call chain | `→ touchMarket()` → optional repay callback → ERC20 transferFrom |
| State modified | `position.debt`, `marketState.withdrawable` |
| Value flow | Loan token in from payer |
| Reentrancy guard | no |

### `Midnight.supplyCollateral()`

| Aspect | Detail |
|--------|--------|
| Visibility | external |
| Caller | Account owner or authorized delegate |
| Parameters | market/collateralIndex/assets/onBehalf (user-controlled) |
| Call chain | `→ touchMarket()` → ERC20 transferFrom |
| State modified | `position.collateral`, `position.collateralBitmap` |
| Value flow | Collateral token in |
| Reentrancy guard | no |

### `Midnight.withdrawCollateral()`

| Aspect | Detail |
|--------|--------|
| Visibility | external |
| Caller | Account owner or authorized delegate |
| Parameters | market/collateralIndex/assets/onBehalf/receiver (user-controlled) |
| Call chain | `→ touchMarket()` → `isHealthy()` → ERC20 transfer |
| State modified | `position.collateral`, `position.collateralBitmap` |
| Value flow | Collateral token out |
| Reentrancy guard | no |

### `Midnight.liquidate()`

| Aspect | Detail |
|--------|--------|
| Visibility | external |
| Caller | Any liquidator allowed by market gate |
| Parameters | market/collateralIndex/seizedAssets/repaidUnits/borrower/mode/receiver/callback/data (user-controlled) |
| Call chain | `→ touchMarket()` → oracle reads → optional callback → ERC20 transfers |
| State modified | borrower `debt`, `collateral`, `collateralBitmap`; `marketState.lossFactor`, `totalUnits`, `withdrawable`, `continuousFeeCredit` |
| Value flow | Collateral out, loan token in |
| Reentrancy guard | no |

### `Midnight.setConsumed()`

| Aspect | Detail |
|--------|--------|
| Visibility | external |
| Caller | Account owner or authorized delegate |
| Parameters | group/amount/onBehalf (user-controlled) |
| Call chain | `→ consumed write` |
| State modified | `consumed[onBehalf][group]` |
| Value flow | none |
| Reentrancy guard | no |

### `Midnight.setIsAuthorized()`

| Aspect | Detail |
|--------|--------|
| Visibility | external |
| Caller | Account owner or authorized delegate |
| Parameters | authorized/newIsAuthorized/onBehalf (user-controlled) |
| Call chain | `→ isAuthorized write` |
| State modified | `isAuthorized[onBehalf][authorized]` |
| Value flow | none |
| Reentrancy guard | no |

### `Midnight.flashLoan()`

| Aspect | Detail |
|--------|--------|
| Visibility | external |
| Caller | Anyone |
| Parameters | tokens/assets/callback/data (user-controlled) |
| Call chain | `→ ERC20 transfers out` → `IFlashLoanCallback.onFlashLoan()` → ERC20 transferFrom in |
| State modified | none |
| Value flow | Tokens out and back in |
| Reentrancy guard | no |

### `Midnight.touchMarket()`

| Aspect | Detail |
|--------|--------|
| Visibility | public |
| Caller | Anyone |
| Parameters | market (user-controlled) |
| Call chain | `→ IdLib.storeInCode()` |
| State modified | `marketState[id]` if first touch |
| Value flow | none |
| Reentrancy guard | no |

### `Midnight.updatePosition()`

| Aspect | Detail |
|--------|--------|
| Visibility | external |
| Caller | Anyone |
| Parameters | market/user (user-controlled) |
| Call chain | `→ _updatePosition()` |
| State modified | `position.credit`, `pendingFee`, `lastLossFactor`, `lastAccrual`, `marketState.continuousFeeCredit` |
| Value flow | none |
| Reentrancy guard | no |

### `MidnightBundles.*`

| Aspect | Detail |
|--------|--------|
| Visibility | external |
| Caller | Taker/onBehalf or authorized delegate |
| Parameters | targets, takes, permits, withdrawals, fee params (user-controlled/user-signed) |
| Call chain | `→ pullToken()` → `Midnight.touchMarket()` → `Midnight.take/repay/supplyCollateral/withdrawCollateral()` → ERC20 transfers |
| State modified | Midnight state through downstream calls; no persistent Bundles state |
| Value flow | Loan/collateral tokens in/out, optional referral fee |
| Reentrancy guard | no |

### `EcrecoverAuthorizer.setIsAuthorized()`

| Aspect | Detail |
|--------|--------|
| Visibility | external |
| Caller | Anyone with valid signer authorization |
| Parameters | authorization (user-signed), signature (user-signed) |
| Call chain | `→ ecrecover()` → `Midnight.setIsAuthorized()` |
| State modified | `nonce[authorizer]`; Midnight `isAuthorized` |
| Value flow | none |
| Reentrancy guard | no |

### `EcrecoverRatifier.cancelRoot()`

| Aspect | Detail |
|--------|--------|
| Visibility | external |
| Caller | Maker or authorized delegate |
| Parameters | maker/root (user-controlled) |
| Call chain | `→ isRootCanceled write` |
| State modified | `isRootCanceled[maker][root]` |
| Value flow | none |
| Reentrancy guard | no |

### `SetterRatifier.setIsRootRatified()`

| Aspect | Detail |
|--------|--------|
| Visibility | public |
| Caller | Maker or authorized delegate |
| Parameters | maker/root/newIsRootRatified (user-controlled) |
| Call chain | `→ isRootRatified write` |
| State modified | `isRootRatified[maker][root]` |
| Value flow | none |
| Reentrancy guard | no |

---

## Role-Gated

### `roleSetter`

| Contract | Function | Parameters | State Modified |
|----------|----------|------------|----------------|
| Midnight | `setRoleSetter()` | newRoleSetter | `roleSetter` |
| Midnight | `setFeeSetter()` | newFeeSetter | `feeSetter` |
| Midnight | `setFeeClaimer()` | newFeeClaimer | `feeClaimer` |
| Midnight | `setTickSpacingSetter()` | newTickSpacingSetter | `tickSpacingSetter` |

### `feeSetter`

| Contract | Function | Parameters | State Modified |
|----------|----------|------------|----------------|
| Midnight | `setMarketSettlementFee()` | id, index, newSettlementFee | market settlement fee breakpoint |
| Midnight | `setDefaultSettlementFee()` | loanToken, index, newSettlementFee | default settlement fee breakpoint |
| Midnight | `setMarketContinuousFee()` | id, newContinuousFee | market continuous fee |
| Midnight | `setDefaultContinuousFee()` | loanToken, newContinuousFee | default continuous fee |

### `feeClaimer`

| Contract | Function | Parameters | State Modified |
|----------|----------|------------|----------------|
| Midnight | `claimSettlementFee()` | token, amount, receiver | `claimableSettlementFee[token]` |
| Midnight | `claimContinuousFee()` | market, amount, receiver | `continuousFeeCredit`, `totalUnits`, `withdrawable` |

### `tickSpacingSetter`

| Contract | Function | Parameters | State Modified |
|----------|----------|------------|----------------|
| Midnight | `setMarketTickSpacing()` | id, newTickSpacing | `marketState[id].tickSpacing` |

---

## Admin-Only

There is no separate `owner` or upgrade admin surface; top-level administration is split across the four mutable role addresses above.

