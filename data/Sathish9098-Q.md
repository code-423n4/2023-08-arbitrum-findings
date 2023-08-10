# LOW FINDINGS

##

## [L-1] ``block.timestamp < thisElectionStartTs`` check allows to create elections even ``6 months`` not ends

### Impact

This check ``potentially break the docs`` . This will allow to ``createElection`` even 6 month not yet ends

If the condition ``block.timestamp < thisElectionStartTs`` is used and ``block.timestamp`` is equal to ``thisElectionStartTs``, then the condition would evaluate to ``false``, and the transaction would not revert. This means that the election could be created even if the desired start time ``thisElectionStartTs`` has already been reached but not yet ended. 

### POC
```solidity
FILE: governance/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

166: uint256 thisElectionStartTs = electionToTimestamp(electionCount);
167:        if (block.timestamp < thisElectionStartTs) {
168:            revert CreateTooEarly(block.timestamp, thisElectionStartTs);
169:        }

```
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L166-L169

### Recommended Mitigation
Change the checks like,

```solidity

167:  if (block.timestamp <= thisElectionStartTs) {

```

##

## [L-2] ``electionCount `` always grow indefinitely, potentially impacting the efficiency and scalability of the network  

### Impact
The Ethereum blockchain has a finite capacity for storing data. If many contracts follow a pattern of unbounded growth for their state variables, it could contribute to blockchain bloat, potentially impacting the efficiency and scalability of the network. As electionCount grows, the execution time of certain transactions might increase. This could lead to longer confirmation times and delays for users interacting with the contract

### POC

```solidity
FILE: governance/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

180: electionCount++;

```
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L180

### Recommended Mitigation
If unbounded growth is necessary, you might need to implement mechanisms to manage and handle large data sets efficiently, such as pagination, data archiving, or off-chain storage solutions.

##

## [L-3] ``getProposeArgs(electionCount - 1)`` this may cause problem when the value is ``1``

### Impact
If ``electionCount`` always starts with 1 and you use the expression ``getProposeArgs(electionCount - 1)``, there could be an issue when the value of ``electionCount ``is ``1``. This is because subtracting 1 from 1 would result in ``0``, and if your contract's data structure is not designed to handle this scenario, it could lead to unexpected behavior or errors.

### POC
```solidity
FILE: governance/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

197: ) = getProposeArgs(electionCount - 1);

```
 https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L197

##

## [L-4] Lack of access control in ``addContender()`` function

### Impact
Anyone could call the ``addContender`` function and add themselves as ``contenders``, even if they shouldn't be allowed to ``participate`` in the election. This could lead to ``manipulation`` of the ``election process``.

### POC
```solidity
FILE: governance/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

function addContender(uint256 proposalId) external {
        ElectionInfo storage election = _elections[proposalId];

        if (election.isContender[msg.sender]) {
            revert AlreadyContender(msg.sender);
        }

        ProposalState state_ = state(proposalId);
        if (state_ != ProposalState.Active) {
            revert ProposalNotActive(state_);
        }

        // check to make sure the contender is not part of the other cohort (the cohort not currently up for election)
        // this only checks against the current the current other cohort, and against the current cohort membership
        // in the security council, so changes to those will mean this check will be inconsistent.
        // this check then is only a relevant check when the elections are running as expected - one at a time,
        // every 6 months. Updates to the sec council manager using methods other than replaceCohort can effect this check
        // and it's expected that the entity making those updates understands this.
        if (securityCouncilManager.cohortIncludes(otherCohort(), msg.sender)) {
            revert AccountInOtherCohort(otherCohort(), msg.sender);
        }

        election.isContender[msg.sender] = true;

        emit ContenderAdded(proposalId, msg.sender);
    }
```
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L218

### Recommended Mitigation
Add access control to critical ``addContender()`` function

##

## [L-5] ``threshold`` not checked .  As per docs ``threshold`` is never lower than the ``member count``

### Impact
The ``threshold`` is an important parameter that controls how many members of the security council must approve an update before it can be executed. If the threshold is too low, it could be easier for an attacker to take control of the ``security council``

### POC
```solidity
FILE: governance/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol

55: // preserve current threshold, the safe ensures that the threshold is never lower than the member count
56: uint256 threshold = securityCouncil.getThreshold();

```
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L55-L56

### Recommended Mitigation
Add ``threshold`` check
 
```solidity

if (threshold < securityCouncil.getOwners().length) {

```

##

## [L-6] Lack of control for critical ``perform()`` function

### Impact
The ``perform`` function does not check if the caller is authorized to perform the update. This is a potential security problem, as it could allow an attacker to call the function and add or remove members from the security council without authorization.


### POC
```solidity
FILE: governance/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol

  /// @notice Updates members of security council multisig to match provided array
    /// @dev    This function contains O(n^2) operations, so doesnt scale for large numbers of members. Expected count is 12, which is acceptable.
    ///         Gnosis OwnerManager handles reverting if address(0) is passed to remove/add owner
    /// @param _securityCouncil The security council to update
    /// @param _updatedMembers  The new list of members. The Security Council will be updated to have this exact list of members
    /// @return res indicates whether an update took place
    function perform(address _securityCouncil, address[] memory _updatedMembers, uint256 _nonce)
        external
        returns (bool res)
    {

```
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L25-L34

### Recommended Mitigation
Add access control to ``perform()`` function

##

## [L-7] The ``replaceCohort`` function does not check if the old cohort is still in use

### Impact
This could allow an ``COHORT_REPLACER_ROLE`` to replace a cohort that is still being used, which could cause the security council to be inoperable.

### POC

```solidity
FILE: Breadcrumbsgovernance/src/security-council-mgmt/SecurityCouncilManager.sol

/// @inheritdoc ISecurityCouncilManager
    function replaceCohort(address[] memory _newCohort, Cohort _cohort)
        external
        onlyRole(COHORT_REPLACER_ROLE)
    {
        if (_newCohort.length != cohortSize) {
            revert InvalidNewCohortLength({cohort: _newCohort, cohortSize: cohortSize});
        }

        // delete the old cohort
        _cohort == Cohort.FIRST ? delete firstCohort : delete secondCohort;

        for (uint256 i = 0; i < _newCohort.length; i++) {
            _addMemberToCohortArray(_newCohort[i], _cohort);
        }

        _scheduleUpdate();
        emit CohortReplaced(_newCohort, _cohort);
    }


```
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L124-L141

### Recommended Mitigation
Before remove or replace any ``cohort`` make sure that ``cohort not in use ``

##

## [L-8] Lack of event emit when adding new member to ``Cohort``

### Impact
The ``_addMemberToCohortArray`` function does not emit an event when a new member is added to a cohort. This could make it difficult to ``track changes`` to the ``security council's members``.

### POC
```solidity
FILE: governance/src/security-council-mgmt/SecurityCouncilManager.sol

function _addMemberToCohortArray(address _newMember, Cohort _cohort) internal {
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

        cohort.push(_newMember);
    }

```
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L143-L159

### Recommended Mitgation
Add event-emit for critical changes 

##



##

## [L-9] ``generateSalt()`` function is not secure 

### Impact
 Even if the ``updateNonce``  value is increased every time a new proposal is scheduled, the ``generateSalt ``function is still not a secure function. This is because the ``generateSalt`` function simply concatenates the ``newMembers`` array and the ``updateNonce`` variable together to create a salt. This salt is not very secure, and it is possible for an attacker to generate a collision and create a valid proposal with the same salt as another proposal

### POC
```solidity

440: salt: this.generateSalt(newMembers, updateNonce),

370: function generateSalt(address[] memory _members, uint256 nonce)
371:        external
372:        pure
373:        returns (bytes32)
374:    {
375:        return keccak256(abi.encodePacked(_members, nonce));
376:    }

```
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L440

### Recommended Mitigation
To make the generateSalt function more secure, you could use a more secure hash function, such as the ``sha3 ``function

##

## [L-10] Division by zero not prevented

### Impact
The divisions below take an input parameter which does not have any zero-value checks, which may lead to the functions reverting when zero is passed. ``params.quorumNumeratorValue`` should be checked before division operation

### POC
```solidity
FILE: governance/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

128: if ((quorumDenominator() / params.quorumNumeratorValue) > 500) {

```
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L128

```solidity
FILE: Breadcrumbsgovernance/src/security-council-mgmt/governors/modules
/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

252: uint256 decreaseAmount =
253:            votes * (blockNumber - fullWeightVotingDeadline_) / decreasingWeightDuration;

```
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L252-L253


### Recommended Mitigation
Add the ``non zero check`` before perform division 

##

## [L-11] Lack of sanity checks when assigning ``uint`` values to critical variables in ``constructor`` and ``initializer`` function

### Impact
The contract's does not have any sanity checks when assigning ``uint`` values to critical variables in the ``constructor`` and ``initializer`` function. This could lead to human errors that would require the contract to be redeployed.

### POC

```solidity
FILE: governance/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol

41: emergencySecurityCouncilThreshold = _emergencySecurityCouncilThreshold;
42: nonEmergencySecurityCouncilThreshold = _nonEmergencySecurityCouncilThreshold;

```
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol#L41-L42

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L63-L75


### Recommended Mitigation
Add ``> 0``, ``MAX``, ``MIN`` value checks 


##

## [L-12] ``Protocol`` uses the ``vulnerable`` version of ``openzeppelin version``

### Impact
The Protocol contract uses the ``vulnerable`` version of the ``openzeppelin`` library. This means that the contract is susceptible to a number of security vulnerabilities that have been patched in newer versions of the library.

Protocol uses the contracts": "4.7.3", contracts-upgradeable": "4.7.3"  there are known vulnerability in this version 

  - Improper Input Validation
  - Missing Authorization
  - Denial of Service (DoS)
  - Improper Verification of Cryptographic Signature

### POC
```solidity
FILE: package.json

69: "@openzeppelin/contracts": "4.7.3",
70: "@openzeppelin/contracts-upgradeable": "4.7.3",

```

### Recommended Mitigation
Use at least ``openzeppelin : 4.9.2 ``

##

## [L-13] Hardcoded ``MAX_SECURITY_COUNCILS`` may cause problem in future 

### Impact
The ``MAX_SECURITY_COUNCILS`` constant in the Protocol contract is hardcoded to a value of ``500``. This means that there can only be a maximum of 500 security councils in the protocol. This could be a problem in the future if the protocol needs to support more than 10 security councils.

If any problem need to ``increase/decrease`` ``security councils`` means its not possible. Need to redeploy the over all contract 

### POC

```solidity
FILE: governance/src/security-council-mgmt/SecurityCouncilManager.sol

67: uint256 public immutable MAX_SECURITY_COUNCILS = 500;

```
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L67

### Recommended Mitigation
Make ``MAX_SECURITY_COUNCILS`` value configurable 

##

## [L-14] Project Upgrade and Stop Scenario should be

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation. This can also be called an ” EMERGENCY STOP (CIRCUIT BREAKER) PATTERN “.

https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol

##

## [L-15] Expressions for constant values such as a call to ``keccak256()``, should use ``immutable`` rather than ``constant``

### Impact

While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand. There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts. constants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor.

```solidity
FILE: Breadcrumbsgovernance/src/security-council-mgmt/SecurityCouncilManager.sol

79:    bytes32 public constant COHORT_REPLACER_ROLE = keccak256("COHORT_REPLACER");
80:    bytes32 public constant MEMBER_ADDER_ROLE = keccak256("MEMBER_ADDER");
81:    bytes32 public constant MEMBER_REPLACER_ROLE = keccak256("MEMBER_REPLACER");
82:    bytes32 public constant MEMBER_ROTATOR_ROLE = keccak256("MEMBER_ROTATOR");
83:    bytes32 public constant MEMBER_REMOVER_ROLE = keccak256("MEMBER_REMOVER");

```
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L79-L83

### Recommended Mitigation
Use immutable instead of constant 

##

## [L-16] Shorter the inheritance list 

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L16-L27

##

## [L-17] initialize() function visibility should be ``external ``

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L58-L69

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L55

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L103




