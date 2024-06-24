# Leaderboard
[Zivoe Results](https://audits.sherlock.xyz/contests/280/leaderboard)<br>

`Rank 16 / 358`

# Audited Code Repo
### [Sherlock: Zivoe](https://github.com/sherlock-audit/2024-03-zivoe)

<br>

#### [Zivoe Final Report](https://audits.sherlock.xyz/contests/280/report)

<br>

# <a id="summaryTable"></a>Bugs Filed & Their Status

| # | Bug ID | Name | URL | Adjudged Status | Count of dups (excluding current submission) |
|--------|--------|------|:------:|-----------------:|:-----------------------------------:|
| 1 | [H-01](#h-01)  | Incorrect interest calculation for late repayment | [50](https://github.com/sherlock-audit/2024-03-zivoe-judging/issues/50) | Rejected | - |
| 2 | [H-02](#h-02)  | distributeYield() calls earningsTrancheuse() with outdated emaSTT & emaJTT while calculating senior & junior tranche yield distributions | [54](https://github.com/sherlock-audit/2024-03-zivoe-judging/issues/54) | Accepted as Medium; Selected Report | 2 |
| 3 | [H-03](#h-03)  | Protocol charges incorrect interest for the first loan term | [102](https://github.com/sherlock-audit/2024-03-zivoe-judging/issues/102) | Rejected | - |
| 4 | [H-04](#h-04)  | totalSupply is incorrectly calculated during revokeVestingSchedule() | [119](https://github.com/sherlock-audit/2024-03-zivoe-judging/issues/119) | Accepted as High | 35 |
| 5 | [H-05](#h-05)  | While combining loans, APR is incorrectly calculated | [231](https://github.com/sherlock-audit/2024-03-zivoe-judging/issues/231) | Rejected | - |
| 6 | [H-06](#h-06)  | depositReward() function reduces rewardRate incorrectly causing delayed reward distribution and can be used by a griefer | [289](https://github.com/sherlock-audit/2024-03-zivoe-judging/issues/289) | Accepted as High | 46 |
| 7 | [H-07](#h-07)  | OCL_ZVE::forwardYield() is susceptible to price manipulation attack due to the logic inside fetchBasis() | [466](https://github.com/sherlock-audit/2024-03-zivoe-judging/issues/466) | Accepted as Medium | 12 |
| 8 | [M-01](#m-01)  | TLC's update of critical params inside ZivoeYDL get applied retrospectively, causing loss of pending yield | [188](https://github.com/sherlock-audit/2024-03-zivoe-judging/issues/188) | Rejected | - |
| 9 | [M-02](#m-02)  | Neither approveCombine() nor applyCombine() checks if late fee is applicable on any of the constituent loans, which could have even accrued after underwriter's call to approveCombine() | [237](https://github.com/sherlock-audit/2024-03-zivoe-judging/issues/237) | Rejected | - |
|10 | [M-03](#m-03)  | No slippage protection inside OCL_ZVE::_forwardYield() while removing liquidity | [350](https://github.com/sherlock-audit/2024-03-zivoe-judging/issues/350) | Rejected | - |
|11 | [M-04](#m-04)  | Protocol supports rebasing token OUSD but calculates basis & yield incorrectly in `OCY_OUSD.sol` | [387](https://github.com/sherlock-audit/2024-03-zivoe-judging/issues/387) | Rejected | - |


<br>

## **HIGH-SEVERITY BUGS**

---

### <a id="h-01"></a>[H-01]
## **Incorrect interest calculation for late repayment**
<br>

## Summary
The [amountOwed()](https://github.com/sherlock-audit/2024-03-zivoe/blob/main/zivoe-core-foundry/src/lockers/OCC/OCC_Modular.sol#L452-L453) function is responsible for calculating `lateFee` if it is past `loans[id].paymentDueBy`. The function uses **_only_** `loans[id].APRLateFee` to calculate the lateFee for the duration of `block.timestamp - loans[id].paymentDueBy` on [L453](https://github.com/sherlock-audit/2024-03-zivoe/blob/main/zivoe-core-foundry/src/lockers/OCC/OCC_Modular.sol#L453). No `APR` interest is charged for this additional duration. This results in the protocol receiving lower interest than it should.
<br>

It's also good to note here that this `APRLateFee` is the **_additional rate_** charged on top of the usual `APR` and **_not inclusive of_** the ongoing `APR`. This was confirmed by the sponsors in this private thread: [APRLateFee thread](https://gist.github.com/t0x1cC0de/f3ee8739e722f87ef9a323da067addf1) and is also obvious from the way the current code logic is. <br>
Had `APRLateFee` been inlcusive of `APR`, the borrower would have been incorrectly charged a higher interest amount for every late payment - one at the time of the repayment for the delayed time-period, and then again for the full payment interval at the time of the next repayment (henceforth referred to as **_double-charging_**). So it makes sense that `APRLateFee` is the 'additional' rate on top of the APR. More details below.

## Code Snippet
```js
  File: src/lockers/OCC/OCC_Modular.sol

  440:              function amountOwed(uint256 id) public view returns (
  441:                  uint256 principal, uint256 interest, uint256 lateFee, uint256 total
  442:              ) {
  443:                  // 0 == Bullet.
  444:                  if (loans[id].paymentSchedule == 0) {
  445:                      if (loans[id].paymentsRemaining == 1) { principal = loans[id].principalOwed; }
  446:                  }
  447:                  // 1 == Amortization (only two options, use else here).
  448:                  else { principal = loans[id].principalOwed / loans[id].paymentsRemaining; }
  449:          
  450: @--->            // Add late fee if past loans[id].paymentDueBy.
  451:                  if (block.timestamp > loans[id].paymentDueBy && loans[id].state == LoanState.Active) {
  452: @--->                lateFee = loans[id].principalOwed * (block.timestamp - loans[id].paymentDueBy) *
  453: @--->                    loans[id].APRLateFee / (86400 * 365 * BIPS);  
  454:                  }
  455: @--->            interest = loans[id].principalOwed * loans[id].paymentInterval * loans[id].APR / (86400 * 365 * BIPS); 
  456:                  total = principal + interest + lateFee;
  457:              }
```

## Vulnerability Detail
- Let's suppose the following loan parameters:
  - Loan amount ($principalOwed$) = 20
  - Term = 2
  - Payment interval = 10 days (This is just for simplicity of example; can also be kept as 7 || 14 || 28 || 91 || 364)
  - APR = 200 bips per day (2%)
  - APRLateFee = 100 bips per day (1%)
  - First due date = $10^{th}$ $January$
  - Second due date = $20^{th}$ $January$
  - Borrower misses the first due date and makes a payment on $16^{th}$ $January$ i.e. late by $6$ $days$.

- Let's now check the calculations for the total loan amount payable:

| Loan Component  | Expected Calculation | Current Protocol Calculation |  Protocol's Calculation Correct/Incorrect ?  | Comment |
|-----------------|:--------------------|:----------------------------|:--------------------------------------------:|:---:|
| `principalOwed` on 16th Jan |  20    | 20  | Correct | |
| `principal` payable on 16th Jan |  20 / 2 = 10    | 10  | Correct | |
| `interest` on 16th Jan  | 20 * (10 days + 6 days) * 2%    | 20 * (10 days) * 2% | Incorrect   | Protocol charges **_less_** by an amount of `20 * (6 days) * 2%` |
| `lateFee`         | 20 * 6 days * 1%    | 20 * 6 days * 1%            | Correct                 | |
| ...         | ...    | ...            | ...                 | ... |
| `principalOwed` on 20th Jan |  20 - 10 = 10    | 10  | Correct | |
| `principal` payable on 20th Jan |  10 / 1 = 10    | 10  | Correct | |
| `interest` on 20th Jan  | 10 * (10 days - 6 days) * 2%    | 10 * (10 days) * 2% | Incorrect   | Protocol charges **_more_** by an amount of `10 * (6 days) * 2%` |

Hence loss for the protocol = `( 20 * (6 days) * 2% ) - ( 10 * (6 days) * 2% )` = 1.2
<br>

It's **critical to note here** that if the late payment occurs on the **_final payment_** of the loan the loss is even greater. In the above example, it would be `( 20 * (6 days) * 2% )` = 2.4
<br>
Here **"final payment"** refers to the last payment made by the borrower which results in closure of the loan. This could either be the last term's payment in the regular schedule, or it could be an early pay off of the loan by the borrower while calling [callLoan()](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCC/OCC_Modular.sol#L489-L492).

## Proof
It's interesting to note that to remove the incorrect calculation of **_double-charging_** of interest, a change was made in this PR: [PR 132](https://github.com/Zivoe/zivoe-core-foundry/pull/132/files#diff-3d0bf62613ebc697e89e5dd91dd9994a1e7c2fa33ff73a7751abe2e1bd375e0cR373). <br>
The [comment by the developer](https://github.com/Zivoe/zivoe-core-foundry/pull/132#issue-1691745976) mentions the [following intention](https://github.com/Zivoe/zivoe-core-foundry/pull/132#:~:text=Removes%20excess%20from%20the%20APRLateFee%20calculation%20in%20OCC_Modular):
```text
 - Removes excess from the APRLateFee calculation in OCC_Modular
```
but the elimination of `APR` term altogether from the calculation is an incorrect way to approach it.

## Impact
`amountOwed()` is internally called by other functions, namely:
- [callLoan()](https://github.com/sherlock-audit/2024-03-zivoe/blob/main/zivoe-core-foundry/src/lockers/OCC/OCC_Modular.sol#L500) which is callable by any borrower to pay off their loan. This means that if the borrower is late, then they are able to close the loan by paying off less than they should have and hence causing a loss of funds for the protocol.
- [makePayment()](https://github.com/sherlock-audit/2024-03-zivoe/blob/main/zivoe-core-foundry/src/lockers/OCC/OCC_Modular.sol#L578) which can be used to close the loan i.e. [loans[id].state = LoanState.Repaid](https://github.com/sherlock-audit/2024-03-zivoe/blob/main/zivoe-core-foundry/src/lockers/OCC/OCC_Modular.sol#L595) if it's the [last payment of the schedule](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCC/OCC_Modular.sol#L594), causing a loss to the protocol.
- [processPayment()](https://github.com/sherlock-audit/2024-03-zivoe/blob/main/zivoe-core-foundry/src/lockers/OCC/OCC_Modular.sol#L650) which is used by underwriters - same behaviour as `makePayment()`.

Invoking any of these functions causes loss of funds for the protocol.

## Tool used
Manual Review

## Recommendation
Two changes need to be made:
- The first is inside the `amountOwed()` function where we need to undo the change made in [PR 132](https://github.com/Zivoe/zivoe-core-foundry/pull/132/files#diff-3d0bf62613ebc697e89e5dd91dd9994a1e7c2fa33ff73a7751abe2e1bd375e0cR373).

- Secondly, the protocol needs to track if a late payment was made in the previous term so that it can reduce the duration in this term accordingly for which interest needs to be calculated.

[Back to Top](#summaryTable)
 
---

### <a id="h-02"></a>[H-02]
## **distributeYield() calls earningsTrancheuse() with outdated emaSTT & emaJTT while calculating senior & junior tranche yield distributions**
<br>

## Summary
The [distributeYield()](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeYDL.sol#L229-L239) function internally calls `earningsTrancheuse()` on [L232](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeYDL.sol#L232) which calculates & returns the `_seniorTranche` & `_juniorTranche` yield distribution. However, this call results in `earningsTrancheuse()` using [outdated values of](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeYDL.sol#L461-L462) `emaSTT` & `emaJTT` on L461-L462 as these variables are updated only on [L238-L239](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeYDL.sol#L238-L239) _after_ the call to `earningsTrancheuse()` is concluded.

## Code Snippet
```js
  File: src/ZivoeYDL.sol

  213:              function distributeYield() external nonReentrant {
  214:                  require(unlocked, "ZivoeYDL::distributeYield() !unlocked"); 
  215:                  require(
  216:                      block.timestamp >= lastDistribution + daysBetweenDistributions * 86400, 
  217:                      "ZivoeYDL::distributeYield() block.timestamp < lastDistribution + daysBetweenDistributions * 86400"
  218:                  );
  219:          
  220:                  // Calculate protocol earnings.
  221:                  uint256 earnings = IERC20(distributedAsset).balanceOf(address(this));
  222:                  uint256 protocolEarnings = protocolEarningsRateBIPS * earnings / BIPS;
  223:                  uint256 postFeeYield = earnings.floorSub(protocolEarnings);
  224:          
  225:                  // Update timeline.
  226:                  distributionCounter += 1;
  227:                  lastDistribution = block.timestamp;
  228:          
  229:                  // Calculate yield distribution (trancheuse = "slicer" in French).
  230:                  (
  231:                      uint256[] memory _protocol, uint256 _seniorTranche, uint256 _juniorTranche, uint256[] memory _residual
  232: @--->            ) = earningsTrancheuse(protocolEarnings, postFeeYield); 
  233:          
  234:                  emit YieldDistributed(_protocol, _seniorTranche, _juniorTranche, _residual);
  235:                  
  236:                  // Update ema-based supply values. 
  237:                  (uint256 aSTT, uint256 aJTT) = IZivoeGlobals_YDL(GBL).adjustedSupplies();
  238: @--->            emaSTT = MATH.ema(emaSTT, aSTT, retrospectiveDistributions.min(distributionCounter));
  239: @--->            emaJTT = MATH.ema(emaJTT, aJTT, retrospectiveDistributions.min(distributionCounter));
                        ...
                        ...
```

and

```js
  File: src/ZivoeYDL.sol

  447:              function earningsTrancheuse(uint256 yP, uint256 yD) public view returns (
  448:                  uint256[] memory protocol, uint256 senior, uint256 junior, uint256[] memory residual
  449:              ) {
  450:                  protocol = new uint256[](protocolRecipients.recipients.length);
  451:                  residual = new uint256[](residualRecipients.recipients.length);
  452:                  
  453:                  // Accounting for protocol earnings.
  454:                  for (uint256 i = 0; i < protocolRecipients.recipients.length; i++) {
  455:                      protocol[i] = protocolRecipients.proportion[i] * yP / BIPS;
  456:                  }
  457:          
  458:                  // Accounting for senior and junior earnings.
  459:                  uint256 _seniorProportion = MATH.seniorProportion(
  460:                      IZivoeGlobals_YDL(GBL).standardize(yD, distributedAsset),
  461: @--->                MATH.yieldTarget(emaSTT, emaJTT, targetAPYBIPS, targetRatioBIPS, daysBetweenDistributions),
  462: @--->                emaSTT, emaJTT, targetAPYBIPS, targetRatioBIPS, daysBetweenDistributions
  463:                  );
  464:                  senior = (yD * _seniorProportion) / RAY;
  465:                  junior = (yD * MATH.juniorProportion(emaSTT, emaJTT, _seniorProportion, targetRatioBIPS)) / RAY; 
  466:                                                                                                                    
  467:                  // Handle accounting for residual earnings.
  468:                  yD = yD.floorSub(senior + junior);
  469:                  for (uint256 i = 0; i < residualRecipients.recipients.length; i++) {
  470:                      residual[i] = residualRecipients.proportion[i] * yD / BIPS;
  471:                  }
  472:              }
```

## Vulnerability Detail
As the [docs explain](https://docs.zivoe.com/user-docs/yield-distribution#liquidity-providers:~:text=Returns%20are%20calculated%20using%20the%20adjusted%20supply%20of%20tranche%20tokens%2C%20with%20an%20Exponential%20Moving%20Average%20(EMA)%20playing%20a%20significant%20role), the EMA plays a significant role in yield distribution calculations.

> Returns are calculated using the adjusted supply of tranche tokens, with an Exponential Moving Average (EMA) playing a significant role. 

> The EMA smoothens the change in the supply of tranche tokens over a look-back period of 2.5 months.

The EMA is used to calculate the:
- yield target via a call to [MATH.yieldTarget()](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeMath.sol#L113-L121) which expects "_ema-based supply of zSTT & zJTT_" as its params.
- senior tranche yield proportion via a call to [MATH.seniorProportion()](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeMath.sol#L59-L70) which again expects ema-based `yT, eSTT & eJTT`.

Both the above mentioned calls happen inside `earningsTrancheuse()` [here](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeYDL.sol#L461-L462). The issue is that these global variables `emaSTT and emaJTT` are still outdated and correspond to the ones belonging to the `lastDistribution` timestamp instead of current updated ones. These values are updated only after the call to `earningsTrancheuse()` has concluded [here](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeYDL.sol#L238-L239).

## Impact
Imagine that in the last 30 days, JTT supply has reduced due to defaulted loans. As a result, the target yield $yT_{real}$ would come down too (i.e. $yT_{real}$ < $yT$ where $yT$ is the protocol calculated incorrect target yield) and the junior tranche ditributable yield will be smaller than before. However if there is excess yield, then the junior tranche would receive more than their fair share since last 30 days have not been considered by the protocol calculations. If the quantum of defaulted loans is high (and hence $yT_{real}$ has dipped significantly), the impact is not only limited to the junior tranche, but then also effects the senior tranche. <br>

On the flip side an increase in the adjusted supplies of STT and JTT in the last 30 days will have a impact on the smoothened emaSTT and emaJTT, making the values go higher. As a result, target yield $yT_{real}$ would go up too. Thus, it could happen that `yD` is less than $yT_{real}$ but the protocol is oblivious to that due to usage of outdated values. This would result in the protcol using [seniorProportionBase()](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeMath.sol#L74) to calculate the senior's yield instead of [seniorProportionShortfall()](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeMath.sol#L72). 
<br>

Finally, since at each of these function calls an outdated eSTT is passed as a param, the value returned would be more/less than what it should be, thus causing gain/loss of yield for both the tranches ([juniorProportion()](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeMath.sol#L74) calculation has the same problem).
<br>

**_Note:_** For the very first distribution i.e. when `distributionCounter = 1`, the values used are from 60 days ago instead of 30 days [as can be seen inside `unlock()`](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeYDL.sol#L328-L331), further increasing the margin of error. The 60 day gap has initially been provided by the protocol to allow for more time to bring in initial yield.

## Tool used
Manual Review

## Recommendation
Update the ema values **_before_** calling `earningsTrancheuse()`:
```diff
+       // Update ema-based supply values.
+       (uint256 aSTT, uint256 aJTT) = IZivoeGlobals_YDL(GBL).adjustedSupplies();
+       emaSTT = MATH.ema(emaSTT, aSTT, retrospectiveDistributions.min(distributionCounter));
+       emaJTT = MATH.ema(emaJTT, aJTT, retrospectiveDistributions.min(distributionCounter));

        // Calculate yield distribution (trancheuse = "slicer" in French).
        (
            uint256[] memory _protocol, uint256 _seniorTranche, uint256 _juniorTranche, uint256[] memory _residual
        ) = earningsTrancheuse(protocolEarnings, postFeeYield); 

        emit YieldDistributed(_protocol, _seniorTranche, _juniorTranche, _residual);
        
-       // Update ema-based supply values.
-       (uint256 aSTT, uint256 aJTT) = IZivoeGlobals_YDL(GBL).adjustedSupplies();
-       emaSTT = MATH.ema(emaSTT, aSTT, retrospectiveDistributions.min(distributionCounter));
-       emaJTT = MATH.ema(emaJTT, aJTT, retrospectiveDistributions.min(distributionCounter));
```

[Back to Top](#summaryTable)
 
---

### <a id="h-03"></a>[H-03]
## **Protocol charges incorrect interest for the first loan term**
<br>

## Summary
The protocol misses to charge interest for the x-days of lead-time introduced due to the ["Friday" Payment Standardization](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCC/OCC_Modular.sol#L475-L476).

## Vulnerability Detail
When a user accepts an offer, the principal amount is immediately disbursed to them and their `paymentDueBy` is set via:
```js
      loans[id].paymentDueBy = block.timestamp - block.timestamp % 7 days + 9 days + loans[id].paymentInterval;
```

Depending on the value of `block.timestamp % 7 days` which could range from 0 to any value less than 7 days, `paymentDueBy` value could be anywhere between `2 days` to `9 days` greater than `block.timestamp + paymentInterval`. However when the user makes a payment on the due date, the interest is calculated inside `amountOwed()` only for the duration of `paymentInterval`, ignoring the `2 to 9 days` of additional period. 
```js
      interest = loans[id].principalOwed * loans[id].paymentInterval * loans[id].APR / (86400 * 365 * BIPS);
```

This results in loss of funds for the protocol.

## Code Snippet
[https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCC/OCC_Modular.sol#L485](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCC/OCC_Modular.sol#L485)
```js
  File: src/lockers/OCC/OCC_Modular.sol

  461:              function acceptOffer(uint256 id) external nonReentrant {
  462:                  require(
  463:                      loans[id].state == LoanState.Offered, 
  464:                      "OCC_Modular::acceptOffer() loans[id].state != LoanState.Offered"
  465:                  );
  466:                  require(
  467:                      block.timestamp < loans[id].offerExpiry, 
  468:                      "OCC_Modular::acceptOffer() block.timestamp >= loans[id].offerExpiry"
  469:                  );
  470:                  require(
  471:                      _msgSender() == loans[id].borrower, 
  472:                      "OCC_Modular::acceptOffer() _msgSender() != loans[id].borrower"
  473:                  );
  474:          
  475:                  // "Friday" Payment Standardization, minimum 7-day lead-time
  476:                  // block.timestamp - block.timestamp % 7 days + 9 days + paymentInterval
  477:                  emit OfferAccepted(
  478:                      id, 
  479:                      loans[id].principalOwed, 
  480:                      loans[id].borrower, 
  481:                      block.timestamp - block.timestamp % 7 days + 9 days + loans[id].paymentInterval
  482:                  );
  483:          
  484:                  loans[id].state = LoanState.Active;
  485: @--->            loans[id].paymentDueBy = block.timestamp - block.timestamp % 7 days + 9 days + loans[id].paymentInterval;
  486:                  IERC20(stablecoin).safeTransfer(loans[id].borrower, loans[id].principalOwed);
  487:              }
```

and [https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCC/OCC_Modular.sol#L455](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCC/OCC_Modular.sol#L455)
```js
  File: src/lockers/OCC/OCC_Modular.sol

  440:              function amountOwed(uint256 id) public view returns (
  441:                  uint256 principal, uint256 interest, uint256 lateFee, uint256 total
  442:              ) {
  443:                  // 0 == Bullet.
  444:                  if (loans[id].paymentSchedule == 0) {
  445:                      if (loans[id].paymentsRemaining == 1) { principal = loans[id].principalOwed; }
  446:                  }
  447:                  // 1 == Amortization (only two options, use else here).
  448:                  else { principal = loans[id].principalOwed / loans[id].paymentsRemaining; }
  449:          
  450:                  // Add late fee if past loans[id].paymentDueBy.
  451:                  if (block.timestamp > loans[id].paymentDueBy && loans[id].state == LoanState.Active) {
  452:                      lateFee = loans[id].principalOwed * (block.timestamp - loans[id].paymentDueBy) *
  453:                          loans[id].APRLateFee / (86400 * 365 * BIPS);
  454:                  }
  455: @--->            interest = loans[id].principalOwed * loans[id].paymentInterval * loans[id].APR / (86400 * 365 * BIPS);
  456:                  total = principal + interest + lateFee;
  457:              }
```

## Impact
Less interest received by the protocol for every loan, effectively resulting in loss of funds.

## Tool used
Manual Review

## Recommendation
The following changes need to be made-
1. Add an extra item inside `struct Loan` named `actualFirstTermDuration`:
```diff

    struct Loan {
        address borrower;               /// @dev The address that receives capital when the loan is accepted.
        uint256 principalOwed;          /// @dev The amount of principal still owed on the loan.
        uint256 APR;                    /// @dev The annualized percentage rate charged on the outstanding principal.
        uint256 APRLateFee;             /// @dev The APR charged on the outstanding principal if payment is late.
        uint256 paymentDueBy;           /// @dev The timestamp (in seconds) for when the next payment is due.
        uint256 paymentsRemaining;      /// @dev The number of payments remaining until the loan is "Repaid".
        uint256 term;                   /// @dev The number of paymentIntervals that will occur (e.g. 12, 24).
        uint256 paymentInterval;        /// @dev The interval of time between payments (in seconds).
        uint256 offerExpiry;            /// @dev The block.timestamp at which the offer for this loan expires.
        uint256 gracePeriod;            /// @dev The number of seconds a borrower has to makePayment() before default.
        int8 paymentSchedule;           /// @dev The payment schedule of the loan (0 = "Bullet" or 1 = "Amortization").
+       uint256 actualFirstTermDuration; /// @dev The actual duration of the first term.
        LoanState state;                /// @dev The state of the loan.
    }
``` 

2. Initialize it at the time of offer acceptance:
```diff
    function acceptOffer(uint256 id) external nonReentrant {
        require(
            loans[id].state == LoanState.Offered, 
            "OCC_Modular::acceptOffer() loans[id].state != LoanState.Offered"
        );
        require(
            block.timestamp < loans[id].offerExpiry, 
            "OCC_Modular::acceptOffer() block.timestamp >= loans[id].offerExpiry"
        );
        require(
            _msgSender() == loans[id].borrower, 
            "OCC_Modular::acceptOffer() _msgSender() != loans[id].borrower"
        );

        // "Friday" Payment Standardization, minimum 7-day lead-time
        // block.timestamp - block.timestamp % 7 days + 9 days + paymentInterval
        emit OfferAccepted(
            id, 
            loans[id].principalOwed, 
            loans[id].borrower, 
            block.timestamp - block.timestamp % 7 days + 9 days + loans[id].paymentInterval
        );

        loans[id].state = LoanState.Active;
        loans[id].paymentDueBy = block.timestamp - block.timestamp % 7 days + 9 days + loans[id].paymentInterval;
+       loans[id].actualFirstTermDuration = loans[id].paymentDueBy - block.timestamp;
        IERC20(stablecoin).safeTransfer(loans[id].borrower, loans[id].principalOwed);
    }
```

3. Use it inside `amountOwed()`:
```diff
    function amountOwed(uint256 id) public view returns (
        uint256 principal, uint256 interest, uint256 lateFee, uint256 total
    ) {
        // 0 == Bullet.
        if (loans[id].paymentSchedule == 0) {
            if (loans[id].paymentsRemaining == 1) { principal = loans[id].principalOwed; }
        }
        // 1 == Amortization (only two options, use else here).
        else { principal = loans[id].principalOwed / loans[id].paymentsRemaining; }

        // Add late fee if past loans[id].paymentDueBy.
        if (block.timestamp > loans[id].paymentDueBy && loans[id].state == LoanState.Active) {
            lateFee = loans[id].principalOwed * (block.timestamp - loans[id].paymentDueBy) *
                loans[id].APRLateFee / (86400 * 365 * BIPS);
        }
+      if (loans[id].paymentsRemaining == loans[id].term) // is the first payment
+       interest = loans[id].principalOwed * loans[id].actualFirstTermDuration * loans[id].APR / (86400 * 365 * BIPS);
+      else
        interest = loans[id].principalOwed * loans[id].paymentInterval * loans[id].APR / (86400 * 365 * BIPS);
        total = principal + interest + lateFee;
    }
```

4. `createOffer()` would need to be modified too to accomodate this new item (similar change needs to be made for other functions too which create the `Loan` struct, for e.g. `applyCombine()`):
```diff
    function createOffer(
        address borrower,
        uint256 borrowAmount,
        uint256 APR,
        uint256 APRLateFee,
        uint256 term,
        uint256 paymentInterval,
        uint256 gracePeriod,
        int8 paymentSchedule
    ) isUnderwriter external {
        require(term > 0, "OCC_Modular::createOffer() term == 0");
        require(
            paymentInterval == 86400 * 7 || paymentInterval == 86400 * 14 || paymentInterval == 86400 * 28 || 
            paymentInterval == 86400 * 91 || paymentInterval == 86400 * 364, 
            "OCC_Modular::createOffer() invalid paymentInterval value, try: 86400 * (7 || 14 || 28 || 91 || 364)"
        );
        require(gracePeriod >= 7 days, "OCC_Modular::createOffer() gracePeriod < 7 days");
        require(paymentSchedule <= 1, "OCC_Modular::createOffer() paymentSchedule > 1");

        emit OfferCreated(
            borrower, loanCounter, borrowAmount, APR, APRLateFee, term,
            paymentInterval, block.timestamp + 3 days, gracePeriod, paymentSchedule
        );

        loans[loanCounter] = Loan(
            borrower, borrowAmount, APR, APRLateFee, 0, term, term, paymentInterval, block.timestamp + 3 days,
-           gracePeriod, paymentSchedule, LoanState.Offered
+           gracePeriod, paymentSchedule, 0, LoanState.Offered
        );

        loanCounter += 1;
    }
```

[Back to Top](#summaryTable)

---

### <a id="h-04"></a>[H-04]
## **totalSupply is incorrectly calculated during revokeVestingSchedule()**
<br>

## Summary
If a user has already withdrawn some amount during the vesting period, then upon calling `revokeVestingSchedule()` the `_totalSupply` is decreased to a greater extent than it should be. This can even lead to a revert due to underflow and the user's vesting schedule can't be revoked.

## Vulnerability Detail
- Imagine a vesting schedule is created at timestamp `t` for Alice and the `amountToVest` she receives is `100`. Hence `vestingScheduleOf[alice].totalVesting = 100`.
- Assuming Alice to be the only user, we can see that `_totalSupply = 100`.
- These are the vesting params:
  - cliff = 10 days
  - total vesting duration = 40 days
- At `t + 10 days` when cliff is reached, she calls [withdraw()](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeRewardsVesting.sol#L501) to withdraw 25 staking tokens which is her `amountWithdrawable()`.
  - Hence `_totalSupply = _totalSupply - 25 = 75` on [L508](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeRewardsVesting.sol#L508)
- ZVL or ITO calls [revokeVestingSchedule()](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeRewardsVesting.sol#L429) at `t + 30`.
  - `amountWithdrawable()` is called internally on [L439](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeRewardsVesting.sol#L439) to determine that `amount = 50` is what she can withdraw.
  - `vestingAmount` is set to equal `vestingScheduleOf[alice].totalVesting` or `100` on [L440](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeRewardsVesting.sol#L440).
  - On [L451](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeRewardsVesting.sol#L451) and L452, even though the `_totalSupply` which remains to be deducted is only `75`, the full `vestingAmount` of `100` is attempted to be deducted. This causes a revert due to underflow.
  - Had there been other users in the system, the revert wouldn't have happened. However the `_totalSupply` would have been decreased by `100` instead of `75` and the sum of the balances of all users would have been greater than the `_totalSupply`. The `_writeCheckpoint()` on L452 would have saved this reduced figure of `_totalSupply`.

## Impact
1. Breaks the invariant that the sum of balances of all users should be equal to `_totalSupply`.
2. As the following PoC shows, it can lead to situations where it's simply not possible to `revokeVestingSchedule()` as it reverts with a panic code (underflow error).
3. A reduced `_totalSupply` means that now the quorum fraction required will be calculated incorrectly. Each existing vote will have more proportional value (power) than before. See the [constructor](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeGovernorV2.sol#L62-L64) of `ZivoeGovernerV2.sol` which internally calls `GovernorVotesQuorumFraction(10)` where you can find [the comment](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/4e5b11919e91b18b6683b6f49a1b4fdede579969/contracts/governance/extensions/GovernorVotesQuorumFraction.sol#L24-L33):
```js
    /**
     * @dev Initialize quorum as a fraction of the token's total supply.
     *
     * The fraction is specified as `numerator / denominator`. By default the denominator is 100, so quorum is
     * specified as a percent: a numerator of 10 corresponds to quorum being 10% of total supply. The denominator can be
     * customized by overriding {quorumDenominator}.
     */
    constructor(uint256 quorumNumeratorValue) {
        _updateQuorumNumerator(quorumNumeratorValue);
    }
```

## Code Snippet
**Proof of Concept:** Add the following test inside `zivoe-core-testing/src/TESTS_Core/Test_ZivoeRewardsVesting.sol` and run via `forge test --rpc-url https://rpc.ankr.com/eth --mt test_t0x1c_bug_revokeVestingSchedule -vvvv` to see it revert with `panic: arithmetic underflow or overflow (0x11)`:
```js
    function test_t0x1c_bug_revokeVestingSchedule() public {
        uint256 t = block.timestamp;
        uint256 PRECISION_SAVER = 1 days;

        uint256 amount = 100 ether * PRECISION_SAVER; // multiplied with PRECISION_SAVER or `1 days` for ease of calculation, so that `vestingPerSecond` calculates as a non-rounded, exact whole number

        assert(zvl.try_createVestingSchedule(
            address(vestZVE), 
            address(moe), 
            10, 
            40,
            amount, 
            true
        ));

        // Pre-state.
        (
            uint256 start, 
            uint256 cliff, 
            uint256 end, 
            uint256 totalVesting, 
            uint256 totalWithdrawn, 
            uint256 vestingPerSecond,
        ) = vestZVE.viewSchedule(address(moe));

        assertEq(start, t);
        assertEq(cliff, t + 10 days);
        assertEq(end, t + 40 days);
        assertEq(totalVesting, amount);
        assertEq(totalWithdrawn, 0);
        assertEq(vestingPerSecond, amount / (40 days));
        assertEq(vestZVE.balanceOf(address(moe)), amount);
        assertEq(vestZVE.totalSupply(), amount);
        assertEq(ZVE.balanceOf(address(moe)), 0);

        hevm.warp(t + 10 days); // reached cliff (25% of vesting duration)

        uint256 amountWithdrawable = vestZVE.amountWithdrawable(address(moe));
        assertEq(amountWithdrawable, 25 ether * PRECISION_SAVER);  // 25% of `amount`
        assert(moe.try_withdraw(address(vestZVE)));
        assertEq(vestZVE.totalSupply(), 75 ether * PRECISION_SAVER); // 75% of `amount`
        assertEq(ZVE.balanceOf(address(moe)), 25 ether * PRECISION_SAVER);

        hevm.warp(t + 30 days); // reached 75% of vesting duration

        amountWithdrawable = vestZVE.amountWithdrawable(address(moe));
        assertEq(amountWithdrawable, 50 ether * PRECISION_SAVER);  // 50% of `amount` since 25% has already been withdrawn
        assert(zvl.try_revokeVestingSchedule(address(vestZVE), address(moe))); // @audit-issue : will revert here with `panic` due to underflow 
    }
```

<br>

Output:
```text

    ...
    ...
    ├─ [16906] Admin::try_revokeVestingSchedule(ZivoeRewardsVesting: [0x8227724C33C1748A42d1C1cD06e21AB8Deb6eB0A], Vester: [0x13aa49bAc059d709dd0a18D6bb63290076a702D7])
    │   ├─ [15275] ZivoeRewardsVesting::revokeVestingSchedule(Vester: [0x13aa49bAc059d709dd0a18D6bb63290076a702D7])
    │   │   ├─ [373] ZivoeGlobals::ZVL() [staticcall]
    │   │   │   └─ ← Admin: [0x2e234DAe75C793f67A35089C9d99245E1C58470b]
    │   │   └─ ← panic: arithmetic underflow or overflow (0x11)
    │   └─ ← false

```

## Tool used
Foundry

## Recommendation
The accounting can be corrected in the following manner:
```diff
    function revokeVestingSchedule(address account) external updateReward(account) onlyZVLOrITO nonReentrant {
        require(
            vestingScheduleSet[account], 
            "ZivoeRewardsVesting::revokeVestingSchedule() !vestingScheduleSet[account]"
        );
        require(
            vestingScheduleOf[account].revokable, 
            "ZivoeRewardsVesting::revokeVestingSchedule() !vestingScheduleOf[account].revokable"
        );
        
        uint256 amount = amountWithdrawable(account);
        uint256 vestingAmount = vestingScheduleOf[account].totalVesting;

        vestingTokenAllocated -= amount;
+       uint256 totalWithdrawnBeforeRevoke = vestingScheduleOf[account].totalWithdrawn;
        vestingScheduleOf[account].totalWithdrawn += amount;
        vestingScheduleOf[account].totalVesting = vestingScheduleOf[account].totalWithdrawn;
        vestingScheduleOf[account].cliff = block.timestamp - 1;
        vestingScheduleOf[account].end = block.timestamp;

        vestingTokenAllocated -= (vestingAmount - vestingScheduleOf[account].totalWithdrawn);

-       _totalSupply = _totalSupply.sub(vestingAmount);
-       _writeCheckpoint(_totalSupplyCheckpoints, _subtract, vestingAmount);
+       _totalSupply = _totalSupply.sub(vestingAmount - totalWithdrawnBeforeRevoke);
+       _writeCheckpoint(_totalSupplyCheckpoints, _subtract, vestingAmount - totalWithdrawnBeforeRevoke);
        _writeCheckpoint(_checkpoints[account], _subtract, amount);
        _balances[account] = 0;
        stakingToken.safeTransfer(account, amount);

        vestingScheduleOf[account].revokable = false;

        emit VestingScheduleRevoked(
            account, 
            vestingAmount - vestingScheduleOf[account].totalWithdrawn, 
            vestingScheduleOf[account].cliff, 
            vestingScheduleOf[account].end, 
            vestingScheduleOf[account].totalVesting, 
            false
        );
    }
```

[Back to Top](#summaryTable)

---

### <a id="h-05"></a>[H-05]
## **While combining loans, APR is incorrectly calculated**
<br>

## Summary
If two loans `Loan1` and `Loan2` are being combined, the currently implemented `APR` calculation of the combined loan works only when `Loan1.term & Loan1.paymentInterval` match with `Loan2.term & Loan2.paymentInterval` which also match with the underwriter's supplied `CombinedLoan.term & CombinedLoan.paymentInterval`. Any mismatch results in an incorrect `APR` of the combined loan since the formula simply does not take into account the `CombinedLoan.term & CombinedLoan.paymentInterval` set by the underwriter. 
<br>

[OCC_Modular::applyCombine()](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCC/OCC_Modular.sol#L773-L774)
```js
                ...

773:            notional += loans[loanID].principalOwed;
774:            APR += loans[loanID].principalOwed * loans[loanID].APR;  // @audit : misses accounting for `term` and `paymentInterval`
            
                ...
```

Importanly, not only this but also when an attempt is made to combine loans with different types of payment schedule (bullet/amortization) into a different resulting schedule, the formula breaks down. For example, trying to combine a bullet loan and an amortization loan into a resultant bullet loan causes an incorrect `APR` calculation currently.

## Vulnerability Detail
Refer the calculations inside the [Google sheet](https://docs.google.com/spreadsheets/d/1PpvFzoSOWia4vzd5ynlDf_LvUcP9OSThNJ7_K367R3Q/edit?usp=sharing):
- The first worksheet `ZivoeCombiLoan1` shows how the protocol will work smoothly when `term` and `paymentInterval` (cells `B3 and B4`; also cells `H3 and H4`) of both Loan1 and Loan2 match prefectly. A combined loan can be created by the underwriter with the same `term = 6` and `paymentInterval = 1 month` (blue cells). The `APR` in cell `O5` which is calculated using the protocol's current formula of `APR += loans[loanID].principalOwed * loans[loanID].APR;` and `APR = APR / notional;` works as intended. The green and yellow cells (`H27, H28` vs `Q24, R24`) match up as expected.

- The second worksheet `ZivoeCombiLoan2` shows how things go wrong with the green & yellow cells not matching anymore as soon as Loan1 and Loan2 have different combination of `term` & `paymentInterval` (orange cells) resulting in the underwriter choosing `term = 6` and `paymentInterval = 1 month` (blue cells). One can play around with the orange & blue cells to see other combinations. The red cell `O5` implements the same formula as the protocol to calculate the new `APR` but it results in Zivoe charging less than it should have.

- The third worksheet `ZivoeCombiLoan3` corrects this formula. Based on the values underwriter provides in the blue cells (`O3` and `O4`), cell `O5` calculates the correct `APR` resulting in no loss for the protocol. The values which need to be incorporated into the formula are the new `term` & `paymentInterval` as well as the interest the borrower would have paid had the loans not been combined (cell `H27`). **One can play around by changing the numbers in the orange & blue cells to any random value to ascertain that Zivoe is never taking a loss.**

- The fourth worksheet `ZivoeBullet1` shows how the current protocol calculations fail when Loan1 is `bullet`, Loan2 is `amortization` and the combined loan is of type `bullet`. This is in spite of the fact that the `term` and `paymentInterval` of all the loans is exactly the same.

- The fifth worksheet `ZivoeBullet2` corrects the formula in cell `O5` such that the green & blue values match.

## Impact
Zivoe can lose funds by receiving less than required interest whenever loans are combined. Or may charge the borrower more than expected.

## Code Snippet
https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCC/OCC_Modular.sol#L773-L781

## Proof of Concept
Add the following test inside `zivoe-core-testing/src/TESTS_Lockers/Test_OCC_Modular.sol` and run via `forge test --rpc-url https://rpc.ankr.com/eth --mt test_t0x1c_OCC_Modular_applyCombine -vvvv` to see the output which shows how Zivoe received less than expected interest (recreates the example from worksheet `ZivoeCombiLoan2` with different loan params):
```js
    function test_t0x1c_OCC_Modular_applyCombine() public {
        simulateITO_and_createOffers(1000 ether, false);
        uint256 loan1 = OCC_Modular_DAI.loanCounter();
        assert(roy.try_createOffer(
            address(OCC_Modular_DAI),
            address(tim),
            120 ether, // borrowAmount,
            1200, // APR,
            600, // APRLateFee
            3, // term,
            14 days, // paymentInterval
            8 days,
            int8(1)
        ));

        uint256 loan2 = OCC_Modular_DAI.loanCounter();
        assert(roy.try_createOffer(
            address(OCC_Modular_DAI),
            address(tim),
            240 ether, // borrowAmount,
            2400, // APR,
            600, // APRLateFee
            6, // term,
            7 days, // paymentInterval
            8 days,
            int8(1)
        ));

        tim_acceptOffer(loan1, DAI);
        tim_acceptOffer(loan2, DAI);

        mint("DAI", address(tim), MAX_UINT / 10**18);
        assert(tim.try_approveToken(address(DAI), address(OCC_Modular_DAI), type(uint256).max));
        uint256 snapshot = vm.snapshot();

        // pay off Loan1
        uint256 interest;
        (,, uint256[10] memory loan1_info) = OCC_Modular_DAI.loanInfo(loan1);
        vm.warp(loan1_info[3]); // skip to `paymentDueBy` 
        (, uint256 interest1,,) = OCC_Modular_DAI.amountOwed(loan1);
        interest += interest1;
        assert(tim.try_makePayment(address(OCC_Modular_DAI), loan1));
        // repeat 2 more times
        for(uint256 rep; rep < 2; rep++) {
            (,, loan1_info) = OCC_Modular_DAI.loanInfo(loan1);
            vm.warp(loan1_info[3]); 
            (, interest1,,) = OCC_Modular_DAI.amountOwed(loan1);
            interest += interest1;
            assert(tim.try_makePayment(address(OCC_Modular_DAI), loan1));
        }
        (,, loan1_info) = OCC_Modular_DAI.loanInfo(loan1);
        assertEq(loan1_info[0], 0); // no principal owed
        emit log_named_decimal_uint("Interest paid for Loan1", interest, 18);

        // pay off Loan2
        uint256 interest_loan2;
        (,, uint256[10] memory loan2_info) = OCC_Modular_DAI.loanInfo(loan2);
        vm.warp(loan2_info[3]); // skip to `paymentDueBy` 
        (, uint256 interest2,,) = OCC_Modular_DAI.amountOwed(loan2);
        interest_loan2 += interest2;
        assert(tim.try_makePayment(address(OCC_Modular_DAI), loan2));
        // repeat 5 more times
        for(uint256 rep; rep < 5; rep++) {
            (,, loan2_info) = OCC_Modular_DAI.loanInfo(loan2);
            vm.warp(loan2_info[3]); 
            (, interest2,,) = OCC_Modular_DAI.amountOwed(loan2);
            interest_loan2 += interest2;
            assert(tim.try_makePayment(address(OCC_Modular_DAI), loan2));
        }
        (,, loan2_info) = OCC_Modular_DAI.loanInfo(loan2);
        assertEq(loan2_info[0], 0); // no principal owed
        emit log_named_decimal_uint("Interest paid for Loan2", interest_loan2, 18);
        emit log_named_decimal_uint("Total Interest paid for Loan1 + Loan2", interest + interest_loan2, 18);


        // Combine the loans now
        vm.revertTo(snapshot);
        uint[] memory loanIDs = new uint[](2);
        loanIDs[0] = loan1;
        loanIDs[1] = loan2;

        hevm.startPrank(address(roy));
        OCC_Modular_DAI.approveCombine(loanIDs, 600, 
                                                6 /* term */, 
                                                7 days /* paymentInterval */, 
                                                8 days, int8(1));
        hevm.stopPrank();

        hevm.startPrank(address(tim));
        uint256 combinedLoanID = OCC_Modular_DAI.loanCounter();
        OCC_Modular_DAI.applyCombine(0);  
        hevm.stopPrank();

        // pay off the combined loan
        uint256 interest_combi;
        (,, uint256[10] memory loanCombi_info) = OCC_Modular_DAI.loanInfo(combinedLoanID);
        vm.warp(loanCombi_info[3]); // skip to `paymentDueBy` 
        (, uint256 interestC,,) = OCC_Modular_DAI.amountOwed(combinedLoanID);
        interest_combi += interestC;
        assert(tim.try_makePayment(address(OCC_Modular_DAI), combinedLoanID));
        // repeat 5 more times
        for(uint256 rep; rep < 5; rep++) {
            (,, loanCombi_info) = OCC_Modular_DAI.loanInfo(combinedLoanID);
            vm.warp(loanCombi_info[3]); 
            (, interestC,,) = OCC_Modular_DAI.amountOwed(combinedLoanID);
            interest_combi += interestC;
            assert(tim.try_makePayment(address(OCC_Modular_DAI), combinedLoanID));
        }
        (,, loanCombi_info) = OCC_Modular_DAI.loanInfo(combinedLoanID);
        assertEq(loanCombi_info[0], 0); // no principal owed
        emit log_named_decimal_uint("Interest paid for Combined Loan =", interest_combi, 18);
        assertGt(interest + interest_loan2, interest_combi); // @audit : less interest was charged in the combined loan
    }
```

<br>

Output:
```text
[PASS] test_t0x1c_OCC_Modular_applyCombine() (gas: 4781520)
Logs:
  Interest paid for Loan1: 1.104657534246575341
  Interest paid for Loan2: 3.866301369863013696
  Total Interest paid for Loan1 + Loan2: 4.970958904109589037
  Interest paid for Combined Loan =: 4.832876712328767123    <--------- less interest charged when loans are combined
```

## Tool used
Foundry

## Recommendation
Add after [L781](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCC/OCC_Modular.sol#L781) the correct formula in line with the [Google sheet's](https://docs.google.com/spreadsheets/d/1PpvFzoSOWia4vzd5ynlDf_LvUcP9OSThNJ7_K367R3Q/edit?usp=sharing) cell `O5` inside `ZivoeCombiLoan3` worksheet:
```diff
                ...

  773:            notional += loans[loanID].principalOwed;
- 774:            APR += loans[loanID].principalOwed * loans[loanID].APR; 
  775:            loans[loanID].principalOwed = 0;
  776:            loans[loanID].paymentDueBy = 0;
  777:            loans[loanID].paymentsRemaining = 0;
  778:            loans[loanID].state = LoanState.Combined;
  779:        }
  780:
- 781:        APR = APR / notional; 
+ 781:        if (combinations[id].paymentSchedule > 0) // @audit : for amortization loans
+ 782:          APR = (BIPS * 365 days * 2 * prevInterest) / notional / (combinations[id].term + 1) / combinations[id].paymentInterval;  //  @audit : `prevInterest` needs to be calculated elsewhere and passed here. This is the interest the borrower would've paid in total if the loans were not combined.
+ 783:        else // @audit : for bullet loans
+ 784:          APR = (BIPS * 365 days * prevInterest) / notional / combinations[id].term / combinations[id].paymentInterval;
            
                ...
```

**_Note_:** The above recommedation works for all types of combinations. For example, Loan1 could be amortization, Loan2 could be bullet and the combined loan could be one or the other. They could have different terms and payment intervals. The recommended solution handles all the cases.

## Formula Derivation
The following gist can be referred to in order to understand how the new formula for `APR` for amortization has been arrived at:
- https://gist.github.com/t0x1cC0de/724080d9506943c0808ab45f67353bca

The variables used are:
- $i$ = prevInterest or the interest the borrower would've paid in total if the loans were not combined.
- $P$ = `principalOwed` or the `notional` value of the combined loan.
- $t$ = `term` of the combined loan.
- $D$ = principal payed back in each term. This equals $P/t$.
- $n$ = `paymentInterval` in seconds.
- $r$ = new `APR` expressed as value/second i.e. 12% per second is represented as `0.12`. To obtain the actual `APR` in BIPS, we should multiply $r$ with `(BIPS * 365 days)`.

The other formula for bullet loan type is quite simple where the prevInterest $i$ needs to be simply divided into an equal amount for each term and hence we calculate the $r$ which makes this possible.

[Back to Top](#summaryTable)

---

### <a id="h-06"></a>[H-06]
## **depositReward() function reduces rewardRate incorrectly causing delayed reward distribution and can be used by a griefer**
<br>

## Summary
The `depositReward()` function in both `ZivoeRewards.sol` and `ZivoeRewardsVesting.sol` modifies the `rewardRate` incorrectly by reducing it more than it ought to. While this in itself is problematic in the normal flow of events, a griefer can further exploit this by depositing `0 wei` and continually delaying reward payout through a reduction in the `rewardRate`.

## Vulnerability Detail
Consider the following scenario:
- Sam has staked some tokens and then some rewards are added into the contract at timestamp `t`.

- Assume the `_rewardsToken` to be DAI which has a `rewardsDuration` of 100. Hence the current `periodFinish` is `t + 100`.

- Assume the amount of DAI added as reward to be `500`.

- At `t + 50` Sam can claim 50% of DAI rewards `= 250`, and ideally the remaining `250` should be claimable at `t + 100`.

- Suppose now that a griefer calls `depositReward()` at `t + 50` with `reward = 0 wei`. Note that 0 amount is allowed and also anyone can call `depositReward()`.
  - Since `block.timestamp` is still less than `t + 100`, the [else clause](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeRewardsVesting.sol#L361) is hit where the `rewardData[_rewardsToken].rewardRate` is is calculated by again dividing `reward + leftover` or `0 + 250` with `rewardsDuration` which is 100. We just halved the `rewardRate`.
  - Furthermore, outside the else clause on [L365](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeRewardsVesting.sol#L365) the `periodFinish` is pushed forward by `rewardsDuration` so that the rewardRate is in sync with the payout duration.

- As a result, if Sam now tries to claim his reward at `t + 100`, he will receive `125` and the remaining `125` fully claimable only at `t + 150`.

- While this was an extreme example, it's trivial to see that even in the normal flow of events when X amount of rewards are added at any `t` less than `t + 100`, the current approach of reducing & "normalizing" the `rewardRate` is incorrect.

## Impact
- Any delayed reward distribution is akin to loss of funds since the receiver could have invested it and earned additional interest had it not been delayed. 
- Griefer can keep on performing the attack in order to reduce the `rewardRate` continuously.

## Code Snippet
- https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeRewards.sol#L237
- https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeRewardsVesting.sol#L361

## Proof of Concept
Add the following test inside `zivoe-core-testing/src/TESTS_Core/Test_ZivoeRewards.sol` and run via `forge test --rpc-url https://rpc.ankr.com/eth --mt test_t0x1c_depositReward -vv` to see the output which shows the halving of values:
```js
    function test_t0x1c_depositReward() public {

        uint256 t = block.timestamp;

        uint256 deposit = 1000 ether;

        // sam stakes some tokens
        assert(sam.try_approveToken(address(ZVE), address(stZVE), IERC20(address(ZVE)).balanceOf(address(sam))));
        sam.try_stake(address(stZVE), IERC20(address(ZVE)).balanceOf(address(sam)));

        uint256 rewardEarnedInitial = stZVE.earned(address(sam), DAI);
        assertEq(rewardEarnedInitial, 0);

        // some reward is deposited at `t`
        depositReward_DAI(address(stZVE), deposit);
        (
            uint256 rewardsDuration,
            uint256 periodFinish,
            uint256 rewardRate,
            uint256 lastUpdateTime,
            uint256 rewardPerTokenStored
        ) = stZVE.rewardData(DAI);
        emit log_named_decimal_uint("rewardRate1", rewardRate, 14);
        uint256 oldRate = rewardRate;
        uint256 t_end = periodFinish;


        hevm.warp(t + (t_end - t)/2); // midway to `periodFinish`
        uint256 rewardEarnedHalfway = stZVE.earned(address(sam), DAI);
        
        // store a snapshot we can return to later
        uint256 snapshot = vm.snapshot();

        hevm.warp(t_end); // to `periodFinish`
        uint256 rewardEarned = stZVE.earned(address(sam), DAI);
        uint256 rewardEarnedDelta1 = rewardEarned - rewardEarnedHalfway;
        emit log_named_decimal_uint("rewardEarnedDelta1", rewardEarnedDelta1, 18);

        // Let's examine the contrasting case when griefer deposits 0 wei at halftime
        vm.revertTo(snapshot);
        depositReward_DAI(address(stZVE), 0); // @audit-info : griefer deposits 0 wei
        (
            rewardsDuration,
            periodFinish,
            rewardRate,
            lastUpdateTime,
            rewardPerTokenStored
        ) = stZVE.rewardData(DAI);
        emit log_named_decimal_uint("rewardRate2", rewardRate, 14);
        assertLe(rewardRate, oldRate / 2); // @audit : reward rate got reduced by half

        hevm.warp(t_end); // to `periodFinish`
        rewardEarned = stZVE.earned(address(sam), DAI);
        uint256 rewardEarnedDelta2 = rewardEarned - rewardEarnedHalfway;
        emit log_named_decimal_uint("rewardEarnedDelta2", rewardEarnedDelta2, 18);
        assertLt(rewardEarnedDelta2, rewardEarnedDelta1); // @audit : reward is reduced by half for this period (got delayed)
    }
```

<br>

Output:
```text
[PASS] test_t0x1c_depositReward() (gas: 430612)
Logs:
  rewardRate1: 3.85802469135802
  rewardEarnedDelta1: 499.999999999999390000
  rewardRate2: 1.92901234567901       <--------- got halved as compared to `rewardRate1`
  rewardEarnedDelta2: 249.999999999999695000  <--------- got halved as compared to `rewardEarnedDelta1`
```

## Tool used
Foundry

## Recommendation
In the above provided example when additional reward is deposited at `t + 50`, the calculation needs to make the following adjustments:
- For the duration of `t + 50` to `t + 100`, the total reward distributable is `125 + R` where `R` is the reward deposited at `t + 50`. Hence `rewardRate` would be `(125 + R) / 50`.
- For the duration of `t + 100` to `t + 150`, the remaining reward is `R / 2` and hence `rewardRate` ought to be `(R / 2) / 50`.

Effectively, the protocol needs to maintain the correct `rewardRate` for each specific duration.

[Back to Top](#summaryTable)

---

### <a id="h-07"></a>[H-07]
## **OCL_ZVE::forwardYield() is susceptible to price manipulation attack due to the logic inside fetchBasis()**
<br>

## Summary
When `forwardYield()` is called, it calculates the surplus by fetching the `amount` & `lp` through a call to [fetchBasis()](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCL/OCL_ZVE.sol#L336) on [L301](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCL/OCL_ZVE.sol#L301). However `fetchBasis()` relies on [live pool balances](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCL/OCL_ZVE.sol#L338-L339) which can be manipulated by an attacker, resulting in an inflated or deflated value of `amount`.

## Vulnerability Detail
`forwardYield()` calls `fetchBasis()` to calculate `amount` before distributing the yield via `_forwardYield()` on [L301](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCL/OCL_ZVE.sol#L301):
```js
@--->   (uint256 amount, uint256 lp) = fetchBasis();
        if (amount > basis) { _forwardYield(amount, lp); }
```

`fetchBasis()` returns this `amount` based on the [spot pool balances](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCL/OCL_ZVE.sol#L338-L339):
```js
    function fetchBasis() public view returns (uint256 amount, uint256 lp) {
        address pool = IFactory_OCL_ZVE(factory).getPair(pairAsset, IZivoeGlobals_OCL_ZVE(GBL).ZVE());
@---->  uint256 pairAssetBalance = IERC20(pairAsset).balanceOf(pool);  // @audit : can be manipulated
        uint256 poolTotalSupply = IERC20(pool).totalSupply();
        lp = IERC20(pool).balanceOf(address(this));
@---->  amount = lp * pairAssetBalance / poolTotalSupply;
    }
```

An attacker can deflate/inflate `pairAssetBalance` and hence the `amount` by withdrawing/depositing `pairAsset` from/into the pool in a sandwich attack. This would result in [a low/high amount of yield to be distributed](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCL/OCL_ZVE.sol#L313) since `lpBurnable` is based on it.

## Impact
Incorrect yield distributed; loss of yield.

## Code Snippet
- https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCL/OCL_ZVE.sol#L338-L339

## Tool used
Manual review

## Recommendation
Consider using a time weighted average balance instead of spot balance to make it less prone to manipulation.

[Back to Top](#summaryTable)

<br><br>

## **MEDIUM-SEVERITY BUGS**
---

### <a id="m-01"></a>[M-01]
## **TLC's update of critical params inside ZivoeYDL get applied retrospectively, causing loss of pending yield**
<br>

## Summary
The yield accrues for `daysBetweenDistributions` inside `ZivoYDL.sol` before it can be claimed by calling [distributeYield()](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeYDL.sol#L216). This pending yield is distributed as per the [contract's parameters](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeYDL.sol#L89-L107) to various participants in pre-decided proportion. The exact values are calulated via an [internal call](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeYDL.sol#L232) to the function [earningsTrancheuse()](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeYDL.sol#L447).<br>

The TLC role can update these contract parameters via calls to:
- [updateDistributedAsset()](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeYDL.sol#L356)
- [updateProtocolEarningsRateBIPS()](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeYDL.sol#L375)
- [updateTargetAPYBIPS()](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeYDL.sol#L420)
- [updateTargetRatioBIPS()](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeYDL.sol#L428)
- [updateRecipients()](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeYDL.sol#L392)

All these changes are applied retrospectively to the pending yield yet to be claimed and may even result in complete loss of this yield to the senior & junior tranches along with the other recipients.

## Similar Past Findings
1. Sherlock finding "**[M-5: Update to `managerFeeBPS` applied to pending tokens yet to be claimed](https://github.com/sherlock-audit/2023-06-arrakis-judging/issues/198)**" in the Arrakis contest. _(Alternate Solodit url: [https://solodit.xyz/issues/m-5-update-to-managerfeebps-applied-to-pending-tokens-yet-to-be-claimed-sherlock-none-arrakis-git](https://solodit.xyz/issues/m-5-update-to-managerfeebps-applied-to-pending-tokens-yet-to-be-claimed-sherlock-none-arrakis-git))_

2. Cyfrin finding "**[Update to StratFeeManagerInitializable::beefyFeeConfig retrospectively applies new fees to pending LP rewards yet to be claimed](https://github.com/solodit/solodit_content/blob/main/reports/Cyfrin/2024-04-06-cyfrin-beefy-finance.md#update-to-stratfeemanagerinitializablebeefyfeeconfig-retrospectively-applies-new-fees-to-pending-lp-rewards-yet-to-be-claimed)**" for beefy finance. _(Alternate Solodit url: [https://solodit.xyz/issues/update-to-stratfeemanagerinitializablebeefyfeeconfig-retrospectively-applies-new-fees-to-pending-lp-rewards-yet-to-be-claimed-cyfrin-none-cyfrin-beefy-finance-markdown](https://solodit.xyz/issues/update-to-stratfeemanagerinitializablebeefyfeeconfig-retrospectively-applies-new-fees-to-pending-lp-rewards-yet-to-be-claimed-cyfrin-none-cyfrin-beefy-finance-markdown))_

## Vulnerability Detail
An example scenario:
- `daysBetweenDistributions` is 20 days. The yield has so far accrued for 19.99 days.
- The current `distributedAsset` is DAI. This is the token used to dole out the rewards.
- TLC decides to call (maliciously or otherwise) [updateDistributedAsset()](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeYDL.sol#L356) and changes `distributedAsset` to USDC.
- Now when `distributeYield()` is called at the end of 20 days, the `earnings` variable on [L221](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeYDL.sol#L221) evaluates to `IERC20(distributedAsset).balanceOf(address(this)) = 0` since there is no USDC balance, only the DAI balance. 
- All pending yield is lost.

- Similarly, `targetAPYBIPS` or `targetRatioBIPS` could have been modified to change the proportion of the yield the senior & junior tranche would have received as per the calculations of [earningsTrancheuse()](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeYDL.sol#L461-L462).

- Even the particpants' list can be changed altogether via [updateRecipients()]([earningsTrancheuse()](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeYDL.sol#L392)), having a retrospective effect.

## Other Code Areas With Same Issue
Similar issue can be found inside:
- `OCE_ZVE.sol` : [updateCompoundingRateBIPS()](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCE/OCE_ZVE.sol#L163) and [updateExponentialDecayPerSecond](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCE/OCE_ZVE.sol#L183) have a retrospective effect on [forwardEmissions()](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCE/OCE_ZVE.sol#L139).
- `OCL_ZVE.sol` : [updateCompoundingRateBIPS()](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCL/OCL_ZVE.sol#L347) has a retrospective effect on [forwardYield()](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCL/OCL_ZVE.sol#L311).

## Impact
Loss of yield for the participants of ZivoeYDL.

## Code Snippet
- [https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeYDL.sol#L221](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeYDL.sol#L216)
- [https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeYDL.sol#L461-L462](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/ZivoeYDL.sol#L447)

## Tool used
Manual Review

## Recommendation
The current values of contract parameters which are being updated should be stored up until the next call to `distributeYield()` is not made. The new parameter changes should be considered as "pending" till then. Once the call to `distributeYield()` has concluded based on the old values, these "pending" values should then come into effect, overwriting the old ones.

[Back to Top](#summaryTable)
 
---

### <a id="m-02"></a>[M-02]
## **Neither approveCombine() nor applyCombine() checks if late fee is applicable on any of the constituent loans, which could have even accrued after underwriter's call to approveCombine()**
<br>

## Summary
When an underwriter approves multiple loans to be combined and calls [approveCombine()](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCC/OCC_Modular.sol#L877) there is no check to verify that the constituents loan are liable to pay a `lateFee` or not. <br>
Even if we consider that this has been checked by the underwriter off-chain, the borrower has [72 hours to accept](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCC/OCC_Modular.sol#L897C86-L897C112) this offer before calling [applyCombine()](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCC/OCC_Modular.sol#L742). It's quite possible that the loan slips past the `paymentDueBy` date within these 72 hours.

## Vulnerability Detail
Consider the scenario:
- Loan1 and Loan2 are to be combined. Underwriter verifies off-chain that the loans are active with no overdue payments. He calls `approveCombine()` at timestamp `t`. This offer would be valid till `t + 72 hours` due to the code on [L897](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCC/OCC_Modular.sol#L897C86-L897C112).
- Loan1 has `paymentDueBy` at `t + 20 hours`.
- For some reason borrower waits until `t + 70 hours` before calling `applyCombine()`. 
- Borrower escapes paying the `lateFee` applicable for the duration of `50 hours`. Once combined with Loan2, he has a "healthy" loan.

## Impact
Loss of interest for the protocol that had accrued as a result of late payment.

## Code Snippet
- https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCC/OCC_Modular.sol#L897C86-L897C112
- https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCC/OCC_Modular.sol#L877

## Tool used
Manual Review

## Recommendation
A simple way would be to check inside `approveCombine()` that none of the constituent loans being combined have a `paymentDueBy` within the next 72 hours. Alternatively, it can also be checked inside `applyCombine()` that none of the constituent loans are past their due date.

[Back to Top](#summaryTable)

---

### <a id="m-03"></a>[M-03]
## **No slippage protection inside OCL_ZVE::_forwardYield() while removing liquidity**
<br>

## Summary
`OCL_ZVE::forwardYield()` internally calls [_forwardYield()](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCL/OCL_ZVE.sol#L316-L317) where `removeLiquidity()` is called on L316. This call has no slippage protection applied.

## Vulnerability Detail
The curent code on [L316-L318](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCL/OCL_ZVE.sol#L316-L318) looks like this:
```js
        (uint256 claimedPairAsset, uint256 claimedZVE) = IRouter_OCL_ZVE(router).removeLiquidity(
            pairAsset, ZVE, lpBurnable, 0, 0, address(this), block.timestamp + 14 days
        );
```

As can be seen, `0, 0` is supplied instead of an apt slippage protection. Thus the `claimedPairAsset & claimedZVE` could be way less than expected as this call to `forwardYield()` can be sandwiched by an attacker resulting in price manipulation. Note that `forwardYield()` is callable by anyone if it has not been [called by the keeper first](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCL/OCL_ZVE.sol#L288-L299).<br>
Even in the absence of a malicious actor, high volatility period can result in an unfavourable deal for the protocol.

## Impact
Possible loss of yield due to missing slippage protection.

## Code Snippet
- https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCL/OCL_ZVE.sol#L316-L317

## Tool used
Manual Review

## Recommendation
Calculate `minPairAsset & minZVE` inside the protocol at runtime such that it's within acceptable limits. Or allow the caller of `forwardYield()` to pass in minimum amounts through a param like `bytes calldata data`. The second option is less preferable since `forwardYield()` can be called by anyone and they could supply 0 as slippage parameters.

[Back to Top](#summaryTable)
 
---

### <a id="m-04"></a>[M-04]
## **Protocol supports rebasing token OUSD but calculates basis & yield incorrectly in `OCY_OUSD.sol`**
<br>

## Summary
The balance of rebasing token OUSD can change dynamically but the protocol uses a stored value of `basis` to calculate distributable yield, which can be outdated and can give incorrect results.

## Vulnerability Detail
Consider the scenario:
- Current `basis` and balance is 100.
- Additional yield of 10 accrues. Balance = 110.
- A positive rebase of OUSD happens as a result of which balances of all holders are reduced by 10.
- `forwardYield()` is called. It [calculates `amountOUSD`](https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCY/OCY_OUSD.sol#L146-L147) to be equal to the current balance i.e. 100.
- Since `amountOUSD > basis` is `false`, no yield is distributed.

In the inverse case of negative rebasing, the protocol ends up forwarding higher than expected yield.

## Impact
Possible loss of yield.

## Code Snippet
https://github.com/sherlock-audit/2024-03-zivoe-t0x1cC0de/blob/main/zivoe-core-foundry/src/lockers/OCY/OCY_OUSD.sol#L146-L147

## Tool used
Manual Review

## Recommendation
OUSD rebasing events will need to be tracked and `basis` updated accordingly.

[Back to Top](#summaryTable)
 
