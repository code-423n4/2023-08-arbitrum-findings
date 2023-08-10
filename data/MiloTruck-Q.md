## Findings Summary
| ID | Description | Severity |
| - | - | :-: |
| [L-01](#l-01-consider-checking-that-msgvalue-is-0-in-_execute-of-governor-contracts) | Consider checking that `msg.value` is 0 in `_execute()` of governor contracts | Low |
| [L-02](#l-02-governor-contracts-should-prevent-users-from-directly-transferring-eth-or-tokens) | Governor contracts should prevent users from directly transferring ETH or tokens | Low |
| [L-03](#l-03-governance-can-dos-elections-by-setting-votingdelay-or-votingperiod-more-than-typeuint64max) | Governance can DOS elections by setting `votingDelay` or `votingPeriod` more than `type(uint64).max` | Low |
| [L-04](#l-04-areaddressarraysequal-isnt-foolproof-when-both-arrays-have-duplicate-elements) | `areAddressArraysEqual()` isn't foolproof when both arrays have duplicate elements | Low |
| [L-05](#l-05-missing-duplicate-checks-in-l2securitycouncilmgmtfactorys-deploy) | Missing duplicate checks in `L2SecurityCouncilMgmtFactory`'s `deploy()` | Low |
| [L-06](#l-06-topnominees-could-consume-too-much-gas) | `topNominees()` could consume too much gas | Low |
| [L-07](#l-07-nominees-excluded-using-excludenominee-cannot-be-added-back-using-includenominee) | Nominees excluded using `excludeNominee()` cannot be added back using `includeNominee()` | Low |
| [L-08](#l-08-_execute-in-securitycouncilnomineeelectiongovernor-is-vulnerable-to-blockchain-re-orgs) | `_execute()` in `SecurityCouncilNomineeElectionGovernor` is vulnerable to blockchain re-orgs | Low |
| [N-01](#n-01-check-that-_addresstoremove-and-_addresstoadd-are-not-equal-in-_swapmembers) | Check that `_addressToRemove` and `_addressToAdd` are not equal in `_swapMembers()` | Non-Critical |
| [N-02](#n-02-document-how-ties-are-handled-for-member-elections) | Document how ties are handled for member elections | Non-Critical |
| [N-03](#n-03-relay-is-not-declared-as-payable) | `relay()` is not declared as `payable` | Non-Critical |


## [L-01] Consider checking that `msg.value` is 0 in `_execute()` of governor contracts

In Openzeppelin's `GovernorUpgradeable`, `execute()` is declared as `payable`:

[GovernorUpgradeable.sol#L295-L300](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/release-v4.7/contracts/governance/GovernorUpgradeable.sol#L295-L300)

```solidity
    function execute(
        address[] memory targets,
        uint256[] memory values,
        bytes[] memory calldatas,
        bytes32 descriptionHash
    ) public payable virtual override returns (uint256) {
```

This makes it possible for users to accidentally transfer ETH to the governor contracts when calling `execute()`.

### Recommendation

In [`SecurityCouncilNomineeElectionGovernor`](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol), [`SecurityCouncilMemberRemovalGovernor`](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol) and [`SecurityCouncilMemberElectionGovernor`](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol), consider overriding `_execute()` and reverting if `msg.value` is not 0. This ensures that users cannot accidentally lose their ETH while calling `execute()`.

## [L-02] Governor contracts should prevent users from directly transferring ETH or tokens

Openzeppelin's `GovernorUpgradeable` contract contains [`receive()`](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/release-v4.7/contracts/governance/GovernorUpgradeable.sol#L88-L93), [`onERC721Received()`](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/release-v4.7/contracts/governance/GovernorUpgradeable.sol#L568-L575), [`onERC1155Received()`](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/release-v4.7/contracts/governance/GovernorUpgradeable.sol#L580-L588) and [`onERC1155BatchReceived()`](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/release-v4.7/contracts/governance/GovernorUpgradeable.sol#L590-L609) to allow inheriting contracts to receive ETH and tokens. 

However, this allows users to accidentally transfer their ETH/tokens to the governor contracts, which will then remain stuck until they are rescued by governance.

### Recommendation

In [`SecurityCouncilNomineeElectionGovernor`](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol), [`SecurityCouncilMemberRemovalGovernor`](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol) and [`SecurityCouncilMemberElectionGovernor`](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol), consider overriding these functions and making them revert. This prevents users from accidentally transferring ETH/tokens to the contracts.

## [L-03] Governance can DOS elections by setting `votingDelay` or `votingPeriod` more than `type(uint64).max`

In the `propose()` function of Openzeppelin's `GovernorUpgradeable` contract, `votingDelay` and `votingPeriod` are cast from `uint256` to `uint64` safely:

[GovernorUpgradeable.sol#L271-L272](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/release-v4.7/contracts/governance/GovernorUpgradeable.sol#L271-L272)

```solidity
        uint64 snapshot = block.number.toUint64() + votingDelay().toUint64();
        uint64 deadline = snapshot + votingPeriod().toUint64();
```

Therefore, if either of these values are set to above `type(uint64).max`, `propose()` will revert.


### Recommendation

In [`SecurityCouncilNomineeElectionGovernor`](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol), [`SecurityCouncilMemberRemovalGovernor`](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol) and [`SecurityCouncilMemberElectionGovernor`](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol), consider overriding the `setVotingDelay()` and `setVotingPeriod()` functions to check that `votingDelay` and `votingPeriod` are not set to values above `type(uint64).max`.


## [L-04] `areAddressArraysEqual()` isn't foolproof when both arrays have duplicate elements

The `areAddressArraysEqual()` function is used to check if `array1` and `array2` contain the same elements. It does so by checking that each element in `array1` exists in `array2`, and vice versa:

[SecurityCouncilMgmtUpgradeLib.sol#L61-L85](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L61-L85)

```solidity
        for (uint256 i = 0; i < array1.length; i++) {
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
        }
```

However, this method isn't foolproof when both `array1` and `array2` contain duplicate elements. For example:
- `array1 = [1, 1, 2]`
- `array2 = [1, 2, 2]` 

Even though both arrays are not equal, `areAddressArraysEqual()` will return true as they have the same length and all elements in one array exist in the other.

### Recommendation

Consider checking that both arrays do not contain duplicate elements.

## [L-05] Missing duplicate checks in `L2SecurityCouncilMgmtFactory`'s `deploy()`

In `L2SecurityCouncilMgmtFactory.sol`, the `deploy()` function only checks that every address in every cohort is an owner in `govChainEmergencySCSafe`:

[L2SecurityCouncilMgmtFactory.sol#L111-L121](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol#L111-L121)

```solidity
        for (uint256 i = 0; i < dp.firstCohort.length; i++) {
            if (!govChainEmergencySCSafe.isOwner(dp.firstCohort[i])) {
                revert AddressNotInCouncil(owners, dp.firstCohort[i]);
            }
        }

        for (uint256 i = 0; i < dp.secondCohort.length; i++) {
            if (!govChainEmergencySCSafe.isOwner(dp.secondCohort[i])) {
                revert AddressNotInCouncil(owners, dp.secondCohort[i]);
            }
        }
```

However, there is no check to ensure that `firstCohort` and `secondCohort` do not contain any duplicates, or that any address in one cohort is not in the other. This makes it possible for the `SecurityCouncilManager` contract to be deployed with incorrect cohorts.

### Recommendation

Consider checking the following:
- `firstCohort` and `secondCohort` do not contain any duplicates
- An address that is in `firstCohort` must not be in `secondCohort`, and vice versa.

## [L-06] `topNominees()` could consume too much gas

In `SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol`, the `topNominees()` function is extremely gas-intensive due to the following reasons:
- `_compliantNominees()` copies the entire nominees array  the storage of the `SecurityCouncilNomineeElectionGovernor` contract:

[SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L178](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L178)

```solidity
        address[] memory nominees = _compliantNominees(proposalId);
```

- `selectTopNominees()` iterates over all nominees and in the worst-case scenario, calls `LibSort.insertionSort()` in each iteration:

[SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L205-L212](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L205-L212)

```solidity
        for (uint16 i = 0; i < nominees.length; i++) {
            uint256 packed = (uint256(weights[i]) << 16) | i;

            if (topNomineesPacked[0] < packed) {
                topNomineesPacked[0] = packed;
                LibSort.insertionSort(topNomineesPacked);
            }
        }
```

If the number of nominees is too large for an election, there is a significant chance that the `topNominees()` function will consume too much gas and revert due to an out-of-gas error. 

If this occurs, member elections will be stuck permanently as proposals cannot be executed. This is because [`_execute()`](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L112-L133) calls `topNominees()` to select the top nominees to replace the cohort in `SecurityCouncilManager`. 

The number of nominees for an election is implicitly limited by the percentage of votes a contender needs to become a nominee. Currently, this is set to 0.2% which makes the maximum number of nominees 500. However, this also means that number of nominees could increase significantly should the percentage be decreased in the future. 

## [L-07] Nominees excluded using `excludeNominee()` cannot be added back using `includeNominee()`

In `SecurityCouncilNomineeElectionGovernor`, once nominees are excluded from the election by the nominee vetter using [`excludeNominee()`](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L263-L283), they cannot be added back using the [`includeNominee()`](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L285-L317).

This is because excluded nominees are not removed from the array of nominees, but are simply marked as excluded in the `isExcluded` mapping:

[SecurityCouncilNomineeElectionGovernor.sol#L279-L280](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L279-L280)

```solidity
        election.isExcluded[nominee] = true;
        election.excludedNomineeCount++;
```

Therefore, following check in `includeNominee()` will still fail when it is called for excluded nominees:

[SecurityCouncilNomineeElectionGovernor.sol#L296-L298](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L296-L298)

```solidity
        if (isNominee(proposalId, account)) {
            revert NomineeAlreadyAdded(account);
        }
```

This could become a problem if the nominee vetter accidentally calls `excludeNominee()` on the wrong nominee, or if there is some other legitimate reason a previously excluded nominee needs to be added back to the election.

## [L-08] `_execute()` in `SecurityCouncilNomineeElectionGovernor` is vulnerable to blockchain re-orgs

In `SecurityCouncilNomineeElectionGovernor`, when a user calls `execute()`, he only needs to provide the `proposalId` and current election index (in `callDatas`):

[SecurityCouncilNomineeElectionGovernor.sol#L324-L329](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L324-L329)

```solidity
    function _execute(
        uint256 proposalId,
        address[] memory, /* targets */
        uint256[] memory, /* values */
        bytes[] memory callDatas,
        bytes32 /*descriptionHash*/
    ) internal virtual override {
```

This will then call `proposeFromNomineeElectionGovernor()` to start the member elections with the current nominees.

However, as the `_execute()`'s parameters do not contain any information about the list of qualified nominees, `execute()` is vulnerable to blockchain re-orgs. For example:

- Assume the following transactions are called in different blocks:
  - Block 1: Governance calls [`excludeNominee()`](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L263-L283) to exclude nominee A from member elections.
  - Block 2: A user cals `execute()` to start member elections.
- A blockchain re-org occurs, block 1 is dropped and block 2 is kept.
- Now, when `execute()` is called in block 2, nominee A will still be in the list of compliant nominees, making him qualfiied for member elections.

Note that the risk of a blockchain re-org occurring is extremely small as Arbitrum cannot re-org unless the L1 itself re-orgs. Nevertheless, there is still a non-zero possibility of such a scenario occurring.

## [N-01] Check that `_addressToRemove` and `_addressToAdd` are not equal in `_swapMembers()`

In `_swapMembers()`, consider checking that `_addressToRemove` and `_addressToAdd` are not the same address:

[SecurityCouncilManager.sol#L218-L229](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L218-L229)

```diff
    function _swapMembers(address _addressToRemove, address _addressToAdd)
        internal
        returns (Cohort)
    {
        if (_addressToRemove == address(0) || _addressToAdd == address(0)) {
            revert ZeroAddress();
        }
+       if (_addressToRemove == _addressToAdd) {
+           revert CannotSwapSameMembers();
+       }    
        Cohort cohort = _removeMemberFromCohortArray(_addressToRemove);
        _addMemberToCohortArray(_addressToAdd, cohort);
        _scheduleUpdate();
        return cohort;
    }
```

This would prevent scheduling unnecessary updates as there no changes to the security council members. 

## [N-02] Document how ties are handled for member elections

In the [Arbitrum Constitution](https://docs.arbitrum.foundation/dao-constitution), there is no specification on how members are chosen in the event nominees are tied for votes.

Currently, `selectTopNominees()` simply picks the first 6 nominees after `LibSort.insertionSort()` is called, which means the nominee selected is random in the event they tie:

[SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L203-L217](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L203-L217)

```solidity
        uint256[] memory topNomineesPacked = new uint256[](k);

        for (uint16 i = 0; i < nominees.length; i++) {
            uint256 packed = (uint256(weights[i]) << 16) | i;

            if (topNomineesPacked[0] < packed) {
                topNomineesPacked[0] = packed;
                LibSort.insertionSort(topNomineesPacked);
            }
        }

        address[] memory topNomineesAddresses = new address[](k);
        for (uint16 i = 0; i < k; i++) {
            topNomineesAddresses[i] = nominees[uint16(topNomineesPacked[i])];
        }
```

This could be confusing for users who expect tiebreaks to be handled in a deterministic manner (eg. whoever got the number of votes first).

### Recommendation

Consider documenting how voting ties are handled in the [Arbitrum Constitution](https://docs.arbitrum.foundation/dao-constitution) to prevent confusion.

## [N-03] `relay()` is not declared as `payable`

In `SecurityCouncilNomineeElectionGovernor`, although `relay()` makes calls with `AddressUpgradeable.functionCallWithValue()`, it is not declared as `payable`:

[SecurityCouncilNomineeElectionGovernor.sol#L254-L261](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L254-L261)

```solidity
    function relay(address target, uint256 value, bytes calldata data)
        external
        virtual
        override
        onlyOwner
    {
        AddressUpgradeable.functionCallWithValue(target, data, value);
    }
```

This limits the functionality of `relay()`, as governance will not be able to send ETH to this contract and transfer the ETH to `target` in a single call to `relay()`.

### Recommendation

Consider declaring `relay()` as `payable`:

[SecurityCouncilNomineeElectionGovernor.sol#L254-L261](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L254-L261)

```solidity
    function relay(address target, uint256 value, bytes calldata data)
        external
+       payable
        virtual
        override
        onlyOwner
    {
        AddressUpgradeable.functionCallWithValue(target, data, value);
    }
```

This applies to the `relay()` function in [`SecurityCouncilMemberElectionGovernor`](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol) and [`SecurityCouncilMemberRemovalGovernor`](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol) as well.