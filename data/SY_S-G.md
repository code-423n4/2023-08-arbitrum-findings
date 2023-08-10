## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] |Use constants instead of type(uintx).max | |1| | 
| [G-02] |Duplicated require()/if() checks should be refactored to a modifier or function | |2| | 
| [G-03] |Empty blocks should be removed or emit something | |1| | 
| [G-04] |>= costs less gas than > | |1| | 
| [G-05] |Avoid contract existence checks by using low level calls | |2| | 
| [G-06] |Use assembly to write address storage values  | |6| | 
| [G-07] |Use assembly for math (add, sub, mul, div)  | |4| | 
| [G-08] |Can make the variable outside the loop to save gas | |4| | 
| [G-09] |Using calldata instead of memory for read-only arguments in external functions saves gas ||4| | 
| [G-10] |Pre-increments and pre-decrements are cheaper than post-increments and post-decrements | |2| | 
| [G-11] |Don't initialize variables with default value | |4| | 
| [G-12] |Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead | |2| | 
| [G-13] |Use assembly to calculate hashes to save gas | |3| | 
| [G-14] |Using Bools For Storage Incurs Overhead | |3| | 
| [G-15] |Use bitmap to save gas | |3| | 
| [G-16] |unchecked {} can be used on the division of two uints in order to save gas | |1| | 
| [G-17] |Use uint256(1)/uint256(2) instead for true and false boolean states | |3| | 




## Gas Optimizations  

## [G-1]  Use constants instead of type(uintx).max

type(uint120).max or type(uint128).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

```solidity
file: /src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

260        if (x > type(uint240).max) {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L260
## [G-2]  Duplicated require()/if() checks should be refactored to a modifier or function   

 to reduce code duplication and improve readability.
•  Modifiers can be used to perform additional checks on the function inputs or state before it is executed. By defining a modifier to perform a specific check, we can reuse it across multiple functions that require the same check.
• A function can also be used to perform a specific check and return a boolean value indicating whether the check has passed or failed. This can be useful when the check is more complex and cannot be performed easily in a modifier.

```solidity
file: /src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

45        if (state_ != ProposalState.Succeeded) {

292       if (state_ != ProposalState.Succeeded) {    

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L145
## [G-3]  Empty blocks should be removed or emit something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.
If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation.

```solidity
file: /src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol

52    function __SecurityCouncilNomineeElectionGovernorCounting_init() internal onlyInitializing {}

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L52

## [G-4] >= costs less gas than >
 
The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas 
Reference:  https://gist.github.com/IllIllI000/3dc79d25acccfa16dee4e83ffdc6ffde

```solidity
file: /src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol
 
128        if ((quorumDenominator() / params.quorumNumeratorValue) > 500) {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L128


## [G-5] Avoid contract existence checks by using low level calls		

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.	

```solidity
file: AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol

49        IUpgradeExecutor upgradeExecutor = IUpgradeExecutor(l2AddressRegistry.coreGov().owner());

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol#L49


```solidity
file: /src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

178        proposalId = GovernorUpgradeable.propose(targets, values, callDatas, description);

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L178


## [G-6] Use assembly to write address storage values

By using assembly to write to address storage values, you can bypass some of these operations and lower the gas cost of writing to storage. Assembly code allows you to directly access the Ethereum Virtual Machine (EVM) and perform low-level operations that are not possible in Solidity.

example of using assembly to write to address storage values:
```solidity
contract MyContract {
    address private myAddress;
    
    function setAddressUsingAssembly(address newAddress) public {
        assembly {
            sstore(0, newAddress)
        }
    }
}
```


```solidity
file: /src/security-council-mgmt/SecurityCouncilManager.sol

100        firstCohort = _firstCohort;

101        secondCohort = _secondCohort;

115        l2CoreGovTimelock = _l2CoreGovTimelock;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L100


```solidity
file: /src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

114        nomineeVetter = params.nomineeVetter;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L114


```solidity
file: /src/UpgradeExecRouteBuilder.sol
 
87        l1TimelockAddr = _l1ArbitrumTimelock;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L87


```solidity
file: /src/gov-action-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol

44        securityCouncilManager = _securityCouncilManager;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol#L44
## [G-7] Use assembly for math (add, sub, mul, div)

Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety. If using Solidity versions < 0.8.0 and you are using Safemath, you can gain significant gas savings by using assembly to calculate values and checking for overflow/underflow.

```solidity
file: /src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol

77        uint256 month = firstNominationStartDate.month - 1;

79        month += 6 * electionIndex;

80        uint256 year = firstNominationStartDate.year + month / 12;

81        month = month % 12;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol#L77-L81
## [G-8] Can make the variable outside the loop to save gas

Consider making the stack variables before the loop which gonna save gas

```solidity
file: /src/security-council-mgmt/SecurityCouncilManager.sol#L252


252            SecurityCouncilData storage existantSecurityCouncil = securityCouncils[i];

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L252


```solidity
file: /src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

206            uint256 packed = (uint256(weights[i]) << 16) | i;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L206


```solidity
file: /src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol

106            address currentOwner = owners[i];

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L106


```solidity
file: /src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol

75            bool found = false;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L75
## [G-9] Using calldata instead of memory for read-only arguments in external functions saves gas 

calldata must be used when declaring  an external function's dynamic parameters.
When a function with a memory array is called externally, the abi.decode ()  step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution.
 

```solidity
file: /src/security-council-mgmt/SecurityCouncilManager.sol

124    function replaceCohort(address[] memory _newCohort, Cohort _cohort)

271    function addSecurityCouncil(SecurityCouncilData memory _securityCouncilData)

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L124


```solidity
file: /src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol
 
31    function perform(address _securityCouncil, address[] memory _updatedMembers, uint256 _nonce)

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L31


```solidity
file: /src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol

81    function deploy(DeployParams memory dp, ContractImplementations memory impls)

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol#L81
## [G-10] Pre-increments and pre-decrements are cheaper than post-increments and post-decrements
       
  Saves 5 gas per iteration
 

```solidity
file: /src/security-council-mgmt/SecurityCouncilManager.sol

424        updateNonce++;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L424


```solidity
file: /src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

180        electionCount++;

280        election.excludedNomineeCount++;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L180
## [G-11] Don't initialize variables with default value

Uninitialized variables are assigned with the types default value. Explicitly initializing a variable with itâ€™s default value costs unnecesary gas.


```solidity
file: /src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol

62            bool found = false;

75            bool found = false;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L62


```solidity
file: /src/UpgradeExecRouteBuilder.sol

52    uint256 public constant DEFAULT_VALUE = 0;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L52


```solidity
file: /src/security-council-mgmt/SecurityCouncilMgmtUtils.sol

20        uint256 intermediateLength = 0;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol#L20
## [G-12] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead         

When using elements that are smaller than 32 bytes, your contract's gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

```solidity
file: /src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol

65        uint8 support,

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L65


```solidity
file: /src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

448    function castVoteBySig(uint256, uint8, uint8, bytes32, bytes32)

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L448
## [G-13] Use assembly to calculate hashes to save gas

Using assembly to calculate hashes can save 80 gas per instance

```solidity
file: /src/security-council-mgmt/SecurityCouncilManager.sol

375        return keccak256(abi.encodePacked(_members, nonce));

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L375


```solidity
file: /src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L200

200            hashProposal(prevTargets, prevValues, prevCallDatas, keccak256(bytes(prevDescription)));

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L200


```solidity
file: /src/gov-action-contracts/execution-record/ActionExecutionRecord.sol

38        return uint256(keccak256(abi.encode(actionContractId, key)));

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/execution-record/ActionExecutionRecord.sol#L38
## [G-14] Using Bools For Storage Incurs Overhead

Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot's contents, replace the bits taken up by the boolean, and then write back. This is the compiler's defense against contract upgrades and pointer aliasing, and it cannot be disabled.

```solidity
file: /src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

55        mapping(address => bool) isContender;

57        mapping(address => bool) isExcluded;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L55-L56


```solidity
file: /src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.so

32        bool isSupportedDateTime = DateTimeLib.isSupportedDateTime({

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol#L32
## [G-15]  Use bitmap to save gas

```solidity
file: /src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

240        election.isContender[msg.sender] = true;

279        election.isExcluded[nominee] = true;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L240


```solidity
file: /src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol

123        _elections[proposalId].isNominee[account] = true;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L123
## [G-16] unchecked {} can be used on the division of two uints in order to save gas

Make such found divisions are unchecked when ensured it is safe to do so.

```solidity
file: /src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol

80        uint256 year = firstNominationStartDate.year + month / 12;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol#L80
## [G-17] Use uint256(1)/uint256(2) instead for true and false boolean states

If you don't use boolean for storage you will avoid Gwarmaccess 100 gas. In addition, state changes of boolean from true to false can cost up to ~20000 gas rather than uint256(2) to uint256(1) that would cost significantly less.

```solidity
file: /src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

240        election.isContender[msg.sender] = true;

279        election.isExcluded[nominee] = true;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L240


```solidity
file: /src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol

123        _elections[proposalId].isNominee[account] = true;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L123


