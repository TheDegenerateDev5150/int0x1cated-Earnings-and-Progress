# Leaderboard
[Renzo Results](https://code4rena.com/audits/2024-04-renzo#top)<br>

`Rank 36 / 122`

# Audited Code Repo
### [Code4rena: Renzo](https://github.com/code-423n4/2024-04-renzo)

<br>

# <a id="summaryTable"></a>Bugs Filed & Their Status

| # | Bug ID | Name | URL | Adjudged Status | Count of dups (excluding current submission) |
|--------|--------|------|:------:|-----------------:|:-----------------------------------:|
| 1 | [H-01](#h-01)  | Incomplete implementation of pause functionality inside `WithdrawQueue.sol` results in it having no effect on the contract's methods | [565](https://github.com/code-423n4/2024-04-renzo-findings/issues/565) | Accepted as Medium | 26 |
| 2 | [H-02](#h-02)  | Incorrect totalTVL calculation due to wrong index used inside `collateralTokens[]` | [223](https://github.com/code-423n4/2024-04-renzo-findings/issues/223) | Accepted as High | 70 |
| 3 | [H-03](#h-03)  | Calculating `amountToRedeem` inside `WithdrawQueue::withdraw()` instead of within `claim()` allows front-running a slashing event while also causing redemption of incorrect amount | [544](https://github.com/code-423n4/2024-04-renzo-findings/issues/544) | Accepted as High | 62 |
| 4 | [H-04](#h-04)  | totalTVL is incorrectly calculated due to the addition of DepositQueue's entire balance | [407](https://github.com/code-423n4/2024-04-renzo-findings/issues/407) | Rejected | - |
| 5 | [H-05](#h-05)  | RestakeManager::deposit() is susceptible to inflation attack | [155v](https://github.com/code-423n4/2024-04-renzo-validation/issues/155) | Rejected | - |
| 6 | [H-06](#h-06)  | Funds get stuck and LRT rate drops when a staker is undelegated | [452](https://github.com/code-423n4/2024-04-renzo-findings/issues/452) | Rejected | - |
| 7 | [H-07](#h-07)  | verifyWithdrawalCredentials() will revert and some validators can't be verified if any of the operators have added a validator directly, from outside the protocol | [359](https://github.com/code-423n4/2024-04-renzo-findings/issues/359) | Rejected | - |
| 8 | [M-01](#m-01)  | Gas refund amount as well as staking start time are dependent upon admin's choice of call between `stakeEthFromQueue()` and `stakeEthFromQueueMulti()` | [504](https://github.com/code-423n4/2024-04-renzo-findings/issues/504) | Accepted as QA | - |
| 9 | [M-02](#m-02)  | Failed ERC20 transfer inside `claim()` results in permanent loss of funds | [54v](https://github.com/code-423n4/2024-04-renzo-validation/issues/54) | Rejected | - |
|10 | [M-03](#m-03)  | Lack of slippage and deadline during withdraw and deposit | [484](https://github.com/code-423n4/2024-04-renzo-findings/issues/484) | Accepted as Medium; Selected Report | 29 |
|11 | [M-04](#m-04)  | Change of strategy via `setTokenStrategy()` will cause incorrect TVL calculation | [480](https://github.com/code-423n4/2024-04-renzo-findings/issues/480) | Rejected | - |
|12 | [M-05](#m-05)  | `removeOperatorDelegator()` does not check if the OD being removed has any existing staked balance | [410](https://github.com/code-423n4/2024-04-renzo-findings/issues/410) | Rejected | - |
|13 | [M-06](#m-06)  | Even after `removeCollateralToken()`, user can withdraw funds using the removed token | [465](https://github.com/code-423n4/2024-04-renzo-findings/issues/465) | Accepted as Medium | 20 |
|14 | [M-07](#m-07)  | Inaccurate WETH and `xezETH` accounting inside `xRenzoBridge::xReceive()` due to absence of try/catch block | [371](https://github.com/code-423n4/2024-04-renzo-findings/issues/371) | Accepted as Medium | 8 |

<br>

## **HIGH-SEVERITY BUGS**
---

### <a id="h-01"></a>[H-01]
## **Incomplete implementation of pause functionality inside `WithdrawQueue.sol` results in it having no effect on the contract's methods**
#### https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L7
<br>

## Description
The `WithdrawQueue.sol` contract [inherits](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L7) OpenZeppelin's `PausableUpgradeable.sol`. It correctly calls [__Pausable_init()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L81) inside `initialize()` and implements [pause()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L139) and [unpause()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L147) too **but misses to apply** the [relevant modifiers](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/utils/PausableUpgradeable.sol#L72-L87) `whenNotPaused` and `whenPaused` on any of the functions as prescribed clearly by OZ [here](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/utils/PausableUpgradeable.sol#L15-L16): 
```text
            /**
            * @dev Contract module which allows children to implement an emergency stop
            * mechanism that can be triggered by an authorized account.
            *
            * This module is used through inheritance. It will make available the
            * modifiers `whenNotPaused` and `whenPaused`, which can be applied to
@------>    * the functions of your contract. Note that they will not be pausable by
@------>    * simply including this module, only once the modifiers are put in place.
            */
 ```
Thus pausing or unpausing the contract has no effect at all on any user flows, for example [withdraw()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L206) or [claim()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L279). 

## Impact
Pausing a contract is typically a special/emergency action taken by governance to remedy issues. This makes it important that certain user actions are suspended when the contract is paused. In the current scenario, it makes sense to not allow users to claim funds when the contract is paused (this could be applicable to `withdraw()` too). Hence the expected flow of events should be -
- The `onlyWithdrawQueueAdmin` role calls [pause()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L139) to pause the contract.
- All users should be unable to call [claim()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L279) now.

However currently, all users are able to call `claim()` (or `withdraw()`). This could result in the users exiting with funds linked to suspicious activities which the protocol governance intended to block. 

## Tools Used
Manual review

## Recommended Mitigation Steps
Add the modifiers `whenNotPaused` and `whenPaused` wherever relevant. For example, `claim()` function could have `whenNotPaused` applied. The `withdraw()` function is another candidate for the same, but depends on the protocol's intention as to which actions they want to limit when the contract is in a paused state.

[Back to Top](#summaryTable)

---

### <a id="h-02"></a>[H-02]
## **Incorrect totalTVL calculation due to wrong index used inside `collateralTokens[]`**
#### https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L318
<br>

## Description
`calculateTVLs()` returns the incorrect `totalTVL` since it always calculates `totalWithdrawalQueueValue` incorrectly on [L318](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L318) which is used to calculate `totalTVL` by adding to it on [L355](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L355).<br>
This incorrect calculation of `totalWithdrawalQueueValue` on [L318](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L318) is due to usage of the wrong looping variable as an index inside `collateralTokens[]`:
```js
  File: contracts/RestakeManager.sol

  315:                          // record token value of withdraw queue
  316:                          if (!withdrawQueueTokenBalanceRecorded) {
  317:                              totalWithdrawalQueueValue += renzoOracle.lookupTokenValue(
  318: @--->                            collateralTokens[i],    // @audit : should be `j` instead of `i`
  319:                                  collateralTokens[j].balanceOf(withdrawQueue)
  320:                              );
  321:                          }
```

The variable `j` loops through all the tokens while `i` loops through all the operatorDelegators. Thus **only the first token's price** is used to [lookupTokenValue()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L307) inside [RenzoOracle.sol](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L71), thus giving incorrect results. The reason "only the first token's price" is used is because [withdrawQueueTokenBalanceRecorded is set to true on L344](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L344) and hence `totalWithdrawalQueueValue` [is calculated only once on L317](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L317).

## Impact
There are multiple critical impacts of this:
- **Impact-1:** The call to [deposit()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L500-L511) can revert with `MaxTVLReached()` error in case `totalTVL` is calculated as less than actual OR it can allow more than targeted deposits.

- **Impact-2:** The call to [depositETH()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L594-L598) can revert with `MaxTVLReached()` error in case `totalTVL` is calculated as less than actual OR it can allow more than targeted deposits. 

- **Impact-3:** [Incorrect amount of ezETH minted](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L605-L612) since the `totalTVL` passed to `renzoOracle.calculateMintAmount()` is incorrect. 

- **Impact-4:** Choice of incorrect OperatorDelegator inside `chooseOperatorDelegatorForDeposit()`. Whenever the `sweepERC20()` function is called, it internally calls [depositTokenRewardsFromProtocol() on L269](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L269) which is responsible for determining which OperatorDelegator to use. This choice is made by calling `chooseOperatorDelegatorForDeposit()` on [L655](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L654-L658) which accepts `totalTVL` as a parameter in order to make these calculations. This `totalTVL` is calculated a couple of lines above on [L652](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L652) via a call to [calculateTVLs()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L274).<br>
Since [chooseOperatorDelegatorForDeposit()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L374) chooses an operatorDelegator with TVL below the threshold, an incorrectly calculated `totalTVL` results in some operators either missing out, or gaining an unfair chance to be chosen:
```js
  File: contracts/RestakeManager.sol

  374: @--->            // Otherwise, find the operator delegator with TVL below the threshold
  375:                  uint256 tvlLength = tvls.length;
  376:                  for (uint256 i = 0; i < tvlLength; ) {
  377:                      if (
  378:                          tvls[i] <
  379: @--->                    (operatorDelegatorAllocations[operatorDelegators[i]] * totalTVL) /
  380:                              BASIS_POINTS /
  381:                              BASIS_POINTS
  382:                      ) {
  383:                          return operatorDelegators[i];
  384:                      }
  385:
  386:                      unchecked {
  387:                          ++i;
  388:                      }
  389:                  }
```

## Proof of Concept
https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L318

## Tools Used
Manual review

## Recommended Mitigation Steps
Change `i` to `j` on [L318](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L318):
```diff
                // record token value of withdraw queue
                if (!withdrawQueueTokenBalanceRecorded) {
                    totalWithdrawalQueueValue += renzoOracle.lookupTokenValue(
-                       collateralTokens[i],
+                       collateralTokens[j],
                        collateralTokens[j].balanceOf(withdrawQueue)
                    );
                }
```

[Back to Top](#summaryTable)

---

### <a id="h-03"></a>[H-03]
## **Calculating `amountToRedeem` inside `WithdrawQueue::withdraw()` instead of within `claim()` allows front-running a slashing event while also causing redemption of incorrect amount**
#### https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L216-L232
<br>

## Summary
When a user requests a `withdraw()` they also provide the `_assetOut` token (ETH or other ERC20 token). The `amountToRedeem` of this token is [calcuated inside the withdraw() function itself and cached](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L216-L232). This is followed by a [coolDownPeriod post which the claim() function can be called](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L287) to make the [actual transfer](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L301-L309) of `amountToRedeem` to the `msg.sender`.
<br>

This use of cached values has multiple repurcussions as detailed in the next section.

## Impact
- **Impact-1:**
    - **_Malicious stakers can front-run a slashing event and submit their withdrawal requests, causing higher penalties for the remaining stakers while escaping themselves_**: Every time a slashing event happens, the TVL decreases. This decrease in TVL impacts the `amountToRedeem` which is obvious from the [calculateRedeemAmount() calculation](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L158) which has `_currentValueInProtocol` term in the numerator. Note that `calculateRedeemAmount()` is [called from inside withdraw() with totalTVL passed as a param for currentValueInProtocol](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L220-L224). 
    A malicious user who witnesses any such slashing event can front-run it and call `withdraw()`. Since the TVL has not yet been reduced, he is able to escape the effect of this TVL reduction. This also means that the remaining users who did not front-run have now to proportionally bear a higher amount of TVL loss. This is high impact because this now would become the de-facto behaviour of all stakers and anyone missing out is the one bearing higher losses.

- **Impact-2:**
    - **_Protocol may payout a higher `amountToRedeem` due to usage of stale price_**: Since the `amountToRedeem` is calculated [based on a price lookup using the oracle](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L229) via a call to [lookupTokenAmountFromValue()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L85) from inside of the `withdraw()` function and cached, it may not be the current price when finally `claim()` gets executed after the cooldown period. If the price has meanwhile dropped, the protocol ends up transferring more than the right amount. Conversely, the **_protocol can also use this loophole and engage in malicious behaviour_** in the following way:
        - The `cooldDownPeriod` can be updated by the admin via a call to [updateCoolDownPeriod()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L129) at any point of time - even between the duration of the user calling `withdraw()` and `claim()`. If the protocol feels the price movement has happened against their interest, they can wait it out by increasing the cooldown period and hoping for a more favourable rate later on.
<br>

Although the two aforementioned impacts look mutually exclusive, decided to raise them under this same bug report since they share the same root cause & hence the fix is the same - relocating the TVL & price calculations. Please refer the '_Recommended Mitigation Steps_' section below.

## Proof of Concept
https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L216-L232

## Tools Used
Manual code review

## Recommended Mitigation Steps
It's recommended to move the calculation of `totalTVL` and `amountToRedeem` out from `withdraw()` and into `claim()` so that rates & values are decided at actual time of disbursement. This will also thwart any attempt to book gains by front-running a slashing event since the TVL would now be calculated after the cooldown period inside `claim()`.

[Back to Top](#summaryTable)

---

### <a id="h-04"></a>[H-04]
## **totalTVL is incorrectly calculated due to the addition of DepositQueue's entire balance**
#### https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L352
<br>

## Summary
`calculateTVLs()` calculates the `totalTVL` by adding the entire native ETH held in the deposit queue. This is incorrect since a part of the deposit queue's balance goes towards refunding the gas cost of the admin. In certain situations this results in the protocol paying out more redemption amount than it should at the the time of withdrawal & claim. 

## Impact
Consider the following flow:
- DepositQueue has `32.01 ether` in it at timestamp `t`.
- Before the admin calls [stakeEthFromQueue()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L187), a withdrawal request is made by a user by calling `withdraw()`.
- `withdraw()` internally calls [calculateTVLs()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L217) to calculate `totalTVL`. Higher the `totalTVL`, higher the `amountToRedeem` transferred to the user in exchange for his `ezETH`. This is because `totalTVL` is used to calculate `amountToRedeem` through a call to [lookupTokenAmountFromValue()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L220-L224) where the `totalTVL` (or `_currentValueInProtocol` as it is known in the param list) [is in the numerator](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L158). 
- As can be seen inside `calculateTVLs()`, the [deposit queue's balance is added to compute the totalTVL](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L352). This is of course done under the assumption that the entire balance of the deposit queue is anyway going to be staked sooner or later and hence should be a part of `totalTVL`.
- Hence the user redeems a certain amount.
- At `t + 1` or later `stakeEthFromQueue()` is called. While `32 ether` [is forwarded via restakeManager.stakeEthInOperatorDelegator()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L195), the remaining `.01 ether` [is used to refund the admin's gas cost](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L205) (assuming here that the admin had to spend `.01 ether` as gas cost). 
- [_refundGas()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L283) of course uses funds from the balance of the `DepositQueue` contract itself for this gas refund.
- Thus the **_enitre balance of the deposit queue was never restaked by the protocol_** and hence shouldn't have been considered in advance as part of it's TVL.

Hence the protocol just redeemed a higher amount to the user under the assumption that it possesses more TVL than it actually does. Please note that this discrepancy in actual and assumed TVLs would be higher in case where there are more funds in the `DepositQueue` contract and are waiting for `stakeEthFromQueue()` or `stakeEthFromQueueMulti()` to be called, because the gas cost refund could be potentially higher too.
<br>

To visualize an extreme case, one could imagine that the user had received the shares when `DepositQueue` had a balance of `32 ether`. He then calls `withdraw()` within a few seconds when `DepositQueue` balance has incremented to `32.01 ether`. The user would receive more amount on redemption and hence will book a profit. On the other hand, upon calling `stakeEthFromQueue()`, the protocol would discover that after covering for admin's gas cost, the TVL has actually gone down from `32.01 ether` to `32 ether`.

## Proof of Concept
https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L352

## Tools Used
Manual review

## Recommended Mitigation Steps
One way to counter this would be to have a separate contract which handles the funds for any gas refunds. That would enable us to continue using the same `totalTVL` calculation on [L352](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L352). Of course the implementation code inside `_refundGas()` will have to be changed.

[Back to Top](#summaryTable)

---

### <a id="h-05"></a>[H-05]
## **RestakeManager::deposit() is susceptible to inflation attack**
#### https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L146
<br>

## Summary
The `deposit()` and `depositETH()` functions mint `ezETH` [based on the returned value](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L565) of the [calculateMintAmount()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L123) function. `calculateMintAmount()` has [a check in place which acts as a partial safeguard](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L146) against inflation attack where it ensures that the minted amount is greater than zero:
```js
        if (mintAmount == 0) revert InvalidTokenAmount();
```

This is however, only partially effective and inflation attack is still possible as described in the next section.

## Impact
Consider the following attack:
- Initially ezETH supply and total TVL are zero.
- Victim attempts to deposit `200 ETH` or `200e18 wei`.
- Attacker front-runs the victim & calls `depositETH()` to deposit `1 wei` and gets `1 ezETH` in return.
- In the same transaction the attacker donates `100 ETH` or `100e18 wei` to the `DepositQueue.sol` contract. This is done because any balance inside `DepositQueue` is considered to be part of `totalTVL` as per the calculation [here](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L352).
- The ezETH supply currently is `1` while the TVL is `1 + 100e18`. The TVL is 1 more than half of 200e18 where 200e18 is the amount victim wants to deposit.
- The victim's transaction now goes through and [as per the calculations inside calculateMintAmount()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L135-L143) he receives just `1 ezETH`.
- The attacker now back-runs and calls [withdraw()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L206) to redeem his `1 ezETH`. Since he owns half of the ezETH supply, he is able to redeem 50% of totalTVL which is approximately `300e18 / 2 = 150e18`, thus making a profit of `50e18 wei` or `50 ETH`.

## Proof of Concept
https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L146

## Tools Used
Manual review

## Recommended Mitigation Steps
Consider using the 'dead shares' technique [used by UniswapV2](https://github.com/Uniswap/v2-core/blob/ee547b17853e71ed4e0101ccfd52e70d5acded58/contracts/UniswapV2Pair.sol#L121). For further desciption refer [OZ's docs here](https://docs.openzeppelin.com/contracts/5.x/erc4626) and [mixbytes.io explanation here](https://mixbytes.io/blog/overview-of-the-inflation-attack).

[Back to Top](#summaryTable)

---

### <a id="h-06"></a>[H-06]
## **Funds get stuck and LRT rate drops when a staker is undelegated**
#### https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L269
<br>

## Description
`OperatorDelegator.sol` allows `onlyNativeEthRestakeAdmin` to call `queueWithdrawals()` which [tracks the shares for TVL accounting inside the queuedShares[] variable](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L232). This is followed up by a call to `completeQueuedWithdrawal()` which [further updates queuedShares[]](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L281). Also, `completeQueuedWithdrawal()` [internally calls delegationManager.completeQueuedWithdrawal()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L273-L274) which finally reaches [this checkpoint](https://github.com/Layr-Labs/eigenlayer-contracts/blob/dev/src/contracts/core/DelegationManager.sol#L574-L577):
```js
        require(
            msg.sender == withdrawal.withdrawer, 
            "DelegationManager._completeQueuedWithdrawal: only withdrawer can complete action"
        );
```
This passes because the `msg.sender` is correctly set inside `queueWithdrawals()` to the `OperatorDelegator.sol` contract address [here](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L229).
<br>

The above logic however breaks when [undelegate()](https://github.com/Layr-Labs/eigenlayer-contracts/blob/dev/src/contracts/core/DelegationManager.sol#L206-L211) is called outside of the protocol by either the staker, their operator or the operator's delegationApprover:
```js
    /**
     * Allows the staker, the staker's operator, or that operator's delegationApprover to undelegate
     * a staker from their operator. Undelegation immediately removes ALL active shares/strategies from
     * both the staker and operator, and places the shares and strategies in the withdrawal queue
     */
    function undelegate( ...
```
As can be seen in the above [natspec](https://github.com/Layr-Labs/eigenlayer-contracts/blob/dev/src/contracts/core/DelegationManager.sol#L206-L211), it's clear that the shares are moved to the withdrawal queue **_without the `queuedShares[]` variable knowing about it_**. Also, the value of `podOwnerShares` comes down to zero, thus reducing the overall `totalTVL` [which depends on tracking the staked ETH balance](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L329).
<br>

This negatively impacts the protocol primarily in two ways which are mentioned in the next section.

## Impact
- **Impact-1**: There's no way to retrieve the shares from the withdrawal queue now. `OperatorDelegator.sol` has only one function `completeQueuedWithdrawal()` which could have done the job but will now fail the aforementioned [checkpoint](https://github.com/Layr-Labs/eigenlayer-contracts/blob/dev/src/contracts/core/DelegationManager.sol#L574-L577) since the `msg.sender` needs to be the address of the one who undelegated and not the `OperatorDelegator.sol` contract address. 

- **Impact-2**: The operator has now achieved LRT (ezETH) price manipulation. Since the `totalTVL` has dropped (because `podOwnerShares` is 0 and `queuedShares[]` doesn't know about the shares in the withdrawal queue), any new deposits will result in minting of ezETH at a discounted rate due to [the calculations inside calculateMintAmount()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L136) where the `_currentValueInProtocol` or totalTVL is used. A similar impact is seen during withdraw causing a loss to users who deposited before such a event and are redeeming now due to the [calculation inside calculateRedeemAmount()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L158).

## Proof of Concept
- https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L269
- https://github.com/Layr-Labs/eigenlayer-contracts/blob/dev/src/contracts/core/DelegationManager.sol#L206-L211
- https://github.com/Layr-Labs/eigenlayer-contracts/blob/dev/src/contracts/core/DelegationManager.sol#L574-L577

## Tools Used
Manual review

## Recommended Mitigation Steps
1. Implement a `permissionedCompleteQueuedWithdrawal()` function inside `OperatorDelegator.sol` which allows a whitelisted address to call `delegationManager.completeQueuedWithdrawal()`. The `onlyNativeEthRestakeAdmin` role could be responsible for granting permission & whitelisting a particular address in such special cases.
2. Handling the `undelegate()` scenario and thus the TVL drop is not straightforward. One approach could be to check if all stakers are active whenever a call is made to deposit() or withdraw(), and also if any shares exist in EigenLayer's queue.

[Back to Top](#summaryTable)

---
 
### <a id="h-07"></a>[H-07]
## **verifyWithdrawalCredentials() will revert and some validators can't be verified if any of the operators have added a validator directly, from outside the protocol**
#### https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L385
<br>

## Summary
The `stakedButNotVerifiedEth` variable is incremented by the protocol whenever [stakeEth() is called](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L358). Hence when later on `verifyWithdrawalCredentials()` is called, the protocol [decreases the stakedButNotVerifiedEth variable](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L385) assuming that it can't underflow. This assumption is incorrect and can render the protocol unable to verify validators under certain conditions.

## Impact
Operators are always free to add their own validators from outside the protocol and stake [by making a direct call](https://github.com/Layr-Labs/eigenlayer-contracts/blob/dev/src/contracts/pods/EigenPodManager.sol#L81-L92) to `eigenPodManager.stake{ value: value }(pubkey, signature, depositDataRoot)`. In such a case, the protocol's `stakedButNotVerifiedEth` variable is never incremented. Considering this, let's consider the following flow:
- An operator adds a validator by calling Renzo's [stakeEth{value: 32 ether}()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L349) function. As a result, the protocol updates `stakedButNotVerifiedEth = 32 ether` on [L358](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L358).
- Another validator is added from outside of Renzo by directly calling EigenLayer's [stake{value: 32 ether}() function](https://github.com/Layr-Labs/eigenlayer-contracts/blob/dev/src/contracts/pods/EigenPodManager.sol#L81-L92). Since Renzo does not know about it, `stakedButNotVerifiedEth` is still `32 ether`. **_Note that it's nowhere required or mentioned by Renzo under their trust assumptions that operators are not allowed to add validators on their own, or that Renzo wouldn't verify validators added from outside_**.
- Admin calls [verifyWithdrawalCredentials()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L364) to verify the 2nd validator with the proofs supplied to him. It goes through correctly and `stakedButNotVerifiedEth` is [decreased to zero](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L385). This is because the `BeaconChainProofs.getEffectiveBalanceGwei()` call on [L382](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L382) returns the correct value of `32 ether` for the 2nd validator.
- Admin now calls `verifyWithdrawalCredentials()` once again, this time to verify the 1st validator. This reverts now due to underflow as `stakedButNotVerifiedEth` is zero. 

Although the admin can try again later once `stakedButNotVerifiedEth` increments to at least 32 ether due to addition of other validators, the `stakedButNotVerifiedEth` variable will now always be out of sync and **_the last validator can never be verified by the protocol_**.

## Proof of Concept
https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L385

## Tools Used
Manual review

## Recommended Mitigation Steps
While one approach could be to not allow operators to add validators from outside or simply not verify such validators, a better code-based approach would be to maintain a validator based `stakedButNotVerifiedEthForValidator` mapping. At the time of decreasing `stakedButNotVerifiedEth` inside `verifyWithdrawalCredentials()`, it can be checked that we do not deduct an amount greater than this value.

[Back to Top](#summaryTable)

<br><br>

## **MEDIUM-SEVERITY BUGS**
---
 
### <a id="m-01"></a>[M-01]
## **Gas refund amount as well as staking start time are dependent upon admin's choice of call between `stakeEthFromQueue()` and `stakeEthFromQueueMulti()`**
#### https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L227-L249
#### https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L205
<br>

## Description
The `NativeEthRestakeAdmin` role can either call [stakeEthFromQueue()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L187) to initiate the restaking process for a single `operatorDelegator` or [stakeEthFromQueueMulti()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L211) for multiple validators in a single transaction. Each function -
- calls `restakeManager.stakeEthInOperatorDelegator()` with `{ value: 32 ether }` for each validator on [L195](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L195) and [L229](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L229).
- makes an internal call to `_refundGas()` (on [L205](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L205) and [L249](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L249) respectively) which refunds the gas to the admin **_if enough ETH is available_** [in the contract](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L285).

However the different orders in which `restakeManager.stakeEthInOperatorDelegator()` and `_refundGas()` are called in these two functions influences whether the admin gets his gas refund or not. It also influences **_when_** the staking commences as it could be delayed based on the admin's choice of functions. Consider the following flow -
- The `DepositQueue.sol` contract's balance is `64 ether`.
- Admin wants to initiate the staking for `operatorDelegator` OD-1 and OD-2.
- **Scenario-1:**
    - Admin calls `stakeEthFromQueue()` for OD-1. He spends `0.1 ether` of gas while doing this.
    - The `0.1 ether` gets refunded completely due the call on [L205](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L205).
    - Admin can't immediately call `stakeEthFromQueue()` for OD-2 as it will revert since the contract's balance is now less than `32 ether` after spending `32 + 0.1 = 32.1 ether` in the first transaction.
    - Admin now waits for `t` time until contract balance reaches `32 ether` again. 
- **Scenario-2:**
    - Admin instead of individual calls, decides to call `stakeEthFromQueueMulti()` for OD-1 & OD-2 combined. He spends `0.2 ether` of gas during this tx.
    - Since the code first loops & stakes **for all the ODs** on [L227](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L227) and **only after that** tries to refund gas on [L249](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L249), the admin will get no refund since no balance remains after the loop concludes. Note that staking for both the ODs succeeded, with no wait time of `t` for OD-2 as was the case in Scenario-1.

## Impact
Depending on how you look at it, this can be seen as either -
- Loss of admin funds spent in gas due to making a bad choice knowingly/unknowingly OR
- Delay in OD-2's staking. Any delay is akin to delay of rewards/interest & hence loss of funds.

## Proof of Concept
https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L227-L249

## Tools Used
Manual review

## Recommended Mitigation Steps
It might be a better option to give admin the choice to sweep the contract & withdraw up to a certain amount every time the balance is above a certain threshold, which can then be used to refund gas. This could be timelocked, where the function call is allowed only once every week or so. The admin gas usage of previous stakes will need to be recorded in a separate variable for this solution to work.

[Back to Top](#summaryTable)
 
---
 
### <a id="m-02"></a>[M-02]
## **Failed ERC20 transfer inside `claim()` results in permanent loss of funds**
#### https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L305
<br>

## Description
Although this issue has been mentioned in the [automated findings](https://github.com/code-423n4/2024-04-renzo/blob/main/4naly3er-report.md#m-9-return-values-of-transfertransferfrom-not-checked), I believe it warrants a clear mention here due to the impact being loss of funds with no way for the user to retry the transaction.
<br>

The `claim()` function [uses the transfer()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L305) function from the [OZ IERC20 interface](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/IERC20.sol#L34-L41) but never checks it's return value to see if it executed successfully or not. If the transfer returns `false` for some reason, there is no way for the user to claim their fund again and they are lost permanently. <br>
This is because the code already has deducted the `amountToRedeem` before the transfer attempt and also deleted the withdrawal request itself from `withdrawRequests[msg.sender][]`. The `ezETH` is burned too:
```js
  File: contracts/Withdraw/WithdrawQueue.sol

  279:              function claim(uint256 withdrawRequestIndex) external nonReentrant {
  280:                  // check if provided withdrawRequest Index is valid
  281:                  if (withdrawRequestIndex >= withdrawRequests[msg.sender].length)
  282:                      revert InvalidWithdrawIndex();
  283:
  284:                  WithdrawRequest memory _withdrawRequest = withdrawRequests[msg.sender][
  285:                      withdrawRequestIndex
  286:                  ];
  287:                  if (block.timestamp - _withdrawRequest.createdAt < coolDownPeriod) revert EarlyClaim();
  288:
  289: @--->            // subtract value from claim reserve for claim asset
  290:                  claimReserve[_withdrawRequest.collateralToken] -= _withdrawRequest.amountToRedeem;
  291:
  292: @--->            // delete the withdraw request
  293:                  withdrawRequests[msg.sender][withdrawRequestIndex] = withdrawRequests[msg.sender][
  294:                      withdrawRequests[msg.sender].length - 1
  295:                  ];
  296:                  withdrawRequests[msg.sender].pop();
  297:
  298: @--->            // burn ezETH locked for withdraw request
  299:                  ezETH.burn(address(this), _withdrawRequest.ezETHLocked);
  300:
  301:                  // send selected redeem asset to user
  302:                  if (_withdrawRequest.collateralToken == IS_NATIVE) {
  303:                      payable(msg.sender).transfer(_withdrawRequest.amountToRedeem);
  304:                  } else {
  305: @--->                IERC20(_withdrawRequest.collateralToken).transfer(
  306:                          msg.sender,
  307:                          _withdrawRequest.amountToRedeem
  308:                      );
  309:                  }
  310:                  // emit the event
  311:                  emit WithdrawRequestClaimed(_withdrawRequest);
  312:              }
```

## Impact
Permanent loss of funds if `transfer()` fails.

## Proof of Concept
https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L305

## Tools Used
Manual review

## Recommended Mitigation Steps
Use [safeTransfer()](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/utils/SafeERC20.sol#L36) instead of `transfer`.

[Back to Top](#summaryTable)

---

### <a id="m-03"></a>[M-03]
## **Lack of slippage and deadline during withdraw and deposit**
#### https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L229
#### https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L565
#### https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L605
<br>

## Impact
When users call `withdraw()` to burn their `ezETH` and receive redemption amount in return, there is no provision to provide any slippage & deadline params. This is necessary because the `withdraw()` function [uses values from the oracle](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L229) and the users may get a worse rate than they planned for. Additionally, the `withdraw()` function also makes use of calls to `calculateTVLs()` to fetch the current `totalTVL`. The `calculateTVLs()` function [makes use of oracle prices too](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L317). Note that though there is a `MAX_TIME_WINDOW` inside these oracle lookup functions, the users are forced to rely on this hardcoded value & can't provide a deadline from their side. 
These facts are apart from the consideration that users' call to `withdraw()` could very well be unintentionally/intentionally front-run which causes a drop in `totalTVL`. <br>
In all of these situations, users receive less than they bargained for and hence a slippage and deadline parameter is necessary.
<br>

Similar issue can be seen inside [deposit()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L565) and [depositETH()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L605).

## Proof of Concept
- https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L229
- https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L565
- https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L605

## Tools Used
Manual review

## Recommended Mitigation Steps
Allow users to pass a slippage tolerance value and a deadline parameter while calling these functions.

[Back to Top](#summaryTable)

---

### <a id="m-04"></a>[M-04]
## **Change of strategy via `setTokenStrategy()` will cause incorrect TVL calculation**
#### https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L106-L114
<br>

## Description
When [setTokenStrategy()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L106-L114) is called by the admin, it ignores to account for any existing token balance in the current strategy before updating the `tokenStrategyMapping` to a new strategy. Such existing balance is never migrated from the old to the new strategy. This will result in a reduced TVL figure when it is calculated inside `calculateTVLs()` since [it internally calls](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L302) `getTokenBalanceFromStrategy()` which would have no idea of any previous balance prior to the change. Effectively, the previous fund is lost.

## Impact
This results in pushing the `ezETH` price downwards and new users calling `deposit()` will get more than intended shares due to a reduced calculated TVL figure. Also, any previous users who had bought `ezETH` before the strategy update will now get a worse than expected redemption amount upon calling `withdraw()`.
<br>

For some supplementary context, [this discussion thread](https://github.com/code-423n4/2023-11-kelp-findings/issues/197#issuecomment-1837271970) can be referred to from a near-identical [issue raised in a previous C4 competition](https://code4rena.com/reports/2023-11-kelp#m-01-update-in-strategy-will-cause-wrong-issuance-of-shares).

## Proof of Concept
https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L106-L114

## Tools Used
Manual review

## Recommended Mitigation Steps
While updating the strategy, ensure that the existing balance in any previous strategy for the particular token is accounted for.

[Back to Top](#summaryTable)

---

### <a id="m-05"></a>[M-05]
## **`removeOperatorDelegator()` does not check if the OD being removed has any existing staked balance**
#### https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L160-L180
<br>

## Impact
The [removeOperatorDelegator() function](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L160-L180) allows the admin to remove an existing OperatorDelegator (OD). It removes the OD from the `operatorDelegators[]` array. However it does not check for any existing balance associated with the OD before the removal. Failure to do so means a sudden reduction in -
- The `totalTVL` calculated inside `calculateTVLs()` [here](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L302) and [here](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L329).
- The rewards calculated inside `getTotalRewardsEarned()` [here](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L700).

## Proof of Concept
https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L160-L180

## Tools Used
Manual review

## Recommended Mitigation Steps
Add a `require()` check inside `removeOperatorDelegator()` which allows removal of the OD only if their balance is zero.

[Back to Top](#summaryTable)

---

### <a id="m-06"></a>[M-06]
## **Even after `removeCollateralToken()`, user can withdraw funds using the removed token**
#### https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L244-L263
<br>

## Impact
A supported token can be removed by the admin through [removeCollateralToken()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L244-L263). It's fair to assume that the action to remove an existing token could be taken in light of some suspicious activity, for example abnormal price manipulation or swings witnessed for the token or some other extreme event. And hence the protocol would want that interactions with the token are cut off.<br>
The `removeCollateralToken()` function updates the `collateralTokens[]` array by removing the said `_token` when called by the role `RestakeManagerAdmin`. However, existing configurations of `withdrawalBufferTarget[_token]`, `tokenOracleLookup[_token]` and `collateralTokenTvlLimits[_token]` are never reset by the protocol unless explicitly called by other admin roles. 
<br> 

**_This is non-trivial because these functions are supposed to be called by different roles_**:
- [removeCollateralToken()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L244-L263) and [setTokenTvlLimit()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L709) have the `onlyRestakeManagerAdmin` modifier. 
- [setOracleAddress()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L52-L57) has the `onlyOracleAdmin` modifier. 
- [updateWithdrawBufferTarget()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L103-L108) has the `onlyWithdrawQueueAdmin` modifier.

Hence if `removeCollateralToken()` for `_token` is called by the `RestakeManagerAdmin` first, a user can back-run this and can call [withdraw()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L206) and choose to receive the redemption amount in `_token` amount. His call will pass the [withdrawalBufferTarget check here](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L211) and the [oracle existence check here](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L229) since they were never automatically reset upon removal of the collateral token.<br>
Given that `_token` is witnessing suspicious price manipulation activity, the user could potentially extract more value than the protocol intended to provide.

## Proof of Concept
https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L244-L263

## Tools Used
Manual review

## Recommended Mitigation Steps
Inside `removeCollateralToken()`, reset the token's oracle address, the withdraw buffer target as well as the token TVL limit to zero.

[Back to Top](#summaryTable)


---

### <a id="m-07"></a>[M-07]
## **Inaccurate WETH and `xezETH` accounting inside `xRenzoBridge::xReceive()` due to absence of try/catch block**
#### https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L175-L193
#### https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L434
<br>

## Impact
The `xReceive()` function inside `xRenzoBridge.sol` does not follow the [official recommendation](https://docs.connext.network/developers/guides/handling-failures#reverts-on-receiver-contract) applicable in case of a revert while making an external call i.e. using a `try/catch` block while making external calls. <br>
It's important to note the following statement made in the Connext docs:
> If the call on the receiver contract (also referred to as "target" contract) reverts, funds sent in with the call will end up on the receiver contract. 

It then goes on to say that:
> To avoid situations where user funds get stuck on the receivers, developers should build any contract implementing IXReceive defensively.
> Ultimately, the goal should be to handle any revert-susceptible code and ensure that the logical owner of funds always maintains agency over them.

Hence, [when xRenzoDeposit::sweep() makes a xcall to](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L434-L439) `xRenzoBridge::xReceive()` with `balance`, the following happens:
- `balance` is passed into the `_amount` param [here](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L141).
- `xRenzoBridge::xReceive()` [internally calls `restakeManager.depositETH()`](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L175) which is followed by [wrapping of ezETH into xezETH via the lockbox and then burning the xezETH](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L186-L193).
- If any of these transactions involving external calls revert for any reason, for example if the [`restakeManager.depositETH()` call](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L175) reverts, then as per the comments in the Connext documentation the `_amount` WETH continues to remain in the contract. There is no provision now for an admin to retry the transaction and ensure that a correct amount of `xezETH` is burned and unwrapped WETH deposited into the `restakeManager`.<br>
Although the admin can next use [recoverERC20()](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L305) to recover the WETH, the `xezETH` accounting remains unattended to.

## Proof of Concept
- https://docs.connext.network/developers/guides/handling-failures#reverts-on-receiver-contract

## Tools Used
Manual review

## Recommended Mitigation Steps
As has been officially recommended, enclose these external call inside `xRenzoBridge::xReceive()` in a `try/catch` block which logs the failure of the transaction. Also add an admin controlled function which can then be called to retry the deposit & burn steps.

[Back to Top](#summaryTable)

