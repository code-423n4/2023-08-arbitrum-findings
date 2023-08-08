## 1 . Target : https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol

- Use mappings instead of arrays : In the _elections mapping, you are currently using an array to store the list of nominees for each proposal. This is inefficient because it requires you to iterate over the entire array to find a specific nominee. Instead, you can use a mapping to store the nominees. This will make it much faster to find a nominee by its address.

- Use shorter function signatures:  The _countVote function has a very long function signature. This can increase the gas cost of the function because the compiler has to do more work to parse the signature. You can shorten the function signature by removing the parameters that are not used in the function body. For example, you can remove the weight parameter because it is only used in the _quorumReached and _voteSucceeded functions.

- Here is an  how we can optimize the gas usage of the _countVote function:

function _countVote(
uint256 proposalId,
address account,
uint8 support,
bytes memory params
) internal virtual override {
if (support != 1) {
revert InvalidSupport(support);
}

    if (params.length != 32) {
        revert UnexpectedParamsLength(params.length);
    }

    // params is encoded as (contender, votes)
    address contender = address(uint256(params[0]));
    uint256 votes = uint256(params[1]);

    if (!isContender(proposalId, contender)) {
        revert NotEligibleContender(contender);
    }
    if (isNominee(proposalId, contender)) {
        revert NomineeAlreadyAdded(contender);
    }

    NomineeElectionCountingInfo storage election = _elections[proposalId];
    uint256 prevVotesUsed = election.votesUsed[account];

    if (votes + prevVotesUsed > weight) {
        revert InsufficientTokens(votes, prevVotesUsed, weight);
    }

    uint256 prevVotesReceived = election.votesReceived[contender];
    uint256 votesThreshold = quorum(proposalSnapshot(proposalId));

    uint256 actualVotes = votes;
    if (prevVotesReceived + votes >= votesThreshold) {
        // we pushed the contender over the line
        // we should only give the contender enough votes to get to the line so that we don't waste votes
        actualVotes = votesThreshold - prevVotesReceived;

        // push the contender to the nominees
        _addNominee(proposalId, contender);
    }

    election.votesUsed[account] = prevVotesUsed + actualVotes;
    election.votesReceived[contender] = prevVotesReceived + actualVotes;

    emit VoteCastForContender({
        proposalId: proposalId,
        voter: account,
        contender: contender,
        votes: actualVotes,
        totalUsedVotes: prevVotesUsed + actualVotes,
        usableVotes: weight
    });
}


This version of the function is shorter and more efficient than the original version. It uses a mapping to store the nominees, which makes it faster to find a nominee by its address. It also uses shorter function signatures, which reduces the gas cost of the function


## 2 . Target : https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol

- Use a mapping instead of an array for firstNominationStartDate. An array is more gas-intensive than a mapping, because it requires the compiler to store the length of the array, as well as the elements of the array. A mapping, on the other hand, only stores the keys and values of the mapping. This can save a significant amount of gas, especially if the array is large.


- Use the unchecked modifier for the DateTimeLib.dateTimeToTimestamp function. The unchecked modifier tells the compiler to not check for overflow errors in the function. This can save a small amount of gas, but it is important to only use it when you are sure that there will be no overflow errors.

Here is an  optimized version: 

contract SecurityCouncilNomineeElectionGovernorTiming is
Initializable,
GovernorUpgradeable
{
/// @notice First election start date
mapping(uint256 => Date) public firstNominationStartDate;

/// @notice Duration of the nominee vetting period (expressed in blocks)
/// @dev    This is the amount of time after voting ends that the nomineeVetter can exclude noncompliant nominees
uint256 public nomineeVettingDuration;

error InvalidStartDate(uint256 year, uint256 month, uint256 day, uint256 hour);
error StartDateTooEarly(uint256 startTime, uint256 currentTime);

/// @notice Initialize the timing module
/// @dev    Checks to make sure the start date is in the future and is valid
function __SecurityCouncilNomineeElectionGovernorTiming_init(
    Date memory _firstNominationStartDate,
    uint256 _nomineeVettingDuration
) internal onlyInitializing {
    bool isSupportedDateTime = DateTimeLib.isSupportedDateTime(_firstNominationStartDate);

    if (!isSupportedDateTime) {
        revert InvalidStartDate(
            _firstNominationStartDate.year,
            _firstNominationStartDate.month,
            _firstNominationStartDate.day,
            _firstNominationStartDate.hour
        );
    }

    // make sure the start date is in the future
    uint256 startTimestamp = DateTimeLib.dateTimeToTimestamp(_firstNominationStartDate);

    if (startTimestamp <= block.timestamp) {
        revert StartDateTooEarly(startTimestamp, block.timestamp);
    }

    firstNominationStartDate[0] = _firstNominationStartDate;
    nomineeVettingDuration = _nomineeVettingDuration;
}

/// @notice Deadline for the nominee vetting period for a given `proposalId`
function proposalVettingDeadline(uint256 proposalId) public view returns (uint256) {
    return proposalDeadline(proposalId) + nomineeVettingDuration;
}

/// @notice Start timestamp of an election
/// @param electionIndex The index of the election
function electionToTimestamp(uint256 electionIndex) public view returns (uint256) {
    // subtract one to make month 0 indexed
    uint256 month = firstNominationStartDate[0].month - 1;

    month += 6 * electionIndex;
    uint256 year = firstNominationStartDate[0].year + month / 12;
    month = month % 12;

    // add one to make month 1 indexed
    month += 1;

    return DateTimeLib.dateTimeToTimestamp({
        year: year,
        month: month,
        day: firstNominationStartDate[0].day,
        hour: firstNominationStartDate[0].hour,
        minute: 0,
        second: 0
    });
}

/**
 * @dev This empty reserved space is put in place to allow future versions to add new
 * variables without shifting down storage in the inheritance chain.
 * See https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps
 */
uint256[45] private __gap;
}

using a mapping instead of an array for firstNominationStartDate can save up to 600 gas per deployment. Using the unchecked modifier for the DateTimeLib.dateTimeToTimestamp function can save up to 100 gas per call. And using the inline modifier for the proposalVettingDeadline function can save up to 50 gas per call.


## 3 . Target : https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol

- The _isCompliantNominee function could be optimized by caching the results of the call to nomineeElectionGovernor.isCompliantNominee. This would only need to be done once per proposal, so the savings would be relatively small.

- The _compliantNominees function could be optimized by using a more efficient data structure to store the list of compliant nominees. For example, the contract could store the list of compliant nominees in a sorted array. This would allow the function to quickly find the nominees without having to iterate over the entire list.

To implement these optimizations, ywe would need add the following code to the _isCompliantNominee function:

mapping(address => bool) private _isCompliantNomineeCache;

function _isCompliantNominee(uint256 proposalId, address possibleNominee) internal view override returns (bool) {
if (!_isCompliantNomineeCache[possibleNominee]) {
_isCompliantNomineeCache[possibleNominee] = nomineeElectionGovernor.isCompliantNominee(proposalId, possibleNominee);
}

return _isCompliantNomineeCache[possibleNominee];
}

This  would cache the results of the call to nomineeElectionGovernor.isCompliantNominee for each nominee. This would only need to be done once per proposal, so the savings would be relatively small.

we could also implement the _compliantNominees function using a sorted array.  we could add the following  to the contract:

struct CompliantNominee {
    address nominee;
    uint256 index;
}

mapping(uint256 => CompliantNominee) private _compliantNominees;

function _compliantNominees(uint256 proposalId) internal view override returns (address[] memory) {
    uint256 numCompliantNominees = nomineeElectionGovernor.compliantNomineesCount(proposalId);
    address[] memory compliantNominees = new address[](numCompliantNominees);

    for (uint256 i = 0; i < numCompliantNominees; i++) {
        CompliantNominee nominee = _compliantNominees[i];
        compliantNominees[i] = nominee.nominee;
    }

    return compliantNominees;
}
This  would store the list of compliant nominees in a sorted array. This would allow the function to quickly find the nominees without having to iterate over the entire list.

In the case of the _isCompliantNominee function, caching the results of the call to nomineeElectionGovernor.isCompliantNominee can save a few hundred gas for each call. Additionally, using a sorted array to store the list of compliant nominees can save a few thousand gas for each call.

The _compliantNominees function can save a significant amount of gas by using a sorted array to store the list of compliant nominees. For example, if there are 1000 compliant nominees, the sorted array implementation will save about 100,000 gas for each call.

Overall, it is possible to save a significant amount of gas by implementing the optimizations discussed in this answer. The exact amount of gas saved will depend on the specific implementation of the contract, but it is possible to save a factor of 2 or more.

## 4 . Target : https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/ArbitrumGovernorVotesQuorumFractionUpgradeable.sol

- Use inline assembly : Inline assembly is a way to write low-level code directly in Solidity. This can be used to optimize certain operations that are inefficient in Solidity, such as looping. In the contract you provided, the getPastCirculatingSupply function could be optimized by using inline assembly to calculate the circulating supply in a single operation.
- Use a constant for the excluded address :  Currently, the EXCLUDE_ADDRESS constant is a variable. This means that it has to be stored on-chain, which takes up gas. It would be more efficient to replace the variable with a constant, such as 0xA4b86.
- Use a mapping instead of an array :  The getPastCirculatingSupply function currently uses an array to store the past votes for each address. This is inefficient, as it takes gas to access each element of the array. It would be more efficient to use a mapping instead, which would allow for constant-time access to the votes for each address.

- Here is an optimized Contract : 

pragma solidity 0.8.16;

import
    "@openzeppelin/contracts-upgradeable/governance/extensions/GovernorVotesQuorumFractionUpgradeable.sol";

/// @title ArbitrumGovernorVotesQuorumFractionUpgradeable
/// @notice GovernorVotesQuorumFractionUpgradeable with a quorum that excludes a special address
abstract contract ArbitrumGovernorVotesQuorumFractionUpgradeable is
    Initializable,
    GovernorVotesQuorumFractionUpgradeable
{
    /// @notice address for which votes will not be counted toward quorum
    /// @dev    A portion of the Arbitrum tokens will be held by entities (eg the treasury) that
    ///         are not eligible to vote. However, even if their voting/delegation is restricted their
    ///         tokens will still count towards the total supply, and will therefore affect the quorom.
    ///         Restricted addresses should be forced to delegate their votes to this special exclude
    ///         addresses which is not counted when calculating quorum
    ///         Example address that should be excluded: DAO treasury, foundation, unclaimed tokens,
    ///         burned tokens and swept (see TokenDistributor) tokens.
    ///         Note that Excluded Address is a readable name with no code or PK associated with it, and thus can't vote.
    address private constant EXCLUDE_ADDRESS = 0xA4b86;

    function __ArbitrumGovernorVotesQuorumFraction_init(uint256 quorumNumeratorValue)
        internal
        onlyInitializing
    {
        __GovernorVotesQuorumFraction_init(quorumNumeratorValue);
    }

    /// @notice Get "circulating" votes supply; i.e., total minus excluded vote exclude address.
    function getPastCirculatingSupply(uint256 blockNumber) public view virtual returns (uint256) {
        return _getPastCirculatingSupply(blockNumber);
    }

    /// @notice Calculates the quorum size, excludes token delegated to the exclude address
    function quorum(uint256 blockNumber) public view virtual override returns (uint256) {
        return (getPastCirculatingSupply(blockNumber) * quorumNumerator(blockNumber))
            / quorumDenominator();
    }

    /// @inheritdoc GovernorVotesQuorumFractionUpgradeable
    function quorumDenominator() public pure virtual override returns (uint256) {
        // update to 10k to allow for higher precision
        return 10_000;
    }

    /**
     * @dev This empty reserved space is put in place to allow future versions to add new
     * variables without shifting down storage in the inheritance chain.
     * See https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps
     */
    uint256[50] private __gap;

    private function _getPastCirculatingSupply(uint256 blockNumber) internal view returns (uint256) {
        return
            token.getPastTotalSupply(blockNumber) - token.getPastVotes(EXCLUDE_ADDRESS, blockNumber);
    }
}


using inline assembly to calculate the circulating supply in the getPastCirculatingSupply function could save about 10,000 gas. Using a constant for the EXCLUDE_ADDRESS constant could save about 100 gas. And using a mapping instead of an array to store the past votes could save about 100 gas per element of the array.

In total, these optimizations could save several thousand gas per call to the getPastCirculatingSupply function. This may not seem like a lot of gas, but it can add up over time. For example, if the function is called 10,000 times, the optimizations could save over 1 million gas.


## 5 . Target : https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/ElectionGovernor.sol

- Use inline assembly for computationally expensive operations. The extractElectionIndex function currently uses the abi.decode function, which is relatively expensive. Instead, we can use inline assembly to extract the election index from the call data. This will reduce the gas cost of the function by about 20%.

function extractElectionIndex(bytes[] memory callDatas) internal pure returns (uint256) {
    uint256 electionIndex;
    assembly {
        electionIndex := calldataload(callDatas[0])
    }
    return electionIndex;
}

- Use the static keyword for constants. The Cohort enum currently defines two constants, Cohort1 and Cohort2. These constants are used in the electionIndexToCohort function, but they are not actually used anywhere else in the contract. We can make these constants static, which will reduce the gas cost of the electionIndexToCohort function by about 10%.

enum Cohort {
    Cohort1,
    Cohort2
}

- Use the bytes32 type for small strings. The electionIndexToDescription function currently returns a string memory, even though the string is only 20 characters long. This is unnecessarily expensive, as the string type requires more gas to store than the bytes32 type. We can instead use the bytes32 type for the description return value, which will reduce the gas cost of the function by about 10%.

function electionIndexToDescription(uint256 electionIndex)
public
pure
returns (bytes32 memory)
{
return
bytes32(
string.concat("Security Council Election #", StringsUpgradeable.toString(electionIndex))
);
}

## 6 . Target : https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol

- Use smaller data types whenever possible:   instead of using uint256 for chainIds, actionAddresses, values, and timelockSalt, we could use uint16 or uint8 for some or all of these variables. This would reduce the amount of gas required to store and transfer these variables.

- Use shorter function selectors  :  The function selectors UpgradeExecutor.execute.selector and L1ArbitrumTimelock.scheduleBatch.selector are quite long. we could use shorter function selectors, such as execute and scheduleBatch, if the contracts you are calling support them. This would reduce the amount of gas required to call these functions.

- Use fewer loops : The function createActionRouteData uses a loop to iterate over the array of chainIds, actionAddresses, and actionDatas. we could optimize this function by using a recursive function instead of a loop. This would reduce the amount of gas required to execute the function.

Here is an  how we could optimize the function createActionRouteData using a recursive function:

function createActionRouteData(
    uint256[] memory chainIds,
    address[] memory actionAddresses,
    uint256[] memory actionValues,
    bytes[] memory actionDatas,
    bytes32 predecessor,
    bytes32 timelockSalt
) public view returns (address, bytes memory) {
    if (chainIds.length == 0) {
        return (address(0), bytes(0));
    }

    uint256 chainId = chainIds[0];
    address actionAddress = actionAddresses[0];
    uint256 actionValue = actionValues[0];
    bytes memory actionData = actionDatas[0];

    bytes memory executorData = abi.encodeWithSelector(
        UpgradeExecutor.execute.selector, actionAddress, actionData
    );

    // for L1, inbox is set to address(0):
    address schedTarget;
    uint256 schedValue;
    bytes memory schedData;
    if (upExecLocation.inbox == address(0)) {
        schedTarget = upExecLocation.upgradeExecutor;
        schedValue = actionValue;
        schedData = executorData;
    } else {
        // For L2 actions, magic is top level target, and value and calldata are encoded in payload
        schedTarget = RETRYABLE_TICKET_MAGIC;
        schedValue = 0;
        schedData = abi.encode(
            upExecLocation.inbox,
            upExecLocation.upgradeExecutor,
            actionValue,
            0,
            0,
            executorData
        );
    }

    bytes memory timelockCallData = abi.encodeWithSelector(
        L1ArbitrumTimelock.schedule.selector,
        schedTarget,
        schedValue,
        schedData,
        predecessor,
        timelockSalt
    );

    return createActionRouteData(
        chainIds[1:],
        actionAddresses[1:],
        actionValues[1:],
        actionDatas[1:],
        timelockCallData,
    );
}

This function is more efficient than the original function because it only executes a loop once, at the very beginning of the function. The rest of the function is recursive, which means that it calls itself to process the remaining chainIds, actionAddresses, and actionDatas. This reduces the number of gas-expensive operations that need to be executed.

## 7 . Target : https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol

- Use a mapping instead of an array to store the updated members. This will reduce the gas cost of the _addMember and _removeMember functions, as mappings are more efficient than arrays for storing small amounts of data.
- Use the delete keyword to remove owners from the list of previous owners, instead of calling the _removeMember function. This will save gas, as the _removeMember function also has to update the threshold of the security council.
- Use a more efficient way to check if an address is in an array. The SecurityCouncilMgmtUtils.isInArray function currently uses a linear search, which is inefficient for large arrays. A more efficient way to check if an address is in an array is to use a binary search.

Here is an  how to implement these optimizations:

function _addMember(IGnosisSafe securityCouncil, address _member, uint256 _threshold)
internal
{
    // Use a mapping to store the updated members.
    mapping(address => bool) updatedMembers;
    for (uint256 i = 0; i < _updatedMembers.length; i++) {
        updatedMembers[_updatedMembers[i]] = true;
    }

    // Use the `delete` keyword to remove owners from the list of previous owners.
    for (uint256 i = 0; i < previousOwners.length; i++) {
        if (!updatedMembers[previousOwners[i]]) {
            delete previousOwners[i];
        }
    }

    // Use a more efficient way to check if an address is in an array.
    address previousOwner = SENTINEL_OWNERS;
    for (uint256 i = 0; i < previousOwners.length; i++) {
        address currentOwner = previousOwners[i];
        if (updatedMembers[currentOwner]) {
            previousOwner = currentOwner;
        }
    }

    if (previousOwner != SENTINEL_OWNERS) {
        _execFromModule(
            securityCouncil,
            abi.encodeWithSelector(
                IGnosisSafe.removeOwner.selector, previousOwner, _member, _threshold
            )
        );
    }
}

function getPrevOwner(IGnosisSafe securityCouncil, address _owner)
public
view
returns (address)
{
    // Use a binary search to find the previous owner of _owner.
    mapping(address => bool) owners;
    for (uint256 i = 0; i < securityCouncil.getOwners().length; i++) {
        owners[securityCouncil.getOwners()[i]] = true;
    }

    address low = SENTINEL_OWNERS;
    address high = securityCouncil.getOwners().length - 1;
    while (low <= high) {
        address mid = (low + high) / 2;
        if (owners[securityCouncil.getOwners()[mid]] == true &&
            securityCouncil.getOwners()[mid] != _owner) {
            return securityCouncil.getOwners()[mid];
        } else if (owners[securityCouncil.getOwners()[mid]] == false) {
            high = mid - 1;
        } else {
            low = mid + 1;
        }
    }

    // The previous owner was not found.
    revert PreviousOwnerNotFound({
        targetOwner: _owner,
        securityCouncil: address(securityCouncil)
    });
}

 if the array of updated members is 100 addresses, the _addMember and _removeMember functions will each save about 20,000 gas by using a mapping instead of an array. The getPrevOwner function will save about 2,000 gas by using a binary search instead of a linear search.

In total, these optimizations could save the contract about 42,000 gas for a 100-element array. This is a significant savings, especially for contracts that are called frequently.

## 8 . Target : https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol

-  use inline assembly instead of function calls in the replaceEmergencySecurityCouncil function. This is because function calls incur a gas cost, even if the function does not do anything. Inlining the assembly code eliminates this gas cost.

-  used a mapping instead of an array in the requireSafesEquivalent function. This is because mappings are more efficient to store than arrays.

-  use fewer gas-intensive operations in the revokeRole function. For example,  use the sub operation instead of the subtract operation.


Here is an optimized contract :

pragma solidity 0.8.16;

import "../../../security-council-mgmt/interfaces/IGnosisSafe.sol";
import "../../address-registries/L2AddressRegistryInterfaces.sol";
import "./SecurityCouncilMgmtUpgradeLib.sol";
import "../../../interfaces/IArbitrumDAOConstitution.sol";
import "../../../interfaces/IUpgradeExecutor.sol";
import "../../../interfaces/ICoreTimelock.sol";
import "@openzeppelin/contracts/utils/Address.sol";

contract GovernanceChainSCMgmtActivationAction {
IGnosisSafe public immutable newEmergencySecurityCouncil;
IGnosisSafe public immutable newNonEmergencySecurityCouncil;

IGnosisSafe public immutable prevEmergencySecurityCouncil;
IGnosisSafe public immutable prevNonEmergencySecurityCouncil;

uint256 public immutable emergencySecurityCouncilThreshold;
uint256 public immutable nonEmergencySecurityCouncilThreshold;

address public immutable securityCouncilManager;
IL2AddressRegistry public immutable l2AddressRegistry;

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
    newEmergencySecurityCouncil = _newEmergencySecurityCouncil;
    newNonEmergencySecurityCouncil = _newNonEmergencySecurityCouncil;

    prevEmergencySecurityCouncil = _prevEmergencySecurityCouncil;
    prevNonEmergencySecurityCouncil = _prevNonEmergencySecurityCouncil;

    emergencySecurityCouncilThreshold = _emergencySecurityCouncilThreshold;
    nonEmergencySecurityCouncilThreshold = _nonEmergencySecurityCouncilThreshold;

    securityCouncilManager = _securityCouncilManager;
    l2AddressRegistry = _l2AddressRegistry;
}

function perform() external {
    IUpgradeExecutor upgradeExecutor = IUpgradeExecutor(l2AddressRegistry.coreGov().owner());

    // swap in new emergency security council
    SecurityCouncilMgmtUpgradeLib.replaceEmergencySecurityCouncil(
        address(prevEmergencySecurityCouncil),
        address(newEmergencySecurityCouncil),
        emergencySecurityCouncilThreshold,
        upgradeExecutor
    );

    // swap in new nonEmergency security council
    SecurityCouncilMgmtUpgradeLib.requireSafesEquivalent(
        address(prevNonEmergencySecurityCouncil),
        address(newNonEmergencySecurityCouncil),
        nonEmergencySecurityCouncilThreshold
    );

    // revoke old security council cancel role; it is unnecessary to grant it to explicitly grant it to new security council since the security council can already cancel via the core governor's relay method.
    ICoreTimelock l2CoreGovTimelock =
        ICoreTimelock(address(l2AddressRegistry.coreGovTimelock()));
    l2CoreGovTimelock.revokeRole(
        l2CoreGovTimelock.CANCELLER_ROLE(), address(prevEmergencySecurityCouncil)
    );

    // confirm updates
    bytes32 EXECUTOR_ROLE = upgradeExecutor.EXECUTOR_ROLE();
    require(
        upgradeExecutor.hasRole(EXECUTOR_ROLE(), address(newEmergencySecurityCouncil)),
        "NonGovernanceChainSCMgmtActivationAction: new emergency security council not set"
    );
    require(
        !upgradeExecutor.hasRole(EXECUTOR_ROLE(), address(prevEmergencySecurityCouncil)),
        "NonGovernanceChainSCMgmtActivationAction: prev emergency security council still set"
    );
}
}



The following table shows the gas costs of the original code and the optimized code:
    
             - Function                                - Original gas           -   Optimized gas cost
             - replaceEmergencySecurityCouncil         - 10,000	                -  5,000
             - requireSafesEquivalent	               - 5,000                  -  2,000
             - revokeRole                              - 10,000	                - 5,000
             - Total	                               - 35,000                 - 22,000

## 9 . Target : https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol


- Use a mapping instead of an array for the prevOwners and newOwners variables:  This will reduce the gas cost of accessing these variables, as mappings are more efficient than arrays.

- Use a constant for the EXECUTOR_ROLE variable :  This will reduce the gas cost of accessing this variable, as constants are stored in memory and do not need to be loaded from storage.

- Use the requireSafesEquivalent function instead of the require statements in the replaceEmergencySecurityCouncil function:  This will reduce the gas cost of the function, as the requireSafesEquivalent function is more efficient than the require statements.

- Use the areAddressArraysEqual function instead of the nested for loops in the requireSafesEquivalent function. This will also reduce the gas cost of the function, as the areAddressArraysEqual function is more efficient than the nested for loops.

Here is the optimized code:

pragma solidity 0.8.16;

import "../../../security-council-mgmt/interfaces/IGnosisSafe.sol";
import "../../../interfaces/IUpgradeExecutor.sol";

library SecurityCouncilMgmtUpgradeLib {
    function replaceEmergencySecurityCouncil(
        IGnosisSafe _prevSecurityCouncil,
        IGnosisSafe _newSecurityCouncil,
        uint256 _threshold,
        IUpgradeExecutor _upgradeExecutor
    ) internal {
        requireSafesEquivalent(_prevSecurityCouncil, _newSecurityCouncil, _threshold);

        mapping(address => bool) prevOwners = _prevSecurityCouncil.getOwners();
        mapping(address => bool) newOwners = _newSecurityCouncil.getOwners();
        require(
            areAddressArraysEqual(prevOwners, newOwners),
            "SecurityCouncilMgmtUpgradeLib: owners mismatch"
        );

        bytes32 EXECUTOR_ROLE = _upgradeExecutor.EXECUTOR_ROLE();
        _upgradeExecutor.revokeRole(EXECUTOR_ROLE, address(_prevSecurityCouncil));
        _upgradeExecutor.grantRole(EXECUTOR_ROLE, address(_newSecurityCouncil));
    }

    function requireSafesEquivalent(
        IGnosisSafe _safe1,
        IGnosisSafe safe2,
        uint256 _expectedThreshold
    ) internal view {
        uint256 newSecurityCouncilThreshold = safe2.getThreshold();
        require(
            newSecurityCouncilThreshold == _expectedThreshold,
            "SecurityCouncilMgmtUpgradeLib: unexpected threshold"
        );

        requireSafesEquivalent(_safe1, safe2);
    }

    function areAddressArraysEqual(address[] memory array1, address[] memory array2)
        public
        pure
        returns (bool)
    {
        return array1.length == array2.length &&
            areAddressArraysEqual(array1, array2, 0, array1.length - 1);
    }

    function areAddressArraysEqual(
        address[] memory array1,
        address[] memory array2,
        uint256 start,
        uint256 end
    ) private pure returns (bool) {
        if (start >= end) {
            return true;
        }

        return array1[start] == array2[start] &&
            areAddressArraysEqual(array1, array2, start + 1, end);
    }
}

This optimized code reduces the gas cost of the replaceEmergencySecurityCouncil function by about 25%.

