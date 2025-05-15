# Audited Code Repo
### [Sherlock: Pareto](https://audits.sherlock.xyz/contests/920)
### [Github: Pareto](https://github.com/sherlock-audit/2025-04-pareto-contest/)

<br>

# <a id="summaryTable"></a>Bugs Filed & Their Status
No findings accepted.

| #      | Bug ID          | Name | URL    | Adjudged Status  |
|--------|-----------------|------|:------:|-----------------:|
| 1      | [H-01](#h-01)   | addYieldSource() can never be called for `USDS` token and `USDS_USDC_PSM` source | [46](https://audits.sherlock.xyz/contests/920/voting/46) | Rejected |
| 2      | [H-02](#h-02)   | Missing check to ensure enough USP exists for loss socialization | [68](https://audits.sherlock.xyz/contests/920/voting/68) | Rejected |
| 3      | [H-03](#h-03)   | User can front-run depositYield() with a deposit plus stake to immediately receive rewards | [83](https://audits.sherlock.xyz/contests/920/voting/83) | Rejected |
| 4      | [H-04](#h-04)   | Vesting duration is bypassed | [100](https://audits.sherlock.xyz/contests/920/voting/100) | Rejected |

# Contest's Valid Findings
- [Defaulted Idle CV with pending withdrawals will permanently break stablecoin accounting](https://audits.sherlock.xyz/contests/920/voting/148)
- [Asymmetric Loss Distribution: Non-stakers Penalized by First-To-Withdraw Advantage After Yield Farming Losses](https://audits.sherlock.xyz/contests/920/voting/202)

<br>
<br>

## **HIGH-SEVERITY BUGS**
---

### <a id="h-01"></a>[H-01]
## **addYieldSource() can never be called for `USDS` token and `USDS_USDC_PSM` source**
#### https://github.com/sherlock-audit/2025-04-pareto-contest/blob/main/USP/src/ParetoDollarQueue.sol#L101-L104
<br>

## Description
It's not possible for the owner to call `addYieldSource()` and add `USDS_USDC_PSM` as the yield source for `USDS` token. This is because `addYieldSource()` [tries to call](https://github.com/sherlock-audit/2025-04-pareto-contest/blob/main/USP/src/ParetoDollarQueue.sol#L592-L593):
```js
    // approve the token for the yield source
    IERC20Metadata(_token).safeIncreaseAllowance(_source, type(uint256).max);
```

which will revert because `safeIncreaseAllowance()` has already been called inside [initialize()](https://github.com/sherlock-audit/2025-04-pareto-contest/blob/main/USP/src/ParetoDollarQueue.sol#L101-L104) for the same combination:
```js
    // add allowance for USDS to USDS-USDC PSM
    if (USDS != address(0) && USDS_USDC_PSM != address(0)) {
      IERC20Metadata(USDS).safeIncreaseAllowance(USDS_USDC_PSM, type(uint256).max);
    }
```

This happens because `safeIncreaseAllowance()` from OpenZeppelin [has implementation](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d183d9b/contracts/token/ERC20/utils/SafeERC20.sol#L68-L70):
```js
    function safeIncreaseAllowance(IERC20 token, address spender, uint256 value) internal {
        uint256 oldAllowance = token.allowance(address(this), spender);
@-->    forceApprove(token, spender, oldAllowance + value);
    }
```
Since `oldAllowance` has already been set to `type(uint256).max` inside the initializer, calling it again inside `addYieldSource()` will attempt `oldAllowance + value` or `type(uint256).max + type(uint256).max` and always revert due to overflow.

## Proof of Concept
Modify [Deploy.s.sol](https://github.com/sherlock-audit/2025-04-pareto-contest/blob/main/USP/script/Deploy.s.sol#L100) as shown below and run any test to see the following error message:
```js
    [FAIL: panic: arithmetic underflow or overflow (0x11)] setUp() (gas: 0)
```

```diff
  File: USP/script/Deploy.s.sol

    94:              // add sky.money sUSDS yield source
    95:              allowedMethods = new IParetoDollarQueue.Method[](3);
    96:              allowedMethods[0] = IParetoDollarQueue.Method(DEPOSIT_4626_SIG, 0);
    97:              allowedMethods[1] = IParetoDollarQueue.Method(WITHDRAW_4626_SIG, 2);
    98:              allowedMethods[2] = IParetoDollarQueue.Method(REDEEM_4626_SIG, 2);
    99:              maxCap = 100_000_000 * 1e18; // 100M USDS
-  100:              queue.addYieldSource(SUSDS, USDS, SUSDS, maxCap, allowedMethods, 2);
+  100:              queue.addYieldSource(USDS_USDC_PSM, USDS, USDC, maxCap, allowedMethods, 2);
```

## Impact
Owner has no way to call `addYieldSource()` with token = `USDS` and source = `USDS_USDC_PSM`.

## Mitigation 
Use `forceApprove()` since we always need to set allowance to `type(uint256).max`. Something along the lines of:
```diff
    function addYieldSource(
        address _source, 
        address _token, 
        address _vaultToken, 
        uint256 _maxCap, 
        Method[] calldata _allowedMethods,
        uint8 _vaultType
    ) external {
        _checkOwner();
        // revert if the token is already in the yield sources
        if (address(yieldSources[_source].token) != address(0)) {
        revert YieldSourceInvalid();
        }
        if (_source == address(0) || _token == address(0) || _vaultToken == address(0) || _allowedMethods.length == 0) {
        revert Invalid();
        }
        YieldSource memory _yieldSource = YieldSource(IERC20Metadata(_token), _source, _vaultToken, _maxCap, 0, _allowedMethods, _vaultType);
        // add the token to the yield sources mapping
        yieldSources[_source] = _yieldSource;
        // add the token to the yield sources list
        allYieldSources.push(_yieldSource);
        // approve the token for the yield source
-       IERC20Metadata(_token).safeIncreaseAllowance(_source, type(uint256).max);
+       IERC20Metadata(_token).forceApprove(_source, type(uint256).max);

        emit YieldSourceAdded(_source, _token);
    }
```

[Back to Top](#summaryTable)
---

### <a id="h-02"></a>[H-02]
## **Missing check to ensure enough USP exists for loss socialization**
#### https://github.com/sherlock-audit/2025-04-pareto-contest/blob/main/USP/src/ParetoDollarStaking.sol#L114
<br>

## Summary
The `collateral : USP` peg can be broken due to missing checks & balances. The protocol lacks any check during [withdraw](https://github.com/sherlock-audit/2025-04-pareto-contest/blob/main/USP/src/ParetoDollarStaking.sol#L114) or redeem from `ParetoDollarStaking.sol` contract to ensure sufficient USP tokens remain staked (as sUSP) to cover potential borrower defaults in credit vaults. 
While yield sources have `maxCap` parameters, there are no checks to prevent users from unstaking their sUSP after collateral has been lent out via the credit vaults (yield sources), potentially leaving the protocol unable to socialize losses during defaults through `emergencyWithdraw()` and `emergencyBurn()`.

This defeats the peg maintenance mechanism on which the protocol relies.

## Description
The protocol uses a loss socialization mechanism through the staking contract to handle borrower defaults. When a borrower defaults, the protocol owner can:
1. Use `emergencyWithdraw()` to withdraw USP from the staking contract
2. Use `emergencyBurn()` to burn these tokens, maintaining the 1:1 peg between USP and collateral. As a result, the sUSP holders bear this loss proportionally.

However, this mechanism relies on having sufficient staked USP (sUSP) to absorb losses. The protocol has no safeguards to ensure this condition is maintained throughout its operation:
1. No checks compare the total funds deployed in credit vaults against total staked USP. Even if we consider that the owner will set [maxCap](https://github.com/sherlock-audit/2025-04-pareto-contest/blob/main/USP/src/ParetoDollarQueue.sol#L511-L514) judiciously so that "excessive" collateral does not get lent out, it still leaves the following risks open.

2. No restrictions prevent users from unstaking their sUSP after collateral is deployed. While there may be enough USP staked currently, there is nothing stopping users to [withdraw](https://github.com/sherlock-audit/2025-04-pareto-contest/blob/main/USP/src/ParetoDollarStaking.sol#L114) USP by unstaking their sUSP AFTER a huge loan has been processed. If the borrower defaults now, there is just not enough USP in the staking contract for the owner to `emergencyWithdraw()` and then `emergencyBurn()` in order to bring back the peg.  

3. No dynamic adjustment of borrow limit in `IdleCDOEpochVariant.sol` or `IdleCreditVault.sol` (out of scope of this audit).


While this can happen organically, in the worst case a malicious user (whale) may also use this to purposefully take out a huge loan, default on it and also unstake their USP to ensure there's not enough USP to re-establish a peg.

## Impact
1. **Broken Peg**: Unable to burn enough USP to match the reduced collateral, potentially breaking the 1:1 peg.
2. **Liquidity Crisis**: Could trigger a "bank run" as users rush to redeem USP before others. This risk is amplified because users are incentivized to unstake during periods of perceived risk, precisely when the buffer is most needed.

In extreme cases, the protocol may be unable to honor redemptions at face value and the protocol could effectively become undercollateralized.

## Mitigation 
Several strategies could help mitigate this risk:
1. Unstaking Restrictions: Implement controlled unstaking that prevents users from unstaking if it would create an unsafe ratio. Something like:
```solidity
    require(totalStakedUSP - unstakeAmount >= totalDeployedRiskyCapital * MIN_COVERAGE_RATIO / 100, "Unstake would create unsafe ratio");
```

**It's important to NOTE that** even if the protocol does not wish to maintain the entire reserves and rather operates on the principle of fractional reserves, the above check & hence the `MIN_COVERAGE_RATIO` ratio is required. `MIN_COVERAGE_RATIO` can be configured as `less than 100` in case of fractional reserves methodology.

2. Borrowing Restrictions (out of scope of this audit): Implement limit on how much amount can be borrowed in `IdleCDOEpochVariant.sol` or `IdleCreditVault.sol`, based on staked USP (sUSP) available.

[Back to Top](#summaryTable)
---

### <a id="h-03"></a>[H-03]
## **User can front-run depositYield() with a deposit plus stake to immediately receive rewards**
#### https://github.com/sherlock-audit/2025-04-pareto-contest/blob/main/USP/src/ParetoDollarQueue.sol#L449
<br>

## Summary
Users can front-run manager triggered [depositYield()](https://github.com/sherlock-audit/2025-04-pareto-contest/blob/main/USP/src/ParetoDollarQueue.sol#L449) calls to immediately capture yield they didn't help generate, creating an unfair advantage over long-term stakers.
- Front-run by calling:
    - [mint()](https://github.com/sherlock-audit/2025-04-pareto-contest/blob/main/USP/src/ParetoDollar.sol#L93) to deposit collateral say, USDC and receive USP
    - [Stake](https://github.com/sherlock-audit/2025-04-pareto-contest/blob/main/USP/src/ParetoDollarStaking.sol#L107) this USP for sUSP and become immediately eligible for proportional rewards
- Claim rewards later by calling:
    - [Unstake](https://github.com/sherlock-audit/2025-04-pareto-contest/blob/main/USP/src/ParetoDollarStaking.sol#L114) sUSP for USP after (or during) the vesting duration, which can then be [withdrawn](https://github.com/sherlock-audit/2025-04-pareto-contest/blob/main/USP/src/ParetoDollar.sol#L118-L136) for USDC.

## Description
The protocol distributes yield to sUSP holders based on current stake rather than time-weighted contributions. When a manager calls `depositYield()`, rewards are calculated and immediately distributed to all current stakers proportionally.

This creates a timing vulnerability where users can:
1. Monitor the mempool for pending `depositYield()` transactions
2. Front-run these transactions by calling `mint()` to get USP for USDC (any collateral) and then staking USP for sUSP
3. Receive a share of rewards that were generated before their capital was deployed
4. Unstake later to capture value.

**NOTE that** this is not only a front-running issue but a general design issue in the protocol. Imagine this:
- Alice has staked 1000 USP for 1000 sUSP and is waiting for her 10-day vesting period to end so that she can receive her rewards amounting to 100 USP.
- After 5 days, Bob stakes 1000 USP and receives some sUSP shares (a bit less than 1000 as 50 additional USP has already "accrued". Here we assume that accrual or vesting of 50 USP has happened because manager called `depositYield()`. If manager has not yet called it then Bob gets the same amount of sUSP as Alice). 
- When Alice now unstakes on the 10th day, she does not get her full 100 reward but instead just slightly above 75 USP (50 for first 5 days and ~25 for next 5 days).
- Bob immediately became eligible for rewards which were already in the vesting duration and got a share of it. 

## Impact
1. Long-term stakers receive diluted rewards as new users capture value they didn't help generate
2. Creates misaligned incentives favoring short-term "yield sniping"

## Mitigation
Several possible solutions can be considered:
1. Snapshot-based rewards: Take snapshots of staker balances at regular intervals and distribute rewards based on time-weighted positions.
2. Vesting period: Implement a cooldown/vesting period for newly staked tokens before they become eligible for rewards. 

[Back to Top](#summaryTable)
---

### <a id="h-04"></a>[H-04]
## **Vesting duration is bypassed**
#### https://github.com/sherlock-audit/2025-04-pareto-contest/blob/main/USP/src/ParetoDollarStaking.sol#L182
<br>

## Summary
The vesting mechanism in `ParetoDollarStaking.sol` can be bypassed when rewards (interest) accrue in the system but haven't been formally recognized through [depositRewards()](https://github.com/sherlock-audit/2025-04-pareto-contest/blob/main/USP/src/ParetoDollarStaking.sol#L182).

## Description
When yield or interest accrues in Credit Vaults, the value of the collateral increases, which is reflected in [totalAssets()](https://github.com/sherlock-audit/2025-04-pareto-contest/blob/main/USP/src/ParetoDollarStaking.sol#L141) of the staking contract. However, the vesting mechanism only applies to rewards that have been formally recognized via `depositYield()` → `depositRewards()`.

Users who withdraw their staked USP after yield has accrued but before `depositRewards()` is called can immediately receive their full share of the yield, bypassing the intended vesting period.

The PoC in the following section confirms the following flow:
1. Alice stakes USP tokens
2. Yield or interest accrues in the system (simulated by directly adding PAR to the staking contract)
3. Alice withdraws and receives the full principal + yield immediately
4. No vesting period is applied because `rewards` and `rewardsLastDeposit` were never updated

## Root Cause
Note that this situation would occur whenever interest has accrued but either of these is true:
- `depositRewards()` has never been called yet and hence [_getUnvestedRewards()](https://github.com/sherlock-audit/2025-04-pareto-contest/blob/main/USP/src/ParetoDollarStaking.sol#L121) will return zero because `if (_timeSinceLastDeposit < _rewardsVesting)` is evaluated to `false`. This is because `rewardsLastDeposit` is zero and `_timeSinceLastDeposit` evaluates to a large value.
- No vesting period is currently ongoing and hence `_unvested` evaluates to zero inside `_getUnvestedRewards()`.

## Impact
1. This issue creates an opportunity for timing-based advantages where users can strategically withdraw their stake before `depositYield()` is called to avoid the vesting period intended to incentivize long-term staking.

2. Since `depositRewards()` never got called, protocol misses out on their `fee` too.

## Proof of Concept
Add this test inside `USP/tests/ParetoDollarStaking.t.sol` and run with `bun run test -- --mt testUnvestedRewardsIssue -vv` to see it pass:
```js
  function testUnvestedRewardsIssue() external {
    uint256 depositAmount = 1e18; // 1 USP
    
    // Set fee to 0
    vm.prank(sPar.owner());
    sPar.updateFeeParams(0, address(this));

    // 1. Alice stakes USP
    address alice = makeAddr("alice");
    uint256 shares = _stake(alice, depositAmount);

    // 2. Verify initial state
    assertEq(sPar.rewards(), 0, "Initial rewards should be 0");
    assertEq(sPar.balanceOf(alice), shares, "Alice should have correct shares");
    
    // 3. Simulate yield accrual by directly giving PAR tokens to sPar
    // This mimics interest accumulating in Credit Vaults without calling depositYield()
    give(address(par), address(sPar), depositAmount); // 100% increase (easy to calculate)
    
    // 4. Verify the yield is reflected in totalAssets
    uint256 totalAssetsBefore = sPar.totalAssets(); 
    assertApproxEqAbs(totalAssetsBefore, depositAmount * 2, 1, "Total assets should include direct transfer");
    
    // 5. Alice withdraws before depositRewards() is called
    vm.startPrank(alice);
    uint256 aliceBalanceBefore = par.balanceOf(alice);
    sPar.redeem(shares, alice, alice);
    uint256 aliceBalanceAfter = par.balanceOf(alice);
    vm.stopPrank();
    
    // 6. Verify Alice receives principal + yield with no vesting
    assertApproxEqAbs(aliceBalanceAfter - aliceBalanceBefore, depositAmount * 2, 1, 
        "Alice should receive full principal + yield with no vesting");
        
    // This test demonstrates that Alice receives all yield with no vesting restriction
    // because the rewards haven't been formally recognized through depositRewards(),
    // even though they're reflected in the total assets of the staking contract.
  }
```

## Mitigation
Consider one of these approaches:
1. **Decouple accrued yield from withdrawals**: Modify `totalAssets()` to only include formally recognized rewards based on vesting progress. This ensures yield is only distributed after proper vesting.
2. **Automatic yield recognition**: Implement a mechanism that automatically calls `depositYield()` when users interact with the contract, ensuring yield is always formally recognized before withdrawals.

[Back to Top](#summaryTable)

