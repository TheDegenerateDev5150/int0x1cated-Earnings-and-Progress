# Leaderboard


# Audited Code Repo
### [CodeHawks: DittoETH Audit Contest](https://github.com/Cyfrin/2023-09-ditto/blob/c52c7cec881e0d41f65b057d0df51e97ccad8513)

<br>

# Bugs Filed & Their Status

| # | Bug ID | Name | URL | Adjudged Status |
|--------|--------|------|:------:|-----------------:|
| 1 | [H-01](#h-01)    | Decreasing collateral after increasing it causes permanent loss of yield for the user | [185](https://www.codehawks.com/submissions/clm871gl00001mp081mzjdlwc/185) | .. as high; `Selected Report` |
| 2 | [H-02](#h-02)    | Users can inadvertently delete their short records via `combineShorts()` and lose all collateral | [186](https://www.codehawks.com/submissions/clm871gl00001mp081mzjdlwc/186) |  |
| 3 | [H-03](#h-03)    | Rounding-up of user's `cRatio` causes loss for the protocol | [234](https://www.codehawks.com/submissions/clm871gl00001mp081mzjdlwc/234) |  |
| 4 | [H-04](#h-04)    | Users cannot re-flag a short between the 16th & 17th hour | [301](https://www.codehawks.com/submissions/clm871gl00001mp081mzjdlwc/301) |  |
| 5 | [H-05](#h-05)    | Users can lose yield in `disburseCollateral()` and `_distributeYield()` due to incorrect calculations | [300](https://www.codehawks.com/submissions/clm871gl00001mp081mzjdlwc/300) |  |
| 6 | [H-06](#h-06)    | `_canLiquidate()` calculates first & second liquidation time-windows incorrectly | [299](https://www.codehawks.com/submissions/clm871gl00001mp081mzjdlwc/299) |  |
| 7 | [H-07](#h-07)    | Flag does not reset on favorable price movement | [321](https://www.codehawks.com/submissions/clm871gl00001mp081mzjdlwc/321) |  |
| 8 | [H-08](#h-08)    | The right of `short.flagger` to liquidate short between `firstLiquidationTime` and `secondLiquidationTime` is incorrectly transferred to another flagger | [492](https://www.codehawks.com/submissions/clm871gl00001mp081mzjdlwc/492) |  |
| 9 | [H-09](#h-09)    | Malicious shorter can escape liquidation perpetually | [541](https://www.codehawks.com/submissions/clm871gl00001mp081mzjdlwc/541) |  |
| 10 | [M-01](#m-01)    | Protocol's calculation fails for `twapPrice < 1e6`  | [187](https://www.codehawks.com/submissions/clm871gl00001mp081mzjdlwc/187) |  |
| 11 | [M-02](#m-02)    | `Errors.InvalidTwapPrice()` is never invoked when `if (twapPriceInEther == 0)` is true   | [188](https://www.codehawks.com/submissions/clm871gl00001mp081mzjdlwc/188) |   |
| 12 | [M-03](#m-03)    | Unbounded loop & lack of duplicate-array-element check in external function `distributeYield()` exposes protocol to DoS attack vector | [189](https://www.codehawks.com/submissions/clm871gl00001mp081mzjdlwc/189) |  |
| 13 | [M-04](#m-04)    | `decreaseCollateral()` allows user to decrease collateral by zero `amount` | [190](https://www.codehawks.com/submissions/clm871gl00001mp081mzjdlwc/190) |  |
| 14 | [M-05](#m-05)    | User can increase collateral by zero `amount` inside `increaseCollateral()` | [191](https://www.codehawks.com/submissions/clm871gl00001mp081mzjdlwc/191) |  |
| 15 | [M-06](#m-06)    | Due to precision loss, some of the unbacked asset debt is not socialized | [287](https://www.codehawks.com/submissions/clm871gl00001mp081mzjdlwc/287) |  |
| 16 | [M-07](#m-07)    | DoS attack possible through `liquidateSecondary()` | [288](https://www.codehawks.com/submissions/clm871gl00001mp081mzjdlwc/288) |  |
| 17 | [M-08](#m-08)    | Unchecked `orderHintArray` & `shortHintArray` arrays open the door to DoS attacks | [313](https://www.codehawks.com/submissions/clm871gl00001mp081mzjdlwc/313) |  |
| 18 | [M-09](#m-09)    | Possible DoS on depositEth, withdrawal & unstaking for `BridgeReth` | [364](https://www.codehawks.com/submissions/clm871gl00001mp081mzjdlwc/364) |  |
| 19 | [M-10](#m-10)    | `BridgeReth.sol::unstake()` is unreliable and depends on excess RocketDepositPool balance which can lead to DoS | [367](https://www.codehawks.com/submissions/clm871gl00001mp081mzjdlwc/367) |  |
| 20 | [M-11](#m-11)    | In `_performForcedBid()` during primary margin call, `m.short.ercDebt` can be calculated less than actual, causing higher than expected `ercDebtSocialized` | [390](https://www.codehawks.com/submissions/clm871gl00001mp081mzjdlwc/390) |  |
| 21 | [M-12](#m-12)    | Unsafe typecasting of `uint256` to `uint88` can result in protocol losing high amount in gasFee | [401](https://www.codehawks.com/submissions/clm871gl00001mp081mzjdlwc/401) |  |
| 22 | [M-13](#m-13)    | The check inside `createVault()` for presence of already created vaults is insufficient | [408](https://www.codehawks.com/submissions/clm871gl00001mp081mzjdlwc/408) |  |
| 23 | [M-14](#m-14)    | `updateYield()` will permanently stop updating yield due to unsafe typecasting of `uint256` to `uint88` | [413](https://www.codehawks.com/submissions/clm871gl00001mp081mzjdlwc/413) |  |
| 24 | [L-01](#l-01)    | Users get less than expected `amtZeth` (ethEscrowed) on calling `redeemErc()` | [339](https://www.codehawks.com/submissions/clm871gl00001mp081mzjdlwc/339) |  |


<br>

## **HIGH-SEVERITY BUGS**
---

### <a id="h-01"></a>[H-01]
## **Decreasing collateral after increasing it causes permanent loss of yield for the user**
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibShortRecord.sol#L353-L359
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/ShortRecordFacet.sol#L38
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/ShortRecordFacet.sol#L82
<br>

## Summary
Decreasing collateral after increasing it causes permanent loss of yield for the user if done within a timespan of `Constants.YIELD_DELAY_HOURS` with `cRatio` at a healthy level.<br>
Similar loss of yield is observed if user tries exiting a short using `exitShort..()` functions, instead of decreasing the collateral.

## Vulnerability Details & Impacts
Developer has provided the following comment inside [disburseCollateral()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibShortRecord.sol#L349-L352):
```js
File: contracts/libraries/LibShortRecord.sol

348               if (yield > 0) {
349 @>                /*
350 @>                @dev If somebody exits a short, gets margin called, decreases their collateral before YIELD_DELAY_HOURS duration is up,
351 @>                they lose their yield to the TAPP
352 @>                */
353                   bool isNotRecentlyModified =
354                       LibOrders.getOffsetTimeHours() - updatedAt > Constants.YIELD_DELAY_HOURS;
355                   if (isNotRecentlyModified) {
356                       s.vaultUser[vault][shorter].ethEscrowed += yield;
357                   } else {
358                       s.vaultUser[vault][address(this)].ethEscrowed += yield;
359                   }
360               }
```
However, the _unintended side effect_ of this is visible in the following flow of events:<br>
-  Step 1: User (Shorter / Receiver) has a fully-filled short record and hence is eligible for a `yield` after certain duration of time. We skip this time-duration (1 hour in this case or `Constants.YIELD_DELAY_HOURS`). His `cRatio >= LibAsset.primaryLiquidationCR(asset)` is `true` at this moment.
-  Step 2: User calls [increaseCollateral()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/ShortRecordFacet.sol#L38) with any amount. The `cRatio >= LibAsset.primaryLiquidationCR(asset)` remains true, and this resets `updatedAt` via a call to [short.resetFlag()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/ShortRecordFacet.sol#L59-L61) and hence `isNotRecentlyModified` will evaluate to `false` in Line 353 above if checked within a duration of `YIELD_DELAY_HOURS`.
-  Step 3: User calls [decreaseCollateral()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/ShortRecordFacet.sol#L82) with any eligible amount before `YIELD_DELAY_HOURS` duration is up.
-  Step 3 ***variation-1*** : User calls [exitShortWallet()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/ExitShortFacet.sol#L69) for a full-exit before `YIELD_DELAY_HOURS` duration is up.
-  Step 3 ***variation-2*** : User calls [exitShortErcEscrowed()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/ExitShortFacet.sol#L122) for a full-exit before `YIELD_DELAY_HOURS` duration is up.
-  Step 3 ***variation-3*** : User calls `exitShort()` for an exit ([partial](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/ExitShortFacet.sol#L236) or [full](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/ExitShortFacet.sol#L221)) before `YIELD_DELAY_HOURS` duration is up.<br>
***Result:*** Any `yield` he was eligible for, goes to TAPP and he receives nothing.<br><br>

I also reference the [docs](https://dittoeth.com/technical/yield#functions) which give the reason for such a logic:

> disburseCollateral is an internal function that is called whenever collateral "leaves" a shortRecord. This is similar to distributeYield but pertains to a particular shortRecord and distributes yield only for the amount of collateral that is being decreased. This way unrealized yield that was not distributed is captured before permanent loss. 
 >> YIELD_DELAY_HOURS is also utilized here to prevent flash loan exploits that quickly make shorts and exit them. When this time condition is not satisfied, the distributed yield is sent to the TAPP instead of the shortRecord owner.

<br>

Also [here](https://dittoeth.com/technical/yield#zeth-yield):

> Eligibility Criteria:
>
>     - Only applies to shortRecord.
>     - shortRecord created/modified more than YIELD_DELAY_HOURS ago.
>         - Upon creation, shortRecord starts accruing yield, but it's only eligible to claim after waiting YIELD_DELAY_HOURS.


The intention of the protocol is to cancel the yield (transfer it to TAPP) if it has been accrued within the last hour. However, any yield rightfully accrued from previously present collateral (which could have been sitting for last 5 hours), can be claimed by the user. This is violated as shown in the following PoC.

## PoC
Add the following tests inside `test/Yield.t.sol`. Also, uncomment [Line 11](https://github.com/Cyfrin/2023-09-ditto/blob/main/test/Yield.t.sol#L11) so that we can do some logging.
```js
    /* solhint-disable no-console */
    function test_normal_decreaseCollateral() public {
        // initial ethEscrowed
        console.log(
            "initial receiver ethEsrowed=",
            diamond.getVaultUserStruct(vault, receiver).ethEscrowed
        ); // 1000
        console.log(
            "initial TAPP ethEsrowed=",
            diamond.getVaultUserStruct(vault, tapp).ethEscrowed
        ); // 0

        fundLimitShort(DEFAULT_PRICE, DEFAULT_AMOUNT, receiver);
        fundLimitBid(DEFAULT_PRICE, DEFAULT_AMOUNT, sender);

        skip(yieldEligibleTime);
        generateYield(2 ether);

        vm.startPrank(receiver);
        diamond.decreaseCollateral(asset, Constants.SHORT_STARTING_ID, 1 ether);
        vm.stopPrank();

        console.log(
            "final receiver ethEsrowed=",
            diamond.getVaultUserStruct(vault, receiver).ethEscrowed
        ); // 1001.3 ether (this is 1000 + 1 + yield = 1000 + 1 + 0.3)
        console.log(
            "final TAPP ethEsrowed=", diamond.getVaultUserStruct(vault, tapp).ethEscrowed
        ); // 0.2
    }

    /* solhint-disable no-console */
    function test_bug_incThenDecreaseCollateral() public {
        // initial ethEscrowed
        uint256 initial_ethEscrowed =
            diamond.getVaultUserStruct(vault, receiver).ethEscrowed;
        console.log("initial receiver ethEsrowed=", initial_ethEscrowed); // 1000
        console.log(
            "initial TAPP ethEsrowed=",
            diamond.getVaultUserStruct(vault, tapp).ethEscrowed
        ); // 0

        fundLimitShort(DEFAULT_PRICE, DEFAULT_AMOUNT, receiver);
        fundLimitBid(DEFAULT_PRICE, DEFAULT_AMOUNT, sender);

        skip(yieldEligibleTime);
        generateYield(2 ether);

        vm.startPrank(receiver);
        uint88 increaseBy = 1; // can be any amount, but let's keep as 1 wei for easy calculation.
        diamond.increaseCollateral(asset, Constants.SHORT_STARTING_ID, increaseBy);
        uint88 decreaseBy = 1 ether;
        diamond.decreaseCollateral(asset, Constants.SHORT_STARTING_ID, decreaseBy);
        vm.stopPrank();

        console.log(
            "final receiver ethEsrowed=",
            diamond.getVaultUserStruct(vault, receiver).ethEscrowed
        ); // 1000999999999999999999 (initial_ethEscrowed + decreaseBy - 1 wei = 1000 ether + 1 ether - 1 wei)
        console.log(
            "final TAPP ethEsrowed=", diamond.getVaultUserStruct(vault, tapp).ethEscrowed
        ); // 500000000000000000 (expected_tapp.ethEscrowed + yield = 0.2 + 0.3 = 0.5 ether)
        assertGt(
            diamond.getVaultUserStruct(vault, receiver).ethEscrowed,
            1000999999999999999999 // initial_ethEscrowed + decreaseBy - 1 wei = 1000 ether + 1 ether - 1 wei = 1000999999999999999999
        );
    }
```
Let's run the "normal" scenario first, where the user has some accumulated yield and see the actual figures. These would be our expected values. Run `forge test --mt test_normal_decreaseCollateral -vv` to get the output:
```
Logs:
  initial receiver ethEsrowed= 1000000000000000000000
  initial TAPP ethEsrowed= 0
  final receiver ethEsrowed= 1001300000000000000000
  final TAPP ethEsrowed= 200000000000000000
```
The yield credited to the user here is `final_ethEsrowed - initial_ethEsrowed - collateralWithdrawn = 1001300000000000000000 - 1000000000000000000000 - 1000000000000000000 = 300000000000000000 = 0.3 ether`.<br><br>
Now, run `forge test --mt test_bug_incThenDecreaseCollateral -vv` to get the output:
```
Logs:
  initial receiver ethEsrowed= 1000000000000000000000
  initial TAPP ethEsrowed= 0
  final receiver ethEsrowed= 1000999999999999999999
  final TAPP ethEsrowed= 500000000000000000
  Error: a > b not satisfied [uint]
    Value a: 1000999999999999999999
    Value b: 1000999999999999999999
```
User receives no yield as his final ethEsrowed remains at `initial_ethEsrowed + collateralWithdrawn - collateralAdded = 1000000000000000000000 + 1000000000000000000 - 1 = 1000999999999999999999 = 1000.999999999999999999 ether`. <br>
All the yield goes to TAPP: `yield_deposited_in_tapp = 500000000000000000 - 200000000000000000 (from the first test) = 300000000000000000 = 0.3 ether --> which equals the yield user had received in the first test`.

## Tools Used
Manual review, forge.

## Recommendations
If `cRatio >= LibAsset.primaryLiquidationCR(asset)` is already `true` when user calls `increaseCollateral()`, then we need a variable to track the yield (or collateral) which the user is so far eligible to receive. Doing this will protect the protocol from giving the user any yield accrued spontaneously from the newly added collateral (via flash-loan or otherwise) and at the same time protect the old accrued yield user rightfully should receive while calling decreaseCollateral().

---

### <a id="h-02"></a>[H-02]
## **Users can inadvertently delete their short records via `combineShorts()` and lose all collateral**
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/ShortRecordFacet.sol#L117
<br>

## Summary
Bug in the logic of [combineShorts()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/ShortRecordFacet.sol#L117) can easily cause users to delete one or **all** of their short records. This causes their collateral and ercDebt to become 0 (they lose all their collateral). <br>

## Vulnerability Details
The happy path of `combineShorts()` expects a user to call it in a manner resembling the following (_Note: `Constants.SHORT_STARTING_ID` has a value 2_): 
```js
uint8[] memory shortIds_to_combine = new uint8[](3);
shortIds_to_combine[0] = Constants.SHORT_STARTING_ID + 2; // id: 4
shortIds_to_combine[1] = Constants.SHORT_STARTING_ID + 1; // id: 3
shortIds_to_combine[2] = Constants.SHORT_STARTING_ID; // id: 2

vm.prank(receiver);
diamond.combineShorts(_cusd, shortIds_to_combine);
```
This merges short records with ids 2 and 3 into id 4. Short record ids 2 and 3 are not needed any more, so they are deleted.

<br>
Even if a user provides an array with duplicate elements in index 1 to n, it doesn't break the system as the protocol rightly reverts with an `InvalidShortId()` error. So, the following will revert:

```js
uint8[] memory shortIds_to_combine = new uint8[](3);
shortIds_to_combine[0] = Constants.SHORT_STARTING_ID + 2;
shortIds_to_combine[1] = Constants.SHORT_STARTING_ID + 1;
shortIds_to_combine[2] = Constants.SHORT_STARTING_ID + 1; // @audit-info : id can not be the same as above

vm.prank(receiver);
diamond.combineShorts(_cusd, shortIds_to_combine);
```
<br>

However, if a user provides the same short record id in `shortIds_to_combine[n]` as in `shortIds_to_combine[0]`, then **all** the short records provided in the array are deleted, even the one they merged into. For example:

```js
uint8[] memory shortIds_to_combine = new uint8[](4);
shortIds_to_combine[0] = Constants.SHORT_STARTING_ID + 2;
shortIds_to_combine[1] = Constants.SHORT_STARTING_ID + 1;
shortIds_to_combine[2] = Constants.SHORT_STARTING_ID; 
shortIds_to_combine[3] = Constants.SHORT_STARTING_ID + 2; // @audit-issue : since this id is the same as the one provided at index 0, this will cause short records with ids 2, 3, 4 to be deleted altogether.

vm.prank(receiver);
diamond.combineShorts(_cusd, shortIds_to_combine);
```
**Since the short record does not exist anymore, this causes its collateral and ercDebt to become 0 (or non-existent) causing loss of funds for the user.**

<br>

## PoC
Add the following code inside `test/fork/EndToEndFork.t.sol` and run it through `forge test --mt testFork_shortRecordCanBeDeletedViaCombineShorts -vv`. Remember to uncomment [L11](https://github.com/Cyfrin/2023-09-ditto/blob/main/test/fork/EndToEndFork.t.sol#L11) to enable logging:
```js
    /* solhint-disable no-console */
    function testFork_shortRecordCanBeDeletedViaCombineShorts() public {
        uint16 cusdInitialMargin = diamond.getAssetStruct(_cusd).initialMargin; // 200%

        // Workflow: Bridge - DepositEth
        // sender
        vm.startPrank(sender);
        assertEq(reth.balanceOf(_bridgeReth), 0);
        diamond.depositEth{value: 500 ether}(_bridgeReth);
        vm.stopPrank();
        // receiver
        vm.startPrank(receiver);
        assertEq(steth.balanceOf(_bridgeSteth), 0);
        diamond.depositEth{value: 500 ether}(_bridgeSteth);

        // Workflow: LimitShort
        MTypes.OrderHint[] memory orderHints = new MTypes.OrderHint[](1);
        currentPrice = diamond.getOraclePriceT(_cusd); // ~ 0.0005 ether
        diamond.createLimitShort(
            _cusd, currentPrice, 100_000 ether, orderHints, shortHints, cusdInitialMargin
        );
        vm.stopPrank();

        // Workflow: LimitBid
        // sender
        shortHints[0] = diamond.getShortIdAtOracle(_cusd);
        vm.prank(sender);
        diamond.createBid(
            _cusd, currentPrice, 100_000 ether, false, orderHints, shortHints
        );
        // @audit-info : short record with id: 2 would have been created here. Let's fetch it.
        STypes.ShortRecord memory receiverShort =
            diamond.getShortRecords(_cusd, receiver)[0];
        assertEq(receiverShort.ercDebt, 100_000 ether);
        assertSR(SR.FullyFilled, receiverShort.status);

        console.log(
            "total receiver collateral here",
            diamond.getShortRecords(_cusd, receiver)[0].collateral
        ); // ~150 ether

        vm.startPrank(receiver);
        // removing 50 ether as minimum collateral required to be maintained is 100 ether
        diamond.decreaseCollateral(_cusd, 2, 50 ether);

        console.log("=================== BEFORE combine ==========================");
        console.log(
            "receiver collateral", diamond.getShortRecords(_cusd, receiver)[0].collateral
        );
        console.log(
            "Number of short records", (diamond.getShortRecords(_cusd, receiver)).length
        );
        console.log("=========================================================");

        // @audit-info : let's combine(delete) the short
        uint8[] memory shortIds_to_combine = new uint8[](2);
        shortIds_to_combine[0] = Constants.SHORT_STARTING_ID;
        // @audit-issue : This is the root cause of the bug. Keeping last index same as the first one deletes the whole merged short record
        shortIds_to_combine[1] = Constants.SHORT_STARTING_ID;
        diamond.combineShorts(_cusd, shortIds_to_combine); // @audit : This will "combine" and also delete the short record id 2
        console.log("=================== AFTER combine ==========================");
        console.log(
            "receiver shortRecords count after buggy combine/deletion",
            (diamond.getShortRecords(_cusd, receiver)).length
        ); // 0
        console.log(
            "receiver collateral all gone from shortRecord diamond.getShortRecords(_cusd, receiver)[0].collateral since shortRecord is deleted."
        );
        vm.stopPrank();
        console.log("=========================================================");

        // Ideally, combineShorts() should have resulted in 1 final shortRecord being present, but
        // that is not the case.
        assertGt(
            (diamond.getShortRecords(_cusd, receiver)).length,
            0,
            "All Short Records Deleted"
        );
    }
```
The output is the following:
```
Logs:
  total receiver collateral here 150549361698162000000
  =================== BEFORE combine ==========================
  receiver collateral 100549361698162000000
  Number of short records 1
  =========================================================
  =================== AFTER combine ==========================
  receiver shortRecords count after buggy combine/deletion 0
  receiver collateral all gone from shortRecord diamond.getShortRecords(_cusd, receiver)[0].collateral since shortRecord is deleted.
  =========================================================
  Error: All Short Records Deleted
  Error: a > b not satisfied [uint]
    Value a: 0
    Value b: 0
```

## Impact
- **User loses all of her collateral**. If we assume this to be an honest mistake by the user where she passes the same id twice, then the penalty she has to pay for this is huge - losing all her collateral across all the short records she passed in the array.
- Direct deletion of short record possible without going through the flow of liquidation/exit first.

## Tools Used
Manual review, forge test.

## Recommendations
Inside `combineShorts()` add a check that the last id in the array `ids` is not the same as that at the first index.
```diff
File: contracts/facets/ShortRecordFacet.sol

117           function combineShorts(address asset, uint8[] memory ids)
118               external
119               isNotFrozen(asset)
120               nonReentrant
121               onlyValidShortRecord(asset, msg.sender, ids[0])
122           {
123               if (ids.length < 2) revert Errors.InsufficientNumberOfShorts();
+                 require(ids[ids.length - 1] != ids[0]);
124               // First short in the array
125               STypes.ShortRecord storage firstShort = s.shortRecords[asset][msg.sender][ids[0]];
126               // @dev Load initial short elements in struct to avoid stack too deep
127               MTypes.CombineShorts memory c;
128               c.shortFlagExists = firstShort.flaggerId != 0;
129               c.shortUpdatedAt = firstShort.updatedAt;
130
131               address _asset = asset;
132               uint88 collateral;
133               uint88 ercDebt;
134               uint256 yield;
135               uint256 ercDebtSocialized;
136               for (uint256 i = ids.length - 1; i > 0; i--) {
137                   uint8 _id = ids[i];
138                   _onlyValidShortRecord(_asset, msg.sender, _id);
139                   STypes.ShortRecord storage currentShort =
140                       s.shortRecords[_asset][msg.sender][_id];
141                   // See if there is at least one flagged short
142                   if (!c.shortFlagExists) {
143                       if (currentShort.flaggerId != 0) {
144                           c.shortFlagExists = true;
145                       }
146                   }
147
148                   //@dev Take latest time when combining shorts (prevent flash loan)
149                   if (currentShort.updatedAt > c.shortUpdatedAt) {
150                       c.shortUpdatedAt = currentShort.updatedAt;
151                   }
152
153                   {
154                       uint88 currentShortCollateral = currentShort.collateral;
155                       uint88 currentShortErcDebt = currentShort.ercDebt;
156                       collateral += currentShortCollateral;
157                       ercDebt += currentShortErcDebt;
158                       yield += currentShortCollateral.mul(currentShort.zethYieldRate);
159                       ercDebtSocialized += currentShortErcDebt.mul(currentShort.ercDebtRate);
160                   }
161
162                   if (currentShort.tokenId != 0) {
163                       //@dev First short needs to have NFT so there isn't a need to burn and re-mint
164                       if (firstShort.tokenId == 0) {
165                           revert Errors.FirstShortMustBeNFT();
166                       }
167
168                       LibShortRecord.burnNFT(currentShort.tokenId);
169                   }
170
171                   // Cancel this short and combine with short in ids[0]
172                   LibShortRecord.deleteShortRecord(_asset, msg.sender, _id);
173               }
174
175               // Merge all short records into the short at position id[0]
176               firstShort.merge(ercDebt, ercDebtSocialized, collateral, yield, c.shortUpdatedAt);
177
178               // If at least one short was flagged, ensure resulting c-ratio > primaryLiquidationCR
179               if (c.shortFlagExists) {
180                   if (
181                       firstShort.getCollateralRatioSpotPrice(
182                           LibOracle.getSavedOrSpotOraclePrice(_asset)
183                       ) < LibAsset.primaryLiquidationCR(_asset)
184                   ) revert Errors.InsufficientCollateral();
185                   // Resulting combined short has sufficient c-ratio to remove flag
186                   firstShort.resetFlag();
187               }
188               emit Events.CombineShorts(asset, msg.sender, ids);
189           }
```

---

### <a id="h-03"></a>[H-03]
## **Rounding-up of user's `cRatio` causes loss for the protocol**
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibShortRecord.sol#L34
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallPrimaryFacet.sol#L54
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/YieldFacet.sol#L108
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallSecondaryFacet.sol#L144
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallSecondaryFacet.sol#L53-L54
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/ShortRecordFacet.sol#L181-L182
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibOrders.sol#L847

<br>

## Summary
At multiple places in the code, user's collateral ratio has been calculated in a manner which causes loss of precision (rounding-up) due to **division before multiplication**. This causes potential loss for the DittoETH protocol, among other problems.

## Root Cause
Use of the following piece of code causes rounding-up:<br>
- Style 1
```js
uint256 cRatio = short.getCollateralRatioSpotPrice(LibOracle.getSavedOrSpotOraclePrice(asset));
```
- Style 2
```js
uint256 oraclePrice = LibOracle.getOraclePrice(asset); // or uint256 oraclePrice = LibOracle.getSavedOrSpotOraclePrice(asset); // or uint256 oraclePrice = LibOracle.getPrice(asset);
 ...
 ...
 ...
uint256 cRatio = short.getCollateralRatioSpotPrice(oraclePrice);
```

<br>

## Vulnerability Details
Let's break the issue down into 4 smaller parts:<br>
### PART 1:
Let us first look inside [getOraclePrice()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibOracle.sol#L20):
```js
  File: contracts/libraries/LibOracle.sol

  20            function getOraclePrice(address asset) internal view returns (uint256) {
  21                AppStorage storage s = appStorage();
  22                AggregatorV3Interface baseOracle = AggregatorV3Interface(s.baseOracle);
  23                uint256 protocolPrice = getPrice(asset);
  24                // prettier-ignore
  25                (
  26                    uint80 baseRoundID,
  27                    int256 basePrice,
  28                    /*uint256 baseStartedAt*/
  29                    ,
  30                    uint256 baseTimeStamp,
  31                    /*uint80 baseAnsweredInRound*/
  32                ) = baseOracle.latestRoundData();
  33
  34                AggregatorV3Interface oracle = AggregatorV3Interface(s.asset[asset].oracle);
  35                if (address(oracle) == address(0)) revert Errors.InvalidAsset();
  36
  37  @>            if (oracle == baseOracle) {
  38                    //@dev multiply base oracle by 10**10 to give it 18 decimals of precision
  39                    uint256 basePriceInEth = basePrice > 0
  40  @>                    ? uint256(basePrice * Constants.BASE_ORACLE_DECIMALS).inv()
  41                        : 0;
  42                    basePriceInEth = baseOracleCircuitBreaker(
  43                        protocolPrice, baseRoundID, basePrice, baseTimeStamp, basePriceInEth
  44                    );
  45  @>                return basePriceInEth;
  46                } else {
  47                    // prettier-ignore
  48                    (
  49                        uint80 roundID,
  50                        int256 price,
  51                        /*uint256 startedAt*/
  52                        ,
  53                        uint256 timeStamp,
  54                        /*uint80 answeredInRound*/
  55                    ) = oracle.latestRoundData();
  56  @>                uint256 priceInEth = uint256(price).div(uint256(basePrice));
  57                    oracleCircuitBreaker(
  58                        roundID, baseRoundID, price, basePrice, timeStamp, baseTimeStamp
  59                    );
  60  @>                return priceInEth;
  61                }
  62            }
```
Based on whether the `oracle` is `baseOracle` or not, the function returns either `basePriceEth` or `priceInEth`.<br>
- `basePriceEth` can be `uint256(basePrice * Constants.BASE_ORACLE_DECIMALS).inv()` which is basically `1e36 / (basePrice * Constants.BASE_ORACLE_DECIMALS)` or simply written, of the form `oracleN / oracleD` where `oracleN` is the numerator with value 1e36 (as defined [here](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/PRBMathHelper.sol#L27)) and `oracleD` is the denominator.<br>
- `priceInEth` is given as `uint256 priceInEth = uint256(price).div(uint256(basePrice))` which again is of the form `oracleN / oracleD`.

<br>

### PART 2:
[getSavedOrSpotOraclePrice()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibOracle.sol#L153) too internally calls the above `getOraclePrice()` function, if it has been equal to or more than 15 minutes since the last time `LibOrders.getOffsetTime()` was set:
```js
  File: contracts/libraries/LibOracle.sol

  153           function getSavedOrSpotOraclePrice(address asset) internal view returns (uint256) {
  154               if (LibOrders.getOffsetTime() - getTime(asset) < 15 minutes) {
  155                   return getPrice(asset);
  156               } else {
  157 @>                return getOraclePrice(asset);
  158               }
  159           }
```
<br>

### PART 3:
[getCollateralRatioSpotPrice()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibShortRecord.sol#L30) calculates `cRatio` as:
```js
  File: contracts/libraries/LibShortRecord.sol

  30            function getCollateralRatioSpotPrice(
  31                STypes.ShortRecord memory short,
  32                uint256 oraclePrice
  33            ) internal pure returns (uint256 cRatio) {
  34  @>            return short.collateral.div(short.ercDebt.mul(oraclePrice));
  35            }
```
<br>

### PART 4 (FINAL PART):
There are multiple places in the code (mentioned below under ***Impacts*** section) which compare the user's `cRatio` to `initialCR` or `LibAsset.primaryLiquidationCR(_asset)` in the following manner:
```js
if (short.getCollateralRatioSpotPrice(LibOracle.getSavedOrSpotOraclePrice(asset)) < LibAsset.primaryLiquidationCR(asset))
```
<br>

Calling `short.getCollateralRatioSpotPrice(LibOracle.getSavedOrSpotOraclePrice(asset))` means the value returned from it would be:
```diff
      // @audit-issue : Potential precision loss. Division before multiplication should not be done.
      shortCollateral / (shortErcDebt * (oracleN / oracleD))           // return short.collateral.div(short.ercDebt.mul(oraclePrice));
```
which has the potential for precision loss (rounding-up) due to division before multiplication. The correct style ought to be:
```diff
+  @>    (shortCollateral * oracleD) / (shortErcDebt * oracleN)
```
<br>

## Impacts
- Impact #1: User gets more `dittoYieldShares` than expected due to [code logic](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/YieldFacet.sol#L108):
```js
  File: contracts/facets/YieldFacet.sol

  76            function _distributeYield(address asset)
  77                private
  78                onlyValidAsset(asset)
  79                returns (uint88 yield, uint256 dittoYieldShares)
  80            {
  81                uint256 vault = s.asset[asset].vault;
  82                // Last updated zethYieldRate for this vault
  83                uint80 zethYieldRate = s.vault[vault].zethYieldRate;
  84                // Protocol time
  85                uint256 timestamp = LibOrders.getOffsetTimeHours();
  86                // Last saved oracle price
  87  @>            uint256 oraclePrice = LibOracle.getPrice(asset);
  88                // CR of shortRecord collateralized at initialMargin for this asset
  89                uint256 initialCR = LibAsset.initialMargin(asset) + 1 ether;
  90                // Retrieve first non-HEAD short
  91                uint8 id = s.shortRecords[asset][msg.sender][Constants.HEAD].nextId;
  92                // Loop through all shorter's shorts of this asset
  93                while (true) {
  94                    // One short of one shorter in this market
  95                    STypes.ShortRecord storage short = s.shortRecords[asset][msg.sender][id];
  96                    // To prevent flash loans or loans where they want to deposit to claim yield immediately
  97                    bool isNotRecentlyModified =
  98                        timestamp - short.updatedAt > Constants.YIELD_DELAY_HOURS;
  99                    // Check for cancelled short
  100                   if (short.status != SR.Cancelled && isNotRecentlyModified) {
  101                       uint88 shortYield =
  102                           short.collateral.mulU88(zethYieldRate - short.zethYieldRate);
  103                       // Yield earned by this short
  104                       yield += shortYield;
  105                       // Update zethYieldRate for this short
  106                       short.zethYieldRate = zethYieldRate;
  107                       // Calculate CR to modify ditto rewards
  108 @>                    uint256 cRatio = short.getCollateralRatioSpotPrice(oraclePrice);
  109 @>                    if (cRatio <= initialCR) {
  110 @>                        dittoYieldShares += shortYield;
  111                       } else {
  112                           // Reduce amount of yield credited for ditto rewards proportional to CR
  113                           dittoYieldShares += shortYield.mul(initialCR).div(cRatio);
  114                       }
  115                   }
  116                   // Move to next short unless this is the last one
  117                   if (short.nextId > Constants.HEAD) {
  118                       id = short.nextId;
  119                   } else {
  120                       break;
  121                   }
  122               }
  123           }
```
This rounding-up can lead to user's `cRatio` to be considered as `>initialCR` even when it's slightly lower. This results in greater `dittoYieldShares` being calculated.
<br>

- Impact #2: [Short can escape being flagged](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallPrimaryFacet.sol#L54-L55) by a Flagger even when actual cRatio is slightly less than `primaryLiquidationCR`:
```js
  File: contracts/facets/MarginCallPrimaryFacet.sol

  43            function flagShort(address asset, address shorter, uint8 id, uint16 flaggerHint)
  44                external
  45                isNotFrozen(asset)
  46                nonReentrant
  47                onlyValidShortRecord(asset, shorter, id)
  48            {
  49                if (msg.sender == shorter) revert Errors.CannotFlagSelf();
  50                STypes.ShortRecord storage short = s.shortRecords[asset][shorter][id];
  51                short.updateErcDebt(asset);
  52
  53                if (
  54  @>                short.getCollateralRatioSpotPrice(LibOracle.getSavedOrSpotOraclePrice(asset))
  55  @>                    >= LibAsset.primaryLiquidationCR(asset)      // @audit-issue : this will evaluate to `true` due to rounding-up and the short will not be eligible for flagging
  56                ) {
  57                    revert Errors.SufficientCollateral();
  58                }
  59
  60                uint256 adjustedTimestamp = LibOrders.getOffsetTimeHours();
  61
  62                // check if already flagged
  63                if (short.flaggerId != 0) {
  64                    uint256 timeDiff = adjustedTimestamp - short.updatedAt;
  65                    uint256 resetLiquidationTime = LibAsset.resetLiquidationTime(asset);
  66
  67                    if (timeDiff <= resetLiquidationTime) {
  68                        revert Errors.MarginCallAlreadyFlagged();
  69                    }
  70                }
  71
  72                short.setFlagger(cusd, flaggerHint);
  73                emit Events.FlagShort(asset, shorter, id, msg.sender, adjustedTimestamp);
  74            }
```
<br>

- Impact #3: `m.cRatio` of a `MTypes.MarginCallSecondary m` is [set after incorrect rounding-up](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallSecondaryFacet.sol#L144). Note that this happens when someone calls [liquidateSecondary()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallSecondaryFacet.sol#L53-L54)
```js
  File: contracts/facets/MarginCallSecondaryFacet.sol

  38            function liquidateSecondary(
  39                address asset,
  40                MTypes.BatchMC[] memory batches,
  41                uint88 liquidateAmount,
  42                bool isWallet
  43            ) external onlyValidAsset(asset) isNotFrozen(asset) nonReentrant {
  44                STypes.AssetUser storage AssetUser = s.assetUser[asset][msg.sender];
  45                MTypes.MarginCallSecondary memory m;
  46                uint256 minimumCR = LibAsset.minimumCR(asset);
  47                uint256 oraclePrice = LibOracle.getSavedOrSpotOraclePrice(asset);
  48                uint256 secondaryLiquidationCR = LibAsset.secondaryLiquidationCR(asset);
  49
  50                uint88 liquidatorCollateral;
  51                uint88 liquidateAmountLeft = liquidateAmount;
  52                for (uint256 i; i < batches.length;) {
  53  @>                m = _setMarginCallStruct(
  54                        asset, batches[i].shorter, batches[i].shortId, minimumCR, oraclePrice
  55                    );
  56

                ......
                ......
                ......


  129           function _setMarginCallStruct(
  130               address asset,
  131               address shorter,
  132               uint8 id,
  133               uint256 minimumCR,
  134               uint256 oraclePrice
  135           ) private returns (MTypes.MarginCallSecondary memory) {
  136               LibShortRecord.updateErcDebt(asset, shorter, id);
  137
  138               MTypes.MarginCallSecondary memory m;
  139               m.asset = asset;
  140               m.short = s.shortRecords[asset][shorter][id];
  141               m.vault = s.asset[asset].vault;
  142               m.shorter = shorter;
  143               m.minimumCR = minimumCR;
  144 @>            m.cRatio = m.short.getCollateralRatioSpotPrice(oraclePrice);
  145               return m;
  146           }

```
<br>

- Impact #4: While combining shorts, say `short1` and `short2`, if short2 was flagged by a flagger, then the protocol checks if [the combined new short has a cRatio above primaryLiquidationCR or not](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/ShortRecordFacet.sol#L178-L186). The rounding-up allows the shorter to combine shorts even if he is just below the required c-ratio.
```js
  File: contracts/facets/ShortRecordFacet.sol

  117           function combineShorts(address asset, uint8[] memory ids)
  118               external
  119               isNotFrozen(asset)
  120               nonReentrant
  121               onlyValidShortRecord(asset, msg.sender, ids[0])
  122           {
  123               if (ids.length < 2) revert Errors.InsufficientNumberOfShorts();
  124               // First short in the array
  125               STypes.ShortRecord storage firstShort = s.shortRecords[asset][msg.sender][ids[0]];
  
                ......
                ......
                ......

  174
  175               // Merge all short records into the short at position id[0]
  176               firstShort.merge(ercDebt, ercDebtSocialized, collateral, yield, c.shortUpdatedAt);
  177
  178               // If at least one short was flagged, ensure resulting c-ratio > primaryLiquidationCR
  179               if (c.shortFlagExists) {
  180                   if (
  181 @>                    firstShort.getCollateralRatioSpotPrice(
  182 @>                        LibOracle.getSavedOrSpotOraclePrice(_asset)
  183 @>                    ) < LibAsset.primaryLiquidationCR(_asset)
  184                   ) revert Errors.InsufficientCollateral();
  185                   // Resulting combined short has sufficient c-ratio to remove flag
  186                   firstShort.resetFlag();
  187               }
  188               emit Events.CombineShorts(asset, msg.sender, ids);
  189           }
```

<br>

***NOTE:*** 
<br>
While the operation done in [this piece of code](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibOrders.sol#L847) is a bit different from the above analysis, I am clubbing it with this bug report as the underlying issue is the same (and the resolution would be similar): _Multiplication and division operations should not be done directly on top of fetched oracle price, without paying attention to new order of evaluation:_

```js
  File: contracts/libraries/LibOrders.sol

  812           function _updateOracleAndStartingShort(address asset, uint16[] memory shortHintArray)
  813               private
  814           {
  815               AppStorage storage s = appStorage();
  815               uint256 oraclePrice = LibOracle.getOraclePrice(asset);
  
                ......
                ......
                ......

  845                       //@dev: force hint to be within 1% of oracleprice
  846                       bool startingShortWithinOracleRange = shortPrice
  847 @>                        <= oraclePrice.mul(1.01 ether)                     // @audit-issue : division before multiplication
  848                           && s.shorts[asset][prevId].price >= oraclePrice;
  
                ......
                ......
                ......

  866           }
```

<br>

The effective calculation being done above is:

```js
    (oracleN / oracleD) * (1.01 ether)        // division before multiplication
```
<br>

Which should have been:

```js
    (oracleN * 1.01 ether) / oracleD
```
<br>

Similar multiplication or division operations have been done on `price` at various places throughout the code, which can be clubbed under this root cause itself.
<br>

## PoC

_Have attempted to keep all values in close proximity to the ones present in forked mainnet tests._<br><br>

Let's assume some values for numerator & denominator and other variables:
```js
    uint256 private short_collateral = 100361729669569000000; // ~ 100 ether
    uint256 private short_ercDebt = 100000000000000000000000; // 100_000 ether
    uint256 private price = 99995505; // oracleN
    uint256 private basePrice = 199270190598; // oracleD
    uint256 private primaryLiquidationCR = 2000000000000000000; // 2 ether (as on forked mainnet)

// For this example, we assume that oracle != baseOracle, so that the below calculation would be done by the protocol
So calculated priceInEth = price.div(basePrice) = 501808648347845  // ~ 0.0005 ether
```
<br>

Let's calculate for the scenario of `flagShort()` where the [code logic](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallPrimaryFacet.sol#L54-L55) says:
```js
  53                if (
  54  @>                short.getCollateralRatioSpotPrice(LibOracle.getSavedOrSpotOraclePrice(asset))
  55  @>                    >= LibAsset.primaryLiquidationCR(asset)      // @audit-issue : this will evaluate to `true`, then revert, due to rounding-up and the short will incorrectly escape flagging
  56                ) {
  57                    revert Errors.SufficientCollateral();
  58                }
```
<br>

Create a file named `test/IncorrectCRatioCheck.t.sol` and paste the following code in it. Some mock functions are included here which mirror protocol's calculation style:
```js
// SPDX-License-Identifier: GPL-3.0-only
pragma solidity 0.8.21;

import {U256} from "contracts/libraries/PRBMathHelper.sol";
import {OBFixture} from "test/utils/OBFixture.sol";
import {console} from "contracts/libraries/console.sol";

contract IncorrectCRatioCheck is OBFixture {
    using U256 for uint256;

    uint256 private short_collateral = 85307470219133700000; // ~ 85.3 ether
    uint256 private short_ercDebt = 100000000000000000000000; // 100_000 ether
    uint256 private price = 99995505; // oracleN
    uint256 private basePrice = 199270190598; // (as on forked mainnet)  // oracleD
    uint256 private primaryLiquidationCR = 1700000000000000000; // 1.7 ether (as on forked mainnet)

    function _getSavedOrSpotOraclePrice() internal view returns (uint256) {
        uint256 priceInEth = price.div(basePrice);
        return priceInEth; // will return 501808648347845 =~ 0.0005 ether  // (as on forked mainnet)
    }

    function getCollateralRatioSpotPrice_IncorrectStyle_As_In_Existing_DittoProtocol(
        uint256 oraclePrice
    ) internal view returns (uint256) {
        return short_collateral.div(short_ercDebt.mul(oraclePrice));
    }

    function getCollateralRatioSpotPrice_CorrectStyle(uint256 oracleN, uint256 oracleD)
        internal
        view
        returns (uint256)
    {
        return (short_collateral.mul(oracleD)).div(short_ercDebt.mul(oracleN));
    }

    /* solhint-disable no-console */
    function test_GetCollateralRatioSpotPrice_IncorrectStyle_As_In_Existing_DittoProtocol(
    ) public view {
        uint256 cRatio =
        getCollateralRatioSpotPrice_IncorrectStyle_As_In_Existing_DittoProtocol(
            _getSavedOrSpotOraclePrice()
        );
        console.log("cRatio calculated (existing style) =", cRatio);
        if (cRatio >= primaryLiquidationCR) {
            console.log("Errors.SufficientCollateral; can not be flagged");
        } else {
            console.log("InsufficientCollateral; can be flagged");
        }
    }

    /* solhint-disable no-console */
    function test_GetCollateralRatioSpotPrice_CorrectStyle() public view {
        uint256 cRatio = getCollateralRatioSpotPrice_CorrectStyle(price, basePrice);
        console.log("cRatio calculated (correct style) =", cRatio);
        if (cRatio >= primaryLiquidationCR) {
            console.log("Errors.SufficientCollateral; can not be flagged");
        } else {
            console.log("InsufficientCollateral; can be flagged");
        }
    }
}
``` 
<br>

- First, let's see the output as per protocol's calculation. Run `forge test --mt test_GetCollateralRatioSpotPrice_IncorrectStyle_As_In_Existing_DittoProtocol -vv`:
```js
Logs:
  cRatio calculated (existing style) = 1700000000000000996
  Errors.SufficientCollateral; can not be flagged
```
So the short can not be flagged as `cRatio > primaryLiquidationCR` of 1700000000000000000.
<br>
<br>

- Now, let's see the output as per the correct calculation. Run `forge test --mt test_GetCollateralRatioSpotPrice_CorrectStyle -vv`:
```js
Logs:
  cRatio calculated (correct style) = 1699999999999899995
  InsufficientCollateral; can be flagged
```
Short's cRatio is actually below primaryLiquidationCR. Should have been flagged ideally.

<br>

## Tools Used
Manual review, forge test.

## Recommendations
These steps need to be taken to fix the issue. Developer may have to make some additional changes since `.mul`, `.div`, etc are being used from the `PRBMathHelper.sol` library. Following is the general workflow required:<br>
- Create additional functions to fetch oracle parameters instead of price: Create copies of `getOraclePrice()` and `getSavedOrSpotOraclePrice()`, but these ones return `oracleN` & `oracleD` instead of the calculated price. Let's assume the new names to be `getOraclePriceParams()` and `getSavedOrSpotOraclePriceParams()`.
- Create a new function to calculate cRatio which will be used in place of the above occurences of `getCollateralRatioSpotPrice()`: 
```js
    function getCollateralRatioSpotPriceFromOracleParams(
        STypes.ShortRecord memory short,
        uint256 oracleN,
        uint256 oracleD
    ) internal pure returns (uint256 cRatio) {
        return (short.collateral.mul(oracleD)).div(short.ercDebt.mul(oracleN));
    }
```

<br>

- For fixing the last issue of `oraclePrice.mul(1.01 ether)` on [L847](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibOrders.sol#L847), first call `getOraclePriceParams()` to get `oracleN` & `oracleD` and then:
```diff
  845                       //@dev: force hint to be within 1% of oracleprice
  846                       bool startingShortWithinOracleRange = shortPrice
- 847                           <= oraclePrice.mul(1.01 ether)
+ 847                           <= (oracleN.mul(1.01 ether)).div(oracleD)
  848                           && s.shorts[asset][prevId].price >= oraclePrice;
``` 

---

### <a id="h-04"></a>[H-04]
## **Users cannot re-flag a short between the 16th & 17th hour**
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallPrimaryFacet.sol#L60
<br>

## Summary
As per [docs](https://dittoeth.com/dao/parameters#orderbook-parameters) and [code comments](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallPrimaryFacet.sol#L345) one can re-flag a short once 16 hours have passed since it was first flagged but not yet liquidated.
Due to a logic error in [flagShort()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallPrimaryFacet.sol#L60), users have to wait till the completion of 17 hours, to be able to flag such a short.

## Vulnerability Details & Impact
[L60](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallPrimaryFacet.sol#L60) calculates `adjustedTimestamp` as:
```js
  File: contracts/facets/MarginCallPrimaryFacet.sol

  60 @>            uint256 adjustedTimestamp = LibOrders.getOffsetTimeHours();
```
which is then compared to `resetLiquidationTime` on [L64-L67](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallPrimaryFacet.sol#L64-L67):
```js
  File: contracts/facets/MarginCallPrimaryFacet.sol

  64               uint256 timeDiff = adjustedTimestamp - short.updatedAt;
  65               uint256 resetLiquidationTime = LibAsset.resetLiquidationTime(asset);
  66
  67 @>            if (timeDiff <= resetLiquidationTime) {
  68                   revert Errors.MarginCallAlreadyFlagged();
  69               }
```

`LibOrders.getOffsetTimeHours()` [returns](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibOrders.sol#L29-L32):
```js
  File: contracts/libraries/LibOrders.sol

  30            function getOffsetTimeHours() internal view returns (uint24 timeInHours) {
  31  @>            return uint24(getOffsetTime() / 1 hours);
  32            }
```
Any timestamp is rounded-down to the hour. So `16 hours 30 minutes` would be considered as `16 hours` and hence not greater than resetLiquidationTime. <br>
Important to note that even the `updatedAt` value of an order is being set using `LibOrders.getOffsetTimeHours()` instead of using an exact timestamp, as can be seen [here](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibShortRecord.sol#L101). <br>
Let's see the problem this poses with an example:
```
Assume that the "starting time" or contract deployment time is at midnight i.e 00:00 .
Suppose short gets updated at timestamp: 5:05 . Protocol will store this in rounded-down form as `updatedAt` = 5:00

Suppose the protocol allows a user to exit (or flag, or some action) this short after `X` hours. Let X = 2. 
So the user should ideally be able to exit after 7:05.

However, when he tries, at 7:06, his `adjustedTimestamp` is calculated (rounded-down) as 7:00. The `timeDiff` = 7:00 - 5:00 = 2 hours. 
This is less than equal to X (or `resetLiquidationTime` or some deadline) i.e `timeDiff <= X`, so he won't be allowed to perform the action. 

The user will have to wait another 54 minutes till 8:00 so that timeDiff is 3 hours.
```

## Impact
Users cannot re-flag a short between the 16th & 17th hour. Shorters gets an extra hour to correct their cRatio.

## PoC
Create a file inside `test/` folder by the name of `IncorrectReFlag.t.sol` and paste the following code. It has the test along with a few helper functions:
```js
// SPDX-License-Identifier: GPL-3.0-only
pragma solidity 0.8.21;

import {Constants} from "contracts/libraries/Constants.sol";
import {Errors} from "contracts/libraries/Errors.sol";
import {U256, U88} from "contracts/libraries/PRBMathHelper.sol";
import {OBFixture} from "test/utils/OBFixture.sol";
import {STypes, SR} from "contracts/libraries/DataTypes.sol";

contract IncorrectReFlag is OBFixture {
    using U256 for uint256;
    using U88 for uint88;

    function _initialAssertStruct() public {
        r.ethEscrowed = 0;
        r.ercEscrowed = DEFAULT_AMOUNT;
        assertStruct(receiver, r);
        s.ethEscrowed = 0;
        s.ercEscrowed = 0;
        assertStruct(sender, s);
        e.ethEscrowed = 0;
        e.ercEscrowed = 0;
        assertStruct(extra, e);
        t.ethEscrowed = 0;
        t.ercEscrowed = 0;
        assertStruct(tapp, t);
    }

    function _prepareAsk(uint80 askPrice, uint88 askAmount) public {
        fundLimitBidOpt(DEFAULT_PRICE, DEFAULT_AMOUNT, receiver);
        fundLimitShortOpt(DEFAULT_PRICE, DEFAULT_AMOUNT, sender);

        fundLimitAskOpt(askPrice, askAmount, receiver);
        assertEq(getShortRecordCount(sender), 1);
        assertEq(
            getShortRecord(sender, Constants.SHORT_STARTING_ID).collateral,
            DEFAULT_AMOUNT.mulU88(DEFAULT_PRICE) * 6
        );
        _initialAssertStruct();
    }

    function _checkFlaggerAndUpdatedAt(
        address _shorter,
        uint16 _flaggerId,
        uint256 _updatedAt
    ) public {
        STypes.ShortRecord memory shortRecord =
            getShortRecord(_shorter, Constants.SHORT_STARTING_ID);

        assertEq(shortRecord.flaggerId, _flaggerId, "flagger id mismatch");
        assertEq(shortRecord.updatedAt, _updatedAt, "updatedAt mismatch");
    }

    function _flagShortAndSkipTime(uint256 timeToSkip, address flagger) public {
        _prepareAsk({askPrice: DEFAULT_PRICE, askAmount: DEFAULT_AMOUNT});
        _setETH(2666 ether);
        _checkFlaggerAndUpdatedAt({_shorter: sender, _flaggerId: 0, _updatedAt: 0});
        vm.prank(flagger);
        diamond.flagShort(asset, sender, Constants.SHORT_STARTING_ID, Constants.HEAD);
        uint256 flagged = diamond.getOffsetTimeHours();
        skipTimeAndSetEth({skipTime: timeToSkip, ethPrice: 2666 ether});
        _checkFlaggerAndUpdatedAt({_shorter: sender, _flaggerId: 1, _updatedAt: flagged});
        assertSR(
            getShortRecord(sender, Constants.SHORT_STARTING_ID).status, SR.FullyFilled
        );
        depositEth(tapp, DEFAULT_TAPP);
    }

    function test_CantflagShortBetween16To17Hours() public {
        address _flagger_01 = makeAddr("_flagger_01");
        address _flagger_02 = makeAddr("_flagger_02");
        _flagShortAndSkipTime({timeToSkip: 16 hours + 30 minutes, flagger: _flagger_01});

        // Case 1: Incorrectly reverts even though
        // 16 hours have passed; anyone should be have been able to flag it again
        vm.prank(_flagger_02);
        vm.expectRevert(Errors.MarginCallAlreadyFlagged.selector);
        diamond.flagShort(asset, sender, Constants.SHORT_STARTING_ID, Constants.HEAD);
        rewind(16 hours + 30 minutes);

        // Case 2: Allows flagging, but only after 17 hours
        skip(17 hours);
        vm.prank(_flagger_02);
        diamond.flagShort(asset, sender, Constants.SHORT_STARTING_ID, Constants.HEAD);
    }
}
```

Run `forge test --mt test_CantflagShortBetween16To17Hours -vv` to verify the impact. 
<br>The test passes, but should have failed as per protocol specifications.

## Tools Used
Manual review, foundry.

## Recommendations
Instead of doing these calculations in `hours`, they should be done in `seconds` to avoid rounding error. 

---

### <a id="h-05"></a>[H-05]
## **Users can lose yield in `disburseCollateral()` and `_distributeYield()` due to incorrect calculations**
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibShortRecord.sol#L353-L359
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/YieldFacet.sol#L97-L100
<br>

## Summary
- [disburseCollateral()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibShortRecord.sol#L331) distributes yield to the user if `isNotRecentlyModified` is true i.e short has not been updated in the last 1 hour (`Constants.YIELD_DELAY_HOURS`). 
- [_distributeYield()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/YieldFacet.sol#L76) distributes yield to the user if `isNotRecentlyModified` is true i.e short has not been updated in the last 1 hour (`Constants.YIELD_DELAY_HOURS`). <br>
<br>

Due to the bug in the code logic where it uses `LibOrders.getOffsetTimeHours()` for checking timestamp, users will lose their rightfully deserved yield.
<br>

I have clubbed both these issues into this single bug report as they both involve comparison with Constants.YIELD_DELAY_HOURS to distribute yield: `bool isNotRecentlyModified = LibOrders.getOffsetTimeHours() - updatedAt > Constants.YIELD_DELAY_HOURS;`

## Vulnerability Details & Impact
In `disburseCollateral()`, variable `isNotRecentlyModified` is [calculated as](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibShortRecord.sol#L353-L359):
```js
  File: contracts/libraries/LibShortRecord.sol

  349                   /*
  350                   @dev If somebody exits a short, gets margin called, decreases their collateral before YIELD_DELAY_HOURS duration is up,
  351                   they lose their yield to the TAPP
  352                   */
  353 @>                bool isNotRecentlyModified =
  354 @>                    LibOrders.getOffsetTimeHours() - updatedAt > Constants.YIELD_DELAY_HOURS;
  355                   if (isNotRecentlyModified) {
  356                       s.vaultUser[vault][shorter].ethEscrowed += yield;
  357                   } else {
  358                       s.vaultUser[vault][address(this)].ethEscrowed += yield;
  359                   }
```

In `_distributeYield()`, [similar pattern](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/YieldFacet.sol#L97-L106):
```js
  File: contracts/facets/YieldFacet.sol

  85  @>            uint256 timestamp = LibOrders.getOffsetTimeHours();
  86                // Last saved oracle price
  87                uint256 oraclePrice = LibOracle.getPrice(asset);
  88                // CR of shortRecord collateralized at initialMargin for this asset
  89                uint256 initialCR = LibAsset.initialMargin(asset) + 1 ether;
  90                // Retrieve first non-HEAD short
  91                uint8 id = s.shortRecords[asset][msg.sender][Constants.HEAD].nextId;
  92                // Loop through all shorter's shorts of this asset
  93                while (true) {
  94                    // One short of one shorter in this market
  95                    STypes.ShortRecord storage short = s.shortRecords[asset][msg.sender][id];
  96                    // To prevent flash loans or loans where they want to deposit to claim yield immediately
  97  @>                bool isNotRecentlyModified =
  98  @>                    timestamp - short.updatedAt > Constants.YIELD_DELAY_HOURS;
  99                    // Check for cancelled short
  100                   if (short.status != SR.Cancelled && isNotRecentlyModified) {
  101                       uint88 shortYield =
  102                           short.collateral.mulU88(zethYieldRate - short.zethYieldRate);
  103                       // Yield earned by this short
  104                       yield += shortYield;
  105                       // Update zethYieldRate for this short
  106                       short.zethYieldRate = zethYieldRate;
```

The function `LibOrders.getOffsetTimeHours()` [returns](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibOrders.sol#L29-L32):
```js
  File: contracts/libraries/LibOrders.sol

  30            function getOffsetTimeHours() internal view returns (uint24 timeInHours) {
  31  @>            return uint24(getOffsetTime() / 1 hours);
  32            }
```
Any timestamp is rounded-down to the hour. So `1 hour 30 minutes` would be considered as `1 hour` and hence not greater than `Constants.YIELD_DELAY_HOURS` which is equal to 1 hour in the protocol currently. <br>
Important to note that even the `updatedAt` value of an order is being set using `LibOrders.getOffsetTimeHours()` instead of using an exact timestamp, as can be seen [here](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibShortRecord.sol#L101). <br>
Let's see the problem this poses with an example:
```
Assume that the "starting time" or contract deployment time is at midnight i.e 00:00 .
Suppose short gets updated at timestamp: 5:05 . Protocol will store this in rounded-down form as `updatedAt` = 5:00

Suppose the protocol allows an action on this short after `X` hours. Here, X = 1. 
So the user should ideally be able to perform the action after 6:05.

However, when he tries, at 6:06, his current time stamp is calculated (rounded-down) as 6:00. The `timeDiff` = 6:00 - 5:00 = 1 hour. 
This is less than equal to X (Constants.YIELD_DELAY_HOURS in this case) i.e `timeDiff <= X`, so he won't be allowed to perform the action. 

The user will have to wait another 54 minutes till 7:00 so that timeDiff is 2 hours.
```
<br>

`isNotRecentlyModified` remains false even if user has crossed the 1 hour mark, but is still within the 2 hour mark.

## Impact
In the above scenario, user will lose the yield they rightfully deserved.

## PoC-1
To test `disburseCollateral()` (which is internally called when external user calls `decreaseCollateral()`), paste the following code inside `test/Yield.t.sol` and run it via `forge test --mt test_disburseCollateralFailsEvenAfter1Hour30Minutes -vv`.
```js
    function test_disburseCollateralFailsEvenAfter1Hour30Minutes() public {
        setUpShortAndCheckInitialEscrowed();

        vm.prank(owner);
        diamond.setTithe(vault, 0);
        generateYield(10 ether);
        skip(90 minutes);
        vm.prank(sender);
        decreaseCollateral(Constants.SHORT_STARTING_ID, 1 wei);

        bool userReceivedCollateral =
            diamond.getVaultUserStruct(vault, sender).ethEscrowed > (1000 ether + 1 wei);
        bool nothingCreditedToTAPP =
            diamond.getVaultUserStruct(vault, tapp).ethEscrowed == 0;

        assertTrue(
            userReceivedCollateral && nothingCreditedToTAPP,
            "User did not receive yield & was taken by TAPP"
        );
    }
```
<br>

Output:
```
Logs:
  Error: User did not receive yield & was taken by TAPP
  Error: Assertion Failed
```
<br>

The protocol should have credited yield to the user after 90 minutes (1 hour 30 minutes), but did not and hence the test failed.<br>
One can change the `skip` value to 120 minutes or 2 hours in the above test and see that it would pass. Basically the user is having to wait an extra hour (which he would have no way of knowing in advance), else he loses his yield to TAPP.

## PoC-2
To test `_distributeYield()` (which is internally called when external user calls `distributeYield()`), paste the following code inside `test/Yield.t.sol` and run it via `forge test --mt test_distributeYieldFailsEvenAfter1Hour59Minutes -vv`.
```js
    function test_distributeYieldFailsEvenAfter1Hour59Minutes() public {
        setUpShortAndCheckInitialEscrowed();
        vm.startPrank(sender);
        generateYield(1 ether);

        address[] memory assets = new address[](1);
        assets[0] = asset;

        skip(1 hours + 59 minutes);
        diamond.distributeYield(assets); // should pass, but incorrectly reverts
    }
```
<br>

Output:
```
Running 1 test for test/Yield.t.sol:YieldTest
[FAIL. Reason: NoYield()] test_distributeYieldFailsEvenAfter1Hour59Minutes() (gas: 994959)
Test result: FAILED. 0 passed; 1 failed; 0 skipped; finished in 154.83ms
```
<br>

The test should have passed as the user is past the 1 hour mark, but it reverts & fails. One can change the `skip` value to 2 hours in the above test and see that it would pass. 

## Tools Used
Manual review, foundry.

## Recommendations
Instead of doing these calculations in `hours`, they should be done in `seconds` to avoid rounding error. 

---

### <a id="h-06"></a>[H-06]
## **`_canLiquidate()` calculates first & second liquidation time-windows incorrectly**
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallPrimaryFacet.sol#L341-L407
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallPrimaryFacet.sol#L118
<br>

## Summary
As per [docs](https://dittoeth.com/technical/margincall#primary-liquidation) liquidation timelines are:
```
Assuming flagShort() was successful,

    - firstLiquidationTime is 10hrs, secondLiquidationTime is 12hrs, resetLiquidationTime is 16hrs (these are modifiable)
    - Between updatedAt and firstLiquidationTime, liquidate() isn't callable.
    - Between firstLiquidationTime and secondLiquidationTime, liquidate() is only callable by short.flagger.
    - Between secondLiquidationTime and resetLiquidationTime, liquidate() is callable by anyone.
    - Past resetLiquidationTime, liquidate() isn't callable.
```
The first timeline of `firstLiquidationTime is 10hrs, secondLiquidationTime is 12hrs` is not properly implemented due to logic error in [_canLiquidate()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallPrimaryFacet.sol#L341-L407). This function is internally called whenever an external user calls [liquidate()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallPrimaryFacet.sol#L118).


## Vulnerability Details & Impact
[L384](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallPrimaryFacet.sol#L384) calculates `timeDiff` as:
```js
  File: contracts/facets/MarginCallPrimaryFacet.sol

  384 @>            uint256 timeDiff = LibOrders.getOffsetTimeHours() - m.short.updatedAt;
```

`LibOrders.getOffsetTimeHours()` [returns](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibOrders.sol#L29-L32):
```js
  File: contracts/libraries/LibOrders.sol

  30            function getOffsetTimeHours() internal view returns (uint24 timeInHours) {
  31  @>            return uint24(getOffsetTime() / 1 hours);
  32            }
```
Any timestamp is rounded-down to the hour. So `10 hours 1 minute` would be considered as 10 hours. <br>
Important to note that even the `updatedAt` value of an order is being set using `LibOrders.getOffsetTimeHours()` instead of using an exact timestamp, as can be seen [here](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibShortRecord.sol#L101). <br>

Let's see the problem this poses with an example:
```
Assume that the "starting time" or contract deployment time is at midnight i.e 00:00 .
Suppose short gets updated at timestamp: 5:01 . Protocol will store this in rounded-down form as `updatedAt` = 5:00

Suppose the protocol allows a user to exit (or flag, or some action) this short after `X` hours. Let X = 2. 
So the user should ideally be able to exit after 7:01.

However, when he tries, at 7:02, even this timestamp is recorded using `LibOrders.getOffsetTimeHours()` (rounded-down) and stored as 7:00. The `timeDiff` = 7:00 - 5:00 = 2 hours. 
This is less than equal to X (or `firstLiquidationTime` or some deadline) i.e `timeDiff <= X`, so he won't be allowed to perform the action. 

The user will have to wait another 58 minutes till 8:00 so that timeDiff is 3 hours.
```
<br>

The relevant piece of code inside `_canLiquidate()` is [here](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallPrimaryFacet.sol#L384-L406):
```js
  File: contracts/facets/MarginCallPrimaryFacet.sol

  384 @>            uint256 timeDiff = LibOrders.getOffsetTimeHours() - m.short.updatedAt;
  385               uint256 resetLiquidationTime = LibAsset.resetLiquidationTime(m.asset);
  386
  387               if (timeDiff >= resetLiquidationTime) {
  388                   return false;
  389               } else {
  390                   uint256 secondLiquidationTime = LibAsset.secondLiquidationTime(m.asset);
  391 @>                bool isBetweenFirstAndSecondLiquidationTime = timeDiff
  392 @>                    > LibAsset.firstLiquidationTime(m.asset) && timeDiff <= secondLiquidationTime
  393                       && s.flagMapping[m.short.flaggerId] == msg.sender;
  394 @>                bool isBetweenSecondAndResetLiquidationTime =
  395 @>                    timeDiff > secondLiquidationTime && timeDiff <= resetLiquidationTime;
  396                   if (
  397                       !(
  398                           (isBetweenFirstAndSecondLiquidationTime)
  399                               || (isBetweenSecondAndResetLiquidationTime)
  400                       )
  401                   ) {
  402                       revert Errors.MarginCallIneligibleWindow();
  403                   }
  404
  405                   return true;
  406               }
```
<br>

Hence, the impact is the following (PoCs follow in the next section):
- **Impact #1**: The flagger who had flagged the short and waited for 10 hours to liquidate it can not do so in the first hour i.e. between 10-11th hour. This **also** means the shorter has ***11 hours*** instead of 10 hours after being flagged, to correct his cRatio.
- **Impact #2**: Anyone (flagger or non-flagger) should be allowed to liquidate between 12-16th hours as per docs. However, a non-flagger is not able to do so between 12-13th hour. Effectively, non-flaggers get only ***3 hours*** instead of the expected 4 hours to liquidate a flagged short because at the stroke of the 16th hour, short's flag is reset (This condition for now, [works correctly](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallPrimaryFacet.sol#L387-L389) in `_canLiquidate()`. This works correctly because `resetLiquidationTime` is currently taken to be a full hour with no minutes i.e 16 hours. Had it been 16 hours 30 minutes, we could have seen a bug here too).

## PoC
Create a file inside `test/` folder by the name of `IncorrectCanLiquidate.t.sol` and paste the following code. It has 2 tests along with a few helper functions:
```js
// SPDX-License-Identifier: GPL-3.0-only
pragma solidity 0.8.21;

import {Constants} from "contracts/libraries/Constants.sol";
import {Errors} from "contracts/libraries/Errors.sol";
import {U256, U88} from "contracts/libraries/PRBMathHelper.sol";
import {OBFixture} from "test/utils/OBFixture.sol";
import {STypes, SR} from "contracts/libraries/DataTypes.sol";

contract IncorrectCanLiquidate is OBFixture {
    using U256 for uint256;
    using U88 for uint88;

    function _initialAssertStruct() public {
        r.ethEscrowed = 0;
        r.ercEscrowed = DEFAULT_AMOUNT;
        assertStruct(receiver, r);
        s.ethEscrowed = 0;
        s.ercEscrowed = 0;
        assertStruct(sender, s);
        e.ethEscrowed = 0;
        e.ercEscrowed = 0;
        assertStruct(extra, e);
        t.ethEscrowed = 0;
        t.ercEscrowed = 0;
        assertStruct(tapp, t);
    }

    function _prepareAsk(uint80 askPrice, uint88 askAmount) public {
        fundLimitBidOpt(DEFAULT_PRICE, DEFAULT_AMOUNT, receiver);
        fundLimitShortOpt(DEFAULT_PRICE, DEFAULT_AMOUNT, sender);

        fundLimitAskOpt(askPrice, askAmount, receiver);
        assertEq(getShortRecordCount(sender), 1);
        assertEq(
            getShortRecord(sender, Constants.SHORT_STARTING_ID).collateral,
            DEFAULT_AMOUNT.mulU88(DEFAULT_PRICE) * 6
        );
        _initialAssertStruct();
    }

    function _checkFlaggerAndUpdatedAt(
        address _shorter,
        uint16 _flaggerId,
        uint256 _updatedAt
    ) public {
        STypes.ShortRecord memory shortRecord =
            getShortRecord(_shorter, Constants.SHORT_STARTING_ID);

        assertEq(shortRecord.flaggerId, _flaggerId, "flagger id mismatch");
        assertEq(shortRecord.updatedAt, _updatedAt, "updatedAt mismatch");
    }

    function _flagShortAndSkipTime(uint256 timeToSkip, address flagger) public {
        _prepareAsk({askPrice: DEFAULT_PRICE, askAmount: DEFAULT_AMOUNT});
        _setETH(2666 ether);
        _checkFlaggerAndUpdatedAt({_shorter: sender, _flaggerId: 0, _updatedAt: 0});
        vm.prank(flagger);
        diamond.flagShort(asset, sender, Constants.SHORT_STARTING_ID, Constants.HEAD);
        uint256 flagged = diamond.getOffsetTimeHours();
        skipTimeAndSetEth({skipTime: timeToSkip, ethPrice: 2666 ether});
        _checkFlaggerAndUpdatedAt({_shorter: sender, _flaggerId: 1, _updatedAt: flagged});
        assertSR(
            getShortRecord(sender, Constants.SHORT_STARTING_ID).status, SR.FullyFilled
        );
        depositEth(tapp, DEFAULT_TAPP);
    }

    function test_01_FlaggerCanNotLiquidateBetween10And11Hours() public {
        address _flagger = makeAddr("_flagger_");

        // Case 1: incorrectly reverts
        _flagShortAndSkipTime({timeToSkip: 10 hours + 1 seconds, flagger: _flagger});
        vm.expectRevert(Errors.MarginCallIneligibleWindow.selector);
        vm.prank(_flagger);
        diamond.liquidate(
            asset, sender, Constants.SHORT_STARTING_ID, shortHintArrayStorage
        );
        rewind(10 hours + 1 seconds);

        // Case 2: incorrectly reverts
        skip(10 hours + 59 minutes);
        vm.expectRevert(Errors.MarginCallIneligibleWindow.selector);
        vm.prank(_flagger);
        diamond.liquidate(
            asset, sender, Constants.SHORT_STARTING_ID, shortHintArrayStorage
        );
        rewind(10 hours + 59 minutes);

        // Case 3: liquidation allowed only after an "extra" hour has passed
        skip(10 hours + 1 hours);
        vm.prank(_flagger);
        diamond.liquidate(
            asset, sender, Constants.SHORT_STARTING_ID, shortHintArrayStorage
        );
    }

    function test_02_NonFlaggerCanNotLiquidateBetween12And13Hours() public {
        address _flagger = makeAddr("_flagger_");
        address _nonFlagger = makeAddr("_non_flagger_");

        // Case 1: incorrectly reverts
        _flagShortAndSkipTime({timeToSkip: 12 hours + 1 seconds, flagger: _flagger});
        vm.expectRevert(Errors.MarginCallIneligibleWindow.selector);
        vm.prank(_nonFlagger);
        diamond.liquidate(
            asset, sender, Constants.SHORT_STARTING_ID, shortHintArrayStorage
        );
        rewind(12 hours + 1 seconds);

        // Case 2: incorrectly reverts
        skip(12 hours + 59 minutes);
        vm.expectRevert(Errors.MarginCallIneligibleWindow.selector);
        vm.prank(_nonFlagger);
        diamond.liquidate(
            asset, sender, Constants.SHORT_STARTING_ID, shortHintArrayStorage
        );
        rewind(12 hours + 59 minutes);

        // Case 3: liquidation allowed only after an "extra" hour has passed
        skip(12 hours + 1 hours);
        vm.prank(_nonFlagger);
        diamond.liquidate(
            asset, sender, Constants.SHORT_STARTING_ID, shortHintArrayStorage
        );
    }
}
```

- Run `forge test --mt test_01_FlaggerCanNotLiquidateBetween10And11Hours -vv` to verify Impact#1. 
- Run `forge test --mt test_02_NonFlaggerCanNotLiquidateBetween12And13Hours -vv` to verify Impact#2.
<br>Both tests pass, but should have failed as per protocol specifications.

## Tools Used
Manual review, foundry.

## Recommendations
Instead of doing these calculations in `hours`, they should be done in `seconds` to avoid rounding error. 

## Informational note to the development team
I can see [pre-existing tests](https://github.com/Cyfrin/2023-09-ditto/blob/main/test/MarginCallPrimary.t.sol#L1187) which could have spotted this issue as the comment provided next to these lines is `//10hrs 1 second`:
```js
  File: test/MarginCallPrimary.t.sol

  1187 @>            skipTimeAndSetEth(TEN_HRS_PLUS, 730 ether); //10hrs 1 second
```
However, the constant `TEN_HRS_PLUS` is defined as:
```js
  File: test/utils/ConstantsTest.sol

  15  @>        uint256 public constant TEN_HRS_PLUS = 10 hours + 1 hours;
  16            uint256 public constant TWELVE_HRS_PLUS = 12 hours + 1 hours;
  17            uint256 public constant SIXTEEN_HRS_PLUS = 16 hours + 1 hours;
```
Maybe you meant it to be `uint256 public constant TEN_HRS_PLUS = 10 hours + 1 seconds;`? Same for the other constants too?<br>
Or maybe you already encountered this issue as is evident by the [comment here](https://github.com/Cyfrin/2023-09-ditto/blob/main/test/invariants/Handler.sol#L786)?<br>
Pointing this out as you may be able to spot more bugs after correcting this.

## Additional note on duplicativeness
A note to the judges, in case it helps move things faster: I have raised two other bugs with the following titles-
- **Bug-A**: Users cannot re-flag a short between the 16th & 17th hour
- **Bug-B**: Users can lose yield in `disburseCollateral()` and `_distributeYield()` due to incorrect calculations
<br>

Bug-A is inside a different function, `flagShort()`, and the fix would need to be applied there. The impact is different too, which relates to the inability of users to re-flag a short.<br>
Bug-B is also different in the sense that it is inside 2 different functions `disburseCollateral()` & `_distributeYield()`, and the impact is related to loss of yield.<br>
Their similarity with the current bug is that they happen due to incorrect placement of `LibOrders.getOffsetTimeHours()`. Raised them separate from this one because they seem like totally different/unique business cases too. Leaving it up to you for further analysis.

---

### <a id="h-07"></a>[H-07]
## **Flag does not reset on favorable price movement**
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallPrimaryFacet.sol#L343
<br>

## Summary
As per [docs](https://dittoeth.com/litepaper#primary-margin-call-method) :
## "
**Primary Margin Call Method**<br>
In the Primary Margin Call Method, when an account is below 200% and is flagged for liquidation, a liquidation timer of 10 hours begins on the account to give a window of opportunity to the flagged shorter to fix their CR and stop the liquidation countdown. <br>
To halt the liquidation timer and remove the flag, the shorter must reach the target maintenance margin CR (200%) again ***by either the price moving favorably*** or by adding additional collateral.
## "
Basically the flag is **expected to be reset** by the price moving favorably.<br>
This behaviour is not followed by the protocol.

## Vulnerability Details
Imagine the following flow of events:
- #1. A short is flagged by a `flagger` as its cRatio < 4
- #2. `Shorter` now has 10 hours. If cRatio is still < 4 after 10 hours, `flagger` has a 2 hour window to liquidate the short.
- #3. Assume that the price moves in favour of the `shorter` and at the 8th hour, cRatio > 4 is achieved. The `shorter` himself did not take any action to do this. He just bet on the market & the price movement to save him. Or you can imagine that in the time he was arranging funds to fund the short, the market moved favourably and hence he did not need to do any action.
- #4. 10 hours pass. Obviously, `flagger` won't be able to liquidate now as cRatio > 4. 
- #5. The flag should also have been reset at the 8th hour. This is necessary so that in case cRatio again dips below 4 in the future, one has to flag the short again and wait for 10 hours before attempting liquidation.<br>
However, the flag is not reset by the protocol in the above scenario.

## Impact-1
Since the flag never got reset, if the price moves unfavourably again such that cRatio < 4, then -
- between 10-12 hour window, `flagger` can liquidate the short immediately, with no waiting period.
- between 12-16 hour window, **anyone** can liquidate the short immediately, with no waiting period.

## Impact-2
Let's now imagine that:
- #1. 16 hours pass. As [sepcified in the docs](https://dittoeth.com/litepaper#primary-margin-call-method) flag should be reset, forcing margin-callers to re-flag the short:
```
In the case where the 16 hour window has passed, margin callers must re-flag the short to begin the process again to liquidate the partial debt position.
```
<br>

Since the flag never got reset, if the price ***ever*** moves unfavourably after the 16 hour window such that cRatio < 4, then -
- **anyone** can liquidate the short immediately, with no waiting period.

## PoC
Create a file inside `test/` folder by the name of `FlagNotResetOnPriceMovement.t.sol` and paste the following code. It has 3 tests along with a few helper functions:
```js
// SPDX-License-Identifier: GPL-3.0-only
pragma solidity 0.8.21;

import {Constants} from "contracts/libraries/Constants.sol";
import {Errors} from "contracts/libraries/Errors.sol";
import {U256, U88} from "contracts/libraries/PRBMathHelper.sol";
import {OBFixture} from "test/utils/OBFixture.sol";
import {STypes, SR} from "contracts/libraries/DataTypes.sol";
import {LibShortRecord} from "contracts/libraries/LibShortRecord.sol";

import {console} from "contracts/libraries/console.sol";

contract FlagNotResetOnPriceMovement is OBFixture {
    using U256 for uint256;
    using U88 for uint88;
    using LibShortRecord for STypes.ShortRecord;

    function _initialAssertStruct() public {
        r.ethEscrowed = 0;
        r.ercEscrowed = DEFAULT_AMOUNT;
        assertStruct(receiver, r);
        s.ethEscrowed = 0;
        s.ercEscrowed = 0;
        assertStruct(sender, s);
        e.ethEscrowed = 0;
        e.ercEscrowed = 0;
        assertStruct(extra, e);
        t.ethEscrowed = 0;
        t.ercEscrowed = 0;
        assertStruct(tapp, t);
    }

    function _prepareAsk(uint80 askPrice, uint88 askAmount) public {
        fundLimitBidOpt(DEFAULT_PRICE, DEFAULT_AMOUNT, receiver);
        fundLimitShortOpt(DEFAULT_PRICE, DEFAULT_AMOUNT, sender);

        fundLimitAskOpt(askPrice, askAmount, receiver);
        assertEq(getShortRecordCount(sender), 1);
        assertEq(
            getShortRecord(sender, Constants.SHORT_STARTING_ID).collateral,
            DEFAULT_AMOUNT.mulU88(DEFAULT_PRICE) * 6
        );
        _initialAssertStruct();
    }

    function _checkFlaggerAndUpdatedAt(
        address _shorter,
        uint16 _flaggerId,
        uint256 _updatedAt
    ) public {
        STypes.ShortRecord memory shortRecord =
            getShortRecord(_shorter, Constants.SHORT_STARTING_ID);

        assertEq(shortRecord.flaggerId, _flaggerId, "flagger id mismatch");
        assertEq(shortRecord.updatedAt, _updatedAt, "updatedAt mismatch");
    }

    function _flagShortAndSkipTime(uint256 timeToSkip, address flagger) public {
        _prepareAsk({askPrice: DEFAULT_PRICE, askAmount: DEFAULT_AMOUNT});
        _setETH(2666 ether);
        _checkFlaggerAndUpdatedAt({_shorter: sender, _flaggerId: 0, _updatedAt: 0});
        vm.prank(flagger);
        diamond.flagShort(asset, sender, Constants.SHORT_STARTING_ID, Constants.HEAD);
        uint256 flagged = diamond.getOffsetTimeHours();
        skipTimeAndSetEth({skipTime: timeToSkip, ethPrice: 2666 ether});
        _checkFlaggerAndUpdatedAt({_shorter: sender, _flaggerId: 1, _updatedAt: flagged});
        assertSR(
            getShortRecord(sender, Constants.SHORT_STARTING_ID).status, SR.FullyFilled
        );
        depositEth(tapp, DEFAULT_TAPP);
    }

    /* solhint-disable no-console */
    function test_01_flagDoesNotResetOnFavorablePriceMovement() public {
        address _flagger_01 = makeAddr("_flagger_01");
        _flagShortAndSkipTime({timeToSkip: 8 hours, flagger: _flagger_01}); // cR < 4 here, hence flagged
        STypes.ShortRecord memory theShortRecord =
            getShortRecord(sender, Constants.SHORT_STARTING_ID);

        // favorable price movement after 8 hours
        _setETH(2700 ether); // cR > 4
        skip(3 hours); // 11 hours have passed since flagging. Eligible for liquidation attempt.
        _setETH(2700 ether); // avoids evm revert. cR still > 4

        // should be true
        bool isFlagged = theShortRecord.flaggerId != 0;
        console.log("isFlagged (should be true) =", isFlagged);

        vm.prank(_flagger_01);
        vm.expectRevert(Errors.SufficientCollateral.selector);
        // Correctly reverts as cR > 4
        diamond.liquidate(
            asset, sender, Constants.SHORT_STARTING_ID, shortHintArrayStorage
        );

        // should be false
        isFlagged = theShortRecord.flaggerId != 0;
        console.log("isFlagged (should be false) =", isFlagged); // @audit-issue : true; still not unflagged!
        assertEq(isFlagged, false, "Short's flag was not reset!");
    }

    /* solhint-disable no-console */
    function test_02_ImmediateLiquidationPossibleOnUnfavorablePriceMovement() public {
        address _flagger_01 = makeAddr("_flagger_01");
        address _anyone = makeAddr("_anyone_");
        _flagShortAndSkipTime({timeToSkip: 8 hours, flagger: _flagger_01}); // cR < 4 here, hence flagged
        STypes.ShortRecord memory theShortRecord =
            getShortRecord(sender, Constants.SHORT_STARTING_ID);

        // favorable price movement after 8 hours
        _setETH(2700 ether); // cR > 4
        skip(3 hours); // 11 hours have passed since flagging. Eligible for liquidation attempt.
        _setETH(2700 ether); // avoids evm revert. cR still > 4

        // should be true
        bool isFlagged = theShortRecord.flaggerId != 0;
        console.log("isFlagged (should be true) =", isFlagged);

        vm.prank(_flagger_01);
        vm.expectRevert(Errors.SufficientCollateral.selector);
        // Correctly reverts as cR > 4
        diamond.liquidate(
            asset, sender, Constants.SHORT_STARTING_ID, shortHintArrayStorage
        );

        // should be false
        isFlagged = theShortRecord.flaggerId != 0;
        console.log("isFlagged (should be false) =", isFlagged); // @audit-issue : true; still not unflagged!

        skip(3 hours); // 14 hours passed since flagging. Anyone can attempt liquidation.
        _setETH(2700 ether); // avoids evm revert. cR still > 4

        vm.prank(_anyone);
        vm.expectRevert(Errors.SufficientCollateral.selector);
        // Correctly reverts as cR > 4
        diamond.liquidate(
            asset, sender, Constants.SHORT_STARTING_ID, shortHintArrayStorage
        );

        skip(2 minutes);
        _setETH(2666 ether); // cR < 4 due to unfavorable price movement.

        vm.prank(_anyone);
        // @audit-issue : ANYONE is able to liquidate (incorrectly), without re-flagging and waiting for 10 hours! Should not be possible.
        diamond.liquidate(
            asset, sender, Constants.SHORT_STARTING_ID, shortHintArrayStorage
        );
    }

    /* solhint-disable no-console */
    function test_03_flagNotResetAfter16Hours() public {
        address _flagger_01 = makeAddr("_flagger_01");
        address _anyone = makeAddr("_anyone_");
        _flagShortAndSkipTime({timeToSkip: 8 hours, flagger: _flagger_01}); // cR < 4 here, hence flagged
        STypes.ShortRecord memory theShortRecord =
            getShortRecord(sender, Constants.SHORT_STARTING_ID);

        // favorable price movement after 8 hours
        _setETH(2700 ether); // cR > 4
        skip(3 hours); // 11 hours have passed since flagging. Eligible for liquidation attempt.
        _setETH(2700 ether); // avoids evm revert. cR still > 4

        // should be true
        bool isFlagged = theShortRecord.flaggerId != 0;
        console.log("isFlagged (should be true) =", isFlagged);

        vm.prank(_flagger_01);
        vm.expectRevert(Errors.SufficientCollateral.selector);
        // Correctly reverts as cR > 4
        diamond.liquidate(
            asset, sender, Constants.SHORT_STARTING_ID, shortHintArrayStorage
        );

        skip(3 hours); // 14 hours passed since flagging. Anyone can attempt liquidation.
        _setETH(2700 ether); // avoids evm revert. cR still > 4

        vm.prank(_anyone);
        vm.expectRevert(Errors.SufficientCollateral.selector);
        // Correctly reverts as cR > 4
        diamond.liquidate(
            asset, sender, Constants.SHORT_STARTING_ID, shortHintArrayStorage
        );

        skip(4 hours); // 18 hours passed since flagging. Flag should have been reset.
        _setETH(2700 ether); // avoids evm revert. cR still > 4

        // should be false
        isFlagged = theShortRecord.flaggerId != 0;
        console.log("isFlagged (should be false) =", isFlagged); // @audit-issue : true; still not unflagged!

        skip(1 minutes);
        _setETH(2666 ether); // cR < 4 due to unfavorable price movement.

        vm.prank(_anyone);
        // @audit-issue : ANYONE is able to liquidate (incorrectly) even after 16 hours, without re-flagging and waiting for 10 hours! Should not be possible.
        diamond.liquidate(
            asset, sender, Constants.SHORT_STARTING_ID, shortHintArrayStorage
        );
    }
}

```

- **Test-1:** Run `forge test --mt test_01_flagDoesNotResetOnFavorablePriceMovement -vv` to verify that the flag is never reset.<br>
Output:
```
Logs:
  isFlagged (should be true) = true
  isFlagged (should be false) = true
  Error: Short's flag was not reset!
  Error: a == b not satisfied [bool]
        Left: true
       Right: false

Test result: FAILED. 0 passed; 1 failed; 0 skipped; finished in 133.72ms
```

- **Test-2:** Run `forge test --mt test_02_ImmediateLiquidationPossibleOnUnfavorablePriceMovement -vv` to verify the aforementioned `Impact-1`.<br>
The test passes, but should have failed as per protocol specifications.<br>

- **Test-3:** Run `forge test --mt test_03_flagNotResetAfter16Hours -vv` to verify the aforementioned `Impact-2`.<br>
The test passes, but should have failed as per protocol specifications.

## Tools Used
Manual review, foundry.

## Recommendations
- Whenever a short is flagged, its price movement needs to be tracked so that its flag can be correctly reset whenever it crosses a healthy cRatio back again. This might need altogether additional new functions to be implemented. An idea: Users can be incentivized to reset a short's flag when the right conditions are met (just like they are incentivized to flag a short).
- Another approach could be that at every liquidation or flagging attempt of a short, the code not only checks whether the short is already flagged, but also the timestamp of the flagging. If it is beyond 16 hours, flag resets. This approach however, will only address `Impact-2`.

---

### <a id="h-08"></a>[H-08]
## **The right of `short.flagger` to liquidate short between `firstLiquidationTime` and `secondLiquidationTime` is incorrectly transferred to another flagger**
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibShortRecord.sol#L393-L394
<br>

## Summary
As per [docs](https://dittoeth.com/technical/margincall#primary-liquidation) :
> Assuming `flagShort()` was successful,
>
>    - `firstLiquidationTime` is 10hrs, `secondLiquidationTime` is 12hrs, `resetLiquidationTime` is 16hrs (these are modifiable)
>    - Between `updatedAt` and `firstLiquidationTime`, `liquidate()` isn't callable.
>  > - Between `firstLiquidationTime` and `secondLiquidationTime`, `liquidate()` is only callable by `short.flagger`.

<br>

The last point can be incorrectly revoked by the protocol under the normal flow of events, violating flagger's priority liquidation time-window.

## Vulnerability Details
Imagine the following flow of events:
- #1. `shortRecord_1` is flagged by `flagger1` at timestamp `ts`. This short now has 10 hours. If `cRatio` is still less than `primaryLiquidationCR` after 10 hours, `flagger1` has a 2 hour window to liquidate the short. The code sets `shortRecord_1.flaggerId` as 1. Read about `flaggerId` [here](https://dittoeth.com/technical/margincall#flagging-short):
> Note: Instead of saving the flagger address, a ShortRecord saves the flaggerId instead. This is done to save space in the struct. A mapping from id to address (flagMapping) is used to lookup the address instead. There is also a corresponding AssetUser.g_flaggerId to lookup a user's flagId.

- #2. 10 hours pass. `shortRecord_1` is now eligible for liquidation by `flagger1`. Right about now, a different short record, `shortRecord_2` is flagged by `flagger2`.
- #3. Due to a logic bug in the function [setFlagger()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibShortRecord.sol#L393-L394), the code sets `shortRecord_2.flaggerId` as 1 too, since it attempts to re-use the _"inactive"_ flaggerId.
- #4. The above point means that as per code, both `shortRecord_1` and `shortRecord_2` are now considered to be flagged by `flagger2`. 
- #5. Since `shortRecord_1` is still in its primary liquidation window, `flagger1` attempts to margin call it only to find that it reverts for him!
- #6. Due to the above logic fallacy, `flagger2` actually now has the power to liquidate `shortRecord_1`. He margin calls `shortRecord_1` immediately and receives the rewards.

## Root Cause
The current check to see if a `flaggerId` is "inactive" or not is incorrect. The protocol currently waits for a duration of `firstLiquidationTime` to pass for determining this. It should rather wait for `secondLiquidationTime`.

## PoC
Create a file inside `test/` folder by the name of `TwoFlaggers.t.sol` and paste the following code. Run it via `forge test --mt test_TwoFlaggers -vv`:
```js
// SPDX-License-Identifier: GPL-3.0-only
pragma solidity 0.8.21;

import {Constants} from "contracts/libraries/Constants.sol";
import {Errors} from "contracts/libraries/Errors.sol";
import {U256, U88, U80} from "contracts/libraries/PRBMathHelper.sol";
import {OBFixture} from "test/utils/OBFixture.sol";
import {STypes} from "contracts/libraries/DataTypes.sol";
import {LibShortRecord} from "contracts/libraries/LibShortRecord.sol";

import {console} from "contracts/libraries/console.sol";

contract TwoFlaggers is OBFixture {
    using U256 for uint256;
    using U88 for uint88;
    using U80 for uint80;
    using LibShortRecord for STypes.ShortRecord;

    function _checkFlaggerAndUpdatedAt(
        uint8 short_id,
        address _shorter,
        uint16 _flaggerId,
        uint256 _updatedAt
    ) public {
        STypes.ShortRecord memory shortRecord = getShortRecord(_shorter, short_id);

        assertEq(shortRecord.flaggerId, _flaggerId, "flagger id mismatch");
        assertEq(shortRecord.updatedAt, _updatedAt, "updatedAt mismatch");
    }

    /* solhint-disable no-console */
    function test_TwoFlaggers() public {
        address _flagger_01 = makeAddr("_flagger_01");
        address _flagger_02_lucky_fella = makeAddr("_flagger_02");

        // create orders
        fundLimitShortOpt(0.00025 ether, DEFAULT_AMOUNT, sender); // default_amount = 4000 ether
        fundLimitShortOpt(0.0003751 ether, DEFAULT_AMOUNT, sender);
        fundLimitBidOpt(0.00025 ether, DEFAULT_AMOUNT, receiver);
        fundLimitBidOpt(0.0003751 ether, DEFAULT_AMOUNT, receiver);
        STypes.ShortRecord memory shortRecord_1 =
            getShortRecord(sender, Constants.SHORT_STARTING_ID);
        STypes.ShortRecord memory shortRecord_2 =
            getShortRecord(sender, Constants.SHORT_STARTING_ID + 1);

        skipTimeAndSetEth({skipTime: 1 minutes, ethPrice: 2666 ether}); // causing `cR < primaryLiquidationCR` for short1
        // flagger1 flags short1
        vm.prank(_flagger_01);
        diamond.flagShort(asset, sender, Constants.SHORT_STARTING_ID, Constants.HEAD);
        // verify that flaggerId == 1
        _checkFlaggerAndUpdatedAt({
            short_id: Constants.SHORT_STARTING_ID,
            _shorter: sender,
            _flaggerId: 1,
            _updatedAt: diamond.getOffsetTimeHours()
        });

        // skip initial duration of 10 hours where short1 has the chance to increase its collateral
        skipTimeAndSetEth({skipTime: TEN_HRS_PLUS, ethPrice: 1750 ether}); // causing `cR < primaryLiquidationCR` for short2. Short1 still remains under-collaterized.

        // flagger2 flags short2
        vm.prank(_flagger_02_lucky_fella);
        diamond.flagShort(asset, sender, Constants.SHORT_STARTING_ID + 1, Constants.HEAD);
        // @audit-issue : same flaggerId (1) re-used by the protocol due to which both shorts now have the same flaggerId
        _checkFlaggerAndUpdatedAt({
            short_id: Constants.SHORT_STARTING_ID + 1,
            _shorter: sender,
            _flaggerId: 1,
            _updatedAt: diamond.getOffsetTimeHours()
        });
        assertEq(
            shortRecord_1.flaggerId,
            shortRecord_2.flaggerId,
            "shorts have different flagger ids"
        );

        // @audit-issue :
        //     !!
        // Values for both `shortRecord_1.flaggerId` & `shortRecord_2.flaggerId` are 1 now. This
        // means that they are now mapped to the same address - that of flagger2. This is mapped via `s.flagMapping[short.flaggerId] => flagger_address`.
        // bug: Flagger1 can now not liquidate `shortRecord_1` even if he tries. However, flagger2 can now liquidate `shortRecord_1`, and
        // receive the proceeds, violating flagger1's priority liquidation time-window. This is despite the fact that flagger2 had never flagged short1.
        //     !!
        fundLimitAskOpt(DEFAULT_PRICE, DEFAULT_AMOUNT, receiver); // adding an ask order to avoid `Errors.NoSells()` during liquidation
        // pre-verfication:
        assertEq(
            diamond.getVaultUserStruct(vault, _flagger_02_lucky_fella).ethEscrowed, 0
        );
        assertEq(diamond.getVaultUserStruct(vault, _flagger_01).ethEscrowed, 0);

        // flagger1 is unable to liquidate `shortRecord_1` now
        vm.prank(_flagger_01);
        vm.expectRevert(Errors.MarginCallIneligibleWindow.selector);
        diamond.liquidate(
            asset, sender, Constants.SHORT_STARTING_ID, shortHintArrayStorage
        );

        // flagger2 has "received" the liquidation rights now for `shortRecord_1` now!
        vm.prank(_flagger_02_lucky_fella);
        diamond.liquidate(
            asset, sender, Constants.SHORT_STARTING_ID, shortHintArrayStorage
        );

        // post-verfication:
        // verify -
        //   - a. if short1 has been liquidated (shortRecord id is no more valid)
        //   - b. if flagger2 is rewarded
        //   - c. that flagger1 is not rewarded

        // a
        vm.prank(_flagger_01);
        vm.expectRevert(Errors.InvalidShortId.selector);
        diamond.liquidate(
            asset, sender, Constants.SHORT_STARTING_ID, shortHintArrayStorage
        );

        // b
        assertGt(
            diamond.getVaultUserStruct(vault, _flagger_02_lucky_fella).ethEscrowed,
            0 ether,
            "flagger2 not rewarded"
        );

        // c
        assertEq(
            diamond.getVaultUserStruct(vault, _flagger_01).ethEscrowed,
            0 ether,
            "flagger1 got rewarded"
        );
    }
}
```

## Tools Used
Manual review, foundry.

## Recommendations
```diff
  File: contracts/libraries/LibShortRecord.sol

  377           function setFlagger(
  378               STypes.ShortRecord storage short,
  379               address cusd,
  380               uint16 flaggerHint
  381           ) internal {
  382               AppStorage storage s = appStorage();
  383               STypes.AssetUser storage flagStorage = s.assetUser[cusd][msg.sender];
  384
  385               //@dev Whenever a new flagger flags, use the flaggerIdCounter.
  386               if (flagStorage.g_flaggerId == 0) {
  387                   address flaggerToReplace = s.flagMapping[flaggerHint];
  388
  389                   uint256 timeDiff = flaggerToReplace != address(0)
  390                       ? LibOrders.getOffsetTimeHours()
  391                           - s.assetUser[cusd][flaggerToReplace].g_updatedAt
  392                       : 0;
  393                   //@dev re-use an inactive flaggerId
- 394                   if (timeDiff > LibAsset.firstLiquidationTime(cusd)) {
+ 394                   if (timeDiff > LibAsset.secondLiquidationTime(cusd)) {
  395                       delete s.assetUser[cusd][flaggerToReplace].g_flaggerId;
  396                       short.flaggerId = flagStorage.g_flaggerId = flaggerHint;
  397                   } else if (s.flaggerIdCounter < type(uint16).max) {
  398                       //@dev generate brand new flaggerId
  399                       short.flaggerId = flagStorage.g_flaggerId = s.flaggerIdCounter;
  400                       s.flaggerIdCounter++;
  401                   } else {
  402                       revert Errors.InvalidFlaggerHint();
  403                   }
  404                   s.flagMapping[short.flaggerId] = msg.sender;
  405               } else {
  406                   //@dev re-use flaggerId if flagger has an existing one
  407                   short.flaggerId = flagStorage.g_flaggerId;
  408               }
  409               short.updatedAt = flagStorage.g_updatedAt = LibOrders.getOffsetTimeHours();
  410           }
```

---

### <a id="h-09"></a>[H-09]
## **Malicious shorter can escape liquidation perpetually**
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibShortRecord.sol#L328
<br>

## Summary
A malicious shorter can create a short record in position 254 and keep on escaping liquidation attempts by flaggers indefinitely, meanwhile enjoying less than mandated `cRatio`. 

## Vulnerability Details
As per [docs](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibShortRecord.sol#L328) :
> shortRecordIdCounter is a uint8 (max 255) for struct packing/gas saving purposes but can overflow relatively easily. A malicious actor can create the max number of shortRecords and prevent matching by flooding the orderbook with short orders that overflow shortRecordIdCounter upon creation of the next shortRecord.
>
> To counteract this behavior the shortRecord in position 254 is modified/blended anytime a new shortRecord would have been created with a shortRecordIdCounter greater than 254. 
<br>

The issue with the above is that if short records till position 254 have been already created, [createShortRecord() internally calls fillShortRecord()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibShortRecord.sol#L107) which then calls [merge()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibShortRecord.sol#L174) which [updates the short.updatedAt to current time stamp](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibShortRecord.sol#L327-L328). 
<br>

In essence, every time any new short record is created post the #254 mark, it is blended into the existing #254 and the timestamp is reset to current time.
<br>

Now, imagine the following flow of events:
- #1. `maliciousShorter` creates short records till position #254 with minimum amounts. 
- #2. `maliciousShorter` creates the actual, large short now. _Optional:_ Note that the `maliciousShorter` now has an option of exiting the previous short records (position #2 to position #253) if he wants to.
- #3. After some time, unfavourable price movement causes the short's `cRatio` to be below `primaryLiquidationCR`.
- #4. `flagger` flags the short record at position #254. `flagger` can now liquidate this if cRatio does not come back up within next 10 hours. 
- #5. Seeing the flag, `maliciousShorter` simply adds a new short record (which gets blended into #254) and causes `updatedAt` to be reset to current timestamp for #254. _Optional:_ If he had exited his previous short records from position #2 to #253 in step 2 above, then he will first need to recreate them. 
- #6. `flagger` can't liquidate now even after 10 hours have passed due to `Errors.MarginCallIneligibleWindow()`.
- #7. `maliciousShorter` keeps on doing this every time someone flags the short record. He escapes liquidation perpetually.
<br>
<br>
<br>

Below are two PoCs. PoC-1 is without the optional step of cancellation of positions #2-#253. PoC-2 shows that optional step.

## PoC-1
Paste the following test inside `test/AskShortOrders.t.sol` and run via `forge test --mt test_escapeLiquidationPerpetually -vv`:
```js
    /* solhint-disable no-console */
    function test_escapeLiquidationPerpetually() public {
        address _flagger_ = makeAddr("_flagger_");
        address maliciousShorter = sender;

        for (uint256 i = Constants.SHORT_STARTING_ID; i < 256; i++) {
            fundLimitBidOpt(DEFAULT_PRICE, DEFAULT_AMOUNT, maliciousShorter);
            fundLimitShortOpt(DEFAULT_PRICE, DEFAULT_AMOUNT, maliciousShorter);
        }

        // Max shortRecordId utilized is 254 and takes any overflow
        assertEq(getShortRecord(maliciousShorter, 253).ercDebt, DEFAULT_AMOUNT);
        assertEq(getShortRecord(maliciousShorter, 254).ercDebt, DEFAULT_AMOUNT * 2);
        assertEq(getShortRecord(maliciousShorter, 255).ercDebt, 0);
        assertEq(
            getShortRecord(maliciousShorter, 254).updatedAt,
            diamond.getOffsetTimeHours(),
            "short record `updatedAt` mismatch"
        );

        // let's simulate liquidation scenario for short record 254
        skipTimeAndSetEth({skipTime: 5 minutes, ethPrice: 1900 ether}); // cR < primaryLiquidationCR now
        // verify the cRatio is less than primaryLiquidationCR
        assertLt(
            diamond.getCollateralRatio(asset, getShortRecord(maliciousShorter, 254)),
            diamond.getAssetNormalizedStruct(asset).primaryLiquidationCR,
            "cRatio is not less than primaryLiquidationCR"
        );
        // flag it
        vm.prank(_flagger_);
        diamond.flagShort(asset, maliciousShorter, 254, 254);
        assertGt(
            getShortRecord(maliciousShorter, 254).flaggerId, 0, "short record not flagged"
        );

        // @audit-issue : Can escape liquidation till eternity.
        // sender (maliciousShorter) games the system upon seeing his short record flagged, by-
        //    1. Waiting until close to the 10 hour mark.
        //    2. Adding a couple of orders. This "merges" the new short record into the existing #254.

        // The protocol updates the `updatedAt` timestamp too here. Hence the calculation for the presence of
        // `FirstLiquidationTime` window gets messed up.

        // setting ethPrice again alongwith skipTime to avoid evm_revert()
        skipTimeAndSetEth({skipTime: 9 hours + 55 minutes, ethPrice: 1900 ether});
        fundLimitShortOpt(0.000527 ether, DEFAULT_AMOUNT * 10, maliciousShorter); // a big order, as an example
        fundLimitBidOpt(0.000527 ether, DEFAULT_AMOUNT, maliciousShorter); // creates bid himself if there's no matching bid in the system, to force a short-record creation
        assertEq(getShortRecord(maliciousShorter, 254).ercDebt, DEFAULT_AMOUNT * 3);
        assertEq(
            getShortRecord(maliciousShorter, 254).updatedAt,
            diamond.getOffsetTimeHours(),
            "short record `updatedAt` was not updated"
        );

        fundLimitAskOpt(0.000527 ether, DEFAULT_AMOUNT, maliciousShorter); // adding an ask order into the system to avoid `Errors.NoSells()` during liquidation later on

        // time to liquidate
        skipTimeAndSetEth({skipTime: 1 hours, ethPrice: 1900 ether});
        // verify the cRatio is still less than primaryLiquidationCR
        assertLt(
            diamond.getCollateralRatio(asset, getShortRecord(maliciousShorter, 254)),
            diamond.getAssetNormalizedStruct(asset).primaryLiquidationCR,
            "cRatio >= primaryLiquidationCR"
        );
        vm.prank(_flagger_);
        // reverts with `Errors.MarginCallIneligibleWindow`. Flagger can't liquidate!
        vm.expectRevert(Errors.MarginCallIneligibleWindow.selector);
        diamond.liquidate(asset, maliciousShorter, 254, shortHintArrayStorage);
        // `maliciousShorter` keeps doing the above step over and over again every 10 hours; escapes liquidation perpetually.
    }
```

## PoC-2
Paste the following test inside `test/AskShortOrders.t.sol` and run via `forge test --mt test_2_escapeLiquidationPerpetually -vv`:
```js
    /* solhint-disable no-console */
    function test_2_escapeLiquidationPerpetually() public {
        address _flagger_ = makeAddr("_flagger_");
        address maliciousShorter = sender;

        for (uint256 i = Constants.SHORT_STARTING_ID; i < 256; i++) {
            fundLimitBidOpt(DEFAULT_PRICE, DEFAULT_AMOUNT, maliciousShorter);
            fundLimitShortOpt(DEFAULT_PRICE, DEFAULT_AMOUNT, maliciousShorter);
        }

        // Max shortRecordId utilized is 254 and takes any overflow
        assertEq(getShortRecord(maliciousShorter, 253).ercDebt, DEFAULT_AMOUNT);
        assertEq(getShortRecord(maliciousShorter, 254).ercDebt, DEFAULT_AMOUNT * 2);
        assertEq(getShortRecord(maliciousShorter, 255).ercDebt, 0);
        assertEq(
            getShortRecord(maliciousShorter, 254).updatedAt,
            diamond.getOffsetTimeHours(),
            "short record `updatedAt` mismatch"
        );
        // Optional Step: exit positions #2 to #253
        for (uint8 i = Constants.SHORT_STARTING_ID; i < 254; i++) {
            depositUsd(maliciousShorter, DEFAULT_AMOUNT);
            exitShortErcEscrowed(i, DEFAULT_AMOUNT, maliciousShorter);
            assertTrue(getShortRecord(maliciousShorter, i).status == SR.Cancelled);
        }

        // let's simulate liquidation scenario for short record 254
        skipTimeAndSetEth({skipTime: 5 minutes, ethPrice: 1900 ether}); // cR < primaryLiquidationCR now
        // verify the cRatio is less than primaryLiquidationCR
        assertLt(
            diamond.getCollateralRatio(asset, getShortRecord(maliciousShorter, 254)),
            diamond.getAssetNormalizedStruct(asset).primaryLiquidationCR,
            "cRatio is not less than primaryLiquidationCR"
        );
        // flag it
        vm.prank(_flagger_);
        diamond.flagShort(asset, maliciousShorter, 254, 254);
        assertGt(
            getShortRecord(maliciousShorter, 254).flaggerId, 0, "short record not flagged"
        );

        // @audit-issue : Can escape liquidation till eternity.
        // sender (maliciousShorter) games the system upon seeing his short record flagged, by-
        //    1. Waiting until close to the 10 hour mark.
        //    2. Adding a couple of orders. This "merges" the new short record into the existing #254.

        // The protocol updates the `updatedAt` timestamp too here. Hence the calculation for the presence of
        // `FirstLiquidationTime` window gets messed up.

        // setting ethPrice again alongwith skipTime to avoid evm_revert()
        skipTimeAndSetEth({skipTime: 9 hours + 55 minutes, ethPrice: 1900 ether});

        // recreate positions #2 to #253 which were exited in the optional step above
        for (uint256 i = Constants.SHORT_STARTING_ID; i < 254; i++) {
            fundLimitBidOpt(0.000527 ether, DEFAULT_AMOUNT, maliciousShorter);
            fundLimitShortOpt(0.000527 ether, DEFAULT_AMOUNT, maliciousShorter);
        }

        fundLimitShortOpt(0.000527 ether, DEFAULT_AMOUNT * 10, maliciousShorter); // a big order, as an example
        fundLimitBidOpt(0.000527 ether, DEFAULT_AMOUNT, maliciousShorter); // creates bid himself if there's no matching bid in the system, to force a short-record creation
        assertEq(getShortRecord(maliciousShorter, 254).ercDebt, DEFAULT_AMOUNT * 3);
        assertEq(
            getShortRecord(maliciousShorter, 254).updatedAt,
            diamond.getOffsetTimeHours(),
            "short record `updatedAt` was not updated"
        );

        fundLimitAskOpt(0.000527 ether, DEFAULT_AMOUNT, maliciousShorter); // adding an ask order into the system to avoid `Errors.NoSells()` during liquidation later on

        // time to liquidate
        skipTimeAndSetEth({skipTime: 1 hours, ethPrice: 1900 ether});
        // verify the cRatio is still less than primaryLiquidationCR
        assertLt(
            diamond.getCollateralRatio(asset, getShortRecord(maliciousShorter, 254)),
            diamond.getAssetNormalizedStruct(asset).primaryLiquidationCR,
            "cRatio >= primaryLiquidationCR"
        );
        vm.prank(_flagger_);
        // reverts with `Errors.MarginCallIneligibleWindow`. Flagger can't liquidate!
        vm.expectRevert(Errors.MarginCallIneligibleWindow.selector);
        diamond.liquidate(asset, maliciousShorter, 254, shortHintArrayStorage);
        // `maliciousShorter` keeps doing the above step over and over again every 10 hours; escapes liquidation perpetually.
    }
```

## Tools Used
Manual review, foundry.

## Recommendations
The function `combineShorts()` does not allow merging short records if one of them was flagged, and if the new merged record would be still under-collateralized. A similar approach can be taken here by constraining the `maliciousShorter` from creating more limit shorts if existing #254 is flagged and the new short record does not make it over-collateralized.

<br><br>

## **MEDIUM-SEVERITY BUGS**
---

### <a id="m-01"></a>[M-01]
## **Protocol's calculation fails for `twapPrice < 1e6`**
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibOracle.sol#L85
<br>

## Summary
Protocol's calculation fails for `twapPrice < 1e6`

## Vulnerability Details
As per developer [comments](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibOracle.sol#L80):
```
File: libraries/LibOracle.sol

80    //@dev if there is issue with chainlink, get twap price. Compare twap and chainlink
```

However, protocol will not be able to fall back on twap & incorrectly always revert when `twapPrice < 1e6`, as division has been performed before multiplication, causing precision loss. <br>
If `twapPrice` returned by the following line of code ([L82](https://github.com/Cyfrin/2023-09-ditto/blob/c52c7cec881e0d41f65b057d0df51e97ccad8513/contracts/libraries/LibOracle.sol#L82)) ever goes below `1e6`,
```js
uint256 twapPrice = IDiamond(payable(address(this))).estimateWETHInUSDC(
    Constants.UNISWAP_WETH_BASE_AMT, 30 minutes
);
``` 
then in the next line, the `twapPriceInEther` is incorrectly calculated as zero:
```js
uint256 twapPriceInEther = (twapPrice / Constants.DECIMAL_USDC) * 1 ether;
```
Multiplication should have been performed first, followed by division.
<br>
**Example:**
Assume `twapPrice` returned to be `1e6 - 1 = 999999`. The `twapPriceInEther` calculated should ideally be:<br>
***CORRECT FORMULA:***
```js
twapPriceInEther = (999999 * 1 ether) / Constants.DECIMAL_USDC = (999999 * 1e18) / 1e6 = 999999000000000000000000 / 1e6 = 999999000000000000
``` 

<br>

However, the protocol incorrectly calculates it as:<br>
***INCORRECT FORMULA:***
```js
twapPriceInEther = (999999 / 1e6) * 1e18 = 0 * 1e18 = 0
```

## Impact
Protocol not able to fall back on twap when `twapPrice < 1e6`.

## Tools Used
Manual review

## Recommendations
Perform multiplication before division:
```diff
-    uint256 twapPriceInEther = (twapPrice / Constants.DECIMAL_USDC) * 1 ether;
+    uint256 twapPriceInEther = (twapPrice * 1 ether) / Constants.DECIMAL_USDC;
```

---

### <a id="m-02"></a>[M-02]
## **`Errors.InvalidTwapPrice()` is never invoked when `if (twapPriceInEther == 0)` is true**
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibOracle.sol#L87-L89
<br>

## Summary & Vulnerability Details
The protocol expects to `revert` with `Errors.InvalidTwapPrice()` when `twapPriceInEther == 0`:
```js
File: contracts/libraries/LibOracle.sol

85            uint256 twapPriceInEther = (twapPrice / Constants.DECIMAL_USDC) * 1 ether;
86            uint256 twapPriceInv = twapPriceInEther.inv();
87            if (twapPriceInEther == 0) {
88                revert Errors.InvalidTwapPrice(); // @audit : unreachable code
89            }
```
However, the control never reaches Line 88 when `twapPriceInEther` is zero. It rather reverts before that with error `Division or modulo by 0`. <br>
***NOTE:*** Due to this bug, `Errors.InvalidTwapPrice()` is **never** invoked/thrown by the protocol even under satisfactory conditions, even though it has been defined. 

## PoC
Since I could not find any helper function inside `contracts/` or `test/` which lets one set the `twapPrice` returned by `uint256 twapPrice = IDiamond(payable(address(this))).estimateWETHInUSDC(Constants.UNISWAP_WETH_BASE_AMT, 30 minutes);` to zero for testing purposes, I have created a simplified PoC which targets the problem area:<br>
Save the following as a file named `test/InvalidTwapPriceErrorCheck.t.sol` and run the test via `forge test --mt testInvalidTwapPriceErrNeverInvoked -vv`. You will find that the test reverts with error `Division or modulo by 0`, but not with `Errors.InvalidTwapPrice()`. The PoC uses the same underlying math libraries and logic path as the protocol does in `contracts/libraries/LibOracle.sol::baseOracleCircuitBreaker()`.
```js
// SPDX-License-Identifier: GPL-3.0-only
pragma solidity 0.8.21;

import {Constants} from "contracts/libraries/Constants.sol";
import {Errors} from "contracts/libraries/Errors.sol";
import {U256} from "contracts/libraries/PRBMathHelper.sol";
import {OBFixture} from "test/utils/OBFixture.sol";

contract InvalidTwapPriceErrorCheck is OBFixture {
    using U256 for uint256;

    function getZeroTwapPriceInEther_IncorrectStyle_As_In_Existing_DittoProtocol()
        internal
        pure
        returns (uint256 twapPriceInEther, uint256 twapPriceInv)
    {
        // fake the twapPrice to 0
        uint256 twapPrice = 0; // IDiamond(payable(address(this))).estimateWETHInUSDC(Constants.UNISWAP_WETH_BASE_AMT, 30 minutes);
        // Following code is copied as-is from
        // `contracts/libraries/LibOracle.sol::baseOracleCircuitBreaker()#L85-L89`
        twapPriceInEther = (twapPrice / Constants.DECIMAL_USDC) * 1 ether;
        twapPriceInv = twapPriceInEther.inv();
        if (twapPriceInEther == 0) {
            revert Errors.InvalidTwapPrice(); // @audit : unreachable code
        }
    }

    function getZeroTwapPriceInEther_CorrectStyle()
        internal
        pure
        returns (uint256 twapPriceInEther, uint256 twapPriceInv)
    {
        // fake the twapPrice to 0
        uint256 twapPrice = 0; // IDiamond(payable(address(this))).estimateWETHInUSDC(Constants.UNISWAP_WETH_BASE_AMT, 30 minutes);
        twapPriceInEther = (twapPrice / Constants.DECIMAL_USDC) * 1 ether;
        if (twapPriceInEther == 0) { 
            revert Errors.InvalidTwapPrice();
        }
        twapPriceInv = twapPriceInEther.inv();
    }

    function testInvalidTwapPriceErrNeverInvoked() public pure {
        getZeroTwapPriceInEther_IncorrectStyle_As_In_Existing_DittoProtocol();
    }

    function testInvalidTwapPriceErrInvokedCorrectly() public {
        vm.expectRevert(Errors.InvalidTwapPrice.selector);
        getZeroTwapPriceInEther_CorrectStyle();
    }
}
```

<br>

In the above test file, you can also run the test which invokes the "fixed" or "correct" code style via `forge test --mt testInvalidTwapPriceErrInvokedCorrectly -vv`. This will invoke the `Errors.InvalidTwapPrice` error, as expected.

## Recommendations & Root Cause
The check on Line 87 (`if` condition) needs to be performed immediately after Line 85.
```diff
    85            uint256 twapPriceInEther = (twapPrice / Constants.DECIMAL_USDC) * 1 ether;
+   86            if (twapPriceInEther == 0) {
+   87                revert Errors.InvalidTwapPrice();
+   88            }
+   89            uint256 twapPriceInv = twapPriceInEther.inv();
-   86            uint256 twapPriceInv = twapPriceInEther.inv();
-   87            if (twapPriceInEther == 0) {
-   88                revert Errors.InvalidTwapPrice();
-   89            }
```
The above fix needed to be done because the `inv()` call caused a revert even before control used to reach the `if` condition.

## Impact
Protocol owner or developer monitoring for a revert due to `Errors.InvalidTwapPrice()` in the logs will never see it and will make debugging & issue resolution harder. 

## Tools Used
Manual audit & foundry.

---

### <a id="m-03"></a>[M-03]
## **Unbounded loop & lack of duplicate-array-element check in external function `distributeYield()` exposes protocol to DoS attack vector**
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/YieldFacet.sol#L53-L69
<br>

## Summary
An attacker can launch a DoS attack by calling the [distributeYield()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/YieldFacet.sol#L53-L69) function with a huge array of `address[] calldata assets`, making the protocol service unavailable for other users.

## Vulnerability Details
`distributeYield()` allows the array `assets` to be passed as a param with no limit on its length. Also, **duplicate values inside the array are allowed**. 
```js
File: contracts/facets/YieldFacet.sol

53            function distributeYield(address[] calldata assets) external nonReentrant {
54                uint256 length = assets.length;
55                uint256 vault = s.asset[assets[0]].vault;
56
57                // distribute yield for the first order book
58                (uint88 yield, uint256 dittoYieldShares) = _distributeYield(assets[0]);
59
60                // distribute yield for remaining order books
61                for (uint256 i = 1; i < length;) {
62                    if (s.asset[assets[i]].vault != vault) revert Errors.DifferentVaults();
63                    (uint88 amtYield, uint256 amtDittoYieldShares) = _distributeYield(assets[i]);
64                    yield += amtYield;
65                    dittoYieldShares += amtDittoYieldShares;
66                    unchecked {
67                        ++i;
68                    }
69                }
70                // claim all distributed yield
71                _claimYield(vault, yield, dittoYieldShares);
72                emit Events.DistributeYield(vault, msg.sender, yield, dittoYieldShares);
73            }
```
So, an attacker can create a huge array with a single asset address at every index and pass it to the function. This causes a DoS attack and can cause the protocol service to be unavailable to other users. Note that there is nothing to be gained in terms of yield by passing an array with duplicate asset addresses, so this would be done only for a DoS attack.<br><br>
Add the following test inside `test/Yield.t.sol` and run it through `forge test --mt test_DistributeYield_HugeDuplicateAssetArray -vv`.
```js
    function test_DistributeYield_HugeDuplicateAssetArray() public {
        fundLimitShortOpt(DEFAULT_PRICE, DEFAULT_AMOUNT, receiver);
        fundLimitBidOpt(DEFAULT_PRICE, DEFAULT_AMOUNT, sender);
        skip(yieldEligibleTime);
        generateYield(1 ether);

        // @audit-info : example shown with array of size 4, but an
        // attacker can create a huge array of duplicate entries here.
        address[] memory assets = new address[](4);
        assets[0] = asset;
        assets[1] = asset;
        assets[2] = asset;
        assets[3] = asset;
        uint256 ethEscrowed1 = diamond.getVaultUserStruct(vault, receiver).ethEscrowed;

        vm.prank(receiver);
        diamond.distributeYield(assets);
        uint256 ethEscrowed2 = diamond.getVaultUserStruct(vault, receiver).ethEscrowed;
        assertApproxEqAbs(ethEscrowed2 - ethEscrowed1, 900000000000000000, MAX_DELTA);
    }
```

## Impact
DoS attack makes the protocol service unavailable for other users.

## Tools Used
Manual inspection and foundry.

## Recommendations
A constraint like the following can be added in `distributeYield()` which puts an upper limit on the `assets` array size being passed into the function. If a user really needs to pass many assets, he can try in separate transactions.
```diff
53            function distributeYield(address[] calldata assets) external nonReentrant {
54                uint256 length = assets.length;
+                 require(length < 10, "Too many assets");
55                uint256 vault = s.asset[assets[0]].vault;
56
57                // distribute yield for the first order book
58                (uint88 yield, uint256 dittoYieldShares) = _distributeYield(assets[0]);
59
60                // distribute yield for remaining order books
61                for (uint256 i = 1; i < length;) {
62                    if (s.asset[assets[i]].vault != vault) revert Errors.DifferentVaults();
63                    (uint88 amtYield, uint256 amtDittoYieldShares) = _distributeYield(assets[i]);
64                    yield += amtYield;
65                    dittoYieldShares += amtDittoYieldShares;
66                    unchecked {
67                        ++i;
68                    }
69                }
70                // claim all distributed yield
71                _claimYield(vault, yield, dittoYieldShares);
72                emit Events.DistributeYield(vault, msg.sender, yield, dittoYieldShares);
73            }
```

---

### <a id="m-04"></a>[M-04]
## **`decreaseCollateral()` allows user to decrease collateral by zero `amount`**
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/ShortRecordFacet.sol#L82
<br>

## Summary & Vulnerability Details
There is no check inside [decreaseCollateral()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/ShortRecordFacet.sol#L82-L106) to stop a user from calling it with param `amount` as zero.
```js
File: contracts/facets/ShortRecordFacet.sol

82  @>        function decreaseCollateral(address asset, uint8 id, uint88 amount)
83                external
84                isNotFrozen(asset)
85                nonReentrant
86                onlyValidShortRecord(asset, msg.sender, id)
87            {
88                STypes.ShortRecord storage short = s.shortRecords[asset][msg.sender][id];
89                short.updateErcDebt(asset);
90                if (amount > short.collateral) revert Errors.InsufficientCollateral();
91
92                short.collateral -= amount;
93
94                uint256 cRatio = short.getCollateralRatio(asset);
95                if (cRatio < LibAsset.initialMargin(asset)) {
96                    revert Errors.CollateralLowerThanMin();
97                }
98
99                uint256 vault = s.asset[asset].vault;
100               s.vaultUser[vault][msg.sender].ethEscrowed += amount;
101
102               LibShortRecord.disburseCollateral(
103                   asset, msg.sender, amount, short.zethYieldRate, short.updatedAt
104               );
105               emit Events.DecreaseCollateral(asset, msg.sender, id, amount);
106           }
```
This benefits the user in no way but causes loss of gas for him.<br><br>
The following PoC shows that it is possible to call `decreaseCollateral()` with zero amount. Add the code inside `test/Yield.t.sol` and run through command `forge test --mt testdecreaseCollateralByZeroIsAllowed -vv`.
```js
    function testdecreaseCollateralByZeroIsAllowed() public {
        fundLimitShort(DEFAULT_PRICE, 100000 ether, receiver);
        fundLimitBid(DEFAULT_PRICE, 100000 ether, sender);
        uint256 ethEscrowedInitial =
            diamond.getVaultUserStruct(vault, receiver).ethEscrowed;

        skip(yieldEligibleTime);
        generateYield();

        vm.prank(receiver);
        // @audit-info : decrease collateral by 0 amount
        diamond.decreaseCollateral(asset, Constants.SHORT_STARTING_ID, 0);

        uint256 ethEscrowedFinal = diamond.getVaultUserStruct(vault, receiver).ethEscrowed;
        assertEq(
            ethEscrowedInitial, ethEscrowedFinal, "Initial & final ethEscrowed not equal"
        );
    }
```

## Impact
No value added for the user but results in loss of gas.

## Tools Used
Manual inspection and foundry.

## Recommendations
Add a constraint that `amount` should not be zero. A custom error `Errors.PriceOrAmountIs0` already exists inside the protocol for that.
```diff
File: contracts/facets/ShortRecordFacet.sol

82            function decreaseCollateral(address asset, uint8 id, uint88 amount)
83                external
84                isNotFrozen(asset)
85                nonReentrant
86                onlyValidShortRecord(asset, msg.sender, id)
87            {
+                 if (amount == 0) revert Errors.PriceOrAmountIs0();
88                STypes.ShortRecord storage short = s.shortRecords[asset][msg.sender][id];
89                short.updateErcDebt(asset);
90                if (amount > short.collateral) revert Errors.InsufficientCollateral();
91
92                short.collateral -= amount;
93
94                uint256 cRatio = short.getCollateralRatio(asset);
95                if (cRatio < LibAsset.initialMargin(asset)) {
96                    revert Errors.CollateralLowerThanMin();
97                }
98
99                uint256 vault = s.asset[asset].vault;
100               s.vaultUser[vault][msg.sender].ethEscrowed += amount;
101
102               LibShortRecord.disburseCollateral(
103                   asset, msg.sender, amount, short.zethYieldRate, short.updatedAt
104               );
105               emit Events.DecreaseCollateral(asset, msg.sender, id, amount);
106           }
```

---

### <a id="m-05"></a>[M-05]
## **User can increase collateral by zero `amount` inside `increaseCollateral()`**
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/ShortRecordFacet.sol#L38
<br>

## Summary & Vulnerability Details
There is no check inside [increaseCollateral()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/ShortRecordFacet.sol#L38-L70) to stop a user from calling it with param `amount` as zero.<br>
If the `cRatio` is already greater than or equal to `LibAsset.primaryLiquidationCR(asset)`, this also unnecessarily calls `short.resetFlag()` [here](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/ShortRecordFacet.sol#L59-L61) which resets the `updatedAt` variable if yield is more than zero.
```js
File: contracts/facets/ShortRecordFacet.sol

38  @>        function increaseCollateral(address asset, uint8 id, uint88 amount)
39                external
40                isNotFrozen(asset)
41                nonReentrant
42                onlyValidShortRecord(asset, msg.sender, id)
43            {
44                STypes.Asset storage Asset = s.asset[asset];
45                uint256 vault = Asset.vault;
46                STypes.Vault storage Vault = s.vault[vault];
47                STypes.VaultUser storage VaultUser = s.vaultUser[vault][msg.sender];
48                if (VaultUser.ethEscrowed < amount) revert Errors.InsufficientETHEscrowed();
49
50                STypes.ShortRecord storage short = s.shortRecords[asset][msg.sender][id];
51                short.updateErcDebt(asset);
52                uint256 yield = short.collateral.mul(short.zethYieldRate);
53                short.collateral += amount;
54
55                uint256 cRatio = short.getCollateralRatio(asset);
56                if (cRatio >= Constants.CRATIO_MAX) revert Errors.CollateralHigherThanMax();
57
58                //@dev reset flag info if new cratio is above primaryLiquidationCR
59                if (cRatio >= LibAsset.primaryLiquidationCR(asset)) {
60  @>                 short.resetFlag();
61                }
62
63                yield += amount.mul(Vault.zethYieldRate);
64                short.zethYieldRate = yield.divU80(short.collateral);
65
66                VaultUser.ethEscrowed -= amount;
67                Vault.zethCollateral += amount;
68                Asset.zethCollateral += amount;
69                emit Events.IncreaseCollateral(asset, msg.sender, id, amount);
70            }
```
This benefits the user in no way but causes loss of gas for him.<br><br>
The following PoC shows that it is possible to call `increaseCollateral()` with zero amount. Add the code inside `test/Yield.t.sol` and run through command `forge test --mt testincreaseCollateralByZeroIsAllowed -vv`.
```js
    function testincreaseCollateralByZeroIsAllowed() public {
        fundLimitShort(DEFAULT_PRICE, 100000 ether, receiver);
        fundLimitBid(DEFAULT_PRICE, 100000 ether, sender);
        uint256 ethEscrowedInitial =
            diamond.getVaultUserStruct(vault, receiver).ethEscrowed;

        skip(yieldEligibleTime);
        generateYield();

        vm.prank(receiver);
        // @audit-info : increase collateral by 0 amount
        diamond.increaseCollateral(asset, Constants.SHORT_STARTING_ID, 0);

        uint256 ethEscrowedFinal = diamond.getVaultUserStruct(vault, receiver).ethEscrowed;
        assertEq(
            ethEscrowedInitial, ethEscrowedFinal, "Initial & final ethEscrowed not equal"
        );
    }
```

## Impact
No value added for the user but results in loss of gas.

## Tools Used
Manual inspection and foundry.

## Recommendations
Add a constraint that `amount` should not be zero. A custom error `Errors.PriceOrAmountIs0` already exists inside the protocol for that.
```diff
File: contracts/facets/ShortRecordFacet.sol

38  @>        function increaseCollateral(address asset, uint8 id, uint88 amount)
39                external
40                isNotFrozen(asset)
41                nonReentrant
42                onlyValidShortRecord(asset, msg.sender, id)
43            {
+                 if (amount == 0) revert Errors.PriceOrAmountIs0();
44                STypes.Asset storage Asset = s.asset[asset];
45                uint256 vault = Asset.vault;
46                STypes.Vault storage Vault = s.vault[vault];
47                STypes.VaultUser storage VaultUser = s.vaultUser[vault][msg.sender];
48                if (VaultUser.ethEscrowed < amount) revert Errors.InsufficientETHEscrowed();
49
50                STypes.ShortRecord storage short = s.shortRecords[asset][msg.sender][id];
51                short.updateErcDebt(asset);
52                uint256 yield = short.collateral.mul(short.zethYieldRate);
53                short.collateral += amount;
54
55                uint256 cRatio = short.getCollateralRatio(asset);
56                if (cRatio >= Constants.CRATIO_MAX) revert Errors.CollateralHigherThanMax();
57
58                //@dev reset flag info if new cratio is above primaryLiquidationCR
59                if (cRatio >= LibAsset.primaryLiquidationCR(asset)) {
60                    short.resetFlag();
61                }
62
63                yield += amount.mul(Vault.zethYieldRate);
64                short.zethYieldRate = yield.divU80(short.collateral);
65
66                VaultUser.ethEscrowed -= amount;
67                Vault.zethCollateral += amount;
68                Asset.zethCollateral += amount;
69                emit Events.IncreaseCollateral(asset, msg.sender, id, amount);
70            }
```

---

### <a id="m-06"></a>[M-06]
## **Due to precision loss, some of the unbacked asset debt is not socialized**
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibShortRecord.sol#L290
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallPrimaryFacet.sol#L233-L235
<br>

## Summary
Docs mention the [redistribution mechanism](https://dittoeth.com/technical/blackswan#redistribution) as:
```
When ETH value as collateral drops precipitously and the TAPP has low amounts of zETH, all orderbooks have the same mechanism in place to save the asset peg by always allowing execution of margin calls. Instead of freezing the market upon the first instance of an under-collateralized shortRecord, the unbacked asset debt is socialized over every shortRecord in proportion to each position's ercDebt balance. As long as the CR of the entire system is above 1, this allows market operations to continue indefinitely and undisturbed unless:

1. The DAO steps in and freezes/dissolves the market
2. The max ercDebtRate is reached (uint64 max ~18.45x)

ercDebtRate is calculated as ercDebt[socialized] / (ercDebt[Asset] - ercDebt[socialized]) and is written to every shortRecord record to modify each position's ercDebt amount. This functions in the same manner as yield rate, but affects the asset debt instead of zETH.
```
<br>

However due to `division before multiplication`, protocol could socialize less ercDebt than what actually needs to be socialized, and hence the protocol can face a loss.

## Vulnerability Details
`ercDebtRate` is updated in the code [here](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallPrimaryFacet.sol#L233-L235):
```js
  File: contracts/facets/MarginCallPrimaryFacet.sol

  202           function _performForcedBid(
  203               MTypes.MarginCallPrimary memory m,
  204               uint16[] memory shortHintArray
  205           ) private {
  206               uint256 startGas = gasleft();
  207               uint88 ercAmountLeft;
  208
                       ...
                       ...
                       ...
  229                   m.short.ercDebt = uint88(
  230                       m.ethDebt.div(_bidPrice.mul(1 ether + m.callerFeePct + m.tappFeePct))
  231                   ); // @dev(safe-cast)
  232                   uint96 ercDebtSocialized = ercDebtPrev - m.short.ercDebt;
  233 @>                // Update ercDebtRate to socialize loss (increase debt) to other shorts
  234 @>                s.asset[m.asset].ercDebtRate +=
  235 @>                    ercDebtSocialized.divU64(s.asset[m.asset].ercDebt - ercDebtPrev);
  236               }
  237
                       ...
                       ...
                       ...
```
<br>

L234-L235 are of the form:
```js
ercDebtRate = x / y;
```
<br>

Now to update the ercDebt, [updateErcDebt()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibShortRecord.sol#L290) is called which does the following on L290:
```js
  File: contracts/libraries/LibShortRecord.sol

  283           function updateErcDebt(address asset, address shorter, uint8 shortId) internal {
  284               AppStorage storage s = appStorage();
  285
  286               STypes.ShortRecord storage short = s.shortRecords[asset][shorter][shortId];
  287
  288               // Distribute ercDebt
  289               uint64 ercDebtRate = s.asset[asset].ercDebtRate;
  290 @>            uint88 ercDebt = short.ercDebt.mulU88(ercDebtRate - short.ercDebtRate);
  291
  292               if (ercDebt > 0) {
  293                   short.ercDebt += ercDebt;
  294                   short.ercDebtRate = ercDebtRate;
  295               }
  296           }
```
Which is equivalent to:
```js
uint88 ercDebt = a * (x/y - b);   // @audit-issue : precision loss (rounding-down) due to division before multiplication

where `a` is 'short.ercDebt', `b` is 'short.ercDebtRate' and x/y is the 's.asset[asset].ercDebtRate' (also known as `ercDebtRate` from the previous function).
```
<br>

It should ideally be:
```js
uint88 ercDebt = (a * (x - (b * y))) / y;

which is equivalent to:
uint88 ercDebt = (short.ercDebt.mulU88(ercDebtSocialized_Numerator - (short.ercDebtRate.mulU64(ercDebtSocialized_Denominator)))).divU64(ercDebtSocialized_Denominator); 
// note: I haven't checked PRBMathHelper library (out of scope) so developer should check & verify correct usage of mulU88, divU64, etc. but the logic remains as above.
```
Here, the following two values need to be propogated from [MarginCallPrimaryFacet.sol#L235](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallPrimaryFacet.sol#L235) so that they can be used in the formula above:
```js
ercDebtSocialized_Numerator = ercDebtSocialized
ercDebtSocialized_Denominator = s.asset[m.asset].ercDebt - ercDebtPrev
```

## Impact
If there are 1000 short records across which an unbacked-asset-debt needs to be socialized, such precision loss could result in the protocol socializing less ercDebt than expected to each one of them, and hence bear a loss at the end of it as some debt will still remain. (The socialized debts across 1000 shorts will not add up to be equal to the unbacked-asset-debt).

## Tools Used
Manual inspection

## Recommendations
As shown in the calculations above, store `ercDebtSocialized_Numerator` & `ercDebtSocialized_Denominator` in storage variables so that they can be used later for correct calculation.

---

### <a id="m-07"></a>[M-07]
## **DoS attack possible through `liquidateSecondary()`**
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallSecondaryFacet.sol#L40
<br>

## Summary
The external function [liquidateSecondary()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallSecondaryFacet.sol#L40) can be called at any point of time by anyone with a huge array containing all duplicate elements and passed as the `batches` param. The current structure of the code logic will continue looping till the end of the array, making the protocol service unavailable for other users.

## Vulnerability Details
Paste the following test inside the existing `test/MarginCallSecondary.t.sol` and run via `forge test --mt test_DoS_LiquidateSecondary -vv`:
```js
    function test_DoS_LiquidateSecondary() public {
        /////////////////// few setup steps ////////////////////////
        address _attacker = makeAddr("_attacker");
        deal(_cusd, _attacker, DEFAULT_AMOUNT);

        // create some active short records in the system
        fundLimitBidOpt(DEFAULT_PRICE, DEFAULT_AMOUNT, receiver);
        fundLimitShortOpt(DEFAULT_PRICE, DEFAULT_AMOUNT, sender); // short id = 2; will be liquidated later
        uint256 initialRecordCount = getShortRecordCount(sender);
        assertEq(initialRecordCount, 1);

        // change oracle price to effect cR
        _setETH(750 ether); //roughly get cratio between 1.1 and 1.5
        uint256 _cRatio = diamond.getCollateralRatio(asset, getShortRecord(sender, 2));
        assertTrue(_cRatio > 1.1 ether && _cRatio < 1.5 ether);
        ////////////////////////////////////////////////////////////

        // attack vector
        vm.startPrank(_attacker);
        uint8 attackArraySize = 100; // @audit-info : an attacker would keep this size huge
        MTypes.BatchMC[] memory batches = new MTypes.BatchMC[](attackArraySize);
        for (uint8 i; i < attackArraySize; ++i) {
            // @audit-issue : can create the array with the same shortId again & again
            batches[i] = MTypes.BatchMC({shorter: sender, shortId: 2});
        }
        diamond.liquidateSecondary(asset, batches, DEFAULT_AMOUNT, true); // @audit-issue : will result in a long loop; DoS
        vm.stopPrank();
    }
```
2 variations:
In the above PoC, an attacker just picks up a valid short eligible for secondary liquidation and then calls `liquidateSecondary()` using a huge duplicate array. In such a case, if and when control reaches [L109](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallSecondaryFacet.sol#L109) (this happens only after every element in the array has been looped through), the condition `if (liquidateAmount == liquidateAmountLeft)` is not satisfied (as a short was genuinely liquidated) and the function exits gracefully.<br>
There could be a variation where the attacker just picks up any existing short record which might not even be eligible for secondary liquidation, then creates a huge duplicate array with this id and finally calls `liquidateSecondary()`. In this case, the code still loops through the whole array but if and when control reaches L109-L110, it reverts as no short got liquidated.

## Impact
DoS attack makes the protocol service unavailable for other users.

## Tools Used
Manual inspection and foundry.

## Recommendations
Add the following constraint:
```diff
  File: contracts/facets/MarginCallSecondaryFacet.sol

  38            function liquidateSecondary(
  39                address asset,
  40                MTypes.BatchMC[] memory batches,
  41                uint88 liquidateAmount,
  42                bool isWallet
  43            ) external onlyValidAsset(asset) isNotFrozen(asset) nonReentrant {
+                   require(batches.length < 10, "batch too big");
  44                STypes.AssetUser storage AssetUser = s.assetUser[asset][msg.sender];
  45                MTypes.MarginCallSecondary memory m;
  46                uint256 minimumCR = LibAsset.minimumCR(asset);
  47                uint256 oraclePrice = LibOracle.getSavedOrSpotOraclePrice(asset);
  48                uint256 secondaryLiquidationCR = LibAsset.secondaryLiquidationCR(asset);
  49
  50                uint88 liquidatorCollateral;
```
Developer can add a duplicate array element check too.

---

### <a id="m-08"></a>[M-08]
## **Unchecked `orderHintArray` & `shortHintArray` arrays open the door to DoS attacks**
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibOrders.sol#L927-L932
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibOrders.sol#L812-L841
<br>

## Summary
Various external functions like [createLimitShort()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/ShortOrdersFacet.sol#L38-L39), [createAsk()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/AskOrdersFacet.sol#L34), [createBid()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/BidOrdersFacet.sol#L44-L45) allow the user to provide arrays `orderHintArray` or `shortHintArray` as parameters which are never checked for duplicate elements or an upper size limit. All such functions eventually call [findOrderHintId()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibOrders.sol#L927-L932) or [_updateOracleAndStartingShort()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibOrders.sol#L812-L841) where a maliciously crafted large array will be looped in its entirety. This will lead to the protocol service being unavailable for other users (Denial of Service).

## Vulnerability Details
The logic of [findOrderHintId()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibOrders.sol#L927-L932) is:
```js
  File: contracts/libraries/LibOrders.sol

  927           function findOrderHintId(
  928               mapping(address => mapping(uint16 => STypes.Order)) storage orders,
  929               address asset,
  930               MTypes.OrderHint[] memory orderHintArray
  931           ) internal returns (uint16 hintId) {
  932 @>            for (uint256 i; i < orderHintArray.length; i++) {
  933                   MTypes.OrderHint memory orderHint = orderHintArray[i];
  934                   O hintOrderType = orders[asset][orderHint.hintId].orderType;
  935 @>                if (hintOrderType == O.Cancelled || hintOrderType == O.Matched) {
  936                       emit Events.FindOrderHintId(0);
  937 @>                    continue;
  938                   } else if (
  939                       orders[asset][orderHint.hintId].creationTime == orderHint.creationTime
  940                   ) {
  941                       emit Events.FindOrderHintId(1);
  942                       return orderHint.hintId;
  943                   } else if (orders[asset][orderHint.hintId].prevOrderType == O.Matched) {
  944                       //@dev If hint was prev matched, it means that the hint was close to HEAD and therefore is reasonable to use HEAD
  945                       emit Events.FindOrderHintId(2);
  946                       return Constants.HEAD;
  947                   }
  948               }
  949               revert Errors.BadHintIdArray();
  950           }
```
A malicious user can craft an `orderHintArray` which looks something like this:
- Let `arraySize = 100000000` : some large number
- Create the array `orderHintArray` of size `arraySize`
- Fill index `0 to arraySize-2` with any `OrderHint` which has `orderType` as `Cancelled` or `Matched` : this will cause the loop to continue on to the next iteration on L935-L937. **There is no duplicate element check so you can just repeat the same element over & over again.**
- This should be enough to keep the loop going for a long period of time. But in case we want it to gracefully exit _if & when_ it ends instead of reverting, then perform the next step.
- Fill index `arraySize-1` (the last element in the array) with a valid `OrderHint` so that the function exits with a return value either on L942 or L946.
<br><br>

Similar logic exists in [_updateOracleAndStartingShort()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibOrders.sol#L812-L841):
```js
  File: contracts/libraries/LibOrders.sol

  812           function _updateOracleAndStartingShort(address asset, uint16[] memory shortHintArray)
  813               private
  814           {
  815               AppStorage storage s = appStorage();
  816               uint256 oraclePrice = LibOracle.getOraclePrice(asset);
  817               uint256 savedPrice = asset.getPrice();
  818               asset.setPriceAndTime(oraclePrice, getOffsetTime());
  819               bool shortOrdersIsEmpty = s.shorts[asset][Constants.HEAD].nextId == Constants.TAIL;
  820               if (shortOrdersIsEmpty) {
  821                   s.asset[asset].startingShortId = Constants.HEAD;
  822               } else {
  823                   if (oraclePrice == savedPrice) {
  824                       return;
  825                   }
  826                   uint16 shortHintId;
  827 @>                for (uint256 i = 0; i < shortHintArray.length;) {
  828                       shortHintId = shortHintArray[i];
  829                       unchecked {
  830                           ++i;
  831                       }
  832
  833                       {
  834                           O shortOrderType = s.shorts[asset][shortHintId].orderType;
  835 @>                        if (
  836 @>                            shortOrderType == O.Cancelled || shortOrderType == O.Matched
  837 @>                                || shortOrderType == O.Uninitialized
  838 @>                        ) {
  839 @>                            continue;
  840                           }
  841                       }
  842
  843                       uint80 shortPrice = s.shorts[asset][shortHintId].price;
  844                       uint16 prevId = s.shorts[asset][shortHintId].prevId;
  845                       //@dev: force hint to be within 1% of oracleprice
  846                       bool startingShortWithinOracleRange = shortPrice
  847                           <= oraclePrice.mul(1.01 ether)
  848                           && s.shorts[asset][prevId].price >= oraclePrice;
  849                       bool isExactStartingShort = shortPrice >= oraclePrice
  850                           && s.shorts[asset][prevId].price < oraclePrice;
  851                       bool allShortUnderOraclePrice = shortPrice < oraclePrice
  852                           && s.shorts[asset][shortHintId].nextId == Constants.TAIL;
  853
  854                       if (startingShortWithinOracleRange || isExactStartingShort) {
  855                           //@dev only consider the x% above oraclePrice if there are prev Shorts with price >= oraclePrice
  856                           s.asset[asset].startingShortId = shortHintId;
  857                           return;
  858                       } else if (allShortUnderOraclePrice) {
  859                           s.asset[asset].startingShortId = Constants.HEAD;
  860                           return;
  861                       }
  862                   }
  863
  864                   revert Errors.BadShortHint();
  865               }
  866           }
```
Here too, a malicious user can craft an `shortHintArray` which looks something like this:
- Let `arraySize = 99999999999` : some large number
- Create the array `shortHintArray` of size `arraySize`
- Fill index `0 to arraySize-2` with any `uint16` (short id) which has `orderType` as `Cancelled` or `Matched` or `Uninitialized`: this will cause the loop to continue on to the next iteration on L835-L839. There is no duplicate element check so you can just repeat the same element over & over again. **In fact, you can also keep them empty!** This is because `s.shorts[asset][short_id].orderType` where `short_id = 0` evaluates to `O.Uninitialized` or `zero`.
- This should be enough to keep the loop going for a long period of time. But in case we want it to gracefully exit _if & when_ it ends instead of reverting, then perform the next step.
- Fill index `arraySize-1` (the last element in the array) with a valid `uint16` (any active short id which does not belong to the above 3 order types would do) so that the function exits with a return value either on L857 or L860.
<br><br>

Here's an example of such an attack using the external function `createLimitShort()`. Add the following test inside `test/fork/EndToEndFork.t.sol` and run via `forge test --mt testFork_DoS_Attack -vv`:
```js
    function testFork_DoS_Attack() public {
        // Setup
        _setupBeforeDoS();

        // Workflow: DoS attack via LimitShort
        vm.startPrank(receiver);
        MTypes.OrderHint[] memory orderHints = new MTypes.OrderHint[](1);
        orderHints[0] = MTypes.OrderHint({hintId: 2, creationTime: 123});
        // ============ craft malicious array  ============
        uint16[] memory largeShortHints = new uint16[](99999); // large-sized array
        // just set the last index to something and leave the rest empty (at default value of 0)
        largeShortHints[99998] = diamond.getShortIdAtOracle(_cusd);
        // =================================================

        // will cause the protocol to enter a long loop inside `_updateOracleAndStartingShort()`
        diamond.createLimitShort(
            _cusd,
            currentPrice,
            50_000 ether,
            orderHints,
            largeShortHints,
            diamond.getAssetStruct(_cusd).initialMargin
        );
        vm.stopPrank();
    }
    
    function _setupBeforeDoS() internal {
        uint16 cusdInitialMargin = diamond.getAssetStruct(_cusd).initialMargin;
        // Workflow: Bridge - DepositEth
        // sender
        vm.startPrank(sender);
        assertEq(reth.balanceOf(_bridgeReth), 0);
        diamond.depositEth{value: 500 ether}(_bridgeReth);
        vm.stopPrank();
        // receiver
        vm.startPrank(receiver);
        assertEq(steth.balanceOf(_bridgeSteth), 0);
        diamond.depositEth{value: 500 ether}(_bridgeSteth);

        // simulate some initial orders into the system
        currentPrice = diamond.getOraclePriceT(_cusd);
        MTypes.OrderHint[] memory orderHints = new MTypes.OrderHint[](1);
        diamond.createLimitShort(
            _cusd,
            currentPrice * 3,
            2_000 ether,
            orderHints,
            shortHints,
            cusdInitialMargin
        );
        diamond.createBid(
            _cusd, currentPrice * 2, 1_000 ether, false, orderHints, shortHints
        );
        vm.stopPrank();
        // set any new price so that protocol does not use saved price
        diamond.setOracleTimeAndPrice(_cusd, currentPrice * 2); 
    }
```
<br>

If required, one can check the [long loop entered inside _updateOracleAndStartingShort()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibOrders.sol#L827) by adding a `console.log` on the next line.

## Impact
DoS attack makes the protocol service unavailable for other users.

## Tools Used
Manual inspection.

## Recommendations
- Add a constraint on the maximum size of the arrays external users can pass.
- Developer can add a duplicate array element check too.

---

### <a id="m-09"></a>[M-09]
## **Possible DoS on depositEth, withdrawal & unstaking for `BridgeReth`**
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/BridgeRouterFacet.sol#L82
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/bridges/BridgeReth.sol#L76
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/BridgeRouterFacet.sol#L112
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/BridgeRouterFacet.sol#L138
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/BridgeRouterFacet.sol#L160
<br>

## Summary
RocketPool `rETH` tokens have a [deposit delay](https://github.com/rocket-pool/rocketpool/blob/967e4d3c32721a84694921751920af313d1467af/contracts/contract/token/RocketTokenRETH.sol#L156-L172) that prevents any user who has recently deposited to transfer, mint or burn tokens. In the past this delay was set to 5760 blocks mined (aprox. 19h, considering one block per 12s). This delay can prevent DittoETH protocol users from withdrawing or unstaking if another user staked recently.

Currently this delay is zero. Any future changes made to this delay by the admins could potentially lead to a denial-of-service (even under normal flow of operations) for - 
- [depositEth()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/bridges/BridgeReth.sol#L76)
- [withdraw()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/bridges/BridgeReth.sol#L94)
- [unstake()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/bridges/BridgeReth.sol#L102)
<br>

These are major functionalities of the protocol, and therefore, it should be classified as a medium severity issue.

## Vulnerability Details
Protocol users' [deposit](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/BridgeRouterFacet.sol#L67) or [withdraw](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/BridgeRouterFacet.sol#L112) actions could prevent other users from depositing, withdrawing or unstaking `rETH` for a few hours. Given that many users would call these functions throughout the day (and that the actual transaction is executed by the protocol address from inside the `BridgeRouterFacet.sol::depositEth` or `BridgeRouterFacet.sol::withdraw` or similar), the delay would constantly reset, making the functions unusable. It's important to note that this only occurs when these functions are used through the `BridgeReth` route. If `rETH` is obtained & returned from an external pool like Uniswap, the delay is not affected.<br><br>

A malicious actor can also exploit this to be able to block all withdrawal/unstake calls. Consider the following scenario where the delay was raised again to 5760 blocks. Bob (malicious actor) calls depositEth() with the minimum amount, consequently triggering deposit to RocketPool and resetting the deposit delay. Alice tries to withdraw her funds, but during rETH transfer/burn, it fails due to the delay check, reverting the call.
If Bob manages to repeatedly deposit() the minimum amount every 19h (or any other interval less then the deposit delay), all future calls for withdrawal will revert.

## Similar past bug report with context
**First, the context & similarity with current scenario:** A bug was discovered in RocketPool's rETH tokens that can cause a denial-of-service attack on the unstake() mechanism of the Asymmetry protocol. This bug is caused by the possibility of rETH's deposit-delay set to 5760 blocks (approx. 19 hours) that prevents any user who has recently deposited to transfer or burn tokens. This delay can prevent Asymmetry protocol users from unstaking if another user staked recently. A malicious actor can exploit this bug by repeatedly staking the minimum amount every 19 hours, blocking all unstake calls. This bug could be mitigated by modifying the function to obtain rETH only through the UniswapV3 pool, as users will avoid any future issues with the deposit delay mechanism. The bug was confirmed by Asymmetry.
<br>

**Link to past bug report:** [Medium Severity Asymmetry Finance Bug](https://solodit.xyz/issues/m-08-possible-dos-on-unstake-code4rena-asymmetry-finance-asymmetry-contest-git)

## Impact
DoS attack ( _or even normal flow of operations_ ), can break direct withdrawal/unstaking of `rETH` if deposit-timelock of `rETH` is changed by RocketToken admins.

## Tools Used
Manual inspection.

## Recommendations
Change the `BridgeReth.sol` deposit + withdraw + unstaking implementations to acquire & release rETH only by swapping via an external pool like Uniswap v3.

---

### <a id="m-10"></a>[M-10]
## **`BridgeReth.sol::unstake()` is unreliable and depends on excess RocketDepositPool balance which can lead to DoS**
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/bridges/BridgeReth.sol#L102
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/BridgeRouterFacet.sol#L138
<br>

## Summary
`burn()` as used [here](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/bridges/BridgeReth.sol#L102) for rETH, targets Rocketpool which is not always solvent. The issue with this is that the [RocketPool's burn() function](https://github.com/rocket-pool/rocketpool/blob/967e4d3c32721a84694921751920af313d1467af/contracts/contract/token/RocketTokenRETH.sol#L105-L123) function only allows for excess balance to be withdrawn i.e. ETH that has been deposited by stakers but that is not yet staked on the Ethereum beacon chain. So Rocketpool allows users to burn rETH and withdraw ETH as long as the excess balance is sufficient.<br>

The issue is: If there is no excess balance because enough users burn rETH or the Minipool capacity increases, Ditto protocol rETH users can't unstake.

## Vulnerability Details
In Rocketpool, users stake 16 eth and node operators stake the remaining 16 eth. A minimum quantity of eth is left in the contract for withdrawals, but the majority of eth are tied up in staking. So users cannot redeem all rETH for eth, since most are controlled by node operators.<br>

Even after the Shanghai upgrade, rETH still remains insolvent. This is because when the liquid eth in the contract dries up, the contract has to wait for a node operator to shut down their node and free up more eth. This is profitable for node operators since an insolvent rETH contract drops the rETH price, allowing node operators to profit from the price drop. Stakers are still expected to wait until enough eth is freed up before burning rETH. Users who don't want to wait are thus encouraged to swap their rETH for eth in LPs, reducing load from the withdrawal mechanism.<br>

In this protocol, however, the only method for unstaking of rETH is the burn mechanism, which can be insolvent. Thus all users of the protocol have to wait until rETH contract has liquidity, in order to unstake.

## Similar past bug report with context
**Context & similarity with current scenario:** The Asymmetry protocol promises that a user can call [`SafETH.unstake`](https://github.com/code-423n4/2023-03-asymmetry/blob/44b5cd94ebedc187a08884a7f685e950e987261c/contracts/SafEth/SafEth.sol#L108-L129) at all times, allowing a user to burn their `SafETH` tokens and receive `ETH` in return. This requires that the derivatives held by the protocol can be withdrawn (i.e. converted to `ETH`). The `Reth` derivative works differently than the `WstETH` and `SfrxETH` derivatives. Withdrawals are made by calling the `RocketTokenRETH.burn` function, but the issue is that the `RocketTokenRETH.burn` function only allows for *excess balance* to be withdrawn. If there is no excess balance, the Asymmetry protocol is unable to operate. The solution for this issue is to have an alternative withdrawal mechanism in case the *excess balance* in the RocketDepositPool is insufficient to handle the withdrawal. This can be done by swapping the `rETH` tokens via the Uniswap pool. The bug was confirmed by Asymmetry.
<br>

**Link to past bug reports:** 
- [Report 1 - Asymmetry Finance Bug](https://github.com/code-423n4/2023-03-asymmetry-findings/issues/132)
- [Report 2 - Asymmetry Finance Bug](https://solodit.xyz/issues/h-07-rethsol-withdrawals-are-unreliable-and-depend-on-excess-rocketdepositpool-balance-which-can-brick-the-whole-protocol-code4rena-asymmetry-finance-asymmetry-contest-git)

## Impact
`BridgeReth.sol::unstake()` becomes unusable for every user.

## Tools Used
Manual inspection.

## Recommendations
The solution for this issue is to have an alternative withdrawal mechanism in case the _excess balance_ in the RocketDepositPool is insufficient to handle the withdrawal.<br>
The alternative withdrawal mechanism is to sell the rETH tokens via an external pool like Uniswap.

---

### <a id="m-11"></a>[M-11]
## **In `_performForcedBid()` during primary margin call, `m.short.ercDebt` can be calculated less than actual, causing higher than expected `ercDebtSocialized`**
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallPrimaryFacet.sol#L229-L232
<br>

## Summary
The unsafe casting of `uint256` to `uint88` can lead to under-reporting of `m.short.ercDebt` inside [_performForcedBid()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallPrimaryFacet.sol#L229-L232) while performing a primary margin call.

## Vulnerability Details
The calculation inside [_performForcedBid()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallPrimaryFacet.sol#L229-L232):
```js
  File: contracts/facets/MarginCallPrimaryFacet.sol

  229                   m.short.ercDebt = uint88(
  230                       m.ethDebt.div(_bidPrice.mul(1 ether + m.callerFeePct + m.tappFeePct))
  231                   ); // @dev(safe-cast)
  232                   uint96 ercDebtSocialized = ercDebtPrev - m.short.ercDebt;
```

The data types of these variables are:
```js
uint88 ercDebt
uint256 ethDebt
uint80 _bidPrice
uint256 callerFeePct
uint256 tappFeePct
```
<br>

The casting of `uint256 ethDebt` to `uint88` can cause truncation of the value and hence the real ercDebt, as shown in the following PoC. This reduced `m.short.ercDebt` causes an increased `ercDebtSocialized` in Line 232.

## PoC
Create a new file under `test/` folder named `MathCastingPerformForcedBid.t.sol` and run the following code via `forge test --mt test_casting_performForcedBid -vv`:
```js
// SPDX-License-Identifier: GPL-3.0-only
pragma solidity 0.8.21;

import {U256, U88, U80} from "contracts/libraries/PRBMathHelper.sol";
import {console} from "contracts/libraries/console.sol";

contract MathCastingPerformForcedBid {
    using U256 for uint256;
    using U88 for uint88;
    using U80 for uint80;

    uint256 private ethDebt;
    uint80 private bidPrice;

    function setUp() public {
        // we could easily keep the below value of `ethDebt` as `type(uint256).max`, but wanted to show that
        // it starts truncating values way before that.
        ethDebt = 1208925819614629174706175 * 0.000001 ether; // equivalent to `type(uint80).max * 0.000001 ether`
        bidPrice = type(uint80).max;
    }

    /* solhint-disable no-console */
    function test_casting_performForcedBid() public view {
        uint256 ercDebt256 = (ethDebt.div(bidPrice.mul(1 ether + 0 + 0))); // assuming  `m.callerFeePct = m.tappFeePct = 0` for a simpler calculation
        uint88 ercDebt88 = uint88(ethDebt.div(bidPrice.mul(1 ether + 0 + 0)));
        console.log("ercDebt256 =");
        console.log(ercDebt256);
        console.log("\nercDebt88 =");
        console.log(ercDebt88);
    }
}
```
<br>

Output:
```text
Logs:
  ercDebt256 =
  1000000000000000000000000000000

ercDebt88 =
  53933267234082950232408064
```
`ercDebt256` is the actual value while `ercDebt88` is the truncated one used by the protocol.

## Impact
- Higher `ercDebtSocialized` to other users and hence loss of funds for them.
- This also throws off other calculations since lesser `ercAmount` is filled than expected via the forced bid.

## Tools Used
Manual inspection, forge test.

## Recommendations
It is recommended to specify a correct upper limit for these values using a `require()` statement inside the function, so that they do not eventually overflow on performing arithmetic operations but instead revert at the start itself.

---

### <a id="m-12"></a>[M-12]
## **Unsafe typecasting of `uint256` to `uint88` can result in protocol losing high amount in gasFee**
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallPrimaryFacet.sol#L250-L252
<br>

## Summary
When user has consumed a lot of gas and value of `gasUsed` is high, they are incorectly charged a very less gasFee due to unsafe typecasting from `uint256` to `uint88`. As per [protocol comments](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallPrimaryFacet.sol#L251): `By basing gasFee off of baseFee instead of priority, adversaries are prevent from draining the TAPP` - this is violated in such cases.

## Vulnerability Details
`m.gasFee` is [calculated as](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallPrimaryFacet.sol#L250-L252):
```js
  File: contracts/facets/MarginCallPrimaryFacet.sol

  250               //@dev manually setting basefee to 1,000,000 in foundry.toml;
  251               //@dev By basing gasFee off of baseFee instead of priority, adversaries are prevent from draining the TAPP
  252 @>            m.gasFee = uint88(gasUsed * block.basefee); // @dev(safe-cast)
```
<br>

Data types of these variables are:
> `uint88` m.gasFee <br>
> `uint256` gasUsed
>> block.basefee is `uint256`
<br>

`block.basefee` is set as 1_000_000_000 [on local chain](https://github.com/Cyfrin/2023-09-ditto/blob/main/foundry.toml#L17) (is even higher on mainnet). For any value of around `gasUsed >= 0.30949 ether`, the value of `m.gasFee` would be calculated as disproportionately low.<br>
Note that these lines of code are inside the `_performForcedBid()` function which calls `createForcedBid()` on [Line 239](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarginCallPrimaryFacet.sol#L239) which has a `shortHintArray` param. An attacker could pass a large array here and cause such high gas usage. 

## PoC
Create a new file under `test/` folder named `MathCastingGasFee.t.sol` and run the following code via `forge test --mt test_casting_gas -vv`:
```js
// SPDX-License-Identifier: GPL-3.0-only
pragma solidity 0.8.21;

import {console} from "contracts/libraries/console.sol";

contract MathCastingGasFee {
    uint256 private gasUsed;

    function setUp() public {
        // we could easily keep the below value of `gasUsed` as `type(uint256).max`, but
        // let's demo with something smaller
        gasUsed = 0.30949 ether;
    }

    /* solhint-disable no-console */
    function test_casting_gas() public view {
        console.log("Current block.basefee =", block.basefee); // On mainnet fork, it's around 8497767262. On local it has been set to 1000000000.
        uint256 gasFee256 = (gasUsed * block.basefee);
        uint88 gasFee88 = uint88(gasUsed * block.basefee);
        console.log("\ngasFee256 =");
        console.log(gasFee256);
        console.log("\ngasFee88 =");
        console.log(gasFee88);
    }
}
```
<br>

Output:
```text
Logs:
  Current block.basefee = 1000000000

gasFee256 =
  309490000000000000000000000

gasFee88 =
  4990178654931275218944
```
`gasFee256` is the actual value while `gasFee88` is the down-casted one used by the protocol.

## Impact
`By basing gasFee off of baseFee instead of priority, adversaries are prevent from draining the TAPP` - this protocol objective is violated when adversary consumes a lot of gas.

## Tools Used
Manual inspection.

## Recommendations
Use a safe typecasting library or a custom function which reverts in such cases. Example of such a function could be:
```js
    function safeU88(uint256 n) internal pure returns (uint88) {
        if (n > type(uint88).max) revert Errors.InvalidUInt88();
        return uint88(n);
    }
```
<br>

And then use it in code:
```diff
  File: contracts/facets/MarginCallPrimaryFacet.sol

  250               //@dev manually setting basefee to 1,000,000 in foundry.toml;
  251               //@dev By basing gasFee off of baseFee instead of priority, adversaries are prevent from draining the TAPP
- 252               m.gasFee = uint88(gasUsed * block.basefee); // @dev(safe-cast)
+ 252               m.gasFee = safeU88(gasUsed * block.basefee); // @audit : safe-casted now
```

---

### <a id="m-13"></a>[M-13]
## **The check inside `createVault()` for presence of already created vaults is insufficient**
#### https://github.com/Cyfrin/2023-09-ditto/blob/a93b4276420a092913f43169a353a6198d3c21b9/contracts/facets/OwnerFacet.sol#L140
<br>

## Summary & Vulnerability Details
[Doc](https://dittoeth.com/technical/concepts#vaults) mentions:
> - There is only a single `Vault` at launch, but a new `Vault` can be made with `createVault`
<br>

The current `carbon zETH` token is mapped to `Vault.CARBON` having vault id as 1. This is through the [mapping](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/AppStorage.sol#L19): `mapping(address zeth => uint256 vault) zethVault`. The [check in createVault()](https://github.com/Cyfrin/2023-09-ditto/blob/a93b4276420a092913f43169a353a6198d3c21b9/contracts/facets/OwnerFacet.sol#L140) ensures same `carbon zETH` token can not map to another vault. 
A new vault should be created with a new zETH token. 
```js
  File: contracts/facets/OwnerFacet.sol

  135           function createVault(
  136 @>            address zeth,
  137               uint256 vault,
  138               MTypes.CreateVaultParams calldata params
  139           ) external onlyDAO {
  140 @>            if (s.zethVault[zeth] != 0) revert Errors.VaultAlreadyCreated();
  141               s.zethVault[zeth] = vault;
  142               _setTithe(vault, params.zethTithePercent);
  143               _setDittoMatchedRate(vault, params.dittoMatchedRate);
  144               _setDittoShorterRate(vault, params.dittoShorterRate);
  145               emit Events.CreateVault(zeth, vault);
  146           }
```
<br>

Imagine the protocol owner or `DAO` wants to create a new vault with a new token. Let's call this `nzETH` (new zETH). So `DAO` would ideally call the function in the following way to create a new vault:
```js
MTypes.CreateVaultParams memory newVaultParams;
createVault(address(nzETH), 5, newVaultParams);
````
This would create a new vault with vault id 5. No problems here.
<br>

Suppose now `DAO` wants to create another new vault for an another token `pzETH` (popular zETH). He wishes to use vault id 8 for this purpose. He calls the following, _making a user input error_:
```js
MTypes.CreateVaultParams memory popularVaultParams;
createVault(address(pzETH), 5, popularVaultParams);  // @audit-info : DAO passes a pre-existing vault id (5) by mistake!
```

## Impact
The impact of this input error is high:
- First, since the code only checks `if (s.zethVault[address(pzETH)] != 0) revert Errors.VaultAlreadyCreated();`, this won't revert and `pzETH` will be mapped to vault id in the next line via `s.zethVault[address(pzETH)] = 5;`.<br>
Vault 5 now has 2 tokens. Also, params for vault 5 like `tithe`, `dittoMatchedRate` & `dittoShorterRate` are now reset using `popularVaultParams` instead of the old `newVaultParams`.<br>This could be problematic because now an external user can call [depositZETH()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/VaultFacet.sol#L35) with `pzETH`.
```js
depositZETH(address(pzETH), 1000_000);
```
and this will pass all the checks there, burn some `pzETH` from external user's wallet, and increase user's `ethEscrowed` as if he deposited `carbon zETH`.
<br>

- The second impact is: Since there is no `unmapTokenFromVault()` or similar function in the protocol, **there is no way for `DAO` now to create another vault using `pzETH`**. Calling `createVault()` will fail at `if (s.zethVault[address(pzETH)] != 0) revert Errors.VaultAlreadyCreated();` since it has a value of 5 now.<br>He also can not _"unmerge"_ these 2 tokens within the vault.
<br><br>

Owner's input error --> High impact on the protocol with no way to correct it, hence raising as medium severity.
<br>
<br>


# PoC
Paste the following code inside `test/Owner.t.sol` and run via `forge test --mt test_InsufficientCheckInCreateVault -vv`. The test would revert with `Reason: VaultAlreadyCreated()` -
```js
    function test_InsufficientCheckInCreateVault() public {
        vm.startPrank(owner);

        // Create a new vault with id 5 for nzETH (new zETH)
        address nzETHaddress = makeAddr("nzETH");
        MTypes.CreateVaultParams memory newVaultParams;
        diamond.createVault(nzETHaddress, 5, newVaultParams);

        // Create a new vault with id 5 for nzETH (new zETH)
        address pzETHaddress = makeAddr("pzETH");
        MTypes.CreateVaultParams memory popularVaultParams;
        // input error - passing vault id as 5 again. Will NOT revert with `Errors.VaultAlreadyCreated`.
        diamond.createVault(pzETHaddress, 5, popularVaultParams);

        // DAO can't correct his mistake now
        diamond.createVault(pzETHaddress, 8, popularVaultParams); // passing correct vault id

        vm.stopPrank();
    }
```

## Tools Used
Manual inspection, forge test.

## Recommendations
Maintain a mapping of already created vault ids and check that too.

---

### <a id="m-14"></a>[M-14]
## **`updateYield()` will permanently stop updating yield due to unsafe typecasting of `uint256` to `uint88`**
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibVault.sol#L68
<br>

## Summary & Vulnerability Details
[updateYield()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibVault.sol#L68) calculates the `yield` as `zethTotalNew - zethTotal`. If `zethTotalNew` is less then `zethTotal`, then no yield is accrued.
```js
  File: contracts/libraries/LibVault.sol

  55            /**
  56             * @notice Updates the vault yield rate from staking rewards earned by bridge contracts holding LSD
  57             * @dev Does not distribute yield to any individual owner of shortRecords
  58             *
  59             * @param vault The vault that will be impacted
  60             */
  61
  62            function updateYield(uint256 vault) internal {
  63                AppStorage storage s = appStorage();
  64
  65                STypes.Vault storage Vault = s.vault[vault];
  66                STypes.VaultUser storage TAPP = s.vaultUser[vault][address(this)];
  67                // Retrieve vault variables
  68  @>            uint88 zethTotalNew = uint88(getZethTotal(vault)); // @dev(safe-cast)
  69                uint88 zethTotal = Vault.zethTotal;
  70                uint88 zethCollateral = Vault.zethCollateral;
  71                uint88 zethTreasury = TAPP.ethEscrowed;
  72
  73                // Calculate vault yield and overwrite previous total
  74                if (zethTotalNew <= zethTotal) return;
  75                uint88 yield = zethTotalNew - zethTotal;
  76                Vault.zethTotal = zethTotalNew;
                    ...
                    ...
                    ...
```
On L68, [getZethTotal()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/libraries/LibVault.sol#L42) returns a `uint256` which is down-casted to `uint88` in an unsafe way. Imagine this:
- `zethTotal` in the vault is equal to `type(uint88).max`
- After a few moments, when `updateYield()` is called, it internally calls `getZethTotal()` to get a zETH balance which has crossed over to a value of `type(uint88).max + 100`
- Since L68 typecasts it to `uint88`, `zethTotalNew` will appear just as 99.
- Function will return with no accrued yield on L74.
- Function will never update the new `Vault.zethTotal` on L76 and hence this will keep on happening again & again each time `updateYield` is called.

## PoC
Create a new file under `test/` folder named `MathCastingUpdateYield.t.sol` and run the following code via `forge test --mt test_casting_updateYield -vv`:
```js
// SPDX-License-Identifier: GPL-3.0-only
pragma solidity 0.8.21;

contract MathCastingUpdateYield {
    uint88 private zethTotal;

    function setUp() public {
        zethTotal = type(uint88).max;
    }

    function mockGetZethTotal() public pure returns (uint256) {
        return 309485009821345068724781055 + 100; // type(uint88).max + 100
    }

    function test_casting_updateYield() public view {
        require(uint88(mockGetZethTotal()) > zethTotal, "Downcasting issue");
    }
}
```
<br>

Output:
```text
Running 1 test for test/MathCastingUpdateYield.t.sol:MathCastingUpdateYield
[FAIL. Reason: Downcasting issue] test_casting_updateYield() (gas: 2628)
Test result: FAILED. 0 passed; 1 failed; 0 skipped; finished in 466.81µs
```

## Impact
`updateYield()` breaks permanently due to high zETH in the vault, but protocol continues to operate without any reverts. Difficult to take any remedial action with no warning system.

## Tools Used
Manual inspection.

## Recommendations
Use a safe typecasting library or a custom function which reverts in such cases. Example of such a function could be:
```js
    function safeU88(uint256 n) internal pure returns (uint88) {
        if (n > type(uint88).max) revert Errors.InvalidUInt88();
        return uint88(n);
    }
```
<br>

And then use it in code:
```diff
  File: contracts/libraries/LibVault.sol

  55            /**
  56             * @notice Updates the vault yield rate from staking rewards earned by bridge contracts holding LSD
  57             * @dev Does not distribute yield to any individual owner of shortRecords
  58             *
  59             * @param vault The vault that will be impacted
  60             */
  61
  62            function updateYield(uint256 vault) internal {
  63                AppStorage storage s = appStorage();
  64
  65                STypes.Vault storage Vault = s.vault[vault];
  66                STypes.VaultUser storage TAPP = s.vaultUser[vault][address(this)];
  67                // Retrieve vault variables
- 68                uint88 zethTotalNew = uint88(getZethTotal(vault)); // @dev(safe-cast)
+ 68                uint88 zethTotalNew = safeU88(getZethTotal(vault)); // @audit : safe-casted now
  69                uint88 zethTotal = Vault.zethTotal;
  70                uint88 zethCollateral = Vault.zethCollateral;
  71                uint88 zethTreasury = TAPP.ethEscrowed;
  72
  73                // Calculate vault yield and overwrite previous total
  74                if (zethTotalNew <= zethTotal) return;
  75                uint88 yield = zethTotalNew - zethTotal;
  76                Vault.zethTotal = zethTotalNew;
```

<br><br>

## **LOW-SEVERITY BUGS**
---

### <a id="l-01"></a>[L-01]
## **Users get less than expected `amtZeth` (ethEscrowed) on calling `redeemErc()`**
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarketShutdownFacet.sol#L93-L100
#### https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarketShutdownFacet.sol#L80
<br>

## Summary
In the event of market shutdown, users can call [redeemErc()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarketShutdownFacet.sol#L80). The user gets less than expected `amtZeth` (ethEscrowed) due to division before multiplication by the protocol.

## Vulnerability Details
In the event of market shutdown, users can call [redeemErc()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarketShutdownFacet.sol#L80):
```js
  File: contracts/facets/MarketShutdownFacet.sol

  57            /**
  58             * @notice Allows user to redeem erc from their wallet and/or escrow at the oracle price of market shutdown
  59             * @dev Market must be permanently frozen, redemptions drawn from the combined collateral of all short records
  60             *
  61             * @param asset The market that will be impacted
  62             */
  63
  64            function redeemErc(address asset, uint88 amtWallet, uint88 amtEscrow)
  65                external
  66                isPermanentlyFrozen(asset)
  67                nonReentrant
  68            {
  69                if (amtWallet > 0) {
  70                    asset.burnMsgSenderDebt(amtWallet);
  71                }
  72
  73                if (amtEscrow > 0) {
  74                    s.assetUser[asset][msg.sender].ercEscrowed -= amtEscrow;
  75                }
  76
  77                uint88 amtErc = amtWallet + amtEscrow;
  78  @>            uint256 cRatio = _getAssetCollateralRatio(asset);
  79                // Discount redemption when asset is undercollateralized
  80  @>            uint88 amtZeth = amtErc.mulU88(LibOracle.getPrice(asset)).mulU88(cRatio);
  81                s.vaultUser[s.asset[asset].vault][msg.sender].ethEscrowed += amtZeth;
  82                emit Events.RedeemErc(asset, msg.sender, amtWallet, amtEscrow);
  83            }
```
<br>

Line 80 contains the calculation `uint88 amtZeth = amtErc.mulU88(LibOracle.getPrice(asset)).mulU88(cRatio);`, where cRatio is calculated on Line 78 as `uint256 cRatio = _getAssetCollateralRatio(asset)` calling the private function in the same file, [_getAssetCollateralRatio()](https://github.com/Cyfrin/2023-09-ditto/blob/main/contracts/facets/MarketShutdownFacet.sol#L99):
```js
  File: contracts/facets/MarketShutdownFacet.sol

  85            /**
  86             * @notice Computes the c-ratio of an asset class
  87             *
  88             * @param asset The market that will be impacted
  89             *
  90             * @return cRatio
  91             */
  92
  93            function _getAssetCollateralRatio(address asset)
  94                private
  95                view
  96                returns (uint256 cRatio)
  97            {
  98                STypes.Asset storage Asset = s.asset[asset];
  99  @>            return Asset.zethCollateral.div(LibOracle.getPrice(asset).mul(Asset.ercDebt));
  100           }
```
<br>

Simplifying, the calculation for `amtZeth` on Line 80 becomes:
```
amtZeth = amtErc * price * (cRatio)   =>   amtErc * price * ( zethCollateral / (price * ercDebt) )
```

Division operation is done before multiplication. Correct form should be:
```
amtZeth = ( amtErc * price *  zethCollateral ) / (price * ercDebt) 
```

## Impact
The user gets less than expected `amtZeth` (ethEscrowed)

## Tools Used
Manual inspection.

## Recommendations
Perform multiplication before division as shown in the above calculation.

