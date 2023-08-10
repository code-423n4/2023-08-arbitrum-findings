# Findings Summary

| ID              | Title                                                                                                                 | Severity     |
| --------------- | --------------------------------------------------------------------------------------------------------------------- | ------------ |
| [L-01](#l-01)   | `SecurityCouncilNomineeElectionGovernor` : No Check On `nomineeVettingDuration`                                       | Low          |
| [L-02](#l-02)   | `SecurityCouncilManager` : No Check for The `cohortSize` When Initialized                                             | Low          |
| [L-03](#l-03)   | `L2SecurityCouncilMgmtFactory::deploy`: No Check on The Address of The Voting Token                                   | Low          |
| [L-04](#l-04)   | `L2SecurityCouncilMgmtFactory::deploy` : No Check If The Cohort Members Are Assigned to The Correct Cohort            | Low          |
| [L-05](#l-05)   | `SecurityCouncilManager`: Cohorts Vacancies Can't Be Filled Via Election                                              | Low          |
| [L-06](#l-06)   | `SecurityCouncilMemberElectionGovernor`: Setting `fullWeightDuration` Equal to `votingPeriod()` Will Break The Voting | Low          |
| [NC-01](#nc-01) | `SecurityCouncilManager::_swapMembers` : No Check If The Two Swapped Addresses Are The Same                           | Non Critical |

# Low

## [L-01] `SecurityCouncilNomineeElectionGovernor` : No Check On `nomineeVettingDuration` <a id="l-01" ></a>

## Details

- In `SecurityCouncilNomineeElectionGovernor` contract : the duration for nominees vetting/compliance checking is set upon contract initialization.
- As per [constitution](https://forum.arbitrum.foundation/t/proposal-security-council-elections-proposed-implementation-spec/15425);this stage is done after the 7 days of nominees election :
  > The Foundation will be given 14 days to vet the prospective nominees.
- And once this value is set; it can never be reset again.

## Impact

- Since some operations can only be performed during the vetting period (as in `excludeNominee`); this will make the time window for these operations either narrow (if it's set to a lower vetting period than intended by design) or wide (if it's set to a higher vetting period than intended by design).

## Proof of Concept

[initialize function/Line 109-111](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L109-L111)

```solidity
File: governance/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol
Line 109-111:
        __SecurityCouncilNomineeElectionGovernorTiming_init(
            params.firstNominationStartDate, params.nomineeVettingDuration
        );
```

[SecurityCouncilNomineeElectionGovernorTiming contract/Line 65](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol#L65-L65)

```solidity
File: governance/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol
Line 65: nomineeVettingDuration = _nomineeVettingDuration;
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

In `SecurityCouncilNomineeElectionGovernor`: check that `params.nomineeVettingDuration` complies with the designed duration (14 days) before initializing it.

## [L-02] `SecurityCouncilManager` : No Check for The `cohortSize` When Initialized <a id="l-02" ></a>

## Details

- In `SecurityCouncilManager` contract : each time this contract is re-deployed; it's initialized with the current first and second security council cohort members.

- The checks that are made on the cohorts sizes :

  1.  In `L2SecurityCouncilMgmtFactory::deploy` upon contract deployment: if the sum of their sizes equals to the number of security council current cohort members:

  ```solidity
   if (owners.length != (dp.firstCohort.length + dp.secondCohort.length)) {
         revert InvalidCohortsSize(owners.length, dp.firstCohort.length, dp.secondCohort.length);
   }
  ```

  2. In `SecurityCouncilManager::initialize`upon contract initialization: if their sizes are equal:

  ```solidity
  if (_firstCohort.length != _secondCohort.length) {
              revert CohortLengthMismatch(_firstCohort, _secondCohort);
          }
  ```

- Then the `cohortSize` will be set equal to `_firstCohort.length`; and there's noway to reset this value again; unless a new governor contract is deployed; and this will require all governor contracts to be re-deployed again as well.

- And as per [constitution](https://docs.arbitrum.foundation/dao-constitution); the cohort size is 6 members.

- Since this check is not done upon initialization; then `cohortSize` could be assigned any value less than 6.

- The scenario of setting the `cohortSize` to a lower value than designed is very likely to happen; as the previous cohorts members might be less than 6 members due to the possibility of any cohort member to be removed and their vacancy is not filled (the two cohorts must be equl to initialize the contract).

## Impact

- This implementation deviates from the design; as the next security council cohorts sizes will be controlled by the wrong value of `cohortSize`, and it will never be possible to elect/add 6 cohort members.

- And this will make the next elections to fill **5** members only, as the number of selected top nominees depend on the `cohortSize`.

- Also as mentioned in the constitution: the security council cohort members are responsible of voting on emergency actions (with 9 out of 12 members approval) and non emergency actions (with 7 out of 12 members approval); so for example: if the security council cohorts were 3 members each upon initialization; then any proposed actions will never be executed as the number falls below voting threshold.

## Proof of Concept

- Code:

  [L2SecurityCouncilMgmtFactory contract/deploy function/Line 107-109](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol#L107-L109)

  ```solidity
  File: governance/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol
  Line 107-109:
          if (owners.length != (dp.firstCohort.length + dp.secondCohort.length)) {
              revert InvalidCohortsSize(owners.length, dp.firstCohort.length, dp.secondCohort.length);
          }
  ```

  [SecurityCouncilManager contract/initialize function/Line 97-102](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L97-L102)

  ```solidity
  File: governance/src/security-council-mgmt/SecurityCouncilManager.sol
  Line 97-102:
          if (_firstCohort.length != _secondCohort.length) {
              revert CohortLengthMismatch(_firstCohort, _secondCohort);
          }
          firstCohort = _firstCohort;
          secondCohort = _secondCohort;
          cohortSize = _firstCohort.length;
  ```

- Foundry PoC:

1.  The following modifications are done to the `setup` function & `testInitialization` test in `SecurityCouncilManagerTest`, to test if the contract can be initialized with firstCohort & secondCohort with less than 6 members, test set-up as follows:

    - modify the lengths of both cohorts to be 5 instead of 6:

      ```diff
      -     address[] firstCohort = new address[](6);
      +     address[] firstCohort = new address[](5);
      -     address[] secondCohort = new address[](6);
      +     address[] secondCohort = new address[](5);
      ```

    - in `setUp`: five members only are added for each cohort instead of 6

      ```diff
      function setUp() public {
      //...... some code
      - for (uint256 i = 0; i < 6; i++) {
      + for (uint256 i = 0; i < 5; i++) {
      secondCohort[i] = _secondCohort[i];
      firstCohort[i] = _firstCohort[i];
      bothCohorts.push(_firstCohort[i]);
      bothCohorts.push(_secondCohort[i]);
      newCohort[i] = _newCohort[i];
      newCohortWithADup[i] = _newCohortWithADup[i];
      }
      ```

    - in `testInitialization`: assert that the cohortSize can be initialized with less than 6 members

      ```diff
      function testInitialization() public {
      //...... some code
      + assertEq(scm.getFirstCohort().length, 5);
      + assertEq(scm.getFirstCohort().length, scm.cohortSize());
      ```

2.  Test result:

    ```bash
    $ forge test --match-test testInitialization
    [PASS] testInitialization() (gas: 200199)
    Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 7.65ms
    Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)
    ```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

When initializing the `SecurityCouncilManager` contract: check that cohorts sizes comply with the designed value (6 members for each cohort),and if it's deemed acceptable to have less than 6 members/cohort (assuming that some cohort members were removed and their vacancies were not filled); then set cohortSize to a constant value of 6:

```diff
File: governance/src/security-council-mgmt/SecurityCouncilManager.sol
    function initialize(
        address[] memory _firstCohort,
        address[] memory _secondCohort,
        SecurityCouncilData[] memory _securityCouncils,
        SecurityCouncilManagerRoles memory _roles,
        address payable _l2CoreGovTimelock,
        UpgradeExecRouteBuilder _router
    ) external initializer {
       //... some code

-102:   cohortSize = _firstCohort.length;
+102:   cohortSize = 6;

       //... some code
```

## [L-03] `L2SecurityCouncilMgmtFactory::deploy`: No Check on The Address of The Voting Token <a id="l-03"></a>

## Details

- In `L2SecurityCouncilMgmtFactory` contract: is a factory contract to deploy and initialize election government contracts and security manager contract on the governance chain via `deploy` function.

- When initializing these contract: the address of the voting token is supposed to be the address of $ARB token on Arbitrum chain; but there's no check made on the address of the assigned token if it's the address of $ARB token.

## Impact

- Assigning a wrong address for the voting token will require re-deployment of the election government contracts.

## Proof of Concept

[Line 200](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol#L200)
[Line 225](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol#L225)
[Line 235](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol#L235)

```solidity
File: governance/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol
Line 200: _token: IVotesUpgradeable(dp.arbToken),
Line 225: token: IVotesUpgradeable(dp.arbToken),
Line 235: _token: IVotesUpgradeable(dp.arbToken),
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

Check that the address of the voting token is the address of the $ARB token before contracts initialization.

## [L-04] `L2SecurityCouncilMgmtFactory::deploy` : No Check If The Cohort Members Are Assigned to The Correct Cohort <a id="l-04" ></a>

## Details

In `L2SecurityCouncilMgmtFactory` contract: when the security manager contract is deployed and initialized via `deploy` function; the only check made on the assigned cohort members is whether they are members of the security council or not; but there's no check made if each member belongs correctly to the assigned cohort (might be in cohort one and assigned to cohort two).

## Impact

The impact is low as this wrong assignment of cohort members can be fixed in `SecurityCouncilManager` contract.

## Proof of Concept

[Line 111-121](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol#L111-L121)

```solidity
File: governance/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol
Line 111-121:
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

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

Add a mechanism to check if each cohort member belongs to the correct cohort before initializing `SecurityCouncilManager` contract

## [L-05] `SecurityCouncilManager`: Cohorts Vacancies Can't Be Filled Via Election <a id="l-05" ></a>

## Details

- When removing a security council cohort member: as per [constitution](https://docs.arbitrum.foundation/dao-constitution):
  the vacancy can be filled either directly by the govChainEmergencySecurityCouncil or by the next election.

- If it's decided to fill the vacancy through an election (which is going to be started as proposal in `SecurityCouncilNomineeElectionGovernor` contract); it will not be possible to start an election before 6-months passed from the last election, which makes it impossible to fill that vacancy via election.

- Another scenario: if two members from the **second** cohorts were removed, and the next election is going to be for the **first** cohort; so the vacancies of the removed second cohort members will never be filled by election, even after 6-months passed from the last election.

## Impact

- This will make filling vacancies in cohorts only possible via direct addition by the govChainEmergencySecurityCouncil, and not by election.

## Proof of Concept

- Code:

  [L2SecurityCouncilMgmtFactory contract/Line 152](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol#L152)

        ```solidity
        File: governance/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol
        Line 152:  memberAdder: dp.govChainEmergencySecurityCouncil,
        ```

  [SecurityCouncilManager contract/Line 176](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L176)

        ```solidity
        File: governance/src/security-council-mgmt/SecurityCouncilManager.sol
        Line 176:  function addMember(address _newMember, Cohort _cohort) external onlyRole(MEMBER_ADDER_ROLE) {
        ```

  [SecurityCouncilNomineeElectionGovernor contract/createElection function](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L166-L169)

        ```solidity
        File: Breadcrumbsgovernance/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol
        Line 166-167:
                uint256 thisElectionStartTs = electionToTimestamp(electionCount);
                if (block.timestamp < thisElectionStartTs) {
                    revert CreateTooEarly(block.timestamp, thisElectionStartTs);// @audit : this will revert when a new election is being started before 6-months from the last election
                }
        ```

- Foundry PoC:

1. `testRemovedCohortMembersCantBeAddedByElection()` test is added to `SecurityCouncilNomineeElectionGovernorTest.t.sol` file:

   - where at first a proposal is created to add second cohort members.
   - then after 3 months; the MEMBER_REMOVER_ROLE removes 2 members from the second cohort (not demonstrated by the test here).
   - when these members vacancies in the second cohort are going to be filled via an election; this won't work as the elction is started before 6-months from the last election.
   - also the next election is going to replace the **FIRST** cohort members not the second; so the vacancies in the second cohort will never be filled by election.

   ```solidity
   function testRemovedCohortMembersCantBeAddedByElection() public {
   //1. first an election proposal to replace SECOND cohort members is created and executed: (copied from testExecutedProposalIdState test and updated to demonstrate the vulnerability)
   uint256 proposalId = _propose();
   uint256 electionIndex = governor.electionCount() - 1;

   //2. nomineeVetter adds 6 nominees:
   vm.roll(governor.proposalVettingDeadline(proposalId) + 1);
   vm.startPrank(initParams.nomineeVetter);
   for (uint8 i = 0; i < cohortSize; i++) {
   _mockCohortIncludes(Cohort.SECOND, _contender(i), false);
   governor.includeNominee(proposalId, _contender(i));
   }
   vm.stopPrank();

   //3. execute:
   vm.mockCall(
   address(initParams.securityCouncilMemberElectionGovernor),
   "",
   abi.encode(proposalId)
   );
   vm.expectCall(
   address(initParams.securityCouncilMemberElectionGovernor),
   abi.encodeWithSelector(
       initParams
       .securityCouncilMemberElectionGovernor
       .proposeFromNomineeElectionGovernor
       .selector,
       electionIndex
   )
   );
   _execute(electionIndex, "");
   assertEq(uint256(governor.currentCohort()), uint256(Cohort.FIRST));
   assertEq(uint256(governor.otherCohort()), uint256(Cohort.SECOND));

   //4. now assume after 3-months the MEMBER_REMOVER_ROLE in SecurityCouncilManager contract removes two members from the second cohort; and these members are going to be added via an election, but  nominee election proposal wont be createdL
   vm.warp(7889400); //represents 3-months in seconds
   uint256 expectedStartTimestamp = _datePlusMonthsToTimestamp(
   initParams.firstNominationStartDate,
   6
   );
   vm.expectRevert(
   abi.encodeWithSelector(
       SecurityCouncilNomineeElectionGovernor.CreateTooEarly.selector,
       block.timestamp,
       expectedStartTimestamp
   )
   );
   governor.createElection();
   }
   ```

2. Test result:

   ```bash
   $ forge test --match-test testRemovedCohortMembersCantBeAddedByElection
   [PASS] testRemovedCohortMembersCantBeAddedByElection() (gas: 545760)
   Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 6.64ms
   Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)
   ```

## Tools Used

Manual Testing & Foundry.

## Recommended Mitigation Steps

Add a mechanism to enable creating elections for cohort vacancies between the main elections (which occurs each 6-months).

## [L-06] `SecurityCouncilMemberElectionGovernor`: Setting `fullWeightDuration` Equal to `votingPeriod()` Will Break The Voting<a id="l-06" ></a>

## Details

- `SecurityCouncilMemberElectionGovernor` contract inherits the counting contract `SecurityCouncilMemberElectionGovernorCountingUpgradeable`; where the logic of voting and voting weights is set.
- When `SecurityCouncilMemberElectionGovernor` is initialized; it sets the value of the fullWeightDuration that is used to calculate votes weight with time & votingPeriod of the proposals.
- The only check made on these values is if `_fullWeightDuration > _votingPeriod`.
- So if these values are set equal, then the inherited `votesToWeight` function will always revert due to division by zero (as the decreasingWeightDuration will be zero).

## Impact

This will make `votesToWeight` and all the functions invoking it temporarily inaccessible (break the voting); unless the owner sets a new fullWeightDuration value.

## Proof of Concept

[SecurityCouncilMemberElectionGovernor/initialize function](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L56-L58)

```solidity
File: governance/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol
Line 56-58:
        if (_fullWeightDuration > _votingPeriod) {
            revert InvalidDurations(_fullWeightDuration, _votingPeriod);
        }
```

[SecurityCouncilMemberElectionGovernorCountingUpgradeable/votesToWeight function](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L242-L253)

```solidity
File: governance/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol
Line 242-253:
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
            votes * (blockNumber - fullWeightVotingDeadline_) / decreasingWeightDuration;
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

Update `SecurityCouncilMemberElectionGovernor/initialize function` to revert if the two values are equal:

```diff
-       if (_fullWeightDuration > _votingPeriod) {
+       if (_fullWeightDuration >= _votingPeriod) {
            revert InvalidDurations(_fullWeightDuration, _votingPeriod);
        }
```

# Non Critical

## [NC-01] `SecurityCouncilManager::_swapMembers` : No Check If The Two Swapped Addresses Are The Same <a id="nc-01" ></a>

## Details

- In `SecurityCouncilManager` contract/ `_swapMembers` function: there's no check if the two swapped addresses are the same (this check is not done in the functions where this function is called as well).
- When this function is called (by another function in the contract) with two similar addresses; it will emit a scheduled update to other chains while there's no actual change has happened.

## Proof of Concept

[\_swapMembers function/Line 218-229](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L218-L229)

```solidity
File: governance/src/security-council-mgmt/SecurityCouncilManager.sol
Line 218-229:
   function _swapMembers(address _addressToRemove, address _addressToAdd)
        internal
        returns (Cohort)
    {
        if (_addressToRemove == address(0) || _addressToAdd == address(0)) {
            revert ZeroAddress();
        }
        Cohort cohort = _removeMemberFromCohortArray(_addressToRemove);
        _addMemberToCohortArray(_addressToAdd, cohort);
        _scheduleUpdate();
        return cohort;
    }
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

Update `swapMemebrs` function to revert on similar addresses:

```diff
   function _swapMembers(address _addressToRemove, address _addressToAdd)
        internal
        returns (Cohort)
    {
        if (_addressToRemove == address(0) || _addressToAdd == address(0)) {
            revert ZeroAddress();
        }
+       require(_addressToRemove != _addressToAdd,"duplicate addresses");
        Cohort cohort = _removeMemberFromCohortArray(_addressToRemove);
        _addMemberToCohortArray(_addressToAdd, cohort);
        _scheduleUpdate();
        return cohort;
    }
```
