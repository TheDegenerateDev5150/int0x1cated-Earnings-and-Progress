# Leaderboard
[Canto Contest](https://code4rena.com/contests/2023-11-canto-application-specific-dollars-and-bonding-curves-for-1155s#top) 
<br>

`Rank 98 / 120`

# Audited Code Repo
### [Code4rena: Canto](https://github.com/code-423n4/2023-11-canto/tree/335930cd53cf9a137504a57f1215be52c6d67cb3)

<br>

# Bugs Filed & Their Status

| # | Bug ID | Name | URL | Adjudged Status |
|--------|--------|------|:------:|-----------------:|
| 1 | [H-01](#h-01)    | Loss of funds due to incorrect calculation in `withdrawCarry()` and `burn()` | [105](https://github.com/code-423n4/2023-11-canto-findings/issues/105) | Invalidated |
| 2 | [M-01](#m-01)    | No slippage protection for user calling `buy()` can cause loss of funds for him | [150](https://github.com/code-423n4/2023-11-canto-findings/issues/150) | Valid as Medium |
| 3 | [M-02](#m-02)    | Loss of funds for user if burning more than one NFT | [154](https://github.com/code-423n4/2023-11-canto-findings/issues/154) | Invalidated (Out of Scope) |
| 4 | [M-03](#m-03)    | User calling `sell()` has no slippage protection which can cause loss of funds for her | [176](https://github.com/code-423n4/2023-11-canto-findings/issues/176) | Self-duplicate |
| 5 | [M-04](#m-04)    | Attacker can sandwich other user's `buy()` order to make immediate profits | [218](https://github.com/code-423n4/2023-11-canto-findings/issues/218) | Self-duplicate |

<br>

## **HIGH-SEVERITY BUGS**
---

### <a id="h-01"></a>[H-01]
## **Loss of funds due to incorrect calculation in `withdrawCarry()` and `burn()`**
#### https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L72-L90
#### https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L60-L67
<br>

## Impact
Due to incorrect `_amount` passed and usage of `CErc20Interface(cNote).redeemUnderlying(_amount)` [here](https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L85) and [here](https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L63), user can not -
1. [withdrawCarry](https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L72-L90) the full interest,
2. [burn](https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L60-L67) his complete `asD` tokens to claim back full `NOTE` tokens.

In both the cases, his funds are stuck in the protocol and not retrievable.
**This breaks the [invariant](https://github.com/code-423n4/2023-11-canto/tree/main#main-invariants):** - `asD: It should always be possible to redeem 1 asD for 1 NOTE`

## Proof of Concept
All the following tests are to be put inside `2023-11-canto/asD/src/test/asD.t.sol` and then can be run individually through `forge test --mt <test-case-name> -vv`.

There are multiple ways in which the calculation error impacts users:
1. The pre-existing test `testWithdrawCarry()` provided by the Dev team shows us how an owner should be able to withdraw `1e18` NOTE tokens when `exchangeRate` equals `1.1e28`. Let's try to do the same in 2 steps -
    - Call `withdrawCarry` with an `_amount` of `0.5e18` NOTE tokens
    - Repeat the above and call `withdrawCarry` with an `_amount` of `0.5e18` NOTE tokens agains. 
The owner should have no problems receiving `1e18` NOTE tokens if the calculations have been performed correctly. However, upon running the following test, you will get `revert: Too many tokens requested` and the owner would never be able to withdraw all his interest.

<details>
<summary>PoC-1 : click to expand</summary>

```js
    function test_t0x1c_WithdrawCarryTwice() public {
        testMint();
        uint256 newExchangeRate = 1.1e28;
        cNOTE.setExchangeRate(newExchangeRate);
        uint256 initialBalance = NOTE.balanceOf(owner);
        uint256 asdSupply = asdToken.totalSupply();
        // Should be able to withdraw 10% in 2 lots
        uint256 withdrawAmount = asdSupply / 20; // = 0.5e18
        vm.prank(owner);
        asdToken.withdrawCarry(withdrawAmount);
        vm.prank(owner);
        asdToken.withdrawCarry(withdrawAmount);
    }
```
</details>

<br><br>

2. The pre-existing test `testWithdrawCarryWithZeroAmount()` provided by the Dev team shows us how an owner should be able to withdraw the **maximum** amount equalling `1e18` NOTE tokens when `exchangeRate` equals `1.1e28`, by passing the `_amount` param to `withdrawCarry()` as 0. Following the same logic -
    - `setExchangeRate()` to `2e28`. Owner should be then able to claim `10e18` NOTE tokens, his maximum claimable amount by passing `_amount` param value as 0 (or he can explicitly pass `10e18`, it works the same way). 
    - However, this fails with `revert: ERC20: burn amount exceeds balance`.

<details>
<summary>PoC-2 : click to expand</summary>

```js
    function test_t0x1c_WithdrawCarryWithZeroAmount() public {
        testMint();
        uint256 newExchangeRate = 2e28;
        cNOTE.setExchangeRate(newExchangeRate);
        uint256 initialBalance = NOTE.balanceOf(owner);
        // Should be able to withdraw 10e18
        uint256 maxWithdrawAmount = 10e18;
        vm.prank(owner);
        asdToken.withdrawCarry(0);
    }
```
</details>

<br><br>

3. The pre-existing test `testBurn()` provided by the Dev team shows us how any user can burn his `asD` tokens to get back NOTE tokens in a `1:1 ratio`. This is made impossible when exchange rate rises -
    - `setExchangeRate()` to `2e28`. 
    - Call `burn()` to burn `6e18` asD tokens. This reverts with `revert: ERC20: burn amount exceeds balance`.

<details>
<summary>PoC-3 : click to expand</summary>

```js
    function test_t0x1c_Burn() public {
        testMint();

        uint256 newExchangeRate = 2e28;
        cNOTE.setExchangeRate(newExchangeRate);

        uint256 burnAmount = 6e18;
        asdToken.burn(burnAmount);
    }
```
</details>

<br><br>

## Tools Used
Manual inspection, foundry.

## Recommended Mitigation Steps
The following changes could be one possible mitigation path. Note that some minor precision loss happens in this case, and hence consider these as rough _(first-draft)_ suggestions. Here's the git patch:

```diff
diff --git a/asD/src/asD.sol b/asD/src/asD.sol
index fc1d14a..bc89b89 100644
--- a/asD/src/asD.sol
+++ b/asD/src/asD.sol
@@ -56,8 +56,8 @@ contract asD is ERC20, Ownable2Step {
     function burn(uint256 _amount) external {
         CErc20Interface cNoteToken = CErc20Interface(cNote);
         IERC20 note = IERC20(cNoteToken.underlying());
-        uint256 returnCode = cNoteToken.redeemUnderlying(_amount); // Request _amount of NOTE (the underlying of cNOTE)
-        require(returnCode == 0, "Error when redeeming"); // 0 on success: https://docs.compound.finance/v2/ctokens/#redeem-underlying
+        uint256 returnCode = cNoteToken.redeem((_amount * 1e28) / CTokenInterface(cNote).exchangeRateCurrent()); // Request _amount of NOTE (the underlying of cNOTE)
+        require(returnCode == 0, "Error when redeeming"); // 0 on success: https://docs.compound.finance/v2/ctokens/#redeem
         _burn(msg.sender, _amount);
         SafeERC20.safeTransfer(note, msg.sender, _amount);
     }
@@ -67,20 +67,22 @@ contract asD is ERC20, Ownable2Step {
     /// @dev The function checks that the owner does not withdraw too much NOTE, i.e. that a 1:1 NOTE:asD exchange rate can be maintained after the withdrawal
     function withdrawCarry(uint256 _amount) external onlyOwner {
         uint256 exchangeRate = CTokenInterface(cNote).exchangeRateCurrent(); // Scaled by 1 * 10^(18 - 8 + Underlying Token Decimals), i.e. 10^(28) in our case
+        uint256 _amount_original = _amount;
+        _amount = ((_amount * 1e28) / exchangeRate);
         // The amount of cNOTE the contract has to hold (based on the current exchange rate which is always increasing) such that it is always possible to receive 1 NOTE when burning 1 asD
         uint256 maximumWithdrawable =
             (CTokenInterface(cNote).balanceOf(address(this)) * exchangeRate) / 1e28 - totalSupply();
         if (_amount == 0) {
-            _amount = maximumWithdrawable;
+            _amount = ((maximumWithdrawable * 1e28) / exchangeRate);
         } else {
-            require(_amount <= maximumWithdrawable, "Too many tokens requested");
+            require(_amount_original <= maximumWithdrawable, "Too many tokens requested");
         }
         // Technically, _amount can still be 0 at this point, which would make the following two calls unnecessary.
         // But we do not handle this case specifically, as the only consequence is that the owner wastes a bit of gas when there is nothing to withdraw
-        uint256 returnCode = CErc20Interface(cNote).redeemUnderlying(_amount);
+        uint256 returnCode = CErc20Interface(cNote).redeem(_amount);
         require(returnCode == 0, "Error when redeeming"); // 0 on success: https://docs.compound.finance/v2/ctokens/#redeem
         IERC20 note = IERC20(CErc20Interface(cNote).underlying());
-        SafeERC20.safeTransfer(note, msg.sender, _amount);
-        emit CarryWithdrawal(_amount);
+        SafeERC20.safeTransfer(note, msg.sender, (_amount * exchangeRate) / 1e28);
+        emit CarryWithdrawal((_amount * exchangeRate) / 1e28);
     }
 }
```

<br><br>

## **MEDIUM-SEVERITY BUGS**
---

### <a id="m-01"></a>[M-01]
## **No slippage protection for user calling `buy()` can cause loss of funds for him**
#### https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L150
<br>

## Impact
When a user calls the [buy()](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L150) function, he can be front-run such that he has to pay way more than he had hoped for, based on the `tokenCount` at the time of placing the order. There is no slippage or `maxPriceLimit` parameter inside `buy()` to protect him from this.

## Proof of Concept
Suppose:
1. The current `shareData[_id].tokenCount` is 10.
2. Bob calls `buy(1, 1)` hoping to pay an amount of `11 * LINEAR_INCREASE` as per the calculation inside [Market::getBuyPrice()](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L135) and [LinearBondingCurve::getBuyPrice()](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L14-L25).
3. Alice front-runs Bob and calls `buy(1, 20)`.
4. `shareData[_id].tokenCount` is now 30.
5. Bob's transaction executes & he ends up paying `31 * LINEAR_INCREASE`, much more than he had planned for. 

## Tools Used
Manual inspection.

## Recommended Mitigation Steps
Add a `maxPriceLimit` parameter inside `buy()` which causes the function to revert if `price` exceeds it.

---

### <a id="m-02"></a>[M-02]
## **Loss of funds for user if burning more than one NFT**
#### https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L295
<br>

## Impact
If a user needs to burn 2 NFTs, and he calls [burnNFT(1, 2)](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L226) then he has to pay more than what he would have if had called `burnNFT(1,1)` twice. <br>
This is because of the difference in the value of calculated [platformPool](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L295) inside `_splitFees()`. Note that a user would instinctively prefer burning both NFTs in one call to save gas, but ends up paying more.

## Proof of Concept
Paste this inside `2023-11-canto/1155tech-contracts/src/test/Market.t.sol` and
- 1st run `forge test --mt test_t0x1c_BurnOneAtATime -vv` to see user's balance as `996098000000000000`.
- Then run `forge test --mt test_t0x1c_BurnTogether -vv` to see that user balance is `996032000000000000`, which is less than before. User had to pay more in fees.

```js
    function test_t0x1c_BurnOneAtATime() public {
        testCreateNewShare();
        token.approve(address(market), type(uint256).max);

        market.buy(1, 2);
        market.mintNFT(1, 1);
        market.mintNFT(1, 1);

        // burn one at a time
        market.burnNFT(1, 1);
        market.burnNFT(1, 1);

        assertEq(token.balanceOf(address(this)), 996098000000000000);
    }

    function test_t0x1c_BurnTogether() public {
        testCreateNewShare();
        token.approve(address(market), type(uint256).max);

        market.buy(1, 2);
        market.mintNFT(1, 1);
        market.mintNFT(1, 1);

        // burn together
        market.burnNFT(1, 2);

        assertEq(token.balanceOf(address(this)), 996032000000000000);
    }
```

## Tools Used
Foundry.

## Recommended Mitigation Steps
Fix the [platformPool](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L295) calculation inside `_splitFees()` so that it matches in both the cases.

---

### <a id="m-03"></a>[M-03]
## **User calling `sell()` has no slippage protection which can cause loss of funds for her**
#### https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L174
<br>

## Impact
When a user calls the [sell()](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L174) function, she can be front-run such that she receives way less than she had hoped for, based on the `tokenCount` at the time of placing the order. There is no slippage or `minPriceExpected` parameter inside `sell()` to protect her from this.

## Proof of Concept
Suppose:
1. The current `shareData[_id].tokenCount` is 30.
2. Alice calls `sell(1, 1)` hoping to receive an amount of `30 * LINEAR_INCREASE` as per the calculation inside [Market::getSellPrice()](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L144) and [LinearBondingCurve::getPriceAndFee()](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L14-L25).
3. Bob front-runs Alice and calls `sell(1, 20)`.
4. `shareData[_id].tokenCount` is now 10.
5. Alice's transaction executes & she ends up receiving only `10 * LINEAR_INCREASE`, much less than she had planned for. She would have rather preferred to wait for tokenCount to go up before selling her tokens.

## Tools Used
Manual inspection.

## Recommended Mitigation Steps
Add a `minPriceExpected` parameter inside `sell()` which causes the function to revert if `price` received is less than it.

---

### <a id="m-04"></a>[M-04]
## **Attacker can sandwich other user's `buy()` order to make immediate profits**
#### https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L150
<br>

## Impact
An attacker can watch the mempool to wait for a user's [buy()](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L150) call and sandwich it to make immediate profits. Since `buy` & `sell` prices depend on the number of tokens, and also since there is no holding-period required before selling, attacker can make immediate profits.

## Proof of Concept
Paste the following inside `2023-11-canto/1155tech-contracts/src/test/Market.t.sol` and run with `forge test --mt test_t0x1c_Sandwich -vv`. Alice is the attacker in this example:

```js
    function test_t0x1c_Sandwich() public {
        testCreateNewShare();
        token.approve(address(market), type(uint256).max);
        vm.prank(alice);
        token.approve(address(market), type(uint256).max);
        deal(address(token), alice, 10e18); // give some funds to Alice

        uint256 aliceInitialBalance = token.balanceOf(alice);

        // simulating some pre-existing orders
        market.buy(1, 2);
        market.buy(1, 3);

        //==============================//
        //======= SANDWICH ATTACK ======//
        //==============================//
        vm.prank(alice);
        market.buy(1, 10); // Front-run

        market.buy(1, 1); // BUY order from naive user

        vm.prank(alice);
        market.sell(1, 10); // Back-run

        // Immediate Profit
        assertGt(token.balanceOf(alice), aliceInitialBalance, "no profit");
    }

```

## Tools Used
Foundry.

## Recommended Mitigation Steps
Protocol can considering adding a `delay` between buy & sell from the same address so that it's not possible within the same block. Also, slippage protection for users will help keep such situations at bay to some extent.

---



