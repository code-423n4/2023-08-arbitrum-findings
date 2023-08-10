# Low issues
### L-01: Insufficient check for cohorts initialization in `SecurityCouncilManager`.initialize
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L89
```solidity
function initialize(
        address[] memory _firstCohort,
        address[] memory _secondCohort,
        SecurityCouncilData[] memory _securityCouncils,
        SecurityCouncilManagerRoles memory _roles,
        address payable _l2CoreGovTimelock,
        UpgradeExecRouteBuilder _router
    ) external initializer {
        if (_firstCohort.length != _secondCohort.length) {
            revert CohortLengthMismatch(_firstCohort, _secondCohort);
        }
        firstCohort = _firstCohort;
        secondCohort = _secondCohort;
        cohortSize = _firstCohort.length;
        ...
```
As you can see the only check on `firstCohort` and `secondCohort` is assuring their length are the same. This is insufficient and can cause many issues, for example:
- The `firstCohort` and `secondCohort` could be of any size, violating the Arbitrum Constitution https://docs.arbitrum.foundation/dao-constitution#section-3-the-security-council, in which it states *The Security Council is a committee of 12 members who are signers of a multi-sig wallet*. Since variable `cohortSize` is set as the length of `firstCohort` and `secondCohort` at initialization, a 0 size cohort cannot be replaced by calling `replaceCohort` or added new member by calling `addMember`. The protocol will basically fail to function.
- One member could end up in both cohorts
- Duplicates in both cohorts


### L-02: `SecurityCouncilMemberElectionGovernorCountingUpgradeable`.votesToWeight precision loss may lead to users not affecting during decreasing weight period.
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L236-#L256
According to the documentation, the member voting process is designed to encourage voters to cast their vote early. Their voting power will eventually decay if they do not cast their vote within the first 7 days:

- 0 - 7 days. Votes cast will carry weight 1 per token
- 7 - 21 days. Votes cast will have their weight linearly decreased based on the amount of time that has passed since the 7 day point. By the 21st day, each token will carry a weight of 0.

The decreasing calculation is implemented as follow:
```solidity
   uint256 endBlock = proposalDeadline(proposalId);
   uint256 decreasingWeightDuration = endBlock - fullWeightVotingDeadline_;
   uint256 decreaseAmount =
          votes * (blockNumber - fullWeightVotingDeadline_) / decreasingWeightDuration;
   return _downCast(votes - decreaseAmount);
```
The functions calculates `decreaseAmount` and then subtract that from original `votes`. However, since this is the integer division, it may end up = 0 if `votes * (blockNumber - fullWeightVotingDeadline_` < `decreasingWeightDuration`. A malicious user can vote with a small amount and get around this.

### L-03: `SecurityCouncilNomineeElectionGovernor.sol`.includeNominee should check if address != address(0)
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L290-#L317
```solidity
function includeNominee(uint256 proposalId, address account) external onlyNomineeVetter {
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
Function `includeNominee` allows nomineeVetter to add an account to nominee list of the current list length < cohort size. However, it doesn't check if `account` is a valid address != address(0). If this happens, when the proposal is passed to `SecurityCouncilMemberElectionGovernor` and executed, `SecurityCouncilManager` will not accept this address and reverts, blocking subsequent elections.

### L-04: There is no way to update `upExecLocations` in contract `UpgradeExecRouteBuilder`
The variable `upExecLocations` is of type `mapping(uint256 => UpExecLocation)`, it decides which chainId is valid for adding security council member in `SecurityConcilManager`:
```solidity
function _addSecurityCouncil(SecurityCouncilData memory _securityCouncilData) internal {
        ...
        if (!router.upExecLocationExists(_securityCouncilData.chainId)) {
            revert SecurityCouncilNotInRouter(_securityCouncilData);
        }
        ...
}

function upExecLocationExists(uint256 _chainId) public view returns (bool) {
        return upExecLocations[_chainId].upgradeExecutor != address(0);
}
```
This variable `upExecLocations` is only updated at initialization and no functions to update it after that; therefore, the protocol cannot add new security council with chainId different than what's initialized.