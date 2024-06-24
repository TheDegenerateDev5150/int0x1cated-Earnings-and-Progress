# Leaderboard
[BakerFi Results](https://code4rena.com/audits/2024-05-bakerfi-invitational#top)<br>

`Rank 3 / 5`

# Audited Code Repo
### [Code4rena: BakerFi (Invitational)](https://github.com/code-423n4/2024-05-bakerfi)

<br>

# <a id="summaryTable"></a>Bugs Filed & Their Status

| # | Bug ID | Name | URL | Adjudged Status |
|--------|--------|------|:------:|-----------------:|
| 1 | [M-01](#m-01)  | User can withdraw in multiple calls with small amount to escape fee | [4](https://github.com/code-423n4/2024-05-bakerfi-findings/issues/4) | QA |
| 2 | [M-02](#m-02)  | `rebalance()` calculates `sharesToMint` by rounding-down against the protocol's favour | [5](https://github.com/code-423n4/2024-05-bakerfi-findings/issues/5) | QA |
| 3 | [M-03](#m-03)  | `deltaCollateralInETH` needs to be rounded up inside `calcDeltaPosition()` | [7](https://github.com/code-423n4/2024-05-bakerfi-findings/issues/7) | Rejected |
| 4 | [M-04](#m-04)  | `GovernableOwnable.sol` would have no `_governer` role after an individual upgrade | [10](https://github.com/code-423n4/2024-05-bakerfi-findings/issues/10) | Rejected |
| 5 | [M-05](#m-05)  | Rounding-down of `flashFee` can result in calls to flash loan to revert | [11](https://github.com/code-423n4/2024-05-bakerfi-findings/issues/11) | Accepted |
| 6 | [M-06](#m-06)  | Missing deadline in `UseSwapper::_swap()` | [12](https://github.com/code-423n4/2024-05-bakerfi-findings/issues/12) | Rejected |
| 7 | [M-07](#m-07)  | Unused user funds not refunded after `exactInputSingle()` swap | [13](https://github.com/code-423n4/2024-05-bakerfi-findings/issues/13) | Rejected |
| 8 | [M-08](#m-08)  | Missing checks for whether the L2 Sequencer is active | [14](https://github.com/code-423n4/2024-05-bakerfi-findings/issues/14) | Rejected |
| 9 | [M-09](#m-09)  | Unhandled chainlink revert can lock price oracle access | [15](https://github.com/code-423n4/2024-05-bakerfi-findings/issues/15) | QA |
|10 | [M-10](#m-10)  | min and maxAnswer never checked for oracle price feed | [16](https://github.com/code-423n4/2024-05-bakerfi-findings/issues/16) | Accepted |
|11 | [M-11](#m-11)  | Missing slippage in the call to `exactInputSingle()` inside `UseSwapper::_swap()` | [17](https://github.com/code-423n4/2024-05-bakerfi-findings/issues/17) | Accepted as High |

<br>

## **MEDIUM-SEVERITY BUGS**
---
 
### <a id="m-01"></a>[M-01]
## **User can withdraw in multiple calls with small amount to escape fee**
#### https://github.com/code-423n4/2024-05-bakerfi/blob/main/contracts/core/Vault.sol#L260-L264
<br>

## Description
The [Vault::withdraw()](https://github.com/code-423n4/2024-05-bakerfi/blob/main/contracts/core/Vault.sol#L260-L264) function rounds-down the fee against the protocol's favour and hence a user can split their withdraw tx into multiple small ones such that `fee` evaluates to zero in each call. On less expensive chains like Arbitrum or Optimism, this strategy would be beneficial for them.
```js
        // Withdraw ETh to Receiver and pay withdrawal Fees
        if (settings().getWithdrawalFee() != 0 && settings().getFeeReceiver() != address(0)) {
@---->      fee = (amount * settings().getWithdrawalFee()) / PERCENTAGE_PRECISION;
            payable(msg.sender).sendValue(amount - fee);
            payable(settings().getFeeReceiver()).sendValue(fee);
```

## Proof of Concept
- Assume `settings().getWithdrawalFee()` to be `1e4`.
- `PERCENTAGE_PRECISION` is defined by the protocol as `1e9`.
- Scenario1 (normal user):
    - `amount` = 1e5
    - Will have to pay a fee of `(1e5 * 1e4) / 1e9 = 1`
- Scenario2 (malicious user):
    - Using 2 txs of `0.5e5` each
    - In each tx `amount` = 0.5e5
    - In each tx will have to pay a fee of `(0.5e5 * 1e4) / 1e9 = 0`
Hence no fee paid by the malicious user.

## Impact
Loss of fee for the protocol

## Tools Used
Manual review

## Recommended Mitigation Steps
Round up in favour of the protocol. A library like solmate can be used which has `mulDivUp`:
```diff
-       fee = (amount * settings().getWithdrawalFee()) / PERCENTAGE_PRECISION;
+       fee = amount.mulDivUp(settings().getWithdrawalFee(), PERCENTAGE_PRECISION);
```

[Back to Top](#summaryTable)
 
---
 
### <a id="m-02"></a>[M-02]
## **`rebalance()` calculates `sharesToMint` by rounding-down against the protocol's favour**
#### https://github.com/code-423n4/2024-05-bakerfi/blob/main/contracts/core/Vault.sol#L153-L158
<br>

## Description
The [Vault::rebalance()](https://github.com/code-423n4/2024-05-bakerfi/blob/main/contracts/core/Vault.sol#L260-L264) function rounds-down the `sharesToMint` against the protocol's favour. It ought to be rounded-up to avoid loss of funds for the protocol.
```js
                    uint256 feeInEthScaled = uint256(balanceChange) *
                        settings().getPerformanceFee();
                    uint256 sharesToMint = (feeInEthScaled * totalSupply()) /
                        _totalAssets(maxPriceAge) /
                        PERCENTAGE_PRECISION;
                    _mint(settings().getFeeReceiver(), sharesToMint);
```

## Impact
Loss of funds for the protocol.

## Tools Used
Manual review

## Recommended Mitigation Steps
Round up in favour of the protocol. A library like solmate can be used which has `mulDivUp`:
```diff
-                   uint256 sharesToMint = (feeInEthScaled * totalSupply()) /
-                       _totalAssets(maxPriceAge) /
-                       PERCENTAGE_PRECISION;
+                   uint256 sharesToMint = feeInEthScaled.mulDivUp(totalSupply(), _totalAssets(maxPriceAge) * PERCENTAGE_PRECISION);
```

[Back to Top](#summaryTable)
 
---
 
### <a id="m-03"></a>[M-03]
## **`deltaCollateralInETH` needs to be rounded up inside `calcDeltaPosition()`**
#### https://github.com/code-423n4/2024-05-bakerfi/blob/main/contracts/core/hooks/UseLeverage.sol#L58
<br>

## Description
The [calcDeltaPosition() function](https://github.com/code-423n4/2024-05-bakerfi/blob/main/contracts/core/hooks/UseLeverage.sol#L58) rounds-down `deltaDebtInETH` as well as `deltaCollateralInETH`. These values are [later deducted from the total debt & collateral](https://github.com/code-423n4/2024-05-bakerfi/blob/main/contracts/core/strategies/StrategyLeverage.sol#L495-L512) respectively. This effectively means that both the remaining debt & the remaining collateral backing this debt get rounded-up. This is incorrect. While it's alright to round-up the debt as it is in the favour of the protocol, the remaining collateral backing the debt should always be rounded-down. Hence, `deltaCollateralInETH` should have been rounded-up.
```js
    function calcDeltaPosition(
        uint256 percentageToBurn,
        uint256 totalCollateralBaseInEth,
        uint256 totalDebtBaseInEth
    ) public pure returns (uint256 deltaCollateralInETH, uint256 deltaDebtInETH) {
        if (percentageToBurn == 0 || percentageToBurn > PERCENTAGE_PRECISION) {
            revert InvalidPercentageValue();
        }
        // Reduce Collateral based on the percentage to Burn
        deltaDebtInETH = (totalDebtBaseInEth * percentageToBurn) / PERCENTAGE_PRECISION; 
        // Reduce Debt based on the percentage to Burn
@--->   deltaCollateralInETH = (totalCollateralBaseInEth * percentageToBurn) / PERCENTAGE_PRECISION;  // @audit : Should have been rounded-up.
    }
```

## Impact
The collateral ratio can be skewed and can dip below acceptable ratio. 

## Tools Used
Manual review

## Recommended Mitigation Steps
Round up `deltaCollateralInETH`. A library like solmate can be used which has `mulDivUp`:
```diff
-   deltaCollateralInETH = (totalCollateralBaseInEth * percentageToBurn) / PERCENTAGE_PRECISION;
+   deltaCollateralInETH = totalCollateralBaseInEth.mulDivUp(percentageToBurn, PERCENTAGE_PRECISION);
```

[Back to Top](#summaryTable)
 
---
 
### <a id="m-04"></a>[M-04]
## **`GovernableOwnable.sol` would have no `_governer` role after an individual upgrade**
#### https://github.com/code-423n4/2024-05-bakerfi/blob/main/contracts/core/GovernableOwnable.sol#L27-L30
<br>

## Description
The [_initializeGovernableOwnable() function](https://github.com/code-423n4/2024-05-bakerfi/blob/main/contracts/core/GovernableOwnable.sol#L27-L30) of the `GovernableOwnable` contract is an internal initializer which at deployment time gets called from the `StrategyAAVEv3.sol` contract. It seems however that [even though `GovernableOwnable.sol` is upgradeable](https://github.com/code-423n4/2024-05-bakerfi/blob/main/contracts/core/GovernableOwnable.sol#L18), it can't be safely individually upgraded without causing loss of `_governer` role and one has to always depend on `StrategyAAVEv3.sol` for an upgrade. 
```js
@--->   contract GovernableOwnable is OwnableUpgradeable {
            
            address private _governor;
            
            error CallerNotTheGovernor();
            error InvalidGovernorAddress();

            event GovernshipTransferred(address indexed previousGovernor, address indexed newGovernor);

@--->       function _initializeGovernableOwnable(address initialOwner, address initialGovernor) internal initializer {
                _transferOwnership(initialOwner);
                _transferGovernorship(initialGovernor); 
            }
```

and

```js
@--->   function transferGovernorship(address _newGovernor) public virtual onlyGovernor {
            if(_newGovernor == address(0)) revert InvalidGovernorAddress();
            _transferGovernorship(_newGovernor);
        }
```

`transferGovernorship()` too can only be called by `onlyGovernor` rendering it unusable after an individual upgrade.

## Impact
Governor role lost after individual upgrade.

## Tools Used
Manual review

## Recommended Mitigation Steps
Add a function `setGoverner()` protected by the `onlyOwner` modifier which can be called after an individual upgrade of this contract.

[Back to Top](#summaryTable)

---
 
### <a id="m-05"></a>[M-05]
## **Rounding-down of `flashFee` can result in calls to flash loan to revert**
#### https://github.com/code-423n4/2024-05-bakerfi/blob/main/contracts/core/flashloan/BalancerFlashLender.sol#L63
#### https://github.com/code-423n4/2024-05-bakerfi/blob/main/contracts/core/strategies/StrategyLeverage.sol#L247-L249
<br>

## Description
The [BalancerFlashLender::flashFee()](https://github.com/code-423n4/2024-05-bakerfi/blob/main/contracts/core/flashloan/BalancerFlashLender.sol#L63) function returns a rounded-down fee. This `fee` is then [later used](https://github.com/code-423n4/2024-05-bakerfi/blob/main/contracts/core/strategies/StrategyLeverage.sol#L247-L249) across the protocol while providing spend approval to the flash loan provider. This can result in an approval less than that expected by the provider and hence cause the call to flash loan to revert. This is because [flash loan providers calculate their fee by rounding up in their favour](https://docs.uniswap.org/contracts/v2/guides/smart-contract-integration/using-flash-swaps#single-token), instead of rounding down:
```js
File: contracts/core/flashloan/BalancerFlashLender.sol

    function flashFee(address, uint256 amount) external view override returns (uint256) {
        uint256 perc = _balancerVault.getProtocolFeesCollector().getFlashLoanFeePercentage();
        if (perc == 0 || amount == 0) {
            return 0;
        }

@--->   return (amount * perc) / _BALANCER_MAX_FEE_PERCENTAGE;
    }
```

and 
```js
File: contracts/core/strategies/StrategyLeverage.sol

    function deploy() external payable onlyOwner nonReentrant returns (uint256 deployedAmount) {
        if (msg.value == 0) revert InvalidDeployAmount();
        // 1. Wrap Ethereum
        address(wETHA()).functionCallWithValue(abi.encodeWithSignature("deposit()"), msg.value);
        // 2. Initiate a WETH Flash Loan
        uint256 leverage = calculateLeverageRatio(
            msg.value,
            getLoanToValue(),
            getNrLoops()
        );
        uint256 loanAmount = leverage - msg.value;
@--->   uint256 fee = flashLender().flashFee(wETHA(), loanAmount);
        //§uint256 allowance = wETH().allowance(address(this), flashLenderA());
@--->   if(!wETH().approve(flashLenderA(), loanAmount + fee)) revert FailedToApproveAllowance();
        if (
            !flashLender().flashLoan(
                IERC3156FlashBorrowerUpgradeable(this),
                wETHA(),
                loanAmount,
                abi.encode(msg.value, msg.sender, FlashLoanAction.SUPPLY_BOORROW)
            )
        ) {
            revert FailedToRunFlashLoan();
        }

        deployedAmount = _pendingAmount;
        _deployedAmount = _deployedAmount + deployedAmount;
        emit StrategyAmountUpdate(_deployedAmount);
        // Pending amount is not cleared to save gas
        // _pendingAmount = 0;
    }
```

## Impact
Flash loan call reverts for many amount and fee percentage combinations.

## Tools Used
Manual review

## Recommended Mitigation Steps
Round up in favour of the protocol. A library like solmate can be used which has `mulDivUp`:
```diff
    function flashFee(address, uint256 amount) external view override returns (uint256) {
        uint256 perc = _balancerVault.getProtocolFeesCollector().getFlashLoanFeePercentage();
        if (perc == 0 || amount == 0) {
            return 0;
        }

-       return (amount * perc) / _BALANCER_MAX_FEE_PERCENTAGE;
+       return amount.mulDivUp(perc, _BALANCER_MAX_FEE_PERCENTAGE);
    }
```

[Back to Top](#summaryTable)
 
---
 
### <a id="m-06"></a>[M-06]
## **Missing deadline in `UseSwapper::_swap()`**
#### https://github.com/code-423n4/2024-05-bakerfi/blob/main/contracts/core/hooks/UseSwapper.sol#L57
<br>

## Description
The [_swap()](https://github.com/code-423n4/2024-05-bakerfi/blob/main/contracts/core/hooks/UseSwapper.sol#L57) function does not provide a `deadline` param either for `ExactInputSingleParams` or for `ExactOutputSingleParams`. Thus the swap could go through way later than the user was ready to accept, when the market conditions may have changed considerably. In the current implementation, there is no way for the user to specify it.

## Tools Used
Manual review

## Recommended Mitigation Steps
Add a param `deadline: params.deadline` inside both `ExactInputSingleParams` and `ExactOutputSingleParams` calls.

[Back to Top](#summaryTable)
 
---

### <a id="m-07"></a>[M-07]
## **Unused user funds not refunded after `exactInputSingle()` swap**
#### https://github.com/code-423n4/2024-05-bakerfi/blob/main/contracts/core/hooks/UseSwapper.sol#L67
<br>

## Description
The [exactInputSingle() call](https://github.com/code-423n4/2024-05-bakerfi/blob/main/contracts/core/hooks/UseSwapper.sol#L67) assumes that all of the user funds or `params.amountIn` will be consumed. This is not necessarily true. A swap can be interrupted earlier than expected when there's not enough liquidity in a pool. Hence it's recommended that the protocol adds the logic to handle any unspent funds after `exactInputSingle()`.

## Proof of Concept
- See a similar past issue: https://solodit.xyz/issues/swaprouter-doesnt-refund-unspent-eth-after-swapping-spearbit-none-velodrome-finance-pdf

Please refer the second point "_A swap can be interrupted earlier when there's not enough liquidity in a pool._".  Also refer its recommendation section which suggests returning unspent ETH at the end of all swap styles, **even for** `SwapRouter.exactInputSingle()`.
<br>

Also refer the exact fix made by the protocol inside `exactInputSingle()` [here](https://github.com/velodrome-finance/slipstream/commit/0c5da40e5258d3c8de7d03c0a85b89209111365a#diff-b9120db8cf76d7ed7f03380f9e411df80c8bc8b9cad19ff84339046ac419b515R121) (_click "Expand all" there to see the full code and the function name_).

## Tools Used
Manual review

## Recommended Mitigation Steps
Just like the protocol has implemented a transfer after `exactOutputSingle` [here](https://github.com/code-423n4/2024-05-bakerfi/blob/main/contracts/core/hooks/UseSwapper.sol#L96), add the logic to transfer any unspent funds after `exactInputSingle()` too.

[Back to Top](#summaryTable)
 
---

### <a id="m-08"></a>[M-08]
## **Missing checks for whether the L2 Sequencer is active**
#### https://github.com/code-423n4/2024-05-bakerfi/blob/main/contracts/oracles/EthOracle.sol#L31
<br>

## Description
Chainlink recommends that users using price oracles, check whether the Arbitrum Sequencer is [active](https://docs.chain.link/data-feeds/l2-sequencer-feeds#arbitrum). If the sequencer goes down, the Chainlink oracles will have stale prices from before the downtime, until a new L2 OCR transaction goes through. Users who submit their transactions via the [L1 Delayed Inbox](https://developer.arbitrum.io/tx-lifecycle#1b--or-from-l1-via-the-delayed-inbox) will be able to take advantage of these stale prices. Use the [Chainlink oracle recommended checks](https://blog.chain.link/how-to-use-chainlink-price-feeds-on-arbitrum/#almost_done!_meet_the_l2_sequencer_health_flag:~:text=as%20%E2%80%9C_priceFeedAddress%E2%80%9D%20parameter.-,Almost%20Done!%20Meet%20the%20L2%20Sequencer%20Health%20Flag,-Transactions%20in%20Arbitrum) to determine whether the sequencer is offline or not, and don't allow operations to take place while the sequencer is offline.

[Link to code](https://github.com/code-423n4/2024-05-bakerfi/blob/main/contracts/oracles/EthOracle.sol#L31):
```js
    function getLatestPrice() public view override returns (IOracle.Price memory price) {
@-->    (, int256 answer, uint256 startedAt, uint256 updatedAt,) = _ethPriceFeed.latestRoundData();
        if ( answer<= 0 ) revert InvalidPriceFromOracle();        
        if ( startedAt ==0 || updatedAt == 0 ) revert InvalidPriceUpdatedAt();    

        price.price = uint256(answer);
        price.lastUpdate = updatedAt;
    }
```

## Tools Used
Manual review

## Recommended Mitigation Steps
Use the [Chainlink oracle recommended checks](https://blog.chain.link/how-to-use-chainlink-price-feeds-on-arbitrum/#almost_done!_meet_the_l2_sequencer_health_flag:~:text=as%20%E2%80%9C_priceFeedAddress%E2%80%9D%20parameter.-,Almost%20Done!%20Meet%20the%20L2%20Sequencer%20Health%20Flag,-Transactions%20in%20Arbitrum) to determine whether the sequencer is offline or not, and don't allow operations to take place while the sequencer is offline.

[Back to Top](#summaryTable)
 
---

### <a id="m-09"></a>[M-09]
## **Unhandled chainlink revert can lock price oracle access**
#### https://github.com/code-423n4/2024-05-bakerfi/blob/main/contracts/oracles/EthOracle.sol#L31
<br>

## Description
Chainlink's multisigs can immediately block access to price feeds at will. Therefore, to prevent denial of service scenarios, it is recommended to query Chainlink price feeds using a defensive approach with Solidity’s try/catch structure. In this way, if the call to the price feed fails, the caller contract is still in control and can handle any errors safely and explicitly.

Refer to https://blog.openzeppelin.com/secure-smart-contract-guidelines-the-dangers-of-price-oracles/ for more information regarding potential risks to account for when relying on external price feed providers.

[Link to code](https://github.com/code-423n4/2024-05-bakerfi/blob/main/contracts/oracles/EthOracle.sol#L31):
```js
    function getLatestPrice() public view override returns (IOracle.Price memory price) {
@-->    (, int256 answer, uint256 startedAt, uint256 updatedAt,) = _ethPriceFeed.latestRoundData();
        if ( answer<= 0 ) revert InvalidPriceFromOracle();        
        if ( startedAt ==0 || updatedAt == 0 ) revert InvalidPriceUpdatedAt();    

        price.price = uint256(answer);
        price.lastUpdate = updatedAt;
    }
```

## Similar past issues
- https://github.com/code-423n4/2022-07-juicebox-findings/issues/59
- https://github.com/sherlock-audit/2023-02-blueberry-judging/issues/161

## Tools Used
Manual review

## Recommended Mitigation Steps
Surround the call to latestRoundData() with try/catch instead of calling it directly and provide a graceful alternative/exit.

[Back to Top](#summaryTable)
 
---

### <a id="m-10"></a>[M-10]
## **min and maxAnswer never checked for oracle price feed**
#### https://github.com/code-423n4/2024-05-bakerfi/blob/main/contracts/oracles/EthOracle.sol#L31
<br>

## Description
Chainlink aggregators have a built in circuit breaker if the price of an asset goes outside of a predetermined price band. The result is that if an asset experiences a huge drop in value (i.e. LUNA crash) the price of the oracle will continue to return the minPrice instead of the actual price of the asset. This would allow user to continue borrowing with the asset but at the wrong price. This is exactly what happened to [Venus on BSC when LUNA imploded](https://rekt.news/venus-blizz-rekt/). However, the protocol misses to implement such a check. 

[Link to code](https://github.com/code-423n4/2024-05-bakerfi/blob/main/contracts/oracles/EthOracle.sol#L31):
```js
    function getLatestPrice() public view override returns (IOracle.Price memory price) {
@-->    (, int256 answer, uint256 startedAt, uint256 updatedAt,) = _ethPriceFeed.latestRoundData();
        if ( answer<= 0 ) revert InvalidPriceFromOracle();        
        if ( startedAt ==0 || updatedAt == 0 ) revert InvalidPriceUpdatedAt();    

        price.price = uint256(answer);
        price.lastUpdate = updatedAt;
    }
```

## Similar past issues
- https://github.com/sherlock-audit/2023-05-USSD-judging/issues/598
- https://github.com/sherlock-audit/2023-02-blueberry-judging/issues/18

## Tools Used
Manual review

## Recommended Mitigation Steps
Add logic along the lines of:
```js
    require(answer >= minPrice && answer <= maxPrice, "invalid price");
```
min and max prices can be gathered using [one of these ways](https://medium.com/cyfrin/chainlink-oracle-defi-attacks-93b6cb6541bf#99af:~:text=Developers%20%26%20Auditors%20can%20find%20Chainlink%E2%80%99s%20oracle%20feed).

[Back to Top](#summaryTable)
 
---
 
### <a id="m-11"></a>[M-11]
## **Missing slippage in the call to `exactInputSingle()` inside `UseSwapper::_swap()`**
#### https://github.com/code-423n4/2024-05-bakerfi/blob/main/contracts/core/hooks/UseSwapper.sol#L72
<br>

## Description
The [exactInputSingle() inside _swap()](https://github.com/code-423n4/2024-05-bakerfi/blob/main/contracts/core/hooks/UseSwapper.sol#L72) function uses a `amountOutMinimum` param with a value of zero. Thus it offers no slippage protection to the caller and can result in sandwich attacks or even under the normal flow of events a bad price for which the caller was not ready.

## Tools Used
Manual review

## Recommended Mitigation Steps
Set it to a user provided param by specifying `amountOutMinimum: params.amountOutMinimum` inside `ExactInputSingleParams`.

[Back to Top](#summaryTable)
 
---