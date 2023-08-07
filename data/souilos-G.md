# VULN 1 

## [GAS] Use != 0 instead of > 0 for unsigned integer comparison
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 134 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol:

        return _elections[proposalId].votesUsed[account] > 0;


Found in line 159 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol:

        return votesUsed(proposalId, account) > 0;

------------------------------------------------------------------------ 

### Mitigation 

Use != 0 instead of > 0 for unsigned integer comparison.










# VULN 2 

## [GAS] ++i/i++ should be unchecked{++i}/unchecked{i++} and ++i costs less gas than i++
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 76 at 2023-08-arbitrum/src/UpgradeExecRouteBuilder.sol:

        for (uint256 i = 0; i < _upgradeExecutors.length; i++) {


Found in line 129 at 2023-08-arbitrum/src/UpgradeExecRouteBuilder.sol:

        for (uint256 i = 0; i < chainIds.length; i++) {


Found in line 193 at 2023-08-arbitrum/src/UpgradeExecRouteBuilder.sol:

        for (uint256 i = 0; i < chainIds.length; i++) {


Found in line 106 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

        for (uint256 i = 0; i < _roles.memberRemovers.length; i++) {


Found in line 118 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

        for (uint256 i = 0; i < _securityCouncils.length; i++) {


Found in line 135 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

        for (uint256 i = 0; i < _newCohort.length; i++) {


Found in line 162 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

        for (uint256 i = 0; i < 2; i++) {


Found in line 164 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

            for (uint256 j = 0; j < cohort.length; j++) {


Found in line 251 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

        for (uint256 i = 0; i < securityCouncils.length; i++) {


Found in line 284 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

        for (uint256 i = 0; i < securityCouncils.length; i++) {


Found in line 339 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

        for (uint256 i = 0; i < firstCohort.length; i++) {


Found in line 342 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

        for (uint256 i = 0; i < secondCohort.length; i++) {


Found in line 392 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

        for (uint256 i = 0; i < securityCouncils.length; i++) {


Found in line 424 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

        updateNonce++;


Found in line 6 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol:

        for (uint256 i = 0; i < arr.length; i++) {


Found in line 22 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol:

        for (uint256 i = 0; i < input.length; i++) {


Found in line 26 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol:

                intermediateLength++;


Found in line 31 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol:

        for (uint256 i = 0; i < intermediateLength; i++) {


Found in line 60 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol:

        for (uint256 i = 0; i < _updatedMembers.length; i++) {


Found in line 67 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol:

        for (uint256 i = 0; i < previousOwners.length; i++) {


Found in line 105 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol:

        for (uint256 i = 0; i < owners.length; i++) {


Found in line 180 at 2023-08-arbitrum/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol:

        electionCount++;


Found in line 280 at 2023-08-arbitrum/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol:

        election.excludedNomineeCount++;


Found in line 181 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol:

        for (uint256 i = 0; i < nominees.length; i++) {


Found in line 205 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol:

        for (uint16 i = 0; i < nominees.length; i++) {


Found in line 215 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol:

        for (uint16 i = 0; i < k; i++) {


Found in line 111 at 2023-08-arbitrum/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol:

        for (uint256 i = 0; i < dp.firstCohort.length; i++) {


Found in line 117 at 2023-08-arbitrum/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol:

        for (uint256 i = 0; i < dp.secondCohort.length; i++) {

------------------------------------------------------------------------ 

### Mitigation 

++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for and while-loops. Moreover ++i costs less gas than i++, especially when its used in for-loops (--i/i-- too).










# VULN 3 

## [GAS] Don’t initialize variables with default value
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 76 at 2023-08-arbitrum/src/UpgradeExecRouteBuilder.sol:

        for (uint256 i = 0; i < _upgradeExecutors.length; i++) {


Found in line 129 at 2023-08-arbitrum/src/UpgradeExecRouteBuilder.sol:

        for (uint256 i = 0; i < chainIds.length; i++) {


Found in line 193 at 2023-08-arbitrum/src/UpgradeExecRouteBuilder.sol:

        for (uint256 i = 0; i < chainIds.length; i++) {


Found in line 106 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

        for (uint256 i = 0; i < _roles.memberRemovers.length; i++) {


Found in line 118 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

        for (uint256 i = 0; i < _securityCouncils.length; i++) {


Found in line 135 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

        for (uint256 i = 0; i < _newCohort.length; i++) {


Found in line 162 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

        for (uint256 i = 0; i < 2; i++) {


Found in line 164 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

            for (uint256 j = 0; j < cohort.length; j++) {


Found in line 251 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

        for (uint256 i = 0; i < securityCouncils.length; i++) {


Found in line 284 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

        for (uint256 i = 0; i < securityCouncils.length; i++) {


Found in line 339 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

        for (uint256 i = 0; i < firstCohort.length; i++) {


Found in line 342 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

        for (uint256 i = 0; i < secondCohort.length; i++) {


Found in line 392 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

        for (uint256 i = 0; i < securityCouncils.length; i++) {


Found in line 6 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol:

        for (uint256 i = 0; i < arr.length; i++) {


Found in line 20 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol:

        uint256 intermediateLength = 0;


Found in line 22 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol:

        for (uint256 i = 0; i < input.length; i++) {


Found in line 31 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol:

        for (uint256 i = 0; i < intermediateLength; i++) {


Found in line 60 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol:

        for (uint256 i = 0; i < _updatedMembers.length; i++) {


Found in line 67 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol:

        for (uint256 i = 0; i < previousOwners.length; i++) {


Found in line 105 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol:

        for (uint256 i = 0; i < owners.length; i++) {


Found in line 181 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol:

        for (uint256 i = 0; i < nominees.length; i++) {


Found in line 205 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol:

        for (uint16 i = 0; i < nominees.length; i++) {


Found in line 215 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol:

        for (uint16 i = 0; i < k; i++) {


Found in line 111 at 2023-08-arbitrum/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol:

        for (uint256 i = 0; i < dp.firstCohort.length; i++) {


Found in line 117 at 2023-08-arbitrum/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol:

        for (uint256 i = 0; i < dp.secondCohort.length; i++) {

------------------------------------------------------------------------ 

### Mitigation 

In such cases, initializing the variables with default values would be unnecessary and can be considered a waste of gas. Additionally, initializing variables with default values can sometimes lead to unnecessary storage operations, which can increase gas costs. For example, if you have a large array of variables, initializing them all with default values can result in a lot of unnecessary storage writes, which can increase the gas costs of your contract.










# VULN 4 

## [GAS] Use calldata instead of memory for function arguments that do not get mutated
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 124 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

    function replaceCohort(address[] memory _newCohort, Cohort _cohort)


Found in line 231 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

    function _addSecurityCouncil(SecurityCouncilData memory _securityCouncilData) internal {


Found in line 271 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

    function addSecurityCouncil(SecurityCouncilData memory _securityCouncilData)


Found in line 279 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

    function removeSecurityCouncil(SecurityCouncilData memory _securityCouncilData)


Found in line 370 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

    function generateSalt(address[] memory _members, uint256 nonce)


Found in line 5 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol:

    function isInArray(address addr, address[] memory arr) internal pure returns (bool) {


Found in line 31 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol:

    function perform(address _securityCouncil, address[] memory _updatedMembers, uint256 _nonce)


Found in line 127 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol:

    function _execFromModule(IGnosisSafe securityCouncil, bytes memory data) internal {


Found in line 103 at 2023-08-arbitrum/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol:

    function initialize(InitParams memory params) public initializer {


Found in line 423 at 2023-08-arbitrum/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol:

    function propose(address[] memory, uint256[] memory, bytes[] memory, string memory)


Found in line 181 at 2023-08-arbitrum/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol:

    function propose(address[] memory, uint256[] memory, bytes[] memory, string memory)


Found in line 34 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/ElectionGovernor.sol:

    function extractElectionIndex(bytes[] memory callDatas) internal pure returns (uint256) {


Found in line 191 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol:

    function selectTopNominees(address[] memory nominees, uint240[] memory weights, uint256 k)


Found in line 71 at 2023-08-arbitrum/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol:

    function _deployProxy(address proxyAdmin, address impl, bytes memory initData)


Found in line 81 at 2023-08-arbitrum/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol:

    function deploy(DeployParams memory dp, ContractImplementations memory impls)

------------------------------------------------------------------------ 

### Mitigation 

Mark data types as calldata instead of memory where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as calldata. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies memory storage.










# VULN 5 

## [GAS] Cache array length outside of loop
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 76 at 2023-08-arbitrum/src/UpgradeExecRouteBuilder.sol:

        for (uint256 i = 0; i < _upgradeExecutors.length; i++) {


Found in line 129 at 2023-08-arbitrum/src/UpgradeExecRouteBuilder.sol:

        for (uint256 i = 0; i < chainIds.length; i++) {


Found in line 193 at 2023-08-arbitrum/src/UpgradeExecRouteBuilder.sol:

        for (uint256 i = 0; i < chainIds.length; i++) {


Found in line 106 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

        for (uint256 i = 0; i < _roles.memberRemovers.length; i++) {


Found in line 118 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

        for (uint256 i = 0; i < _securityCouncils.length; i++) {


Found in line 135 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

        for (uint256 i = 0; i < _newCohort.length; i++) {


Found in line 164 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

            for (uint256 j = 0; j < cohort.length; j++) {


Found in line 251 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

        for (uint256 i = 0; i < securityCouncils.length; i++) {


Found in line 284 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

        for (uint256 i = 0; i < securityCouncils.length; i++) {


Found in line 339 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

        for (uint256 i = 0; i < firstCohort.length; i++) {


Found in line 342 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

        for (uint256 i = 0; i < secondCohort.length; i++) {


Found in line 392 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

        for (uint256 i = 0; i < securityCouncils.length; i++) {


Found in line 6 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol:

        for (uint256 i = 0; i < arr.length; i++) {


Found in line 22 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol:

        for (uint256 i = 0; i < input.length; i++) {


Found in line 60 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol:

        for (uint256 i = 0; i < _updatedMembers.length; i++) {


Found in line 67 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol:

        for (uint256 i = 0; i < previousOwners.length; i++) {


Found in line 105 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol:

        for (uint256 i = 0; i < owners.length; i++) {


Found in line 181 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol:

        for (uint256 i = 0; i < nominees.length; i++) {


Found in line 205 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol:

        for (uint16 i = 0; i < nominees.length; i++) {


Found in line 111 at 2023-08-arbitrum/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol:

        for (uint256 i = 0; i < dp.firstCohort.length; i++) {


Found in line 117 at 2023-08-arbitrum/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol:

        for (uint256 i = 0; i < dp.secondCohort.length; i++) {

------------------------------------------------------------------------ 

### Mitigation 

If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).










# VULN 6 

## [GAS] Use assembly to check for address(0)
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 72 at 2023-08-arbitrum/src/UpgradeExecRouteBuilder.sol:

        if (_l1ArbitrumTimelock == address(0)) {


Found in line 78 at 2023-08-arbitrum/src/UpgradeExecRouteBuilder.sol:

            if (chainAndUpExecLocation.location.upgradeExecutor == address(0)) {


Found in line 131 at 2023-08-arbitrum/src/UpgradeExecRouteBuilder.sol:

            if (upExecLocation.upgradeExecutor == address(0)) {


Found in line 143 at 2023-08-arbitrum/src/UpgradeExecRouteBuilder.sol:

            if (upExecLocation.inbox == address(0)) {


Found in line 144 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

        if (_newMember == address(0)) {


Found in line 184 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

        if (_member == address(0)) {


Found in line 222 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

        if (_addressToRemove == address(0) || _addressToAdd == address(0)) {


Found in line 237 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

            _securityCouncilData.updateAction == address(0)


Found in line 238 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

                || _securityCouncilData.securityCouncil == address(0)


Found in line 102 at 2023-08-arbitrum/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol:

        if (dp.nomineeVetter == address(0)) {

------------------------------------------------------------------------ 

### Mitigation 

Using assembly to check for the zero address can result in significant gas savings compared to using a Solidity expression, especially if the check is performed frequently or in a loop. However, it's important to note that using assembly can make the code less readable and harder to maintain, so it should be used judiciously and with caution.










# VULN 7 

## [GAS] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 166 at 2023-08-arbitrum/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol:

        uint8 support,


Found in line 226 at 2023-08-arbitrum/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol:

        uint8 support,


Found in line 433 at 2023-08-arbitrum/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol:

    function castVote(uint256, uint8) public virtual override returns (uint256) {


Found in line 438 at 2023-08-arbitrum/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol:

    function castVoteWithReason(uint256, uint8, string calldata)


Found in line 448 at 2023-08-arbitrum/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol:

    function castVoteBySig(uint256, uint8, uint8, bytes32, bytes32)


Found in line 191 at 2023-08-arbitrum/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol:

    function castVote(uint256, uint8) public virtual override returns (uint256) {


Found in line 196 at 2023-08-arbitrum/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol:

    function castVoteWithReason(uint256, uint8, string calldata)


Found in line 206 at 2023-08-arbitrum/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol:

    function castVoteBySig(uint256, uint8, uint8, bytes32, bytes32)


Found in line 50 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol:

    error InvalidSupport(uint8 support);


Found in line 65 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol:

        uint8 support,


Found in line 65 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol:

    error InvalidSupport(uint8 support);


Found in line 98 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol:

        uint8 support,


Found in line 205 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol:

        for (uint16 i = 0; i < nominees.length; i++) {


Found in line 215 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol:

        for (uint16 i = 0; i < k; i++) {


Found in line 216 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol:

            topNomineesAddresses[i] = nominees[uint16(topNomineesPacked[i])];

------------------------------------------------------------------------ 

### Mitigation 

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size. https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html. Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.










# VULN 8 

## [GAS] Public Functions to external
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 433 at 2023-08-arbitrum/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol:

    function castVote(uint256, uint8) public virtual override returns (uint256) {


Found in line 148 at 2023-08-arbitrum/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol:

    function quorum(uint256) public pure override returns (uint256) {


Found in line 191 at 2023-08-arbitrum/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol:

    function castVote(uint256, uint8) public virtual override returns (uint256) {


Found in line 128 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol:

    function COUNTING_MODE() public pure virtual override returns (string memory) {


Found in line 133 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol:

    function hasVoted(uint256 proposalId, address account) public view override returns (bool) {


Found in line 143 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol:

    function COUNTING_MODE() public pure virtual override returns (string memory) {


Found in line 158 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol:

    function hasVoted(uint256 proposalId, address account) public view override returns (bool) {


Found in line 38 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/ArbitrumGovernorVotesQuorumFractionUpgradeable.sol:

    function quorum(uint256 blockNumber) public view virtual override returns (uint256) {


Found in line 44 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/ArbitrumGovernorVotesQuorumFractionUpgradeable.sol:

    function quorumDenominator() public pure virtual override returns (uint256) {

------------------------------------------------------------------------ 

### Mitigation 

The following functions could be set external to save gas and improve code quality. External call cost is less expensive than of public functions.










# VULN 9 

## [GAS] <x> += <y> Costs More Gas Than <x> = <x> + <y> For State Variables
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 79 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol:

        month += 6 * electionIndex;


Found in line 84 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol:

        month += 1;

------------------------------------------------------------------------ 

### Mitigation 

When you use the += operator on a state variable, the EVM has to perform three operations: load the current value of the state variable, add the new value to it, and then store the result back in the state variable. On the other hand, when you use the = operator and then add the values separately, the EVM only needs to perform two operations: load the current value of the state variable and add the new value to it. Better use <x> = <x> + <y> For State Variables.










# VULN 10 

## [GAS] Multiple Address Mappings Can Be Combined Into A Single Mapping Of An Address To A Struct, Where Appropriate
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 55 and 56 at 2023-08-arbitrum/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol:

        mapping(address => bool) isContender;

        mapping(address => bool) isExcluded;


Found in line 19 and 20 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol:

        mapping(address => uint256) votesUsed;

        mapping(address => uint256) votesReceived;


Found in line 23 and 24 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol:

        ///      Using uint240 because of the sorting implementation, see `selectTopNominees`

        mapping(address => uint240) weightReceived;

------------------------------------------------------------------------ 

### Mitigation 

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot.










# VULN 11 

## [GAS] Using private rather than public for constants, saves gas
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 60 at 2023-08-arbitrum/src/UpgradeExecRouteBuilder.sol:

    uint256 public immutable l1TimelockMinDelay;


Found in line 67 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

    uint256 public immutable MAX_SECURITY_COUNCILS = 500;


Found in line 19 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol:

    uint256 public immutable emergencySecurityCouncilThreshold;


Found in line 20 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol:

    uint256 public immutable nonEmergencySecurityCouncilThreshold;


Found in line 12 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/L1SCMgmtActivationAction.sol:

    uint256 public immutable emergencySecurityCouncilThreshold;


Found in line 10 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/NonGovernanceChainSCMgmtActivationAction.sol:

    uint256 public immutable emergencySecurityCouncilThreshold;

------------------------------------------------------------------------ 

### Mitigation 

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that returns a tuple of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it’s used, and not adding another entry to the method ID table.










# VULN 12 

## [GAS] >= costs less gas than >
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 157 at 2023-08-arbitrum/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol:

        return voteSuccessDenominator * forVotes > (forVotes + againstVotes) * voteSuccessNumerator;


Found in line 396 at 2023-08-arbitrum/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol:

        return (electionCount < 2) ? Cohort.SECOND : electionIndexToCohort(electionCount - 2);


Found in line 134 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol:

        return _elections[proposalId].votesUsed[account] > 0;


Found in line 159 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol:

        return votesUsed(proposalId, account) > 0;

------------------------------------------------------------------------ 

### Mitigation 

The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas.










# VULN 13 

## [GAS] Empty blocks should be removed or emit something
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 23 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol:

    {}


Found in line 52 at 2023-08-arbitrum/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol:

    function __SecurityCouncilNomineeElectionGovernorCounting_init() internal onlyInitializing {}

------------------------------------------------------------------------ 

### Mitigation 

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}}).










# VULN 14 

## [GAS] Use assembly to write address storage values
------------------------------------------------------------------------ 

### Proof of concept 

Found in lines 67 to 71 at 2023-08-arbitrum/src/UpgradeExecRouteBuilder.sol:

    constructor(
        ChainAndUpExecLocation[] memory _upgradeExecutors,
        address _l1ArbitrumTimelock,
        uint256 _l1TimelockMinDelay
    ) {


Found in lines 25 to 34 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol:

    constructor(
        IGnosisSafe _newEmergencySecurityCouncil,
        IGnosisSafe _newNonEmergencySecurityCouncil,
        IGnosisSafe _prevEmergencySecurityCouncil,
        IGnosisSafe _prevNonEmergencySecurityCouncil,
        uint256 _emergencySecurityCouncilThreshold,
        uint256 _nonEmergencySecurityCouncilThreshold,
        address _securityCouncilManager,
        IL2AddressRegistry _l2AddressRegistry
    ) {

------------------------------------------------------------------------ 

### Mitigation 

Use assembly to write address storage values. Here are a few reasons: * Reduced opcode usage: When using assembly, you can directly manipulate storage values using lower-level instructions like sstore (storage store) instead of relying on higher-level Solidity storage assignments. These direct operations typically result in fewer opcode executions, reducing gas costs. * Avoiding unnecessary checks: Solidity storage assignments often involve additional checks and operations, such as enforcing security modifiers or triggering events. By using assembly, you can bypass these additional checks and perform the necessary storage operations directly, resulting in gas savings. * Optimized packing: Assembly provides greater flexibility in packing and unpacking data structures. By carefully arranging and manipulating the storage layout in assembly, you can achieve more efficient storage utilization and minimize wasted storage space. * Fine-grained control: Assembly allows for precise control over gas-consuming operations. You can optimize gas usage by selecting specific instructions and minimizing unnecessary operations or data copying.










# VULN 15 

## [GAS] require() or revert() statements that check input arguments should be at the top of the function (Also restructured some if’s)
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 72 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol:

        require(


Found in line 78 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol:

        require(


Found in line 92 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol:

        require(


Found in line 97 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol:

        require(


Found in line 104 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol:

        require(


Found in line 115 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol:

        require(


Found in line 119 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol:

        require(


Found in line 124 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol:

        require(


Found in line 130 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol:

        require(


Found in line 137 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol:

        require(


Found in line 141 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol:

        require(


Found in line 41 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/L1SCMgmtActivationAction.sol:

        require(


Found in line 45 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/L1SCMgmtActivationAction.sol:

        require(


Found in line 54 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/L1SCMgmtActivationAction.sol:

        require(


Found in line 58 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/L1SCMgmtActivationAction.sol:

        require(


Found in line 36 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/NonGovernanceChainSCMgmtActivationAction.sol:

        require(


Found in line 40 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/NonGovernanceChainSCMgmtActivationAction.sol:

        require(

------------------------------------------------------------------------ 

### Mitigation 

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting alot of gas in a function that may ultimately revert in the unhappy case.
