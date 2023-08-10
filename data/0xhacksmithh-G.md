### [Gas-01] `constant` State Variable Should Always `value` Not `Expressions`
Constant state variable should always store end result, not expressions. 
```solidity
    bytes32 public constant COHORT_REPLACER_ROLE = keccak256("COHORT_REPLACER"); // @audit G= immutable and pre-determine value
    bytes32 public constant MEMBER_ADDER_ROLE = keccak256("MEMBER_ADDER");
    bytes32 public constant MEMBER_REPLACER_ROLE = keccak256("MEMBER_REPLACER");
    bytes32 public constant MEMBER_ROTATOR_ROLE = keccak256("MEMBER_ROTATOR");
    bytes32 public constant MEMBER_REMOVER_ROLE = keccak256("MEMBER_REMOVER");
```
*Instances(5)*
```
File: security-council-mgmt/SecurityCouncilManager.sol#L4-L14
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L79-L83
```

### [Gas-02] Variables Those Store `keccak256()` Result, Should Be `immutable`
```solidity
    bytes32 public constant COHORT_REPLACER_ROLE = keccak256("COHORT_REPLACER"); // @audit G= immutable and pre-determine value
    bytes32 public constant MEMBER_ADDER_ROLE = keccak256("MEMBER_ADDER");
    bytes32 public constant MEMBER_REPLACER_ROLE = keccak256("MEMBER_REPLACER");
    bytes32 public constant MEMBER_ROTATOR_ROLE = keccak256("MEMBER_ROTATOR");
    bytes32 public constant MEMBER_REMOVER_ROLE = keccak256("MEMBER_REMOVER");
```
*Instances(5)*
```
File: security-council-mgmt/SecurityCouncilManager.sol#L4-L14
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L79-L83
```

### [Gas-03] Use `constant` Value Instead Of `type(uint240).max`
```solidity
if (x > type(uint240).max) { // @audit G = Constant can be used
            revert UintTooLarge(x);
        }
```
*Instances(1)*
```
File: security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L260
```

### [Gas-04] Nested `if` Is Cheaper Than Single Statement
```solidity 
if (
                existantSecurityCouncil.chainId == _securityCouncilData.chainId
                    && existantSecurityCouncil.securityCouncil == _securityCouncilData.securityCouncil  // @audit
            ) {

 if (
                securityCouncilData.securityCouncil == _securityCouncilData.securityCouncil
                    && securityCouncilData.chainId == _securityCouncilData.chainId // @audit G=
                    && securityCouncilData.updateAction == _securityCouncilData.updateAction
            ) {
```
*Instances(2)*
```
File: security-council-mgmt/SecurityCouncilManager.sol
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L163.sol#L254-L257
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L163.sol#L286-L290
```

### [Gas-05] In Some Places `bool` Initialized To Their Default Value
```solidity
for (uint256 i = 0; i < array1.length; i++) {
            bool found = false;


for (uint256 i = 0; i < array2.length; i++) {
            bool found = false;
```
*Instances(2)*
```
File: gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L62

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L75
```

### [Gas-06] Non-usage Of Specific `imports`
*Instances()*
```
File:
```

### [Gas-07] Empty Blocks Should Be Removed Or Emit Something
The code should be refactored such that they no longer exist, or the block should do something useful, such as emiting an event or reverting.
*Instances()*
```
File:
```

### [Gas-08] Gas-Saving Benefits of Same Value Checks for Boolean State Variables
Saves `60000 GAS, 3 SSTORE`

This ensures that the value of the `bool` state variable is never overwritten with the same value. This will saves `60000 GAS and 3 SSTORE`. Add same value check before assigning values

```solidity
if (!isContender(proposalId, contender)) { // @audit
            revert NotEligibleContender(contender);
        }
```
```solidity
      if (
            !securityCouncilManager.firstCohortIncludes(memberToRemove)
                && !securityCouncilManager.secondCohortIncludes(memberToRemove) // @audit
        ) {

      if (!(0 < _voteSuccessNumerator && _voteSuccessNumerator <= voteSuccessDenominator)) { // @audit
            revert InvalidVoteSuccessNumerator(_voteSuccessNumerator);
        }
```
*Instances(4)*
```
File: src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L134-L137

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L183-L185
```
```
File: security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L22
```

### [Gas-09] Increment or Decrement By 1 Can Use `++X` Instead Of `X+=1` Saves Gas In State Variables
using the increment or decrement operator (++) or (--) instead of the assignment operator (+=) or (-=) can save gas when incrementing or decrementing a state variable.

The reason for this is that the increment or decrement operator is a single instruction, while the assignment operator is two instructions. The first instruction assigns the value of the variable to itself, and the second instruction increments or decrements the value.

Saves `32 GAS` per Instance
```solidity
month += 1;
```
*Instances(1)*

```
File: security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol#L84
```

### [Gas-10] Use `assembly` For `Zero address` Checks
```solidity
if (_addressToRemove == address(0) || _addressToAdd == address(0)) {  // @audit
            revert ZeroAddress();
        }
```
```solidity
if (_l1ArbitrumTimelock == address(0)) {

if (chainAndUpExecLocation.location.upgradeExecutor == address(0)) {
```
*Instances(5)*
```
File: security-council-mgmt/SecurityCouncilManager.sol
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L163.sol#L184
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L163.sol#L222
```
```
File: UpgradeExecRouteBuilder.sol
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L72
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L78
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L143
```

### [Gas-11] Use Of `!=0` Is More Gas Efficient Than Using `> 0` In Case Of `uint`
```solidity
 return _elections[proposalId].votesUsed[account] > 0; 
```
```solidity
return votesUsed(proposalId, account) > 0;
```
*Instances(2)*

```
File: security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L22
```
```
File: security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L159
```


### [Gas-12] Using `bools` For Storage Incurs Overhead.
    Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the  slot's contents, replace the bits taken up by the boolean, and then write     back. This is the compiler's defense against contract upgrades and pointer aliasing, and it cannot be disabled.

```solidity
mapping(address => bool) isNominee;
```
```
File: security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L22
```

### [Gas-13] Usage Of `UINTS/INTS` Smaller Than 32 Bytes (256 BITS) Incurs Overhead
When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
```solidity
   uint64 removalGovMinPeriodAfterQuorum;
```
```
File:
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol#L34
```

### [Gas-14] Use `assembly` In Place Of `abi.decode` To Extract `calldata` Value More Efficiently.
```solidity
(address contender, uint256 votes) = abi.decode(params, (address, uint256));
```
```solidity
address memberToRemove = abi.decode(rest, (address));
```
*Instances(4)*

```
File: security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L22
```
```
File: src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L133
```
```
File: security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L110
```

### [Gas-15] Use `assembly` To Read From And Write To Packed Storage Slots.
*Instances()*
```
File:
```

### [Gas-16] Use `assembly` To Validate `msg.sender`
```solidity
if (msg.sender != address(nomineeElectionGovernor)) { 
```
*Instances(Multiple)*
```
File:
```
```
File: security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L79
```

### [Gas-17] Importing An Entire Library While Only Using One Functions Isn't Necessary
```solidity
import "./modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol";
import "../interfaces/ISecurityCouncilMemberElectionGovernor.sol";
import "../interfaces/ISecurityCouncilNomineeElectionGovernor.sol";
import "../interfaces/ISecurityCouncilManager.sol";
import "./modules/ElectionGovernor.sol";
```
```solidity
import "../../interfaces/ISecurityCouncilManager.sol"; // @audit
import "solady/utils/DateTimeLib.sol";
import "../../Common.sol";
```
```solidity
import "./../interfaces/ISecurityCouncilManager.sol";
import "../Common.sol";
import "./modules/ArbitrumGovernorVotesQuorumFractionUpgradeable.sol";
import "./modules/ArbitrumGovernorProposalExpirationUpgradeable.sol";
```
```solidity
import "../ArbitrumTimelock.sol";
import "../UpgradeExecutor.sol";
import "../L1ArbitrumTimelock.sol";
import "./SecurityCouncilMgmtUtils.sol";
import "./interfaces/ISecurityCouncilManager.sol";
import "./SecurityCouncilMemberSyncAction.sol";
import "../UpgradeExecRouteBuilder.sol";
import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
import "@openzeppelin/contracts/utils/Address.sol";
import "@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol";
import "./Common.sol";
```
```
import "./interfaces/IGnosisSafe.sol";
import "./SecurityCouncilMgmtUtils.sol";  // @audit
import "../gov-action-contracts/execution-record/ActionExecutionRecord.sol";
```
*Instances(All contracts have)*
```
File: security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L7-L11
```
```
File: security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol#L4-L8
```
```
File: src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L10-L13
```
```
File: security-council-mgmt/SecurityCouncilManager.sol#L4-L14
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L4-L14
```
```
File: security-council-mgmt/SecurityCouncilMemberSyncAction.sol
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L4-L6
```

### [Gas-18] Creation Of `stack variable` Should Be In Outside Of Loop, And Those Get Overriden With Each Iteration Of Loop.
Creation of `address[] storage cohort` variable should be in outside of loop.
```solidity
for (uint256 i = 0; i < 2; i++) {
            address[] storage cohort = i == 0 ? firstCohort : secondCohort; 
````
```solidity
for (uint256 i = 0; i < input.length; i++) {
            address nominee = input[i];
```
```solidity
for (uint256 i = 0; i < _updatedMembers.length; i++) {
            address member = _updatedMembers[i];

for (uint256 i = 0; i < previousOwners.length; i++) {
            address owner = previousOwners[i];
```
*Instances(4)*
```
File: security-council-mgmt/SecurityCouncilManager.sol
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L163.sol#L163
```
```
File: security-council-mgmt/SecurityCouncilMgmtUtils.sol
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol#L23
```
```
File: security-council-mgmt/SecurityCouncilMemberSyncAction.sol
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L61

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L68
```

### [Gas-19] Avoid Use Of `block.timestamp` In Event Emission
```solidity
if (startTimestamp <= block.timestamp) {
            revert StartDateTooEarly(startTimestamp, block.timestamp); // @audit G
        }
```
*Instances(1)*
```
File: security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol#L61
```

### [Gas-20] Use `do while loops` Instead Of `for` loops
A `do while loop` will cost less gas since the condition is not being checked for first iteration 
*Instances(Multiple Instances)*
```
File: security-council-mgmt/SecurityCouncilManager.sol
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L163.sol#L163
```
```
File: security-council-mgmt/SecurityCouncilMgmtUtils.sol
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol#L23
```
```
File: security-council-mgmt/SecurityCouncilMemberSyncAction.sol
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L61

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L68
```

