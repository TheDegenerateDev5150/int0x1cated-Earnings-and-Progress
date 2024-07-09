# Leaderboard
[Hodl Results](https://code4rena.com/audits/2024-05-hodl-invitational)<br>

`Rank 3 / 5`

# Audited Code Repo
### [Code4rena: Hodl (Invitational)](https://github.com/code-423n4/2024-05-hodl)

<br>

# <a id="summaryTable"></a>Bugs Filed & Their Status

| # | Bug ID | Name | URL | Adjudged Status | Count of dups (excluding current submission) |
|--------|--------|------|:------:|-----------------:|:-----------------------------------:|
| 1 | [H-01](#h-01)  | Can't redeem plETH later on if price goes above strike & comes down again with no redemption occuring in that time window | [17](https://github.com/code-423n4/2024-05-hodl-findings/issues/17) | Accepted as High; Selected Report | 1 |
| 2 | [H-02](#h-02)  | `YMultiToken::mint()` and `HodlMultiToken::safeTransferFrom()` use `_update()` instead of `_updateWithAcceptanceCheck()` which causes tokens to be lost & locked forever when the receiver is a contract address with no `IERC1155-onERC1155Received()` implemented | [19](https://github.com/code-423n4/2024-05-hodl-findings/issues/19) | QA | - |
| 3 | [M-01](#m-01)  | deadline has no effect when directly set to `block.timestamp + 1` | [2](https://github.com/code-423n4/2024-05-hodl-findings/issues/2) | Accepted as Medium | 2 |
| 4 | [M-02](#m-02)  | Lack of slippage protection inside `addLiquidity()` | [23](https://github.com/code-423n4/2024-05-hodl-findings/issues/23) | Rejected | - |
| 5 | [M-03](#m-03)  | `hodlBuy()` doesn't refund unspent ETH after swap | [37](https://github.com/code-423n4/2024-05-hodl-findings/issues/37) | Accepted as Medium | 2 |


<br>

## **HIGH-SEVERITY BUGS**
---

### <a id="h-01"></a>[H-01]
## **Can't redeem plETH later on if price goes above strike & comes down again with no redemption occuring in that time window**
#### https://github.com/code-423n4/2024-05-hodl/blob/main/src/Vault.sol#L350-L353
<br>

## Description
The [natspec says](https://github.com/code-423n4/2024-05-hodl/blob/main/src/Vault.sol#L350-L353) that _"The redemption can happen even if the price later dips below"_:
```js
    // redeem converts a stake into the underlying tokens if the price has
    // touched the strike. The redemption can happen even if the price later
    // dips below.
    function redeem(uint32 stakeId, uint256 amount) external nonReentrant {
```

Similar comment is present on the [audit page](https://github.com/code-423n4/2024-05-hodl?tab=readme-ov-file):
> plETH tokens can be staked. Staking associates them with a timestamp, and those staked tokens can be redeemed if the price ever touches/exceeds their strike.

**_"ever touches"_** is to be noted.
<br>

This is however not followed inside code for `plETH`. It's the same case with `ybETH` where the yield should stop accumulating the moment strike price is hit, but it continues to accrue in the below explained "price swing" scenario.

## Impact
Consider the scenario:
- Alice stakes 2 ether of `plETH` and `ybETH` at strike of `100`. Current price is `95`.
- Price goes up to `101` and within 5 seconds comes down to `99`.
- Due to the sudden price movement, neither Alice nor any other user who may have staked at the same strike is able to call `redeem` within those 5 seconds.
- Alice and other users do not worry since the protocol clearly says they can redeem at a later point of time too, as the strike price was breached once.
- Alice now tries to call `redeem`. It reverts because `_closeOutEpoch()` [never got called from inside redeem](https://github.com/code-423n4/2024-05-hodl/blob/main/src/Vault.sol#L364) ever and hence [the call to canRedeem()](https://github.com/code-423n4/2024-05-hodl/blob/main/src/Vault.sol#L358) returns `false` later on.

Alice and the other users lose their chance to book profits in spite of being directionally right about the price movement and trusting the claims of the protocol.
<br>

It's **important to note** that the issue is not only constrained to a case of "quick price swing". It's possible that Alice had staked on a quite high strike price, a long shot, on which no one else had. Since she is the only one capable of calling redeem and hence closing the epoch, if she misses to do so while the price was above the strike then she loses out on the opportunity completely.

## Proof of Concept
Add this test inside `test/Vault.t.sol` and run via `forge test --mt test_t0x1c_PriceMovement -vv` to see it revert with reason _"revert: cannot redeem"_:
```js
    function test_t0x1c_PriceMovement() public {
        initVault();

        vm.startPrank(alice);

        // Mint hodl tokens
        vault.mint{value: 2 ether}(strike1);

        // Stake before strike hits
        uint32 stake1 = vault.hodlStake(strike1, 2 ether - 1, alice);

        // Strike hits
        oracle.setPrice(strike1 + 1);

        // In few seconds, go below strike
        skip(5 seconds);
        oracle.setPrice(strike1 - 1);

        // @audit : attempt to redeem staked amount reverts!
        vault.redeem(stake1, 1 ether);

        vm.stopPrank();
    }
```

## Tools Used
Foundry

## Recommended Mitigation Steps
Unless there is a cron job or monitoring system which tracks the strikes staked on, and triggers a dummy redeem of 0 amount the moment it is hit (or calls all the necessary internal functions like `_closeOutEpoch()`), this issue seems to be difficult to get rid of.

[Back to Top](#summaryTable)
 
---
 
### <a id="h-02"></a>[H-02]
## **`YMultiToken::mint()` and `HodlMultiToken::safeTransferFrom()` use `_update()` instead of `_updateWithAcceptanceCheck()` which causes tokens to be lost & locked forever when the receiver is a contract address with no `IERC1155-onERC1155Received()` implemented**
#### https://github.com/code-423n4/2024-05-hodl/blob/main/src/multi/HodlMultiToken.sol#L63
#### https://github.com/code-423n4/2024-05-hodl/blob/main/src/multi/YMultiToken.sol#L47
<br>

## Impact
ERC1155's implementation of `safeTransferFrom()` and also `mint()`, `burn()` etc. ensures a revert when the receiver is a contract but has not registered itself as aware of the ERC1155 protocol. This ensures tokens sent to such receivers are not lost forever as [explained here](https://docs.openzeppelin.com/contracts/3.x/erc1155#:~:text=A%20key%20difference%20when%20using,usually%20due%20to%20user%20error.). <br>
OpenZeppelin implements this by internally calling [_updateWithAcceptanceCheck()](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/ef68ac3ed83f85e4ce078b26170280c468ffeabb/contracts/token/ERC1155/ERC1155.sol#L233) which has the [necessary checks](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/ef68ac3ed83f85e4ce078b26170280c468ffeabb/contracts/token/ERC1155/ERC1155.sol#L206). Any attempt to make such a transfer results in a revert with **"Reason: ERC1155InvalidReceiver"**. 
<br>

This has however been overridden by the protocol by a vanilla call to `_update()` 
- [here inside HodlMultiToken::safeTransferFrom()](https://github.com/code-423n4/2024-05-hodl/blob/main/src/multi/HodlMultiToken.sol#L63) 
    - which is in turn called by [HodlToken::transfer()](https://github.com/code-423n4/2024-05-hodl/blob/main/src/single/HodlToken.sol#L54) and [HodlToken::transferFrom()](https://github.com/code-423n4/2024-05-hodl/blob/main/src/single/HodlToken.sol#L84). **Note that this is the reason to raise the issue as `High` severity - since all `HodlToken` transfers involve taking this path.**
- [here inside YMultiToken::mint()](https://github.com/code-423n4/2024-05-hodl/blob/main/src/multi/YMultiToken.sol#L47). 
_Please note that it has also been overridden inside `YMultiToken::burn()` but that is not an issue as it has exactly the same behaviour as the original implementation since `to == address(0)`._

Due to this customization, when the receiver is a contract which has not implemented the `IERC1155-onERC1155Received()` function, the transferred tokens would be lost forever.

## Proof of Concept
The following test shows how a transfer to such a contract does not revert, even though a user using "safe" transfer functionality would expect it to.<br>
First, add the following lines at the very end of `test/Vault.t.sol`. This will serve as our receiver contract which has no implmentation of `onERC1155Received()` inside it:
```js
contract ReceivingContract {
    // @audit : Contract has not implemented `onERC1155Received`
    // function onERC1155Received(address, address, uint256, uint256, bytes memory) public virtual returns (bytes4) {
    //     return this.onERC1155Received.selector;
    // }
}
```

Add this test inside `test/Vault.t.sol` now and run via `forge test --mt test_t0x1c_ERC1155Check -vv` to see it pass instead of reverting:
```js
    function test_t0x1c_ERC1155Check() public {
        initVault();

        ReceivingContract receiver = new ReceivingContract();
        address _receiver = address(receiver);

        vm.startPrank(alice);
        vault.mint{value: 1 ether}(strike1);

        console.log("Initial Alice HODL BALANCE IS      = %s", vault.hodlMulti().balanceOf(alice, strike1));
        console.log("Initial Receiver HODL BALANCE IS   = %s\n", vault.hodlMulti().balanceOf(_receiver, strike1));

        vault.hodlMulti().safeTransferFrom(alice, _receiver, strike1, 1 ether - 1, "");

        console.log("Updated Alice HODL BALANCE IS      = %s", vault.hodlMulti().balanceOf(alice, strike1));
        console.log("Updated Receiver HODL BALANCE IS   = %s", vault.hodlMulti().balanceOf(_receiver, strike1));

        vm.stopPrank();
    }
```

## Tools Used
Foundry

## Recommended Mitigation Steps
Make the following change inside `src/multi/HodlMultiToken.sol`:
```diff
    function safeTransferFrom(address from,
                              address to,
                              uint256 strike,
                              uint256 amount,
                              bytes memory) public override {
        if (from != msg.sender &&
            !isApprovedForAll(from, msg.sender) &&
            !authorized[msg.sender]) {

            revert ERC1155MissingApprovalForAll(msg.sender, from);
        }

        uint256[] memory strikes = new uint256[](1);
        uint256[] memory amounts = new uint256[](1);
        strikes[0] = strike;
        amounts[0] = amount;

-       _update(from, to, strikes, amounts);
+       _updateWithAcceptanceCheck(from, to, strikes, amounts, "");
    }
```

and inside `src/multi/YMultiToken.sol`, the local `_update()` function needs to include the following logic (picked up from OZ official implementation, please change the variable names as needed):
```js
        ....
        ....
        // @audit : add this logic ..
        if (to != address(0)) {
            address operator = _msgSender();
            if (ids.length == 1) {
                uint256 id = ids.unsafeMemoryAccess(0);
                uint256 value = values.unsafeMemoryAccess(0);
                _doSafeTransferAcceptanceCheck(operator, from, to, id, value, data);
            } else {
                _doSafeBatchTransferAcceptanceCheck(operator, from, to, ids, values, data);
            }
        }
```

[Back to Top](#summaryTable)
 
<br><br>

## **MEDIUM-SEVERITY BUGS**
---
 
### <a id="m-01"></a>[M-01]
## **deadline has no effect when directly set to `block.timestamp + 1`**
#### https://github.com/code-423n4/2024-05-hodl/blob/main/src/Router.sol#L209
<br>

## Impact
At multiple places in the code the protocol calls the `ISwapRouter.ExactInputSingleParams()` function with `deadline` as `block.timestamp + 1`. This has basically no effect and should rather be an actual `uint256` value. An example of [one such instance](https://github.com/code-423n4/2024-05-hodl/blob/main/src/Router.sol#L209):
```js
    function hodlBuy(uint64 strike, uint256 minOut) external nonReentrant payable returns (uint256) {
        IERC20 token = vault.deployments(strike);
        require(address(token) != address(0), "no deployed ERC20");
        address uniPool = pool(strike);
        require(uniPool != address(0), "no uni pool");

        weth.deposit{value: msg.value}();

        IERC20(address(weth)).forceApprove(address(address(swapRouter)), msg.value);

        ISwapRouter.ExactInputSingleParams memory params =
            ISwapRouter.ExactInputSingleParams({
                tokenIn: address(weth),
                tokenOut: address(token),
                fee: FEE,
                recipient: address(this),
@------>        deadline: block.timestamp + 1, // @audit : does not work as intended
                amountIn: msg.value,
                amountOutMinimum: minOut,
                sqrtPriceLimitX96: 0 });

        uint256 out = swapRouter.exactInputSingle(params);

        emit HodlBuy(msg.sender, strike, msg.value, out);

        return out;
    }
```

Since `block.timestamp` is always relative, using it in any way is equivalent to using no deadline at all. Needs to use a user defined input to effectively enforce any deadline.<br>
Without a deadline, the transaction might be left hanging in the mempool and be executed way later than the user wanted. That could lead to user getting a worse price, because a validator can just hold onto the transaction. And when it does get around to putting the transaction in a block, it'll be `block.timestamp + 1`, so they've got no protection there.
<br>

This is a well-known vulnerability and here's an article detailing the impact and how Uniswap has a very clear deadline set, in addition to the minimumOut check - https://web.archive.org/web/20230525014603/https://blog.bytes032.xyz/p/why-you-should-stop-using-block-timestamp-as-deadline-in-swaps

## Proof of Concept
Instances of incorrect deadline in code:
- https://github.com/code-423n4/2024-05-hodl/blob/main/src/Router.sol#L209
- https://github.com/code-423n4/2024-05-hodl/blob/main/src/Router.sol#L254
- https://github.com/code-423n4/2024-05-hodl/blob/main/src/Router.sol#L165
- https://github.com/code-423n4/2024-05-hodl/blob/main/src/Router.sol#L371
- https://github.com/code-423n4/2024-05-hodl/blob/main/src/Router.sol#L477
- https://github.com/code-423n4/2024-05-hodl/blob/main/src/Router.sol#L504

## Previously reported bug on C4
- https://github.com/code-423n4/2023-12-particle-findings/issues/59

## Tools Used
Manual review

## Recommended Mitigation Steps
The following modification would work. Similar change needs to be made at other places too:
```diff
    function hodlBuy(uint64 strike, uint256 minOut) external nonReentrant payable returns (uint256) {
        IERC20 token = vault.deployments(strike);
        require(address(token) != address(0), "no deployed ERC20");
        address uniPool = pool(strike);
        require(uniPool != address(0), "no uni pool");

        weth.deposit{value: msg.value}();

        IERC20(address(weth)).forceApprove(address(address(swapRouter)), msg.value);
+       uint256 deadline = block.timestamp + 1;
        ISwapRouter.ExactInputSingleParams memory params =
            ISwapRouter.ExactInputSingleParams({
                tokenIn: address(weth),
                tokenOut: address(token),
                fee: FEE,
                recipient: address(this),
-               deadline: block.timestamp + 1,
+               deadline: deadline,
                amountIn: msg.value,
                amountOutMinimum: minOut,
                sqrtPriceLimitX96: 0 });

        uint256 out = swapRouter.exactInputSingle(params);

        emit HodlBuy(msg.sender, strike, msg.value, out);

        return out;
    }
```

[Back to Top](#summaryTable)
 
---
 
### <a id="m-02"></a>[M-02]
## **Lack of slippage protection inside `addLiquidity()`**
#### https://github.com/code-423n4/2024-05-hodl/blob/main/src/Router.sol#L162-L163
<br>

## Impact
Inside `Router.sol`, there's a lack of slippage protection [inside addLiquidity()](https://github.com/code-423n4/2024-05-hodl/blob/main/src/Router.sol#L162-L163) on L62-63:
```js
        INonfungiblePositionManager.MintParams memory params = INonfungiblePositionManager.MintParams({
            token0: token0,
            token1: token1,
            fee: FEE,
            tickLower: tickLower,
            tickUpper: tickUpper,
            amount0Desired: token0Amount,
            amount1Desired: token1Amount,
@------>    amount0Min: 0,
@------>    amount1Min: 0,
            recipient: msg.sender,
            deadline: block.timestamp + 1 });
```

This can result in receiving a worse price than intended and can also be a target of MEV attacks, thus causing loss of funds.

## Proof of Concept
- https://github.com/code-423n4/2024-05-hodl/blob/main/src/Router.sol#L162-L163

## Tools Used
Manual review

## Recommended Mitigation Steps
Pass an expected minimum out-amount param which will serve as slippage protection for each of these functions.

[Back to Top](#summaryTable)
 
---
 
### <a id="m-03"></a>[M-03]
## **`hodlBuy()` doesn't refund unspent ETH after swap**
#### https://github.com/code-423n4/2024-05-hodl/blob/main/src/Router.sol#L193
<br>

## Impact
The [hodlBuy() function](https://github.com/code-423n4/2024-05-hodl/blob/main/src/Router.sol#L193) is a payable function which accepts ETH and goes on to swap WETH to an ERC20 token. The protocol assumes by using `swapRouter.exactInputSingle()` that all of the ETH or `msg.value` will be consumed. This is not necessarily true. A swap can be interrupted earlier than expected when there's not enough liquidity in a pool. It's recommended that the protocol refunds any remaining ETH at the end of the swap operation back to the `msg.sender` else it remains stuck in the contract.

## Proof of Concept
- See a similar past issue: https://solodit.xyz/issues/swaprouter-doesnt-refund-unspent-eth-after-swapping-spearbit-none-velodrome-finance-pdf

Please refer the second point "_A swap can be interrupted earlier when there's not enough liquidity in a pool._".  Also refer its recommendation section which suggests returning unspent ETH at the end of all swap styles, **even for** `SwapRouter.exactInputSingle()`.
<br>

Also refer the exact fix made by the protocol inside `exactInputSingle()` [here](https://github.com/velodrome-finance/slipstream/commit/0c5da40e5258d3c8de7d03c0a85b89209111365a#diff-b9120db8cf76d7ed7f03380f9e411df80c8bc8b9cad19ff84339046ac419b515R121) (_click "Expand all" there to see the full code and the function name_).

## Tools Used
Manual review

## Additional Impact
It appears [any extra WETH received after the swap operation](https://github.com/code-423n4/2024-05-hodl/blob/main/src/Router.sol#L373-L378) inside `_yBuyViaLoan()` too remains idle inside the contract. We can see `amountOutMinimum: loan + fee` which will be later reclaimed by Aave at the end of the flash loan i.e. at the end of the [executeOperation() function](https://github.com/code-423n4/2024-05-hodl/blob/main/src/Router.sol#L523-L537). Hence any WETH received over and above the `amountOutMinimum` is never moved back to the caller.

## Recommended Mitigation Steps
After the swap operation inside `hodlBuy()`, add the logic to transfer any unspent ETH back to the caller. Not only here, but other functions utilizing the swap functionality would benefit too from the addition of refund logic after their corresponding swaps. Consider adding those too.

[Back to Top](#summaryTable)
