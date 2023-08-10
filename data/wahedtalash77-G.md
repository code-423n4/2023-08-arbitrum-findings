|     |
| --- |
| \[G-01\] Make for loop unchecked |
| \[G‑02\] Using calldata instead of memory for read-only arguments in external functions saves gas |
| \[G-03\] If statements that use && can be refactored into nested if statements |
| \[G-04\] `variable == false` instead of `!variable`. |
| \[G-05\] x += y/x -= y costs more gas than x = x + y/x = x - y for state variables |
| \[G-06\] The result of a function call should be cached rather than re-calling the function |
| \[G-07\] Avoiding Initialization of the variables to it's default value. |
| \[G-08\] Make 3 event parameters indexed when possible |
| \[G-09\] Use solidity version 0.8.19 to gain some gas boost |
| \[G-10\] For loops in public or external functions should be avoided due to high gas costs and possible DOS |
| \[G‑11\] Using private rather than public for constants, saves gas |
| \[G‑12\] Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, |
| \[G-13\] Replace string errors with custom errors |
| \[G‑14\] Avoid contract existence checks by using low level calls |
| \[G-15\] Use immutable for constant variables. |
| \[G-16\] Avoid unnecessary read of array length in for loops can save gas |
| \[G-17\] Events are not indexed |


*** 
## \[G-01\] Make for loop unchecked

The risk of for loops getting overflowed is extremely low. Because it always increments by 1 and is limited to the arrays length. Even if the arrays are extremely long, it will take a massive amount of time and gas to let the for loop overflow.

```
for (uint256 i = 0; i < 2; i++) {
            address[] storage cohort = i == 0 ? firstCohort : secondCohort;
            for (uint256 j = 0; j < cohort.length; j++) {
                if (_member == cohort[j]) {
                    cohort[j] = cohort[cohort.length - 1];
                    cohort.pop();
                    return i == 0 ? Cohort.FIRST : Cohort.SECOND;
                }
            }
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L162C9-L170C14

```
for (uint256 i = 0; i < _roles.memberRemovers.length; i++) {
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L106

```
 for (uint256 i = 0; i < _securityCouncils.length; i++) {
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L118

```
for (uint256 i = 0; i < _newCohort.length; i++) {
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L135

```
 for (uint256 i = 0; i < securityCouncils.length; i++) {
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L251

```
function getBothCohorts() public view returns (address[] memory) {
        address[] memory members = new address[](firstCohort.length + secondCohort.length);
        for (uint256 i = 0; i < firstCohort.length; i++) {
            members[i] = firstCohort[i];
        }
        for (uint256 i = 0; i < secondCohort.length; i++) {
            members[firstCohort.length + i] = secondCohort[i];
        }
        return members;
    }
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L337C5-L346C6

```
for (uint256 i = 0; i < nominees.length; i++) {
            weights[i] = election.weightReceived[nominees[i]];
        }
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L181C9-L183C10

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L205

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L215

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L52C1-L81C14

## \[G‑02\] Using calldata instead of memory for read-only arguments in external functions saves gas

When a function with a `memory` array is called externally, the `abi.decode()` step has to use a for-loop to copy each index of the `calldata` to the `memory` index. Each iteration of this for-loop costs at least 60 gas (i.e. `60 * <mem_array>.length`). Using `calldata` directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having `memory` arguments, it’s still valid for implementation contracs to use `calldata` arguments instead.

If the array is passed to an `internal` function which passes the array to another internal function where the array is modified and therefore `memory` is used in the `external` call, it’s still more gass-efficient to use `calldata` when the `external` function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

```
  function initialize(
        address[] memory _firstCohort,
        address[] memory _secondCohort,
        SecurityCouncilData[] memory _securityCouncils,
        SecurityCouncilManagerRoles memory _roles,
        address payable _l2CoreGovTimelock,
        UpgradeExecRouteBuilder _router
    ) external initializer {
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L89C3-L96C29

```
    function addSecurityCouncil(SecurityCouncilData memory _securityCouncilData)
```

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L271

```
 function getFirstCohort() external view returns (address[] memory) {
        return firstCohort;
    }

    /// @inheritdoc ISecurityCouncilManager
    function getSecondCohort() external view returns (address[] memory) {
        return secondCohort;
    }
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L327C4-L334C6

```
function createActionRouteData(
        uint256[] memory chainIds,
        address[] memory actionAddresses,
        uint256[] memory actionValues,
        bytes[] memory actionDatas,
        bytes32 predecessor,
        bytes32 timelockSalt
    ) public view returns (address, bytes memory) {
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L105C5-L112C52

## \[G-03\] If statements that use && can be refactored into nested if statements

```
 if (
                existantSecurityCouncil.chainId == _securityCouncilData.chainId
                    && existantSecurityCouncil.securityCouncil == _securityCouncilData.securityCouncil
            )
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L254C12-L257C14

```
if (
                securityCouncilData.securityCouncil == _securityCouncilData.securityCouncil
                    && securityCouncilData.chainId == _securityCouncilData.chainId
                    && securityCouncilData.updateAction == _securityCouncilData.updateAction
            ) {
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L286C13-L290C16

```
 if (
            !securityCouncilManager.firstCohortIncludes(memberToRemove)
                && !securityCouncilManager.secondCohortIncludes(memberToRemove)
        ) {
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L134C8-L137C12

```
 if (!(0 < _voteSuccessNumerator && _voteSuccessNumerator <= voteSuccessDenominator)) {
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L183

## \[G-04\] `variable == false` instead of `!variable`.

a bit cheapier when you replace:

```
 if (!isSupportedDateTime) {
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol#L41

```
 if (!found) {
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L82

```
  if (!Address.isContract(routerAddress)) {
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L318

```
   if (!isContender(proposalId, contender)) {
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L80

```
 if (!(0 < _voteSuccessNumerator && _voteSuccessNumerator <= voteSuccessDenominator)) {
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L183

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol#L86C8-L100C10

## \[G-05\] x += y/x -= y costs more gas than x = x + y/x = x - y for state variables

```
 month += 6 * electionIndex;
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol#L79

```
 month += 1;
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol#L84

## \[G-06\] The result of a function call should be cached rather than re-calling the function

```
return keccak256(abi.encodePacked(_members, nonce));
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L375

```
function isCompliantNominee(uint256 proposalId, address account) public view returns (bool) {
        return isNominee(proposalId, account) && !_elections[proposalId].isExcluded[account];
    }
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L368C5-L370C6

```
function currentCohort() public view returns (Cohort) {
        // current cohort is at electionCount - 1
        return electionCount == 0 ? Cohort.FIRST : electionIndexToCohort(electionCount - 1);
    }
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L388C5-L391C6

```
  function otherCohort() public view returns (Cohort) {
        // previous cohort is at electionCount - 2
        return (electionCount < 2) ? Cohort.SECOND : electionIndexToCohort(electionCount - 2);
    }
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L394C3-L397C6

```
 function compliantNominees(uint256 proposalId) public view returns (address[] memory) {
        ElectionInfo storage election = _elections[proposalId];
        address[] memory maybeCompliantNominees =
            SecurityCouncilNomineeElectionGovernorCountingUpgradeable.nominees(proposalId);
        return SecurityCouncilMgmtUtils.filterAddressesWithExcludeList(
            maybeCompliantNominees, election.isExcluded
        );
    }
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L373C4-L380C6

```
return DateTimeLib.dateTimeToTimestamp({
            year: year,
            month: month,
            day: firstNominationStartDate.day,
            hour: firstNominationStartDate.hour,
            minute: 0,
            second: 0
        });
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol#L86C9-L93C12

```
    function topNominees(uint256 proposalId) public view returns (address[] memory) {
        address[] memory nominees = _compliantNominees(proposalId);
        uint240[] memory weights = new uint240[](nominees.length);
        ElectionInfo storage election = _elections[proposalId];
        for (uint256 i = 0; i < nominees.length; i++) {
            weights[i] = election.weightReceived[nominees[i]];
        }
        return selectTopNominees(nominees, weights, _targetMemberCount());
    }
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L177C5-L185C6

```
   function _castVote(
        uint256 proposalId,
        address account,
        uint8 support,
        string memory reason,
        bytes memory params
    )
        internal
        override(GovernorUpgradeable, GovernorPreventLateQuorumUpgradeable)
        returns (uint256)
    {
        return GovernorPreventLateQuorumUpgradeable._castVote(
            proposalId, account, support, reason, params
        );
    }
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L223C2-L237C6

```
  function computeKey(uint256 key) public view returns (uint256) {
        return uint256(keccak256(abi.encode(actionContractId, key)));
    }
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/execution-record/ActionExecutionRecord.sol#L37C3-L39C6

## \[G-07\] Avoiding Initialization of the variables to it's default value.

```
uint256 public constant DEFAULT_VALUE = 0;
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L52

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol

> all loops have a default value

## \[G-08\] Make 3 event parameters indexed when possible

It’s the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

[Reference](https://code4rena.com/reports/2023-01-drips#g-05-make-3-event-parameters-indexed-when-possible)

```
    event CohortReplaced(address[] newCohort, Cohort indexed cohort);
    event MemberAdded(address indexed newMember, Cohort indexed cohort);
    event MemberRemoved(address indexed member, Cohort indexed cohort);
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L33C5-L35C72

```
event NomineeVetterChanged(address indexed oldNomineeVetter, address indexed newNomineeVetter);
    event ContenderAdded(uint256 indexed proposalId, address indexed contender);
    event NomineeExcluded(uint256 indexed proposalId, address indexed nominee);
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L76C4-L78C80

```
 event VoteSuccessNumeratorSet(uint256 indexed voteSuccessNumerator);
    event MemberRemovalProposed(address memberToRemove, string description);
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L32C4-L33C77

```
 event NewNominee(uint256 indexed proposalId, address indexed nominee);
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L44




## \[G-10\] For loops in public or external functions should be avoided due to high gas costs and possible DOS

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L89C4-L121C6

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L124C1-L125C17

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L279

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L337

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L379

## \[G‑11\] Using private rather than public for constants, saves gas

```
bytes32 public constant COHORT_REPLACER_ROLE = keccak256("COHORT_REPLACER");
    bytes32 public constant MEMBER_ADDER_ROLE = keccak256("MEMBER_ADDER");
    bytes32 public constant MEMBER_REPLACER_ROLE = keccak256("MEMBER_REPLACER");
    bytes32 public constant MEMBER_ROTATOR_ROLE = keccak256("MEMBER_ROTATOR");
    bytes32 public constant MEMBER_REMOVER_ROLE = keccak256("MEMBER_REMOVER");
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L79C5-L83C79

```
    uint256 public constant voteSuccessDenominator = 10_000;
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L28

```
address public constant EXCLUDE_ADDRESS = address(0xA4b86);
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/ArbitrumGovernorVotesQuorumFractionUpgradeable.sol#L22C5-L22C64

```
address public constant RETRYABLE_TICKET_MAGIC = 0xa723C008e76E379c55599D2E4d93879BeaFDa79C;
    /// @notice Default args for creating a proposal, used by createProposalWithDefaulArgs and createProposalBatchWithDefaultArgs
    ///         Default is function selector for a perform function with no args: 'function perform() external'
    bytes public constant DEFAULT_GOV_ACTION_CALLDATA =
        abi.encodeWithSelector(DefaultGovAction.perform.selector);
    uint256 public constant DEFAULT_VALUE = 0;
    /// @notice Default predecessor used when calling the L1 timelock
    bytes32 public constant DEFAULT_PREDECESSOR = bytes32(0);

```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L47C5-L54C62

```
 address public constant SENTINEL_OWNERS = address(0x1);
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L19C60-L19C60

## \[G‑12\] Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct,

where appropriate Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.

```
  mapping(address => uint256) votesUsed;
        /// @dev The weight of votes received by a nominee. At the start of the election
        ///      each vote has weight 1, however after a cutoff point the weight of each
        ///      vote decreases linearly until it is 0 by the end of the election.
        ///      Using uint240 because of the sorting implementation, see `selectTopNominees`
        mapping(address => uint240) weightReceived;

```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L19C7-L24C52

## \[G-13\] result of the calculation should be cached

```
return voteSuccessDenominator * forVotes > (forVotes + againstVotes) * voteSuccessNumerator;
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L157

```
function quorum(uint256 blockNumber) public view virtual override returns (uint256) {
        return (getPastCirculatingSupply(blockNumber) * quorumNumerator(blockNumber))
            / quorumDenominator();
    }
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/ArbitrumGovernorVotesQuorumFractionUpgradeable.sol#L38C5-L41C6


## \[G-09\] Use solidity version 0.8.19 to gain some gas boost

> Upgrade to the latest solidity version 0.8.19 to get additional gas savings.
> 
> all contest
> See latest release for reference: https://blog.soliditylang.org/2023/02/22/solidity-0.8.19-release-announcement/
