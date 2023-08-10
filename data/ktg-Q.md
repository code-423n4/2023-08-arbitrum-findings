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
