# Leaderboard
[Taiko Results](https://code4rena.com/audits/2024-03-taiko#top)<br>

`Rank 2 / 69`

# Audited Code Repo
### [Code4rena: Taiko](https://github.com/code-423n4/2024-03-taiko)

<br>

# Bugs Filed & Their Status

| # | Bug ID | Name | URL | Adjudged Status | Count of Dups (apart from this submission) |
|--------|--------|------|:------:|-----------------:|:-----:|
| 1 | [H-01](#h-01)  | suspendMessages() sets incorrect `receivedAt` timestamp which enables user to bypass `_proof` | [47](https://github.com/code-423n4/2024-03-taiko-findings/issues/47) | Accepted as Medium | 3 |
| 2 | [H-02](#h-02)  | sendToken() function inside ERC721Vault is completely broken | [108](https://github.com/code-423n4/2024-03-taiko-findings/issues/108) | Rejected | - |
| 3 | [H-03](#h-03)  | Incomplete validation in _invokeMessageCall() allows attackers to pass arbitrary data and steal tokens | [133](https://github.com/code-423n4/2024-03-taiko-findings/issues/133) | Rejected | - |
| 4 | [H-04](#h-04)  | Modifier `onlyFromNamed("taiko")` is applied incorrectly assuming there is only one "taiko" address on a chain, thus breaking AssignmentHook::onBlockProposed() and SgxVerifier::verifyProof() | [135](https://github.com/code-423n4/2024-03-taiko-findings/issues/135) | Rejected | - |
| 5 | [H-05](#h-05)  | Missing addressManager in TaikoToken.sol | [144](https://github.com/code-423n4/2024-03-taiko-findings/issues/144) | Accepted as Med | 3 |
| 6 | [M-01](#m-01)  | No way to `changeBridgedToken()` back to an old btoken | [43](https://github.com/code-423n4/2024-03-taiko-findings/issues/43) | QA | - |
| 7 | [M-02](#m-02)  | Banned address can still `_invokeMessageCall()` by calling `retryMessage()` | [48](https://github.com/code-423n4/2024-03-taiko-findings/issues/48) | Accepted as Med | 12 |
| 8 | [M-03](#m-03)  | No incentive for message non-owners to retryMessage() | [69](https://github.com/code-423n4/2024-03-taiko-findings/issues/69) | Rejected | - |
| 9 | [M-04](#m-04)  | Malicious caller of `processMessage()` can pocket the fee while forcing `excessivelySafeCall()` to fail | [97](https://github.com/code-423n4/2024-03-taiko-findings/issues/97) | Accepted as Med; Selected report | 2 |
|10 | [M-05](#m-05)  | Incorrect calculations for provingWindow and cooldownWindow could cause a state transition to spend more than expected time in these windows | [119](https://github.com/code-423n4/2024-03-taiko-findings/issues/119) | QA | - |
|11 | [M-06](#m-06)  | Invocation delays are not honoured when protocol unpauses | [170](https://github.com/code-423n4/2024-03-taiko-findings/issues/170) | Accepted as Med; Selected report; `Unique` | 0 |
|12 | [M-07](#m-07)  | Proposers would choose to avoid higher tier by exploiting non-randomness of parameter used in getMinTier() | [172](https://github.com/code-423n4/2024-03-taiko-findings/issues/172) | Accepted as Med; Selected report | 1 |
|13 | [M-08](#m-08)  | Griefer can front-run processMessage() with a call to recallMessage() due to incorrect invocationDelay implementation | [175](https://github.com/code-423n4/2024-03-taiko-findings/issues/175) | Rejected | - |
|14 | [M-09](#m-09)  | Prover loses funds even if proven right after multiple contests | [227](https://github.com/code-423n4/2024-03-taiko-findings/issues/227) | Accepted as High | 1 |
|15 | [M-10](#m-10)  | Protocol does not check inside GuardianProver::approve() if all the guardians are approving the same proof | [248](https://github.com/code-423n4/2024-03-taiko-findings/issues/248) | Accepted as Med | 1 |
|16 | [M-11](#m-11)  | Incorrect implementation of "precision is maintained at the token level rather than the wei level" leads to division before multiplication & hence fee loss | [265](https://github.com/code-423n4/2024-03-taiko-findings/issues/265) | QA | - |


<br>

## **HIGH-SEVERITY BUGS**

---

### <a id="h-01"></a>[H-01]
## **suspendMessages() sets incorrect `receivedAt` timestamp which enables user to bypass `_proof`**
#### https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L89-L92
<br>

## Summary & Impact
Even for unproven messages, the value of `receivedAt` can be set to a non-zero value inside [suspendMessages()](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L89-L92) due to incomplete validation.

Whenever a message is unsuspended, the protocol does not check for it's previous proven/unproven status. So even if the message is unproven and `receivedAt` is supposed to be zero, the unsuspension sets it to `block.timestamp`. This practically acts as marking the message as proven, thus enabling a user to bypass important checks and: 
- perform the function calls `processMessage()` and `recallMessage()` without a valid `_proof`. 
- process a message even when its `_proveSignalReceived = false`.
- get tokens/ether credited without actually proving it.
- recall the message & get back ether even though the message never failed.

## Details
Consider the following flow of events:
- Alice sends a message.
- Owner calls `suspendMessages()` to suspend Alice's message i.e. `_suspend = true`.
- Soon after, owner calls `suspendMessages()` to un-suspend Alice's message i.e. `_suspend = false`.
  - **_Note that_** owner may even directly un-suspend the message, even though it was never in a suspended state initially (basically skipping step-2) as there is no check which ensures that only a suspended message is allowed to be unsuspended. But that would be quite illogical to do, so we've assumed a valid flow of events.
- Alice's message, which had not yet been proven and still had a `receivedAt = 0`, now is incorrectly assigned `receivedAt = block.timestamp` on [L92](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L92).
- Two attack vectors can now be carried out:
  - **_Attack Vector - 1:_**
    - User now calls `processMessage()` with an invalid `_proof`. 
    - The `if` condition on [L235](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L235) is bypassed since the message is incorrectly considered 'proven' by the protocol on [L231](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L231). Hence it is never checked on [L236](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L236) if the message signal was really received.
    - Funds are drained on [L294-L300](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L294-L300).

  - **_Attack Vector - 2:_**
    - User now calls `recallMessage()` with an invalid `_proof`. 
    - The `if` condition on [L171](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L171) is bypassed since the message is incorrectly considered 'proven' by the protocol on [L169](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L169). Hence it is never checked on [L179](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L179) if the message really failed.
    - Alice receives her ether back either on [L199](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L199) or on [L206](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L206). 
    - There's **an additional griefing attack vector** possible here:
      - **_A griefer_** can call `recallMessage()` (even front-running an authentic `processMessage()` by the message owner) at `timestamp = invocationDelay + receivedAt` and cause the message status to be changed to `RECALLED`, thus disallowing any future chances of a `processMessage()` call. Note that the griefer need not provide a valid `_proof` now since the check on [L179](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L179) does not ever happen.

<br>

<details><summary>Click to view relevant code snippets</summary>

```js
  File: protocol/contracts/bridge/Bridge.sol

  155:              function recallMessage(
  156:                  Message calldata _message,
  157:                  bytes calldata _proof
  158:              )
  159:                  external
  160:                  nonReentrant
  161:                  whenNotPaused
  162:                  sameChain(_message.srcChainId)
  163:              {
  164:                  bytes32 msgHash = hashMessage(_message);
  165:
  166:                  if (messageStatus[msgHash] != Status.NEW) revert B_STATUS_MISMATCH();
  167:
  168:                  uint64 receivedAt = proofReceipt[msgHash].receivedAt;
  169: @--->            bool isMessageProven = receivedAt != 0;
  170:
  171: @--->            if (!isMessageProven) {
  172:                      address signalService = resolve("signal_service", false);
  173:
  174:                      if (!ISignalService(signalService).isSignalSent(address(this), msgHash)) {
  175:                          revert B_MESSAGE_NOT_SENT();
  176:                      }
  177:
  178:                      bytes32 failureSignal = signalForFailedMessage(msgHash);
  179: @--->                if (!_proveSignalReceived(signalService, failureSignal, _message.destChainId, _proof)) {
  180:                          revert B_NOT_FAILED();
  181:                      }
  182:
  183:                      receivedAt = uint64(block.timestamp);
  184:                      proofReceipt[msgHash].receivedAt = receivedAt;
  185:                  }
  186:
  187:                  (uint256 invocationDelay,) = getInvocationDelays();
  188:
  189:                  if (block.timestamp >= invocationDelay + receivedAt) {
  190:                      delete proofReceipt[msgHash];
  191:                      messageStatus[msgHash] = Status.RECALLED;
  192:
  193:                      // Execute the recall logic based on the contract's support for the
  194:                      // IRecallableSender interface
  195:                      if (_message.from.supportsInterface(type(IRecallableSender).interfaceId)) {
  196:                          _storeContext(msgHash, address(this), _message.srcChainId);
  197:
  198:                          // Perform recall
  199: @--->                    IRecallableSender(_message.from).onMessageRecalled{ value: _message.value }(
  200:                              _message, msgHash
  201:                          );
  202:
  203:                          // Must reset the context after the message call
  204:                          _resetContext();
  205:                      } else {
  206: @--->                    _message.srcOwner.sendEther(_message.value);
  207:                      }
  208:                      emit MessageRecalled(msgHash);
  209:                  } else if (!isMessageProven) {
  210:                      emit MessageReceived(msgHash, _message, true);
  211:                  } else {
  212:                      revert B_INVOCATION_TOO_EARLY();
  213:                  }
  214:              }
  215:
  216:              /// @inheritdoc IBridge
  217:              function processMessage(
  218:                  Message calldata _message,
  219:                  bytes calldata _proof
  220:              )
  221:                  external
  222:                  nonReentrant
  223:                  whenNotPaused
  224:                  sameChain(_message.destChainId)
  225:              {
  226:                  bytes32 msgHash = hashMessage(_message);
  227:                  if (messageStatus[msgHash] != Status.NEW) revert B_STATUS_MISMATCH();
  228:
  229:                  address signalService = resolve("signal_service", false);
  230:                  uint64 receivedAt = proofReceipt[msgHash].receivedAt;
  231: @--->            bool isMessageProven = receivedAt != 0;
  232:
  233:                  (uint256 invocationDelay, uint256 invocationExtraDelay) = getInvocationDelays();
  234:
  235: @--->            if (!isMessageProven) {
  236:                      if (!_proveSignalReceived(signalService, msgHash, _message.srcChainId, _proof)) {
  237:                          revert B_NOT_RECEIVED();
  238:                      }
  239:
  240:                      receivedAt = uint64(block.timestamp);
  241:
  242:                      if (invocationDelay != 0) {
  243: @--->                    proofReceipt[msgHash] = ProofReceipt({
  244:                              receivedAt: receivedAt,
  245:                              preferredExecutor: _message.gasLimit == 0 ? _message.destOwner : msg.sender
  246:                          });
  247:                      }
  248:                  }
  249:
  250:                  if (invocationDelay != 0 && msg.sender != proofReceipt[msgHash].preferredExecutor) {
  251:                      // If msg.sender is not the one that proved the message, then there
  252:                      // is an extra delay.
  253:                      unchecked {
  254:                          invocationDelay += invocationExtraDelay;
  255:                      }
  256:                  }
  257:
  258:                  if (block.timestamp >= invocationDelay + receivedAt) {
  259:                      // If the gas limit is set to zero, only the owner can process the message.
  260:                      if (_message.gasLimit == 0 && msg.sender != _message.destOwner) {
  261:                          revert B_PERMISSION_DENIED();
  262:                      }
  263:
  264:                      delete proofReceipt[msgHash];
  265:
  266:                      uint256 refundAmount;
  267:
  268:                      // Process message differently based on the target address
  269:                      if (
  270:                          _message.to == address(0) || _message.to == address(this)
  271:                              || _message.to == signalService || addressBanned[_message.to]
  272:                      ) {
  273:                          // Handle special addresses that don't require actual invocation but
  274:                          // mark message as DONE
  275:                          refundAmount = _message.value;
  276:                          _updateMessageStatus(msgHash, Status.DONE);
  277:                      } else {
  278:                          // Use the specified message gas limit if called by the owner, else
  279:                          // use remaining gas
  280:                          uint256 gasLimit = msg.sender == _message.destOwner ? gasleft() : _message.gasLimit;
  281:
  282:                          if (_invokeMessageCall(_message, msgHash, gasLimit)) {
  283:                              _updateMessageStatus(msgHash, Status.DONE);
  284:                          } else {
  285:                              _updateMessageStatus(msgHash, Status.RETRIABLE);
  286:                          }
  287:                      }
  288:
  289:                      // Determine the refund recipient
  290:                      address refundTo =
  291:                          _message.refundTo == address(0) ? _message.destOwner : _message.refundTo;
  292:
  293:                      // Refund the processing fee
  294:                      if (msg.sender == refundTo) {
  295:                          refundTo.sendEther(_message.fee + refundAmount);
  296:                      } else {
  297:                          // If sender is another address, reward it and refund the rest
  298:                          msg.sender.sendEther(_message.fee);
  299:                          refundTo.sendEther(refundAmount);
  300:                      }
  301:                      emit MessageExecuted(msgHash);
  302:                  } else if (!isMessageProven) {
  303:                      emit MessageReceived(msgHash, _message, false);
  304:                  } else {
  305:                      revert B_INVOCATION_TOO_EARLY();
  306:                  }
  307:              }
```

```js
  File: protocol/contracts/bridge/Bridge.sol

  79:               /// @notice Suspend or unsuspend invocation for a list of messages.
  80:               /// @param _msgHashes The array of msgHashes to be suspended.
  81:               /// @param _suspend True if suspend, false if unsuspend.
  82:               function suspendMessages(
  83:                   bytes32[] calldata _msgHashes,
  84:                   bool _suspend
  85:               ) 
  86:                   external
  87:                   onlyFromOwnerOrNamed("bridge_watchdog")
  88:               {
  89:  @--->            uint64 _timestamp = _suspend ? type(uint64).max : uint64(block.timestamp);  
  90:                   for (uint256 i; i < _msgHashes.length; ++i) {
  91:                       bytes32 msgHash = _msgHashes[i];
  92:  @--->                proofReceipt[msgHash].receivedAt = _timestamp;
  93:                       emit MessageSuspended(msgHash, _suspend);
  94:                   }
  95:               }
```

</details>

<br>

## Tools Used
Manual review

## Recommended Mitigation Steps
Introduce a new boolean mapping `isProven[msgHash]` which stores whether the message was proven or not before being suspended/unsuspended.
```diff
    function suspendMessages(
        bytes32[] calldata _msgHashes,
        bool _suspend
    )
        external
        onlyFromOwnerOrNamed("bridge_watchdog")
    {
        uint64 _timestamp = _suspend ? type(uint64).max : uint64(block.timestamp);
        for (uint256 i; i < _msgHashes.length; ++i) {
            bytes32 msgHash = _msgHashes[i];
            proofReceipt[msgHash].receivedAt = _timestamp;
+           if (!isProven[msgHash] && !_suspend) proofReceipt[msgHash].receivedAt = 0;  // set it to zero if unsuspension of an unproven message is being done
            emit MessageSuspended(msgHash, _suspend);
        }
    }


    ...
    ...


    function recallMessage(
        Message calldata _message,
        bytes calldata _proof
    )
        external
        nonReentrant
        whenNotPaused
        sameChain(_message.srcChainId)
    {
        bytes32 msgHash = hashMessage(_message);

        if (messageStatus[msgHash] != Status.NEW) revert B_STATUS_MISMATCH();

        uint64 receivedAt = proofReceipt[msgHash].receivedAt;
        bool isMessageProven = receivedAt != 0;
+       if (receivedAt != type(uint64).max) isProven[msgHash] = isMessageProven;  // if not suspended, update `isProven`

        if (!isMessageProven) {
            address signalService = resolve("signal_service", false);


    ...
    ...


    function processMessage(
        Message calldata _message,
        bytes calldata _proof
    )
        external
        nonReentrant
        whenNotPaused
        sameChain(_message.destChainId)
    {
        bytes32 msgHash = hashMessage(_message);
        if (messageStatus[msgHash] != Status.NEW) revert B_STATUS_MISMATCH();

        address signalService = resolve("signal_service", false);
        uint64 receivedAt = proofReceipt[msgHash].receivedAt;
        bool isMessageProven = receivedAt != 0;
+       if (receivedAt != type(uint64).max) isProven[msgHash] = isMessageProven;  // if not suspended, update `isProven`

        (uint256 invocationDelay, uint256 invocationExtraDelay) = getInvocationDelays();

        if (!isMessageProven) {
            if (!_proveSignalReceived(signalService, msgHash, _message.srcChainId, _proof)) {

```

---

### <a id="h-02"></a>[H-02]
## **sendToken() function inside ERC721Vault is completely broken**
#### https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/tokenvault/ERC721Vault.sol#L35
<br>

## Details & Impact
No user can send any ERC721 token due to use of `!=` instead of `==` on [L35](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/tokenvault/ERC721Vault.sol#L35):
```js
  File: contracts/tokenvault/ERC721Vault.sol

  21:               /// @notice Transfers ERC721 tokens to this vault and sends a message to the
  22:               /// destination chain so the user can receive the same (bridged) tokens
  23:               /// by invoking the message call.
  24:               /// @param _op Option for sending the ERC721 token.
  25:               /// @return message_ The constructed message.
  26:               function sendToken(BridgeTransferOp memory _op)
  27:                   external
  28:                   payable
  29:                   nonReentrant
  30:                   whenNotPaused
  31:                   withValidOperation(_op)
  32:                   returns (IBridge.Message memory message_)
  33:               {
  34:                   for (uint256 i; i < _op.tokenIds.length; ++i) {
  35:  @--->                if (_op.amounts[i] != 0) revert VAULT_INVALID_AMOUNT();
  36:                   }
```

## Tools Used
Manual review

## Recommended Mitigation Steps
```diff
  File: contracts/tokenvault/ERC721Vault.sol

  21:               /// @notice Transfers ERC721 tokens to this vault and sends a message to the
  22:               /// destination chain so the user can receive the same (bridged) tokens
  23:               /// by invoking the message call.
  24:               /// @param _op Option for sending the ERC721 token.
  25:               /// @return message_ The constructed message.
  26:               function sendToken(BridgeTransferOp memory _op)
  27:                   external
  28:                   payable
  29:                   nonReentrant
  30:                   whenNotPaused
  31:                   withValidOperation(_op)
  32:                   returns (IBridge.Message memory message_)
  33:               {
  34:                   for (uint256 i; i < _op.tokenIds.length; ++i) {
- 35:                       if (_op.amounts[i] != 0) revert VAULT_INVALID_AMOUNT();
+ 35:                       if (_op.amounts[i] == 0) revert VAULT_INVALID_AMOUNT();
  36:                   }
```

---

### <a id="h-03"></a>[H-03]
## **Incomplete validation in _invokeMessageCall() allows attackers to pass arbitrary data and steal tokens**
#### https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L492
#### https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/tokenvault/ERC20Vault.sol#L253
<br>

## Summary
In `_invokeMessageCall()`, the [check for msg.data](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L492) not being equal to `IMessageInvocable.onMessageInvocation.selector` is made to ensure the low-level `excessivelySafeCall()` can't be used to execute malicious arbitrary data. This check however is not enough because in `ERC20Vault.sol`, `ERC721Vault.sol` and `ERC1155Vault.sol`, the function `onMessageInvocation()` is [marked external](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/tokenvault/ERC20Vault.sol#L254) and the ['onlyFromBridge' check](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/tokenvault/ERC20Vault.sol#L262) will pass too because in the case of a low-level call, `msg.sender` would be the bridge contract.
This enables the attacker to pass `_message` inside `_invokeMessageCall()` which has `_message.to` equal to the address of the Vault contract and `msg.data[0:4]` equalling `IMessageInvocable.onMessageInvocation.selector`, thus bypassing all checks on [L491-L493](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L491-L493).

```js
  File: contracts/bridge/Bridge.sol

  477:              function _invokeMessageCall(
  478:                  Message calldata _message,
  479:                  bytes32 _msgHash,
  480:                  uint256 _gasLimit
  481:              )
  482:                  private
  483:                  returns (bool success_)
  484:              {
  485:                  if (_gasLimit == 0) revert B_INVALID_GAS_LIMIT();
  486:                  assert(_message.from != address(this));
  487:          
  488:                  _storeContext(_msgHash, _message.from, _message.srcChainId);
  489:          
  490:                  if (
  491:                      _message.data.length >= 4 // msg can be empty
  492: @--->                    && bytes4(_message.data) != IMessageInvocable.onMessageInvocation.selector
  493:                          && _message.to.isContract()
  494:                  ) {
  495:                      success_ = false;
  496:                  } else {
  497:                      (success_,) = ExcessivelySafeCall.excessivelySafeCall(
  498:                          _message.to,
  499:                          _gasLimit,
  500:                          _message.value,
  501:                          64, // return max 64 bytes
  502:                          _message.data
  503:                      );
  504:                  }
```

and 

```js
  File: contracts/tokenvault/ERC20Vault.sol

  253:              function onMessageInvocation(bytes calldata _data)
  254:                  external
  255:                  payable
  256:                  nonReentrant
  257:                  whenNotPaused
  258:              {
  259:                  (CanonicalERC20 memory ctoken, address from, address to, uint256 amount) =
  260:                      abi.decode(_data, (CanonicalERC20, address, address, uint256));
  261:          
  262: @--->            // `onlyFromBridge` checked in checkProcessMessageContext
  263:                  IBridge.Context memory ctx = checkProcessMessageContext();
  264:          
  265:                  // Don't allow sending to disallowed addresses.
  266:                  // Don't send the tokens back to `from` because `from` is on the source chain.
  267:                  if (to == address(0) || to == address(this)) revert VAULT_INVALID_TO();
  268:          
  269:                  // Transfer the ETH and the tokens to the `to` address
  270: @--->            address token = _transferTokens(ctoken, to, amount);
  271:                  to.sendEther(msg.value);
  272:          
  273:                  emit TokenReceived({
  274:                      msgHash: ctx.msgHash,
  275:                      from: from,
  276:                      to: to,
  277:                      srcChainId: ctx.srcChainId,
  278:                      ctoken: ctoken.addr,
  279:                      token: token,
  280:                      amount: amount
  281:                  });
  282:              }
```

## Description
This issue is similar to the first High severity issue reported by QuillAudits in their report -
> 1. Unverified arbitrary data call allows attackers to bypass all validations and pass malicious message hashes directly to the signal service

The checks in [L491-L493](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L491-L493) were introduced [in this PR](https://github.com/taikoxyz/taiko-mono/commit/f7a12b8601937eef97068c3029c91dff431c03a8#diff-30d39df7524f605d6996c381114d5f82102018c6a676d064d9d22f1c4ae2953d) to mitigate the issue. However the presence of `onMessageInvocation()` inside the vault contracts leaves an entry point open for the attacker. The attacker can craft malicious data which decodes to desired values here:

```js
  File: contracts/tokenvault/ERC20Vault.sol

  253:              function onMessageInvocation(bytes calldata _data)
  254:                  external
  255:                  payable
  256:                  nonReentrant
  257:                  whenNotPaused
  258:              {
  259:  @---->          (CanonicalERC20 memory ctoken, address from, address to, uint256 amount) =
  260:  @---->              abi.decode(_data, (CanonicalERC20, address, address, uint256));
  ```

The attack would be complete when tokens are transferred into their account on [L270](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/tokenvault/ERC20Vault.sol#L270):
```js
  File: contracts/tokenvault/ERC20Vault.sol

  253:              function onMessageInvocation(bytes calldata _data)
  254:                  external
  255:                  payable
  256:                  nonReentrant
  257:                  whenNotPaused
  258:              {
  259:                  (CanonicalERC20 memory ctoken, address from, address to, uint256 amount) =
  260:                      abi.decode(_data, (CanonicalERC20, address, address, uint256));
  261:          
  262:                  // `onlyFromBridge` checked in checkProcessMessageContext
  263:                  IBridge.Context memory ctx = checkProcessMessageContext();
  264:          
  265:                  // Don't allow sending to disallowed addresses.
  266:                  // Don't send the tokens back to `from` because `from` is on the source chain.
  267:                  if (to == address(0) || to == address(this)) revert VAULT_INVALID_TO();
  268:          
  269:                  // Transfer the ETH and the tokens to the `to` address
  270: @--->            address token = _transferTokens(ctoken, to, amount);
```

The attacker can craft such data to call a function like [processMessage()](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L217-L218) which internally [calls _invokeMessageCall()](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L282).

## Tools Used
Manual review

## Recommended Mitigation Steps
One possible solution would be to add an extra check between [L491-L493](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L491-L493) which makes sure that `message.to` is not one of the vault contracts.

---

### <a id="h-04"></a>[H-04]
## **Modifier `onlyFromNamed("taiko")` is applied incorrectly assuming there is only one "taiko" address on a chain, thus breaking AssignmentHook::onBlockProposed() and SgxVerifier::verifyProof()**
#### https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/L1/hooks/AssignmentHook.sol#L70
#### https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/verifiers/SgxVerifier.sol#L145
<br>

## Summary
In a [similar PR](https://github.com/taikoxyz/taiko-mono/pull/15807) in February, it was observed that the protocol cannot assume that only one "taiko" address exists on a particular chain, and hence `onlyFromNamed("taiko")` modifier shouldn't be used in Signal Service. Although [that was fixed](https://github.com/taikoxyz/taiko-mono/commit/a652ae8dc0108e2799a449cce4e5f795f87908a1#diff-d764a5c346ea985003ec5db284de6cac04816c864998addb01f2ed2b653ccec4), the same issue still persists in [AssignmentHook::onBlockProposed()](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/L1/hooks/AssignmentHook.sol#L70) and [SgxVerifier::verifyProof()](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/verifiers/SgxVerifier.sol#L145).

## Description
```js
  File: contracts/L1/hooks/AssignmentHook.sol

  62:               function onBlockProposed(
  63:                   TaikoData.Block memory _blk,
  64:                   TaikoData.BlockMetadata memory _meta,
  65:                   bytes memory _data
  66:               )
  67:                   external
  68:                   payable
  69:                   nonReentrant
  70:  @--->            onlyFromNamed("taiko")
  71:               {
```

and

```js
  File: contracts/verifiers/SgxVerifier.sol

  139:              function verifyProof(
  140:                  Context calldata _ctx,
  141:                  TaikoData.Transition calldata _tran,
  142:                  TaikoData.TierProof calldata _proof
  143:              )
  144:                  external
  145: @--->            onlyFromNamed("taiko")
  146:              {
```

Although the current tests are setup with [only one relayer on L1](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/test/L1/TaikoL1TestBase.sol#L112) and [L2](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/test/L1/TaikoL1TestBase.sol#L118) each and are named "taiko", in production this might not be always true. Multiple relayers could be registered under different names and we need to ensure a relayer is authorized if they are calling [AssignmentHook::onBlockProposed()](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/L1/hooks/AssignmentHook.sol#L70) or [SgxVerifier::verifyProof()](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/verifiers/SgxVerifier.sol#L145).

As per current logic, other relayers will not be able to access these functions, hence breaking major functionality like not being able to verify a proof or assign a block.

## Additional Impact
[SgxVerifier.sol::getSignedHash()](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/verifiers/SgxVerifier.sol#L181) employs similar logic, thus not handling the possibility of multiple L1 relayers:
```js
    function getSignedHash(
        TaikoData.Transition memory _tran,
        address _newInstance,
        address _prover,
        bytes32 _metaHash
    )
        public
        view
        returns (bytes32)
    {
@---->  address taikoL1 = resolve("taiko", false);
        return keccak256(
            abi.encode(
                "VERIFY_PROOF",
@---->          ITaikoL1(taikoL1).getConfig().chainId,
                address(this),
                _tran,
                _newInstance,
                _prover,
                _metaHash
            )
        );
    }
```

## Tools Used
Manual review

## Recommended Mitigation Steps
The owner needs to call [authorize()](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/signal/SignalService.sol#L56) first to authorize any relayers. Then, remove the `onlyFromNamed("taiko")` modifier and add the following check inside these functions:
```js
if (!isAuthorized[msg.sender]) revert SS_UNAUTHORIZED();
```

---

### <a id="h-05"></a>[H-05]
## **Missing addressManager in TaikoToken.sol**
#### https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/L1/TaikoToken.sol#L34
#### https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/L1/TaikoToken.sol#L52
<br>

## Description & Impact
The [snapshot() function](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/L1/TaikoToken.sol#L52) has the `onlyFromOwnerOrNamed("snapshooter")` modifier attached to it. This modifier calls [resolve()](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/common/EssentialContract.sol#L42) inside `EssentialContract.sol` which eventually calls [_resolve() inside AddressResolver.sol](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/common/AddressResolver.sol#L72). The `_resolve()` function reverts if `addressManager` is `address(0)` on [L81](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/common/AddressResolver.sol#L81).

Since the [init() function of TaikoToken.sol only calls __Essential_init(_owner) and never __Essential_init(_owner, addressManager)](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/L1/TaikoToken.sol#L34) i.e. never passing `addressManager` as a param to the `init()` function, the [snapshot() function](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/L1/TaikoToken.sol#L52) will always revert, thus never allowing the 'snapshooter' to take a snapshot.

## Relevant Code Snippets
```js
  File: contracts/L1/TaikoToken.sol

  25:               function init(
  26:                   address _owner,
  27:                   string calldata _name,
  28:                   string calldata _symbol,
  29:                   address _recipient
  30:               )
  31:                   public
  32:                   initializer
  33:               {
  34:  @--->            __Essential_init(_owner);
  35:                   __ERC20_init(_name, _symbol);
  36:                   __ERC20Snapshot_init();
  37:                   __ERC20Votes_init();
  38:                   __ERC20Permit_init(_name);
  39:           
  40:                   // Mint 1 billion tokens
  41:                   _mint(_recipient, 1_000_000_000 ether);
  42:               }
  43:           
  44:               /// @notice Burns tokens from the specified address.
  45:               /// @param _from The address to burn tokens from.
  46:               /// @param _amount The amount of tokens to burn.
  47:               function burn(address _from, uint256 _amount) public onlyOwner {
  48:                   _burn(_from, _amount);
  49:               }
  50:           
  51:               /// @notice Creates a new token snapshot.
  52:  @--->        function snapshot() public onlyFromOwnerOrNamed("snapshooter") {
  53:                   _snapshot();
  54:               }
```

```js
  File: contracts/common/EssentialContract.sol

  41:               modifier onlyFromOwnerOrNamed(bytes32 _name) {
  42:  @--->            if (msg.sender != owner() && msg.sender != resolve(_name, true)) revert RESOLVER_DENIED();
  43:                   _;
  44:               }
```

```js
  File: contracts/common/AddressResolver.sol

  72:               function _resolve(
  73:                   uint64 _chainId,
  74:                   bytes32 _name,
  75:                   bool _allowZeroAddress
  76:               )
  77:                   private
  78:                   view
  79:                   returns (address payable addr_)
  80:               {
  81:  @--->            if (addressManager == address(0)) revert RESOLVER_INVALID_MANAGER();
  82:           
  83:                   addr_ = payable(IAddressManager(addressManager).getAddress(_chainId, _name));
  84:           
  85:                   if (!_allowZeroAddress && addr_ == address(0)) {
  86:                       revert RESOLVER_ZERO_ADDR(_chainId, _name);
  87:                   }
  88:               }
```

## Tools Used
Manual review

## Recommended Mitigation Steps
Pass the `addressManager` as a param to `init()` and call `__Essential_init(_owner, addressManager)` inside it.


<br><br>

## **MEDIUM-SEVERITY BUGS**
---

### <a id="m-01"></a>[M-01]
## **No way to `changeBridgedToken()` back to an old btoken**
#### https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/tokenvault/ERC20Vault.sol#L181
<br>

## Summary
Owner can call [changeBridgedToken()](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/tokenvault/ERC20Vault.sol#L148) to change the bridged token from `btokenOld_` to `_btokenNew`. However, if later on he wants to change it back to `btokenOld_` (or map `btokenOld_` to another canonical `_ctoken`), there's no way to do so as the function call `changeBridgedToken()` will revert. This is because `btokenOld_` has been blacklisted now with no function existent which allows whitelisting.

## Vulnerability Details
Whenever `changeBridgedToken()` is called, it blacklists the old bridged token, `btokenOld_`. This coupled with the fact that there is no function inside the protocol which lets the owner remove a token from a blacklist, makes it impossible to ever call `changeBridgedToken()` again and set the new bridged token back as `btokenOld_` since the call will always revert on [L162](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/tokenvault/ERC20Vault.sol#L162):
```js
  File: contracts/tokenvault/ERC20Vault.sol

  144:              /// @notice Change bridged token.
  145:              /// @param _ctoken The canonical token.
  146:              /// @param _btokenNew The new bridged token address.
  147:              /// @return btokenOld_ The old bridged token address.
  148:              function changeBridgedToken(
  149:                  CanonicalERC20 calldata _ctoken,
  150:                  address _btokenNew
  151:              )
  152:                  external
  153:                  nonReentrant
  154:                  whenNotPaused
  155:                  onlyOwner
  156:                  returns (address btokenOld_)
  157:              {
  158:                  if (_btokenNew == address(0) || bridgedToCanonical[_btokenNew].addr != address(0)) {
  159:                      revert VAULT_INVALID_NEW_BTOKEN();
  160:                  }
  161:
  162: @--->            if (btokenBlacklist[_btokenNew]) revert VAULT_BTOKEN_BLACKLISTED();
  163:
  164:                  if (IBridgedERC20(_btokenNew).owner() != owner()) {
  165:                      revert VAULT_NOT_SAME_OWNER();
  166:                  }
  167:
  168:                  btokenOld_ = canonicalToBridged[_ctoken.chainId][_ctoken.addr];
  169:
  170:                  if (btokenOld_ != address(0)) {
  171:                      CanonicalERC20 memory ctoken = bridgedToCanonical[btokenOld_];
  172:
  173:                      // The ctoken must match the saved one.
  174:                      if (
  175:                          ctoken.decimals != _ctoken.decimals
  176:                              || keccak256(bytes(ctoken.symbol)) != keccak256(bytes(_ctoken.symbol))
  177:                              || keccak256(bytes(ctoken.name)) != keccak256(bytes(_ctoken.name))
  178:                      ) revert VAULT_CTOKEN_MISMATCH();
  179:
  180:                      delete bridgedToCanonical[btokenOld_]; 
  181: @--->                btokenBlacklist[btokenOld_] = true;
  182:
  183:                      // Start the migration
  184:                      IBridgedERC20(btokenOld_).changeMigrationStatus(_btokenNew, false);
  185:                      IBridgedERC20(_btokenNew).changeMigrationStatus(btokenOld_, true);
  186:                  }
```

## Tools Used
Manual review

## Recommended Mitigation Steps
Add a function with the `onlyOwner` modifier which allows whiltelisting of a token if the owner wishes so. Thus the owner can call this function prior to calling `changeBridgedToken()` and avoid a revert.

---

### <a id="m-02"></a>[M-02]
## **Banned address can still `_invokeMessageCall()` by calling `retryMessage()`**
#### https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L331-L332
<br>

## Summary & Impact
The function [banAddress()](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L101) can be used by the owner to ban or unban an address. As the natspec mentions:
```js
  File: contracts/bridge/Bridge.sol

  97:  @--->        /// @notice Ban or unban an address. A banned addresses will not be invoked upon
  98:  @--->        /// with message calls.
  99:               /// @param _addr The address to ban or unban.
  100:              /// @param _ban True if ban, false if unban.
  101:              function banAddress(
```
However, a user can bypass this & is able to invoke a message via [retryMessage()](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L331-L332) which has no check in place for banned addresses.

## Root Cause
The banned address check currently exists only within `processMessage()` and hence the check never occurs while retrying a message.

## Details
Consider the following flow of events:
- Alice calls `processMessage()` but the internal call to `_invokeMessageCall()` fails and the message status is set as `RETRIABLE` on [L285](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L285).
- Owner calls `banAddress()` to ban Alice's `message.to` address.
- Alice calls `retryMessage()`. Her message gets processed and a call to `_invokeMessageCall()` is made since the function never checks for banned addresses. This should not have happened. Instead, it should have been directly marked DONE and the `_message.value` refunded.

## Proof of Concept
Add the following test inside `protocol/test/bridge/Bridge.t.sol` and run via `forge test -vv --mt test_t0x1c_Bridge_retry_message_with_banned_address` to see it pass, even though it should have failed as per specs:
```js
    function test_t0x1c_Bridge_retry_message_with_banned_address() public {
        vm.startPrank(Alice);
        (IBridge.Message memory message, bytes memory proof) =
            setUpPredefinedSuccessfulProcessMessageCall();

        // etch bad receiver at the to address, so it fails.
        vm.etch(message.to, address(badReceiver).code);

        bytes32 msgHash = destChainBridge.hashMessage(message);

        destChainBridge.processMessage(message, proof);

        IBridge.Status status = destChainBridge.messageStatus(msgHash);

        assertEq(status == IBridge.Status.RETRIABLE, true);

        // ban `message.to`
        destChainBridge.banAddress(message.to, true);

        vm.stopPrank();

        vm.prank(message.destOwner);
        destChainBridge.retryMessage(message, true); 
        IBridge.Status postRetryStatus = destChainBridge.messageStatus(msgHash);
        assertEq(postRetryStatus == IBridge.Status.FAILED, true);  // @audit : should have been marked as DONE and refund issued
    }
```

## Tools Used
Foundry

## Recommended Mitigation Steps
- Add the banned address check inside `retryMessage()`
- Issue a refund when applicable

```diff
    function retryMessage(
        Message calldata _message,
        bool _isLastAttempt
    )
        external
        nonReentrant
        whenNotPaused
        sameChain(_message.destChainId)
    {
        // If the gasLimit is set to 0 or isLastAttempt is true, the caller must
        // be the message.destOwner.
        if (_message.gasLimit == 0 || _isLastAttempt) {
            if (msg.sender != _message.destOwner) revert B_PERMISSION_DENIED();
        }

        bytes32 msgHash = hashMessage(_message);
        if (messageStatus[msgHash] != Status.RETRIABLE) {
            revert B_NON_RETRIABLE();
        }

+     if (!addressBanned[_message.to]) {
        // Attempt to invoke the messageCall.
        if (_invokeMessageCall(_message, msgHash, gasleft())) {
            _updateMessageStatus(msgHash, Status.DONE);
        } else if (_isLastAttempt) {
            _updateMessageStatus(msgHash, Status.FAILED);
        }
+     } else {
+       uint256 refundAmount = _message.value;
+       address refundTo = _message.refundTo == address(0) ? _message.destOwner : _message.refundTo;
+       _updateMessageStatus(msgHash, Status.DONE);
+       refundTo.sendEther(refundAmount);
+     }
      emit MessageRetried(msgHash);
    }
```

---

### <a id="m-03"></a>[M-03]
## **No incentive for message non-owners to retryMessage()**
#### https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L310-L337
<br>

## Description
The function `processMessage()` [provides a reward to the msg.sender](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L298) if they are not the `refundTo` address. This incentivizes them to call the function.
[If this call fails while calling _invokeMessageCall()](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L285), then the protocol allows calling of [retryMessage()](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L310-L337) later on. As the comments [here](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L319-L320) indicate, the caller could be the `message.destOwner` or someone else. 
```js
        // If the gasLimit is set to 0 or isLastAttempt is true, the caller must
        // be the message.destOwner.
```

However unlike `processMessage()`, there is absolutely no incentive for someone to call `retryMessage()` since there is no logic of a reward anywhere inside it.

## Impact
The only ones who will be interested in calling `retryMessage()` would be `message.destOwner` or the `message.to`; no one else. This potentially slows down the message processing rate of the system.

## Tools Used
Manual review

## Recommended Mitigation Steps
One of these two options can be implemented based on the protocol's discretion. I personally prefer the first one:
- Option1: Do not give a reward if message goes to `RETRIABLE` state after `processMessage()` is called. Give it only if message reached `DONE` state. Then, in case `retryMessage()` is called, distribute the reward at that time.
- Option2: Continue giving out a reward in `processMessage()` and make no changes there. However to incentivize `retryMessage()` calls, add a field `_message.retryFee` which is tapped into only in case of a retry scenario, else refunded to the specified `refundTo` address. Note that this architecture opens up an attack vector mentioned in the bug report titled _"Malicious caller of `processMessage()` can pocket the fee while forcing `excessivelySafeCall()` to fail"_.

---

### <a id="m-04"></a>[M-04]
## **Malicious caller of `processMessage()` can pocket the fee while forcing `excessivelySafeCall()` to fail**
#### https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L280-L298
<br>

## Summary
The logic inside function `processMessage()` [provides a reward to the msg.sender](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L298) if they are not the `refundTo` address. However this reward or `_message.fee` is awarded even if the `_invokeMessageCall()` on [L282](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L282) fails and the message goes into a `RETRIABLE` state. In the retriable state, it has to be called by someone again and the current `msg.sender` has no obligation to be the one to call it.

This logic can be gamed by a malicious user using the **63/64th rule specified in** [EIP-150](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-150.md).

```js
  File: contracts/bridge/Bridge.sol

  278:                          // Use the specified message gas limit if called by the owner, else
  279:                          // use remaining gas
  280: @--->                    uint256 gasLimit = msg.sender == _message.destOwner ? gasleft() : _message.gasLimit;
  281:
  282: @--->                    if (_invokeMessageCall(_message, msgHash, gasLimit)) {
  283:                              _updateMessageStatus(msgHash, Status.DONE);
  284:                          } else {
  285: @--->                        _updateMessageStatus(msgHash, Status.RETRIABLE);
  286:                          }
  287:                      }
  288:
  289:                      // Determine the refund recipient
  290:                      address refundTo =
  291:                          _message.refundTo == address(0) ? _message.destOwner : _message.refundTo;
  292:
  293:                      // Refund the processing fee
  294:                      if (msg.sender == refundTo) {
  295:                          refundTo.sendEther(_message.fee + refundAmount);
  296:                      } else {
  297:                          // If sender is another address, reward it and refund the rest
  298: @--->                    msg.sender.sendEther(_message.fee);
  299:                          refundTo.sendEther(refundAmount);
  300:                      }
```

## Description
The `_invokeMessageCall()` on [L282](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L282) internally calls `excessivelySafeCall()` on [L497](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L497). When `excessivelySafeCall()` makes an external call, only 63/64th of the gas is used by it. Thus the following scenario can happen:
- Malicious user notices that [L285-L307](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L285-L307) uses approx 165_000 gas.

- He also notices that [L226-L280](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L226-L280) uses approx 94_000 gas.

- He calculates that he must provide approximately a minimum of `94000 + (64 * 165000) = 10_654_000` gas so that the function execution does not revert anywhere.

- Meanwhile, a message owner has message which has a `_message.gasLimit` of 11_000_000. This is so because the `receive()` function of the contract receiving ether is expected to consume gas in this range due to its internal calls & transactions. The owner expects at least 10_800_000 of gas would be used up and hence has provided some extra buffer.

- Note that **any message** that has a processing requirement of greater than `63 * 165000 = 10_395_000` gas can now become a target of the malicious user.

- Malicious user now calls `processMessage()` with a specific gas figure. Let's use an example figure of `{gas: 10_897_060}`. This means only `63/64 * (10897060 - 94000) = 10_634_262` is forwarded to `excessivelySafeCall()` and `1/64 * (10897060 - 94000) = 168_797` will be kept back which is enough for executing the remaining lines of code L285-L307. Note that since `(10897060 - 94000) = 10_803_060` which is less than the message owner's provided `_message.gasLimit` of `11_000_000`, what actually gets considered is only `10_803_060`.

- The external call reverts inside `receive()` due to out of gas error (since 10_634_262 < 10_800_000) and hence `_success` is set to `false` on [L44](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/thirdparty/nomad-xyz/ExcessivelySafeCall.sol#L44).

- The remaining L285-L307 are executed and the malicious user receives his reward.

- The message goes into `RETRIABLE` state now and someone will need to call `retryMessage()` later on. 
A different bug report titled **_"No incentive for message non-owners to retryMessage()"_** has also been raised which highlights the incentivization scheme of `retryMessage()`.

## Impact
- Protocol can be gamed by a user to gain rewards while additionaly saving money by providing the least possible gas. 

- There is no incentive for any external user now to ever provide more than `{gas: 10_897_060}` (approx figure).

## Proof of Concept
Apply the following patch to add the test inside `protocol/test/bridge/Bridge.t.sol` and run via `forge test -vv --mt test_t0x1c_gasManipulation` to see it pass:
```diff
diff --git a/packages/protocol/test/bridge/Bridge.t.sol b/packages/protocol/test/bridge/Bridge.t.sol
index 6b7dca6..ce77ce2 100644
--- a/packages/protocol/test/bridge/Bridge.t.sol
+++ b/packages/protocol/test/bridge/Bridge.t.sol
@@ -1,11 +1,19 @@
 // SPDX-License-Identifier: MIT
 pragma solidity 0.8.24;
 
 import "../TaikoTest.sol";
 
+contract ToContract {
+    receive() external payable {
+        uint someVar;
+        for(uint loop; loop < 86_990; ++loop)
+            someVar += 1e18;
+    }
+}
+
 // A contract which is not our ErcXXXTokenVault
 // Which in such case, the sent funds are still recoverable, but not via the
 // onMessageRecall() but Bridge will send it back
 contract UntrustedSendMessageRelayer {
     function sendMessage(
         address bridge,
@@ -115,12 +123,71 @@ contract BridgeTest is TaikoTest {
         register(address(addressManager), "bridge", address(destChainBridge), destChainId);
 
         register(address(addressManager), "taiko", address(uint160(123)), destChainId);
         vm.stopPrank();
     }
 
+    
+    function test_t0x1c_gasManipulation() public {
+        //**************** SETUP **********************
+        ToContract toContract = new ToContract();
+        IBridge.Message memory message = IBridge.Message({
+            id: 0,
+            from: address(bridge),
+            srcChainId: uint64(block.chainid),
+            destChainId: destChainId,
+            srcOwner: Alice,
+            destOwner: Alice,
+            to: address(toContract),
+            refundTo: Alice,
+            value: 1000,
+            fee: 1000,
+            gasLimit: 11_000_000,
+            data: "",
+            memo: ""
+        });
+        // Mocking proof - but obviously it needs to be created in prod
+        // corresponding to the message
+        bytes memory proof = hex"00";
+
+        bytes32 msgHash = destChainBridge.hashMessage(message);
+
+        vm.chainId(destChainId);
+        skip(13 hours);
+        assertEq(destChainBridge.messageStatus(msgHash) == IBridge.Status.NEW, true);
+        uint256 carolInitialBalance = Carol.balance;
+
+        uint256 snapshot = vm.snapshot();
+        //**************** SETUP ENDS **********************
+
+
+
+        //**************** NORMAL USER **********************
+        console.log("\n**************** Normal User ****************");
+        vm.prank(Carol, Carol);
+        destChainBridge.processMessage(message, proof);
+
+        assertEq(destChainBridge.messageStatus(msgHash) == IBridge.Status.DONE, true);
+        assertEq(Carol.balance, carolInitialBalance + 1000, "Carol balance mismatch");
+        if (destChainBridge.messageStatus(msgHash) == IBridge.Status.DONE)
+            console.log("message status = DONE");
+
+
+
+        //**************** MALICIOUS USER **********************
+        vm.revertTo(snapshot);
+        console.log("\n**************** Malicious User ****************");
+        vm.prank(Carol, Carol);
+        destChainBridge.processMessage{gas: 10_897_060}(message, proof); // @audit-info : specify gas to force failure of excessively safe external call
+
+        assertEq(destChainBridge.messageStatus(msgHash) == IBridge.Status.RETRIABLE, true); // @audit : message now in RETRIABLE state. Carol receives the fee.
+        assertEq(Carol.balance, carolInitialBalance + 1000, "Carol balance mismatched");
+        if (destChainBridge.messageStatus(msgHash) == IBridge.Status.RETRIABLE)
+            console.log("message status = RETRIABLE");
+    }
+
     function test_Bridge_send_ether_to_to_with_value() public {
         IBridge.Message memory message = IBridge.Message({
             id: 0,
             from: address(bridge),
             srcChainId: uint64(block.chainid),
             destChainId: destChainId,
```

## Tools Used
Foundry

## Recommended Mitigation Steps
Reward the `msg.sender` (provided it's a _non-refundTo_ address) with `_message.fee` only if `_invokeMessageCall()` returns `true`. Additionally, it is advisable to release this withheld reward after a successful `retryMessage()` to that function's caller.

---

### <a id="m-05"></a>[M-05]
## **Incorrect calculations for cooldownWindow and provingWindow could cause a state transition to spend more than expected time in these windows**
#### https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/L1/libs/LibProving.sol#L414-L415
#### https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/L1/libs/LibVerifying.sol#L152-L153
<br>

## Summary
The protocol supports [pauseProving()](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/L1/TaikoL1.sol#L111) and [unpausing](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/L1/TaikoL1.sol#L124) mechanisms. While calculating  `cooldownWindow` and `provingWindow`, it calculates the timespan since the last unpause in the following manner. The protocol however fails to account for already elapsed time the state transition may have spent in these windows before unpause happened. As a result, these window time spans are not adhered to:

```js
  File: contracts/L1/libs/LibVerifying.sol

  85:               function verifyBlocks(
  86:                   TaikoData.State storage _state,
  87:                   TaikoData.Config memory _config,
  88:                   IAddressResolver _resolver,
  89:                   uint64 _maxBlocksToVerify
  90:               )
  91:                   internal
  92:               {
  93:                   if (_maxBlocksToVerify == 0) {
  94:                       return;
  95:                   }
  96:           
  97:                   // Retrieve the latest verified block and the associated transition used
  98:                   // for its verification.
  99:                   TaikoData.SlotB memory b = _state.slotB;
  
                          ....
                          ....

  138:          
  139:                          // A transition with the correct `parentHash` has been located.
  140:                          TaikoData.TransitionState storage ts = _state.transitions[slot][tid];
  141:          
  142:                          // It's not possible to verify this block if either the
  143:                          // transition is contested and awaiting higher-tier proof or if
  144:                          // the transition is still within its cooldown period.
  145:                          if (ts.contester != address(0)) {
  146:                              break;
  147:                          } else {
  148:                              if (tierProvider == address(0)) {
  149:                                  tierProvider = _resolver.resolve("tier_provider", false);
  150:                              }
  151:                              if (
  152: @--->                            uint256(ITierProvider(tierProvider).getTier(ts.tier).cooldownWindow) * 60
  153: @--->                                + uint256(ts.timestamp).max(_state.slotB.lastUnpausedAt) > block.timestamp
  154:                              ) {
                                    
                                    ....
                                    ....
```

and 

```js
  File: contracts/L1/libs/LibProving.sol

  401:              function _checkProverPermission(
  402:                  TaikoData.State storage _state,
  403:                  TaikoData.Block storage _blk,
  404:                  TaikoData.TransitionState storage _ts,
  405:                  uint32 _tid,
  406:                  ITierProvider.Tier memory _tier
  407:              )
  408:                  private
  409:                  view
  410:              {
  411:                  // The highest tier proof can always submit new proofs
  412:                  if (_tier.contestBond == 0) return;
  413:          
  414: @--->            bool inProvingWindow = uint256(_ts.timestamp).max(_state.slotB.lastUnpausedAt)
  415: @--->                + _tier.provingWindow * 60 >= block.timestamp;
  416:                  bool isAssignedPover = msg.sender == _blk.assignedProver;
  417:          
  418:                  // The assigned prover can only submit the very first transition.
  419:                  if (_tid == 1 && _ts.tier == 0 && inProvingWindow) {
  420:                      if (!isAssignedPover) revert L1_NOT_ASSIGNED_PROVER();
  421:                  } else {
  422:                      // Disallow the same address to prove the block so that we can detect that the
  423:                      // assigned prover should not receive his liveness bond back
  424:                      if (isAssignedPover) revert L1_ASSIGNED_PROVER_NOT_ALLOWED();
  425:                  }
  426:              }
```

## Description
Mutliple scenarios can play out here:
- **_Scenario-1_** : (Cooldown window period violation)
    - Assumption: `cooldownPeriod = 60 minutes`.
    - User submits a state transition at timestamp `t`. Cooldown period should end after 60 minutes of unpaused state.
    - Owner calls `pauseProving(true)` at `t+55`. This means user's state transition has already spent 55 minutes in cooldown period. No other user has called `proveBlock()` yet to contest this state transition.
    - Owner calls `pauseProving(false)` at `t+120`.
    - User can't call `verifyBlocks()` at `t+125` even though 60 minutes have elapsed. The cooldown window is now calculated by the protocol starting from `lastUnpausedAt` which is `t+120`. Hence user needs to wait till `t+180` and anyone can contest till `t+180`.

<br>

- **_Scenario-2_** : (Proving window period violation)
    - Assumption: `provingWindow = 60 minutes`.
    - The protocol dictates that the assigned prover can only submit the very first transition while we are still within the proving window i.e. `inProvingWindow = true`.
    - Suppose initial submission happened at `t`. Proving window should end after 60 minutes of unpaused state.
    - Owner pauses at `t+55`.
    - Owner unpauses at `t+120`. This is stored as the `lastUnpausedAt`. 
    - Protocol calculates proving window from `lastUnpausedAt`. Which means it now ends at `t+180` instead of expected `t+125`. Assigned prover just got 55 additional minutes. Conversely, a non-assigned prover is not able to prove it till `t+180`.

<br>

- **_Scenario-3_** : (Multiple pauses and unpauses cause a reset each time)
    - In both the above scenarios, the wait period is "_reset_" to start counting from `lastUnpausedAt` whenever an unpause happens. So in the worst case, one may imagine a situation where multiple pause + unpauses are happening, each one pushing the wait time further & further.
    - User submits at `t`. Wait window should end after 60 minutes of unpaused state.
    - Owner pauses at `t+55`.
    - Owner unpauses at `t+120`.
    - Owner pauses at `t+175`.
    - Owner unpauses at `t+240`. _(And so on, repeatedly)_
    - The new end of window is now calculated as `t+300`.
    
<br>

**_It's important to note_** that in the above scenarios there's another variation possible (although it has a lesser probability to occur):
- If no action has been taken by any user even after the legitimate wait time is over, then occurence of a pause + unpause still pushes the wait times further into the future. Example:
    - User submits at `t`.
    - Cooldown period ends at `t+60`. No contestation happened.
    - User does not call `verifyBlocks()` immediately (could be due to network issues, or simply forgot for some time).
    - At `t+62` owner pauses.
    - At `t+100` owner unpauses. The cooldown period is "reset" now. User needs to wait till `t+160` before calling `verifyBlocks()`. His 2-minute delay cost him a wait of another 60 minutes.

## Tools Used
Manual review

## Recommended Mitigation Steps
Introduce a new variable which keeps track of how much time the state transition has already spent in the wait window. Then a change can be made which could be along the following lines:

```diff
  414:                  bool inProvingWindow = uint256(_ts.timestamp).max(_state.slotB.lastUnpausedAt)
- 415:                      + _tier.provingWindow * 60 >= block.timestamp;
+ 415:                      + _tier.provingWindow * 60 - _ts.timeAlreadySpentInWaitWindow >= block.timestamp;
```

However before this code change, one also needs to make sure that `_ts.timeAlreadySpentInWaitWindow` is still less than `_tier.provingWindow * 60` (or whichever wait window duration we are tracking). This will help in avoiding the scenario mentioned after Scenario-3 in the "_It's important to note_" section.


---

### <a id="m-06"></a>[M-06]
## **Invocation delays are not honoured when protocol unpauses**
#### https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/common/EssentialContract.sol#L78
#### https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L258
<br>

## Summary
**_Context:_** The protocol has `pause()` and [unpause()](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/common/EssentialContract.sol#L78) functions inside `EssentialContract.sol` which are tracked throughout the protocol via the modifiers [whenPaused](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/common/EssentialContract.sol#L53) and [whenNotPaused](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/common/EssentialContract.sol#L58).

**_Issue:_** Various delays and time lapses throughout the protocol ignore the effect of such pauses. The example in focus being that of `processMessage()` which does not take into account the pause duration while [checking invocationDelay and invocationExtraDelay](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L233-L258). One impact of this is that it allows a non-preferred executor to front run a `preferredExecutor`, after an unpause.

```js
  File: contracts/bridge/Bridge.sol

  233: @--->            (uint256 invocationDelay, uint256 invocationExtraDelay) = getInvocationDelays();
  234:          
  235:                  if (!isMessageProven) {
  236:                      if (!_proveSignalReceived(signalService, msgHash, _message.srcChainId, _proof)) {
  237:                          revert B_NOT_RECEIVED();
  238:                      }
  239:          
  240:                      receivedAt = uint64(block.timestamp);
  241:          
  242:                      if (invocationDelay != 0) {
  243:                          proofReceipt[msgHash] = ProofReceipt({
  244:                              receivedAt: receivedAt,
  245:                              preferredExecutor: _message.gasLimit == 0 ? _message.destOwner : msg.sender
  246:                          });
  247:                      }
  248:                  }
  249:          
  250:                  if (invocationDelay != 0 && msg.sender != proofReceipt[msgHash].preferredExecutor) {
  251:                      // If msg.sender is not the one that proved the message, then there
  252:                      // is an extra delay.
  253:                      unchecked {
  254:                          invocationDelay += invocationExtraDelay;
  255:                      }
  256:                  }
  257:          
  258: @--->            if (block.timestamp >= invocationDelay + receivedAt) {
```

## Description & Impact
Consider the following flow:
- Assumption: `invocationDelay = 60 minutes` and `invocationExtraDelay = 30 minutes`.
- A message is sent.
- First call to `processMessage()` occurred at `t` where it was proven by Bob i.e. its `receivedAt = t`. Bob is marked as the `preferredExecutor`.
- Preferred executor should be able to call `processMessage()` at `t+60` while a non-preferred executor should be able to call it only at `t+90` due to the code logic on [L250](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L250).
- At `t+55`, protocol is paused.
- At `t+100`, protocol is unpaused.
- **_Impact:_** The 30-minute time window advantage which the preferred executor had over the non-preferred one is now lost to him. [L258](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L258) now considers the invocation delays to have passed and hence the non-preferred executor can immediately call `processMessage()` by front-running Bob and hence pocketing the reward of `message.fee` on [L98](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L298).
```js
  File: contracts/bridge/Bridge.sol

  293:                      // Refund the processing fee
  294:                      if (msg.sender == refundTo) {
  295:                          refundTo.sendEther(_message.fee + refundAmount);
  296:                      } else {
  297:                          // If sender is another address, reward it and refund the rest
  298: @--->                    msg.sender.sendEther(_message.fee);
  299:                          refundTo.sendEther(refundAmount);
  300:                      }
```

<br>

Similar behaviour where the paused time is ignored by the protocol can be witnessed in:
- [recallMessage()](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L187-L189) which similarly uses `invocationDelay`. However, no `invocationExtraDelay` is used there.
- [TimelockTokenPool.sol](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/team/TimelockTokenPool.sol) for:
```js
        // If non-zero, indicates the start time for the recipient to receive
        // tokens, subject to an unlocking schedule.
        uint64 grantStart;
        // If non-zero, indicates the time after which the token to be received
        // will be actually non-zero
        uint64 grantCliff;
        // If non-zero, specifies the total seconds required for the recipient
        // to fully own all granted tokens.
        uint32 grantPeriod;
        // If non-zero, indicates the start time for the recipient to unlock
        // tokens.
        uint64 unlockStart;
        // If non-zero, indicates the time after which the unlock will be
        // actually non-zero
        uint64 unlockCliff;
        // If non-zero, specifies the total seconds required for the recipient
        // to fully unlock all owned tokens.
        uint32 unlockPeriod;
```

- [TaikoData.sol](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/L1/TaikoData.sol) for:
```js
        // The max period in seconds that a blob can be reused for DA.
        uint24 blobExpiry;
```

## Tools Used
Manual review

## Recommended Mitigation Steps
Introduce a new variable which keeps track of how much time has already been spent in the valid wait window before a pause happened. Also track the last unpause timestamp (similar to how it is done in [pauseProving()](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/L1/TaikoL1.sol#L111) and [unpausing](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/L1/TaikoL1.sol#L124) mechanisms). 
Also refer my other recommendation under the report titled: _"Incorrect calculations for cooldownWindow and provingWindow could cause a state transition to spend more than expected time in these windows"_. That will help fix the issue without any further leaks.

---

### <a id="m-07"></a>[M-07]
## **Proposers would choose to avoid higher tier by exploiting non-randomness of parameter used in getMinTier()**
#### https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/L1/tiers/MainnetTierProvider.sol#L66-L70
<br>

## Description
The issue exists for both `MainnetTierProvider.sol` and `TestnetTierProvider.sol`. For this report, we shall concentrate only on describing it via `MainnetTierProvider.sol`.

The proving tier is chosen by the [getMinTier()](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/L1/tiers/MainnetTierProvider.sol#L66-L70) function which accepts a `_rand` param.
```js
  File: contracts/L1/tiers/MainnetTierProvider.sol

  66:               function getMinTier(uint256 _rand) public pure override returns (uint16) {
  67:                   // 0.1% require SGX + ZKVM; all others require SGX
  68: @--->             if (_rand % 1000 == 0) return LibTiers.TIER_SGX_ZKVM;
  69:                   else return LibTiers.TIER_SGX;
  70:               }
```

If `_rand % 1000 == 0`, a costlier tier `TIER_SGX_ZKVM` is used instead of the cheaper `TIER_SGX`. The `_rand` param is passed in the form of `meta_.difficulty` [which is calculated inside](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/L1/libs/LibProposing.sol#L199-L209) `proposeBlock()`:
```js
  File: contracts/L1/libs/LibProposing.sol

  199:                  // Following the Merge, the L1 mixHash incorporates the
  200:                  // prevrandao value from the beacon chain. Given the possibility
  201:                  // of multiple Taiko blocks being proposed within a single
  202:                  // Ethereum block, we choose to introduce a salt to this random
  203:                  // number as the L2 mixHash.
  204: @--->            meta_.difficulty = keccak256(abi.encodePacked(block.prevrandao, b.numBlocks, block.number));
  205:          
  206:                  // Use the difficulty as a random number
  207:                  meta_.minTier = ITierProvider(_resolver.resolve("tier_provider", false)).getMinTier(
  208: @--->                uint256(meta_.difficulty)
  209:                  );
```

As can be seen, all the parameters used in L204 to calculate `meta_.difficulty` can be known in advance and hence a proposer can choose not to propose when `meta_.difficulty` modulus 1000 equals zero, because in such cases it will cost him more to afford the proof (sgx + zk proof in this case).

## Impact
Since the proposer will now wait for the next or any future block to call `proposeBlock()` instead of the current costlier one, **transactions will now take longer to finalilze**.

If `_rand` were truly random, it would have been an even playing field in all situations as the proposer wouldn't be able to pick & choose since he won't know in advance which tier he might get. We would then truly have:
```js
  67:                   // 0.1% require SGX + ZKVM; all others require SGX
```

## Tools Used
Manual review

## Recommended Mitigation Steps
Consider using VRF like solutions to make `_rand` truly random.

---

### <a id="m-08"></a>[M-08]
## **Griefer can front-run processMessage() with a call to recallMessage() due to incorrect invocationDelay implementation**
#### https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L187-L189
<br>

## Summary
The [processMessage()](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L250-L256) function has the logic of having an `invocationExtraDelay` which gives the `preferredExecutor` an exclusive initial time window to call the function between `invocationDelay` and `invocationExtraDelay`.
A similar logic however is missing in [recallMessage()](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L187-L189) which means that at timestamp of `invocationDelay`, a griefer can front-run the `processMessage()` call by calling `recallMessage()` to change message's status to `RECALLED`. The `processMessage()` hence would revert due to the check on [L227](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L227) which expected the message status to be `NEW`.

## Description & Impact
Consider the following flow:
- Assumption: `invocationDelay = 60 minutes` and `invocationExtraDelay = 30 minutes`.
- A message is sent.
- First call to `processMessage()` occurred at `t` where it was proven by Bob i.e. its `receivedAt = t`. Bob is marked as the `preferredExecutor`.
- Bob should be able to call `processMessage()` at `t+60` while a non-preferred executor should be able to call it only at `t+90` due to the code logic on [L250](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L250).
- Bob tries to call `processMessage()` at `t+60` but is front-run by the griefer, Carol who calls `recallMessage()`. Since the condition on [L189](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/bridge/Bridge.sol#L189) is satisfied, the message is recalled.
- Bob transaction eventually goes through but reverts as the message status isn't `NEW` anymore.

## Tools Used
Manual review

## Recommended Mitigation Steps
Just like `processMessage()`, add the functionality of `invocationExtraDelay` inside `recallMessage()`. Only the `preferredExecutor` should be allowed to recall between `invocationDelay` and `invocationExtraDelay`. Non-preferred actors should be allowed to recall only after `receivedAt + invocationDelay + invocationExtraDelay` has passed.

---

### <a id="m-09"></a>[M-09]
## **Prover loses funds even if proven right after multiple contests**
#### https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/L1/libs/LibProving.sol#L384
<br>

## Description & Impact
Consider the following flow:
- Assumption: Tier levels from low to high are `TIER_OPTIMISTIC` --> `TIER_SGX` --> `TIER_GUARDIAN` (assuming this so that it's in line with the [code logic inside TaikoL1LibProvingWithTiers.t.sol](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/test/L1/TaikoL1LibProvingWithTiers.t.sol#L40-L44) and our PoC matches).
- Alice proposes a block with Bob as the prover in tier `TIER_OPTIMISTIC`. 
- The correct proof entails `blockHash = blockHash` and `stateRoot = stateRoot`.
- Bob calls `proveBlock()` with above params.
- Carol contests this by calling `proveBlock()` with `blockHash = stateRoot` and `stateRoot = stateRoot`.
- At the next higher tier `TIER_SGX`, Carol calls `proveBlock()` to accept the proof and is rewarded.
- Bob loses his bond deposit.
- Alice immediately contests Carol's proof by calling `proveBlock()` with `blockHash = blockHash` and `stateRoot = stateRoot`.
- David proves Alice right at the next higher tier of `TIER_GUARDIAN`. Carol loses her bond deposit.
- Hence eventually, it turns out Bob was right all along but has still endured a loss of funds.

## Proof of Concept
Add the following helper function and test case inside `protocol/test/L1/TaikoL1LibProvingWithTiers.t.sol` and run via `forge test -vv --mt test_t0x1c_L1_MultipleContests` to see the subsequent output:
```js
    function proveHigherTierProof(
        address caller,
        TaikoData.BlockMetadata memory meta,
        bytes32 parentHash,
        bytes32 stateRoot,
        bytes32 blockHash,
        uint16 minTier
    )
        internal
    {
        uint16 tierToProveWith;
        if (minTier == LibTiers.TIER_OPTIMISTIC) {
            tierToProveWith = LibTiers.TIER_SGX;
        } else if (minTier == LibTiers.TIER_SGX) {
            tierToProveWith = LibTiers.TIER_GUARDIAN;
        } 
        proveBlock(caller, caller, meta, parentHash, blockHash, stateRoot, tierToProveWith, "");
    }
    
    function test_t0x1c_L1_MultipleContests() external {
        giveEthAndTko(Alice, 1e8 ether, 1000 ether);
        giveEthAndTko(Bob, 1e8 ether, 100 ether);
        giveEthAndTko(Carol, 1e8 ether, 1000 ether);
        giveEthAndTko(David, 1e8 ether, 1000 ether);

        console2.log("\nBefore Test");
        emit log_named_decimal_uint("Bob balance", tko.balanceOf(Bob), 18);

        bytes32 parentHash = GENESIS_BLOCK_HASH;
        uint256 blockId = 1;

        (TaikoData.BlockMetadata memory meta,) = proposeBlock(Alice, Bob, 1_000_000, 1024);
        mine(1);

        bytes32 blockHash = bytes32(1e10 + blockId);
        bytes32 stateRoot = bytes32(1e9 + blockId);
        
        uint16 minTier = meta.minTier; // TIER_OPTIMISTIC
        proveBlock(Bob, Bob, meta, parentHash, blockHash, stateRoot, minTier, "");

        // Carol contests
        proveBlock(Carol, Carol, meta, parentHash, stateRoot, stateRoot, minTier, "");
        vm.roll(block.number + 15 * 12);
        vm.warp(block.timestamp + tierProvider().getTier(minTier).cooldownWindow * 60 + 1);

        // Cannot verify block because it is contested..
        verifyBlock(Carol, 1);

        proveHigherTierProof(Carol, meta, parentHash, stateRoot, stateRoot, minTier);

        // Alice contests, now at the higher tier
        proveBlock(Alice, Alice, meta, parentHash, blockHash, stateRoot, LibTiers.TIER_SGX, "");
        
        proveHigherTierProof(David, meta, parentHash, stateRoot, blockHash, LibTiers.TIER_SGX);

        // Now can verify
        vm.warp(
            block.timestamp + tierProvider().getTier(LibTiers.TIER_GUARDIAN).cooldownWindow * 60
                + 1
        );
        verifyBlock(David, 1);
        
        console2.log("\nAfter Test");
        emit log_named_decimal_uint("Bob balance", tko.balanceOf(Bob), 18);
    }
```

Output:
```  
Before Test
  Bob balance: 100000000.000000000000000000
  
After Test
  Bob balance: 99999749.000000000000000000

Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 34.36ms
```

## Tools Used
Foundry

## Recommended Mitigation Steps
We would need to track and store the user proofs submitted before any contests and ensure that if the final verification matches any of the previous rejected ones, then the validityBond ought to be refunded to that user.

---

### <a id="m-10"></a>[M-10]
## **Protocol does not check inside GuardianProver::approve() if all the guardians are approving the same proof**
#### https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/L1/provers/GuardianProver.sol#L33
<br>

## Summary
The [natspec for GuardianProver::approve() says the following on L33](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/L1/provers/GuardianProver.sol#L33) but the **_"sent the same proof"_** clause is never really checked inside the function:
```js
  File: contracts/L1/provers/GuardianProver.sol

  29:               /// @dev Called by guardians to approve a guardian proof
  30:               /// @param _meta The block's metadata.
  31:               /// @param _tran The valid transition.
  32:               /// @param _proof The tier proof.
  33:  @--->        /// @return approved_ If the minimum number of participants sent the same proof, and proving
  34:               /// transaction is fired away returns true, false otherwise.
  35:               function approve(
  36:                   TaikoData.BlockMetadata calldata _meta,
  37:                   TaikoData.Transition calldata _tran,
  38:                   TaikoData.TierProof calldata _proof
  39:               )
  40:                   external
  41:                   whenNotPaused
  42:                   nonReentrant
  43:                   returns (bool approved_)
  44:               {
  45:                   if (_proof.tier != LibTiers.TIER_GUARDIAN) revert INVALID_PROOF();
  46:                   bytes32 hash = keccak256(abi.encode(_meta, _tran));
  47:                   approved_ = approve(_meta.id, hash);
  48:           
  49:                   if (approved_) {
  50:                       deleteApproval(hash);
  51:  @--->                ITaikoL1(resolve("taiko", false)).proveBlock(_meta.id, abi.encode(_meta, _tran, _proof));
  52:                   }
  53:           
  54:                   emit GuardianApproval(msg.sender, _meta.id, _tran.blockHash, approved_);
  55:               }
```

## Description & Impact
- Assume there are a total of 9 guardians. Hence approvals needed for `approved_ = true` is `(9 + 1) / 2 = 5`.
- Guardian1 to Guardian3 pass `proofX` while calling `approve()`.
- Guardian4 passes `proofY`.
- Guardian5 passes `proofX`. Since the `hash` on L46 is calculated without involving the proof, L47 will return `true` at the 5th guardian's call even though Guardian4 had used `proofY`.
- `proveBlock()` is called with `_proof` equalling `proofX`.
- Note that whatever `_proof` Guradian5 uses while calling `approve()` will be used to call `proveBlock()` on L51, irrespective of whatever the previous guardians passed.

Thus the guardian proof is approved even when in reality the required votes have not been achieved.

## Tools Used
Manual review

## Recommended Mitigation Steps
Compare the `_proof.data` to make sure approval for the same proof is being provided by all the guardians.

---

### <a id="m-11"></a>[M-11]
## **Incorrect implementation of "precision is maintained at the token level rather than the wei level" leads to division before multiplication & hence fee loss**
#### https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/team/TimelockTokenPool.sol#L195-L198
<br>

## Summary
This report **does not** contest the [protocol's design decision](https://github.com/code-423n4/2024-03-taiko/blob/main/packages/protocol/contracts/team/TimelockTokenPool.sol#L195) which states that:
```js
  195: @--->            // Note: precision is maintained at the token level rather than the wei level, otherwise,
  196:                  // `costPaid` must be a uint256.
```

This report instead highlights the fact that the code implementation of the above design choice is flawed and causes precision loss due to division before multiplication and hence a lowered `costToWithdraw` for the user which in turn means fee loss for the protocol.

## Description
```js
  File: contracts/team/TimelockTokenPool.sol

  195:                  // Note: precision is maintained at the token level rather than the wei level, otherwise,
  196:                  // `costPaid` must be a uint256.
  197: @--->            uint128 _amountUnlocked = amountUnlocked / 1e18; // divide first
  198:                  costToWithdraw = _amountUnlocked * r.grant.costPerToken - r.costPaid;
```

Consider the following numbers:
- Assume that `costPerToken = 2 USD` i.e. each TKO (1e18) will need some USD stable worth `$2` to purhcase.
- If `amountUnlocked = 1.9e18` then the cost to withdraw should ideally be `$3.8`. Note that just storing the `costPerToken` at token level instead of wei level does not mean that the protocol intends to forego the cost applicable over the 'decimal' portion.
- The `costToWithdraw` calculation (assuming zero `costPaid` so far for simplicity):
    - Current flawed implementation _(division before multiplication)_:
        - `costToWithdraw = ( 1.9e18 / 1e18 ) * $2 = $2`
    - Recommended correct implementation _(division after multiplication)_:
        - `costToWithdraw = ( 1.9e18 * $2 ) / 1e18 = $3`

## Impact
User pays lower `costToWithdraw` by manipulating his withdrawal amount (**which can be done by choosing the withdrawal timestamp**) and causes loss of fee for the protocol.

## Tools Used
Manual review

## Recommended Mitigation Steps
```diff
  File: contracts/team/TimelockTokenPool.sol

  195:                  // Note: precision is maintained at the token level rather than the wei level, otherwise,
  196:                  // `costPaid` must be a uint256.
- 197:                  uint128 _amountUnlocked = amountUnlocked / 1e18; // divide first
- 198:                  costToWithdraw = _amountUnlocked * r.grant.costPerToken - r.costPaid;
+ 197:                  costToWithdraw = amountUnlocked * r.grant.costPerToken / 1e18 - r.costPaid;
```

---

