# Leaderboard
[IQAI Results](https://code4rena.com/audits/2025-01-iq-ai)<br>

`Rank 5 / 671`

# Audited Code Repo
### [Code4rena: IQAI](https://code4rena.com/audits/2025-01-iq-ai)
### [Github: IQAI](https://github.com/code-423n4/2025-01-iq-ai)

<br>

# <a id="summaryTable"></a>Bugs Filed & Their Status

| #      | Bug ID          | Name | URL    | Adjudged Status  |
|--------|-----------------|------|:------:|-----------------:|
| 1      | [H-01](#h-01)   | Attacker can either force createAgent() to always revert or steal funds from Agent | [35](https://code4rena.com/evaluate/2025-01-iq-ai/submissions/S-35) | Low |
| 2      | [H-02](#h-02)   | Rounding in favour of user & a flawed fee calculation can cause token ratios to be skewed | [58](https://code4rena.com/evaluate/2025-01-iq-ai/submissions/S-58) | Low |
| ~~3~~      | ~~[H-03](#h-03)~~   | ~~WITHDRAWN ::::::::::  getPrice() can report zero or significantly truncated price due to lack of a high enough precision saver~~ |  |  |
| 4      | [H-04](#h-04)   | No check inside BootstrapPool's buy() that agentToken balance is not less than agentTokenFeeEarned | [77](https://code4rena.com/evaluate/2025-01-iq-ai/submissions/S-77) | Low |
| 5      | [H-05](#h-05)   | DoS attack on moveLiquidity() can stop migration of bootstrap pool to Fraxswap | [83](https://code4rena.com/evaluate/2025-01-iq-ai/submissions/S-83) | Med, 9 dups |
| 6      | [H-06](#h-06)   | Existing Fraxswap pool can skew the price desired by the bootstrap pool | [119](https://code4rena.com/evaluate/2025-01-iq-ai/submissions/S-119) | Low |
| 7      | [M-01](#m-01)   | Missing slippage and deadline check in BootstrapPool's buy(), sell() | [63](https://code4rena.com/evaluate/2025-01-iq-ai/submissions/S-63) | Low |
| 8      | [M-02](#m-02)   | Missing deadline check in AgentRouter's buy(), sell() | [73](https://code4rena.com/evaluate/2025-01-iq-ai/submissions/S-73) | Low |

<br>
<br>

## **HIGH-SEVERITY BUGS**
---

### <a id="h-01"></a>[H-01]
## **Attacker can either force createAgent() to always revert or steal funds from Agent**
#### https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/AgentFactory.sol#L93
#### https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/AgentFactory.sol#L203-L216
<br>

## Description & Impact
An integral part of the protocol mechanics is to call [createAgent()](https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/AgentFactory.sol#L93) which internally calls [deployAgent()](https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/AgentFactory.sol#L203-L216) and uses `create2` with `salt = agents.length`:
```js
    function createAgent(
        string memory _name,
        string memory _symbol,
        string memory _url,
        uint256 _amountToBuy
    ) external returns (Agent agent) {
        // Collect creation fee
        if (creationFee > 0) {
            currencyToken.transferFrom(msg.sender, address(this), creationFee);
        }

        // Deploy the agent
@---->  agent = Agent(deployAgent(_name, _symbol, _url));
        ...
        ...
        ...
```

and

```js
    function deployAgent(
        string memory name,
        string memory symbol,
        string memory url
    ) internal returns (Agent agentAddress) {
@--->   uint256 salt = agents.length;
        bytes memory bytecodeWithArgs = abi.encodePacked(agentBytecode, abi.encode(name, symbol, url, address(this)));
        assembly {
@---->      agentAddress := create2(0, add(bytecodeWithArgs, 0x20), mload(bytecodeWithArgs), salt) // @audit-issue : will revert if contract already exists at the address
            if iszero(extcodesize(agentAddress)) {
                revert(0, 0)
            }
        }
    }
```

All the parameters used by `create2` here, including the `salt` is public & the deployment address can be calculated by an attacker in advance. `create2` calculates the address via `keccak256(0xff + sender_address + salt + keccak256(initialisation_code))[12:]`.
Knowing this, the attacker can - 
1. Either do the following:
    - Deploy their malicious contract at the same address by brute forcing & figuring out a `keccak256(0xff + attacker_address + attacker_salt + keccak256(attacker_initialisation_code))[12:]` combination which collides with `keccak256(0xff + sender_address + salt + keccak256(initialisation_code))[12:]`
    - Set infinite allowance for an attacker controlled address for any token they want
    - Destroy the contract using selfdestruct. Post Dencun hardfork, [selfdestruct is still possible if the contract was created in the same transaction](https://eips.ethereum.org/EIPS/eip-6780). The only catch is that all 3 of these steps must be done in one tx.
    - Now when the protocol deploys an agent on this address and the contract holds enough funds, attacker can use their allowance to steal all the funds.

2. OR deploy their malicious contract at the same address by brute forcing & figuring out a `keccak256(0xff + attacker_address + attacker_salt + keccak256(initialisation_code))[12:]` combination which collides with `keccak256(0xff + sender_address + salt + keccak256(initialisation_code))[12:]` and leave it there. Now the protocol's call to `deployAgent()` will revert. 

The feasibility of such an attack can be seen in following references:
- https://web.archive.org/web/20240304154323/https://www.mystenlabs.com/blog/ambush-attacks-on-160bit-objectids-addresses
- https://github.com/code-423n4/2024-10-ronin-validation/issues/237
- https://github.com/code-423n4/2024-10-ramses-exchange-validation/issues/216


Similar attacks can be done for `deployLiquidityManager()` and `deployGovernor()`.

## Mitigation 
Having a random `salt` will make it impossible for the attacker to predict the deployment address in advance. Something like:
```js
        bytes32 salt = keccak256(abi.encodePacked(
            agents.length,
            blockhash(block.number - 1),
            block.timestamp,
            block.chainid,
            block.difficulty,
            msg.sender
        )); // @audit-info : Can consider adding Chainlink VRF too for dependable randomness (https://docs.chain.link/vrf/v2/introduction)
```

[Back to Top](#summaryTable)
---

### <a id="h-02"></a>[H-02]
## **Rounding in favour of user & a flawed fee calculation can cause token ratios to be skewed**
#### https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/BootstrapPool.sol#L87
#### https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/BootstrapPool.sol#L151
<br>

## Description
When `buy()` is called, it first calls `getAmountOut()` and then [calculates the fee earned](https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/BootstrapPool.sol#L87):
```js
    function buy(uint256 _amountIn, address _recipient) public nonReentrant notKilled returns (uint256) {
@--->   uint256 _amountOut = getAmountOut(_amountIn, address(currencyToken));
@--->   currencyTokenFeeEarned += _amountIn - (_amountIn * fee) / 10_000;
        currencyToken.safeTransferFrom(msg.sender, address(this), _amountIn);
        agentToken.safeTransfer(_recipient, _amountOut);
        emit Swap(msg.sender, _amountIn, 0, 0, _amountOut, _recipient);
        return _amountOut;
    }
```

The fee is calculated in the aforementioned line but `getAmountOut()` independently deducts this fee from the `_amountIn` before using it to calculate `_amountOut` [here](https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/BootstrapPool.sol#L151):
```js
    function getAmountOut(uint256 _amountIn, address _tokenIn) public view notKilled returns (uint256 _amountOut) {
        uint256 _reserveIn;
        uint256 _reserveOut;
        if (_tokenIn == address(currencyToken)) {
            (_reserveIn, _reserveOut) = getReserves();
        } else if (_tokenIn == address(agentToken)) {
            (_reserveOut, _reserveIn) = getReserves();
        }
        require(_amountIn > 0 && _reserveIn > 0 && _reserveOut > 0); // INSUFFICIENT_INPUT_AMOUNT/INSUFFICIENT_LIQUIDITY
@--->   uint256 _amountInWithFee = _amountIn * fee;
        uint256 _numerator = _amountInWithFee * _reserveOut;
        uint256 _denominator = (_reserveIn * 10_000) + _amountInWithFee;
        _amountOut = _numerator / _denominator;
    }
```

The issue is that these two figures are not necessarily the same. Consider this:
1. The fee set in `AgentFactory.sol` is 0.01% or `1 bips`. Inside `BootstrapPool.sol`, the fee is transformed to `10_000 - 1 = 9999` [inside the constructor](https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/BootstrapPool.sol#L68).
2. When `getAmountOut()` is called by `buy()` with say, `_amountIn = 1`, we get `_amountInWithFee = _amountIn * fee` or `_amountInWithFee = 1 * 9999 = 9999`. Notice that this is a scaled up figure i.e. scaled up by 10_000.
3. Some outTokens get purchased.
4. Back in `buy()`, the protocol now calculates the fee `currencyTokenFeeEarned` as `_amountIn - (_amountIn * fee) / 10_000` or `1 - (1 * 9999) / 10_000` or `1 - 0` which is `1`. Due to the rounding down, **_the entire_** amountIn got taken up as fees. 
5. Subsequently, whenever [getReserves()](https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/BootstrapPool.sol#L133-L136) will be called it will not consider this fee in the token supply. Effectively, outTokens were given to the buyer but no inTokens were added to the reserves (although the amount was correctly taken from the buyer). 
6. For the next `buy()`, the buyer gets a better rate since the reserves on the inToken side remains the same.

**_Note that_** the above example takes `_amountIn` as `1 wei` to highlight the issue but this under-reporting happens continually for numerous values where the correct amount is not added to the pool reserves. I have provided such an example at the end of the report under the section `## Additional Example`, in case it's required.

Similar corrections would be required while executing `sell()` too.

## Impact
Multiple impacts:
1. Hinders with proper AMM working where the reserve of token0 (inToken) ought to go up when token1 (outToken) is transferred to the buyer.

2. **Delayed Pool Migration**: Now that `_reserveCurrencyToken` will be under-calculated (lower than what it ideally should've been) whenever `bootstrapPool.getReserves()` is called, this will delay the [necessary criteria](https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/LiquidityManager.sol#L106-L110) required for a successful `moveLiquidity()`:
```js
    /// @dev Move the liquidity from the bootstrap pool to Fraxswap
    function moveLiquidity() external {
        require(!bootstrapPool.killed(), "BootstrapPool already killed");
        uint256 price = bootstrapPool.getPrice();
@--->   (uint256 _reserveCurrencyToken, ) = bootstrapPool.getReserves();
        _reserveCurrencyToken = _reserveCurrencyToken - bootstrapPool.phantomAmount();
        uint256 factoryTargetCCYLiquidity = AgentFactory(owner).targetCCYLiquidity();
        require(
@--->       _reserveCurrencyToken >= targetCCYLiquidity || _reserveCurrencyToken >= factoryTargetCCYLiquidity,
            "Bootstrap end-criterion not reached"
        );
        ...
        ...
        ...
```

3. This also causes [getPrice()](https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/BootstrapPool.sol#L129) to always under-calculate.
```js
    /// @dev Get the price of the agent token in currency token
    /// @return _price The price of the agent token in currency token
    function getPrice() external view notKilled returns (uint256 _price) {
        (uint256 _reserveCurrencyToken, uint256 _reserveAgentToken) = getReserves(); ❌ --> lower than actual `_reserveCurrencyToken`
        _price = (_reserveCurrencyToken * 1e18) / _reserveAgentToken;  ❌ --> leads to lower than actual `_price`
    }
```
So even if we are able to go ahead with pool migration, this impacts `addLiquidityToFraxswap()`. This is because `liquidityAmount` will be calculated as a [higher than actual](https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/LiquidityManager.sol#L117-L120) value since `price` is in the denominator of the formula:
```js
        // Determine liquidity amount to add
        uint256 currencyAmount = currencyToken.balanceOf(address(this));
        uint256 liquidityAmount = (currencyAmount * 1e18) / price;  ❌ --> `price` is in the denominator, inflating `liquidityAmount`  

        // Add liquidity to Fraxswap
        IFraxswapPair fraxswapPair = addLiquidityToFraxswap(liquidityAmount, currencyAmount); ❌ --> inflated `liquidityAmount` will be added as a result of this.
```

4. **Inability to sell all tokens**: Since the reserve is not incremented but fee is, this means the bought agentTokens can't be sold. It would revert when `sell()` is called due to the [following check](https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/BootstrapPool.sol#L110): 
```js
    function sell(uint256 _amountIn, address _recipient) public nonReentrant notKilled returns (uint256) {
        uint256 _amountOut = getAmountOut(_amountIn, address(agentToken));
        agentTokenFeeEarned += _amountIn - (_amountIn * fee) / 10_000;
        agentToken.safeTransferFrom(msg.sender, address(this), _amountIn);
        currencyToken.safeTransfer(_recipient, _amountOut);
➡️➡️   require(currencyToken.balanceOf(address(this)) >= currencyTokenFeeEarned, "INSUFFICIENT_LIQUIDITY");
        emit Swap(msg.sender, 0, _amountIn, _amountOut, 0, _recipient);
        return _amountOut;
    }
```

## Proof of Concept
Add this inside `BootstrapPoolTest.sol` to see the following output when run with `forge test --mt test_bootstrap_fee_bug -vv`. In the PoC inToken is the `currencyToken` and outToken is the `agentToken`. We can see how the entire invested inToken amount is taken up as fees and never gets added to the reserves:
```js
    function test_bootstrap_fee_bug() public {
        // Set up the test environment
        setUpFraxtal(12_918_968);
        address attacker = makeAddr("Attacker");
        
        // Configure factory with minimal fee (1 bip)
        factory = new AgentFactory(currencyToken, 0);
        factory.setAgentBytecode(type(Agent).creationCode);
        factory.setGovenerBytecode(type(TokenGovernor).creationCode);
        factory.setLiquidityManagerBytecode(type(LiquidityManager).creationCode);
        factory.setTargetCCYLiquidity(100000e18);
        factory.setInitialPrice(0.99e18); 
        factory.setTradingFee(1); // 1 bip fee (minimal possible fee)

        // Create agent and bootstrap pool
        agent = factory.createAgent("VulnTest", "VULN", "https://example.com", 0);
        token = agent.token();
        LiquidityManager manager = LiquidityManager(factory.agentManager(address(agent)));
        bootstrapPool = manager.bootstrapPool();

        // Deal some currency tokens to attacker
        deal(address(currencyToken), attacker, 100); 

        // Log initial state
        console.log("Initial Setup:");
        console.log("Bootstrap Pool Address:", address(bootstrapPool));
        (uint256 reserveCurrency, uint256 reserveAgent) = bootstrapPool.getReserves();
        emit log_named_decimal_uint("Initial Currency Reserve:", reserveCurrency, 18);
        emit log_named_decimal_uint("Initial Agent Reserve:", reserveAgent, 18);
        emit log_named_decimal_uint("Phantom Amount:", bootstrapPool.phantomAmount(), 18);
        console.log("Initial Fee Earned:", bootstrapPool.currencyTokenFeeEarned());

        // Start attack simulation
        vm.startPrank(attacker);
        currencyToken.approve(address(bootstrapPool), type(uint256).max);
        
        uint256 totalCurrencySpent = 0;
        uint256 totalAgentTokensReceived = 0;

        // Perform multiple 1 wei buys
        for(uint256 i = 0; i < 100; i++) {
            uint256 amountIn = 1; // 1 wei of currency token
            uint256 balanceBefore = token.balanceOf(attacker);
            
            // Execute buy
            bootstrapPool.buy(amountIn);
            
            // Calculate tokens received
            uint256 tokensReceived = token.balanceOf(attacker) - balanceBefore;
            totalCurrencySpent += amountIn;
            totalAgentTokensReceived += tokensReceived;
        }
        vm.stopPrank();

        // Log final results
        console.log("\nAttack Results:");
        console.log("Total Currency Spent (wei):", totalCurrencySpent);
        console.log("Total Agent Tokens Received:", totalAgentTokensReceived);
        console.log("Final Fee Earned  :", bootstrapPool.currencyTokenFeeEarned());
    }
```
<br>

Output:
```js
Ran 1 test for test/BootstrapPoolTest.sol:BootstrapPoolV1Test
[PASS] test_bootstrap_fee_bug() (gas: 45737696)
Logs:
  Initial Setup:
  Bootstrap Pool Address: 0x11CF387C2ecAb9cB4F505aAC97a362aC99De058D
  Initial Currency Reserve:: 99000000.000000000000000000
  Initial Agent Reserve:: 100000000.000000000000000000
  Phantom Amount:: 99000000.000000000000000000
  Initial Fee Earned: 0

Attack Results:
  Total Currency Spent (wei): 100  // <------ 1️⃣
  Total Agent Tokens Received: 100
  Final Fee Earned  : 100  // <------ Entire amount spent in 1️⃣ is taken as fee and does not contribute to pool liquidity
```

## Mitigation 
While there are multiple ways to handle this, the following is a feasible one. Similar modifications could be required while executing `sell()` too.
Simply scale down **and then** scale up `_amountInWithFee`. Now `currencyTokenFeeEarned` inside `buy()` will match this fee figure. Also, check that `_amountInWithFee > 0`:
```diff
    function getAmountOut(uint256 _amountIn, address _tokenIn) public view notKilled returns (uint256 _amountOut) {
        uint256 _reserveIn;
        uint256 _reserveOut;
        if (_tokenIn == address(currencyToken)) {
            (_reserveIn, _reserveOut) = getReserves();
        } else if (_tokenIn == address(agentToken)) {
            (_reserveOut, _reserveIn) = getReserves();
        }
-       require(_amountIn > 0 && _reserveIn > 0 && _reserveOut > 0); // INSUFFICIENT_INPUT_AMOUNT/INSUFFICIENT_LIQUIDITY
-       uint256 _amountInWithFee = _amountIn * fee;
+       uint256 _amountInWithFee = (_amountIn * fee) / 10_000 * 10_000; // @audit-info : scale down then scale up
+       require(_amountInWithFee > 0 && _reserveIn > 0 && _reserveOut > 0); // INSUFFICIENT_INPUT_AMOUNT/INSUFFICIENT_LIQUIDITY
        uint256 _numerator = _amountInWithFee * _reserveOut;
        uint256 _denominator = (_reserveIn * 10_000) + _amountInWithFee;
        _amountOut = _numerator / _denominator;
    }
```

## Additional Example
<details>

<summary>
Click to View Additional Example
</summary>

- _amountIn = 1e18- 1 = 999999999999999999
- fee in BootStrap = 10_000 - 100 = 9900
- _amountInWithFee = _amountIn * fee = 9899999999999999990100
- Let's say _reserveOut = 100_000e18 and _reserveIn = 3e18. 
- Thus _numerator = _amountInWithFee * _reserveOut = 9899999999999999990100 * 100_000e18
- _denominator = (3e18 * 10_000) + 9899999999999999990100 = 39899999999999999990100
- _amountOut = _numerator / _denominator = 24812030075187969906156
<br>

- agentTokenFeeEarned = _amountIn - (_amountIn * fee) / 10_000 = 999999999999999999 - 9899999999999999990100/10_000 = 999999999999999999 - 989999999999999999 {0.0100 truncated} = 10000000000000000 = 0.01e18 
    - Was 989999999999999999 really used for buy?

Let's check:
- Suupose _amountInWithFee was indeed 989999999999999999 * 10_000.
- Then _numerator = _amountInWithFee * _reserveOut = 9899999999999999990000 * 100_000e18
- _denominator = (3e18 * 10_000) + 9899999999999999990000 = 39899999999999999990000
- _amountOut = _numerator / _denominator = 24812030075187969905967. This is `189` less than actual agenTokens given to user.

So if in reality `agentTokenFeeEarned` were 10_000, the user would have got `189` less agentTokens. But got more. Which means that after a fees of 10_000, the protocol gave away more agentTokens to the user than it was supposed to i.e. the buyer gets better than expected rates. 
**Or put another way**, the remaining agentTokens are now backed by a worse than expected quantity of currencyTokens.

</details>

[Back to Top](#summaryTable)
---

### <a id="h-03"></a>[H-03]
## **WITHDRAWN ::::::::::  getPrice() can report zero or significantly truncated price due to lack of a high enough precision saver**
#### https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/BootstrapPool.sol#L127
<br>

## Description
[getPrice()](https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/BootstrapPool.sol#L127) multiplies the numerator with `1e18` to prevent truncation but that is not enough to act as a precision saver for `currencyTokens` like USDC which have 6 decimal precision:
```js
    /// @dev Get the price of the agent token in currency token
    /// @return _price The price of the agent token in currency token
    function getPrice() external view notKilled returns (uint256 _price) {
        (uint256 _reserveCurrencyToken, uint256 _reserveAgentToken) = getReserves();
@--->   _price = (_reserveCurrencyToken * 1e18) / _reserveAgentToken; 
    }
```

Consider a state where the reserve ratio look like this:
- 1e6 agentTokens for each USDC, or in other words 1 USDC can buy 1M agentTokens. 
Since agentTokens has 18 decimal precision and USDC has 6, the `getReserves()` call will return `(n * 1e6, n * 1_000_000 * 1e18)` where `n` could be any number.

This is the limit of `_price` inside `getPrice()` as it returns `(_reserveCurrencyToken * 1e18) / _reserveAgentToken` or `(ne6 * 1e18) / ne24` or `1`. The moment `_reserveAgentToken` increments even by `1` (or `_reserveCurrencyToken` decreases), the `_price` returned is zero. Even in situations where it does not round down to 0, significant precision truncation happens.

**Note that** the above is an extreme example to drive home the point. However even under normal protocol operations, under-reporting by `1 wei` can continually happen. Please refer PoC section for such an example.

## Impact
This under-reporting is problematic because `getPrice()` is used while migrating the pool in [moveLiquidity()](https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/LiquidityManager.sol#L117-L120) and results in -
- EITHER a higher `liquidityAmount` to be added to Fraxswap when it is being initialized
- OR the tx reverts due to `division by zero`
```js
        // Determine liquidity amount to add
        uint256 currencyAmount = currencyToken.balanceOf(address(this));
        uint256 liquidityAmount = (currencyAmount * 1e18) / price;  ❌ --> `price` is in the denominator, inflating `liquidityAmount` OR a `division by zero` revert. 

        // Add liquidity to Fraxswap
        IFraxswapPair fraxswapPair = addLiquidityToFraxswap(liquidityAmount, currencyAmount); ❌ --> inflated `liquidityAmount` will be added as a result of this.
```

## Proof of Concept
Add this inside `BootstrapPoolTest.sol` and run to see it pass with the following output. This PoC shows how a price of zero is reported. However, even under normal protocol operations, under-reporting by `1 wei` can continually happen. To witness that behaviour please change 1️⃣ below to `factory.setInitialPrice(5.999195e6)`. You will see in the output logs that the price reported is `5.999194e6` i.e. less by `1 wei`:
```js
    function test_getPrice_bug() public {
        // Set up the test environment
        setUpFraxtal(12_918_968);
        address bob = makeAddr("Bob"); 
        
        factory = new AgentFactory(currencyToken, 0);
        factory.setAgentBytecode(type(Agent).creationCode);
        factory.setGovenerBytecode(type(TokenGovernor).creationCode);
        factory.setLiquidityManagerBytecode(type(LiquidityManager).creationCode);
        factory.setTargetCCYLiquidity(100000e18);
        factory.setTradingFee(100); 

        // @audit-info : Change this to `factory.setInitialPrice(5.999195e6)` if you want to see the second example
1️⃣-->  factory.setInitialPrice(0.000001e6); // 1 USDC = 1_000_000 AITokens i.e. 1e6 wei of USDC = 1_000_000e18 wei of AITokens

        // Create agent and bootstrap pool
        agent = factory.createAgent("VulnTest", "VULN", "https://example.com", 0);
        token = agent.token();
        LiquidityManager manager = LiquidityManager(factory.agentManager(address(agent)));
        bootstrapPool = manager.bootstrapPool();

        (uint256 reserveCurrency, uint256 reserveAgent) = bootstrapPool.getReserves();
        // Log initial state
        console.log("\nInitial State:");
        emit log_named_decimal_uint("Initial Currency Reserve:", reserveCurrency, 6);
        emit log_named_decimal_uint("Initial Agent Reserve:", reserveAgent, 18);
        emit log_named_decimal_uint("Phantom Amount:", bootstrapPool.phantomAmount(), 6);
        emit log_named_decimal_uint("Initial Price (in USDC terms):", bootstrapPool.getPrice(), 6);
        assertGt(bootstrapPool.getPrice(), 0);

        // Deal some agent tokens to Bob
        deal(address(token), bob, 1); 
        vm.prank(bob);
        // donate agentTokens (simulate a price swing causing change in reserve ratios)
        token.transfer(address(bootstrapPool), 1);

        // Log final state
        console.log("\nFinal State:");
        (reserveCurrency, reserveAgent) = bootstrapPool.getReserves();
        emit log_named_decimal_uint("Final Currency Reserve:", reserveCurrency, 6);
        emit log_named_decimal_uint("Final Agent Reserve:", reserveAgent, 18);
        emit log_named_decimal_uint("Final Price (in USDC terms):", bootstrapPool.getPrice(), 6);
        assertEq(bootstrapPool.getPrice(), 0);
    }
```
<br>

Output:
```js
[PASS] test_getPrice_bug() (gas: 43641827)
Logs:

Initial State:
  Initial Currency Reserve:: 100.000000
  Initial Agent Reserve:: 100000000.000000000000000000
  Phantom Amount:: 100.000000
  Initial Price (in USDC terms):: 0.000001

Final State:
  Final Currency Reserve:: 100.000000
  Final Agent Reserve:: 100000000.000000000000000001
  Final Price (in USDC terms):: 0.000000  ❌ <--- reported as 0
```

## Mitigation 
Use a higher `PRECISION_SAVER` like `1e36` instead of `1e18` and refactor the remaining code in line with that.

[Back to Top](#summaryTable)
---

### <a id="h-04"></a>[H-04]
## **No check inside BootstrapPool's buy() that agentToken balance is not less than than agentTokenFeeEarned**
#### https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/BootstrapPool.sol#L89-L90
<br>

## Description
BootstrapPool's `sell()` function correctly checks that `currencyToken` [balance is not less](https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/BootstrapPool.sol#L110) than `currencyTokenFeeEarned`: 
```js
    function sell(uint256 _amountIn, address _recipient) public nonReentrant notKilled returns (uint256) {
        uint256 _amountOut = getAmountOut(_amountIn, address(agentToken));
        agentTokenFeeEarned += _amountIn - (_amountIn * fee) / 10_000;
        agentToken.safeTransferFrom(msg.sender, address(this), _amountIn);
        currencyToken.safeTransfer(_recipient, _amountOut);
✔️✔️   require(currencyToken.balanceOf(address(this)) >= currencyTokenFeeEarned, "INSUFFICIENT_LIQUIDITY");
        emit Swap(msg.sender, 0, _amountIn, _amountOut, 0, _recipient);
        return _amountOut;
    }
```

Such a check is however missing from the [buy()](https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/BootstrapPool.sol#L89-L90) function:
```js
    function buy(uint256 _amountIn, address _recipient) public nonReentrant notKilled returns (uint256) {
        uint256 _amountOut = getAmountOut(_amountIn, address(currencyToken));
        currencyTokenFeeEarned += _amountIn - (_amountIn * fee) / 10_000;
        currencyToken.safeTransferFrom(msg.sender, address(this), _amountIn);
❗❗      agentToken.safeTransfer(_recipient, _amountOut);
❗❗      emit Swap(msg.sender, _amountIn, 0, 0, _amountOut, _recipient);
        return _amountOut;
    }
```

## Impact
After a buy there could be insufficient funds for the protocol to claim their `agentTokenFeeEarned` amount. This would cause `_sweepFees()` to revert and hence `kill()` won't execute and we can't migrate/graduate the pool using [moveLiquidity()](https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/LiquidityManager.sol#L113).

## Mitigation 
```diff
    function buy(uint256 _amountIn, address _recipient) public nonReentrant notKilled returns (uint256) {
        uint256 _amountOut = getAmountOut(_amountIn, address(currencyToken));
        currencyTokenFeeEarned += _amountIn - (_amountIn * fee) / 10_000;
        currencyToken.safeTransferFrom(msg.sender, address(this), _amountIn);
        agentToken.safeTransfer(_recipient, _amountOut);
+       require(agentToken.balanceOf(address(this)) >= agentTokenFeeEarned, "INSUFFICIENT_AGENT_TOKEN_LIQUIDITY");
        emit Swap(msg.sender, _amountIn, 0, 0, _amountOut, _recipient);
        return _amountOut;
    }
```

[Back to Top](#summaryTable)
---

### <a id="h-05"></a>[H-05]
## **DoS attack on moveLiquidity() can stop migration of bootstrap pool to Fraxswap**
#### https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/LiquidityManager.sol#L115-L120
<br>

## Description
[moveLiquidity()](https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/LiquidityManager.sol#L115-L120) relies on `LiquidityManager` contract's `currencyToken` balance to calculate `liquidityAmount` which is then transferred to Fraxswap: 
```js
    /// @dev Move the liquidity from the bootstrap pool to Fraxswap
    function moveLiquidity() external {
        require(!bootstrapPool.killed(), "BootstrapPool already killed");
        uint256 price = bootstrapPool.getPrice();
        (uint256 _reserveCurrencyToken, ) = bootstrapPool.getReserves();
        _reserveCurrencyToken = _reserveCurrencyToken - bootstrapPool.phantomAmount();
        uint256 factoryTargetCCYLiquidity = AgentFactory(owner).targetCCYLiquidity();
        require(
            _reserveCurrencyToken >= targetCCYLiquidity || _reserveCurrencyToken >= factoryTargetCCYLiquidity,
            "Bootstrap end-criterion not reached"
        );
        bootstrapPool.kill();

        // Determine liquidity amount to add
@--->   uint256 currencyAmount = currencyToken.balanceOf(address(this));
@--->   uint256 liquidityAmount = (currencyAmount * 1e18) / price;

        // Add liquidity to Fraxswap
@--->   IFraxswapPair fraxswapPair = addLiquidityToFraxswap(liquidityAmount, currencyAmount);

        // Send all remaining tokens to the agent.
        agentToken.safeTransfer(address(agent), agentToken.balanceOf(address(this)));
        currencyToken.safeTransfer(address(agent), currencyToken.balanceOf(address(this)));
        emit LiquidityMoved(agent, address(agentToken), address(fraxswapPair));

        AgentFactory(owner).setAgentStage(agent, 1);
    }
```

The `currencyToken.balanceOf(address(this))` is actually equal to what used to be the bootstrap pool's `currencyToken` reserves minus the `phantomAmount`. And the `LiquidityManager` contract currently holds all the `agentToken` reserves which was present in the bootstrap pool. Hence this attack is now possible:
- Donate `phantomAmount + 1` of currencyTokens to `LiquidityManager.sol`. This will result in `liquidityAmount` to be greater than `LiquidityManager.sol`'s agentToken balance and hence transfer to Fraxswap will revert [here](https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/LiquidityManager.sol#L141). 

## Impact
The revert of `moveLiquidity()` can only be avoided now if the bootstrap pool's price mechanics causes `price` to rise high enough such that `liquidityAmount` becomes less than the agentToken balance. This is not easy to artificially attain in a natural market driven AMM, hence this causes a significant DoS/griefing of the pool migration process. Although at first glance it seems that the attack has a cost with no apparent immediate gains, what the attacker has achieved is to force the bootstrap pool to migrate ONLY after a certain `price` has reached and not simply rely on the `targetCCYLiquidity` threshold. The impact on the protocol is high.

## Proof of Concept
Add this test inside `test/MoveLiquidityTest.sol` and run to see it pass:
```js
    function test_moveLiquidity_DoS_bug() public {
        setUpFraxtal(12_918_968);
        address whale = 0x00160baF84b3D2014837cc12e838ea399f8b8478;
        factory = new AgentFactory(currencyToken, 0);
        factory.setAgentBytecode(type(Agent).creationCode);
        factory.setGovenerBytecode(type(TokenGovernor).creationCode);
        factory.setLiquidityManagerBytecode(type(LiquidityManager).creationCode);
        factory.setTargetCCYLiquidity(0); // for testing
        factory.setInitialPrice(0.1e18);
        factory.setMintToAgent(1000); //10%
        vm.startPrank(whale);
        currencyToken.approve(address(factory), 1e18);
        agent = factory.createAgent("AIAgent", "AIA", "https://example.com", 0);
        token = agent.token();
        manager = LiquidityManager(factory.agentManager(address(agent)));
        bootstrapPool = manager.bootstrapPool();
        currencyToken.approve(address(bootstrapPool), 10_000_000e18);

        // Execute buy
        bootstrapPool.buy(1e18);

        // Case1: Successfully move the liquidity from the bootstrap pool to Fraxswap
        uint256 snap = vm.snapshot();
        manager.moveLiquidity();
        console2.log("\n moveLiquidity() successful under normal scenario");
        vm.revertTo(snap);

        // Case2: REVERT while moving the liquidity from the bootstrap pool to Fraxswap
        currencyToken.transfer(address(manager), bootstrapPool.phantomAmount() + 1);
        vm.expectRevert();
        manager.moveLiquidity();
        console2.log("\n moveLiquidity() reverted due to donation attack");
    }
```

## Mitigation 
Have the `kill()` function return the `currencyTokens` amount being transferred from bootstrap pool to `LiquidityManager.sol` and use that figure instead of using the contract balance:
```diff
    /// @dev Kill the pool
-   function kill() external nonReentrant onlyOwner {
+   function kill() external nonReentrant onlyOwner returns (uint256 currencySent) {
        _sweepFees();
        killed = true;
+       currencySent = currencyToken.balanceOf(address(this));
        agentToken.safeTransfer(owner, agentToken.balanceOf(address(this)));
-       currencyToken.safeTransfer(owner, currencyToken.balanceOf(address(this)));
+       currencyToken.safeTransfer(owner, currencySent);
    }
```

and

```diff
    /// @dev Move the liquidity from the bootstrap pool to Fraxswap
    function moveLiquidity() external {
        require(!bootstrapPool.killed(), "BootstrapPool already killed");
        uint256 price = bootstrapPool.getPrice();
        (uint256 _reserveCurrencyToken, ) = bootstrapPool.getReserves();
        _reserveCurrencyToken = _reserveCurrencyToken - bootstrapPool.phantomAmount();
        uint256 factoryTargetCCYLiquidity = AgentFactory(owner).targetCCYLiquidity();
        require(
            _reserveCurrencyToken >= targetCCYLiquidity || _reserveCurrencyToken >= factoryTargetCCYLiquidity,
            "Bootstrap end-criterion not reached"
        );
-       bootstrapPool.kill();
+       uint256 currencyAmount = bootstrapPool.kill();

        // Determine liquidity amount to add
-       uint256 currencyAmount = currencyToken.balanceOf(address(this));
        uint256 liquidityAmount = (currencyAmount * 1e18) / price;

        // Add liquidity to Fraxswap
        IFraxswapPair fraxswapPair = addLiquidityToFraxswap(liquidityAmount, currencyAmount);

        // Send all remaining tokens to the agent.
        agentToken.safeTransfer(address(agent), agentToken.balanceOf(address(this)));
        currencyToken.safeTransfer(address(agent), currencyToken.balanceOf(address(this)));
        emit LiquidityMoved(agent, address(agentToken), address(fraxswapPair));

        AgentFactory(owner).setAgentStage(agent, 1);
    }
```

[Back to Top](#summaryTable)
---

### <a id="h-06"></a>[H-06]
## **Existing Fraxswap pool can skew the price desired by the bootstrap pool**
#### https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/LiquidityManager.sol#L154-L157
<br>

## Description
During migration of the bootstrap pool to Fraxswap, if the Fraxswap pair already exists then [three rounds of swaps are done to get close to the "correct" price](https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/LiquidityManager.sol#L154-L157). The expectation by the protocol is:
```js
            // Do three rounds of swaps to get close to the correct price.
            // We need to do this because the price in the pair is might not be the same as the price in the bootstrap
            // pool, and we need to get the price in the pair close to the price in the bootstrap pool before we add
            // liquidity. We do this in three rounds, because the swap amount calculation is not precisely correct.

```
Worth noting in the above statement is:
> we need to get the price in the pair close to the price in the bootstrap pool 

However, what the logic behind this mechanics does is "smoothen or average out" the price difference instead of replicating the bootstrap pool price. The effect is not evident in tests because the Fraxswap pool is assumed to have very little reserves as compared to the bootstrap pool.

The PoC section shows how reserves of 2e18 of each token (i.e. a reserve ratio not in line with that of bootstrap pool's ratio) can impact the expected price by more than 3%.

## Impact
While this is not a profitable attack path for a malicious party, if this happens inadvertently the impact on the protocol is big:
- It gives the users a skewed favourable rate to execute the initial trades.
- Since a portion of the Fraxswap pool's LP tokens are sent to BAMM for lending activities, the amount of BAMM LP tokens minted are skewed too. Also the pair tokens backing the BAMM pool are now in a skewed ratio.
    - These effects could propagate through the DeFi ecosystem due to BAMM's composability. The incorrect token ratios in BAMM pools could affect other protocols/users who rely on BAMM for:
        - Borrowing calculations
        - Interest rate determinations
        - Liquidation thresholds

## Proof of Concept
Add this inside `test/MoveLiquidityTest.sol` and see it pass with the following output:
```js
    function test_moveLiquidity_priceManipulation_bug() public {
        uint256 initialBootstrapPrice = 10e18; // = 10 currencyTokens buys 1 agentToken i.e 10:1 rate
        
        // buy amount to reach targetCCYLiquidity threshold
        uint256 currencyInAmountBootstrap = 545e18;

        // manipulate Fraxswap pool to have 1:1 rate 
        uint256 initialLiquidityCurrency = 2e18; 
        uint256 initialLiquidityToken = 2e18;

        setUpFraxtal(12_918_968);
        address whale = 0x00160baF84b3D2014837cc12e838ea399f8b8478;
        factory = new AgentFactory(currencyToken, 0);
        factory.setAgentBytecode(type(Agent).creationCode);
        factory.setGovenerBytecode(type(TokenGovernor).creationCode);
        factory.setLiquidityManagerBytecode(type(LiquidityManager).creationCode);
        factory.setTargetCCYLiquidity(500e18);
        factory.setInitialPrice(initialBootstrapPrice);
        vm.startPrank(whale);
        currencyToken.approve(address(factory), 1e18);
        agent = factory.createAgent("AIAgent", "AIA", "https://example.com", 0);
        token = agent.token();

        // Buy from the bootstrap pool
        manager = LiquidityManager(factory.agentManager(address(agent)));
        bootstrapPool = manager.bootstrapPool();
        currencyToken.approve(address(bootstrapPool), 100_000e18);
        uint256 agentTokensBought = bootstrapPool.buy(currencyInAmountBootstrap);

        // Log bootstrap pool price before migration
        uint256 bootstrapPrice = bootstrapPool.getPrice();
        emit log_named_decimal_uint("Bootstrap Pool Price before migration:", bootstrapPrice, 18);

        // Create and fund the Fraxswap Pool
        emit log_named_decimal_uint("initialLiquidityCurrency for Frax pool:", initialLiquidityCurrency, 18);
        emit log_named_decimal_uint("initialLiquidityToken for Frax pool   :", initialLiquidityToken, 18);
        IFraxswapPair fraxswapPair = IFraxswapPair(
            manager.fraxswapFactory().createPair(address(currencyToken), address(token), 100)
        );
        currencyToken.transfer(address(fraxswapPair), initialLiquidityCurrency);
        token.transfer(address(fraxswapPair), initialLiquidityToken);
        fraxswapPair.mint(address(whale));
        
        (uint112 reserve0, uint112 reserve1,) = fraxswapPair.getReserves();
        uint256 initPrice;
        if(fraxswapPair.token0() == address(currencyToken)) {
            initPrice = (uint256(reserve0) * 1e18) / uint256(reserve1);
        } else {
            initPrice = (uint256(reserve1) * 1e18) / uint256(reserve0);
        }

        // Log Fraxswap pool price before migration
        emit log_named_decimal_uint("Init Fraxswap Price before migration :", initPrice, 18);

        {
            // Move liquidity
            uint256 expectedCcyTkn = currencyToken.balanceOf(address(factory)) + bootstrapPool.currencyTokenFeeEarned();
            uint256 expectedAgentTkn = token.balanceOf(address(factory)) + bootstrapPool.agentTokenFeeEarned();
            manager.moveLiquidity();
            require(currencyToken.balanceOf(address(factory)) == expectedCcyTkn, "incorrect fees earned");
            require(token.balanceOf(address(factory)) == expectedAgentTkn, "incorrect fees earned");
        }

        uint256 finalPrice;
        {
            // Log Fraxswap pool price after migration
            fraxswapPair = IFraxswapPair(
                manager.fraxswapFactory().getPair(address(currencyToken), address(token))
            );
            (reserve0, reserve1,) = fraxswapPair.getReserves();
            if(fraxswapPair.token0() == address(currencyToken)) {
                finalPrice = (uint256(reserve0) * 1e18) / uint256(reserve1);
            } else {
                finalPrice = (uint256(reserve1) * 1e18) / uint256(reserve0);
            }
            emit log_named_decimal_uint("Final Fraxswap Price after migration :", finalPrice, 18);
            emit log_named_decimal_uint("percentChangeInPrice :", ((bootstrapPrice - finalPrice) * 1e18) / bootstrapPrice, 16);
        }
        vm.stopPrank();
        assertLt(finalPrice, 97 * bootstrapPrice / 100, "rate not reduced by more than 3%");
    }
```
<br>

Output:
```js
[PASS] test_moveLiquidity_priceManipulation_bug() (gas: 49485095)
Logs:
  Bootstrap Pool Price before migration:: 10.000010791002911142  <----- fair rate
  initialLiquidityCurrency for Frax pool:: 2.000000000000000000
  initialLiquidityToken for Frax pool   :: 2.000000000000000000
  Init Fraxswap Price before migration :: 1.000000000000000000
  Final Fraxswap Price after migration :: 9.678323000607636815   <----- the rate reduced
  percentChangeInPrice :: 3.2168744326226065                     <----- by 3.21%
```

## Recommendation
One of the ways to approach the fix would be to check that the Fraxswap pool has at least a minimum amount of reserves before trusting it's rate. This figure could be a percentage of the bootstrap pool's reserves, like 20%. This makes the Fraxswap pool's reported figures more trustworthy and stable due to market forces. 

[Back to Top](#summaryTable)

<br>

## **MEDIUM-SEVERITY BUGS**
---

### <a id="m-01"></a>[M-01]
## **Missing slippage and deadline check in BootstrapPool's buy(), sell()**
#### https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/BootstrapPool.sol#L77
#### https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/BootstrapPool.sol#L97
<br>

## Description
Throughout the protocol, BootstrapPool's functions like [buy()](https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/BootstrapPool.sol#L77) and [sell()](https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/BootstrapPool.sol#L97) are implemented without any slippage protection or deadline support. In general, as [this informative article](https://dacian.me/defi-slippage-attacks#heading-minting-exposes-users-to-unlimited-slippage) quite nicely outlines, situations or functions where transfer of tokens may be taking place may be masked under different names, and hence should always incorporate slippage protection & deadline params.

## Impact
Absence of a deadline means that if the transaction does not get executed immediately, which is quite common, or other large transactions inadvertently front-run the current transaction, then a rate fluctuation can result in the caller either paying more in term of assets or receiving lesser shares than anticipated. 

## Recommendation
Allow the caller to specify `minOut` or `maxIn` alongwith a `deadline`.

[Back to Top](#summaryTable)
---

### <a id="m-02"></a>[M-02]
## **Missing deadline check in AgentRouter's buy(), sell()**
#### https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/AgentRouter.sol#L52
#### https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/AgentRouter.sol#L94
<br>

## Description
AgentRouter's functions [buy()](https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/AgentRouter.sol#L52) and [sell()](https://github.com/code-423n4/2025-01-iq-ai/blob/main/src/AgentRouter.sol#L94) are implemented without any deadline support. 

## Impact
Without a deadline parameter, the transaction may sit in the mempool and be executed at a much later time. The user may not be ready to go ahead with the tx params they had set long back due to different market conditions now. The slippage params provided them may not be fit anymore but they do not have any way out now.

## Recommendation
Allow the caller to specify a `deadline`.

[Back to Top](#summaryTable)
