# gas

# summary
|      | issue | instance  |
|------|-------|-----------|
|[G-01]|Use calldata instead of memory for function arguments that do not get mutated|4|
|[G-02]|Use do while loops instead of for loops|1|
|[G-03]|Use assembly to perform efficient back-to-back calls|3|
|[G-04]|Use Modifiers Instead of Functions To Save Gas|4|
|[G-05]|Use assembly to write address storage values|4|
|[G-06]|Can Make The Variable Outside The Loop To Save Gas|5|
|[G-07]|Use of Bit shift operators|1|
|[G-08]|Use Assembly To Check For address(0)|8|
|[G-09]|Use constants instead of type(uintx).max|1|
|[G-10]| State variables can be packed to use fewer storage slots|~|
|[G-11]| >=/<= costs less gas than >/<|4|
|[G-12]|Use != 0 instead of > 0 for unsigned integer comparison|2|
|[G-13]| A modifier used only once and not being inherited should be inlined to save gas|1|
|[G-14]|use Mappings Instead of Arrays|2|
|[G-15]|Unnecessary computation|1|
|[G-16]|Use assembly for math (add, sub, mul, div)|2|

## [G-01] Use calldata instead of memory for function arguments that do not get mutated
Mark data types as calldata instead of memory where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as calldata. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies memory storage.


```solidity
File: security-council-mgmt/SecurityCouncilManager.sol
90        address[] memory _firstCohort,
91        address[] memory _secondCohort,

370   function generateSalt(address[] memory _members, uint256 nonce)
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L90-L91

```solidity
File: security-council-mgmt/SecurityCouncilMemberSyncAction.sol
31   function perform(address _securityCouncil, address[] memory _updatedMembers, uint256 _nonce)
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L31

## [G-02] Use do while loops instead of for loops
A do while loop will cost less gas since the condition is not being checked for the first iteration.
[Reffrence](https://code4rena.com/reports/2023-05-ajna#g-09-use-do-while-loops-instead-of-for-loops)

```solidity
File: security-council-mgmt/SecurityCouncilManager.sol
162   for (uint256 i = 0; i < 2; i++) {
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L162

recommend code:
To convert the given for loop to a do while loop, you can rewrite the code as follows:
```
uint256 i = 0;
do {
    address[] storage cohort = i == 0 ? firstCohort : secondCohort;
    uint256 j = 0;
    do {
        if (_member == cohort[j]) {
            cohort[j] = cohort[cohort.length - 1];
            cohort.pop();
            return i == 0 ? Cohort.FIRST : Cohort.SECOND;
        }
        j++;
    } while (j < cohort.length);
    i++;
} while (i < 2);
```

## [G-03] Use assembly to perform efficient back-to-back calls
If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.
Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if neccessary.
[Reffrence](https://code4rena.com/reports/2023-05-ajna#g-10-use-assembly-to-perform-efficient-back-to-back-calls)


```solidity
File: gov-action-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol
69    bytes32 TIMELOCK_PROPOSAL_ROLE = l2CoreGovTimelock.PROPOSER_ROLE();

70    bytes32 TIMELOCK_CANCELLER_ROLE = l2CoreGovTimelock.CANCELLER_ROLE();
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol#L69-L70

```solidity
File: gov-action-contracts/AIPs/SecurityCouncilMgmt/L1SCMgmtActivationAction.sol
50           l1Timelock.revokeRole(TIMELOCK_CANCELLER_ROLE, address(prevEmergencySecurityCouncil));

51           l1Timelock.grantRole(TIMELOCK_CANCELLER_ROLE, address(l1UpgradeExecutor));
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/L1SCMgmtActivationAction.sol#L50-L51

```solidity
File: security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol
135      !securityCouncilManager.firstCohortIncludes(memberToRemove)

136      && !securityCouncilManager.secondCohortIncludes(memberToRemove)
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L135-L136


## [G-04] Use Modifiers Instead of Functions To Save Gas
Example of two contracts with modifiers and internal view function:

```
// SPDX-License-Identifier: MIT
pragma solidity 0.8.9;
contract Inlined {
    function isNotExpired(bool _true) internal view {
        require(_true == true, "Exchange: EXPIRED");
    }
function foo(bool _test) public returns(uint){
            isNotExpired(_test);
            return 1;
    }
}
// SPDX-License-Identifier: MIT
pragma solidity 0.8.9;
contract Modifier {
modifier isNotExpired(bool _true) {
        require(_true == true, "Exchange: EXPIRED");
        _;
    }
function foo(bool _test) public isNotExpired(_test)returns(uint){
        return 1;
    }
}
```

Differences:

```
Deploy Modifier.sol
108727
Deploy Inlined.sol
110473
Modifier.foo
21532
Inlined.foo
21556
```

This with 0.8.9 compiler and optimization enabled. As you can see it's cheaper to deploy with a modifier, and it will save you about 30 gas. But sometimes modifiers increase code size of the contract.


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

```solidity
File: security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol
138  function isNominee(uint256 proposalId, address contender) public view returns (bool) {
        return _elections[proposalId].isNominee[contender];
    }
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L138

## [G-05] Use assembly to write address storage values
By using assembly to write to address storage values, you can bypass some of these operations and lower the gas cost of writing to storage. Assembly code allows you to directly access the Ethereum Virtual Machine (EVM) and perform low-level operations that are not possible in Solidity.

example of using assembly to write to address storage values:
```
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
File: security-council-mgmt/SecurityCouncilManager.sol
100    firstCohort = _firstCohort;
101    secondCohort = _secondCohort;

115    l2CoreGovTimelock = _l2CoreGovTimelock;
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L100

```solidity
File: security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol
248   nomineeVetter = _nomineeVetter;
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L248

## [G-06] Can Make The Variable Outside The Loop To Save Gas 
When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. Here's an example:

```
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        uint256 total = 0;
        
        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }
        
        return total;
    }
}
```

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

```solidity
File: security-council-mgmt/SecurityCouncilMgmtUtils.sol
23    address nominee = input[i];
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol#L23

## [G-07]Use of Bit shift operators
Check if arithmethic operations can be achieved using bitwise operators, if yes, implement same operations using bitwise operators and compare the gas consumed. Usually bitwise logic will be cheaper (ofc we cannot use bitwise everywhere, there some limitations)

Note: Bit shift << and >> operators are not among the arithmetic ones, and thus don’t revert on overflow.
exmple:
-    uint256 alpha = someVar / 256;
-    uint256 beta = someVar % 256;
+    uint256 alpha = someVar >> 8;
+    uint256 beta = someVar & 0xff;


-    uint 256 alpha = delta / 2;
-    uint 256 beta = delta / 4; 
-    uint 256 gamma = delta * 8;
+    uint 256 alpha = delta >> 1;
+    uint 256 beta = delta >> 2; 
+    uint 256 gamma = delta << 3;



```solidity
File:  security-council-mgmt/governors/modules/ElectionGovernor.sol
51   return Cohort(electionIndex % 2);
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/ElectionGovernor.sol#L51

## [G-08] Use Assembly To Check For address(0)
it's generally more gas-efficient to use assembly to check for a zero address (address(0)) than to use the == operator.

The reason for this is that the == operator generates additional instructions in the EVM bytecode, which can increase the gas cost of your contract. By using assembly, you can perform the zero address check more efficiently and reduce the overall gas cost of your contract.

Here's an example of how you can use assembly to check for a zero address:

```
contract MyContract {
    function isZeroAddress(address addr) public pure returns (bool) {
        uint256 addrInt = uint256(addr);
        
        assembly {
            // Load the zero address into memory
            let zero := mload(0x00)
            
            // Compare the address to the zero address
            let isZero := eq(addrInt, zero)
            
            // Return the result
            mstore(0x00, isZero)
            return(0, 0x20)
        }
    }
}
```
In the above example, we have a function isZeroAddress that takes an address as input and returns a boolean value indicating whether the address is equal to the zero address. Inside the function, we convert the address to an integer using uint256(addr), and then use assembly to compare the integer to the zero address.

By using assembly to perform the zero address check, we can make our code more gas-efficient and reduce the overall cost of our contract. It's important to note that assembly can be more difficult to read and maintain than Solidity code, so it should be used with caution and only when necessary

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

237     _securityCouncilData.updateAction == address(0)
238      || _securityCouncilData.securityCouncil == address(0)
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L144

```solidity
File: UpgradeExecRouteBuilder.sol
72    if (_l1ArbitrumTimelock == address(0)) {

143   if (upExecLocation.inbox == address(0)) {    
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L72


## [G-09] Use constants instead of type(uintx).max
 it's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

By using a constant instead of type(uintX).max, you can avoid these additional gas costs and make your code more efficient.

Here's an example of how you can use a constant instead of type(uintX).max:
```
contract MyContract {
    uint120 constant MAX_VALUE = 2**120 - 1;
    
    function doSomething(uint120 value) public {
        require(value <= MAX_VALUE, "Value exceeds maximum");
        
        // Do something
    }
}
```
In the above example, we have a contract with a constant MAX_VALUE that represents the maximum value of a uint120. When the doSomething function is called with a value parameter, it checks whether the value is less than or equal to MAX_VALUE using the <= operator.

By using a constant instead of type(uint120).max, we can make our code more efficient and reduce the gas cost of our contract.

It's important to note that using constants can make your code more readable and maintainable, since the value is defined in one place and can be easily updated if necessary. However, constants should be used with caution and only when their value is known at compile-time.

```solidity
File: security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol
260   if (x > type(uint240).max) {
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L260

## [G-10] State variables can be packed to use fewer storage slots
The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

```solidity
File: UpgradeExecRouteBuilder.sol
47  address public constant RETRYABLE_TICKET_MAGIC = 0xa723C008e76E379c55599D2E4d93879BeaFDa79C;
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
60  uint256 public immutable l1TimelockMinDelay;
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L47-L60

recommend code:
```solidity
File: UpgradeExecRouteBuilder.sol
-  address public constant RETRYABLE_TICKET_MAGIC = 0xa723C008e76E379c55599D2E4d93879BeaFDa79C;
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

+  address public constant RETRYABLE_TICKET_MAGIC = 0xa723C008e76E379c55599D2E4d93879BeaFDa79C;       
```



## [G-11] >=/<= costs less gas than >/<
The compiler uses opcodes GT and ISZERO for code that uses >, but only requires LT for >=. A similar behaviour applies for >, which uses opcodes LT and ISZERO, but only requires GT for <=.
[Reffrence](https://gist.github.com/CloudEllie/05fda0fdecba2ed7d85f68c709e46c8d#gas-18--costs-less-gas-than-)


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

```solidity
File:  security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol
339   if (cnCount < cohortSize) {
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L339


## [G-12]Use != 0 instead of > 0 for unsigned integer comparison
it's generally more gas-efficient to use != 0 instead of > 0 when
comparing unsigned integer types.

This is because the Solidity compiler can optimize the != 0 comparison to a simple bitwise operation,
 while the > 0 comparison requires an additional subtraction operation.
  As a result, using != 0 can be more gas-efficient and can help to reduce the overall cost of your contract.

Here's an example of how you can use != 0 instead of > 0:

```
contract MyContract {
    uint256 public myUnsignedInteger;
    
    function myFunction() public view returns (bool) {
        // Use != 0 instead of > 0
        return myUnsignedInteger != 0;
    }
}
```

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


## [G-13] A modifier used only once and not being inherited should be inlined to save gas
When you use a modifier in Solidity, Solidity generates code to check the conditions of the modifier and execute the modified function if the conditions are met. This generated code can consume gas, especially if the modifier is used frequently or if the modified function is called multiple times.
By inlining a modifier that is used only once and not being inherited, you can eliminate the overhead of the generated code and reduce the gas cost of your contract.

```solidity
File: security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol
78   modifier onlyNomineeElectionGovernor() {
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L78

## [G-14]use Mappings Instead of Arrays
Arrays are useful when you need to maintain an ordered list of data that can be iterated over, but they have a higher gas cost for read and write operations, especially when the size of the array is large. This is because Solidity needs to iterate over the entire array to perform certain operations, such as finding a specific element or deleting an element.

Mappings, on the other hand, are useful when you need to store and access data based on a key, rather than an index. Mappings have a lower gas cost for read and write operations, especially when the size of the mapping is large, since Solidity can perform these operations based on the key directly, without needing to iterate over the entire data structure.

```solidity
File: security-council-mgmt/SecurityCouncilManager.sol
51     address[] internal firstCohort;

52     address[] internal secondCohort;
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L51-L52

## [G-15] Unnecessary computation
When emitting an event that includes a new and an old value, it is cheaper in gas to avoid caching the old value in memory. Instead, emit the event, then save the new value in storage.

Proof of Concept
Instances include:

```
OwnableProxyDelegation.sol
function _setOwner
Recommended Mitigation
```
Replace
```
address oldOwner = _owner;
_owner = newOwner;
emit OwnershipTransferred(oldOwner, newOwner)
```
with
```
emit OwnershipTransferred(_owner_, newOwner)
_owner = newOwner;
```
```solidity
File:  security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol
249    emit NomineeVetterChanged(oldNomineeVetter, _nomineeVetter); 
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L249


## [G-16] Use assembly for math (add, sub, mul, div)
Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety.

Addition:
```
//addition in SolidityCache multiple accesses of a mapping/array
function addTest(uint256 a, uint256 b) public pure {
    uint256 c = a + b;
}
```
Gas: 303
```
//addition in assembly
function addAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := add(a, b)
        if lt(c, a) {
            mstore(0x00, "overflow")
            revert(0x00, 0x20)
        }
    }
}
```
Gas: 263
```
Subtraction
```
//subtraction in Solidity
function subTest(uint256 a, uint256 b) public pure {
  uint256 c = a - b;
}
```
Gas: 300
```
//subtraction in assembly
function subAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := sub(a, b)
        if gt(c, a) {
            mstore(0x00, "underflow")
            revert(0x00, 0x20)
        }
    }
}
```
Gas: 263
```
Multiplication
```
//multiplication in Solidity
function mulTest(uint256 a, uint256 b) public pure {
    uint256 c = a * b;
}
```
Gas: 325
```
//multiplication in assembly
function mulAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := mul(a, b)
        if lt(c, a) {
            mstore(0x00, "overflow")
            revert(0x00, 0x20)
        }
    }
}
```
Gas: 265

Division
```
//division in Solidity
function divTest(uint256 a, uint256 b) public pure {
    uint256 c = a * b;
}
```
Gas: 325

```
//division in assembly
function divAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := div(a, b)
        if gt(c, a) {
            mstore(0x00, "underflow")
            revert(0x00, 0x20)
        }
    }
}
```
Gas: 265


```solidity
File: security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol
166   return startBlock + fullWeightDuration;

249   uint256 decreasingWeightDuration = endBlock - fullWeightVotingDeadline_;
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L166