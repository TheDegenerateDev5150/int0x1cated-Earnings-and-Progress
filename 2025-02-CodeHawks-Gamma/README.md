# Leaderboard
[Gamma Results](https://codehawks.cyfrin.io/c/2025-02-gamma)<br>

`Rank 6 / 108`<br>
![Result](image.png)

# Audited Code Repo
### [CodeHawks: Gamma](https://codehawks.cyfrin.io/c/2025-02-gamma)
### [Github: Gamma](https://github.com/CodeHawks-Contests/2025-02-gamma)

<br>

# <a id="summaryTable"></a>Bugs Filed & Their Status

| #      | Bug ID          | Name | URL    | Adjudged Status  |
|--------|-----------------|------|:------:|-----------------:|
| 1      | [H-01](#h-01)   | Depositor can mint shares without paying execution fee or bearing negative price impact when positionIsClosed is True | [491](https://codehawks.cyfrin.io/c/2025-02-gamma/s/491) | Rejected |
| 2      | [H-02](#h-02)   | User may withdraw more than expected if ADL event happens | [571](https://codehawks.cyfrin.io/c/2025-02-gamma/s/571) | Med; 4 |
| 3      | [H-03](#h-03)   | Entire negative PnL value deducted from withdrawer's collateralDeltaAmount instead of a proportional value | [702](https://codehawks.cyfrin.io/c/2025-02-gamma/s/702) | High; 10 |
| 4      | [H-04](#h-04)   | Donation can be used to steal deposits from others and issue reduced shares | [574](https://codehawks.cyfrin.io/c/2025-02-gamma/s/574) | Rejected |
| 5      | [M-01](#m-01)   | Unused fee never returned to user | [529](https://codehawks.cyfrin.io/c/2025-02-gamma/s/529) | Rejected |
| 6      | [M-02](#m-02)   | getExecutionGasLimit() reports a lower gas limit due to gasPerSwap miscalculation | [543](https://codehawks.cyfrin.io/c/2025-02-gamma/s/543) | Med; 5 |
| 7      | [M-03](#m-03)   | usedFee is overestimated and actual refund received from GMX is not used by Gamma to calculate refund in `_handleReturn()` | [564](https://codehawks.cyfrin.io/c/2025-02-gamma/s/564) | Rejected |
| 8      | [M-04](#m-04)   | Keeper can not call cancelFlow() if depositor gets blacklisted by USDC | [610](https://codehawks.cyfrin.io/c/2025-02-gamma/s/610) | Low |
| 9      | [M-05](#m-05)   | Incorrect share accounting as totalAmountBefore is miscalculated inside mint() | [496](https://codehawks.cyfrin.io/c/2025-02-gamma/s/496) | Rejected |
|10      | [L-01](#L-01)   | feeAmount not rounded in protocol's favour during withdrawal | [617](https://codehawks.cyfrin.io/c/2025-02-gamma/s/617) | Low |
|11      | [L-02](#L-02)   | PnL not rounded in protocol's favour during withdrawal | [622](https://codehawks.cyfrin.io/c/2025-02-gamma/s/622) | Low |

<br>
<br>

## **HIGH-SEVERITY BUGS**
---

### <a id="h-01"></a>[H-01]
## **Depositor can mint shares without paying execution fee or bearing negative price impact when positionIsClosed is True**
#### https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/PerpetualVault.sol#L233-L236
<br>

## Summary
A vault with no open positions & no ongoing flows can be exploited by a depositor since their shares are minted immediately and are not subjected to price impact calculations or are asked to pay any execution fees. This benefits both:
1. The first depositor of the vault.
2. If Keeper ever decides to momentarily close all open positions due to volatile market conditions, any user who deposits at this point of time.

Note that `positionIsClosed = true` simply means Gamma does not currently have a open position in the target GMX market. The GMX market liquidity exists independently and hence any position opened up by Gamma, even if it is the first from the PerpetualVault's perspective is capable of impacting GMX's existing market's prices and hence is subject to price impact calculations by GMX.

## Description
[deposit() mints tokens to the user](https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/PerpetualVault.sol#L233-L236) via `_mint()` whenever `positionIsClosed = true`:
```solidity
  File: contracts/PerpetualVault.sol

   215:            function deposit(uint256 amount) external nonReentrant payable {
   216:              _noneFlow();
   217:              if (depositPaused == true) {
   218:                revert Error.Paused();
   219:              }
   220:              if (amount < minDepositAmount) {
   221:                revert Error.InsufficientAmount();
   222:              }
   223:              if (totalDepositAmount + amount > maxDepositAmount) {
   224:                revert Error.ExceedMaxDepositCap();
   225:              }
   226:              flow = FLOW.DEPOSIT;
   227:              collateralToken.safeTransferFrom(msg.sender, address(this), amount);
   228:              counter++;
   229:              depositInfo[counter] = DepositInfo(amount, 0, msg.sender, 0, block.timestamp, address(0));
   230:              totalDepositAmount += amount;
   231:              EnumerableSet.add(userDeposits[msg.sender], counter);
   232:          
   233:@--->         if (positionIsClosed) {
   234:                MarketPrices memory prices;
   235:                _mint(counter, amount, false, prices);
   236:                _finalize(hex'');
   237:              } else {
   238:                _payExecutionFee(counter, true);
   239:                // mint share token in the NextAction to involve off-chain price data and improve security
   240:                nextAction.selector = NextActionSelector.INCREASE_ACTION;
   241:                nextAction.data = abi.encode(beenLong);
   242:              }
   243:            }
```

Note that:
- `positionIsClosed` is `true` for the first depositor to the vault. 
- Also, if Keeper ever decides to momentarily close all open positions due to volatile market conditions, OR there is some time gap between closing of an existing position & opening of a new one, then `positionIsClosed` is `true` for this duration, with no ongoing flows.
- Any deposits made in such conditions can - 
  - escape paying the execution fee.
  - escape being subjected to price impact calculations which happens only in the case of `FLOW.DEPOSIT` inside the `afterOrderExecution()` [callback](https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/PerpetualVault.sol#L483):
```solidity
  File: contracts/PerpetualVault.sol

   481:              if (orderResultData.orderType == Order.OrderType.MarketIncrease) {
   482:                curPositionKey = positionKey;
   483:@--->           if (flow == FLOW.DEPOSIT) {
   484:                  uint256 amount = depositInfo[counter].amount;
   485:                  uint256 feeAmount = vaultReader.getPositionFeeUsd(market, orderResultData.sizeDeltaUsd, false) / prices.shortTokenPrice.min;
   486:                  uint256 prevSizeInTokens = flowData;
   487:@--->             int256 priceImpact = vaultReader.getPriceImpactInCollateral(curPositionKey, orderResultData.sizeDeltaUsd, prevSizeInTokens, prices);
   488:                  uint256 increased;
   489:                  if (priceImpact > 0) {
   490:@--->               increased = amount - feeAmount - uint256(priceImpact) - 1;
   491:                  } else {
   492:                    increased = amount - feeAmount + uint256(-priceImpact) - 1;
   493:                  }
   494:@--->             _mint(counter, increased, false, prices);
```

Inside [_mint()](https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/PerpetualVault.sol#L762) it's never checked if execution fee has been paid or not. So let's suppose Keeper decides to create a long position in the next run, they have to fund this execution fee out of their own pockets.
As the contest page specifies for Depositors:
> Must provide sufficient execution fees for operations

## Impact
1. Depositor is able to escape any price impact penalties and receives more than their rightful proportion of shares.
2. Other depositors have to bear a higher burden.
3. User receives shares without paying for execution fee.
4. This could lead to a situation where the Keeper doesn't have enough gas to execute the tx and the queue remains stuck. However this impact _could_ be ignored since the contest page states that:
> Assume that Keeper will always have enough gas to execute transactions.  There is a pay execution fee function, but the assumption should be that there's more than enough gas to cover transaction failures, retries, etc

## Proof Of Concept
This PoC shows that the depositor can pay no execution fees and still get shares minted. Add this inside `test/PerpetualVault.t.sol` and run via `forge test --mt test_Deposit_With_No_Exec_Fee -vv --via-ir --rpc-url arbitrum` to see it pass:
```js
  function test_Deposit_With_No_Exec_Fee() external {
    uint256 amount = 1e10; 
    address alice = makeAddr("alice");

    uint256[] memory userDeposits = PerpetualVault(vault).getUserDeposits(alice);
    uint256 userSharesInit;
    uint256 userSharesFinal;
    if(userDeposits.length > 0) {
        (,userSharesInit,,,,) = PerpetualVault(vault).depositInfo(userDeposits[0]);
    }
    emit log_named_decimal_uint("Alice Shares Initial =", userSharesInit, 18);
    
    // Deposit
    IERC20 collateralToken = PerpetualVault(vault).collateralToken();
    vm.startPrank(alice);
    deal(address(collateralToken), alice, amount);
    collateralToken.approve(vault, amount);
    // No execution fee provided by user
    PerpetualVault(vault).deposit{value: 0}(amount);
    vm.stopPrank();

    userDeposits = PerpetualVault(vault).getUserDeposits(alice);
    if(userDeposits.length > 0) {
        (,userSharesFinal,,,,) = PerpetualVault(vault).depositInfo(userDeposits[0]);
    }
    emit log_named_decimal_uint("Alice Shares Final   =", userSharesFinal, 18);
    assertGt(userSharesFinal, userSharesInit);
  }
```

## Recommendation
The fix could involve multiple changes but at its core:
- The protocol needs to do the price impact calculations even when a user deposits during `positionIsClosed = true` and mint shares to them only after these calculations are taken into account.
- Charge the execution fee even when `positionIsClosed = true`.

[Back to Top](#summaryTable)
---

### <a id="h-02"></a>[H-02]
## **User may withdraw more than expected if ADL event happens**
#### https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/GmxProxy.sol#L248-L258
<br>

## Description
First, let's take a look at the action performed by Gamma when GMX's ADL (Automatic Deleveraging) invokes `GmxProxy.sol`'s [afterOrderExecution() callback](https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/GmxProxy.sol#L248-L258):
```solidity
    } else if (msg.sender == address(adlHandler)) {
      uint256 sizeInUsd = dataStore.getUint(keccak256(abi.encode(positionKey, SIZE_IN_USD)));
      if (eventData.uintItems.items[0].value > 0) {
@-->    IERC20(eventData.addressItems.items[0].value).safeTransfer(perpVault, eventData.uintItems.items[0].value);
      }
      if (eventData.uintItems.items[1].value > 0) {
@-->    IERC20(eventData.addressItems.items[1].value).safeTransfer(perpVault, eventData.uintItems.items[1].value);
      }
@-->  if (sizeInUsd == 0) {
        IPerpetualVault(perpVault).afterLiquidationExecution();
      }
```
Any collateral & index tokens received from GMX are transferred to the PerpetualVault. And if this ADL has not resulted in the entire position size to be de-leveraged i.e. `sizeInUsd > 0`, then `afterLiquidationExecution()` is NOT called.

With that in mind, let's now look at a user's withdrawal flow:
1. User calls `withdraw()`. This sets `flow = FLOW.WITHDRAW`.
2. If there's an open position, calls `_settle()` which creates a GMX order.
3. GMX executes the settle order, triggering `afterOrderExecution()`. This sets `nextAction = NextActionSelector.WITHDRAW_ACTION`.
4. Keeper calls `runNextAction()` which if needed, swaps any indexTokens to collateralTokens.
5. Calls `_withdraw()` which creates a GMX decrease position order.
6. GMX executes decrease order, triggering `afterOrderExecution()`. This sets `nextAction = NextActionSelector.FINALIZE`.
7. Keeper calls `runNextAction()` which calls `_finalize()` which calls `_handleReturn()` to process the withdrawal.

However, `_finalize()` and `_handleReturn()` rely on contract's collateral token balance to determine how much withdrawn amount is to be credited back to the user. If an ADL inadvertently happens between steps 6 and 7, i.e. Keeper's `runNextAction()` gets front-runned coincidentally in the same block (Or Keeper fails to notice the ADL event and calls `runNextAction()` anyway), then user receives more than they should:
```solidity
  function _finalize(bytes memory data) internal {
    if (flow == FLOW.WITHDRAW) {
      (uint256 prevCollateralBalance, bool positionClosed, bool refundFee) = abi.decode(data, (uint256, bool, bool));
      // @audit : collateralToken balance has already increased due to ADL, inflating the `withdrawn` figure
@->   uint256 withdrawn = collateralToken.balanceOf(address(this)) - prevCollateralBalance;
      _handleReturn(withdrawn, positionClosed, refundFee);
    } else {
      delete swapProgressData;
      delete flowData;
      delete flow;
    }
  }

  
  function _handleReturn(uint256 withdrawn, bool positionClosed, bool refundFee) internal {
    (uint256 depositId) = flowData;
    uint256 shares = depositInfo[depositId].shares;
    uint256 amount;
    if (positionClosed) {
      amount = collateralToken.balanceOf(address(this)) * shares / totalShares;
    } else {
      // @audit-info : this is correctly calculated since both collateralTokenBalance and `withdrawn` are inflated by an equal degree  
@->   uint256 balanceBeforeWithdrawal = collateralToken.balanceOf(address(this)) - withdrawn; 
      // @audit : but this results in inflated `amount` due to inflated `withdrawn`
@->   amount = withdrawn + balanceBeforeWithdrawal * shares / totalShares; 
    }
    if (amount > 0) {
@->   _transferToken(depositId, amount); // @audit : user credited more than their fair share
    }
    // ... Rest of the code
```

## Impact
- User will receive both their proportional share from the decrease order AND the ADL tokens
- This is incorrect since ADL tokens should be distributed across all position holders. Essentially, other position holders take a loss.

## Recommendation
The fix may not be quite that simple. We may need to properly track credited tokens during ADL, perhaps by having ADL tokens flow into a separate accounting bucket.
Another way _could be_ to increment `prevCollateralBalance` inside the `if (msg.sender == address(adlHandler))` branch when `afterOrderExecution()` callback happens.

[Back to Top](#summaryTable)
---

### <a id="h-03"></a>[H-03]
## **Entire negative PnL value deducted from withdrawer's collateralDeltaAmount instead of a proportional value**
#### https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/VaultReader.sol#L186
<br>

## Description
During a withdrawal flow we have a call to `_withdraw()` [which calls](https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/PerpetualVault.sol#L1110) `getPnl()`:
```solidity
  File: contracts/PerpetualVault.sol

    1089:            function _withdraw(uint256 depositId, bytes memory metadata, MarketPrices memory prices) internal {
    1090:              uint256 shares = depositInfo[depositId].shares;
    1091:              if (shares == 0) {
    1092:                revert Error.ZeroValue();
    1093:              }
    1094:              
    1095:              if (positionIsClosed) {
    1096:                _handleReturn(0, true, false);
    1097:              } else if (_isLongOneLeverage(beenLong)) {  // beenLong && leverage == BASIS_POINTS_DIVISOR
    1098:                uint256 swapAmount = IERC20(indexToken).balanceOf(address(this)) * shares / totalShares;
    1099:                nextAction.selector = NextActionSelector.SWAP_ACTION;
    1100:                // abi.encode(swapAmount, swapDirection): if swap direction is true, swap collateralToken to indexToken
    1101:                nextAction.data = abi.encode(swapAmount, false);
    1102:              } else if (curPositionKey == bytes32(0)) {    // vault liquidated
    1103:                _handleReturn(0, true, false);
    1104:              } else {
    1105:                IVaultReader.PositionData memory positionData = vaultReader.getPositionInfo(curPositionKey, prices);
    1106:                uint256 collateralDeltaAmount = positionData.collateralAmount * shares / totalShares;
    1107:                uint256 sizeDeltaInUsd = positionData.sizeInUsd * shares / totalShares;
    1108:                // we always charge the position fee of negative price impact case.
    1109:                uint256 feeAmount = vaultReader.getPositionFeeUsd(market, sizeDeltaInUsd, false) / prices.shortTokenPrice.max;
    1110:@--->           int256 pnl = vaultReader.getPnl(curPositionKey, prices, sizeDeltaInUsd);
    1111:                if (pnl < 0) {
    1112:@--->             collateralDeltaAmount = collateralDeltaAmount - feeAmount - uint256(-pnl) / prices.shortTokenPrice.max;
    1113:                } else {
    1114:                  collateralDeltaAmount = collateralDeltaAmount - feeAmount;
    1115:                }
    1116:                uint256 acceptablePrice = abi.decode(metadata, (uint256));
    1117:                _createDecreasePosition(collateralDeltaAmount, sizeDeltaInUsd, beenLong, acceptablePrice, prices);
    1118:              }
    1119:            }
```
This `pnl` is deducted from `collateralDeltaAmount` on L1112.

If we see [getPnl()](https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/VaultReader.sol#L171-L190):
```solidity
  File: contracts/VaultReader.sol

   171:            function getPnl(
   172:              bytes32 key,
   173:              MarketPrices memory prices,
   174:              uint256 sizeDeltaUsd
   175:            ) external view returns (int256) {
   176:              uint256 sizeInTokens = getPositionSizeInUsd(key);
   177:              if (sizeInTokens == 0) return 0;
   178:              
   179:              PositionInfo memory positionInfo = gmxReader.getPositionInfo(
   180:                address(dataStore),
   181:                referralStorage,
   182:                key,
   183:                prices,
   184:@--->           sizeDeltaUsd,
   185:                address(0),
   186:@--->           true
   187:              );
   188:          
   189:              return positionInfo.pnlAfterPriceImpactUsd;
   190:            }
```
We see that GMX's `gmxReader.getPositionInfo()` is called with last param as `true` which if we see [in GMX's implementation](https://github.com/gmx-io/gmx-synthetics/blob/b8fb11349eb59ae48a1834c239669d4ad63a38b5/contracts/reader/ReaderPositionUtils.sol#L185), corresponds to `usePositionSizeAsSizeDeltaUsd`. We further see that when `usePositionSizeAsSizeDeltaUsd = true`, the passed param `sizeDeltaUsd` [is ignored and the **entire position size** is considered](https://github.com/gmx-io/gmx-synthetics/blob/b8fb11349eb59ae48a1834c239669d4ad63a38b5/contracts/reader/ReaderPositionUtils.sol#L198-L200) instead:
```solidity
  File: contracts/reader/ReaderPositionUtils.sol

   178:              function getPositionInfo(
   179:                  DataStore dataStore,
   180:                  IReferralStorage referralStorage,
   181:                  Position.Props memory position,
   182:                  MarketUtils.MarketPrices memory prices,
   183:                  uint256 sizeDeltaUsd,
   184:                  address uiFeeReceiver,
   185:@--->             bool usePositionSizeAsSizeDeltaUsd
   186:              ) internal view returns (PositionInfo memory) {
   187:                  if (position.account() == address(0)) {
   188:                      revert Errors.EmptyPosition();
   189:                  }
   190:          
   191:                  PositionInfo memory positionInfo;
   192:                  GetPositionInfoCache memory cache;
   193:          
   194:                  positionInfo.position = position;
   195:                  cache.market = MarketStoreUtils.get(dataStore, positionInfo.position.market());
   196:                  cache.collateralTokenPrice = MarketUtils.getCachedTokenPrice(positionInfo.position.collateralToken(), cache.market, prices);
   197:          
   198:@--->             if (usePositionSizeAsSizeDeltaUsd) {
   199:@--->                 sizeDeltaUsd = positionInfo.position.sizeInUsd();
   200:                  }
   201:          
                        // ... Rest of the code
```

Hence, the withdrawer is burdened with the entire negative PnL instead of their proportional amount.

## Impact
- Withdrawer takes a greater than expected loss.
- Additionally, a few lines later inside `_withdraw()` [we have calls](https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/PerpetualVault.sol#L1117) to `_createDecreasePosition()` --> `vaultReader.willPositionCollateralBeInsufficient(..., collateralDeltaAmount)` which will now estimate this incorrectly.

## Mitigation 
`getPnl()` seems to have been called only from within `_withdraw()`, so we can modify the function itself:
```diff
  function getPnl(
    bytes32 key,
    MarketPrices memory prices,
    uint256 sizeDeltaUsd
  ) external view returns (int256) {
    uint256 sizeInTokens = getPositionSizeInUsd(key);
    if (sizeInTokens == 0) return 0;
    
    PositionInfo memory positionInfo = gmxReader.getPositionInfo(
      address(dataStore),
      referralStorage,
      key,
      prices,
      sizeDeltaUsd,
      address(0),
-     true
+     false
    );

    return positionInfo.pnlAfterPriceImpactUsd;
  }
```

[Back to Top](#summaryTable)
---

### <a id="h-04"></a>[H-04]
## **Donation can be used to steal deposits from others and issue reduced shares**
#### https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/PerpetualVault.sol#L771-L774
<br>

## Description
[_mint()](https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/PerpetualVault.sol#L771-L774) relies on current collateralToken and indexToken balances to calculate how many shares need to be issued to the depositor:
```solidity
  File: contracts/PerpetualVault.sol

   762:            function _mint(uint256 depositId, uint256 amount, bool refundFee, MarketPrices memory prices) internal {
   763:              uint256 _shares;
   764:              if (totalShares == 0) {
   765:                _shares = depositInfo[depositId].amount * 1e8;
   766:              } else {
   767:                uint256 totalAmountBefore;
   768:                if (positionIsClosed == false && _isLongOneLeverage(beenLong)) {
   769:                  totalAmountBefore = IERC20(indexToken).balanceOf(address(this)) - amount;
   770:                } else {
   771:@--->             totalAmountBefore = _totalAmount(prices) - amount;
   772:                }
   773:                if (totalAmountBefore == 0) totalAmountBefore = 1;
   774:@--->           _shares = amount * totalShares / totalAmountBefore;
   775:              }
   776:          
   777:              depositInfo[depositId].shares = _shares;
   778:              totalShares = totalShares + _shares;
   779:          
   780:              if (refundFee) {
   781:                uint256 usedFee = callbackGasLimit * tx.gasprice;
   782:                if (depositInfo[counter].executionFee > usedFee) {
   783:                  try IGmxProxy(gmxProxy).refundExecutionFee(depositInfo[counter].owner, depositInfo[counter].executionFee - usedFee) {} catch {}
   784:                }
   785:              }
   786:          
   787:              emit Minted(depositId, depositInfo[depositId].owner, _shares, amount);
   788:            }
```

and 

```solidity
  File: contracts/PerpetualVault.sol

   821:            function _totalAmount(MarketPrices memory prices) internal view returns (uint256) {
   822:              if (positionIsClosed) {
   823:                return collateralToken.balanceOf(address(this));
   824:              } else {
   825:                IVaultReader.PositionData memory positionData = vaultReader.getPositionInfo(curPositionKey, prices);
   826:@--->           uint256 total = IERC20(indexToken).balanceOf(address(this)) * prices.indexTokenPrice.min / prices.shortTokenPrice.min
   827:@--->               + collateralToken.balanceOf(address(this))
   828:                    + positionData.netValue / prices.shortTokenPrice.min;
   829:          
   830:                return total;
   831:              }
   832:            }
```

This can be exploited by an attacker. They can donate some collateral tokens to the vault such that `totalAmountBefore` is inflated and `_shares = amount * totalShares / totalAmountBefore` evaluates to zero (or at least negatively impacted). This donation would happen before the `afterOrderExecution()` callback from GMX gets executed. 

Please refer PoC section for a concrete example. **Note** that it's quite tricky to change the config params of the test suite hence the PoC sticks to the standard deposit amount of `100 USDC` and the configured market prices. This results in the donation amount to come out to be a large figure. However for collateral tokens with 18 decimal precision and different market prices, this goes down significantly. Alternatively, attacker can choose to donate index tokens instead of collateral tokens, which would have the same effect.

Attack Path:
1. Attacker has a position in the vault and owns some shares. 
2. A position is opened by the vault.
3. Alice the naive user deposits some collateral tokens.
4. During the deposit flow, Keeper calls `runNextAction()`.
5. After this, we wait for the GMX callback to `afterOrderExecution()`.
6. Attacker front-runs this callback (or back-runs step 4's `runNextAction()`) with a donation. **Note that** the attacker DOES NOT need to monitor the mempool (which is private) in any manner. Attacker can simply monitor on-chain txs and the moment they see `runNextAction()` is executed, they donate. Since the callback from GMX is likely to have some time lag, this would work.
7. `afterOrderExecution()` gets called now which calls `_mint()`. Due to the donation, zero shares are issued to Alice even though funds have been deposited.
8. Attacker can now withdraw their shares. Since withdrawal too depends on current balance of the vault, Alice's funds are siphoned off to the attacker.

## Impact
Depositor's funds can be stolen by an attacker. Depositor receives less shares than expected.

## Proof of Concept
Add these two tests inside `test/PerpetualVault.t.sol` and run via `forge test --mt test_donationBug_Pass -vv --via-ir --rpc-url arbitrum`. The first test, `test_donationBug_Pass_1_someSharesMinted()` shows the normal path where some shares are minted to the depositor. The second test, `test_donationBug_Pass_2_zeroSharesMinted()` shows how no shares are minted after an attacker's donation:
```js
  function test_donationBug_Pass_1_someSharesMinted() external {
    address keeper = PerpetualVault(vault).keeper();
    address alice = makeAddr("alice");
    vm.deal(alice, 0.5 ether); // some ETH for gas fees

    // ############################
    console.log("\n=== Deposit1 === \n");
    depositFixture(alice, 1e8);
    
    MarketPrices memory prices = mockData.getMarketPrices();
    bytes[] memory data = new bytes[](1);
    data[0] = abi.encode(3380000000000000);
    vm.prank(keeper);
    PerpetualVault(vault).run(true, false, prices, data);
    GmxOrderExecuted(true);
    vm.prank(keeper);
    PerpetualVault(vault).runNextAction(prices, data);
    // ###############################

    console.log("\n=== Deposit2 === \n");
    depositFixture(alice, 1e8);
    vm.prank(keeper);
    PerpetualVault(vault).runNextAction(prices, data);

    console.log("\nGMX callback starts");
    GmxOrderExecuted(true);
    console.log("GMX callback ends");
    uint256[] memory aliceDeposits = PerpetualVault(vault).getUserDeposits(alice);
    (,uint256 sharesIssued,,,,) = PerpetualVault(vault).depositInfo(aliceDeposits[1]);
    assertGt(sharesIssued, 0); 
  }

  function test_donationBug_Pass_2_zeroSharesMinted() external {
    address keeper = PerpetualVault(vault).keeper();
    address alice = makeAddr("alice");
    vm.deal(alice, 0.5 ether); // some ETH for gas fees

    // ############################
    console.log("\n=== Deposit1 === \n");
    depositFixture(alice, 1e8);
    
    MarketPrices memory prices = mockData.getMarketPrices();
    bytes[] memory data = new bytes[](1);
    data[0] = abi.encode(3380000000000000);
    vm.prank(keeper);
    PerpetualVault(vault).run(true, false, prices, data);
    GmxOrderExecuted(true);
    vm.prank(keeper);
    PerpetualVault(vault).runNextAction(prices, data);
    // ###############################

    console.log("\n=== Deposit2 === \n");
    depositFixture(alice, 1e8);
    vm.prank(keeper);
    PerpetualVault(vault).runNextAction(prices, data);

    // $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
    // Attacker donates some collateral tokens by front-running GMX callback
    console.log("\nAttacker Donates\n");
    address attacker = makeAddr("attacker");
    uint256 donationAmount = 1000582829999999900258284;
    IERC20 collateralToken = PerpetualVault(vault).collateralToken();
    deal(address(collateralToken), attacker, donationAmount);
    vm.prank(attacker);
    IERC20(collateralToken).transfer(vault, donationAmount);
    // $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$

    console.log("\nGMX callback starts");
    GmxOrderExecuted(true);
    console.log("GMX callback ends");
    uint256[] memory aliceDeposits = PerpetualVault(vault).getUserDeposits(alice);
    (,uint256 sharesIssued,,,,) = PerpetualVault(vault).depositInfo(aliceDeposits[1]);
    assertEq(sharesIssued, 0); // @audit-issue : no shares issued but funds taken from Alice
  }
```

## Recommendation 
- It would be better to track balances internally so that they can't be manipulated by donations.
- Some initial "dead" shares can be minted to the protocol to make this attack unprofitable.
- Set a `minShares` param which must be issued during deposits.

[Back to Top](#summaryTable)

<br>

## **MEDIUM-SEVERITY BUGS**
---

### <a id="m-01"></a>[M-01]
## **Unused fee never returned to user**
#### https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/PerpetualVault.sol#L271-L279
<br>

## Description & Impact
Inside [withdraw()](https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/PerpetualVault.sol#L271-L279) Gamma collects execution fee from the user:
```solidity
            depositInfo[depositId].recipient = recipient;
@---->      _payExecutionFee(depositId, false);
            if (curPositionKey != bytes32(0)) {
                nextAction.selector = NextActionSelector.WITHDRAW_ACTION;
                _settle();  // Settles any outstanding fees and updates state before processing withdrawal
            } else {
                MarketPrices memory prices;
                _withdraw(depositId, hex'', prices);
            }
```

- But this fee is never used if `curPositionKey != bytes32(0)` is `false` i.e. there's no open position. The control moves to the `else` clause where `_withdraw()` is called which [internally calls](https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/PerpetualVault.sol#L1096) `_handleReturn(0, true, false)`. the presence of `false` as the third param ensures no refund is issued.

- [Similar behaviour](https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/PerpetualVault.sol#L238) is observed in `deposit()`. Even if all the fee provided by user is not spent on GMX and the unspent amount is refunded to Gamma, it never makes it way back to the user.

## Mitigation 
- Do not ask the user to pay execution fee if `curPositionKey == bytes32(0)`
- Incorporate a mechanism to refund unspent GMX fees back to the user at the time of `deposit()`

[Back to Top](#summaryTable)
---

### <a id="m-02"></a>[M-02]
## **getExecutionGasLimit() reports a lower gas limit due to gasPerSwap miscalculation**
#### https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/GmxProxy.sol#L169
<br>

## Description
When a user calls `deposit()` or `withdraw()`, these functions internally call: `_payExecutionFee --> PerpetualVault::getExecutionGasLimit() --> GmxProxy::getExecutionGasLimit()`. The function [getExecutionGasLimit()](https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/GmxProxy.sol#L169) in `GmxProxy.sol` fetches `gasPerSwap` correctly as:
```solidity
    uint256 gasPerSwap = dataStore.getUint(SINGLE_SWAP_GAS_LIMIT);
```
but then assumes the swapPath length to be `1` & never bothers to multiply it with the correct swap hops.

Let's have a look at the GMX implementation. We can see [here](https://github.com/gmx-io/gmx-synthetics/blob/b8fb11349eb59ae48a1834c239669d4ad63a38b5/contracts/data/Keys.sol#L603-L605) that `SINGLE_SWAP_GAS_LIMIT` key is retuned by the `singleSwapGasLimitKey()` function. 
```solidity
    // @dev key for single swap gas limit
    // @return key for single swap gas limit
    function singleSwapGasLimitKey() internal pure returns (bytes32) {
        return SINGLE_SWAP_GAS_LIMIT;
    }
```

We can also see that a function like `estimateExecuteDecreaseOrderGasLimit()` (among many others) calculates the gas limit in the [following manner](https://github.com/gmx-io/gmx-synthetics/blob/b8fb11349eb59ae48a1834c239669d4ad63a38b5/contracts/gas/GasUtils.sol#L325-L333):
```solidity
    // @dev the estimated gas limit for decrease orders
    // @param dataStore DataStore
    // @param order the order to estimate the gas limit for
    function estimateExecuteDecreaseOrderGasLimit(DataStore dataStore, Order.Props memory order) internal view returns (uint256) {
        uint256 gasPerSwap = dataStore.getUint(Keys.singleSwapGasLimitKey());
        uint256 swapCount = order.swapPath().length;
        if (order.decreasePositionSwapType() != Order.DecreasePositionSwapType.NoSwap) {
            swapCount += 1;
        }

@--->   return dataStore.getUint(Keys.decreaseOrderGasLimitKey()) + gasPerSwap * swapCount + order.callbackGasLimit();
    }
```
Notice the `gasPerSwap * swapCount` term in the `return` statement. The Gamma implementation misses this or assumes that tokens like WBTC or LINK on both Arbitrum and Avalanche chains will be swappable with WETH in one hop, which is not necessarily true and GMX may use an optimized swap path with more than one hops.

## Impact
User may end up paying less than required execution fee and the Keepers end up paying additional amount from their own pockets.

As the contest page specifies for Depositors:
> Must provide sufficient execution fees for operations

## Recommendation 
Either fetch the swapPath hops from GMX and multiply that to `gasPerSwap` OR increase the buffer on top of the calculated gas limit to stay in the safe zone.

[Back to Top](#summaryTable)
---

### <a id="m-03"></a>[M-03]
## **usedFee is overestimated and actual refund received from GMX is not used by Gamma to calculate refund in `_handleReturn()`**
#### https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/PerpetualVault.sol#L1146
<br>

## Description
`_handleReturn()` estimates the `usedFee` by [using the max possible gas limit](https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/PerpetualVault.sol#L1146):
```solidity
    if (refundFee) {
@-->  uint256 usedFee = callbackGasLimit * tx.gasprice;
      if (depositInfo[depositId].executionFee > usedFee) {
        try IGmxProxy(gmxProxy).refundExecutionFee(depositInfo[counter].owner, depositInfo[counter].executionFee - usedFee) {} catch {}
      }
    }
```

This is not necessarily the fee amount which GMX refunds back to Gamma via the [refundExecutionFee() callback](https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/GmxProxy.sol#L327). Gamma can readily calculate the actual amount refunded by GMX and use that to refund the user but instead it refunds only the amount in excess of max possible `callbackGasLimit` via `depositInfo[counter].executionFee - usedFee`. 
This will almost always lead to no refunds for the user while Gamma itself benefits from the refunds.

## Impact
User fee refunds not based on actual usage and in majority of txs, users will be given back nothing while Gamma itself keeps on receiving refunds from GMX.

## Recommendation 
Introduce accounting for the refund received from GMX and use that to base the user's refund calculation on inside `_handleReturn()`.

[Back to Top](#summaryTable)
---

### <a id="m-04"></a>[M-04]
## **Keeper can not call cancelFlow() if depositor gets blacklisted by USDC**
#### https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/PerpetualVault.sol#L1223
<br>

## Description
The execution path is [cancelFlow()](https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/PerpetualVault.sol#L421) --> [_cancelFlow()](https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/PerpetualVault.sol#L1223) which attempts to transfer collateral token to the depositor:
```solidity
  function _cancelFlow() internal {
    if (flow == FLOW.DEPOSIT) {
      uint256 depositId = counter;
@-->  collateralToken.safeTransfer(depositInfo[depositId].owner, depositInfo[depositId].amount);
    // ... Rest of the code
```

If the depositor has gotten blacklisted by USDC at this point of time, then `cancelFlow()` will revert. There would be no way for the Keeper to now remove this flow and delete the `depositInfo[depositId]`.

## Impact
As the [natspec mentions](https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/PerpetualVault.sol#L415), the need for `cancelFlow()` is because:
```solidity
    /**
     * @notice
     *  Cancel the current ongoing flow.
     * @dev
     *  In the case of 1x long leverage, we never cancel the ongoing flow.
@->  *  In the case of gmx position, we could cancel current ongoing 
@->  *   flow due to some accidents from our side or gmx side.
     */
    function cancelFlow() external nonReentrant gmxLock {
```

As a result of this vulnerability, there would now be no way to cancel the current ongoing flow due to any accidents from Gamma's or GMX's side.

## Mitigation 
Wrap the `collateralToken.safeTransfer()` call in a `try-catch` block.

[Back to Top](#summaryTable)
---

### <a id="m-05"></a>[M-05]
## **Incorrect share accounting as totalAmountBefore is miscalculated inside mint()**
#### https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/PerpetualVault.sol#L768-L769
<br>

## Description & Impact
When there's a 1x Long open position, [`totalAmountBefore` is calculated inside `_mint()`](https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/PerpetualVault.sol#L768-L769) in the following manner:
```solidity
  File: contracts/PerpetualVault.sol

   762:            function _mint(uint256 depositId, uint256 amount, bool refundFee, MarketPrices memory prices) internal {
   763:              uint256 _shares;
   764:              if (totalShares == 0) {
   765:                _shares = depositInfo[depositId].amount * 1e8;
   766:              } else {
   767:                uint256 totalAmountBefore;
   768:@--->           if (positionIsClosed == false && _isLongOneLeverage(beenLong)) {
   769:@--->             totalAmountBefore = IERC20(indexToken).balanceOf(address(this)) - amount;
   770:                } else {
   771:                  totalAmountBefore = _totalAmount(prices) - amount;
   772:                }
   773:                if (totalAmountBefore == 0) totalAmountBefore = 1;
   774:@--->           _shares = amount * totalShares / totalAmountBefore;
   775:              }
                       // ... Rest of the code
```

However the vault could also be holding some collateral tokens at the time which should contribute to this calculation. These collateral tokens could have arrived due to:
- Fees
- ADL (Auto deleveraging) by GMX
- Partial Liquidations

These need to be accounted for, else the depositor receives a greater share ratio than intended.

## Recommendation
```diff
    if (positionIsClosed == false && _isLongOneLeverage(beenLong)) {
-        totalAmountBefore = IERC20(indexToken).balanceOf(address(this)) - amount;
+        totalAmountBefore = _totalAmount(prices) * prices.shortTokenPrice.min / prices.indexTokenPrice.min - amount;
    }
```

[Back to Top](#summaryTable)

<br>
<br>

## **LOW-SEVERITY BUGS**
---

### <a id="l-01"></a>[L-01]
## **feeAmount not rounded in protocol's favour during withdrawal**
#### https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/PerpetualVault.sol#L1109
<br>

## Description
[feeAmount](https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/PerpetualVault.sol#L1109) calculation needs to be in favour of the protocol. The correct code ought to be:
```diff
-   uint256 feeAmount = vaultReader.getPositionFeeUsd(market, sizeDeltaInUsd, false) / prices.shortTokenPrice.max;
+   uint256 feeAmount = vaultReader.getPositionFeeUsd(market, sizeDeltaInUsd, false) / prices.shortTokenPrice.min;
```

Division should occur by shortToken's `min` price, not the `max` price.

## Impact
Protocol receives less fees and user is able to withdraw more.

[Back to Top](#summaryTable)
---

### <a id="l-02"></a>[L-02]
## **PnL not rounded in protocol's favour during withdrawal**
#### https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/PerpetualVault.sol#L1112
<br>

## Description
[pnl](https://github.com/CodeHawks-Contests/2025-02-gamma/blob/main/contracts/PerpetualVault.sol#L1112) calculation needs to be in favour of the protocol. The correct code ought to be:
```diff
-   collateralDeltaAmount = collateralDeltaAmount - feeAmount - uint256(-pnl) / prices.shortTokenPrice.max;
+   collateralDeltaAmount = collateralDeltaAmount - feeAmount - uint256(-pnl) / prices.shortTokenPrice.min;
```

Division should occur by shortToken's `min` price, not the `max` price.

## Impact
User is able to withdraw more and protocol receives lesser negative PnL position fee.

[Back to Top](#summaryTable)

