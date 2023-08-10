## GAS-1: <X> <= <Y> costs more gas than <X> < <Y> + 1

### Description

In Solididy, the opcode 'less or equal' doesn't exist. So the EVM will translate this by two distinct operation, first the inferior, and then the equal which cost more gas then a strict less.

### Affected file

* SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol (Line: 232, 243)
* SecurityCouncilMemberRemovalGovernor.sol (Line: 183)
* SecurityCouncilMemberSyncAction.sol (Line: 43)
* SecurityCouncilNomineeElectionGovernor.sol (Line: 302, 333)
* SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol (Line: 98)
* SecurityCouncilNomineeElectionGovernorTiming.sol (Line: 60)

## GAS-2: Do not calculate constants

### Description

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

### Affected file

* SecurityCouncilManager.sol (Line: 79, 80, 81, 82, 83)

## GAS-3: Duplicated require()/revert() checks should be refactored to a modifier or function

### Description

This saves deployment gas.

### Affected file

* GovernanceChainSCMgmtActivationAction.sol (Line: 115, 119)
* NonGovernanceChainSCMgmtActivationAction.sol (Line: 36, 40)
* SecurityCouncilMemberElectionGovernor.sol (Line: 72)
* SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol (Line: 102, 106)
* SecurityCouncilMemberRemovalGovernor.sol (Line: 78)
* SecurityCouncilMgmtUpgradeLib.sol (Line: 69, 82)
* SecurityCouncilNomineeElectionGovernor.sol (Line: 145, 236, 292, 296, 312)
* SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol (Line: 69, 73, 83)

## GAS-4: Empty blocks should be removed or emit something

### Description

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.

### Affected file

* SecurityCouncilMemberSyncAction.sol (Line: 21)
* SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol (Line: 52)

## GAS-5: Replace modifier with function

### Description

Modifiers make code more elegant, but cost more than normal functions.

### Affected file

* SecurityCouncilMemberElectionGovernor.sol (Line: 78)
* SecurityCouncilNomineeElectionGovernor.sol (Line: 134, 142)

## GAS-6: Usage of uint/int smaller than 32 bytes (256 bits) incurs overhead

### Description

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.

### Affected file

* SecurityCouncilMemberElectionGovernor.sol (Line: 191, 196, 206)
* SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol (Line: 95, 115, 115, 126, 126, 179, 179, 191, 191, 205, 215, 216, 225, 225, 259, 259, 260, 263, 263)
* SecurityCouncilMemberRemovalGovernor.sol (Line: 58, 163, 223)
* SecurityCouncilMemberSyncAction.sol (Line: 119, 119, 123, 123)
* SecurityCouncilNomineeElectionGovernor.sol (Line: 433, 438, 448)
* SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol (Line: 62)

## GAS-7: Use ```assembly``` to write address storage values

### Affected file

* ActionExecutionRecord.sol (Line: 19, 20)
* GovernanceChainSCMgmtActivationAction.sol (Line: 35, 36, 38, 39, 41, 42, 44, 45)
* L1SCMgmtActivationAction.sol (Line: 23, 24, 25, 26, 27)
* NonGovernanceChainSCMgmtActivationAction.sol (Line: 19, 20, 21, 22)
* SecurityCouncilManager.sol (Line: 100, 101, 102, 115, 252, 285, 322)
* SecurityCouncilMemberElectionGovernor.sol (Line: 71, 75)
* SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol (Line: 72, 82, 120, 180)
* SecurityCouncilMemberRemovalGovernor.sol (Line: 82, 186)
* SecurityCouncilNomineeElectionGovernor.sol (Line: 114, 118, 122, 219, 248, 271, 374)
* SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol (Line: 87)
* SecurityCouncilNomineeElectionGovernorTiming.sol (Line: 64, 65)
* UpgradeExecRouteBuilder.sol (Line: 87, 88)

## GAS-8: Use calldata instead of memory for function parameters

### Description

If a reference type function parameter is read-only, it is cheaper in gas to use calldata instead of memory. Calldata is a non-modifiable, non-persistent area where function arguments are stored, and behaves mostly like memory.

### Affected file

* ActionExecutionRecord.sol (Line: 18)
* ElectionGovernor.sol (Line: 34)
* L2SecurityCouncilMgmtFactory.sol (Line: 71, 81, 81, 192, 212)
* SecurityCouncilManager.sol (Line: 89, 89, 89, 89, 124, 231, 271, 279, 370)
* SecurityCouncilMemberElectionGovernor.sol (Line: 115)
* SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol (Line: 95, 191, 191)
* SecurityCouncilMemberRemovalGovernor.sol (Line: 106, 163, 223, 223)
* SecurityCouncilMemberSyncAction.sol (Line: 31, 127)
* SecurityCouncilMgmtUtils.sol (Line: 15)
* SecurityCouncilNomineeElectionGovernor.sol (Line: 103, 324)
* SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol (Line: 62)
* SecurityCouncilNomineeElectionGovernorTiming.sol (Line: 28)
* UpgradeExecRouteBuilder.sol (Line: 67, 105, 105, 105, 186, 186)

## GAS-9: Use constants instead of type(uintx).max

### Description

It uses more gas in the distribution process and also for each transaction than constant usage.

### Affected file

* SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol (Line: 260)

## GAS-10: Use nested if and avoid multiple check combinations

### Description

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

### Affected file

* SecurityCouncilManager.sol (Line: 254, 286)
* SecurityCouncilMemberRemovalGovernor.sol (Line: 134, 183)

## GAS-11: abi.encode() is less efficient than abi.encodePacked()

### Description

Use abi.encodePacked() where possible to save gas.

### Affected file

* ActionExecutionRecord.sol (Line: 38)
* ElectionGovernor.sol (Line: 23)
* KeyValueStore.sol (Line: 27)
* UpgradeExecRouteBuilder.sol (Line: 151)