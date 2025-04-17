# Leaderboard
[Symm_io Results](https://audits.sherlock.xyz/contests/838)<br>

`Rank 12 / 461`

# Audited Code Repo
### [Sherlock: Symm_io](https://audits.sherlock.xyz/contests/838)
### [Github: Symm_io](https://github.com/sherlock-audit/2025-03-symm-io-stacking/)

<br>

# <a id="summaryTable"></a>Bugs Filed & Their Status

| #      | Bug ID          | Name | URL    | Adjudged Status  |
|--------|-----------------|------|:------:|-----------------:|
| 1      | [H-01](#h-01)   | Reward rate can be diluted by attacker | [284](https://audits.sherlock.xyz/contests/838/voting/284) | Medium |
| 2      | [H-02](#h-02)   | Use of initializer modifier instead of onlyInitializing in Vesting.sol will cause reverts for more than one SymmVesting deployments | [345](https://audits.sherlock.xyz/contests/838/voting/345) | Medium |
| 3      | [H-03](#h-03)   | Incorrect constraint in resetVestingPlans() blocks admin and users from using acceptable amounts | [412](https://audits.sherlock.xyz/contests/838/voting/412) | Medium |
| 4      | [H-04](#h-04)   | Vesting commences immediately & vesting duration is extended if restVestingPlans() is called before startTime | [391](https://audits.sherlock.xyz/contests/838/voting/391) | Rejected |
| 5      | [H-05](#h-05)   | User can unlock tokens before schedule due to vesting error in addLiquidityProcess() | [489](https://audits.sherlock.xyz/contests/838/voting/489) | Rejected |
| 6      | [M-01](#m-01)   | configureRewardToken() can't remove token if user with pending rewards gets blacklisted | [292](https://audits.sherlock.xyz/contests/838/voting/292) | Rejected |
| 7      | [M-02](#m-02)   | User can't claim other tokens' rewards if they are blacklisted by any single token | [298](https://audits.sherlock.xyz/contests/838/voting/298) | Rejected |
| 8      | [M-03](#m-03)   | configureRewardToken() can get front-run and block a token's removal | [303](https://audits.sherlock.xyz/contests/838/voting/303) | Rejected |
| 9      | [M-04](#m-04)   | User can claim 1 wei of locked token at a time to avoid penalty | [430](https://audits.sherlock.xyz/contests/838/voting/430) | Rejected |
|10      | [M-05](#m-05)   | Incorrect accounting of totalVested SYMM tokens during call to `_addLiquidityProcess()` | [447](https://audits.sherlock.xyz/contests/838/voting/447) | Rejected |
|11      | [M-06](#m-06)   | Invalid amount check should be done in `_addLiquidityProcess()` after liquidity addition | [590](https://audits.sherlock.xyz/contests/838/voting/590) | Rejected |

<br>
<br>

## **HIGH-SEVERITY BUGS**
---

### <a id="h-01"></a>[H-01]
## **Reward rate can be diluted by attacker**
#### https://github.com/sherlock-audit/2025-03-symm-io-stacking/blob/main/token/contracts/staking/SymmStaking.sol#L374
<br>

## Description
The current flow allows anyone to call `notifyRewardAmount()` --> `_addRewardsForToken()` which [decreases](https://github.com/sherlock-audit/2025-03-symm-io-stacking/blob/main/token/contracts/staking/SymmStaking.sol#L374) the `state.rate` and [increases](https://github.com/sherlock-audit/2025-03-symm-io-stacking/blob/main/token/contracts/staking/SymmStaking.sol#L378) `state.periodFinish`. This dilutes rewards:
```solidity
	function _addRewardsForToken(address token, uint256 amount) internal {
		TokenRewardState storage state = rewardState[token];

		if (block.timestamp >= state.periodFinish) {
			state.rate = amount / state.duration;
		} else {
			uint256 remaining = state.periodFinish - block.timestamp;
			uint256 leftover = remaining * state.rate;
@--->      	state.rate = (amount + leftover) / state.duration;
		}

		state.lastUpdated = block.timestamp;
@--->	state.periodFinish = block.timestamp + state.duration;
	}
```

An attack vector would be:
1. Assume current `state.rate` to be `1` like it is in [existing test](https://github.com/sherlock-audit/2025-03-symm-io-stacking/blob/main/token/tests/symmStaking.behavior.ts#L96) `"should calculate reward correctly after single user deposit"`. The period duration is `1 week`. Reward amount is `604800` which `24 * 7 * 60 * 60`.
2. After some duration has passed say, half a week, attacker calls `notifyRewardAmount()` with some reward token amount as `1 wei`
3. Since half of the period duration has passed, `_addRewardsForToken()` calculates `state.rate = (amount + leftover) / state.duration = (1 + 604800/2) / 1 week` which rounds down to `0`.
4. The remaining reward is not paid now since rate has been reduced to `0`. While this is an extreme case, in other cases too the rate will get diluted. And of course the period duration keeps on getting extended with each reward addition.

## Impact
1. **Reward Rate Dilution**: Each time new rewards are added to an ongoing period, the reward rate gets reduced, providing stakers with lower rewards than expected within a defined time duration. Such delays in receiving rewards are akin to loss of funds (loss of investment opportunity due to time delays).
2. **Economic Misalignment**: The distribution pattern does not align with typical staking expectations, where reward rates should remain consistent within a single reward period.
3. **Perpetual Extension**: If rewards are added regularly, the reward period could be extended indefinitely, preventing it from ever ending.

Note that even if an attacker is not involved, normal course of operations by the admin causes this issue. See `Root Cause & Mitigation` section below for a fix recommendation.

## Proof of Concept
Add this inside `token/tests/symmStaking.behavior.ts` under the `describe("Reward Calculation"` block and run to see the following output:
```js
		it("shows bug - how reward rate can be manipulated with small amounts", async function () {
			const depositAmount = "604800"
			const initialRewardAmount = depositAmount
			
			// Setup reward token
			await stakingToken.connect(admin).approve(await symmStaking.getAddress(), initialRewardAmount)
			await symmStaking.connect(admin).configureRewardToken(await stakingToken.getAddress(), true)
			await symmStaking.connect(admin).notifyRewardAmount([await stakingToken.getAddress()], [initialRewardAmount])
			
			// Get initial reward rate
			const initialState = await symmStaking.rewardState(await stakingToken.getAddress())
			const initialRate = initialState.rate
			const initialPeriodFinish = initialState.periodFinish
			console.log("\nInitial reward rate:", initialRate.toString())
			
			// User deposits tokens
			await stakingToken.connect(user1).approve(await symmStaking.getAddress(), depositAmount)
			await symmStaking.connect(user1).deposit(depositAmount, user1.address)
		  
			// Wait some time to accumulate rewards
			await time.increase(Number(initialState.duration) / 2)
		  
			// Now simulate an attack with minimal reward amount to dilute the rate. --> user2 = attacker
			const attackAmount = "1" // Just 1 wei
			await stakingToken.connect(user2).approve(await symmStaking.getAddress(), attackAmount)
			await symmStaking.connect(user2).notifyRewardAmount([await stakingToken.getAddress()], [attackAmount])
		  
			// Check how rate was manipulated
			const attackedState = await symmStaking.rewardState(await stakingToken.getAddress())
			const attackedRate = attackedState.rate
			const attackedPeriodFinish = attackedState.periodFinish
			console.log("\nAttacked reward rate:", attackedRate.toString())
		  
			// Verify the impact of the attack
			expect(attackedPeriodFinish).to.be.gt(initialPeriodFinish) // Period has been extended
			expect(attackedRate).to.be.lt(initialRate) // Rate has been reduced
			expect(attackedRate).to.eq(0) // Rate has been reduced
		})
```

Output:
```js

Initial reward rate: 1

Attacked reward rate: 0
          ✔ shows bug - how reward rate can be manipulated with small amounts

```

## Root Cause & Mitigation 
The fundamental problem is that the current design doesn't properly separate different "tranches" of rewards. When new rewards are added, they're combined with existing rewards in a way that disrupts the original distribution plan. A truly fair system would need to:
- Keep track of separate reward schedules for each reward addition.
- Distribute each reward schedule according to its own timeline & rate.
- Although not strictly required, the protocol can even choose to make `notifyRewardAmount()` permissioned and only callable by admin as a measure of additional security.

[Back to Top](#summaryTable)
---

### <a id="h-02"></a>[H-02]
## **Use of initializer modifier instead of onlyInitializing in Vesting.sol will cause reverts for more than one SymmVesting deployments**
#### https://github.com/sherlock-audit/2025-03-symm-io-stacking/blob/main/token/contracts/vesting/Vesting.sol#L76
<br>

## Description
- Inside `Vesting.sol`, function `__vesting_init()` uses the [`initializer` modifier](https://github.com/sherlock-audit/2025-03-symm-io-stacking/blob/main/token/contracts/vesting/Vesting.sol#L76) instead of `onlyInitializing`.
- It is then inherited by `SymmVesting.sol` and its `initialize()` [calls `__vesting_init()` internally](https://github.com/sherlock-audit/2025-03-symm-io-stacking/blob/main/token/contracts/vesting/SymmVesting.sol#L77).
- The structure of params in `initialize()` in `SymmVesting.sol` shows that there could be multiple deployments of this contract (for example different POOLs or ROUTERs or VAULTs etc) OR there could be other contracts which could also inherit `Vesting.sol`. In either case, the call to `__vesting_init()` will revert since it is protected by the `initializer` modifier and can be called only once. 
- [OpenZeppelin recommmends](https://docs.openzeppelin.com/upgrades-plugins/writing-upgradeable#initializers) using the `onlyInitializing` modifier for parent contracts so that inheritance structure can be correctly deployed:
> Note that the `initializer` modifier can only be called once even when using inheritance, so parent contracts should use the `onlyInitializing` modifier

## Impact
No other child contract can be deployed which inherits from `Vesting.sol`.

## Mitigation
Use `onlyInitializing` modifier in `__vesting_init()`.

[Back to Top](#summaryTable)
---

### <a id="h-03"></a>[H-03]
## **Incorrect constraint in resetVestingPlans() blocks admin and users from using acceptable amounts**
#### https://github.com/sherlock-audit/2025-03-symm-io-stacking/blob/main/token/contracts/vesting/Vesting.sol#L231
<br>

## Description
When `resetVestingPlans() --> _resetVestingPlans()` is called the [following check](https://github.com/sherlock-audit/2025-03-symm-io-stacking/blob/main/token/contracts/vesting/Vesting.sol#L231) is not required as it stops admin from setting acceptable amounts:
```diff
  File: token/contracts/vesting/Vesting.sol

   222:          	function _resetVestingPlans(address token, address[] memory users, uint256[] memory amounts) internal {
   223:          		if (users.length != amounts.length) revert MismatchArrays();
   224:          		uint256 len = users.length;
   225:          		for (uint256 i = 0; i < len; i++) {
   226:          			address user = users[i];
   227:          			uint256 amount = amounts[i];
   228:          			// Claim any unlocked tokens before resetting.
   229:          			_claimUnlockedToken(token, user);
   230:          			VestingPlan storage vestingPlan = vestingPlans[token][user];
-  231:          			if (amount < vestingPlan.unlockedAmount()) revert AlreadyClaimedMoreThanThis();
   232:          			uint256 oldTotal = vestingPlan.lockedAmount();
   233:          			vestingPlan.resetAmount(amount);
   234:          			totalVested[token] = totalVested[token] - oldTotal + amount;
   235:          			emit VestingPlanReset(token, user, amount);
   236:          		}
   237:          	}
```

Here's the underlying mechanics:
1. On L229 we call `_claimUnlockedToken(token, user)` to claim all pending unlocked tokens yet to be claimed. What remains now is just the locked amount.
2. On L233 we call  `vestingPlan.resetAmount(amount)` which overwrites this remaining locked amount and has nothing to do with the already unlocked amount:
```solidity
  File: token/contracts/vesting/libraries/LibVestingPlan.sol

    71:          	function resetAmount(VestingPlan storage self, uint256 amount) public returns (VestingPlan storage) {
    72:          		if (claimable(self) != 0) revert ShouldClaimFirst();
    73:          		if (!isSetup(self)) revert ShouldSetupFirst();
    74:          		// Rebase the vesting plan from now.
    75:          		uint256 remaining = remainingDuration(self);
    76:          		if (remaining == 0) revert PlanIsFinished();
    77:          		self.startTime = block.timestamp;
    78:          		self.endTime = block.timestamp + remaining;
    79:@--->     		self.amount = amount;
    80:          		self.claimedAmount = 0;
    81:          		return self;
    82:          	}
```
3. Hence the check on L231 about the new `amount` being greater than `vestingPlan.unlockedAmount()` makes no sense and rather blocks the admins from using acceptable values. Example:
    - In a vesting duration of 10 days, the total amount is 100.
    - After 7 days, admin calls `resetVestingPlans()` with `amount` as 50.
    - The unlocked amount thus far is `7/10 * 100 = 70`. Since `50 < 70`, this would revert, which makes little sense. The admin can't reduce the remaining amount at all, or even increase it unless it is greater than 70.

## Impact
1. Admin can't call `resetVestingPlans()` with acceptable values.

2. Whenever a user intends to call `addLiquidity()` --> [_addLiquidityProcess() --> _resetVestingPlans()](https://github.com/sherlock-audit/2025-03-symm-io-stacking/blob/main/token/contracts/vesting/SymmVesting.sol#L156) their tx can revert if their `amount` value results in an `lpVestingPlan.lockedAmount() + lpAmount` which is less than the unlocked `SYMM_LP` amount. A valid attempt at adding liquidity would fail. (This would happen if they already added liquidity once and are attempting to do so once again after some duration of time has passed).

## Mitigation
Remove the check from inside `_resetVestingPlans()` as shown above.

[Back to Top](#summaryTable)
---

### <a id="h-04"></a>[H-04]
## **Vesting commences immediately & vesting duration is extended if restVestingPlans() is called before startTime**
#### https://github.com/sherlock-audit/2025-03-symm-io-stacking/blob/main/token/contracts/vesting/libraries/LibVestingPlan.sol#L77
<br>

## Description
If `resetVestingPlans() --> resetAmount()` is called **_before_** the original `startTime`, it gets overwritten to a longer duration & the [vesting duration commences immediately](https://github.com/sherlock-audit/2025-03-symm-io-stacking/blob/main/token/contracts/vesting/libraries/LibVestingPlan.sol#L77). 
For example:
1. Original plan: 9-month vesting with `startTime` at `T` and `endTime` at `T+9 months`
2. If reset at `T-1 month` (before starting): New plan starts immediately and ends at `current + 10 months`
```solidity
	function resetAmount(VestingPlan storage self, uint256 amount) public returns (VestingPlan storage) {
		if (claimable(self) != 0) revert ShouldClaimFirst();
		if (!isSetup(self)) revert ShouldSetupFirst();
		// Rebase the vesting plan from now.
		uint256 remaining = remainingDuration(self);  <---- this returns the remaining duration from current time which is greater than `endTime - startTime`
		if (remaining == 0) revert PlanIsFinished();
		self.startTime = block.timestamp;             <---- this causes the vesting startTime to commence immediately
		self.endTime = block.timestamp + remaining;   <---- this causes the vesting duration to be longer than intended
		self.amount = amount;
		self.claimedAmount = 0;
		return self;
	}
```

and 

```solidity
	function remainingDuration(VestingPlan storage self) public view returns (uint256) {
		return self.endTime > block.timestamp ? self.endTime - block.timestamp : 0;
	}
```

## Impact
While the overwriting of `startTime` is okay if `resetVestingPlans()` is called once vesting period has started, doing so in the above case gives rise to two issues:
1. Admin has no way to change the vested amount without causing the vesting to commence immediately instead of the prior planned `startTime` AND causing the vesting duration to increase by an amount of `startTime - currentTime`.

2. The second issue is one similar to yet a distinct vulnerability which has been reported separately under the title `"User can unlock tokens before schedule due to vesting error in addLiquidityProcess()"`. That needs to be fixed in conjunction with this one. That issue allows user to unlock tokens ahead of time via `SYMM_LP` vesting if their `SYMM` vesting schedule is due to start in the future. Note that even if that other issue is fixed, a user can:
    - first call `addLiquidity()` with a small amount (or zero). ( If in the other issue the `_setupVestingPlans()` parameter setting vulnerability is fixed inside `_addLiquidityProcess()` then `startTime` gets set properly )
    - then user calls `addLiquidity()` again with entire locked `SYMM` amount which [this time calls](https://github.com/sherlock-audit/2025-03-symm-io-stacking/blob/main/token/contracts/vesting/SymmVesting.sol#L155-L156) `_resetVestingPlans(SYMM_LP, users, amounts)` and resets the `lpVestingPlan.startTime` to current timestamp, causing the impact from our current vulnerability.

## Mitigation 
Make the following change to avoid disturbing the vesting start & end if called before `startTime`:
```diff
	function resetAmount(VestingPlan storage self, uint256 amount) public returns (VestingPlan storage) {
		if (claimable(self) != 0) revert ShouldClaimFirst();
		if (!isSetup(self)) revert ShouldSetupFirst();
		// Rebase the vesting plan from now.
		uint256 remaining = remainingDuration(self);
		if (remaining == 0) revert PlanIsFinished();
+     if (self.startTime <= block.timestamp) {
		self.startTime = block.timestamp;           
		self.endTime = block.timestamp + remaining;
+     }
		self.amount = amount;
		self.claimedAmount = 0;
		return self;
	}
```

[Back to Top](#summaryTable)
---

### <a id="h-05"></a>[H-05]
## **User can unlock tokens before schedule due to vesting error in addLiquidityProcess()**
#### https://github.com/sherlock-audit/2025-03-symm-io-stacking/blob/main/token/contracts/vesting/SymmVesting.sol#L158
<br>

## Description
When `addLiquidity()` --> [_addLiquidityProcess()](https://github.com/sherlock-audit/2025-03-symm-io-stacking/blob/main/token/contracts/vesting/SymmVesting.sol#L158) is called a new vesting schedule for `SYMM_LP` is created via `_setupVestingPlans(SYMM_LP, block.timestamp, symmVestingPlan.endTime, users, amounts)`. The second param passed by the function is `block.timestamp` which starts the vesting from current time. The function fails to validate if the user's `SYMM` vesting was due to start in the future i.e. if `symmVestingPlan.startTime > block.timestamp` is true or not. In such cases the `lpVestingPlan.startTime` needs to be equal to `symmVestingPlan.startTime` instead of `block.timestamp`.

This gives a malicious user an attack path to unlock their `SYMM` tokens before time in form of `SYMM_LP` tokens:
1. Alice has a `SYMM` vesting plan with start time `T+1` month and ending at `T+5` month i.e. vesting duration of 4 months. Let's assume quantity of `SYMM` vested as `100`.
2. Alice calls `addLiquidity()` at current timestamp of `T` and this results in locked `SYMM` to be vested as `SYMM_LP` tokens. But this is done via `_setupVestingPlans(SYMM_LP, block.timestamp, symmVestingPlan.endTime, users, amounts)` i.e. vesting is immediately started at `T`. The end time here is still `T+5` i.e. vesting duration of 5 months.
3. At `T+1`, Alice claims her unlocked `SYMM_LP` tokens `= 1/5 * 100 = 20` which under her normal `SYMM` vesting schedule would have been zero.

Note that `_resetVestingPlans()` has a similar yet distinct vulnerability which has been reported separately under the title `"Vesting commences immediately & vesting duration is extended if restVestingPlans() is called before startTime"`. That needs to be fixed in conjunction with this one. This is because even if the current issue is fixed, a user can:
    - first call `addLiquidity()` with a small amount (or zero). ( If the current `_setupVestingPlans()` parameter setting vulnerability is fixed inside `_addLiquidityProcess()` then `startTime` gets set properly )
    - then call `addLiquidity()` again with entire locked `SYMM` amount which [this time calls](https://github.com/sherlock-audit/2025-03-symm-io-stacking/blob/main/token/contracts/vesting/SymmVesting.sol#L155-L156) `_resetVestingPlans(SYMM_LP, users, amounts)` and resets the `lpVestingPlan.startTime` to current timestamp, causing the vulnerability to re-emerge.

## Impact
Malicious users can use `addLiquidity()` and `addLiquidityByPercentage()` to unlock tokens before time.

## Mitigation
This is the fix for the current param setting issue:
```diff
	function _addLiquidityProcess(
		uint256 amount,
		uint256 minLpAmount,
		uint256 maxUsdcIn
	) internal returns (uint256[] memory amountsIn, uint256 lpAmount) {
		// Claim any unlocked SYMM tokens first.
		_claimUnlockedToken(SYMM, msg.sender);

		VestingPlan storage symmVestingPlan = vestingPlans[SYMM][msg.sender];
		uint256 symmLockedAmount = symmVestingPlan.lockedAmount();
		if (symmLockedAmount < amount) revert InvalidAmount();

		_ensureSufficientBalance(SYMM, amount);

		// Add liquidity to the pool.
		(amountsIn, lpAmount) = _addLiquidity(amount, minLpAmount, maxUsdcIn);

		// Update SYMM vesting plan by reducing the locked amount.
		symmVestingPlan.resetAmount(symmLockedAmount - amountsIn[0]);

		// Claim any unlocked SYMM LP tokens.
		_claimUnlockedToken(SYMM_LP, msg.sender);

		VestingPlan storage lpVestingPlan = vestingPlans[SYMM_LP][msg.sender];

		address[] memory users = new address[](1);
		users[0] = msg.sender;
		uint256[] memory amounts = new uint256[](1);
		amounts[0] = lpVestingPlan.lockedAmount() + lpAmount;

		// Increase the locked amount by the received LP tokens.
		if (lpVestingPlan.isSetup()) {
			_resetVestingPlans(SYMM_LP, users, amounts);
		} else {
-			_setupVestingPlans(SYMM_LP, block.timestamp, symmVestingPlan.endTime, users, amounts);
+			uint256 lpVestingStartTime = block.timestamp < symmVestingPlan.startTime ? symmVestingPlan.startTime : block.timestamp;
+			_setupVestingPlans(SYMM_LP, lpVestingStartTime, symmVestingPlan.endTime, users, amounts);
		}

		emit LiquidityAdded(msg.sender, amountsIn[0], amountsIn[1], lpAmount);
	}
```

[Back to Top](#summaryTable)

<br>

## **MEDIUM-SEVERITY BUGS**
---

### <a id="m-01"></a>[M-01]
## **configureRewardToken() can't remove token if user with pending rewards gets blacklisted**
#### https://github.com/sherlock-audit/2025-03-symm-io-stacking/blob/main/token/contracts/staking/SymmStaking.sol#L320
<br>

## Description
We have inside `configureRewardToken()` the [following check](https://github.com/sherlock-audit/2025-03-symm-io-stacking/blob/main/token/contracts/staking/SymmStaking.sol#L320) while removing a reward token:
```solidity
    if (pendingRewards[token] > 10) revert OngoingRewardPeriodForToken(token, pendingRewards[token]);
```

This can result in:
1. Alice has some pending rewards in USDC which is greater than 10.
2. Alice gets blacklisted by USDC before she can claim her rewards. Now neither `claimRewards()` nor `claimFor()` can be called successfully.
3. Now `configureRewardToken(address(USDC), false)` can't be called as it would revert.

## Impact
Reward tokens which have a blacklist functionality can enter a state where they can't be removed.

## Mitigation 
We already have a `rescueTokens()` [function](https://github.com/sherlock-audit/2025-03-symm-io-stacking/blob/main/token/contracts/staking/SymmStaking.sol#L343) which can be modified:
```diff
-       function rescueTokens(address token, uint256 amount, address receiver) external nonReentrant onlyRole(DEFAULT_ADMIN_ROLE) {
+       function rescueTokens(address token, uint256 amount, address receiver, bool updatePendingRewards) external nonReentrant onlyRole(DEFAULT_ADMIN_ROLE) {
            IERC20(token).safeTransfer(receiver, amount);
+           if (updatePendingRewards) pendingRewards[token] = pendingRewards[token] - (pendingRewards[token] > amount? amount : pendingRewards[token]);
            emit RescueToken(token, amount, receiver);
        }
```
Now the admin can first call `rescueTokens(address(USDC), blackListedUserPendingRewardAmount, rescueAddress, true)` and then proceed with `configureRewardToken(address(USDC), false)`.

[Back to Top](#summaryTable)
---

### <a id="m-02"></a>[M-02]
## **User can't claim other tokens' rewards if they are blacklisted by any single token**
#### https://github.com/sherlock-audit/2025-03-symm-io-stacking/blob/main/token/contracts/staking/SymmStaking.sol#L320
<br>

## Description
Both `claimRewards()` and `claimFor()` internally call [_claimRewardsFor()](https://github.com/sherlock-audit/2025-03-symm-io-stacking/blob/main/token/contracts/staking/SymmStaking.sol#L385) which loops through all the tokens and transfers any rewards for that specific token to the user:
```solidity
	function _claimRewardsFor(address user) internal {
		uint256 length = rewardTokens.length;
@-->	for (uint256 i = 0; i < length; ) {
			address token = rewardTokens[i];
			uint256 reward = rewards[user][token];
			if (reward > 0) {
				rewards[user][token] = 0;
				pendingRewards[token] -= reward;
@-->			IERC20(token).safeTransfer(user, reward);
				emit RewardClaimed(user, token, reward);
			}
			unchecked {
				++i;
			}
		}
	}
```

This can result in:
1. Alice has some pending rewards in USDC and USDT.
2. Alice gets blacklisted by USDC before she can claim her rewards. 
3. If `claimRewards()` or `claimFor()` is called then `_claimRewardsFor()` loops through all tokens and attempts to `safeTransfer()` USDC reward amount to Alice. This reverts.
4. Alice doesn't get to receive her USDT rewards too now, which she rightfully is owed since she has not been blacklisted by USDT.

Note that even if admin calls `rescueTokens(address(USDT), usdtRewardAmount, address(alice))` now, the `pendingRewards[token]` mapping would not get updated.

## Impact
User can't claim their rewards.

## Mitigation 
There are two ways to handle this, either (or both) of which can be implemented:
1. Introduce `try-catch`:
```diff
	function _claimRewardsFor(address user) internal {
		uint256 length = rewardTokens.length;
		for (uint256 i = 0; i < length; ) {
			address token = rewardTokens[i];
			uint256 reward = rewards[user][token];
			if (reward > 0) {
-				rewards[user][token] = 0;
-				pendingRewards[token] -= reward;
-				IERC20(token).safeTransfer(user, reward);
-				emit RewardClaimed(user, token, reward);
+               try IERC20(token).safeTransfer(user, reward) {
+                   rewards[user][token] = 0;
+                   pendingRewards[token] -= reward;
+                   emit RewardClaimed(user, token, reward);
+                } catch { emit RewardClaimFailed(user, token, reward); }
			}
			unchecked {
				++i;
			}
		}
	}
```

2. Introduce a `removeRewardsForUser()` function which can only be called by the admin:
```solidity
        function removeRewardsForUser(address token, uint256 amount, address user, address adminAddress) external nonReentrant onlyRole(DEFAULT_ADMIN_ROLE) {
            pendingRewards[token] -= amount;
            rewards[user][token] -= amount;
            IERC20(token).safeTransfer(adminAddress, amount);
        }
```
Now the admin can call `removeRewardsForUser(address(USDC), rewards[user][address(USDC)], user, adminAddress)` to reset user's USDC rewards to zero and transfer them to admin address at the same time.

[Back to Top](#summaryTable)
---

### <a id="m-03"></a>[M-03]
## **configureRewardToken() can get front-run and block a token's removal**
#### https://github.com/sherlock-audit/2025-03-symm-io-stacking/blob/main/token/contracts/staking/SymmStaking.sol#L320
<br>

## Description
We have inside `configureRewardToken()` the [following check](https://github.com/sherlock-audit/2025-03-symm-io-stacking/blob/main/token/contracts/staking/SymmStaking.sol#L320) while removing a reward token:
```solidity
    if (pendingRewards[token] > 10) revert OngoingRewardPeriodForToken(token, pendingRewards[token]);
```

This can result in:
1. Admin calling `configureRewardToken(token, false)` to remove a reward token from the list.
2. It gets front-run by another tx of `notifyRewardAmount()` depositing `11 wei`. This increases `pendingRewards[token]` by `11`.
3. Now `configureRewardToken(token, false)` would revert.

Note that although on Base chain we have a private mempool and hence an attacker can't monitor it and front-run, the aforementioned sequence can still happen inadvertetnly. Also to be noted is that it's reasonable to assume that the protocol would inform it's users that a token is going to be retired and publish the dates for it and/or wait for pending rewards to be claimed. With this public knowledge, the probability increases that an attacker uses this attack path for griefing.

## Impact
Raising this as Med severity because although `configureRewardToken()` can revert, the protocol can distribute the attacker's donated tokens via `claimFor()` and continue.
It must be said though that adding `pendingRewards[token] -= amount` inside `rescueTokens()` functions would make the admin's task easier. Please refer the next section for a deatailed note.

## Mitigation 
- Firstly, `notifyRewardAmount()` should be made permissioned so that it can be called only by the admin.
- Secondly, we have a `rescueTokens()` [function](https://github.com/sherlock-audit/2025-03-symm-io-stacking/blob/main/token/contracts/staking/SymmStaking.sol#L343) which can be modified:
```diff
-       function rescueTokens(address token, uint256 amount, address receiver) external nonReentrant onlyRole(DEFAULT_ADMIN_ROLE) {
+       function rescueTokens(address token, uint256 amount, address receiver, bool updatePendingRewards) external nonReentrant onlyRole(DEFAULT_ADMIN_ROLE) {
            IERC20(token).safeTransfer(receiver, amount);
+           if (updatePendingRewards) pendingRewards[token] = pendingRewards[token] - (pendingRewards[token] > amount? amount : pendingRewards[token]);
            emit RescueToken(token, amount, receiver);
        }
```
Now the admin can first call `rescueTokens(address(USDC), blackListedUserPendingRewardAmount, rescueAddress, true)` and then proceed with `configureRewardToken(address(USDC), false)`.

[Back to Top](#summaryTable)
---

### <a id="m-04"></a>[M-04]
## **User can claim 1 wei of locked token at a time to avoid penalty**
#### https://github.com/sherlock-audit/2025-03-symm-io-stacking/blob/main/token/contracts/vesting/Vesting.sol#L290
<br>

## Description
`claimLockedToken()` --> [_claimLockedToken()](https://github.com/sherlock-audit/2025-03-symm-io-stacking/blob/main/token/contracts/vesting/Vesting.sol#L290) rounds down the penalty in favour of the user. On a chain like Base which has low gas cost, user can split their tx into smaller pieces to escape penalty:
```solidity
  File: token/contracts/vesting/Vesting.sol

   281:          	function _claimLockedToken(address token, address user, uint256 amount) internal {
   282:          		// First, claim any unlocked tokens.
   283:          		_claimUnlockedToken(token, user);
   284:          		VestingPlan storage vestingPlan = vestingPlans[token][user];
   285:          		if (vestingPlan.lockedAmount() < amount) revert InvalidAmount();
   286:          
   287:          		// Adjust the vesting plan
   288:          		vestingPlan.resetAmount(vestingPlan.lockedAmount() - amount);
   289:          		totalVested[token] -= amount;
   290:@--->     		uint256 penalty = (amount * lockedClaimPenalty) / 1e18;
   291:          
   292:          		// Ensure sufficient balance (minting if necessary)
   293:          		_ensureSufficientBalance(token, amount);
   294:          
   295:          		IERC20(token).transfer(user, amount - penalty);
   296:          		IERC20(token).transfer(lockedClaimPenaltyReceiver, penalty);
   297:          
   298:          		emit LockedTokenClaimed(token, user, amount, penalty);
   299:          	}
```

Considering current configured `lockedClaimPenalty` of `50%` or `0.5e18`, an `amount = 1` will round down to zero. If the `lockedClaimPenalty` is lower, then `amount` could be kept even higher.

## Impact
User can call `claimLockedToken()` while escaping penalty and hence withdrawing more than expected. This robs the protocol of their fee.

## Mitigation 
Round up the penalty in favour of the protocol:
```diff
	function _claimLockedToken(address token, address user, uint256 amount) internal {
		// First, claim any unlocked tokens.
		_claimUnlockedToken(token, user);
		VestingPlan storage vestingPlan = vestingPlans[token][user];
		if (vestingPlan.lockedAmount() < amount) revert InvalidAmount();

		// Adjust the vesting plan
		vestingPlan.resetAmount(vestingPlan.lockedAmount() - amount);
		totalVested[token] -= amount;
		uint256 penalty = (amount * lockedClaimPenalty) / 1e18;
+       if (((penalty * 1e18) / lockedClaimPenalty) < amount) penalty++;

		// Ensure sufficient balance (minting if necessary)
		_ensureSufficientBalance(token, amount);

		IERC20(token).transfer(user, amount - penalty);
		IERC20(token).transfer(lockedClaimPenaltyReceiver, penalty);

		emit LockedTokenClaimed(token, user, amount, penalty);
	}
```

[Back to Top](#summaryTable)
---

### <a id="m-05"></a>[M-05]
## **Incorrect accounting of totalVested SYMM tokens during call to `_addLiquidityProcess()`**
#### https://github.com/sherlock-audit/2025-03-symm-io-stacking/blob/main/token/contracts/vesting/SymmVesting.sol#L141-L142
<br>

## Description
After `resetAmount()` [is called inside](https://github.com/sherlock-audit/2025-03-symm-io-stacking/blob/main/token/contracts/vesting/SymmVesting.sol#L141-L142) `_addLiquidityProcess()`, the `totalVested` mapping is never updated. It ought to be:
```diff
   124:          	function _addLiquidityProcess(
   125:          		uint256 amount,
   126:          		uint256 minLpAmount,
   127:          		uint256 maxUsdcIn
   128:          	) internal returns (uint256[] memory amountsIn, uint256 lpAmount) {
   129:          		// Claim any unlocked SYMM tokens first.
   130:          		_claimUnlockedToken(SYMM, msg.sender);
   131:          
   132:          		VestingPlan storage symmVestingPlan = vestingPlans[SYMM][msg.sender];
   133:          		uint256 symmLockedAmount = symmVestingPlan.lockedAmount();
   134:          		if (symmLockedAmount < amount) revert InvalidAmount();
   135:          
   136:          		_ensureSufficientBalance(SYMM, amount);
   137:          
   138:          		// Add liquidity to the pool.
   139:          		(amountsIn, lpAmount) = _addLiquidity(amount, minLpAmount, maxUsdcIn);
   140:          
   141:          		// Update SYMM vesting plan by reducing the locked amount.
   142:          		symmVestingPlan.resetAmount(symmLockedAmount - amountsIn[0]);
+  143:          		totalVested[SYMM] -= amountsIn[0];
   144:          		// Claim any unlocked SYMM LP tokens.
   145:          		_claimUnlockedToken(SYMM_LP, msg.sender);
   146:          
   147:          		VestingPlan storage lpVestingPlan = vestingPlans[SYMM_LP][msg.sender];
   148:          
   149:          		address[] memory users = new address[](1);
   150:          		users[0] = msg.sender;
   151:          		uint256[] memory amounts = new uint256[](1);
   152:          		amounts[0] = lpVestingPlan.lockedAmount() + lpAmount;
   153:          
   154:          		// Increase the locked amount by the received LP tokens.
   155:          		if (lpVestingPlan.isSetup()) {
   156:          			_resetVestingPlans(SYMM_LP, users, amounts);
   157:          		} else {
   158:          			_setupVestingPlans(SYMM_LP, block.timestamp, symmVestingPlan.endTime, users, amounts);
   159:          		}
   160:          
   161:          		emit LiquidityAdded(msg.sender, amountsIn[0], amountsIn[1], lpAmount);
   162:          	}
```

## Impact
While this does not directly effect the user balances or claimable amounts, the issue could potentially cause problems if:
- Other functions (internal or external integrations) rely on `totalVested` for critical calculations
- The contract has a maximum cap on total vested tokens that uses this variable
- An audit function examines total vested tokens vs actual contract balance

## Mitigation 
Decrease `totalVested` inside `_addLiquidityProcess()` as shown above.

[Back to Top](#summaryTable)
---

### <a id="m-06"></a>[M-06]
## **Invalid amount check should be done in `_addLiquidityProcess()` after liquidity addition**
#### https://github.com/sherlock-audit/2025-03-symm-io-stacking/blob/main/token/contracts/vesting/SymmVesting.sol#L134
<br>

## Description
The check inside [_addLiquidityProcess()](https://github.com/sherlock-audit/2025-03-symm-io-stacking/blob/main/token/contracts/vesting/SymmVesting.sol#L134) should be based upon the actual `SYMM` tokens being added as liquidity  instead of that being passed by the user. This is because the actual `SYMM` tokens added can be lower. The current check does not allow the user to maintain a safe margin or buffer while passing `amount` if they wish to. The correct implementation should be:
```diff
	function _addLiquidityProcess(
		uint256 amount,
		uint256 minLpAmount,
		uint256 maxUsdcIn
	) internal returns (uint256[] memory amountsIn, uint256 lpAmount) {
		// Claim any unlocked SYMM tokens first.
		_claimUnlockedToken(SYMM, msg.sender);

		VestingPlan storage symmVestingPlan = vestingPlans[SYMM][msg.sender];
		uint256 symmLockedAmount = symmVestingPlan.lockedAmount();
-		if (symmLockedAmount < amount) revert InvalidAmount();

		_ensureSufficientBalance(SYMM, amount);

		// Add liquidity to the pool.
		(amountsIn, lpAmount) = _addLiquidity(amount, minLpAmount, maxUsdcIn);
+		if (symmLockedAmount < amountsIn[0]) revert InvalidAmount();

		// Update SYMM vesting plan by reducing the locked amount.
		symmVestingPlan.resetAmount(symmLockedAmount - amountsIn[0]);
        // .... Rest of the code
```

## Impact
Transaction could revert prematurely basing its decision on not the actual added figure.

## Mitigation 
Compare against `amountsIn[0]` instead of `amount`, as shown above.

[Back to Top](#summaryTable)