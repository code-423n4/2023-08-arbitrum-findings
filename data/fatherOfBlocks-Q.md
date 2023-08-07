**UpgradeExecRouteBuilder.sol**
- L87/88 - No validation is performed in the constructor and the variables (l1TimelockAddr and l1TimelockMinDelay) are immutable, therefore they should be validated before setting the variable to != 0x.


**ActionExecutionRecord.sol**
- L19 - No validation is performed in the constructor and the store variable is immutable, therefore they should be validated before setting the variable to != 0x.


**NonGovernanceChainSCMgmtActivationAction.sol**
- L8/9/11 - No validation is performed in the constructor and the variables (newEmergencySecurityCouncil, prevEmergencySecurityCouncil and upgradeExecutor) are immutable, therefore they should be validated before setting the variable to != 0x.


**L1SCMgmtActivationAction.sol**
- L23/24/26/27 - No validation is performed in the constructor and the variables (newEmergencySecurityCouncil, prevEmergencySecurityCouncil, l1UpgradeExecutor and l1Timelock) are immutable, therefore they should be validated before setting the variable to != 0x.


**GovernanceChainSCMgmtActivationAction.sol**
- L13/14/16/17/23 - No validation is performed in the constructor and the variables (newEmergencySecurityCouncil, newNonEmergencySecurityCouncil, prevEmergencySecurityCouncil, prevNonEmergencySecurityCouncil and l2AddressRegistry) are immutable, therefore they should be validated before setting the variable to != 0x.


**ElectionGovernor.sol**
- L59 - A __gap is used, but the contract is not upgradeable, therefore the storage will never be modified. This would be unnecessary, in addition to generating extra gas, it can cause confusion for developers or users who see the code.


**SecurityCouncilManager.sol**
- L375 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.


**SecurityCouncilNomineeElectionGovernor.sol**
- L128 - It is missing to validate params.quorumNumeratorValue since a division is made and it could be == 0. This is necessary to handle the exception if it happens.

- L128 - A validation is carried out for 500, this value should be stored as far as possible in a variable in memory.
