# Gas Optimization 

# summary

|      |  issue  | instance |
|------|---------|----------|
|[G-01]|Use assembly to write address storage values|4|
|[G-02]|A modifier used only once and not being inherited should be inlined to save gas|1|
|[G-03]|Unnecessary computation|1|
|[G-04]|se assembly for math (add, sub, mul, div)|2|
|[G-05]|Use assembly to perform efficient back-to-back calls|4|
|[G-06]|Use of Bit shift operators|1|
|[G-07]|Use Modifiers Instead of Functions To Save Gas|4|
|[G-08]|Can Make The Variable Outside The Loop To Save Gas |5|
|[G-09]| >=/<= costs less gas than >/<|4|
|[G-10]|State variables can be packed to use fewer storage slots|3|
|[G-11]|use Mappings Instead of Arrays|2|
|[G-12]| Use do while loops instead of for loops|1|
|[G-13]|Use calldata instead of memory for function arguments that do not get mutated|4|
|[G-14]|Use Assembly To Check For address(0)|7|
|[G-15]|Use constants instead of type(uintx).max|1|


## [G-01] Use assembly to write address storage values

By using assembly to write to address storage values, you can bypass some of these operations and lower the gas cost of writing to storage. Assembly code allows you to directly access the Ethereum Virtual Machine (EVM) and perform low-level operations that are not possible in Solidity.
```

```solidity
File: security-council-mgmt/SecurityCouncilManager.sol
100    firstCohort = _firstCohort;
101    secondCohort = _secondCohort;

115    l2CoreGovTimelock = _l2CoreGovTimelock;
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L100

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L101

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L115

```solidity
File: security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol
248   nomineeVetter = _nomineeVetter;
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L248

## [G-02] A modifier used only once and not being inherited should be inlined to save gas

By inlining a modifier that is used only once and not being inherited, you can eliminate the overhead of the generated code and reduce the gas cost of your contract.

```solidity
File: security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol
78   modifier onlyNomineeElectionGovernor() {
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L78



## [G-03] Unnecessary computation

When emitting an event that includes a new and an old value, it is cheaper in gas to avoid caching the old value in memory. Instead, emit the event, then save the new value in storage.


```solidity
File:  security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol
249    emit NomineeVetterChanged(oldNomineeVetter, _nomineeVetter); 
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L249

## [G-04] Use assembly for math (add, sub, mul, div)

Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety.




```solidity
File: security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol
166   return startBlock + fullWeightDuration;

249   uint256 decreasingWeightDuration = endBlock - fullWeightVotingDeadline_;
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L166

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L249


## [G-05] Use assembly to perform efficient back-to-back calls


In Solidity, performing back-to-back function calls can be expensive in terms of gas costs, especially if the called functions modify the state of the contract. However, using assembly, you can optimize the gas usage by performing efficient back-to-back calls.

```solidity
File: gov-action-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol
69    bytes32 TIMELOCK_PROPOSAL_ROLE = l2CoreGovTimelock.PROPOSER_ROLE();
70    bytes32 TIMELOCK_CANCELLER_ROLE = l2CoreGovTimelock.CANCELLER_ROLE();
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol#L69

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol#L70


```solidity
File: gov-action-contracts/AIPs/SecurityCouncilMgmt/L1SCMgmtActivationAction.sol
50           l1Timelock.revokeRole(TIMELOCK_CANCELLER_ROLE, address(prevEmergencySecurityCouncil));
51           l1Timelock.grantRole(TIMELOCK_CANCELLER_ROLE, address(l1UpgradeExecutor));
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/L1SCMgmtActivationAction.sol#L50

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/L1SCMgmtActivationAction.sol#L51


## [G-06]Use of Bit shift operators


Check if arithmethic operations can be achieved using bitwise operators, if yes, implement same operations using bitwise operators and compare the gas consumed. Usually bitwise logic will be cheaper (ofc we cannot use bitwise everywhere, there some limitations)




```solidity
File:  security-council-mgmt/governors/modules/ElectionGovernor.sol
51   return Cohort(electionIndex % 2);
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/ElectionGovernor.sol#L51


## [G-07] Use Modifiers Instead of Functions To Save Gas

In Solidity, modifiers can be used to reduce gas costs by avoiding the need for separate functions with duplicated code. Modifiers are a way to add pre- and post-conditions to functions, allowing you to define reusable code that can be applied to multiple functions within a contract.


```solidity
File: security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol
368   function isCompliantNominee(uint256 proposalId, address account) public view returns (bool) {
        return isNominee(proposalId, account) && !_elections[proposalId].isExcluded[account];
      }

400   function isExcluded(uint256 proposalId, address possibleExcluded) public view returns (bool) {
        return _elections[proposalId].isExcluded[possibleExcluded];
      }      

410   function isContender(uint256 proposalId, address possibleContender)
        public
        view
        virtual
        override
        returns (bool)
     {
        return _elections[proposalId].isContender[possibleContender];
     }      
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L368


https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L400


https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L410





```solidity
File: security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol
138  function isNominee(uint256 proposalId, address contender) public view returns (bool) {
        return _elections[proposalId].isNominee[contender];
    }
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L138


## [G-08] Can Make The Variable Outside The Loop To Save Gas 


By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract.


```solidity
File: security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol
206   uint256 packed = (uint256(weights[i]) << 16) | i;
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L206

```solidity
File: security-council-mgmt/SecurityCouncilMemberSyncAction.sol
61  address member = _updatedMembers[i];

68  address owner = previousOwners[i];

106 address currentOwner = owners[i];
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L61


https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L68


https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L106

```solidity
File: security-council-mgmt/SecurityCouncilMgmtUtils.sol
23    address nominee = input[i];
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol#L23



## [G-09] >=/<= costs less gas than >/<

The compiler uses opcodes GT and ISZERO for code that uses >, but only requires LT for >=. A similar behaviour applies for >, which uses opcodes LT and ISZERO, but only requires GT for <=.


```solidity
File: security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol
56    if (_fullWeightDuration > _votingPeriod) {
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L56

```solidity
File: security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol
78   if (newFullWeightDuration > votingPeriod()) {

237  if (blockNumber > endBlock) {        
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L78


https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L237

```solidity
File:  security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol
339   if (cnCount < cohortSize) {
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L339

## [G-10] State variables can be packed to use fewer storage slots

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

```solidity
File: UpgradeExecRouteBuilder.sol
    address public constant RETRYABLE_TICKET_MAGIC = 0xa723C008e76E379c55599D2E4d93879BeaFDa79C;
    /// @notice Default args for creating a proposal, used by createProposalWithDefaulArgs and createProposalBatchWithDefaultArgs
    ///         Default is function selector for a perform function with no args: 'function perform() external'
    bytes public constant DEFAULT_GOV_ACTION_CALLDATA =
        abi.encodeWithSelector(DefaultGovAction.perform.selector);
    uint256 public constant DEFAULT_VALUE = 0;
    /// @notice Default predecessor used when calling the L1 timelock
    bytes32 public constant DEFAULT_PREDECESSOR = bytes32(0);

    /// @notice Address of the L1 timelock targetted by this route builder
    address public immutable l1TimelockAddr;
    /// @notice The minimum delay of the L1 timelock targetted by this route builder
    /// @dev    If the min delay for this timelock changes then a new route builder will need to be deployed
    uint256 public immutable l1TimelockMinDelay;
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L47-L60

```solidity
File: security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol
159   return votesUsed(proposalId, account) > 0;
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L159

```solidity
File: security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol
134   return _elections[proposalId].votesUsed[account] > 0;
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L134

## [G-11]use Mappings Instead of Arrays


Mappings,  are useful when you need to store and access data based on a key, rather than an index. Mappings have a lower gas cost for read and write operations, especially when the size of the mapping is large, since Solidity can perform these operations based on the key directly, without needing to iterate over the entire data structure.

```solidity
File: security-council-mgmt/SecurityCouncilManager.sol
51     address[] internal firstCohort;
52     address[] internal secondCohort;
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L51-L52



## [G-12] Use do while loops instead of for loops
A do while loop will cost less gas since the condition is not being checked for the first iteration.

```solidity
File: security-council-mgmt/SecurityCouncilManager.sol
162   for (uint256 i = 0; i < 2; i++) {
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L162



## [G-13] Use calldata instead of memory for function arguments that do not get mutated

Mark data types as calldata instead of memory where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as calldata. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies memory storage.


```solidity
File: security-council-mgmt/SecurityCouncilManager.sol
90        address[] memory _firstCohort,
91        address[] memory _secondCohort,

370   function generateSalt(address[] memory _members, uint256 nonce)
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L90-L91

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L90-L370

```solidity
File: security-council-mgmt/SecurityCouncilMemberSyncAction.sol
31   function perform(address _securityCouncil, address[] memory _updatedMembers, uint256 _nonce)
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L31



## [G-14] Use Assembly To Check For address(0)

it's generally more gas-efficient to use assembly to check for a zero address (address(0)) than to use the == operator.

```solidity
File: security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol
102   if (dp.nomineeVetter == address(0)) {    
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol#L102

```solidity
File: security-council-mgmt/SecurityCouncilManager.sol
144    if (_newMember == address(0)) {

184    if (_member == address(0)) {

222     if (_addressToRemove == address(0) || _addressToAdd == address(0)) {

236    if (
            _securityCouncilData.updateAction == address(0)
                || _securityCouncilData.securityCouncil == address(0)
        ) {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L144


https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L184


https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L222


https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L236

```solidity
File: UpgradeExecRouteBuilder.sol
72    if (_l1ArbitrumTimelock == address(0)) {

143   if (upExecLocation.inbox == address(0)) {    
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L72


https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L143





## [G-15] Use constants instead of type(uintx).max

 it's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

```solidity
File: security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol
260   if (x > type(uint240).max) {
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L260

