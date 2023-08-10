# Gas Optimizations

| Number | Issue | Instances | Total gas saved |
|--------|-------|-----------|-----------------|
|[G-01]| Use calldata instead of memory for function parameters  | 17 |  6120  |
|[G-02]| Expensive operation inside a for loop  | 16 |    |
|[G-03]| Cache storage values in memory to minimize SLOADs  | 10 |    |
|[G-04]| Use the existing Local variable/global variable when equal to a state variable to avoid reading from state  | 1 |    |
|[G-05]| Nested if is cheaper than single statement  | 4 |  12  |
|[G-06]| Empty Blocks Should Be Removed Or Emit Something  | 2 |  8012  |
|[G-07]| Change public function visibility to external  | 21 |    |
|[G-08]| Create ISecurityCouncilNomineeElectionGovernor and ISecurityCouncilManager variable  immutable variable to avoid redundant external calls  | 1 |    |
|[G-09]| Use assembly to perform efficient back-to-back calls  | 3 |    |
|[G-10]| Use assembly in place of abi.decode to extract calldata values more efficiently  | 4 |    |
|[G-11]| Use assembly to validate msg.sender  | 2 |    |
|[G-12]| Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate  | 2 |  40,084  |
|[G-13]| Avoid emitting storage values  | 1 |    |
|[G-14]| Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead  | 15 | 150 |
|[G-15]| x += y costs more gas than x = x + y for state variables  | 17 |  226  |
|[G-16]| Using unchecked blocks to save gas  | 11 |  935  |
|[G-17]| Use assembly to write address storage values  | 7 |  518  |
|[G-18]| >= costs less gas than >  | 8 | 24   |
|[G-19]| Inverting the condition of an if-else-statement wastes gas  | 6 | 18  |
|[G-20]| Using bools for storage incurs overhead  | 4 |  68,400  |
|[G-21]| Use assembly to check for address(0)  | 11 |  66  |
|[G-22]| abi.encode() is less efficient than abi.encodePacked()  | 5 | 500  |
|[G-23]| Make 3 event parameters indexed when possible  | 6 | 220  |
|[G-24]| Assigning keccak operations to constant variables results in extra gas costs  | 5 |    |
|[G-25]| Use constants instead of type(uintx).max  | 1 |    |
|[G-26]| Use uint256(1)/uint256(2) instead for true and false boolean states  | 12 | 205,200 |
|[G-27]| Avoid contract existence checks by using low level calls | 2 | 200 |
|[G-28]| Pre-increments and pre-decrements are cheaper than post-increments and post-decrements | 3 |  |
|[G-29]| Use bitmap to save gas | 3 | 210 |
|[G-30]| Use assembly for math (add, sub, mul, div) | 4 |  |
|[G-31]| Don't initialize variables with default value | 4 | 24 |
|[G-32]| Duplicated require()/if() checks should be refactored to a modifier or function  | 2 | 56 |



## [G-01] Use calldata instead of memory for function parameters

If a reference type function parameter is read-only, it is cheaper in gas to use calldata instead of memory. Calldata is a non-modifiable, non-persistent area where function arguments are stored, and behaves mostly like memory.

Note that I’ve also flagged instances where the function is public but can be marked as external since it’s not called by the contract

```solidity
file:     src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol

31       function perform(address _securityCouncil, address[] memory _updatedMembers, uint256 _nonce)
        external
        returns (bool res)
    {

127   function _execFromModule(IGnosisSafe securityCouncil, bytes memory data) internal {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L31-L33

```solidity
file:   src/security-council-mgmt/SecurityCouncilManager.sol

89       function initialize(
        address[] memory _firstCohort,
        address[] memory _secondCohort,
        SecurityCouncilData[] memory _securityCouncils,
        SecurityCouncilManagerRoles memory _roles,
        address payable _l2CoreGovTimelock,
        UpgradeExecRouteBuilder _router
    ) external initializer {

124       function replaceCohort(address[] memory _newCohort, Cohort _cohort)
        external
        onlyRole(COHORT_REPLACER_ROLE)
    {

271      function addSecurityCouncil(SecurityCouncilData memory _securityCouncilData)
        external
        onlyRole(DEFAULT_ADMIN_ROLE)
    {

279      function removeSecurityCouncil(SecurityCouncilData memory _securityCouncilData)
        external
        onlyRole(DEFAULT_ADMIN_ROLE)
        returns (bool)
    {
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L89-L96

```solidity
file:   src/UpgradeExecRouteBuilder.sol

105       function createActionRouteData(
        uint256[] memory chainIds,
        address[] memory actionAddresses,
        uint256[] memory actionValues,
        bytes[] memory actionDatas,
        bytes32 predecessor,
        bytes32 timelockSalt
    ) public view returns (address, bytes memory)


186      function createActionRouteDataWithDefaults(
        uint256[] memory chainIds,
        address[] memory actionAddresses,
        bytes32 timelockSalt
    ) public view returns (address, bytes memory) {


```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L105-L112


```solidity
file:   SecurityCouncilMgmtUpgradeLib.sol

52      function areAddressArraysEqual(address[] memory array1, address[] memory array2)
        public
        pure
        returns (bool)
    {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L52-L58

```solidity
file:    src/security-council-mgmt/SecurityCouncilMgmtUtils.sol

15       function filterAddressesWithExcludeList(
        address[] memory input,
        mapping(address => bool) storage excludeList
    ) internal view returns (address[] memory) {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol#L15-L18

```solidity
file:   src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol

115       function _execute(
        uint256 proposalId,
        address[] memory, /* targets */
        uint256[] memory, /* values */
        bytes[] memory callDatas,
        bytes32 /* descriptionHash */
    ) internal override {

181     function propose(address[] memory, uint256[] memory, bytes[] memory, string memory)
        public
        virtual
        override
        returns (uint256)
    {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L115-L121 

```solidity
file:  src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol

106       function propose(
        address[] memory targets,
        uint256[] memory values,
        bytes[] memory calldatas,
        string memory description
    ) public override returns (uint256) {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L106-L111

```solidity
file:    src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

324       function _execute(
        uint256 proposalId,
        address[] memory, /* targets */
        uint256[] memory, /* values */
        bytes[] memory callDatas,
        bytes32 /*descriptionHash*/
    ) internal virtual override {

423      function propose(address[] memory, uint256[] memory, bytes[] memory, string memory)
        public
        virtual
        override
        returns (uint256)
    {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L324-L330

```solidity
file:  src/security-council-mgmt/governors/modules/ElectionGovernor.sol

34   function extractElectionIndex(bytes[] memory callDatas) internal pure returns (uint256) {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/ElectionGovernor.sol#L34

```solidity
file:  src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

191       function selectTopNominees(address[] memory nominees, uint240[] memory weights, uint256 k)
        public
        pure
        returns (address[] memory)
    {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L191-L194


## [G-02] Expensive operation inside a for loop

```solidity
file:   src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

181     for (uint256 i = 0; i < nominees.length; i++) {
            weights[i] = election.weightReceived[nominees[i]];
        }


215    for (uint16 i = 0; i < k; i++) {
            topNomineesAddresses[i] = nominees[uint16(topNomineesPacked[i])];
        }


```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L181-L183

```solidity
file:   src/security-council-mgmt/SecurityCouncilMgmtUtils.sol

31     for (uint256 i = 0; i < intermediateLength; i++) {
            output[i] = intermediate[i];
        }
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol#L31-L33

```solidity
file:    src/security-council-mgmt/SecurityCouncilManager.sol

106      for (uint256 i = 0; i < _roles.memberRemovers.length; i++) {
            _grantRole(MEMBER_REMOVER_ROLE, _roles.memberRemovers[i]);
        }

118      for (uint256 i = 0; i < _securityCouncils.length; i++) {
            _addSecurityCouncil(_securityCouncils[i]);
        }

135      for (uint256 i = 0; i < _newCohort.length; i++) {
            _addMemberToCohortArray(_newCohort[i], _cohort);
        }

339      for (uint256 i = 0; i < firstCohort.length; i++) {
            members[i] = firstCohort[i];
        }

342      for (uint256 i = 0; i < secondCohort.length; i++) {
            members[firstCohort.length + i] = secondCohort[i];
        }
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L106-L108

```solidity
file:  src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol

60     for (uint256 i = 0; i < _updatedMembers.length; i++) {
            address member = _updatedMembers[i];
            if (!securityCouncil.isOwner(member)) {
                _addMember(securityCouncil, member, threshold);
            }
        }

67     for (uint256 i = 0; i < previousOwners.length; i++) {
            address owner = previousOwners[i];
            if (!SecurityCouncilMgmtUtils.isInArray(owner, _updatedMembers)) {
                _removeMember(securityCouncil, owner, threshold);
            }
        }

105    for (uint256 i = 0; i < owners.length; i++) {
            address currentOwner = owners[i];
            if (currentOwner == _owner) {
                return previousOwner;
            }
            previousOwner = currentOwner;
        }

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L60-L65

```solidity
file:  src/security-council-mgmt/SecurityCouncilMgmtUtils.sol

22   for (uint256 i = 0; i < input.length; i++) {
            address nominee = input[i];
            if (!excludeList[nominee]) {
                intermediate[intermediateLength] = nominee;
                intermediateLength++;
            }
        }

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol#L22-L28



```solidity
file:   src/UpgradeExecRouteBuilder.sol

193            for (uint256 i = 0; i < chainIds.length; i++) {
            actionDatas[i] = DEFAULT_GOV_ACTION_CALLDATA;
            values[i] = DEFAULT_VALUE;
        }

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L193-L196

```solidity
file:     src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol

61           for (uint256 i = 0; i < array1.length; i++) {
            bool found = false;
            for (uint256 j = 0; j < array2.length; j++) {
                if (array1[i] == array2[j]) {
                    found = true;
                    break;
                }
            }
            if (!found) {
                return false;
            }
        }

74          for (uint256 i = 0; i < array2.length; i++) {
            bool found = false;
            for (uint256 j = 0; j < array1.length; j++) {
                if (array2[i] == array1[j]) {
                    found = true;
                    break;
                }
            }
            if (!found) {
                return false;
            }
        }

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L61-L72

```solidity
file:   src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

105     for (uint16 i = 0; i < nominees.length; i++) {
            uint256 packed = (uint256(weights[i]) << 16) | i;

            if (topNomineesPacked[0] < packed) {
                topNomineesPacked[0] = packed;
                LibSort.insertionSort(topNomineesPacked);
            }
        }

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L105-L112

## [G-03] Cache storage values in memory to minimize SLOADs

The code can be optimized by minimizing the number of SLOADs.

### L1SCMgmtActivationAction.sol.perform(): l1Timelock should be cached to save gas 

```solidity
file:   src/gov-action-contracts/AIPs/SecurityCouncilMgmt/L1SCMgmtActivationAction.sol

40        bytes32 TIMELOCK_CANCELLER_ROLE = l1Timelock.CANCELLER_ROLE();

41   require(
            l1Timelock.hasRole(TIMELOCK_CANCELLER_ROLE, address(prevEmergencySecurityCouncil)),
            "GovernanceChainSCMgmtActivationAction: prev emergency security council should have cancellor role"
        );

45    require(
            !l1Timelock.hasRole(TIMELOCK_CANCELLER_ROLE, address(l1UpgradeExecutor)),
            "GovernanceChainSCMgmtActivationAction: l1UpgradeExecutor already has cancellor role"
        );

50    l1Timelock.revokeRole(TIMELOCK_CANCELLER_ROLE, address(prevEmergencySecurityCouncil));

51    l1Timelock.grantRole(TIMELOCK_CANCELLER_ROLE, address(l1UpgradeExecutor));

54    require(
            l1Timelock.hasRole(TIMELOCK_CANCELLER_ROLE, address(l1UpgradeExecutor)),
            "GovernanceChainSCMgmtActivationAction: l1UpgradeExecutor canceller role not set"
        );

58    require(
            !l1Timelock.hasRole(TIMELOCK_CANCELLER_ROLE, address(prevEmergencySecurityCouncil)),
            "GovernanceChainSCMgmtActivationAction: prevEmergencySecurityCouncil canceller role not revoked"
        );

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/L1SCMgmtActivationAction.sol#L40-L45

### Recomanded code 

```solidity
    
    +   ICoreTimelock _l1Timelock = l1TimeLock;

    -   bytes32 TIMELOCK_CANCELLER_ROLE = l1Timelock.CANCELLER_ROLE();

    +   bytes32 TIMELOCK_CANCELLER_ROLE = _l1Timelock.CANCELLER_ROLE();

        require(
    -        l1Timelock.hasRole(TIMELOCK_CANCELLER_ROLE, address(prevEmergencySecurityCouncil)),
            "GovernanceChainSCMgmtActivationAction: prev emergency security council should have cancellor role"
        );

        require(
    +        _l1Timelock.hasRole(TIMELOCK_CANCELLER_ROLE, address(prevEmergencySecurityCouncil)),
            "GovernanceChainSCMgmtActivationAction: prev emergency security council should have cancellor role"
        );

```

### NonGovernanceChainSCMgmtActivationAction.sol.perform(): upgradeExecutor should be cached to save gas

```solidity
file:  src/gov-action-contracts/AIPs/SecurityCouncilMgmt/NonGovernanceChainSCMgmtActivationAction.sol

35     bytes32 EXECUTOR_ROLE = upgradeExecutor.EXECUTOR_ROLE();

36     require(
            upgradeExecutor.hasRole(EXECUTOR_ROLE, address(newEmergencySecurityCouncil)),
            "NonGovernanceChainSCMgmtActivationAction: new emergency security council not set"
        );

40     require(
            !upgradeExecutor.hasRole(EXECUTOR_ROLE, address(prevEmergencySecurityCouncil)),
            "NonGovernanceChainSCMgmtActivationAction: prev emergency security council still set"
        );

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/NonGovernanceChainSCMgmtActivationAction.sol#L35

### Recommanded code 

```solidity

     +    IUpgradeExecutor _upgradeExecutor = upgradeExecutor;

     -    bytes32 EXECUTOR_ROLE = upgradeExecutor.EXECUTOR_ROLE();

     +    bytes32 EXECUTOR_ROLE = _upgradeExecutor.EXECUTOR_ROLE();


         require(
    -        upgradeExecutor.hasRole(EXECUTOR_ROLE, address(newEmergencySecurityCouncil)),
            "NonGovernanceChainSCMgmtActivationAction: new emergency security council not set"
        );


         require(
    +        _upgradeExecutor.hasRole(EXECUTOR_ROLE, address(newEmergencySecurityCouncil)),
            "NonGovernanceChainSCMgmtActivationAction: new emergency security council not set"
        );


```

## [G-04] Use the existing Local variable/global variable when equal to a state variable to avoid reading from state


### Local variable _router should be used instead of reading routerAddress


```solidity
file:   src/security-council-mgmt/SecurityCouncilManager.sol

315           function _setUpgradeExecRouteBuilder(UpgradeExecRouteBuilder _router) internal {
        address routerAddress = address(_router);

        if (!Address.isContract(routerAddress)) {
            revert NotAContract({account: routerAddress});
        }

        router = _router;
        emit UpgradeExecRouteBuilderSet(routerAddress);
    }

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L315-324

## [G-05] Nested if is cheaper than single statement

```solidity
file:  src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol

134     if (
            !securityCouncilManager.firstCohortIncludes(memberToRemove)
                && !securityCouncilManager.secondCohortIncludes(memberToRemove)
        ) 

183    if (!(0 < _voteSuccessNumerator && _voteSuccessNumerator <= voteSuccessDenominator)) {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L134-L137

```solidity
file:    src/security-council-mgmt/SecurityCouncilManager.sol

254         if (
                existantSecurityCouncil.chainId == _securityCouncilData.chainId
                    && existantSecurityCouncil.securityCouncil == _securityCouncilData.securityCouncil
            )
    
286       if (
                securityCouncilData.securityCouncil == _securityCouncilData.securityCouncil
                    && securityCouncilData.chainId == _securityCouncilData.chainId
                    && securityCouncilData.updateAction == _securityCouncilData.updateAction
            )
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L254-L257

## [G-06] Empty Blocks Should Be Removed Or Emit Something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if(x){}else if(y){…}else{…} => if(!x){if(y){…}else{…}})

```solidity
file: src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol
 
52     function __SecurityCouncilNomineeElectionGovernorCounting_init() internal onlyInitializing {}

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L52

```solidity
file:   src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol
 
21       constructor(KeyValueStore _store)
        ActionExecutionRecord(_store, "SecurityCouncilMemberSyncAction")
    {}

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L21-L24


## [G-07] Change public function visibility to external

Since the following public functions are not called from within the contract, their visibility can be made external for gas optimization.


```solidity
file:  src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol

178    function setVoteSuccessNumerator(uint256 _voteSuccessNumerator) public onlyOwner {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L178


```solidity
file:  src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

103    function initialize(InitParams memory params) public initializer {

368    function isCompliantNominee(uint256 proposalId, address account) public view returns (bool) 

373    function compliantNominees(uint256 proposalId) public view returns (address[] memory) {

388    function currentCohort() public view returns (Cohort) {

438      function castVoteWithReason(uint256, uint8, string calldata)
        public
        virtual
        override
        returns (uint256)
    {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L103

```solidity
file:  src/security-council-mgmt/governors/modules/ArbitrumGovernorVotesQuorumFractionUpgradeable.sol

38     function quorum(uint256 blockNumber) public view virtual override returns (uint256) {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/ArbitrumGovernorVotesQuorumFractionUpgradeable.sol#L38

```solidity
file:   src/security-council-mgmt/governors/modules/ElectionGovernor.sol

50      function electionIndexToCohort(uint256 electionIndex) public pure returns (Cohort) {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/ElectionGovernor.sol#L50

```solidity
file:
186      function createActionRouteDataWithDefaults(
        uint256[] memory chainIds,
        address[] memory actionAddresses,
        bytes32 timelockSalt
    ) public view returns (address, bytes memory) {


```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L186-L190

```solidity
file:  src/security-council-mgmt/SecurityCouncilManager.sol

349   function securityCouncilsLength() public view returns (uint256) {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L349

```solidity
file:   src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol

48       function initialize(
        ISecurityCouncilNomineeElectionGovernor _nomineeElectionGovernor,
        ISecurityCouncilManager _securityCouncilManager,
        IVotesUpgradeable _token,
        address _owner,
        uint256 _votingPeriod,
        uint256 _fullWeightDuration
    ) public initializer {

148   function quorum(uint256) public pure override returns (uint256) {

191   function castVote(uint256, uint8) public virtual override returns (uint256) {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L48-L55

```solidity
file:  src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

77     function setFullWeightDuration(uint256 newFullWeightDuration) public onlyGovernance {

143    function COUNTING_MODE() public pure virtual override returns (string memory) {

158    function hasVoted(uint256 proposalId, address account) public view override returns (bool) {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L77

```solidity
file:  src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol

128     function COUNTING_MODE() public pure virtual override returns (string memory) {

133     function hasVoted(uint256 proposalId, address account) public view override returns (bool) {

143     function nomineeCount(uint256 proposalId) public view returns (uint256) {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L128

```solidity
file:  src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol
 
69    function proposalVettingDeadline(uint256 proposalId) public view returns (uint256) {

75    function electionToTimestamp(uint256 electionIndex) public view returns (uint256) {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol#L69


## [G-08] Create ISecurityCouncilNomineeElectionGovernor and ISecurityCouncilManager variable  immutable variable to avoid redundant external calls

```solidity
file:  src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol

48        function initialize(
        ISecurityCouncilNomineeElectionGovernor _nomineeElectionGovernor,
        ISecurityCouncilManager _securityCouncilManager,
        IVotesUpgradeable _token,
        address _owner,
        uint256 _votingPeriod,
        uint256 _fullWeightDuration
    ) public initializer {


```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L48-L55

## [G-09] Use assembly to perform efficient back-to-back calls

If similar external calls are performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer), which can potentially allow us to avoid memory expansion costs. In this case we are also able to efficiently store both function signatures together in memory as one word, saving one MLOAD in the process.

Note: In order to do this optimization safely we will cache and restore the free memory pointer after we are done with our function calls.

### use assembly for  l1Timelock to save gas 

```solidity 
file:   src/gov-action-contracts/AIPs/SecurityCouncilMgmt/L1SCMgmtActivationAction.sol

30      function perform() external {
        // swap in new emergency security council
        SecurityCouncilMgmtUpgradeLib.replaceEmergencySecurityCouncil({
            _prevSecurityCouncil: prevEmergencySecurityCouncil,
            _newSecurityCouncil: newEmergencySecurityCouncil,
            _threshold: emergencySecurityCouncilThreshold,
            _upgradeExecutor: l1UpgradeExecutor
        });

        // swap in new emergency security counicl canceller role
        bytes32 TIMELOCK_CANCELLER_ROLE = l1Timelock.CANCELLER_ROLE();
        require(
            l1Timelock.hasRole(TIMELOCK_CANCELLER_ROLE, address(prevEmergencySecurityCouncil)),
            "GovernanceChainSCMgmtActivationAction: prev emergency security council should have cancellor role"
        );
        require(
            !l1Timelock.hasRole(TIMELOCK_CANCELLER_ROLE, address(l1UpgradeExecutor)),
            "GovernanceChainSCMgmtActivationAction: l1UpgradeExecutor already has cancellor role"
        );

        l1Timelock.revokeRole(TIMELOCK_CANCELLER_ROLE, address(prevEmergencySecurityCouncil));
        l1Timelock.grantRole(TIMELOCK_CANCELLER_ROLE, address(l1UpgradeExecutor));

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/L1SCMgmtActivationAction.sol#L30-L51



### use assembly for  l2CoreGovTimelock to save gas 

```solidity
file:   src/gov-action-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol

48       function perform() external {
        IUpgradeExecutor upgradeExecutor = IUpgradeExecutor(l2AddressRegistry.coreGov().owner());

        // swap in new emergency security council
        SecurityCouncilMgmtUpgradeLib.replaceEmergencySecurityCouncil({
            _prevSecurityCouncil: prevEmergencySecurityCouncil,
            _newSecurityCouncil: newEmergencySecurityCouncil,
            _threshold: emergencySecurityCouncilThreshold,
            _upgradeExecutor: upgradeExecutor
        });

        // swap in new nonEmergency security council
        SecurityCouncilMgmtUpgradeLib.requireSafesEquivalent(
            prevNonEmergencySecurityCouncil,
            newNonEmergencySecurityCouncil,
            nonEmergencySecurityCouncilThreshold
        );

        ICoreTimelock l2CoreGovTimelock =
            ICoreTimelock(address(l2AddressRegistry.coreGovTimelock()));

        bytes32 TIMELOCK_PROPOSAL_ROLE = l2CoreGovTimelock.PROPOSER_ROLE();
        bytes32 TIMELOCK_CANCELLER_ROLE = l2CoreGovTimelock.CANCELLER_ROLE();

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol#L48-L70

### use assembly for  getOwners to save gas 

```solidity
file:   src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol

29        function requireSafesEquivalent(
        IGnosisSafe _safe1,
        IGnosisSafe safe2,
        uint256 _expectedThreshold
    ) internal view {
        uint256 newSecurityCouncilThreshold = safe2.getThreshold();
        require(
            _safe1.getThreshold() == newSecurityCouncilThreshold,
            "SecurityCouncilMgmtUpgradeLib: threshold mismatch"
        );
        require(
            newSecurityCouncilThreshold == _expectedThreshold,
            "SecurityCouncilMgmtUpgradeLib: unexpected threshold"
        );

        address[] memory prevOwners = _safe1.getOwners();
        address[] memory newOwners = safe2.getOwners();

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L29-L45

## [G-10] Use assembly in place of abi.decode to extract calldata values more efficiently

Instead of using abi.decode, we can use assembly to decode our desired calldata values directly. This will allow us to avoid decoding calldata values that we will not use.

```solidity
file:  src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol

133    address memberToRemove = abi.decode(rest, (address));

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L133

```solidity
file:  src/security-council-mgmt/governors/modules/ElectionGovernor.sol

35   return abi.decode(callDatas[0], (uint256));

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/ElectionGovernor.sol#L35

```solidity 
file:    src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

110      (address nominee, uint256 votes) = abi.decode(params, (address, uint256));

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L110

```solidity
file:  src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol
 
78    (address contender, uint256 votes) = abi.decode(params, (address, uint256));

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L78

## [G-11] Use assembly to validate msg.sender

We can use assembly to efficiently validate msg.sender for the didPay and uniswapV3SwapCallback functions with the least amount of opcodes necessary. Additionally, we can use xor() instead of iszero(eq()), saving 3 gas. We can also potentially save gas on the unhappy path by using scratch space to store the error selector, potentially avoiding memory expansion costs.

```solidity
file:  src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol

79     if (msg.sender != address(nomineeElectionGovernor)) {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L79

```solidity
file:  src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

135     if (msg.sender != nomineeVetter) {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L135


## [G-12] Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate

We can combine multiple mappings below into structs. This will result in cheaper storage reads since multiple mappings are accessed in functions and those values are now occupying the same storage slot, meaning the slot will become warm after the first SLOAD. In addition, when writing to and reading from the struct values we will avoid a Gsset (20000 gas) and Gcoldsload (2100 gas) since multiple struct values are now occupying the same slot.

```solidity
file:  src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

55     mapping(address => bool) isContender;
56     mapping(address => bool) isExcluded;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L55-L56

```solidity
file:  src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol

119      mapping(address => uint256) votesUsed;
120      mapping(address => uint256) votesReceived;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L119-L120

## [G-13] Avoid emitting storage values

Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. We can avoid unecessary SLOADs by caching storage values that were previously accessed and emitting those cached values.


###  emit cached prevWeightReceived + weight value instead of  election.weightReceived[nominee]

```solidity
file:    src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

128    election.weightReceived[nominee] = prevWeightReceived + weight;

        emit VoteCastForNominee({
            voter: account,
            proposalId: proposalId,
            nominee: nominee,
            votes: votes,
            weight: weight,
            totalUsedVotes: prevVotesUsed + votes,
            usableVotes: availableVotes,
            weightReceived: election.weightReceived[nominee]
        });

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L128-L139

## [G-14] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

The EVM operates with 32 byte words. Therefore, if you declare state variables less than 32 bytes the EVM will need to perform extra operations to cast your value to the specified size.


```solidity
file:  src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol

191    function castVote(uint256, uint8) public virtual override returns (uint256) {

196    function castVoteWithReason(uint256, uint8, string calldata)

206    function castVoteBySig(uint256, uint8, uint8, bytes32, bytes32)

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L191


```solidity
file:   src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol

163        function _countVote(
        uint256 proposalId,
        address account,
        uint8 support,
        uint256 weight,
        bytes memory params
    ) 

223      function _castVote(
        uint256 proposalId,
        address account,
        uint8 support,
        string memory reason,
        bytes memory params
    )

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L163-L169

```solidity
file:  src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

433    function castVote(uint256, uint8) public virtual override returns (uint256) {

438    function castVoteWithReason(uint256, uint8, string calldata)

448    function castVoteBySig(uint256, uint8, uint8, bytes32, bytes32)

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L433

```solidity
file:  src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

65    error InvalidSupport(uint8 support);

98     uint8 support,

205   for (uint16 i = 0; i < nominees.length; i++) {

215   for (uint16 i = 0; i < k; i++) {

216   topNomineesAddresses[i] = nominees[uint16(topNomineesPacked[i])];
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L65

```solidity
file:  src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol

50     error InvalidSupport(uint8 support);

65     uint8 support,
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L50

## [G-15] x += y costs more gas than x = x + y for state variables

```solidity
file:  src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol

79    month += 6 * electionIndex;

84    month += 1;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol#L79


## [G-16] Using unchecked blocks to save gas

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block.

```solidity
file:  src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol

101    actualVotes = votesThreshold - prevVotesReceived;

107    election.votesUsed[account] = prevVotesUsed + actualVotes;

108    election.votesReceived[contender] = prevVotesReceived + actualVotes;

115    totalUsedVotes: prevVotesUsed + actualVotes,

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L101

```solidity
file:  src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

127      election.votesUsed[account] = prevVotesUsed + votes;

128      election.weightReceived[nominee] = prevWeightReceived + weight;

138      totalUsedVotes: prevVotesUsed + votes,

166      return startBlock + fullWeightDuration;

249      uint256 decreasingWeightDuration = endBlock - fullWeightVotingDeadline_;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L127

```solidity
file:  src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol

77    uint256 month = firstNominationStartDate.month - 1;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol#L77

```solidity
file:    src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

142      modifier onlyVettingPeriod(uint256 proposalId) {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L142

## [G-17] Use assembly to write address storage values

By using assembly to write to address storage values, you can bypass some of these operations and lower the gas cost of writing to storage. Assembly code allows you to directly access the Ethereum Virtual Machine (EVM) and perform low-level operations that are not possible in Solidity.

```solidity
file:   src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

246       function setNomineeVetter(address _nomineeVetter) external onlyGovernance {
        address oldNomineeVetter = nomineeVetter;
        nomineeVetter = _nomineeVetter;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L246

### Recommendation Code

```solidity

   246:     setNomineeVetter(
- 248:         nomineeVetter = _nomineeVetter;
+                  assembly {                      
+                      sstore(vault. nomineeVetter , _nomineeVetter)
+                  }                               
          
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


## [G‑18] >= costs less gas than >

The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas


```solidity
file:  src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

78    if (newFullWeightDuration > votingPeriod()) {

122   if (prevVotesUsed + votes > availableVotes) {

237   if (blockNumber > endBlock) {

260    if (x > type(uint240).max) {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L78

```solidity
file: src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol

90    if (votes + prevVotesUsed > weight) {

134   return _elections[proposalId].votesUsed[account] > 0;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L90

```solidity
file:  src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol

56    if (_fullWeightDuration > _votingPeriod) {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L56

```solidity
file:  src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

157    return voteSuccessDenominator * forVotes > (forVotes + againstVotes) * voteSuccessNumerator;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#157


## [G‑19] Inverting the condition of an if-else-statement wastes gas

Flipping the true and false blocks instead saves 3 gas

```solidity
file: src/security-council-mgmt/SecurityCouncilManager.sol

147    address[] storage cohort = _cohort == Cohort.FIRST ? firstCohort : secondCohort;

163    address[] storage cohort = i == 0 ? firstCohort : secondCohort;

133    _cohort == Cohort.FIRST ? delete firstCohort : delete secondCohort;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L147


```solidity
file:  src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

390    return electionCount == 0 ? Cohort.FIRST : electionIndexToCohort(electionCount - 1);

396     return (electionCount < 2) ? Cohort.SECOND : electionIndexToCohort(electionCount - 2);

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L390

```solidity
file:  src/security-council-mgmt/SecurityCouncilMgmtUtils.sol

17    mapping(address => bool) storage excludeList

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol#L17

## [G‑20] Using bools for storage incurs overhead

    // Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.

```solidity
file:  src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

55      mapping(address => bool) isContender;

56      mapping(address => bool) isExcluded;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L55


```solidity
file:  src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol

22    mapping(address => bool) isNominee;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L22

```solidity
file: /src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.so

32        bool isSupportedDateTime = DateTimeLib.isSupportedDateTime({

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol#L32


## [G-21] Use assembly to check for address(0)

Saves 6 gas per instance



```solidity
file:  src/UpgradeExecRouteBuilder.sol

72     if (_l1ArbitrumTimelock == address(0)) {

78     if (chainAndUpExecLocation.location.upgradeExecutor == address(0)) {

94     return upExecLocations[_chainId].upgradeExecutor != address(0);

131    if (upExecLocation.upgradeExecutor == address(0)) {

143    if (upExecLocation.inbox == address(0)) {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L72


```solidity
file:  src/security-council-mgmt/SecurityCouncilManager.sol

144    if (_newMember == address(0)) {

184    if (_member == address(0)) {

222    if (_addressToRemove == address(0) || _addressToAdd == address(0)) {

237    _securityCouncilData.updateAction == address(0)

138    || _securityCouncilData.securityCouncil == address(0)

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L144

```solidity
file:  src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol

102    if (dp.nomineeVetter == address(0)) {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol#L102


## [G-22] abi.encode() is less efficient than abi.encodePacked()

Consider changing it if possible.

```solidity
file:   src/UpgradeExecRouteBuilder.sol

151     schedData[i] = abi.encode(

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L151

```solidity
file:  src/gov-action-contracts/execution-record/ActionExecutionRecord.sol

38      return uint256(keccak256(abi.encode(actionContractId, key)));

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/execution-record/ActionExecutionRecord.sol#L38

```solidity
file:  src/gov-action-contracts/execution-record/KeyValueStore.sol

27    return uint256(keccak256(abi.encode(owner, key)));

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/execution-record/KeyValueStore.sol#L27

```solidity
file: src/security-council-mgmt/SecurityCouncilManager.sol

375    return keccak256(abi.encodePacked(_members, nonce));

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L375

```solidity
file:  src/security-council-mgmt/governors/modules/ElectionGovernor.sol

23     electionData[0] = abi.encode(electionIndex);

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/ElectionGovernor.sol#L23


## [G-23] Make 3 event parameters indexed when possible

It’s the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

```solidity
file:  src/security-council-mgmt/SecurityCouncilManager.sol

41    event SecurityCouncilRemoved(
        address securityCouncil, address updateAction, uint256 securityCouncilsLength
    );

44    event UpgradeExecRouteBuilderSet(address UpgradeExecRouteBuilder);

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L41-L43


```solidity
file:  src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol

14    event UpdateNonceTooLow(
        address indexed securityCouncil, uint256 currrentNonce, uint256 providedNonce
    );

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L14-L16

```solidity
file:  src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol

32    event VoteSuccessNumeratorSet(uint256 indexed voteSuccessNumerator);

33    event MemberRemovalProposed(address memberToRemove, string description);

```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L32

```solidity
file:   src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

53     event FullWeightDurationSet(uint256 newFullWeightDuration);
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L53

## [G-24] Assigning keccak operations to constant variables results in extra gas costs


```solidity
file:  src/security-council-mgmt/SecurityCouncilManager.sol

79     bytes32 public constant COHORT_REPLACER_ROLE = keccak256("COHORT_REPLACER");

80     bytes32 public constant MEMBER_ADDER_ROLE = keccak256("MEMBER_ADDER");

81     bytes32 public constant MEMBER_REPLACER_ROLE = keccak256("MEMBER_REPLACER");

82     bytes32 public constant MEMBER_ROTATOR_ROLE = keccak256("MEMBER_ROTATOR");

83     bytes32 public constant MEMBER_REMOVER_ROLE = keccak256("MEMBER_REMOVER");

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L79

## [G-25] Use constants instead of type(uintx).max

type(uint120).max or type(uint112).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

```solidity
file:  src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

260   if (x > type(uint240).max) {


```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/
security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L260


## [G-26] Use uint256(1)/uint256(2) instead for true and false boolean states

```solidity
file:  src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol

58     return false;

62     bool found = false;

65     found = true;

70     return false;

75     bool found = false;

78     found = true;

83     return false;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L58

```solidity
file:  src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

240    election.isContender[msg.sender] = true;

279    election.isExcluded[nominee] = true;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L240

```solidity
file:  src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol

123     _elections[proposalId].isNominee[account] = true;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L123


```solidity
file:  src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol

47    return false;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L47

```solidity
file:  src/security-council-mgmt/SecurityCouncilMgmtUtils.sol

11     return false;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol#L11


## [G-27] Avoid contract existence checks by using low level calls             

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.   

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

## [G-28] Pre-increments and pre-decrements are cheaper than post-increments and post-decrements

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

## [G-29] Use bitmap to save gas

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


## [G-30] Use assembly for math (add, sub, mul, div)

Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety. If using Solidity versions < 0.8.0 and you are using Safemath, you can gain significant gas savings by using assembly to calculate values and checking for overflow/underflow.

```solidity
file: /src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol

77        uint256 month = firstNominationStartDate.month - 1;

79        month += 6 * electionIndex;

80        uint256 year = firstNominationStartDate.year + month / 12;

81        month = month % 12;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol#L77-L81

## [G-31] Don't initialize variables with default value

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


## [G-32] Duplicated require()/if() checks should be refactored to a modifier or function   

to reduce code duplication and improve readability.
•  Modifiers can be used to perform additional checks on the function inputs or state before it is executed. By defining a modifier to perform a specific check, we can reuse it across multiple functions that require the same check.
• A function can also be used to perform a specific check and return a boolean value indicating whether the check has passed or failed. This can be useful when the check is more complex and cannot be performed easily in a modifier.

```solidity
file: /src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

45        if (state_ != ProposalState.Succeeded) {

292       if (state_ != ProposalState.Succeeded) {   

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L145