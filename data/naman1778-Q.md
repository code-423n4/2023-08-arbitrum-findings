## [N-01] Lack of address(0) checks in the constructor

Zero-address check should be used in the constructors, to avoid the risk of setting smth as address(0) at deploying time.

There is 1 instance of this issue in 1 file:

    File: GovernanceChainSCMgmtActivationAction.sol	

    25: constructor(

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol

## [N-02] *0 address* check for *asset*

Also check of the address to protect the code from 0x0 address problem just in case. This is best practice or instead of suggesting that they verify address *!= 0x0*, you could add some good NatSpec comments explaining what is valid and what is invalid and what are the implications of accidentally using an invalid address.

There are 23 instances of this issue in 7 files:

    File: SecurityCouncilManager.sol	

    /// @audit _member
    161: function _removeMemberFromCohortArray(address _member) internal returns (Cohort) {

    /// @audit _newMember
    176: function addMember(address _newMember, Cohort _cohort) external onlyRole(MEMBER_ADDER_ROLE) {

    /// @audit _memberToReplace, _newMember
    193: function replaceMember(address _memberToReplace, address _newMember)
    194:     external
    195:     onlyRole(MEMBER_REPLACER_ROLE)

    /// @audit _currentAddress, _newAddress
    206: function rotateMember(address _currentAddress, address _newAddress)
    207:     external
    208:     onlyRole(MEMBER_ROTATOR_ROLE)

    /// @audit account
    354: function firstCohortIncludes(address account) public view returns (bool) {

    /// @audit account
    358: function secondCohortIncludes(address account) public view returns (bool) {

    /// @audit account
    364: function cohortIncludes(Cohort cohort, address account) public view returns (bool) {

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol

    File: SecurityCouncilNomineeElectionGovernor.sol	

    /// @audit _nomineeVetter
    246: function setNomineeVetter(address _nomineeVetter) external onlyGovernance {

    /// @audit target
    254: function relay(address target, uint256 value, bytes calldata data)

    /// @audit nominee
    266: function excludeNominee(uint256 proposalId, address nominee)

    /// @audit account
    290: function includeNominee(uint256 proposalId, address account) external onlyNomineeVetter {

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

    File: SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol	

    /// @audit account
    62: function _countVote(
    63:     uint256 proposalId,
    64:     address account,
    65:     uint8 support,
    66:     uint256 weight,
    67:     bytes memory params
    68: ) internal virtual override {

    121: function _addNominee(uint256 proposalId, address account) internal {

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol

    File: SecurityCouncilMemberElectionGovernor.sol	

    /// @audit _owner
    48: function initialize(
    49:     ISecurityCouncilNomineeElectionGovernor _nomineeElectionGovernor,
    50:     ISecurityCouncilManager _securityCouncilManager,
    51:     IVotesUpgradeable _token,
    52:     address _owner,
    53:     uint256 _votingPeriod,
    54:     uint256 _fullWeightDuration
    55: ) public initializer {

    /// @audit target
    103: function relay(address target, uint256 value, bytes calldata data)

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol

    File: SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol	

    /// @audit account
    95: function _countVote(
    96:     uint256 proposalId,
    97:     address account,
    98:     uint8 support,
    99:     uint256 availableVotes,
    100:    bytes memory params
    101: ) internal virtual override {

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

    File: SecurityCouncilMemberRemovalGovernor.sol	

    /// @audit _owner
    58: function initialize(
    59:     uint256 _voteSuccessNumerator,
    60:     ISecurityCouncilManager _securityCouncilManager,
    61:     IVotesUpgradeable _token,
    62:     address _owner,
    63:     uint256 _votingDelay,
    64:     uint256 _votingPeriod,
    65:     uint256 _quorumNumerator,
    66:     uint256 _proposalThreshold,
    67:     uint64 _minPeriodAfterQuorum,
    68:     uint256 _proposalExpirationBlocks
    69: ) public initializer {

    /// @audit account
    169: function _countVote(
    170:     uint256 proposalId,
    171:     address account,
    172:     uint8 support,
    173:     uint256 weight,
    174:     bytes memory params
    175: ) internal virtual override(GovernorCountingSimpleUpgradeable, GovernorUpgradeable) {

    /// @audit target
    203: function relay(address target, uint256 value, bytes calldata data)

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol

    File: SecurityCouncilMemberSyncAction.sol	

    /// @audit _member
    76: function _addMember(IGnosisSafe securityCouncil, address _member, uint256 _threshold)

    /// @audit _member
    85: function _removeMember(IGnosisSafe securityCouncil, address _member, uint256 _threshold)

    /// @audit securityCouncil
    97: function getPrevOwner(IGnosisSafe securityCouncil, address _owner)

    /// @audit securityCouncil 
    122: function _setUpdateNonce(address securityCouncil, uint256 nonce)

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol

## [N-03] Contract Owner Has Too Many Privileges

The owner of the contracts has too many privileges relative to standard users. The consequence is disastrous if the contract owner’s private key has been compromised. And, in the event the key was lost or unrecoverable, no implementation upgrades and system parameter updates will ever be possible.

For a project this grand, it increases the likelihood that the owner will be targeted by an attacker, especially given the insufficient protection on sensitive owner private keys. The concentration of privileges creates a single point of failure; and, here are some of the incidents that could possibly transpire:

Transfer ownership and mess up with all the setter functions, hijacking the entire protocol.

Consider:

--splitting privileges (e.g. via the multisig option) to ensure that no one address has excessive ownership of the system,
--clearly documenting the functions and implementations the owner can change,
--documenting the risks associated with privileged users and single points of failure, and
--ensuring that users are aware of all the risks associated with the system.

## [N-04] Empty/Unused Function Parameters

Empty or unused function parameters should be commented out as a better and declarative way to silence runtime warning messages.

There are 11 instances of this issue in 4 files:

    File: SecurityCouncilNomineeElectionGovernor.sol	

    423: function propose(address[] memory, uint256[] memory, bytes[] memory, string memory)

    433: function castVote(uint256, uint8) public virtual override returns

    438: function castVoteWithReason(uint256, uint8, string calldata)

    448: function castVoteBySig(uint256, uint8, uint8, bytes32, bytes32)

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

    File: SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol	    

    170: function _quorumReached(uint256) internal pure override returns
    
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol

    File: SecurityCouncilMemberElectionGovernor.sol	

    181: function propose(address[] memory, uint256[] memory, bytes[] memory, string memory)

    191: function castVote(uint256, uint8) public virtual override returns (uint256) {

    196: function castVoteWithReason(uint256, uint8, string calldata)

    206: function castVoteBySig(uint256, uint8, uint8, bytes32, bytes32)

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol

    File: SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol	

    267: function _quorumReached(uint256) internal pure override returns (bool) {

    273: function _voteSucceeded(uint256) internal pure override returns (bool) {

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

