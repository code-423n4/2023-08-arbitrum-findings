## QA
---

### Layout Order [1]

- The best-practices for layout within a contract is the following order: state variables, events, modifiers, constructor and functions.

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol

```solidity
// place this variable before all the all the errors and events. 
19:    address public constant SENTINEL_OWNERS = address(0x1);
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol

```solidity
// place this modifier before the constructor
78:    modifier onlyNomineeElectionGovernor() {
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

```solidity
// place these modifier before the constructor
134:    modifier onlyNomineeVetter() {
142:    modifier onlyVettingPeriod(uint256 proposalId) {
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol

```solidity
// place these events after state variables
33:    event CohortReplaced(address[] newCohort, Cohort indexed cohort);
34:    event MemberAdded(address indexed newMember, Cohort indexed cohort);
35:    event MemberRemoved(address indexed member, Cohort indexed cohort);
36:    event MemberReplaced(address indexed replacedMember, address indexed newMember, Cohort cohort);
37    event MemberRotated(address indexed replacedAddress, address indexed newAddress, Cohort cohort);
38:    event SecurityCouncilAdded(
41:    event SecurityCouncilRemoved(
44:    event UpgradeExecRouteBuilderSet(address UpgradeExecRouteBuilder);
```

---

### Function Visibility [2]

- Order of Functions: Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private. Within a grouping, place the view and pure functions last.

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/execution-record/ActionExecutionRecord.sol

```solidity
// place these internal for last 
25:    function _set(uint256 key, uint256 value) internal {
31:    function _get(uint256 key) internal view returns (uint256) {
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol

```solidity
// place this public function first 
52:    function areAddressArraysEqual(address[] memory array1, address[] memory array2)
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol

```solidity
// place this internal for last 
71:    function _deployProxy(address proxyAdmin, address impl, bytes memory initData)
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol

```solidity
// place these internals after all the other functions since there are no private functions.
76:    function _addMember(IGnosisSafe securityCouncil, address _member, uint256 _threshold)
85:    function _removeMember(IGnosisSafe securityCouncil, address _member, uint256 _threshold)
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/ElectionGovernor.sol

```solidity
// place this internal for last
34:    function extractElectionIndex(bytes[] memory callDatas) internal pure returns (uint256) {
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol

```solidity
// place these external functions before all the others
90:    function separateSelector(bytes calldata calldataWithSelector)
203:    function relay(address target, uint256 value, bytes calldata data)

// place these internals after all the other functions since there are no private functions.
147:    function _voteSucceeded(uint256 proposalId)
163:    function _countVote(
182:    function _setVoteSuccessNumerator(uint256 _voteSuccessNumerator) internal {
223:    function _castVote(
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

```solidity
// place these internals after all the other functions since there are no private functions.
68:    function __SecurityCouncilMemberElectionGovernorCounting_init(uint256 initialFullWeightDuration)
95:    function _countVote(
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol

```solidity
// place these internals after all the other functions since there are no private functions.
115:    function _execute(
154:    function _isCompliantNominee(uint256 proposalId, address possibleNominee)
164:    function _compliantNominees(uint256 proposalId)
174:    function _targetMemberCount() internal view override returns (uint256) {
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol

```solidity
// place this internal after all the other functions since there are no private functions.
28:    function __SecurityCouncilNomineeElectionGovernorTiming_init(
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol

```solidity
// place these internals after all the other functions since there are no private functions.
62:    function _countVote(
121:    function _addNominee(uint256 proposalId, address account) internal {
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

```solidity
// palcea these internal functions for last since there are no private functions.
187:    function _requireLastMemberElectionHasExecuted() internal view {
324:    function _execute(
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol

```solidity
// place all internal functions for last since there are no private functions
143:    function _addMemberToCohortArray(address _newMember, Cohort _cohort) internal {
161:    function _removeMemberFromCohortArray(address _member) internal returns (Cohort) {
218:    function _swapMembers(address _addressToRemove, address _addressToAdd)
231:    function _addSecurityCouncil(SecurityCouncilData memory _securityCouncilData) internal {
315:    function _setUpgradeExecRouteBuilder(UpgradeExecRouteBuilder _router) internal {

// place this external on top with the others
370:    function generateSalt(address[] memory _members, uint256 nonce)
```

---

### natSpec missing [3]

Some functions are missing @params or @returns. Specification Format.” These are written with a triple slash (///) or a double asterisk block(/** ... */) directly above function declarations or statements to generate documentation in JSON format for developers and end-users. It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). These comments contain different types of tags:
- @title: A title that should describe the contract/interface @author: The name of the author (for contract, interface) 
- @notice: Explain to an end user what this does (for contract, interface, function, public state variable, event) 
- @dev: Explain to a developer any extra details (for contract, interface, function, state variable, event) 
- @param: Documents a parameter (just like in doxygen) and must be followed by parameter name (for function, event)
- @return: Documents the return variables of a contract’s function (function, public state variable)
- @inheritdoc: Copies all missing tags from the base function and must be followed by the contract name (for function, public state variable)
- @custom…: Custom tag, semantics is application-defined (for everywhere)

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/execution-record/ActionExecutionRecord.sol

```solidity
18:    constructor(KeyValueStore _store, string memory _uniqueActionName) {

// @param missing
25:    function _set(uint256 key, uint256 value) internal {

// @param and @return missing
31:    function _get(uint256 key) internal view returns (uint256) {
37:    function computeKey(uint256 key) public view returns (uint256) {
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/execution-record/KeyValueStore.sol

```solidity
// @param missing
11:    function set(uint256 key, uint256 value) external {

// @param and @return missing
16:    function get(uint256 key) external view returns (uint256) {
21:    function get(address owner, uint256 key) external view returns (uint256) {
26:    function computeKey(address owner, uint256 key) public pure returns (uint256) {

30:    function _get(address owner, uint256 key) internal view returns (uint256) {
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol

```solidity
7:  library SecurityCouncilMgmtUpgradeLib {
8:    function replaceEmergencySecurityCouncil(
29:    function requireSafesEquivalent(
52:    function areAddressArraysEqual(address[] memory array1, address[] memory array2)
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/NonGovernanceChainSCMgmtActivationAction.sol

```solidity
7:  contract NonGovernanceChainSCMgmtActivationAction {
13:    constructor(
25:    function perform() external {
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/L1SCMgmtActivationAction.sol

```solidity
9:  contract L1SCMgmtActivationAction {
16:    constructor(
30:    function perform() external {
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol

```solidity
12:  contract GovernanceChainSCMgmtActivationAction {
25:    constructor(
48:    function perform() external {
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol

```solidity
18:  struct DeployParams {
46:  struct ContractImplementations {
56:    event ContractsDeployed(DeployedContracts deployedContracts);
59:    struct DeployedContracts {
67:    error AddressNotInCouncil(address[] securityCouncil, address account);
71:    function _deployProxy(address proxyAdmin, address impl, bytes memory initData)

// @return missing
81:    function deploy(DeployParams memory dp, ContractImplementations memory impls)

// @params missing
192:    function _initRemovalGov(
212:    function _initElectionGovernors(
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/Common.sol

```solidity
13:  struct Date {
20:  error ZeroAddress();
21:  error NotAContract(address account);
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol

```solidity
4:  library SecurityCouncilMgmtUtils {
5:    function isInArray(address addr, address[] memory arr) internal pure returns (bool) {
15:    function filterAddressesWithExcludeList(
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol

```solidity
11:    error PreviousOwnerNotFound(address targetOwner, address securityCouncil);
12:    error ExecFromModuleError(bytes data, address securityCouncil);
14:    event UpdateNonceTooLow(
21:    constructor(KeyValueStore _store)
76:    function _addMember(IGnosisSafe securityCouncil, address _member, uint256 _threshold)
85:    function _removeMember(IGnosisSafe securityCouncil, address _member, uint256 _threshold)
97:    function getPrevOwner(IGnosisSafe securityCouncil, address _owner)
118:    function getUpdateNonce(address securityCouncil) public view returns (uint256) {
122:    function _setUpdateNonce(address securityCouncil, uint256 nonce) internal {
127:    function _execFromModule(IGnosisSafe securityCouncil, bytes memory data) internal {
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol

```solidity
9:  interface DefaultGovAction {
10:    function perform() external;
22:  struct ChainAndUpExecLocation {
40:    error UpgadeExecDoesntExist(uint256 chainId);
41:    error UpgradeExecAlreadyExists(uint256 chindId);
42:    error ParamLengthMismatch(uint256 len1, uint256 len2);
43:    error EmptyActionBytesData(bytes[]);

// @dev and @return missing
105:    function createActionRouteData(
186:    function createActionRouteDataWithDefaults(

```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/ElectionGovernor.sol

```solidity
// @return missing
34:    function extractElectionIndex(bytes[] memory callDatas) internal pure returns (uint256) {
40:    function electionIndexToDescription(uint256 electionIndex)

// @param and @return missing
50:    function electionIndexToCohort(uint256 electionIndex) public pure returns (Cohort) {
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/ArbitrumGovernorVotesQuorumFractionUpgradeable.sol

```solidity
24:    function __ArbitrumGovernorVotesQuorumFraction_init(uint256 quorumNumeratorValue)

// @param and @return missing
32:    function getPastCirculatingSupply(uint256 blockNumber) public view virtual returns (uint256) {
38:    function quorum(uint256 blockNumber) public view virtual override returns (uint256) {
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol

```solidity
32:    event VoteSuccessNumeratorSet(uint256 indexed voteSuccessNumerator);
33:    event MemberRemovalProposed(address memberToRemove, string description);
35:    error InvalidOperationsLength(uint256 len);
36:    error TargetNotManager(address target);
37:    error ValueNotZero(uint256 value);
38:    error UnexpectedCalldataLength(uint256 len);
39:    error CallNotRemoveMember(bytes4 selector, bytes4 expectedSelector);
40:    error MemberNotFound(address memberToRemove);
41:    error AbstainDisallowed();
42:    error InvalidVoteSuccessNumerator(uint256 voteSuccessNumerator);
44:    constructor() {
182:    function _setVoteSuccessNumerator(uint256 _voteSuccessNumerator) internal {
249:    function state(uint256 proposalId)

// @dev/notice missing
90:    function separateSelector(bytes calldata calldataWithSelector)

// @return missing
106:    function propose(
147:    function _voteSucceeded(uint256 proposalId)
203:    function relay(address target, uint256 value, bytes calldata data)

// @params missing
63:    function _countVote(
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

```solidity
55:    error FullWeightDurationGreaterThanVotingPeriod(
58:    error UnexpectedParamsLength(uint256 paramLength);
59:    error NotCompliantNominee(address nominee);
60:    error ZeroWeightVote(uint256 blockNumber, uint256 votes);
61:    error InsufficientVotes(uint256 prevVotesUsed, uint256 votes, uint256 availableVotes);
62:    error LengthsDontMatch(uint256 nomineesLength, uint256 weightsLength);
63:    error NotEnoughNominees(uint256 numNominees, uint256 k);
64:    error UintTooLarge(uint256 x);
65:    error InvalidSupport(uint8 support);

// @params missing
77:    function setFullWeightDuration(uint256 newFullWeightDuration) public onlyGovernance {

// @params and @return missing
148:    function votesUsed(uint256 proposalId, address account) public view returns (uint256) {
153:    function weightReceived(uint256 proposalId, address nominee) public view returns (uint256) {
158:    function hasVoted(uint256 proposalId, address account) public view override returns (bool) {
164:    function fullWeightVotingDeadline(uint256 proposalId) public view returns (uint256) {
225:    function votesToWeight(uint256 proposalId, uint256 blockNumber, uint256 votes)
259:    function _downCast(uint256 x) internal pure returns (uint240) {
267:    function _quorumReached(uint256) internal pure override returns (bool) {
273:    function _voteSucceeded(uint256) internal pure override returns (bool) {
278:    function _isCompliantNominee(uint256 proposalId, address possibleNominee)
285:    function _compliantNominees(uint256 proposalId)

// @return missing
177:    function topNominees(uint256 proposalId) public view returns (address[] memory) {
191:    function selectTopNominees(address[] memory nominees, uint240[] memory weights, uint256 k)
292:    function _targetMemberCount() internal view virtual returns (uint256);
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol

```solidity
38:    constructor() {
78:    modifier onlyNomineeElectionGovernor() {

// @params missing
103:    function relay(address target, uint256 value, bytes calldata data)
115:    function _execute(
138:    function proposalThreshold()

// @param and @return missing
148:    function quorum(uint256) public pure override returns (uint256) {
154:    function _isCompliantNominee(uint256 proposalId, address possibleNominee)
164:    function _compliantNominees(uint256 proposalId)
181:    function propose(address[] memory, uint256[] memory, bytes[] memory, string memory)
191:    function castVote(uint256, uint8) public virtual override returns (uint256) {
196:    function castVoteWithReason(uint256, uint8, string calldata)
206:    function castVoteBySig(uint256, uint8, uint8, bytes32, bytes32)
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol

```solidity
// @params missing
28:    function __SecurityCouncilNomineeElectionGovernorTiming_init(

// @return and @param missing
69:    function proposalVettingDeadline(uint256 proposalId) public view returns (uint256) {

// @return missing
75:    function electionToTimestamp(uint256 electionIndex) public view returns (uint256) {
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol

```solidity
44:    event NewNominee(uint256 indexed proposalId, address indexed nominee);
46:    error UnexpectedParamsLength(uint256 paramLength);
47:    error NotEligibleContender(address contender);
48:    error NomineeAlreadyAdded(address nominee);
49:    error InsufficientTokens(uint256 votes, uint256 prevVotesUsed, uint256 weight);
50:    error InvalidSupport(uint8 support);
52:    function __SecurityCouncilNomineeElectionGovernorCounting_init() internal onlyInitializing {}

// @params missing
121:    function _addNominee(uint256 proposalId, address account) internal {

// @params and @return missing
133:    function hasVoted(uint256 proposalId, address account) public view override returns (bool) {
138:    function isNominee(uint256 proposalId, address contender) public view returns (bool) {
143:    function nomineeCount(uint256 proposalId) public view returns (uint256) {
148:    function nominees(uint256 proposalId) public view returns (address[] memory) {
153:    function votesUsed(uint256 proposalId, address account) public view returns (uint256) {
158:    function votesReceived(uint256 proposalId, address contender) public view returns (uint256) {
163:    function isContender(uint256 proposalId, address possibleContender)
170:    function _quorumReached(uint256) internal pure override returns (bool) {
175:    function _voteSucceeded(uint256) internal pure override returns (bool) {
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

```solidity
76:    event NomineeVetterChanged(address indexed oldNomineeVetter, address indexed newNomineeVetter);
77:    event ContenderAdded(uint256 indexed proposalId, address indexed contender);
78:    event NomineeExcluded(uint256 indexed proposalId, address indexed nominee);
80:    error OnlyNomineeVetter();
81:    error CreateTooEarly(uint256 blockTimestamp, uint256 startTime);
82:    error AlreadyContender(address contender);
83:    error ProposalNotActive(ProposalState state);
84:    error AccountInOtherCohort(Cohort cohort, address account);
85:    error ProposalNotSucceededState(ProposalState state);
86:    error ProposalNotInVettingPeriod(uint256 blockNumber, uint256 vettingDeadline);
87:    error NomineeAlreadyExcluded(address nominee);
88:    error CompliantNomineeTargetHit(uint256 nomineeCount, uint256 expectedCount);
89:    error ProposalInVettingPeriod(uint256 blockNumber, uint256 vettingDeadline);
90:    error InsufficientCompliantNomineeCount(uint256 compliantNomineeCount, uint256 expectedCount);
91:    error ProposeDisabled();
92:    error NotNominee(address nominee);
93:    error ProposalIdMismatch(uint256 nomineeProposalId, uint256 memberProposalId);
94:    error QuorumNumeratorTooLow(uint256 quorumNumeratorValue);
95:    error CastVoteDisabled();
96:    error LastMemberElectionNotExecuted(uint256 prevProposalId);

98:    constructor() {

// @param missing
218:    function addContender(uint256 proposalId) external {
246:    function setNomineeVetter(address _nomineeVetter) external onlyGovernance {
254:    function relay(address target, uint256 value, bytes calldata data)
290:    function includeNominee(uint256 proposalId, address account) external onlyNomineeVetter {
324:    function _execute(

// @param and return missing
357:    function proposalThreshold()
383:    function compliantNomineeCount(uint256 proposalId) public view returns (uint256) {
400:    function isExcluded(uint256 proposalId, address possibleExcluded) public view returns (bool) {
405:    function excludedNomineeCount(uint256 proposalId) public view returns (uint256) {
423:    function propose(address[] memory, uint256[] memory, bytes[] memory, string memory)
433:    function castVote(uint256, uint8) public virtual override returns (uint256) {
438:    function castVoteWithReason(uint256, uint8, string calldata)
448:    function castVoteBySig(uint256, uint8, uint8, bytes32, bytes32)

// @return missing
388:    function currentCohort() public view returns (Cohort) {
394:    function otherCohort() public view returns (Cohort) {
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol

```solidity
33:    event CohortReplaced(address[] newCohort, Cohort indexed cohort);
34:    event MemberAdded(address indexed newMember, Cohort indexed cohort);
35:    event MemberRemoved(address indexed member, Cohort indexed cohort);
36:    event MemberReplaced(address indexed replacedMember, address indexed newMember, Cohort cohort);
37    event MemberRotated(address indexed replacedAddress, address indexed newAddress, Cohort cohort);
38:    event SecurityCouncilAdded(
41:    event SecurityCouncilRemoved(
44:    event UpgradeExecRouteBuilderSet(address UpgradeExecRouteBuilder);
85:    constructor() {
89:    function initialize(
143:    function _addMemberToCohortArray(address _newMember, Cohort _cohort) internal {
161:    function _removeMemberFromCohortArray(address _member) internal returns (Cohort) {
218:    function _swapMembers(address _addressToRemove, address _addressToAdd)
231:    function _addSecurityCouncil(SecurityCouncilData memory _securityCouncilData) internal {
315:    function _setUpgradeExecRouteBuilder(UpgradeExecRouteBuilder _router) internal {
327:    function getFirstCohort() external view returns (address[] memory) {
332:    function getSecondCohort() external view returns (address[] memory) {
337:    function getBothCohorts() public view returns (address[] memory) {
```

---

### State variable and function names [4]

- Variables should be named according to their specifications
- private and internal `variables` should preppend with `underline`
- private and internal `functions` should preppend with `underline`
- constant state variables should be UPPER_CASE

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol

```solidity
// private and internal `functions` should preppend with `underline`
8:    function replaceEmergencySecurityCouncil(
29:    function requireSafesEquivalent(
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol

```solidity
// private and internal `functions` should preppend with `underline`
15:    function filterAddressesWithExcludeList(
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/ElectionGovernor.sol

```solidity
// private and internal `functions` should preppend with `underline`
34:    function extractElectionIndex(bytes[] memory callDatas) internal pure returns (uint256) {
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol

```solidity
// private and internal `variables` should preppend with `underline`
51:    address[] internal firstCohort;
52:    address[] internal secondCohort;
```