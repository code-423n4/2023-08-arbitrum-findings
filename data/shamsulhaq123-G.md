## Gas optimization 

No | Issue |Instances|
|-|:-|:-:|:-:|
|G-01|Use calldata instead of memory for function parameters|20
|G-02|Use functions instead of modifiers|2
|G-03|Use assembly in place of abi.decode to extract calldata values more efficiently|4
|G-04|abi.encode() is less efficient than abi.encodePacked()|4
|G-05|Use assembly to validate msg.sender|5
|G-06|Using bools for storage incurs overhead|3
|G-07| Unnecessary look up in if condition|2
|G-08|If statements that use && can be refactored into nested if statements|4
|G-09|>= costs less gas than >|8
|G-10|Use assembly to check for address(0)|9
|G-11| Modifiers are redundant if used only once or not used at all. |1
|G-12|Use constants instead of type(uintx).max|1
|G-13|Use != 0 instead of > 0 for unsigned integer comparison|2
|G-14|Bytes constants are more efficient than String constants|11
|G-15|Use do while loops instead of for loops|20
|G-16|Avoid emitting constants|18

### [G-01]  Use calldata instead of memory for function parameters
If a reference type function parameter is read-only, it is cheaper in gas to use calldata instead of memory. Calldata is a non-modifiable, non-persistent area where function arguments are stored, and behaves mostly like memory.

Note that I’ve also flagged instances where the function is public but can be marked as external since it’s not called by the contract.

```solidity

file: security-council-mgmt/SecurityCouncilManager.sol

88   function initialize(
        address[] memory _firstCohort,
        address[] memory _secondCohort,
        SecurityCouncilData[] memory _securityCouncils,
        SecurityCouncilManagerRoles memory _roles,
        address payable _l2CoreGovTimelock,
        UpgradeExecRouteBuilder _router
96    )

103 function initialize(InitParams memory params) public initializer 

324     function _execute(
        uint256 proposalId,
        address[] memory, /* targets */
        uint256[] memory, /* values */
        bytes[] memory callDatas,
        bytes32 /*descriptionHash*/
330    )

423     function propose(address[] memory, uint256[] memory, bytes[] memory, string memory)
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L88-L96
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L103
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L324-L330
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L423

```solidity

file: security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol

62    function _countVote(
        uint256 proposalId,
        address account,
        uint8 support,
        uint256 weight,
        bytes memory params
68     )


```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L62-68

```solidity

file: security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol

28    function __SecurityCouncilNomineeElectionGovernorTiming_init(
        Date memory _firstNominationStartDate,
        uint256 _nomineeVettingDuration
32    )


```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol#L28-L32

```solidity

file: security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol

115    function _execute(
        uint256 proposalId,
        address[] memory, /* targets */
        uint256[] memory, /* values */
        bytes[] memory callDatas,
        bytes32 /* descriptionHash */
121    ) 

181   function propose(address[] memory, uint256[] memory, bytes[] memory, string memory)
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L115-L121
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L181

```solidity

file: security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

95    function _countVote(
        uint256 proposalId,
        address account,
        uint8 support,
        uint256 availableVotes,
        bytes memory params
101    )

191   function selectTopNominees(address[] memory nominees, uint240[] memory weights, uint256 k)
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L95-L101
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L191

```solidity

file: security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol

106   function propose(
        address[] memory targets,
        uint256[] memory values,
        bytes[] memory calldatas,
        string memory description
111    )

163     function _countVote(
        uint256 proposalId,
        address account,
        uint8 support,
        uint256 weight,
        bytes memory params
169     )

223    function _castVote(
        uint256 proposalId,
        address account,
        uint8 support,
        string memory reason,
        bytes memory params
229    )
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L106-L111
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L163-L169
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L223-L229

```solidity

file: security-council-mgmt/governors/modules/ElectionGovernor.sol

34   function extractElectionIndex(bytes[] memory callDatas) internal pure returns (uint256) {
        return abi.decode(callDatas[0], (uint256));
    }



```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/ElectionGovernor.sol#L34

```solidity

file: UpgradeExecRouteBuilder.sol

105   function createActionRouteData(
        uint256[] memory chainIds,
        address[] memory actionAddresses,
        uint256[] memory actionValues,
        bytes[] memory actionDatas,
        bytes32 predecessor,
        bytes32 timelockSalt
112    )
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L105-L112
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L186-L190

```solidity

file: security-council-mgmt/SecurityCouncilMemberSyncAction.sol

31  function perform(address _securityCouncil, address[] memory _updatedMembers, uint256 _nonce)

127  function _execFromModule(IGnosisSafe securityCouncil, bytes memory data) internal 
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L31
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L127

```solidity

file: ecurity-council-mgmt/SecurityCouncilMgmtUtils.so

5  function isInArray(address addr, address[] memory arr) internal pure returns (bool)

15  function filterAddressesWithExcludeList(
        address[] memory input,
        mapping(address => bool) storage excludeList
18    )
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol#L5
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol#L15-L18

### [G-02] Use functions instead of modifiers

```solidity

file: security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

134   modifier onlyNomineeVetter() {
        if (msg.sender != nomineeVetter) {
            revert OnlyNomineeVetter();
        }
        _;
    }

    /// @notice Some operations can only be performed during the vetting period.
142    modifier onlyVettingPeriod(uint256 proposalId)

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L134

```solidity

file: security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol

78   modifier onlyNomineeElectionGovernor() {
        if (msg.sender != address(nomineeElectionGovernor)) {
            revert OnlyNomineeElectionGovernor();
        }
        _;
83    }

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L78-L83

### [G-03] Use assembly in place of abi.decode to extract calldata values more efficiently
Instead of using abi.decode, we can use assembly to decode our desired calldata values directly. This will allow us to avoid decoding calldata values that we will not use.

```solidity

file: security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol

78    (address contender, uint256 votes) = abi.decode(params, (address, uint256))

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L78

```solidity

file: security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

110   (address nominee, uint256 votes) = abi.decode(params, (address, uint256));


```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L110

```solidity

file: security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.so

133     address memberToRemove = abi.decode(rest, (address));

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L133

```solidity

file: security-council-mgmt/governors/modules/ElectionGovernor.sol

35   return abi.decode(callDatas[0], (uint256));

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/ElectionGovernor.sol#L35

### [G-04] abi.encode() is less efficient than abi.encodePacked()
use abi.encodePacked() where possible to save gas

```solidity

file: security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol

78    (address contender, uint256 votes) = abi.decode(params, (address, uint256))

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L78

```solidity

file: security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

110   (address nominee, uint256 votes) = abi.decode(params, (address, uint256));


```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L110

```solidity

file: security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.so

133     address memberToRemove = abi.decode(rest, (address));

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L133

```solidity

file: security-council-mgmt/governors/modules/ElectionGovernor.sol

35   return abi.decode(callDatas[0], (uint256));

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/ElectionGovernor.sol#L35

### [G-05] Use assembly to validate msg.sender
We can use assembly to efficiently validate msg.sender

```solidity

file: security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.so

135 if (msg.sender != nomineeVetter)

221   if (election.isContender[msg.sender]) {
222            revert AlreadyContender(msg.sender);

236    if (securityCouncilManager.cohortIncludes(otherCohort(), msg.sender)) {
237            revert AccountInOtherCohort(otherCohort(), msg.sender);

240   election.isContender[msg.sender] = true;

242      emit ContenderAdded(proposalId, msg.sender);
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L135
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L221-L222
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L236-L237
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L240-L242

```solidity

file: security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol

79  if (msg.sender != address(nomineeElectionGovernor))

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L79

### [G-06] Using bools for storage incurs overhead
Both state variables can potentially be set back and forth from true and false. This would result in a Gsset (20000 gas) everytime the values are set to true from false. We can instead use uint(1) in place of true and uint(2) in place of false and pay the Gsset (20000 gas) once during deployment (to set the slot to uint(1). This would save two Gsset (20000 gas). However, a more efficient mitigation would be to pack the variables into a slot with other variables that would inevitably be written to. 

```solidity

file: ecurity-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

55  mapping(address => bool) isContender;
56  mapping(address => bool) isExcluded;


```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L55-L56

```solidity

file: security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol

22   mapping(address => bool) isNominee;
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L22

```solidity

file: security-council-mgmt/SecurityCouncilMgmtUtils.sol

17   mapping(address => bool) storage excludeList

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol#L17

### [G-07]  Unnecessary look up in if condition
If the || condition isn’t required, the second condition will have been looked up unnecessarily.


```solidity

file: security-council-mgmt/SecurityCouncilManager.sol

222    if (_addressToRemove == address(0) || _addressToAdd == address(0)) 

236 
        if (
            _securityCouncilData.updateAction == address(0)
238          || _securityCouncilData.securityCouncil == address(0)
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L222
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L236-L238

### [G-08] If statements that use && can be refactored into nested if statements

```solidity

file: security-council-mgmt/SecurityCouncilManager.sol

254  if (
                existantSecurityCouncil.chainId == _securityCouncilData.chainId
                    && existantSecurityCouncil.securityCouncil == _securityCouncilData.securityCouncil
257            )


286     if (
                securityCouncilData.securityCouncil == _securityCouncilData.securityCouncil
                    && securityCouncilData.chainId == _securityCouncilData.chainId
                    && securityCouncilData.updateAction == _securityCouncilData.updateAction
290           ) 
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L254-L257 
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L286L290

```solidity

file: security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol

134    if (
            !securityCouncilManager.firstCohortIncludes(memberToRemove)
                && !securityCouncilManager.secondCohortIncludes(memberToRemove)
137        )

183    if (!(0 < _voteSuccessNumerator && _voteSuccessNumerator <= voteSuccessDenominator))
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L134-L137
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L183

### [G-09] >= costs less gas than >
The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas

```solidity

file: security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

128  if ((quorumDenominator() / params.quorumNumeratorValue) > 500) 

151  if (block.number > vettingDeadline) 
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L128
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L151

```solidity

file: security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol

56  if (_fullWeightDuration > _votingPeriod) 
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L56

```solidity

file: security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

78  if (newFullWeightDuration > votingPeriod())

122   if (prevVotesUsed + votes > availableVotes)

134   return _elections[proposalId].votesUsed[account] > 0;

237  if (blockNumber > endBlock) 

260   if (x > type(uint240).max) 
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L78
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L122
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L134
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L237
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L260

### [G-10] Use assembly to check for address(0)


```solidity

file: security-council-mgmt/SecurityCouncilManager.sol

144   if (_newMember == address(0)) {

184    if (_member == address(0))  

222     if (_addressToRemove == address(0) || _addressToAdd == address(0)) 

236  if (
            _securityCouncilData.updateAction == address(0)
                || _securityCouncilData.securityCouncil == address(0)
239        ) ecurityCouncilData.securityCouncil == address(0)
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L144
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L184
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L222
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L236-L239

```solidity

file: UpgradeExecRouteBuilder.sol

72   if (_l1ArbitrumTimelock == address(0)) 

78      if (chainAndUpExecLocation.location.upgradeExecutor == address(0)) 

94   return upExecLocations[_chainId].upgradeExecutor != address(0);

131 if (upExecLocation.upgradeExecutor == address(0)) 

143   if (upExecLocation.inbox == address(0)) 
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L72
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L78
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L94
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L131
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L143

### [G-11] Modifiers are redundant if used only once or not used at all. 
• Deployment. Gas Saved: 33 649
• Minumal Method Call. Gas Saved: -202
• Average Method Call. Gas Saved: 2 700
• Maximum Method Call. Gas Saved: 4 931

```solidity

file: security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

134   modifier onlyNomineeVetter() {
        if (msg.sender != nomineeVetter) {
            revert OnlyNomineeVetter();
        }
        _;
    }

    /// @notice Some operations can only be performed during the vetting period.
142    modifier onlyVettingPeriod(uint256 proposalId) 
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L134-L142

### [G-12] Use constants instead of type(uintx).max

type(uint120).max or type(uint128).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.


```solidity

file: security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#

260  if (x > type(uint240).max) 
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L260

## [G-13] Use != 0 instead of > 0 for unsigned integer comparison

it's generally more gas-efficient to use != 0 instead of > 0 when
comparing unsigned integer types.

This is because the Solidity compiler can optimize the != 0 comparison to a simple bitwise operation,
 while the > 0 comparison requires an additional subtraction operation.
  As a result, using != 0 can be more gas-efficient and can help to reduce the overall cost of your contract.

Here's an example of how you can use != 0 instead of > 0:
``` solidity
contract MyContract {
    uint256 public myUnsignedInteger;
    
    function myFunction() public view returns (bool) {
        // Use != 0 instead of > 0
        return myUnsignedInteger != 0;
    }
}

```

```solidity

file: security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol
Use constants instead of type(uintx).max
134   return _elections[proposalId].votesUsed[account] > 0;
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L134

```solidity

file: security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

159     return votesUsed(proposalId, account) > 0;
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L159

### [G-14] Bytes constants are more efficient than String constants

If data can fit into 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is cheaper in solidity.

Use assembly to perform efficient back-to-back calls
```solidity

file: security-council-mgmt/governors/modules/ElectionGovernor.sol

40     function electionIndexToDescription(uint256 electionIndex)
        public
        pure
        returns (string memory)
    {
        return
            string.concat("Security Council Election #", StringsUpgradeable.toString(electionIndex));
47    }
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/ElectionGovernor.sol#L40-L47

```solidity

file: gov-action-contracts/execution-record/ActionExecutionRecord.sol

18  constructor(KeyValueStore _store, string memory _uniqueActionName)
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/execution-record/ActionExecutionRecord.sol#L18


```solidity

file: security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

175    string memory description

196   string memory description
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L175
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L196


```solidity

file: security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

423     function propose(address[] memory, uint256[] memory, bytes[] memory, string memory)

438   function castVoteWithReason(uint256, uint8, string calldata)
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L423
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L438

```solidity

file: security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol

128   function COUNTING_MODE() public pure virtual override returns (string memory) 
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L128

```solidity

file: security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#

96    string memory description

181   function propose(address[] memory, uint256[] memory, bytes[] memory, string memory)

196   function castVoteWithReason(uint256, uint8, string calldata)
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L96
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L181
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L196

```solidity

file: security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

143   function COUNTING_MODE() public pure virtual override returns (string memory) 
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L143

### [G-15] Use do while loops instead of for loops

A do while loop will cost less gas since the condition is not being checked for the first iteration.

```solidity

file: UpgradeExecRouteBuilder.sol

76  for (uint256 i = 0; i < _upgradeExecutors.length; i++) 

129   for (uint256 i = 0; i < chainIds.length; i++) 

193 for (uint256 i = 0; i < chainIds.length; i++) 
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L76
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L129
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L193

```solidity

file: security-council-mgmt/SecurityCouncilMemberSyncAction.sol

60      for (uint256 i = 0; i < _updatedMembers.length; i++) {
            address member = _updatedMembers[i];
            if (!securityCouncil.isOwner(member)) {
                _addMember(securityCouncil, member, threshold);
            }
        }

        for (uint256 i = 0; i < previousOwners.length; i++) {
            address owner = previousOwners[i];
            if (!SecurityCouncilMgmtUtils.isInArray(owner, _updatedMembers)) {
                _removeMember(securityCouncil, owner, threshold);
            }
72        }

105   for (uint256 i = 0; i < owners.length; i++) 
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L60-L72
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L105

```solidity

file: security-council-mgmt/SecurityCouncilMgmtUtils.sol

6    for (uint256 i = 0; i < arr.length; i++) {
            if (arr[i] == addr) {
                return true;
            }

22     for (uint256 i = 0; i < input.length; i++) {
            address nominee = input[i];
            if (!excludeList[nominee]) {
                intermediate[intermediateLength] = nominee;
                intermediateLength++;
            }
        }

        address[] memory output = new address[](intermediateLength);
        for (uint256 i = 0; i < intermediateLength; i++) {
            output[i] = intermediate[i];
33        }
``` 
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol#L6
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol#L22-L33

```solidity

file: security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol

111     for (uint256 i = 0; i < dp.firstCohort.length; i++) {
            if (!govChainEmergencySCSafe.isOwner(dp.firstCohort[i])) {
                revert AddressNotInCouncil(owners, dp.firstCohort[i]);
            }
        }

        for (uint256 i = 0; i < dp.secondCohort.length; i++) {
            if (!govChainEmergencySCSafe.isOwner(dp.secondCohort[i])) {
                revert AddressNotInCouncil(owners, dp.secondCohort[i]);
            }
121        }

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol#L111-L121

```solidity

file: gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol

61   for (uint256 i = 0; i < array1.length; i++) {
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

        for (uint256 i = 0; i < array2.length; i++) {
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
81       }
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L61-L81

```solidity

file: security-council-mgmt/SecurityCouncilManager.sol

106   for (uint256 i = 0; i < _roles.memberRemovers.length; i++) 

118  for (uint256 i = 0; i < _securityCouncils.length; i++) 

135    for (uint256 i = 0; i < _newCohort.length; i++)

162    for (uint256 i = 0; i < 2; i++) {
            address[] storage cohort = i == 0 ? firstCohort : secondCohort;
            for (uint256 j = 0; j < cohort.length; j++) {
                if (_member == cohort[j]) {
                    cohort[j] = cohort[cohort.length - 1];
                    cohort.pop();
                    return i == 0 ? Cohort.FIRST : Cohort.SECOND;
169                }

251 for (uint256 i = 0; i < securityCouncils.length; i++)

284  for (uint256 i = 0; i < securityCouncils.length; i++)

339        for (uint256 i = 0; i < firstCohort.length; i++) {
            members[i] = firstCohort[i];
        }
        for (uint256 i = 0; i < secondCohort.length; i++) {
            members[firstCohort.length + i] = secondCohort[i];
344        }

392       for (uint256 i = 0; i < securityCouncils.length; i++) {
            SecurityCouncilData memory securityCouncilData = securityCouncils[i];
            actionAddresses[i] = securityCouncilData.updateAction;
            chainIds[i] = securityCouncilData.chainId;
            actionDatas[i] = abi.encodeWithSelector(
                SecurityCouncilMemberSyncAction.perform.selector,
                securityCouncilData.securityCouncil,
                newMembers,
                nonce
            );
402        }
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L106
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/
SecurityCouncilManager.sol#L118
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L135
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L162-L169
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L251
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L284
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L339-L344
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L392-L402

```solidity

file: security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

181   for (uint256 i = 0; i < nominees.length; i++) 

205   for (uint16 i = 0; i < nominees.length; i++) 

215   for (uint16 i = 0; i < k; i++) 
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L181
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L205
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L215

### [G-16] Avoid emitting constants.
A log topic (declared with indexed) has a gas cost of Glogtopic (375 gas).

```solidity

file: security-council-mgmt/SecurityCouncilMemberSyncAction.sol

46       emit UpdateNonceTooLow(_securityCouncil, updateNonce, _nonce);
            return false;
        }
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L46

```solidity

file: security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol

187       emit ContractsDeployed(deployedContracts);
        return deployedContracts;
    }
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol#L187

```solidity

file: security-council-mgmt/SecurityCouncilManager.sol

140   emit CohortReplaced(_newCohort, _cohort);

179   emit MemberAdded(_newMember, _cohort);

189   emit MemberRemoved({member: _member, cohort: cohort});

198      emit MemberReplaced({
            replacedMember: _memberToReplace,
            newMember: _newMember,
            cohort: cohort
        });

211     emit MemberRotated({Avoid emitting constants.s,
            newAddress: _newAddress,
            cohort: cohort
        });    

263        emit SecurityCouncilAdded(
            _securityCouncilData.securityCouncil,
            _securityCouncilData.updateAction,
            securityCouncils.length
        );   emit VoteCastForNominee({
            voter: account,
            proposalId: proposalId,
            nominee: nominee,
            votes: votes,
            weight: weight,
            totalUsedVotes: prevVotesUsed + votes,
            usableVotes: availableVotes,
            weightReceived: election.weightReceived[nominee]
        });
    }    

296        emit SecurityCouncilRemoved(
                    securityCouncilData.securityCouncil,
                    securityCouncilData.updateAction,
                    securityCouncils.length
                );
                return true;
            }  

323       emit UpgradeExecRouteBuilderSet(routerAddress);
    }
                      
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L140
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L179
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L189
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L198
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L211
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L263
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L296
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L323

```solidity

file: security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

242    emit ContenderAdded(proposalId, msg.sender);

249      emit NomineeVetterChanged(oldNomineeVetter, _nomineeVetter);
    }

282 emit NomineeExcluded(proposalId, nominee);    
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L242
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L249
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L282

```solidity

file: security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol

110        emit VoteCastForContender({
            proposalId: proposalId,
            voter: account,
            contender: contender,
            votes: actualVotes,
            totalUsedVotes: prevVotesUsed + actualVotes,
            usableVotes: weight
        });
    

124      emit NewNominee(proposalId, account);
    
```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L110
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L124

```solidity

file: security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

73      emit FullWeightDurationSet(initialFullWeightDuration);

83    emit FullWeightDurationSet(newFullWeightDuration);

130    emit VoteCastForNominee({
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
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L73
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L83
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L130