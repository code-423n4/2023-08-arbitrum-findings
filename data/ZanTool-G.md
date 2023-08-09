# 1. Parameters Should Be Declared as Calldata
## Location
src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol: 31
src/security-council-mgmt/SecurityCouncilManager.sol: 90, 91, 92, 93, 124, 370
src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol: 103
src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol: 107, 108, 109, 110
src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol: 191
src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol: 181
src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol: 423
## Description
When the compiler parses the external or public function, it can directly read the function parameters from calldata. Setting it to other storage locations may waste gas.
## Recommendation
In external or public functions, the storage location of function parameters should be set to calldata to save gas.
# 2. Prefer uint256
## Location
src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol: 205, 215
## Description
It is recommended to replace integer types that are not 32 bytes in size and cannot be combined with other storage with uint256 to avoid the gas overhead caused by filling 32 bytes in operation.
## Recommendation
It is recommended to replace integer types that are not 32 bytes in size and cannot be combined with other storage with uint256.
# 3. Variables Should Be Constants
## Location
src/security-council-mgmt/SecurityCouncilManager.sol: 67
## Description
There are unchanging state variables in the contract, and putting unchanging state variables in storage will waste gas.
## Recommendation
The unchanging state variables in the contract should be declared as constants, which can save gas.
# 4. Use != 0 Instead of > 0 for Unsigned Integer Comparison
## Location
src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol: 183
## Description
For unsigned integers, use !=0 for comparison, which consumes less gas than >0.
## Recommendation
For unsigned integers, it is recommended to use !=0 instead of >0 for comparison.