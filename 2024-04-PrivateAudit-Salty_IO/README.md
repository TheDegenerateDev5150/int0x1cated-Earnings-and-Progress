<h1 style="text-align:center"><b>Salty.IO Security Review</b></h1>
<h3 style="text-align:center">Audited by: <i><a href='https://twitter.com/int0x1catedCode' target='_blank' style='color: #d75b5b'>t0x1c</a></i></h3>
<h4 style="text-align:center">8 April, 2024</h4>

<br>

---

# About t0x1c
Having spent 16+ years in the software security industry, **_t0x1c_** has a firm grasp of not only the technical but also the business aspects of a protocol which ought to be focussed on during an audit. He is an independent security researcher with primary focus on the blockchain space. <br>
Reach out on [X (twitter)](https://twitter.com/int0x1catedCode) or [Discord](https://discord.com/users/1055071974948352010) to connect & get smart contracts audited.

<br>

# Disclaimer
Security reviews are a means to uncover bugs in a system. Any security review however can never verify the _complete absence_ of vulnerabilities and hence this audit does not guarantee it either.<br>
The current exercise encompasses a time-bound effort by _t0x1c_ to uncover vulnerabilities while giving his sincerest effort.

<br>

# About Salty.IO
Salty.IO is a Decentralized Exchange which uses Automatic Atomic Arbitrage (AAA) to generate yield and provide Zero Fees on all swaps.<br>
Futher details about the project can be found at [https://docs.salty.io](https://docs.salty.io).

<br>

# Severity Classification Matrix
| Finding               | High Impact       | Medium Impact       | Low Impact              |
|-----------------------|-------------------|---------------------|-------------------------|
| High Likelihood       | Critical Severity | High Severity       | Medium Severity         |
| Medium Likelihood     | High Severity     | Medium Severity     | Low / Informational     |
| Low Likelihood        | Medium Severity   | Low / Informational | Low / Informational     |

<br>

### Impact Classification
- High - Leads to significant loss of assets in the protocol or significantly harms a group of users.
- Medium - Small amount of funds can be lost (for example loss of fee or leakage of value or DoS for an extended period of time) or an important functionality of the protocol is affected.
- Low - Unexpected behavior with some functionality impacted that can be easily repaired or ignored.

### Likelihood Classification
- High - Requires very little or no conditional prerequisites for the attacker to be incentivized and the funds lost are higher than the attack cost.
- Medium - Requires conditional prerequisites for the attacker to be incentivized, but still relatively likely.
- Low - Little or no incentive for the attacker or requires too many unlikely assumptions.

### Action Required for Severity Levels
- Critical - Must fix as soon as possible (if already deployed).
- High - Must fix (before deployment if not already deployed).
- Medium - Should be fixed.
- Low / Informational - For the protocol to decide.

<br>

---

# Executive Summary

### Audited Code Commit
[https://github.com/othernet-global/salty-io/commit/17018a862f13432fee1cf5e4c5bf4173aec1224d](https://github.com/othernet-global/salty-io/commit/17018a862f13432fee1cf5e4c5bf4173aec1224d)

### Scope of Current Audit
The single Airdrop which previously distributed staked xSALT now has two phases with SALT distributed linearly over a year to smooth the distribution. Important files which have seen refactoring are:

- Airdrop.sol 
- BootstrapBallot.sol 
- InitialDistribution.sol 
- Deployment.sol

This audit focuses on the impact of these changes and any vulnerabilities which may have arisen as a result.

<h2 id="summaryTable">Issues Found</h2>

| Severity | Findings |
|:--------|:--------:|
| Critical              | 0 |
| High                  | 0 |
| Medium                | 0 |
| Low / Informational   | 4 |
| **TOTAL**             | **4** |

<br>

---

# Finding Details

| # | ID | Title | Severity |
|--------|--------|------|:------:|
| 1 | [L-01](#L-01)  | Off-chain check required to ensure airdrop allocation to users does not exceed 3 million | Low |
| 2 | [L-02](#L-02)  | `BootstrapBallot::bytes32ToHexString()` is never called | Low |
| 3 | [L-03](#L-03)  | No check that `airdrop2DelayTillDistribution` is greater than 0 | Low |
| 4 | [L-04](#L-04)  | SALT can remain stuck in `Airdrop.sol` | Low |

<br>

## **Low / Informational Findings**

---

<h2 id="L-01">[L-01]</h2>

## **Off-chain check required to ensure airdrop allocation to users does not exceed 3 million**
[https://github.com/othernet-global/salty-io/commit/17018a862f13432fee1cf5e4c5bf4173aec1224d#diff-da73cb28581806405b9a6c18aa354ee8746ada59ee138a43b431c3c7c74fb525L62](https://github.com/othernet-global/salty-io/commit/17018a862f13432fee1cf5e4c5bf4173aec1224d#diff-da73cb28581806405b9a6c18aa354ee8746ada59ee138a43b431c3c7c74fb525L62)
<br>

## Description
In the [previous version of Airdrop.sol](https://github.com/othernet-global/salty-io/commit/17018a862f13432fee1cf5e4c5bf4173aec1224d#diff-da73cb28581806405b9a6c18aa354ee8746ada59ee138a43b431c3c7c74fb525L62) all users received an equal share of the airdrop. 

```js
File: src/launch/Airdrop.sol (old version)

  62:    	// All users receive an equal share of the airdrop.
  63:	    uint256 saltBalance = salt.balanceOf(address(this));
  64:             saltAmountForEachUser = saltBalance / numberAuthorized();
```

Hence it was guaranteed that every eligible user would be able to claim their share of the airdrop i.e. the funds would always be there to be claimed. [In the new implementation](https://github.com/othernet-global/salty-io/commit/17018a862f13432fee1cf5e4c5bf4173aec1224d#diff-da73cb28581806405b9a6c18aa354ee8746ada59ee138a43b431c3c7c74fb525R40-R42), the protocol decides off-chain the `airdropPerUser` amount. Hence the protocol needs to take care off-chain that the sum of `airdropPerUser` for all wallets never exceeds the total airdrop SALT funds or 3 million SALT.

```js
File: src/launch/Airdrop.sol (new version)

  40:     	// Authorize the wallet as being able to claim a specific amount of the airdrop.
  41:      	// The BootstrapBallot would have already confirmed the user is authorized to receive the specified saltAmount.
  42:    	function authorizeWallet( address wallet, uint256 saltAmount ) external
```

## Severity
Likelihood: `Low` (since this is admin controlled and it's assumed that care would be taken off-chain)<br>
Impact: `Med` (user's call to `claim()` will revert due to lack of funds but they can try again later once the admin has added more funds to the contract as a remedy)
<br>

Severity: `Low`

## Recommendation
Ensure the off-chain toolkit verifies that the airdrop allocations do not exceed 3 million for each phase.

[Back to Top](#summaryTable)
 
---

<h2 id="L-02">[L-02]</h2>

## **`BootstrapBallot::bytes32ToHexString()` is never called**
[https://github.com/othernet-global/salty-io/blob/17018a862f13432fee1cf5e4c5bf4173aec1224d/src/launch/BootstrapBallot.sol#L54-L65](https://github.com/othernet-global/salty-io/blob/17018a862f13432fee1cf5e4c5bf4173aec1224d/src/launch/BootstrapBallot.sol#L54-L65)
<br>

## Description
The internal function [bytes32ToHexString()](https://github.com/othernet-global/salty-io/blob/17018a862f13432fee1cf5e4c5bf4173aec1224d/src/launch/BootstrapBallot.sol#L54-L65) is never called by the protocol thus wasting deployment time gas.

```js
  File: src/launch/BootstrapBallot.sol

  54:                   function bytes32ToHexString(bytes32 input) internal pure returns (string memory) {
  55:                                   bytes memory lookup = "0123456789abcdef";
  56:                                   bytes memory result = new bytes(64);
  57:                                   for (uint i = 0; i < 32; i++) {
  58:                                           uint8 currentByte = uint8(input[i]);
  59:                                           uint8 hi = uint8(currentByte / 16);
  60:                                           uint8 lo = currentByte - 16 * hi;
  61:                                           result[i*2] = lookup[hi];
  62:                                           result[i*2+1] = lookup[lo];
  63:                                   }
  64:                                   return string(result);
  65:                           }
```

## Recommendation
Remove the unused function.

[Back to Top](#summaryTable)

 
---

<h2 id="L-03">[L-03]</h2>

## **No check that `airdrop2DelayTillDistribution` is greater than 0**
[https://github.com/othernet-global/salty-io/blob/17018a862f13432fee1cf5e4c5bf4173aec1224d/src/launch/BootstrapBallot.sol#L38](https://github.com/othernet-global/salty-io/blob/17018a862f13432fee1cf5e4c5bf4173aec1224d/src/launch/BootstrapBallot.sol#L38)
<br>

## Description
It can be inferred from the code comments and general flow that the protocol intends the `airdrop2` to happen some interval after `airdrop1` claims are opened. However, such an explicit check seems to be missing and the admin can call `BootstrapBallot.sol` with `airdrop2DelayTillDistribution = 0`. 

```js
  File: src/launch/BootstrapBallot.sol

  38:                   constructor( IExchangeConfig _exchangeConfig, IAirdrop _airdrop1, IAirdrop _airdrop2, uint256 ballotDuration, uint256 airdrop2DelayTillDistribution )
  39:                           {
  40:                           require( ballotDuration > 0, "ballotDuration cannot be zero" );
  41:
  42:                           exchangeConfig = _exchangeConfig;
  43:                           airdrop1 = _airdrop1;
  44:                           airdrop2 = _airdrop2;
  45:
  46:                           // Airdrop I is claimable when the BootstrapBallot is completed
  47:                           claimableTimestamp1 = block.timestamp + ballotDuration;
  48:
  49:                           // Airdrop 2 is claimable a certain number of days after Airdrop 1 completes
  50:  @--->                    claimableTimestamp2 = claimableTimestamp1 + airdrop2DelayTillDistribution;
  51:                           }
```

## Severity
Likelihood: `Low` (since this is admin controlled)<br>
Impact: `Low` (no functionality is actually impacted)
<br>

Severity: `Low`

## Recommendation
Add a `require` statement to check that `airdrop2DelayTillDistribution > 0`.

[Back to Top](#summaryTable)

---

<h2 id="L-04">[L-04]</h2>

## **SALT can remain stuck in `Airdrop.sol`**
[https://github.com/othernet-global/salty-io/blob/17018a862f13432fee1cf5e4c5bf4173aec1224d/src/launch/Airdrop.sol#L63](https://github.com/othernet-global/salty-io/blob/17018a862f13432fee1cf5e4c5bf4173aec1224d/src/launch/Airdrop.sol#L63)
<br>

## Description
It's possible that some `airdrop1` users do not vote and hence are not authorized to claim their SALT via `claim()`. Similarly in `airdrop2`, it's possible that some users fail to call `authorizeAirdrop2()` and hence can't claim their SALT later on. In such cases, the [SALT transferred to the airdrop contracts](https://github.com/othernet-global/salty-io/blob/17018a862f13432fee1cf5e4c5bf4173aec1224d/src/launch/InitialDistribution.sol#L59-L63) would remain stuck in the contract since there is no way for the admin to add beneficiaries later on or transfer these funds out of the contract.

## Severity
Likelihood: `Medium` (there often are users who miss the time window)<br>
Impact: `Low` (even though some SALT is stuck in the contract, there seems to be no huge negative impact on the overall day-to-day functioning of the protocol)
<br>

Severity: `Low`

## Recommendation
Add a `withdrawStuckFunds()` function which can be called by the admin only 2 years after the start of `airdrop2`. This would ensure that the admin is not able to steal funds in the intermediate time period.

[Back to Top](#summaryTable)

