## QA Summary<a name="QA Summary">

### Low Risk Issues
| |Issue|Contexts|
|-|:-|:-:|
| [LOW&#x2011;1](#LOW-QA&#x2011;8) | Remove unused code | 1 |
| [LOW&#x2011;2](#LOW-QA&#x2011;12) | Upgrade OpenZeppelin Contract Dependency | 2 |
| [LOW&#x2011;3](#LOW-QA&#x2011;14) | Use SafeCast to safely downcast variables | 4 |

Total: 7 contexts over 3 issues

### Non-critical Issues
| |Issue|Contexts|
|-|:-|:-:|
| [NC&#x2011;1](#NC-QA&#x2011;4) | Condition will not revert when `block.timestamp` is `==` to the compared variable | 1 |
| [NC&#x2011;2](#NC-QA&#x2011;7) | Consistent usage of require vs custom error | 24 |
| [NC&#x2011;3](#NC-QA&#x2011;21) | Functions that alter state should emit events | 1 |
| [NC&#x2011;4](#NC-QA&#x2011;25) | Generate perfect code headers for better solidity code layout and readability | 3 |
| [NC&#x2011;5](#NC-QA&#x2011;46) | `override` function arguments that are unused should have the variable name removed or commented out to avoid compiler warnings | 8 |
| [NC&#x2011;6](#NC-QA&#x2011;59) | Zero as a function argument should have a descriptive meaning | 17 |

Total: 54 contexts over 6 issues

## Low Risk Issues

### <a href="#qa-summary">[LOW&#x2011;1]</a><a name="LOW&#x2011;1"> Remove unused code
This code is not used in the main project contract files, remove it or add event-emit
Code that is not in use, suggests that they should not be present and could potentially contain insecure functionalities.

No function was found calling the `_voteSucceeded` function.

#### <ins>Proof Of Concept</ins>

```solidity
File: SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol

function _voteSucceeded
```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L175



### <a href="#qa-summary">[LOW&#x2011;2]</a><a name="LOW&#x2011;2"> Upgrade OpenZeppelin Contract Dependency

An outdated OZ version is used (which has known vulnerabilities, see: https://github.com/OpenZeppelin/openzeppelin-contracts/security/advisories).
Release: https://github.com/OpenZeppelin/openzeppelin-contracts/releases/tag/v4.9.2

#### <ins>Proof Of Concept</ins>

Require dependencies to be at least version of 4.9.2 to include critical vulnerability patches.

```solidity
File: package.json

    "@openzeppelin/contracts": "4.7.3"
```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/package.json#L69

```solidity
File: package.json

    "@openzeppelin/contracts-upgradeable": "4.7.3"
```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/package.json#L70



#### <ins>Recommended Mitigation Steps</ins>

Update OpenZeppelin Contracts Usage in package.json and require at least version of 4.9.2 via `>=4.9.2`




### <a href="#qa-summary">[LOW&#x2011;3]</a><a name="LOW&#x2011;3"> Use SafeCast to safely downcast variables

Downcasting `int`/`uint`s in Solidity can be unsafe due to the potential for data loss and unintended behavior. When downcasting a larger integer type to a smaller one (e.g., `uint256` to `uint128`), the value may exceed the range of the target type, leading to truncation and loss of significant digits. This data loss can result in unexpected state changes, incorrect calculations, or other contract vulnerabilities, ultimately compromising the contracts functionality and reliability. To prevent these risks, developers should carefully consider the range of values their variables may hold and ensure that proper checks are in place to prevent out-of-range values before performing downcasting. Also consider using OZ SafeCast functionality.

#### <ins>Proof Of Concept</ins>

```solidity
File: SecurityCouncilMemberSyncAction.sol

119: return _get(uint160(securityCouncil));

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L119

```solidity
File: SecurityCouncilMemberSyncAction.sol

123: _set(uint160(securityCouncil), nonce);

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L123

```solidity
File: SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

216: topNomineesAddresses[i] = nominees[uint16(topNomineesPacked[i])];

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L216

```solidity
File: SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

263: return uint240(x);

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L263



## Non Critical Issues

### <a href="#qa-summary">[NC&#x2011;1]</a><a name="NC&#x2011;1"> Condition will not revert when `block.timestamp` is `==` to the compared variable

The condition does not revert when block.timestamp is equal to the compared `>` or `<` variable. For example, if there is a check for `block.timestamp > expireTime` then there should be a check for cases where `block.timestamp` is `==` to `expireTime`
 
#### <ins>Proof Of Concept</ins>

```solidity
File: SecurityCouncilNomineeElectionGovernor.sol

167: if (block.timestamp < thisElectionStartTs) {
```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L167





### <a href="#qa-summary">[NC&#x2011;2]</a><a name="NC&#x2011;2"> Consistent usage of require vs custom error

Consider using the same approach throughout the codebase to improve the consistency of the code.
Some codes were found to use `require` while some were found to use `revert`

#### <ins>Proof Of Concept</ins>


```solidity
File: SecurityCouncilNomineeElectionGovernor.sol

116: revert NotAContract(address(params.securityCouncilManager));
120: revert NotAContract(address(params.securityCouncilMemberElectionGovernor));
129: revert QuorumNumeratorTooLow(params.quorumNumeratorValue);
136: revert OnlyNomineeVetter();
146: revert ProposalNotSucceededState(state_);
293: revert ProposalNotSucceededState(state_);
152: revert ProposalNotInVettingPeriod(block.number, vettingDeadline);
168: revert CreateTooEarly(block.timestamp, thisElectionStartTs);
207: revert LastMemberElectionNotExecuted(prevProposalId);
222: revert AlreadyContender(msg.sender);
227: revert ProposalNotActive(state_);
237: revert AccountInOtherCohort(otherCohort(), msg.sender);
273: revert NomineeAlreadyExcluded(nominee);
276: revert NotNominee(nominee);
297: revert NomineeAlreadyAdded(account);
303: revert CompliantNomineeTargetHit(cnCount, cohortSize);
313: revert AccountInOtherCohort(otherCohort(), account);
334: revert ProposalInVettingPeriod(block.number, vettingDeadline);
340: revert InsufficientCompliantNomineeCount(cnCount, cohortSize);
350: revert ProposalIdMismatch(proposalId, memberElectionProposalId);
429: revert ProposeDisabled();
434: revert CastVoteDisabled();
444: revert CastVoteDisabled();
454: revert CastVoteDisabled();

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L116

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L120

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L129

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L136

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L146

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L293

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L152

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L168

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L207

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L222

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L227

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L237

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L273

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L276

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L297

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L303

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L313

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L334

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L340

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L350

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L429

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L434

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L444

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L454



### <a href="#qa-summary">[NC&#x2011;3]</a><a name="NC&#x2011;3"> Functions that alter state should emit events

#### <ins>Proof Of Concept</ins>


```solidity
File: KeyValueStore.sol

11: function set(uint256 key, uint256 value) external {
        store[computeKey(msg.sender, key)] = value;
    }
```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/execution-record/KeyValueStore.sol#L11




### <a href="#qa-summary">[NC&#x2011;4]</a><a name="NC&#x2011;4"> Generate perfect code headers for better solidity code layout and readability

It is recommended to use pre-made headers for Solidity code layout and readability:
https://github.com/transmissions11/headers

```solidity
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```


#### <ins>Proof Of Concept</ins>


```solidity
File: SecurityCouncilManager.sol

8: SecurityCouncilManager.sol

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L8

```solidity
File: SecurityCouncilMemberElectionGovernor.sol

8: SecurityCouncilMemberElectionGovernor.sol

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L8

```solidity
File: SecurityCouncilNomineeElectionGovernor.sol

7: SecurityCouncilNomineeElectionGovernor.sol

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L7



### <a href="#qa-summary">[NC&#x2011;5]</a><a name="NC&#x2011;5"> `override` function arguments that are unused should have the variable name removed or commented out to avoid compiler warnings

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: SecurityCouncilMemberElectionGovernor.sol

191: function castVote : uint8

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L191

```solidity
File: SecurityCouncilMemberElectionGovernor.sol

196: function castVoteWithReason : uint8
196: function castVoteWithReason : string calldata

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L196

```solidity
File: SecurityCouncilNomineeElectionGovernor.sol

433: function castVote : uint8

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L433

```solidity
File: SecurityCouncilNomineeElectionGovernor.sol

438: function castVoteWithReason : uint8
438: function castVoteWithReason : string calldata

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L438

```solidity
File: SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

267: function _quorumReached : uint256

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L267

```solidity
File: SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

273: function _voteSucceeded : uint256

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L273



</details>


### <a href="#qa-summary">[NC&#x2011;6]</a><a name="NC&#x2011;6"> Zero as a function argument should have a descriptive meaning

Consider using descriptive constants or an enum instead of passing zero directly on function calls, as that might be error-prone, to fully describe the caller's intention.

#### <ins>Proof Of Concept</ins>


<details>


```solidity
File: SecurityCouncilMemberSyncAction.sol

130: address(securityCouncil), 0, data, OpEnum.Operation.Call

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L130

```solidity
File: SecurityCouncilNomineeElectionGovernorTiming.sol

37: minute: 0,
56: minute: 0,
91: minute: 0,

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol#L37

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol#L56

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol#L91


</details>