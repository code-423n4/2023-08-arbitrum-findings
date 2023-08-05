### Q1

During casting vote for member election, the votes weight are decreased linearly as time passes to incentives voters to cast early.
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L95
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L225

Let's say:
 - `votingPeriod = 14 days`
 - `fullWeightVotingDeadline_ = 4 days`
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L242C17-L242C43

It means that the weight of any casted vote during day 4 to day 14 (which is 10 days), will be decreased linearly.
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L247-L255

Suppose, Bob would like to cast 100 vote to a nominee on day 6. Since 2 days are passed from the `fullWeightVotingDeadline_ `, the Bob's vote wight will be decreased by `(6 - 4)/10 = %20`, i.e. `100 * 0.2 = 20` votes will be decreased from Bob's 100 votes. So, only 80 votes will be effective.

To bypass this limitation, Bob votes 4 to the nominee. By doing so, the amount of decrease will be `4 * 0.2 = 0.8` which will be rounded down to zero. So, no weight decrease is applied to Bob's vote in this case, and all of his 4 votes will be effective.
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L252

Bob can cast 4 votes 25 times, in which no weight decrease is applied to his votes, so in total 100 votes will be effective even though he is casting vote late and is supposed to lose some weight.