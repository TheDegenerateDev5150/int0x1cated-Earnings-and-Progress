# Leaderboard
[Euler Finance Results](https://cantina.xyz/competitions/41306bb9-2bb8-4da6-95c3-66b85e11639f/leaderboard)<br>

`Rank 36 / 610` <br>
![ResultCard](image.png)

# Audited Code Repo
### [Cantina: Euler Finance](https://github.com/euler-xyz/)

<br>

# <a id="summaryTable"></a>Bugs Filed & Their Status

| # | Bug ID | Name | URL | Adjudged Status |
|--------|--------|------|:------:|-----------------:|
| 1      | [H-01](#h-01)  | Liquidator receives higher profit and more than expected debt gets socialized due to incorrect reverse-Dutch Auction formula | [218](https://cantina.xyz/code/41306bb9-2bb8-4da6-95c3-66b85e11639f/findings/218) |  Rejected  |
| 2      | [H-02](#h-02)  | Lack of slippage protection & deadline in pullDebt() can lead to loss for caller | [221](https://cantina.xyz/code/41306bb9-2bb8-4da6-95c3-66b85e11639f/findings/221) |  Rejected  |
| 3      | [H-03](#h-03)  | In spite of correctly specifying minYieldBalance, incomplete slippage protection & missing deadline in liquidate() lead to loss | [220](https://cantina.xyz/code/41306bb9-2bb8-4da6-95c3-66b85e11639f/findings/220) | Accepted as Low  |
| 4      | [H-04](#h-04)  | Missing slippage protection & deadline in borrow() can lead to loss for the borrower | [219](https://cantina.xyz/code/41306bb9-2bb8-4da6-95c3-66b85e11639f/findings/219) | Rejected  |
| 5      | [M-01](#m-01)  | Excess amountIn is never refunded when using swapToUnderlyingGivenIn() or swapToSynthGivenIn() | [5](https://cantina.xyz/code/41306bb9-2bb8-4da6-95c3-66b85e11639f/findings/5) | Accepted as Low  |
| 6      | [M-02](#m-02)  | Swaps can be split into smaller pieces across multiple transactions in order to pay less fees | [6](https://cantina.xyz/code/41306bb9-2bb8-4da6-95c3-66b85e11639f/findings/6) |  Rejected  |
| 7      | [M-03](#m-03)  | Incorrect smearing calculation when maxGulp is reached | [1](https://cantina.xyz/code/41306bb9-2bb8-4da6-95c3-66b85e11639f/findings/1) | Rejected  |
| 8      | [M-04](#m-04)  | maxRepayValue is rounded-down in favour of the liquidator and against the protocol | [222](https://cantina.xyz/code/41306bb9-2bb8-4da6-95c3-66b85e11639f/findings/222) | Rejected  |
| 9      | [M-05](#m-05)  | `liqCache.repay` is rounded-down against the protocol and in favour of the liquidator | [223](https://cantina.xyz/code/41306bb9-2bb8-4da6-95c3-66b85e11639f/findings/223) |  Rejected  |
| 10     | [M-06](#m-06)  | resolveOracle() can run out of gas in case of multi-nested vaults and cause DoS | [268](https://cantina.xyz/code/41306bb9-2bb8-4da6-95c3-66b85e11639f/findings/268) |  Rejected  |

<br>

## **HIGH-SEVERITY BUGS**
---

### <a id="h-01"></a>[H-01]
## **Liquidator receives higher profit and more than expected debt gets socialized due to incorrect reverse-Dutch Auction formula**
#### https://github.com/euler-xyz/euler-vault-kit/blob/cantina-contest/src/EVault/modules/Liquidation.sol#L164
<br>

## Summary
In the following [code snippet](https://github.com/euler-xyz/euler-vault-kit/blob/cantina-contest/src/EVault/modules/Liquidation.sol#L164) `maxRepayValue` shouldn't be scaled down directly like it is being now. A different formula ought to be used which is consistent with the Dutch Auction system:
```js
  File: src/EVault/modules/Liquidation.sol

   160:@--->             // Limit yield to borrower's available collateral, and reduce repay if necessary. This can happen when borrower
   161:@--->             // has multiple collaterals and seizing all of this one won't bring the violator back to solvency
   162:
   163:                  if (collateralValue < maxYieldValue) {
   164:@--->                 maxRepayValue = collateralValue * discountFactor / 1e18;
   165:                      maxYieldValue = collateralValue;
   166:                  }
```

Due the above error the liquidator is transferred less than expected borrower's debt allowing the liquidator to book higher profits. Also, the incorrect remaining debt (higher than expected) is socialized among other depositors acting as a loss to them.

## Description
Let's understand with the help of an example. This is later reproduced with the same numbers as a coded PoC under section _Proof of Concept-1_. We will make a few config assumptions for ease of calculation:
- Assume:
    - Vault's LTV to be `0.5e4` i.e. `50%`
    - Vault's `maxLiquidationDiscount` to be `0.4e4` i.e. `40%`. 
    - Bob to be the borrower and Alice to be the liquidator.

- Bob deposits `10` tokens of collateral priced at `1`. Total collateral value = `10 * 1 = 10`.

- Bob borrows `4`, which is now his debt. Since LTV is `50%`, he is allowed to borrow any amount less than `5`.

- Collateral price drops to `0.5`. New `collateralValue = 10 * 0.5 = 5`. New `collateralAdjustedValue = 50% of collateralValue = 2.5` which less than the debt of `4`. Bob can be liquidated now.

- Alice triggers Bob's liquidation for the entire amount. 

- This results in the calculaiton of `discountFactor = collateralAdjustedValue / liabilityValue = 2.5 / 4 = 0.625`. Lower the `discountFactor`, higher the liquidation bonus offered to the liquidator.

- The entire debt of `4` becomes the `maxRepayValue`.

- Next, `maxYieldValue` is calculated by the protocol as `maxRepayValue / discountFactor = 4 / 0.625 = 6.4` on [L130](https://github.com/euler-xyz/euler-vault-kit/blob/cantina-contest/src/EVault/modules/Liquidation.sol#L130) and [L158](https://github.com/euler-xyz/euler-vault-kit/blob/cantina-contest/src/EVault/modules/Liquidation.sol#L158).

- Since there isn't enough `collateralValue` to meet the `maxYieldValue`, it is lowered. Additionally, the figure of `maxRepayValue` is scaled down too on [L164](https://github.com/euler-xyz/euler-vault-kit/blob/cantina-contest/src/EVault/modules/Liquidation.sol#L164) and it becomes `maxRepayValue = 5 * 0.625 = 3.125`:
```js
  File: src/EVault/modules/Liquidation.sol

   160:                  // Limit yield to borrower's available collateral, and reduce repay if necessary. This can happen when borrower
   161:                  // has multiple collaterals and seizing all of this one won't bring the violator back to solvency
   162:
   163:                  if (collateralValue < maxYieldValue) {
   164:@--->                 maxRepayValue = collateralValue * discountFactor / 1e18;
   165:                      maxYieldValue = collateralValue;
   166:                  }
```

- Alice will now be transferred collateral worth `5` for a debt worth `3.125`. This means that:
    - The borrower can use his alternate account to act as a liquidator now in the following manner: 
        - Bob's current position is `= liabilityValue - collateralAdjustedValue = 4 - 5 = -1` i.e. a loss of `1`. 
        - He himself acts as the liquidator. The transaction nets him `= collateralAdjustedValue - liabilityValue = 5 - 3.125 = 1.875`. 
        - Bob just made a profit of `1.875 - 1 = 0.875` and a portion of his debt got socialized.

<br>

- **_To conclude the analysis, another way to look at it is_**:
    - We are rewarding Alice the liquidator with a `discountFactor` calculated at the time she "recognized" the bad debt. Even though Alice is eventually asked to take up a smaller portion of the bad debt, which effectively increases her discount factor to an `execution_discountFactor` value ( `= 2.5 / 3.125 = 0.8` ), the protocol still goes ahead and rewards her based on her lower `recognition_discountFactor` value ( `= 2.5 / 4 = 0.625` ) **_on the debt value of `3.125` ( i.e. 5 = 3.125 / 0.625 )_** thus causing a greater amount of remaining debt to be socialized & boosting her profits. _(Lower the `discountFactor`, higher the liquidation bonus offered to the liquidator)._
    
    - Please also note that simply using `discountFactor` to "reverse-calculate" the `maxRepayValue` may not be the right way to do this. Care needs to be taken than the protocol does not offer a liquidation profit greater than the loss the borrower is currently at and hence additional conditions may need to be added to existing logic.
    
    - However, **_even if we go ahead & follow the protocol's line of reasoning, the correct debt value or `maxRepayValue` should have been `3.5355` (approx) and NOT `3.125`_**. We can easily check this:
        - `discountFactor` if debt value were `3.5355` would be `2.5 / 3.5355 = 0.7071`
        - Thus `collateralValue` to be transferred to Alice = `3.5355 / 0.7071 = 5` i.e. the entire collateral.
        - This is akin to the protocol saying the following to the liquidator and _can_ be considered as an acceptable line of thinking (albeit not optimal in my personal view) - 
        > "Sorry mate, we don't have enought collateral to pay you for taking up the entire debt of `4`, but our reverse Dutch auction system does allow us to pay you the entire collatral of worth `5` if you take up `3.5355` worth of debt. We'll not worry about the remaining `1.4645` of debt and will handle it interally by socializing it."

<br>
Whichever way you look at it, Alice is currently being given a higher-than-deserved profit and the depositors paying the cost for it.
<br>
<br>

**An example of the borrower profiting from "self-liquidation" with less extreme numbers (reproduced later with code under section _Proof of Concept-2_):**

- Setup: Max allowed LTV = `0.8e14`; Max Liquidation Discount = `0.2e4`; Collateral Value Deposited = `10`; Borrowed Value = `7.99`. Bob's virtual loss right now = `10 - 7.99 = 2.01`.
- Borrowed asset price rises and hence borrowed value becomes = `9`. LTV is breached, so Bob can be liquidated.
- Due to scaling-down calculations, protocol calculates `maxRepayValue = 8.889 (approx)` and `maxYieldValue = 10`.
- Bob uses his alternate account to act as the liquidator and effectively receives a profit of `maxYieldValue - maxRepayValue = 10 - 8.889 = 1.111`.
- Bob still has the initial borrow which is now worth `9` due to the price increase. His final profit = `1.111 + 9 - initialCollateralValueDeposited = 1.111 + 9 - 10 = 0.111`.

</details>
<br>


## Impact
1. Liquidator profits more than expected.
2. The above "extra" profit comes at the cost of other depositors who have to incorrectly bear the cost of the socialized debt (or witness a higher-than-expected devaluation of the token).

## Proof of Concept-1

<details><summary>Click to view patch</summary>

```diff
diff --git a/src/EVault/modules/Liquidation.sol b/src/EVault/modules/Liquidation.sol
index 07e5bb3..cd1f05f 100644
--- a/src/EVault/modules/Liquidation.sol
+++ b/src/EVault/modules/Liquidation.sol
@@ -8,7 +8,7 @@ import {BalanceUtils} from "../shared/BalanceUtils.sol";
 import {LiquidityUtils} from "../shared/LiquidityUtils.sol";
 
 import "../shared/types/Types.sol";
-
+import {console} from "forge-std/Test.sol";
 /// @title LiquidationModule
 /// @custom:security-contact security@euler.xyz
 /// @author Euler Labs (https://www.eulerlabs.com/)
@@ -214,6 +214,7 @@ abstract contract LiquidationModule is ILiquidation, BalanceUtils, LiquidityUtil
         ) {
             Assets owedRemaining = liqCache.liability.subUnchecked(liqCache.repay);
             decreaseBorrow(vaultCache, liqCache.violator, owedRemaining);
+            console.log("Socialized debt                    =: %s.%s", owedRemaining.toUint()/1e18, owedRemaining.toUint() - owedRemaining.toUint()/1e18 * 1e18); // express in decimal notation
 
             // decreaseBorrow emits Repay without any assets entering the vault. Emit Withdraw from and to zero address
             // to cover the missing amount for offchain trackers.
diff --git a/test/unit/evault/EVaultTestBase.t.sol b/test/unit/evault/EVaultTestBase.t.sol
index ed537c2..3a20539 100644
--- a/test/unit/evault/EVaultTestBase.t.sol
+++ b/test/unit/evault/EVaultTestBase.t.sol
@@ -134,7 +134,7 @@ contract EVaultTestBase is AssertionsCustomTypes, Test, DeployPermit2 {
             factory.createProxy(address(0), true, abi.encodePacked(address(assetTST), address(oracle), unitOfAccount))
         );
         eTST.setInterestRateModel(address(new IRMTestDefault()));
-        eTST.setMaxLiquidationDiscount(0.2e4);
+        eTST.setMaxLiquidationDiscount(0.4e4);
         eTST.setFeeReceiver(feeReceiver);
 
         eTST2 = IEVault(
diff --git a/test/unit/evault/modules/Liquidation/basic.t.sol b/test/unit/evault/modules/Liquidation/basic.t.sol
index 2a0cc6e..23c34d1 100644
--- a/test/unit/evault/modules/Liquidation/basic.t.sol
+++ b/test/unit/evault/modules/Liquidation/basic.t.sol
@@ -6,7 +6,7 @@ import {EVaultTestBase} from "../../EVaultTestBase.t.sol";
 import {Events} from "../../../../../src/EVault/shared/Events.sol";
 import {SafeERC20Lib} from "../../../../../src/EVault/shared/lib/SafeERC20Lib.sol";
 import {IAllowanceTransfer} from "permit2/src/interfaces/IAllowanceTransfer.sol";
-
+import {TestERC20} from "../../../../mocks/TestERC20.sol";
 import {console} from "forge-std/Test.sol";
 
 import "../../../../../src/EVault/shared/types/Types.sol";
@@ -183,4 +183,60 @@ contract LiquidationUnitTest is EVaultTestBase {
         assertEq(eTST.debtOf(borrower), 0);
         assertEq(eTST2.balanceOf(borrower), 0);
     }
+
+    
+    function test_t0x1c_yieldAndSocialization() public {
+        TestERC20 assetTST3;
+        assetTST3 = new TestERC20("Test Token 3", "TST3", 18, false);
+        IEVault eTST3 = IEVault(
+            factory.createProxy(address(0), true, abi.encodePacked(address(assetTST3), address(oracle), unitOfAccount))
+        );
+        oracle.setPrice(address(assetTST3), unitOfAccount, 1e18);
+
+        startHoax(borrower);
+        assetTST3.mint(borrower, type(uint256).max);
+        assetTST3.approve(address(eTST3), type(uint256).max);
+        uint256 borrowerCollateralQty = 10e18;
+        eTST3.deposit(borrowerCollateralQty, borrower);
+        evc.enableCollateral(borrower, address(eTST3));
+        evc.enableController(borrower, address(eTST));
+        vm.stopPrank();
+
+        uint256 ltv = 0.5e4;
+        eTST.setLTV(address(eTST3), uint16(ltv), uint16(ltv), 0);
+
+        startHoax(borrower);
+        uint borrowAmt = 4e18;
+        eTST.borrow(borrowAmt, borrower);
+        assertEq(assetTST.balanceOf(borrower), borrowAmt);
+        vm.stopPrank();
+
+        // collateral price drops
+        uint droppedPrice = 0.5e18;
+        oracle.setPrice(address(assetTST3), unitOfAccount, droppedPrice);
+        (uint256 collateralValueB, uint256 liabilityValueB) = eTST.accountLiquidity(borrower, false);
+        emit log_named_decimal_uint("collateralValue (adjusted) is      =", collateralValueB, 18); // = borrowerCollateralQty * droppedPrice * ltv = 10e18 * 0.5 * 0.5 = 2.5e18
+        emit log_named_decimal_uint("liabilityValue is                  =", liabilityValueB, 18); 
+
+        (uint256 maxRepay, uint256 yield) = eTST.checkLiquidation(liquidator, borrower, address(eTST3));
+        emit log_named_decimal_uint("maxRepay                           =", maxRepay, 18); // @audit-issue : reduced due to incorrect calculation from 4e18 to 3.125e18
+        emit log_named_decimal_uint("yield                              =", yield, 18);
+
+        startHoax(liquidator);
+        uint depositInitialAmt = 1000e18;
+        evc.enableCollateral(liquidator, address(eTST3));
+        evc.enableController(liquidator, address(eTST));
+        assetTST3.mint(liquidator, type(uint256).max);
+        assetTST3.approve(address(eTST3), type(uint256).max);
+        eTST3.deposit(depositInitialAmt, liquidator);
+        
+        uint prevBal1 = eTST3.balanceOf(liquidator);
+        eTST.liquidate(borrower, address(eTST3), type(uint256).max, 0);
+
+        emit log_named_decimal_uint("debt of liquidator                 =", eTST.debtOf(liquidator), 18);
+        emit log_named_decimal_uint("collateral received by liquidator  =", eTST3.balanceOf(liquidator) - prevBal1, 18);
+        emit log_named_decimal_uint("collateral value of received       =", droppedPrice * (eTST3.balanceOf(liquidator) - prevBal1) / 1e18, 18);
+        emit log_named_decimal_uint("remaining debt of borrower         =", eTST.debtOf(borrower), 18);
+        emit log_named_decimal_uint("collateral balance of borrower     =", eTST3.balanceOf(borrower), 18);
+    }
 }
```

</details>
<br>

Apply the provided patch and run `test_t0x1c_yieldAndSocialization()` to see the following output:
```
Ran 1 test for test/unit/evault/modules/Liquidation/basic.t.sol:LiquidationUnitTest
[PASS] test_t0x1c_yieldAndSocialization() (gas: 4466394)
Logs:
collateralValue (adjusted) is      =: 2.500000000000000000
liabilityValue is                  =: 4.000000000000000000
maxRepay                           =: 3.125000000000000000
yield                              =: 10.000000000000000000
Socialized debt                    =: 0.875000000000000000
debt of liquidator                 =: 3.125000000000000000   <----------- debt worth or liabilityValue
collateral received by liquidator  =: 10.000000000000000000
collateral value of received       =: 5.000000000000000000   <----------- collateral worth or collateralValue
remaining debt of borrower         =: 0.000000000000000000
collateral balance of borrower     =: 0.000000000000000000
```
<br>

## Proof of Concept-2

<details><summary>Click to view patch-2 (to be applied on the commit with tag `cantina-contest`)</summary>

```diff
diff --git a/src/EVault/modules/Liquidation.sol b/src/EVault/modules/Liquidation.sol
index 07e5bb3..8ddf853 100644
--- a/src/EVault/modules/Liquidation.sol
+++ b/src/EVault/modules/Liquidation.sol
@@ -8,7 +8,7 @@ import {BalanceUtils} from "../shared/BalanceUtils.sol";
 import {LiquidityUtils} from "../shared/LiquidityUtils.sol";
 
 import "../shared/types/Types.sol";
-
+import {console} from "forge-std/Test.sol";
 /// @title LiquidationModule
 /// @custom:security-contact security@euler.xyz
 /// @author Euler Labs (https://www.eulerlabs.com/)
@@ -214,6 +214,7 @@ abstract contract LiquidationModule is ILiquidation, BalanceUtils, LiquidityUtil
         ) {
             Assets owedRemaining = liqCache.liability.subUnchecked(liqCache.repay);
             decreaseBorrow(vaultCache, liqCache.violator, owedRemaining);
+            console.log("\nSocialized debt                    =: %s.%s\n", owedRemaining.toUint()/1e18, owedRemaining.toUint() - owedRemaining.toUint()/1e18 * 1e18); // express in decimal notation
 
             // decreaseBorrow emits Repay without any assets entering the vault. Emit Withdraw from and to zero address
             // to cover the missing amount for offchain trackers.
diff --git a/test/unit/evault/modules/Liquidation/basic.t.sol b/test/unit/evault/modules/Liquidation/basic.t.sol
index 2a0cc6e..cf413e5 100644
--- a/test/unit/evault/modules/Liquidation/basic.t.sol
+++ b/test/unit/evault/modules/Liquidation/basic.t.sol
@@ -6,7 +6,7 @@ import {EVaultTestBase} from "../../EVaultTestBase.t.sol";
 import {Events} from "../../../../../src/EVault/shared/Events.sol";
 import {SafeERC20Lib} from "../../../../../src/EVault/shared/lib/SafeERC20Lib.sol";
 import {IAllowanceTransfer} from "permit2/src/interfaces/IAllowanceTransfer.sol";
-
+import {TestERC20} from "../../../../mocks/TestERC20.sol";
 import {console} from "forge-std/Test.sol";
 
 import "../../../../../src/EVault/shared/types/Types.sol";
@@ -183,4 +183,63 @@ contract LiquidationUnitTest is EVaultTestBase {
         assertEq(eTST.debtOf(borrower), 0);
         assertEq(eTST2.balanceOf(borrower), 0);
     }
+
+    
+    function test_t0x1c_ProfitSelfLiquidation() public {
+        TestERC20 assetTST3;
+        assetTST3 = new TestERC20("Test Token 3", "TST3", 18, false);
+        IEVault eTST3 = IEVault(
+            factory.createProxy(address(0), true, abi.encodePacked(address(assetTST3), address(oracle), unitOfAccount))
+        );
+        oracle.setPrice(address(assetTST3), unitOfAccount, 1e18);
+        oracle.setPrice(address(assetTST), unitOfAccount, 7.99e18);
+
+        startHoax(borrower);
+        assetTST3.mint(borrower, type(uint256).max);
+        assetTST3.approve(address(eTST3), type(uint256).max);
+        uint256 borrowerCollateralQty = 10e18;
+        eTST3.deposit(borrowerCollateralQty, borrower);
+        evc.enableCollateral(borrower, address(eTST3));
+        evc.enableController(borrower, address(eTST));
+        vm.stopPrank();
+
+        uint256 ltv = 0.8e4;
+        eTST.setLTV(address(eTST3), uint16(ltv), uint16(ltv), 0);
+
+        startHoax(borrower);
+        uint borrowAmt = 1e18;
+        eTST.borrow(borrowAmt, borrower);
+        assertEq(assetTST.balanceOf(borrower), borrowAmt);
+        vm.stopPrank();
+
+        // debt asset price increases
+        uint debtAssetPrice = 9e18;
+        oracle.setPrice(address(assetTST), unitOfAccount, debtAssetPrice);
+        (uint256 collateralValueB, uint256 liabilityValueB) = eTST.accountLiquidity(borrower, false);
+        emit log_named_decimal_uint("collateralValue (adjusted) is      =", collateralValueB, 18);
+        emit log_named_decimal_uint("liabilityValue is                  =", liabilityValueB, 18); 
+
+        (uint256 maxRepay, uint256 yield) = eTST.checkLiquidation(liquidator, borrower, address(eTST3));
+        emit log_named_decimal_uint("maxRepay                           =", maxRepay, 18); 
+        emit log_named_decimal_uint("yield                              =", yield, 18);
+
+        startHoax(liquidator);
+        uint depositInitialAmt = 1000e18;
+        evc.enableCollateral(liquidator, address(eTST3));
+        evc.enableController(liquidator, address(eTST));
+        assetTST3.mint(liquidator, type(uint256).max);
+        assetTST3.approve(address(eTST3), type(uint256).max);
+        eTST3.deposit(depositInitialAmt, liquidator);
+        
+        uint prevBal1 = eTST3.balanceOf(liquidator);
+        eTST.liquidate(borrower, address(eTST3), type(uint256).max, 0);
+
+        (uint256 collateralValueL, uint256 liabilityValueL) = eTST.accountLiquidity(liquidator, false);
+
+        emit log_named_decimal_uint("debt of liquidator                 =", eTST.debtOf(liquidator), 18);
+        emit log_named_decimal_uint("debt value of liquidator           =", liabilityValueL, 18);
+        emit log_named_decimal_uint("collateral value of received       =", 1e18 * (eTST3.balanceOf(liquidator) - prevBal1) / 1e18, 18);
+        emit log_named_decimal_uint("remaining debt of borrower         =", eTST.debtOf(borrower), 18);
+        emit log_named_decimal_uint("collateral balance of borrower     =", eTST3.balanceOf(borrower), 18);
+    }
 }
```

</details>
<br>

Output-2:
```
Ran 1 test for test/unit/evault/modules/Liquidation/basic.t.sol:LiquidationUnitTest
[PASS] test_t0x1c_ProfitSelfLiquidation() (gas: 4500491)
Logs:
  collateralValue (adjusted) is      =: 8.000000000000000000
  liabilityValue is                  =: 9.000000000000000000
  maxRepay                           =: 0.987654320987654320
  yield                              =: 10.000000000000000000

Socialized debt                    =: 0.12345679012345680

  debt of liquidator                 =: 0.987654320987654320
  debt value of liquidator           =: 8.888888888888888880     <-------
  collateral value of received       =: 10.000000000000000000    <------- Profit during liquidation: 10 - 8.888888888888888880 = 1.11111111111111112
  remaining debt of borrower         =: 0.000000000000000000
  collateral balance of borrower     =: 0.000000000000000000
```
<br>

## Recommendation
Modify [L164](https://github.com/euler-xyz/euler-vault-kit/blob/cantina-contest/src/EVault/modules/Liquidation.sol#L164) so that it uses the correct scaling-down formula while ensuring that the difference in collateralValue & liabilityValue does not provide a profit to the borrower if they act as liquidators. While the final solution depends on the design-decision of the protocol, one approach could be:
1. If there aren't enough funds to cover the `maxYieldValue` during lliquidation, then there should be no compulsion to stick to the `discountFactor`. In our example-1, where `liabilityValue = 4` and `collateralValue = 5` and `collateralValue < maxYieldValue`, we could easily recalculate the payout ratio that liquidating `4` debt equals the liquidator getting `5`. No socialization is required yet. You can optionally let the liquidator provide a slippage param like `min_CollateralValue_By_DebtValue_Ratio`, so that a payout below that reverts.
2. Further, if and when `collateralValue` goes below (or equal to) the `liabilityValue`, any actor can call `liquidate()` at which point the outstanding debt can be socialized.

Even if Euler chooses to not follow the above remediation for some reason and goes with an alternate fix, I believe it would be fair to comment that the **new calculations should be based on the debt value of `3.5355`** as shown in the analysis above. This would still allow the borrower to profit from self-lquidation but the quantum of profit will be lesser than that in the current scenario. At the same time, such an implementation will continue to be consistent with the current intended logic followed by the protocol.

## Existing documentation around the issue
Please note that Section 3.1.4 of https://docs.euler.finance/Dutch_Liquidation_Analysis.pdf mentions the following in the context of flash loan attacks:
>"
>
> 3.1.4 Summary
>
> For a flashloan attack, the attacker sandwiches a price oracle update, either when collateral decreases in price or the debt asset increases in price. Their goal is to flashloan and set up an unhealthy position that they can self-liquidate for profit moments later.
>
> In both cases, the lower the LTV parameter v, the less likely an attack is to happen, because a much larger drop in price of collateral or much larger increase in the price of the debt asset is required for a malicious actor to be profitable.
>
> Flashloan attacks cannot be made economically infeasible in the absence of other mechanisms without setting v = 0. They can only be made less likely by choosing v to be a value that is unlikely to ever correspond to a collateral price drop or debt asset price rise of a certain size.
>
>"

Euler has already taken steps to mitigate instant self-liquidation attacks by incorporating a cooldown period check. However, the specific case (even in the absence of flash loans) of the scenario where `collateralValue < maxYieldValue` isn't handled properly. 

[Back to Top](#summaryTable)

---

### <a id="h-02"></a>[H-02]
## **Lack of slippage protection & deadline in pullDebt() can lead to loss for caller**
#### https://github.com/euler-xyz/euler-vault-kit/blob/cantina-contest/src/EVault/modules/Liquidation.sol#L163
<br>

## Description
The [pullDebt()](https://github.com/euler-xyz/euler-vault-kit/blob/cantina-contest/src/EVault/modules/Liquidation.sol#L163) function allows anyone to transfer a borrower's debt to themselves. The protocol expects the caller to pass `amount` as a param which is the `"Amount of debt in asset units"` as per the natspec:
```js
  File: src/EVault/modules/Borrowing.sol

   131:@--->         function pullDebt(uint256 amount, address from) public virtual nonReentrant returns (uint256) {
   132:                  (VaultCache memory vaultCache, address account) = initOperation(OP_PULL_DEBT, CHECKACCOUNT_CALLER);
   133:
   134:                  if (from == account) revert E_SelfTransfer();
   135:
   136:                  Assets assets = amount == type(uint256).max ? getCurrentOwed(vaultCache, from).toAssetsUp() : amount.toAssets();
   137:
   138:                  if (assets.isZero()) return 0;
   139:                  transferBorrow(vaultCache, from, account, assets);
   140:
   141:                  emit PullDebt(from, account, assets.toUint());
   142:
   143:                  return assets.toUint();
   144:              }
```

[Natspec](https://github.com/euler-xyz/euler-vault-kit/blob/cantina-contest/src/EVault/IEVault.sol#L250-L254):
```js
  File: src/EVault/IEVault.sol

   250:              /// @notice Take over debt from another account
   251:@--->         /// @param amount Amount of debt in asset units (use max uint256 for all the account's debt)
   252:              /// @param from Account to pull the debt from
   253:              /// @return Amount of debt pulled in asset units.
   254:              function pullDebt(uint256 amount, address from) external returns (uint256);
```

The protocol however offers no way for the user to specify either a slippage param by way of a `min_CollateralAdjustedValue_By_DebtValue_Ratio` or a deadline param. 

## Impact
This means that if the transaction does not get executed immediately, which is quite common, then a price fluctuation in either the debt asset (upward) or the collateral asset (downward) can result in the caller pulling a much higher debt than they had anticipated. Additional absence of a deadline means this could remain in the mempool for a long time before getting executed at a worse price.
<br>

**There could even be a case where the debt finally gets pulled at a rate very close to the permissible LTV and then even a slight price movement causes the caller to be liquidated, thus resulting in a loss.**

## Recommendation
Allow the caller to specify the lowest `min_CollateralAdjustedValue_By_DebtValue_Ratio` they are comfortable with and also the deadline timestamp. If either are violated, revert the transaction.
<br>

In general, [this informative article](https://dacian.me/defi-slippage-attacks#heading-minting-exposes-users-to-unlimited-slippage) quite nicely outlines the situations or functions where transfer of tokens may be taking place but are masked under a different name, and hence should always incorporate slippage protection & deadline params.

[Back to Top](#summaryTable)

---

### <a id="h-03"></a>[H-03]
## **In spite of correctly specifying minYieldBalance, incomplete slippage protection & missing deadline in liquidate() lead to loss**
#### https://github.com/euler-xyz/euler-vault-kit/blob/cantina-contest/src/EVault/modules/Liquidation.sol#L46-L57
<br>

## Description
The [liquidate()](https://github.com/euler-xyz/euler-vault-kit/blob/cantina-contest/src/EVault/modules/Liquidation.sol#L46-L57) function allows the liquidator to specify a `minYieldBalance` param. The protocol explains this to be the `"The minimum acceptable amount of collateral to be transferred from violator to sender, in collateral balance units"` as per the natspec:
```js
  File: src/EVault/modules/Liquidation.sol

    46:@--->         function liquidate(address violator, address collateral, uint256 repayAssets, uint256 minYieldBalance)
    47:                  public
    48:                  virtual
    49:                  nonReentrant
    50:              {
    51:                  (VaultCache memory vaultCache, address liquidator) = initOperation(OP_LIQUIDATE, CHECKACCOUNT_CALLER);
    52:
    53:                  LiquidationCache memory liqCache =
    54:                      calculateLiquidation(vaultCache, liquidator, violator, collateral, repayAssets);
    55:
    56:@--->             executeLiquidation(vaultCache, liqCache, minYieldBalance);
    57:              }
```

[Natspec](https://github.com/euler-xyz/euler-vault-kit/blob/cantina-contest/src/EVault/IEVault.sol#L283-L290):
```js
  File: src/EVault/IEVault.sol

   283:              /// @notice Attempts to perform a liquidation
   284:              /// @param violator Address that may be in collateral violation
   285:              /// @param collateral Collateral which is to be seized
   286:              /// @param repayAssets The amount of underlying debt to be transferred from violator to sender, in asset units (use
   287:              /// max uint256 to repay the maximum possible amount).
   288:@--->         /// @param minYieldBalance The minimum acceptable amount of collateral to be transferred from violator to sender, in
   289:@--->         /// collateral balance units (shares for vaults)
   290:              function liquidate(address violator, address collateral, uint256 repayAssets, uint256 minYieldBalance) external;
```

"The minimum acceptable amount of collateral" however is not enough to provide slippage protection **_even when the user correctly specifies `minYieldBalance`_**, as is shown in the following Proof of Concept section. It gives the liquidator very little control over the expected price ratio of debt & collateral. They should have been able to specify a `min_CollateralValue_By_DebtValue_Ratio`. Also, there is no provision to specify a deadline parameter while calling `liquidate()`.

## Impact
This means that if the transaction does not get executed immediately, which is quite common, then a price fluctuation in either the debt asset (upward) or the collateral asset (downward) can result in the liquidator taking on a much higher debt than they had anticipated. Additional absence of a deadline means this could remain in the mempool for a long time before getting executed at a worse price.
<br>

**There could even be a case where the debt finally gets transferred to the liquidator at a rate just above par and then even a slight price movement causes the caller to be either in loss or even worse, get liquidated if LTV is breached.**

## Proof of Concept
Consider the following example where the liquidator specifies `10e18` as the `minYieldBalance` so that he gets at least 10 units of collateral after the transaction. This however is not sufficient to protect him from an adverse price movement (the examples are reproduced with the same numbers in the coded PoC):
- Let's assume LTV of 90%. Bob borrows `8e18` against a collateral of `10e18` (10 collateral tokens priced at `1e18` each). The configuration for current `maxLiquidationDiscount` is `0.2e4`.

- Collateral price drops to `0.8e18` per token. Debt price rises to `1.1e18` per token. Hence, total `collateralValue = 8e18` while total `liabilityValue = 8.8e18`. Debt is now greater than permissible LTV, hence Bob can be liquidated.

- The liquidator expects a decent profit to be made after doing his calculations and hence calls `liquidate(violator, collateralAddr, type(uint256).max, 10e18);`. **Note the slippage protection specified by him of 10 tokens which is respected in both the cases, yet still dangerous for him in the second case**. Also, for our examples he maintains an initial collateral balance of `10` before calling `liquidate()`.

- Let's consider 2 cases (expectation vs reality):
    - **_Case-1: (Expectation)_**
        - The `liabilityValue` transferred over to him in this case is `6.545454545454545447e18` while the `collateralAdjustedValue` transferred is ` 7.200000000000000007e18`.
        - He is comfortably below the max LTV and enjoys a healthy profit.

    - **_Case-2: (Reality)_**
        - The tx with call to `liquidate()` remains in the mempool for a few seconds or minutes. Meanwhile, the debt price drops to `1e18` per token.
        - When the tx finally gets picked up, the `liabilityValue` transferred over to the liquidator in this case is `7.2e18` while the `collateralAdjustedValue` transferred is ` 7.200000000000000007e18`.
        - He is just below the allowed LTV and precariously close to being eligible for being liquidated even if a slight negative price movement happens in next few seconds. Also, his profits are negligible or could even be negative after taking into account the gas cost.
<br>

Apply the following patch and run `test_t0x1c_incompleteSlippage()` to see the results:

<details><summary>Click to view patch</summary>

```diff
diff --git a/test/unit/evault/modules/Liquidation/basic.t.sol b/test/unit/evault/modules/Liquidation/basic.t.sol
index 2a0cc6e..b273a90 100644
--- a/test/unit/evault/modules/Liquidation/basic.t.sol
+++ b/test/unit/evault/modules/Liquidation/basic.t.sol
@@ -6,7 +6,7 @@ import {EVaultTestBase} from "../../EVaultTestBase.t.sol";
 import {Events} from "../../../../../src/EVault/shared/Events.sol";
 import {SafeERC20Lib} from "../../../../../src/EVault/shared/lib/SafeERC20Lib.sol";
 import {IAllowanceTransfer} from "permit2/src/interfaces/IAllowanceTransfer.sol";
-
+import {TestERC20} from "../../../../mocks/TestERC20.sol";
 import {console} from "forge-std/Test.sol";
 
 import "../../../../../src/EVault/shared/types/Types.sol";
@@ -183,4 +183,101 @@ contract LiquidationUnitTest is EVaultTestBase {
         assertEq(eTST.debtOf(borrower), 0);
         assertEq(eTST2.balanceOf(borrower), 0);
     }
+    
+    function test_t0x1c_incompleteSlippage() public {
+        TestERC20 assetTST3;
+        assetTST3 = new TestERC20("Test Token 3", "TST3", 18, false);
+
+        IEVault eTST3 = IEVault(
+            factory.createProxy(address(0), true, abi.encodePacked(address(assetTST3), address(oracle), unitOfAccount))
+        );
+
+        oracle.setPrice(address(assetTST3), unitOfAccount, 1e18); 
+
+        startHoax(borrower);
+
+        assetTST3.mint(borrower, type(uint256).max);
+        assetTST3.approve(address(eTST3), type(uint256).max);
+
+        uint256 borrowerCollateralQty = 10e18;
+        eTST3.deposit(borrowerCollateralQty, borrower);
+
+        evc.enableCollateral(borrower, address(eTST3));
+        evc.enableController(borrower, address(eTST));
+        vm.stopPrank();
+
+        uint256 ltv = 0.9e4;
+        eTST.setLTV(address(eTST3), uint16(ltv), uint16(ltv), 0);
+
+        startHoax(borrower);
+        uint borrowAmt = 8e18;
+        eTST.borrow(borrowAmt, borrower);
+        assertEq(assetTST.balanceOf(borrower), borrowAmt);
+        vm.stopPrank();
+
+        uint depositInitialAmt = 10; // buffer collateral maintained by liquidator before liquidating
+
+        uint256 collateralDroppedPrice = 0.8e18; 
+
+        uint256 snapshot = vm.snapshot();
+        console.log("\n============================== Case_1 (Liquidator's Expectation) ==============================================\n");
+
+        uint256 newDebtAssetPrice = 1.1e18; // Case_1 (asset price rises at the same time)
+        
+        oracle.setPrice(address(assetTST3), unitOfAccount, collateralDroppedPrice);
+        oracle.setPrice(address(assetTST), unitOfAccount, newDebtAssetPrice);
+
+        startHoax(liquidator);
+
+        evc.enableCollateral(liquidator, address(eTST3));
+        evc.enableController(liquidator, address(eTST));
+
+        assetTST3.mint(liquidator, type(uint256).max);
+        assetTST3.approve(address(eTST3), type(uint256).max);
+        eTST3.deposit(depositInitialAmt, liquidator);
+        
+        uint prevBal1 = eTST3.balanceOf(liquidator);
+        eTST.liquidate(borrower, address(eTST3), type(uint256).max, 10e18); // @audit-info : user has provided `minYieldBalance = 10 tokens`
+
+        (uint256 collateralValueL, uint256 liabilityValueL) = eTST.accountLiquidity(liquidator, false);
+
+        emit log_named_decimal_uint("debt balance of liquidator                 =", eTST.debtOf(liquidator), 18);
+        emit log_named_decimal_uint("debtValue of liquidator                    =", newDebtAssetPrice * eTST.debtOf(liquidator) / 1e18, 18);
+        emit log_named_decimal_uint("collateral value received by liquidator    =", collateralDroppedPrice * (eTST3.balanceOf(liquidator) - prevBal1) / 1e18, 18);
+        emit log_named_decimal_uint("collateral balance received by liquidator  =", eTST3.balanceOf(liquidator) - prevBal1, 18);
+        emit log_named_decimal_uint("liquidator's total collateralAdjustedValue =", collateralValueL, 18);
+        emit log_named_decimal_uint("remaining debt of borrower                 =", eTST.debtOf(borrower), 18);
+        emit log_named_decimal_uint("collateral balance of borrower             =", eTST3.balanceOf(borrower), 18);
+
+
+        console.log("\n\n============================== Case_2 (Reality) ==============================================\n");
+
+        vm.revertTo(snapshot);
+        newDebtAssetPrice = 1e18; // Case_2 (no change)
+
+        oracle.setPrice(address(assetTST3), unitOfAccount, collateralDroppedPrice);
+        oracle.setPrice(address(assetTST), unitOfAccount, newDebtAssetPrice);
+
+        startHoax(liquidator);
+
+        evc.enableCollateral(liquidator, address(eTST3));
+        evc.enableController(liquidator, address(eTST));
+
+        assetTST3.mint(liquidator, type(uint256).max);
+        assetTST3.approve(address(eTST3), type(uint256).max);
+        eTST3.deposit(depositInitialAmt, liquidator);
+        
+        prevBal1 = eTST3.balanceOf(liquidator);
+        eTST.liquidate(borrower, address(eTST3), type(uint256).max, 10e18); // @audit-info : user has provided `minYieldBalance = 10 tokens`
+
+        (collateralValueL, liabilityValueL) = eTST.accountLiquidity(liquidator, false);
+
+        emit log_named_decimal_uint("debt balance of liquidator                 =", eTST.debtOf(liquidator), 18);
+        emit log_named_decimal_uint("debtValue of liquidator                    =", newDebtAssetPrice * eTST.debtOf(liquidator) / 1e18, 18);
+        emit log_named_decimal_uint("collateral value received by liquidator    =", collateralDroppedPrice * (eTST3.balanceOf(liquidator) - prevBal1) / 1e18, 18);
+        emit log_named_decimal_uint("collateral balance received by liquidator  =", eTST3.balanceOf(liquidator) - prevBal1, 18);
+        emit log_named_decimal_uint("liquidator's total collateralAdjustedValue =", collateralValueL, 18);
+        emit log_named_decimal_uint("remaining debt of borrower                 =", eTST.debtOf(borrower), 18);
+        emit log_named_decimal_uint("collateral balance of borrower             =", eTST3.balanceOf(borrower), 18);
+    }
 }
```

</details>
<br>

Output:
```
Ran 1 test for test/unit/evault/modules/Liquidation/basic.t.sol:LiquidationUnitTest
[PASS] test_t0x1c_incompleteSlippage() (gas: 4877933)
Logs:

============================== Case_1 (Liquidator's Expectation) ==============================================

  debt balance of liquidator                 =: 5.950413223140495861
  debtValue of liquidator                    =: 6.545454545454545447    <-------
  collateral value received by liquidator    =: 8.000000000000000000
  collateral balance received by liquidator  =: 10.000000000000000000   --------> equals 10 tokens i.e. `minYieldBalance` hence tx does not revert.
  liquidator's total collateralAdjustedValue =: 7.200000000000000007    <------- Ample gap between this and the above debtValue. Liquidator is quite safe from liquidation.
  remaining debt of borrower                 =: 0.000000000000000000
  collateral balance of borrower             =: 0.000000000000000000


============================== Case_2 (Reality) ==============================================

  debt balance of liquidator                 =: 7.200000000000000000
  debtValue of liquidator                    =: 7.200000000000000000    <------- higher debtValue than Case_1; lower profits. Might even be a loss after gas costs.
  collateral value received by liquidator    =: 8.000000000000000000
  collateral balance received by liquidator  =: 10.000000000000000000   --------> equals 10 tokens i.e. `minYieldBalance` hence tx does not revert.
  liquidator's total collateralAdjustedValue =: 7.200000000000000007    <------- Precariously close to the above debtValue. Liquidator can be liquidated even if a slight negative price movement happens in next few seconds.
  remaining debt of borrower                 =: 0.000000000000000000
  collateral balance of borrower             =: 0.000000000000000000
```
<br>

## Recommendation
Allow the caller to specify the lowest `min_CollateralValue_By_DebtValue_Ratio` they are comfortable with and also the deadline timestamp. If either are violated, revert the transaction.

[Back to Top](#summaryTable)

---

### <a id="h-04"></a>[H-04]
## **Missing slippage protection & deadline in borrow() can lead to loss for the borrower**
#### https://github.com/euler-xyz/euler-vault-kit/blob/cantina-contest/src/EVault/modules/Borrowing.sol#L64-L78
<br>

## Description
The [borrow()](https://github.com/euler-xyz/euler-vault-kit/blob/cantina-contest/src/EVault/modules/Borrowing.sol#L64-L78) function expects the borrower to pass `amount` as a param which is the `"Amount of assets to borrow"` as per the natspec:
```js
  File: src/EVault/modules/Borrowing.sol

    64:              /// @inheritdoc IBorrowing
    65:@--->         function borrow(uint256 amount, address receiver) public virtual nonReentrant returns (uint256) {
    66:                  (VaultCache memory vaultCache, address account) = initOperation(OP_BORROW, CHECKACCOUNT_CALLER);
    67:
    68:                  Assets assets = amount == type(uint256).max ? vaultCache.cash : amount.toAssets();
    69:                  if (assets.isZero()) return 0;
    70:
    71:                  if (assets > vaultCache.cash) revert E_InsufficientCash();
    72:
    73:                  increaseBorrow(vaultCache, account, assets);
    74:
    75:                  pushAssets(vaultCache, receiver, assets);
    76:
    77:                  return assets.toUint();
    78:              }
```

[Natspec](https://github.com/euler-xyz/euler-vault-kit/blob/cantina-contest/src/EVault/IEVault.sol#L230-L234):
```js
  File: src/EVault/IEVault.sol

   230:              /// @notice Transfer underlying tokens from the vault to the sender, and increase sender's debt
   231:@--->         /// @param amount Amount of assets to borrow (use max uint256 for all available tokens)
   232:              /// @param receiver Account receiving the borrowed tokens
   233:              /// @return Amount of assets borrowed
   234:              function borrow(uint256 amount, address receiver) external returns (uint256);
```

The protocol thus offers no way for the user to specify either a slippage param by way of a `min_CollateralAdjustedValue_By_DebtValue_Ratio` or a deadline param.

## Impact
In case the transaction does not get executed immediately, which is quite common, then a price fluctuation in either the debt asset (upward) or the collateral asset (downward) can result in the caller borrowing a much higher debt than they had anticipated. Additional absence of a deadline means this could remain in the mempool for a long time before getting executed at a worse price.
<br>

**There could even be a case where the borrow finally gets executed at a rate very close to the permissible LTV and subsequently even a slight price movement causes the caller to be liquidated, thus resulting in a loss.**

## Recommendation
Allow the borrower to specify the lowest `min_CollateralAdjustedValue_By_DebtValue_Ratio` they are comfortable with and also the deadline timestamp. If either are violated, revert the transaction.

[Back to Top](#summaryTable)

<br><br>

## **MEDIUM-SEVERITY BUGS**
---
 
### <a id="m-01"></a>[M-01]
## **Excess amountIn is never refunded when using swapToUnderlyingGivenIn() or swapToSynthGivenIn()**
#### https://github.com/euler-xyz/euler-vault-kit/blob/cantina-contest/src/Synths/PegStabilityModule.sol#L75-L78
#### https://github.com/euler-xyz/euler-vault-kit/blob/cantina-contest/src/Synths/PegStabilityModule.sol#L107-L110
<br>

## Description
It's a norm in the DeFi space that while calling a `swap()`, if the user has provided some `amountIn` which is unused or over & above what was required to swap into `amountOut`, then the excess funds are returned. However in Euler, due to the rounding-down which happens in favour of the protocol, any extra Synth or Underlying which the user specifies while calling `swapToUnderlyingGivenIn()` or `swapToSynthGivenIn()` is never returned by the protocol & is completely burned/pulled. <br>

Consider the following scenario which has been later reproduced in the PoC. Please refer those tests for verifying the numbers:
- Suppose Alice wanted to swap some of her Synth to receive `290,000 ether` of the underlying.
- Let's assume a fee of `70%` and conversion rate of `1e18`.
- Option 1: 
    - Alice can make use of the `swapToUnderlyingGivenOut()` function by specifying `amountOut` as `290,000 ether`. 
    - It can be seen in the PoC that the protocol burns `966666666666666666666666` of Alice's synth and transfers `290,000 ether` of the underlying.
- Alice however goes for Option 2:
    - She calls `swapToUnderlyingGivenIn()` but with `amountIn` as `966666666666666666666669` i.e. virtually sending in an additional 3 wei of Synth.
    - The protocol burns all of this Synth while providing Alice with the same amount of underlying `290,000 ether`.
    - The additional 3 wei are burned too and never returned to Alice.

The same scenario plays out when a user swaps the other way round using `swapToSynthGivenIn()`, also shown in the PoC.
<br>

In essence the conversion rate is not strictly respected here and can be skewed if this is done multiple times by various users.

## Relevant Code
- https://github.com/euler-xyz/euler-vault-kit/blob/cantina-contest/src/Synths/PegStabilityModule.sol#L75-L78
- https://github.com/euler-xyz/euler-vault-kit/blob/cantina-contest/src/Synths/PegStabilityModule.sol#L107-L110

## Proof of Concept
Add the following file at `test/unit/pegStabilityModules/IncorrectPegSwap.t.sol` and run both the tests via `forge test --mt test_t0x1c -vv` to see them pass:
```js
// SPDX-License-Identifier: GPL-2.0-or-later

pragma solidity ^0.8.0;

import "forge-std/Test.sol";

import {PegStabilityModule, EVCUtil} from "../../../src/Synths/PegStabilityModule.sol";
import {ESynth, IEVC} from "../../../src/Synths/ESynth.sol";
import {TestERC20} from "../../mocks/TestERC20.sol";
import {EthereumVaultConnector} from "ethereum-vault-connector/EthereumVaultConnector.sol";

contract IncorrectPegSwap is Test {
    uint256 public TO_UNDERLYING_FEE = 7000;
    uint256 public TO_SYNTH_FEE = 7000;
    uint256 public BPS_SCALE = 10000;

    ESynth public synth;
    TestERC20 public underlying;

    PegStabilityModule public psm;

    IEVC public evc;

    address public owner = makeAddr("owner");
    address public wallet1 = makeAddr("wallet1");
    address public wallet2 = makeAddr("wallet2");

    function setUp() public {
        // Deploy EVC
        evc = new EthereumVaultConnector();

        // Deploy underlying
        underlying = new TestERC20("TestUnderlying", "TUNDERLYING", 18, false);

        // Deploy synth
        vm.prank(owner);
        synth = new ESynth(evc, "TestSynth", "TSYNTH");

        // Deploy PSM
        vm.prank(owner);
        psm = new PegStabilityModule(address(evc), address(synth), address(underlying), TO_UNDERLYING_FEE, TO_SYNTH_FEE, 1e18);
        vm.label(address(psm), "_PSM_");

        // Give PSM and wallets some underlying
        underlying.mint(address(psm), type(uint128).max / 2);
        underlying.mint(wallet1, type(uint128).max / 2);

        // Approve PSM to spend underlying
        vm.startPrank(wallet1);
        underlying.approve(address(psm), type(uint128).max - 1);
        synth.approve(address(psm), type(uint128).max - 1);
        vm.stopPrank();

        // Set PSM as minter
        vm.prank(owner);
        synth.setCapacity(address(psm), type(uint128).max);

        // Mint some synth to wallets
        vm.startPrank(owner);
        synth.setCapacity(owner, type(uint128).max);
        synth.mint(wallet1, type(uint128).max / 2);
        vm.stopPrank();
    }

    function test_t0x1c_SwapToUnderlying() public {
        uint256 amountOut = 290_000e18;

        vm.startPrank(wallet1);
        uint256 aIn = psm.swapToUnderlyingGivenOut(amountOut, wallet2);
        emit log_named_decimal_uint("synthAmountIn (Case 1)", aIn, 18);

        uint256 balanceBefore = synth.balanceOf(wallet1);
        uint256 aOut = psm.swapToUnderlyingGivenIn(aIn + 3, wallet2); // @audit-info : extra 3 wei supplied by the user while swapping
        assertEq(aOut, amountOut);

        emit log_named_decimal_uint("synthAmountIn (Case 2)", balanceBefore - synth.balanceOf(wallet1), 18);
        assertGt(balanceBefore - synth.balanceOf(wallet1), aIn); // @audit : Excess funds never returned
    }

    function test_t0x1c_SwapToSynth() public {
        uint256 amountOut = 290_000e18;

        vm.startPrank(wallet1);
        uint256 aIn = psm.swapToSynthGivenOut(amountOut, wallet2);
        emit log_named_decimal_uint("underlyingAmountIn (Case 1)", aIn, 18);

        uint256 balanceBefore = underlying.balanceOf(wallet1);
        uint256 aOut = psm.swapToSynthGivenIn(aIn + 3, wallet2); // @audit-info : extra 3 wei supplied by the user while swapping
        assertEq(aOut, amountOut);

        emit log_named_decimal_uint("underlyingAmountIn (Case 2)", balanceBefore - underlying.balanceOf(wallet1), 18);
        assertGt(balanceBefore - underlying.balanceOf(wallet1), aIn); // @audit : Excess funds never returned
    }
}
```

Output:
```
Ran 2 tests for test/unit/pegStabilityModules/IncorrectPegSwap.t.sol:PSMTest
[PASS] test_t0x1c_SwapToSynth() (gas: 110085)
Logs:
  underlyingAmountIn (Case 1): 966666.666666666666666666
  underlyingAmountIn (Case 2): 966666.666666666666666669

[PASS] test_t0x1c_SwapToUnderlying() (gas: 110574)
Logs:
  synthAmountIn (Case 1): 966666.666666666666666666
  synthAmountIn (Case 2): 966666.666666666666666669
```

## Recommendation
Do a reverse calculation to figure out the actual minimum `amountIn` required and only burn/pull that amount:
```diff
    function swapToUnderlyingGivenIn(uint256 amountIn, address receiver) external returns (uint256) {
        uint256 amountOut = quoteToUnderlyingGivenIn(amountIn);
        if (amountIn == 0 || amountOut == 0) {
            return 0;
        }

+       uint256 amountInRequired = amountOut * BPS_SCALE * PRICE_SCALE / (BPS_SCALE - TO_UNDERLYING_FEE) / conversionPrice;        
-       synth.burn(_msgSender(), amountIn);
+       synth.burn(_msgSender(), amountInRequired);
        underlying.safeTransfer(receiver, amountOut);

        return amountOut;
    }
```

and

```diff
    function swapToSynthGivenIn(uint256 amountIn, address receiver) external returns (uint256) {
        uint256 amountOut = quoteToSynthGivenIn(amountIn);
        if (amountIn == 0 || amountOut == 0) {
            return 0;
        }

+       uint256 amountInRequired = amountOut * BPS_SCALE * conversionPrice / (BPS_SCALE - TO_SYNTH_FEE) / PRICE_SCALE;       
-       underlying.safeTransferFrom(_msgSender(), address(this), amountIn);
+       underlying.safeTransferFrom(_msgSender(), address(this), amountInRequired);
        synth.mint(receiver, amountOut);

        return amountOut;
    }
```

[Back to Top](#summaryTable)

---
 
### <a id="m-02"></a>[M-02]
## **Swaps can be split into smaller pieces across multiple transactions in order to pay less fees**
#### https://github.com/euler-xyz/euler-vault-kit/blob/cantina-contest/src/Synths/PegStabilityModule.sol#L140
#### https://github.com/euler-xyz/euler-vault-kit/blob/cantina-contest/src/Synths/PegStabilityModule.sol#L154
<br>

## Description
The swap functions inside PegStabilityModule named [quoteToUnderlyingGivenOut()](https://github.com/euler-xyz/euler-vault-kit/blob/cantina-contest/src/Synths/PegStabilityModule.sol#L140) and [quoteToSynthGivenOut()](https://github.com/euler-xyz/euler-vault-kit/blob/cantina-contest/src/Synths/PegStabilityModule.sol#L154) effectively round-down the fee charged by the protocol. This opens up the possibility that the user can split their transaction into multiple smaller amount and get away with paying less than expected fees. <br>
Please refer the PoC for exact numbers. Test case `test_t0x1c_EscapeFee_1()` shows how a user has has to provide `322222222222222222222222` of Synth to receive `290,000 ether` of underlying, which is as expected. <br>
Test `test_t0x1c_EscapeFee_2()` then goes on to show how a malicious user can split their swaps into multiple transactions of `amountOut = 89 ether` each followed by a last tx of `amountOut = 38 ether` to receive the same total of `290,000 ether` but only paying `322222222222222222219326` of Synth this time.
<br>

This strategy can turn out to be profitable on mainnet depending on the gas costs and even more so on other chains like Arbitrum or Optimism which are much less expensive.

## Proof of Concept
Add the following file at `test/unit/pegStabilityModules/SaveSwapFee.t.sol` and run both the tests via `forge test --mt test_t0x1c_EscapeFee -vv` to see them pass with the output showing how the user has to provide a smaller `amountIn` when transactions are split:
```js
// SPDX-License-Identifier: GPL-2.0-or-later

pragma solidity ^0.8.0;

import "forge-std/Test.sol";

import {PegStabilityModule, EVCUtil} from "../../../src/Synths/PegStabilityModule.sol";
import {ESynth, IEVC} from "../../../src/Synths/ESynth.sol";
import {TestERC20} from "../../mocks/TestERC20.sol";
import {EthereumVaultConnector} from "ethereum-vault-connector/EthereumVaultConnector.sol";

contract SaveSwapFee is Test {
    uint256 public TO_UNDERLYING_FEE = 1000;
    uint256 public TO_SYNTH_FEE = 1000;
    uint256 public BPS_SCALE = 10000;

    ESynth public synth;
    TestERC20 public underlying;

    PegStabilityModule public psm;

    IEVC public evc;

    address public owner = makeAddr("owner");
    address public wallet1 = makeAddr("wallet1");
    address public wallet2 = makeAddr("wallet2");

    function setUp() public {
        // Deploy EVC
        evc = new EthereumVaultConnector();

        // Deploy underlying
        underlying = new TestERC20("TestUnderlying", "TUNDERLYING", 18, false);

        // Deploy synth
        vm.prank(owner);
        synth = new ESynth(evc, "TestSynth", "TSYNTH");

        // Deploy PSM
        vm.prank(owner);
        psm = new PegStabilityModule(address(evc), address(synth), address(underlying), TO_UNDERLYING_FEE, TO_SYNTH_FEE, 1e18);
        vm.label(address(psm), "_PSM_");

        // Give PSM and wallets some underlying
        underlying.mint(address(psm), type(uint128).max / 2);
        underlying.mint(wallet1, type(uint128).max / 2);

        // Approve PSM to spend underlying
        vm.startPrank(wallet1);
        underlying.approve(address(psm), type(uint128).max - 1);
        synth.approve(address(psm), type(uint128).max - 1);
        vm.stopPrank();

        // Set PSM as minter
        vm.prank(owner);
        synth.setCapacity(address(psm), type(uint128).max);

        // Mint some synth to wallets
        vm.startPrank(owner);
        synth.setCapacity(owner, type(uint128).max);
        synth.mint(wallet1, type(uint128).max / 2);
        vm.stopPrank();
    }
    
    function test_t0x1c_EscapeFee_1() public {
        uint256 amountOut = 290_000e18;
        uint256 aIn = 0;

        vm.prank(wallet1);
        aIn = psm.swapToUnderlyingGivenOut(amountOut, wallet2);

        emit log_named_decimal_uint("aIn-1", aIn, 18);
    }
    
    function test_t0x1c_EscapeFee_2() public {
        uint256 aOut = 0;
        uint256 aIn = 0;

        uint256 fragments = uint256(290000)/89; // = 3258

        vm.startPrank(wallet1);
        for(uint256 i; i < fragments; i++) {
            aOut += 89e18;
            aIn += psm.swapToUnderlyingGivenOut(89e18, wallet2);
        }
        aIn += psm.swapToUnderlyingGivenOut(290_000e18 - aOut, wallet2); // last swap for the remaining 38e18 (since 290000 - 89 * 3258 = 38)

        emit log_named_decimal_uint("aIn-2", aIn, 18);
    }
}
```

Output:
```
Ran 2 tests for test/unit/pegStabilityModules/SaveSwapFee.t.sol:SaveSwapFee
[PASS] test_t0x1c_EscapeFee_1() (gas: 87224)
Logs:
  aIn-1: 322222.222222222222222222

[PASS] test_t0x1c_EscapeFee_2() (gas: 53724061)
Logs:
  aIn-2: 322222.222222222222219326          <----------------- @audit : less than the above value
```

## Tools Used
Foundry

## Recommended Mitigation Steps
It's advisable to round-up the `amountIn` in favour of the protocol so that such attacks are not profitable. Consider using a [function like `mulDivUp`](https://github.com/transmissions11/solmate/blob/main/src/utils/FixedPointMathLib.sol#L53) from solmate and changing the code for both the functions to look something along these lines:
```diff
    function quoteToUnderlyingGivenOut(uint256 amountOut) public view returns (uint256) {
-       return amountOut * BPS_SCALE * PRICE_SCALE / (BPS_SCALE - TO_UNDERLYING_FEE) / conversionPrice;
+       return amountOut.mulDivUp(BPS_SCALE * PRICE_SCALE, (BPS_SCALE - TO_UNDERLYING_FEE) * conversionPrice);
    }
```

[Back to Top](#summaryTable)

---
 
### <a id="m-03"></a>[M-03]
## **Incorrect smearing calculation when maxGulp is reached**
#### https://github.com/euler-xyz/euler-vault-kit/blob/cantina-contest/src/Synths/EulerSavingsRate.sol#L205-L209
<br>

## Description
The [gulp() function](https://github.com/euler-xyz/euler-vault-kit/blob/cantina-contest/src/Synths/EulerSavingsRate.sol#L205-L209) caps the interest to a max value so that it never overflows `type(uint168).max` and the vault can continue to function normally:
```js
  File: euler-vault-kit/src/Synths/EulerSavingsRate.sol

  199:              function gulp() public nonReentrant {
  200:                  ESRSlot memory esrSlotCache = updateInterestAndReturnESRSlotCache();
  201:
  202:                  uint256 assetBalance = IERC20(asset()).balanceOf(address(this));
  203:                  uint256 toGulp = assetBalance - _totalAssets - esrSlotCache.interestLeft;
  204:
  205: @--->            uint256 maxGulp = type(uint168).max - esrSlotCache.interestLeft;
  206: @--->            if (toGulp > maxGulp) toGulp = maxGulp; // cap interest, allowing the vault to function
  207:
  208:                  esrSlotCache.interestSmearEnd = uint40(block.timestamp + INTEREST_SMEAR);
  209: @--->            esrSlotCache.interestLeft += uint168(toGulp); // toGulp <= maxGulp <= max uint168
  210:
  211:                  // write esrSlotCache back to storage in a single SSTORE
  212:                  esrSlot = esrSlotCache;
  213:              }
```

While this is acceptable, this capped `interestLeft` is now used for calculating the accrued interest as per the smearing logic inside [interestAccruedFromCache()](https://github.com/euler-xyz/euler-vault-kit/blob/cantina-contest/src/Synths/EulerSavingsRate.sol#L252):
```js
  File: euler-vault-kit/src/Synths/EulerSavingsRate.sol

  237:              function interestAccruedFromCache(ESRSlot memory esrSlotCache) internal view returns (uint256) {
  238:                  // If distribution ended, full amount is accrued
  239:                  if (block.timestamp >= esrSlotCache.interestSmearEnd) {
  240:                      return esrSlotCache.interestLeft;
  241:                  }
  242:
  243:                  // If just updated return 0
  244:                  if (esrSlotCache.lastInterestUpdate == block.timestamp) {
  245:                      return 0;
  246:                  }
  247:
  248:                  // Else return what has accrued
  249:                  uint256 totalDuration = esrSlotCache.interestSmearEnd - esrSlotCache.lastInterestUpdate;
  250:                  uint256 timePassed = block.timestamp - esrSlotCache.lastInterestUpdate;
  251:
  252: @--->            return esrSlotCache.interestLeft * timePassed / totalDuration;
  253:              }
```

This is incorrect and instead of this capped `esrSlotCache.interestLeft`, the code should have accounted for the entire remaining interest and then re-checked it for any overflows. Consider the following scenario:
- Let's assume for ease of visualization that instead of `type(uint168).max`, the limit we are working with is `50`. The coded PoC later on shows this with `type(uint168).max` too.
- Assume `depositAmount = 100` and `interestAmount = 80` at timestamp `t`.
- `gulp()` is called at `t`. 
    - As expected right now, `totalAssets = 100` and `interestLeft = 50` which is the capped interest since `80 > 50`.
- We skip to `t2 = t + 1 week` i.e. half of the smear duration.
    - Now even though the `interestLeft` was capped at 50 to prevent overflow, the actual interest remaining was 80. Thus 30 was kept aside for the moment.
    - Let's see what the code does:
        - Accrued interest is calculated as `50 / 2 = 25` and moved to `totalAssets` which now becomes `100 + 25 = 125`.
        - `interestLeft` is calculated as `50 - 25 + 30 = 55`. Since `55 > 50`, it is capped at `50` once again.
    - Let's see what the correct implementation should've been:
        - Accrued interest is calculated as `80 / 2 = 40` and moved to `totalAssets` which now becomes `100 + 40 = 140`.
        - `interestLeft` is calculated as `80 - 40 = 40`. Since `40 < 50`, there's no overflow and hence is acceptable.
- Let us also go one step further to see the calculation. We skip to `t3 = t + 1 week + 1 week`.
    - Let's see the code behaviour:
        - Accrued interest is calculated as `50 / 2 = 25` and moved to `totalAssets` which now becomes `100 + 25 + 25 = 150`.
        - `interestLeft` is calculated as `50 - 25 + 5 = 30`. Since `30 < 50`, it is marked as the interest left.
    - Let's see what ought to be the correct calculation:
        - Accrued interest is calculated as `40 / 2 = 20` and moved to `totalAssets` which now becomes `100 + 40 + 20 = 160`.
        - `interestLeft` is calculated as `40 - 20 = 20`.

As can be seen, current code logic distributes lesser amount to the users. It also underestimates `totalAssets`, affecting any calculations based on it across the protocol.
<br>

- To see the above example in action, please refer "_PoC-2 (Simplified version using `50` instead of `type(uint168.max)` for maxGulp calculations)_".
- To see the PoC with actual numbers using `type(uint168).max` instead of `50`, please refer "_PoC-1_".

## Proof of Concept
### PoC-1
Apply the following patch and run the test `test_t0x1c_maxGulp_poc` to see it pass:
```diff
diff --git a/test/unit/esr/ESR.Gulp.t.sol b/test/unit/esr/ESR.Gulp.t.sol
index 65ee26d..5f4f179 100644
--- a/test/unit/esr/ESR.Gulp.t.sol
+++ b/test/unit/esr/ESR.Gulp.t.sol
@@ -82,4 +82,34 @@ contract ESRGulpTest is ESRTest {
         assertEq(esrSlot.lastInterestUpdate, block.timestamp);
         assertEq(esrSlot.interestSmearEnd, block.timestamp + esr.INTEREST_SMEAR());
     }
+    
+    function test_t0x1c_maxGulp_poc() public {
+        uint256 depositAmount = 100;
+        uint256 interestAmount = type(uint168).max / uint256(5) * 8; // = 598631070650737835296229307480589524851069969602968
+
+        doDeposit(user, depositAmount);
+        asset.mint(address(esr), interestAmount);
+
+        // Step1
+        esr.gulp();
+        EulerSavingsRate.ESRSlot memory esrSlot = esr.getESRSlot();
+        assertEq(esr.totalAssets(), depositAmount);
+        assertEq(esrSlot.interestLeft, type(uint168).max); // this is fine since this interest cap is to allow the vault to function
+
+
+        // Step2
+        skip(esr.INTEREST_SMEAR() / 2);
+        esr.gulp(); 
+        esrSlot = esr.getESRSlot();
+        assertEq(esr.totalAssets(), depositAmount + type(uint168).max / 2); // @audit : this should've been higher i.e. `depositAmount + 299315535325368917648114653740294762425534984801484`. See reason in next comment.
+        assertEq(esrSlot.interestLeft, type(uint168).max); // @audit :  type(uint168).max equals `374144419156711147060143317175368453031918731001855`. This should've been reduced to `299315535325368917648114653740294762425534984801484`. Calculation needs to be: 50% of `type(uint168).max / uint256(5) * 8` which is `598631070650737835296229307480589524851069969602968 / 2 = 299315535325368917648114653740294762425534984801484`, and then overflow check should've been applied. Since this is less than `maxGulp` it should've been considered.
+
+
+        // Step3
+        skip(esr.INTEREST_SMEAR() / 2);
+        esr.gulp(); 
+        esrSlot = esr.getESRSlot();
+        assertEq(esr.totalAssets(), depositAmount + type(uint168).max / 2 + type(uint168).max / 2); // should've been higher
+        assertApproxEqAbs(esrSlot.interestLeft, interestAmount - type(uint168).max, 1); // should've been lower
+    }
 }
```

### PoC-2 (Simplified version using `50` instead of `type(uint168.max)` for maxGulp calculations)
Apply the following patch and run the test `test_t0x1c_maxGulp_simplified` to see it pass:
```diff
diff --git a/src/Synths/EulerSavingsRate.sol b/src/Synths/EulerSavingsRate.sol
index 30cf87b..c5c445d 100644
--- a/src/Synths/EulerSavingsRate.sol
+++ b/src/Synths/EulerSavingsRate.sol
@@ -195,21 +195,21 @@ contract EulerSavingsRate is EVCUtil, ERC4626 {
         super._withdraw(caller, receiver, owner, assets, shares);
     }
 
     /// @notice Smears any donations to this vault as interest.
     function gulp() public nonReentrant {
         ESRSlot memory esrSlotCache = updateInterestAndReturnESRSlotCache();
 
         uint256 assetBalance = IERC20(asset()).balanceOf(address(this));
         uint256 toGulp = assetBalance - _totalAssets - esrSlotCache.interestLeft;
 
-        uint256 maxGulp = type(uint168).max - esrSlotCache.interestLeft;
+        uint256 maxGulp = 50 - esrSlotCache.interestLeft;
         if (toGulp > maxGulp) toGulp = maxGulp; // cap interest, allowing the vault to function
 
         esrSlotCache.interestSmearEnd = uint40(block.timestamp + INTEREST_SMEAR);
         esrSlotCache.interestLeft += uint168(toGulp); // toGulp <= maxGulp <= max uint168
 
         // write esrSlotCache back to storage in a single SSTORE
         esrSlot = esrSlotCache;
     }
 
     /// @notice Updates the interest and returns the ESR storage slot cache.
diff --git a/test/unit/esr/ESR.Gulp.t.sol b/test/unit/esr/ESR.Gulp.t.sol
index 65ee26d..489204e 100644
--- a/test/unit/esr/ESR.Gulp.t.sol
+++ b/test/unit/esr/ESR.Gulp.t.sol
@@ -75,11 +75,41 @@ contract ESRGulpTest is ESRTest {
         asset.mint(address(esr), interestAmount);
         esr.gulp();
 
         EulerSavingsRate.ESRSlot memory esrSlot = esr.getESRSlot();
 
         assertEq(esr.totalAssets(), depositAmount + interestAmount);
         assertEq(esrSlot.interestLeft, interestAmount);
         assertEq(esrSlot.lastInterestUpdate, block.timestamp);
         assertEq(esrSlot.interestSmearEnd, block.timestamp + esr.INTEREST_SMEAR());
     }
+    
+    function test_t0x1c_maxGulp_simplified() public {
+        uint256 depositAmount = 100;
+        uint256 interestAmount = 80; 
+
+        doDeposit(user, depositAmount);
+        asset.mint(address(esr), interestAmount);
+
+        // Step1
+        esr.gulp();
+        EulerSavingsRate.ESRSlot memory esrSlot = esr.getESRSlot();
+        assertEq(esr.totalAssets(), depositAmount);
+        assertEq(esrSlot.interestLeft, 50); // this is fine since this interest cap is to allow the vault to function
+
+
+        // Step2
+        skip(esr.INTEREST_SMEAR() / 2);
+        esr.gulp(); 
+        esrSlot = esr.getESRSlot();
+        assertEq(esr.totalAssets(), depositAmount + 25); // @audit : this should've been higher i.e. `depositAmount + 40`. See reason in next comment.
+        assertEq(esrSlot.interestLeft, 50); // @audit : this should've been reduced to 40. Calculation needs to be on `80` i.e. 80/2 = 40, and then overflow check should've been applied.
+        
+
+        // Step3
+        skip(esr.INTEREST_SMEAR() / 2);
+        esr.gulp(); 
+        esrSlot = esr.getESRSlot();
+        assertEq(esr.totalAssets(), depositAmount + 50); // @audit : should've been higher. This should've been `depositAmount + 40 + 20`
+        assertEq(esrSlot.interestLeft, 30); // @audit : should've been lower. Only 20 should've been left here
+    }
 }
```

## Recommendation
Correct the calculation by considering the entire remaining interest for smearing calculations. Perform overflow checks on this figure to limit to a max value whenever necessary.

[Back to Top](#summaryTable)

---
 
### <a id="lm-001"></a>[LM-001]
## **INVALID ----  INSTEAD SAY THAT gulp() IS GRIEFABLE AND USER NEVER CAN CLAIM ENTIRE INTEREST-------------`public` function `updateInterestAndReturnESRSlotCache()` allows accrued interest to be marked eligible for distribution without pushing `interestSmearEnd` forward by 2 weeks**
#### https://github.com/euler-xyz/euler-vault-kit/blob/master/src/Synths/EulerSavingsRate.sol#L150
<br>

## Impact
Even if protocol fixes the "known" issue of gulp() by whitelisting etc (please refer the other report titled "_gulp() can be griefed to delay interest distribution indefinitely_" for analysis of the `gulp()` issue), this fn can be called by anyone and is identical to gulp() except the fact that smearEnd timeline is not pushed forward, so no "smearing" happens. Hence a separate fix is required for this vulnerability.<br>
Show textual flow steps as mentioned in "audit-info" below.

## Proof of Concept
Add the test inside `test/unit/esr/ESR.Gulp.t.sol`:
```js
    function test_t0x1c_updateInterestAndReturnESRSlotCache() public { 
        uint256 depositAmount = 100e18;
        doDeposit(user, depositAmount);

        uint256 interestAmount = 10e18;
        // Mint interest directly into the contract
        asset.mint(address(esr), interestAmount);
        esr.gulp();
        uint256 smearEnd = block.timestamp + esr.INTEREST_SMEAR();


        EulerSavingsRate.ESRSlot memory esrSlot = esr.getESRSlot();

        assertEq(esr.totalAssets(), depositAmount, "totalAsset mismatch");
        assertEq(esrSlot.interestLeft, interestAmount, "interestLeft mismatch");
        assertEq(esrSlot.lastInterestUpdate, block.timestamp, "lastInterestUpdate mismatch");
        assertEq(esrSlot.interestSmearEnd, smearEnd, "interestSmearEnd mismatch");

        skip(esr.INTEREST_SMEAR() / 2);
        uint256 snapshot = vm.snapshot();


        // ********** SCENARIO - 1 (NORMAL) ************        
        emit log_named_string("************** SCENARIO - 1", "NORMAL      **************");
        esr.gulp();

        esrSlot = esr.getESRSlot();
        assertEq(esr.totalAssets(), depositAmount + interestAmount / 2, "totalAsset mismatch");
        assertEq(esrSlot.interestLeft, interestAmount / 2, "interestLeft != interestAmount / 2");
        assertEq(esrSlot.lastInterestUpdate, block.timestamp, "lastInterestUpdate mismatch");
        assertEq(esrSlot.interestSmearEnd, block.timestamp + esr.INTEREST_SMEAR(), "interestSmearEnd mismatch");
        emit log_named_uint("interestSmearEnd normal          =", esrSlot.interestSmearEnd);



        // ********** SCENARIO - 2 (MANIPULATED) ************    
        emit log_named_string("************** SCENARIO - 2", "MANIPULATED **************");
        vm.revertTo(snapshot);    
        esr.updateInterestAndReturnESRSlotCache(); // @audit : attacker calls this before any call to gulp() is made

        esrSlot = esr.getESRSlot();
        assertEq(esr.totalAssets(), depositAmount + interestAmount / 2, "updateInterest() totalAsset mismatch");
        assertEq(esrSlot.interestLeft, interestAmount / 2, "updateInterest() interestLeft != interestAmount / 2");
        assertEq(esrSlot.lastInterestUpdate, block.timestamp, "updateInterest() lastInterestUpdate mismatch");
        assertEq(esrSlot.interestSmearEnd, smearEnd, "updateInterest() interestSmearEnd mismatch");
        emit log_named_uint("interestSmearEnd manipulated     =", esrSlot.interestSmearEnd);

        
        // @audit-info : the following code is included just to demonstrate for clarity & completeness that 
        // the "known" issue with gulp() still allows an attacker to push interestSmearEnd forward. However,
        // if this call to gulp() is delayed by any amount of time then due to the above call to updateInterestAndReturnESRSlotCache()
        // the attacker would have managed to receive some "smear-free interest" by then.

        esr.gulp(); // some other party calls gulp now
        esrSlot = esr.getESRSlot();
        assertEq(esr.totalAssets(), depositAmount + interestAmount / 2, "gulp() totalAsset mismatch");
        assertEq(esrSlot.interestLeft, interestAmount / 2, "gulp() interestLeft != interestAmount / 2");
        assertEq(esrSlot.lastInterestUpdate, block.timestamp, "gulp() lastInterestUpdate mismatch");
        assertEq(esrSlot.interestSmearEnd, block.timestamp + esr.INTEREST_SMEAR(), "gulp() interestSmearEnd mismatch");
        emit log_named_uint("interestSmearEnd now (known bug) =", esrSlot.interestSmearEnd);
    }
```

## Tools Used
Foundry

## Recommended Mitigation Steps
Remove `public` modifier from `updateInterestAndReturnESRSlotCache()` and mark it `internal`.

[Back to Top](#summaryTable)

---
 
### <a id="m-04"></a>[M-04]
## **maxRepayValue is rounded-down in favour of the liquidator and against the protocol**
#### https://github.com/euler-xyz/euler-vault-kit/blob/cantina-contest/src/EVault/modules/Liquidation.sol#L164
<br>

## Description
When `liquidate()` is called and the collateral is not enough to cover the yield, then the `maxRepayValue` is scaled-down [here on L164](https://github.com/euler-xyz/euler-vault-kit/blob/cantina-contest/src/EVault/modules/Liquidation.sol#L164):
```js
  File: src/EVault/modules/Liquidation.sol

   160:                  // Limit yield to borrower's available collateral, and reduce repay if necessary. This can happen when borrower
   161:                  // has multiple collaterals and seizing all of this one won't bring the violator back to solvency
   162:          
   163:                  if (collateralValue < maxYieldValue) {
   164:@--->                 maxRepayValue = collateralValue * discountFactor / 1e18;
   165:                      maxYieldValue = collateralValue;
   166:                  }
```

This rounded-down `maxRepayValue` is the debt which is taken over by the liquidator in lieu of the `collateralValue`. Hence this should be rounded-up. This is also important because:
- the remaining unpaid debt is socialized amongst other depositors and hence they are effectively being given the rounded-up remaining debt.
- it causes a higher-than-expected devaluation of the token. 
 
Over a period of time with continuous usage of the protocol such cases could pile up to a noticeable value.

## Recommendation
Round up `maxRepayValue` by either using custom logic or a library like solmate with `mulDivUp`:
```diff
  File: src/EVault/modules/Liquidation.sol

   160:                  // Limit yield to borrower's available collateral, and reduce repay if necessary. This can happen when borrower
   161:                  // has multiple collaterals and seizing all of this one won't bring the violator back to solvency
   162:          
   163:                  if (collateralValue < maxYieldValue) {
-  164:                      maxRepayValue = collateralValue * discountFactor / 1e18;
+  164:                      maxRepayValue = collateralValue.mulDivUp(discountFactor, 1e18);
   165:                      maxYieldValue = collateralValue;
   166:                  }
```

[Back to Top](#summaryTable)

---
 
### <a id="m-05"></a>[M-05]
## **`liqCache.repay` is rounded-down against the protocol and in favour of the liquidator**
#### https://github.com/euler-xyz/euler-vault-kit/blob/cantina-contest/src/EVault/modules/Liquidation.sol#L168
<br>

## Description
Inisde the `liquidate()` function, `liqCache.repay` is rounded-down [on L168](https://github.com/euler-xyz/euler-vault-kit/blob/cantina-contest/src/EVault/modules/Liquidation.sol#L168):
```js
  File: src/EVault/modules/Liquidation.sol

   167:          
   168:@--->             liqCache.repay = (maxRepayValue * liqCache.liability.toUint() / liabilityValue).toAssets();
   169:                  liqCache.yieldBalance = maxYieldValue * collateralBalance / collateralValue;
   170:          
   171:                  return liqCache;
```

`liqCache.repay` is the debt which is taken over by the liquidator in exchange for collateral which equals `liqCache.yieldBalance`. Hence this should be rounded-up. 
This is also important because:
- the remaining unpaid debt is socialized amongst other depositors and hence they are effectively being given the rounded-up remaining debt.
- a higher-than-expected devaluation of the token. 

Over a period of time with continuous usage of the protocol such cases could pile up to a noticeable value.

## Recommendation
Round up `liqCache.repay` by either using custom logic or a library like solmate with `mulDivUp`:
```diff
  File: src/EVault/modules/Liquidation.sol

   167:          
-  168:                  liqCache.repay = (maxRepayValue * liqCache.liability.toUint() / liabilityValue).toAssets();
+  168:                  liqCache.repay = (maxRepayValue.mulDivUp(liqCache.liability.toUint(), liabilityValue)).toAssets();
   169:                  liqCache.yieldBalance = maxYieldValue * collateralBalance / collateralValue;
   170:          
   171:                  return liqCache;
```

[Back to Top](#summaryTable)

---
 
### <a id="m-06"></a>[M-06]
## **resolveOracle() can run out of gas in case of multi-nested vaults and cause DoS**
#### https://github.com/euler-xyz/euler-price-oracle/blob/master/src/EulerRouter.sol#L123-L143
<br>

## Description
The [resolveOracle() function](https://github.com/euler-xyz/euler-price-oracle/blob/master/src/EulerRouter.sol#L123-L143) recursively calls itself to resolve the `base` vault & asset. There is no limit to how many nested vaults may have to be traversed here and hence in spite of a valid oracle existing for a base/quote pair, the function may run out of gas when multiple vaults are nested within each other. This can cause the call to revert and cause denial of service for the user.
```js
  File: EulerRouter.sol

  123:              function resolveOracle(uint256 inAmount, address base, address quote)
  124:                  public
  125:                  view
  126:                  returns (uint256, /* resolvedAmount */ address, /* base */ address, /* quote */ address /* oracle */ )
  127:              {
  128:                  // 1. Check the base case.
  129:                  if (base == quote) return (inAmount, base, quote, address(0));
  130:                  // 2. Check if there is a PriceOracle configured for base/quote.
  131:                  address oracle = getConfiguredOracle(base, quote);
  132:                  if (oracle != address(0)) return (inAmount, base, quote, oracle);
  133: @--->            // 3. Recursively resolve `base`.
  134:                  address baseAsset = resolvedVaults[base];
  135:                  if (baseAsset != address(0)) {
  136:                      inAmount = IERC4626(base).convertToAssets(inAmount);
  137: @--->                return resolveOracle(inAmount, baseAsset, quote);
  138:                  }
  139:                  // 4. Return the fallback or revert if not configured.
  140:                  oracle = fallbackOracle;
  141:                  if (oracle == address(0)) revert Errors.PriceOracle_NotSupported(base, quote);
  142:                  return (inAmount, base, quote, oracle);
  143:              }
```

## Recommendation
Implement a max nesting limit so that the user knows it would not be possible to resolve oracle beyond that level of nesting.

[Back to Top](#summaryTable)

---
