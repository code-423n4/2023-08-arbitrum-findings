
# LOW RISK

| Number | Issue | Instances |
|--------|-------|-----------|
|[LOW-01]| Using zero as a parameter  | 8 |
|[LOW-02]| Execution at deadlines should be allowed  | 1 |
|[LOW-03]| Arrays can grow in size without a way to shrink them  | 3 |
|[LOW-04]| Emitting storage values instead of the memory one  | 1 |
|[LOW-05]| Functions calling contracts/addresses with transfer hooks are missing reentrancy guards  | 3 |
|[LOW-06]| Use of onlyOwner functions can be lost  | 1 |
|[LOW-07]| Unbounded state variable arrays exist within the project which are iterated upon  | 3 |
|[LOW-08]|  For loops in public or external functions should be avoided due to high gas costs and possible DOS  | 1 |
|[LOW-09]| Functions which set address state variables should have zero address checks  | 3 |
|[LOW-10]| Potential division by zero should have zero checks in place  | 2 |
|[LOW-11]| Constructors contains no validation  | 2 |
|[LOW-12]| Array lengths not checked  | 1 |


## [LOW-01] Using zero as a parameter

Taking 0 as a valid argument in Solidity without checks can lead to severe security issues. A historical example is the infamous 0x0 address bug where numerous tokens were lost. This happens because '0' can be interpreted as an uninitialized address, leading to transfers to the '0x0' address, effectively burning tokens. Moreover, 0 as a denominator in division operations would cause a runtime exception. It's also often indicative of a logical error in the caller's code. It's important to always validate input and handle edge cases like 0 appropriately. Use require() statements to enforce conditions and provide clear error messages to facilitate debugging and safer code.

### Findings
Findings are labeled with ' <= FOUND'

```solidity
file:   src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol

65     __GovernorSettings_init(0, _votingPeriod, 0);     //  <= FOUND

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L65


```solidity
file:  src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

108    __GovernorSettings_init(0, params.votingPeriod, 0);  //  <= FOUND

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L108

```solidity
file:   src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol

128   if (
            !securityCouncil.execTransactionFromModule(
                address(securityCouncil), 0, data, OpEnum.Operation.Call        //  <= FOUND
            )
        ) {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L128-L132


```solidity
file:   src/UpgradeExecRouteBuilder.sol

143     if (upExecLocation.inbox == address(0)) {
                schedTargets[i] = upExecLocation.upgradeExecutor;
                schedValues[i] = actionValues[i];
                schedData[i] = executorData;
            } else {
                // For L2 actions, magic is top level target, and value and calldata are encoded in payload
                schedTargets[i] = RETRYABLE_TICKET_MAGIC;
                schedValues[i] = 0;
                schedData[i] = abi.encode(
                    upExecLocation.inbox,
                    upExecLocation.upgradeExecutor,
                    actionValues[i],
                    0,               //  <= FOUND
                    0,                //  <= FOUND
                    executorData
                );
            }
        }

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L143-L160


```solidity
file:    src/security-council-mgmt/SecurityCouncilManager.sol

405            bytes32 salt = this.generateSalt(newMembers, nonce);
        (address to, bytes memory data) = router.createActionRouteData(
            chainIds,
            actionAddresses,
            new uint256[](securityCouncils.length), // all values are always 0
            actionDatas,
            0,            //  <= FOUND
            salt
        );

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L405-L413


```solidity
file:    src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol

32       bool isSupportedDateTime = DateTimeLib.isSupportedDateTime({
            year: _firstNominationStartDate.year,
            month: _firstNominationStartDate.month,
            day: _firstNominationStartDate.day,
            hour: _firstNominationStartDate.hour,
            minute: 0,       //  <= FOUND
            second: 0        //  <= FOUND
        });


51      uint256 startTimestamp = DateTimeLib.dateTimeToTimestamp({
            year: _firstNominationStartDate.year,
            month: _firstNominationStartDate.month,
            day: _firstNominationStartDate.day,
            hour: _firstNominationStartDate.hour,
            minute: 0,       //  <= FOUND
            second: 0        //  <= FOUND
        });

86     return DateTimeLib.dateTimeToTimestamp({
            year: year,
            month: month,
            day: firstNominationStartDate.day,
            hour: firstNominationStartDate.hour,
            minute: 0,     //  <= FOUND
            second: 0      //  <= FOUND
        });

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol#L32-L39

## [LOW-02] Execution at deadlines should be allowed

The condition may be wrong in these cases, as when block.timestamp is equal to the compared > or < variable these blocks will not be executed.

```solidity
file:  src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

167    if (block.timestamp < thisElectionStartTs) {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L167

## [LOW-03] Arrays can grow in size without a way to shrink them

It's a good practice to maintain control over the size of array state variables in Solidity, especially if they are dynamically updated. If a contract includes a mechanism to push items into an array, it should ideally also provide a mechanism to remove items. This is because Solidity arrays don't automatically shrink when items are deleted - their length needs to be manually adjusted.

Ignoring this can lead to bloated and inefficient contracts. For instance, iterating over a large array can cause your contract to hit the block gas limit. Additionally, if entries are only marked for deletion but never actually removed, you may end up dealing with stale or irrelevant data, which can cause logical errors.

Therefore, implementing a method to 'pop' items from arrays helps manage contract's state, improve efficiency and prevent potential issues related to gas limits or stale data. Always ensure to handle potential underflow conditions when popping elements from the array.



```solidity
file:  src/security-council-mgmt/SecurityCouncilManager.sol

59    SecurityCouncilData[] public securityCouncils;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L59

```solidity
file:  src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol

19     ChainAndUpExecLocation[] upgradeExecutors;

36     SecurityCouncilData[] securityCouncils;

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol#L19

## [LOW-04] Emitting storage values instead of the memory one

Emitted values should not be read from storage again. Instead, the existing values from memory should be used

```solidity
file:    src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

 /// @audit election.weightReceived[nominee]

130       emit VoteCastForNominee({
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
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L130-L139

## [LOW-05] Functions calling contracts/addresses with transfer hooks are missing reentrancy guards


While adherence to the check-effects-interaction pattern is commendable, the absence of a reentrancy guard in functions, especially where transfer hooks might be present, can expose the protocol users to risks of read-only reentrancies. Such reentrancy vulnerabilities can be exploited to execute malicious actions even without altering the contract state. Without a reentrancy guard, the only potential mitigation would be to blocklist the entire protocol - an extreme and disruptive measure. Therefore, incorporating a reentrancy guard into these functions is vital to bolster security, as it helps protect against both traditional reentrancy attacks and read-only reentrancies, ensuring robust and safe protocol operations.

```solidity
file:  src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol


66     _transferOwnership(_owner);


```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#66


```solidity
file:   src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol

76     _transferOwnership(_owner);


```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L76

```solidity
file:    src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

112    _transferOwnership(params.owner);

```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L112

## [LOW-06] Use of onlyOwner functions can be lost

In Solidity, renouncing ownership of a contract essentially transfers ownership to the zero address. This is an irreversible operation and has considerable security implications. If the renounceOwnership function is used, the contract will lose the ability to perform any operations that are limited to the owner. This can be problematic if there are any bugs, flaws, or unexpected events that require owner intervention to resolve. Therefore, in some instances, it is better to disable or omit the renounceOwnership function, and instead implement a secure transferOwnership function. This way, if necessary, ownership can be transferred to a new, trusted party without losing the potential for administrative intervention.

```solidity
file:  src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol

55    contract L2SecurityCouncilMgmtFactory is Ownable {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol#L55


## [LOW-07] Unbounded state variable arrays exist within the project which are iterated upon

Unbounded arrays in Solidity can cause gas-related issues due to their potential for excessive growth, leading to increased computational complexity and resource consumption. Since Ethereum's gas fees are determined by the amount of computational effort required to execute a transaction, operations involving large, unbounded arrays can result in prohibitively high costs for users. Additionally, if a function iterates over an unbounded array, it may exceed the gas limit set by the network, causing the transaction to fail. To avoid these issues, developers should use bounded arrays or alternative data structures like mappings, ensuring efficient resource management and a better user experience.

### Findings
Findings are labeled with ' <= FOUND'

```solidity
file:   src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol

121       function _addNominee(uint256 proposalId, address account) internal {
        _elections[proposalId].nominees.push(account);     //  <= FOUND
        _elections[proposalId].isNominee[account] = true;
        emit NewNominee(proposalId, account);
    }
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L121-L125

```solidity
file:    src/security-council-mgmt/SecurityCouncilManager.sol

143        function _addMemberToCohortArray(address _newMember, Cohort _cohort) internal {
        if (_newMember == address(0)) {
            revert ZeroAddress();
        }
        address[] storage cohort = _cohort == Cohort.FIRST ? firstCohort : secondCohort;
        if (cohort.length == cohortSize) {
            revert CohortFull({cohort: _cohort});
        }
        if (firstCohortIncludes(_newMember)) {
            revert MemberInCohort({member: _newMember, cohort: Cohort.FIRST});
        }
        if (secondCohortIncludes(_newMember)) {
            revert MemberInCohort({member: _newMember, cohort: Cohort.SECOND});
        }

        cohort.push(_newMember);    //  <= FOUND
    }

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L143-159



## [LOW-08] For loops in public or external functions should be avoided due to high gas costs and possible DOS

In Solidity, for loops can potentially cause Denial of Service (DoS) attacks if not handled carefully. DoS attacks can occur when an attacker intentionally exploits the gas cost of a function, causing it to run out of gas or making it too expensive for other users to call. Below are some scenarios where for loops can lead to DoS attacks: Nested for loops can become exceptionally gas expensive and should be used sparingly

### Findings
Findings are labeled with ' <= FOUND'


```solidity
file:   src/UpgradeExecRouteBuilder.sol

76      for (uint256 i = 0; i < _upgradeExecutors.length; i++) {
            ChainAndUpExecLocation memory chainAndUpExecLocation = _upgradeExecutors[i];
            if (chainAndUpExecLocation.location.upgradeExecutor == address(0)) {
                revert ZeroAddress();
            }
            if (upExecLocationExists(chainAndUpExecLocation.chainId)) {        // <= FOUND
                revert UpgradeExecAlreadyExists(chainAndUpExecLocation.chainId);
            }
            upExecLocations[chainAndUpExecLocation.chainId] = chainAndUpExecLocation.location;
        }

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L76-L85


## [LOW-09] Functions which set address state variables should have zero address checks

```solidity
file:   src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

246       function setNomineeVetter(address _nomineeVetter) external onlyGovernance {
        address oldNomineeVetter = nomineeVetter;
        nomineeVetter = _nomineeVetter;
        emit NomineeVetterChanged(oldNomineeVetter, _nomineeVetter);
    }

290      function includeNominee(uint256 proposalId, address account) external onlyNomineeVetter {
        ProposalState state_ = state(proposalId);
        if (state_ != ProposalState.Succeeded) {
            revert ProposalNotSucceededState(state_);
        }

        if (isNominee(proposalId, account)) {
            revert NomineeAlreadyAdded(account);
        }

        uint256 cnCount = compliantNomineeCount(proposalId);
        uint256 cohortSize = securityCouncilManager.cohortSize();
        if (cnCount >= cohortSize) {
            revert CompliantNomineeTargetHit(cnCount, cohortSize);
        }

        // can't include nominees from the other cohort (the cohort not currently up for election)
        // this only checks against the current the current other cohort, and against the current cohort membership
        // in the security council, so changes to those will mean this check will be inconsistent.
        // this check then is only a relevant check when the elections are running as expected - one at a time,
        // every 6 months. Updates to the sec council manager using methods other than replaceCohort can effect this check
        // and it's expected that the entity making those updates understands this.
        if (securityCouncilManager.cohortIncludes(otherCohort(), account)) {
            revert AccountInOtherCohort(otherCohort(), account);
        }

        _addNominee(proposalId, account);
    }

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L246-L250


```solidity
file:    src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol
 
121       function _addNominee(uint256 proposalId, address account) internal {
        _elections[proposalId].nominees.push(account);
        _elections[proposalId].isNominee[account] = true;
        emit NewNominee(proposalId, account);
    }


```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L121-L125





## [LOW-10] Potential division by zero should have zero checks in place

### Findings
Findings are labeled with ' <= FOUND'



```solidity
file:   src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

225        function votesToWeight(uint256 proposalId, uint256 blockNumber, uint256 votes)
        public
        view
        returns (uint240)
    {
        // Before proposalSnapshot all votes have 0 weight
        uint256 startBlock = proposalSnapshot(proposalId);
        if (blockNumber <= startBlock) {
            return 0;
        }
        // After proposalDeadline all votes have zero weight
        uint256 endBlock = proposalDeadline(proposalId);
        if (blockNumber > endBlock) {
            return 0;
        }

        // Between proposalSnapshot and fullWeightVotingDeadline all votes will have 100% weight - each vote has weight 1
        uint256 fullWeightVotingDeadline_ = fullWeightVotingDeadline(proposalId);
        if (blockNumber <= fullWeightVotingDeadline_) {
            return _downCast(votes);
        }

        // Between the fullWeightVotingDeadline and the proposalDeadline each vote will have weight linearly decreased by time since fullWeightVotingDeadline
        // slope denominator
        uint256 decreasingWeightDuration = endBlock - fullWeightVotingDeadline_;
        // slope numerator is -votes, slope denominator is decreasingWeightDuration, delta x is blockNumber - fullWeightVotingDeadline_
        // y intercept is votes
        uint256 decreaseAmount =
            votes * (blockNumber - fullWeightVotingDeadline_) / decreasingWeightDuration;  // <= FOUND
        // subtract the decreased amount to get the remaining weight
        return _downCast(votes - decreaseAmount);
    }

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L225-L256

```solidity
file:   src/security-council-mgmt/governors/modules/ArbitrumGovernorVotesQuorumFractionUpgradeable.sol

37  function quorum(uint256 blockNumber) public view virtual override returns (uint256) {
        return (getPastCirculatingSupply(blockNumber) * quorumNumerator(blockNumber))
            / quorumDenominator();     // <= FOUND
    }

```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/ArbitrumGovernorVotesQuorumFractionUpgradeable.sol#L37-L40

## [LOW-11] Constructors contains no validation

In Solidity, when values are being assigned in constructors to unsigned or integer variables, it's crucial to ensure the provided values adhere to the protocol's specific operational boundaries as laid out in the project specifications and documentation. If the constructors lack appropriate validation checks, there's a risk of setting state variables with values that could cause unexpected and potentially detrimental behavior within the contract's operations, violating the intended logic of the protocol. This can compromise the contract's security and impact the maintainability and reliability of the system. In order to avoid such issues, it is recommended to incorporate rigorous validation checks in constructors. These checks should align with the project's defined rules and constraints, making use of Solidity's built-in require function to enforce these conditions. If the validation checks fail, the require function will cause the transaction to revert, ensuring the integrity and adherence to the protocol's expected behavior.


```solidity
file:   src/gov-action-contracts/execution-record/ActionExecutionRecord.sol

18     constructor(KeyValueStore _store, string memory _uniqueActionName) {
        store = _store;
        actionContractId = keccak256(bytes(_uniqueActionName));
    }

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/execution-record/ActionExecutionRecord.sol#L18-L21


```solidity
file:   src/gov-action-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol

25       constructor(
        IGnosisSafe _newEmergencySecurityCouncil,
        IGnosisSafe _newNonEmergencySecurityCouncil,
        IGnosisSafe _prevEmergencySecurityCouncil,
        IGnosisSafe _prevNonEmergencySecurityCouncil,
        uint256 _emergencySecurityCouncilThreshold,
        uint256 _nonEmergencySecurityCouncilThreshold,
        address _securityCouncilManager,
        IL2AddressRegistry _l2AddressRegistry
    ) {
        newEmergencySecurityCouncil = _newEmergencySecurityCouncil;
        newNonEmergencySecurityCouncil = _newNonEmergencySecurityCouncil;

        prevEmergencySecurityCouncil = _prevEmergencySecurityCouncil;
        prevNonEmergencySecurityCouncil = _prevNonEmergencySecurityCouncil;

        emergencySecurityCouncilThreshold = _emergencySecurityCouncilThreshold;
        nonEmergencySecurityCouncilThreshold = _nonEmergencySecurityCouncilThreshold;

        securityCouncilManager = _securityCouncilManager;
        l2AddressRegistry = _l2AddressRegistry;
    }

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol#L25-L46



## [LOW-12] Array lengths not checked

If the length of the arrays are not required to be of the same length, user operations may not be fully executed due to a mismatch in the number of items iterated over, versus the number of items provided in the second array


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
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L89-L96