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

### Q2

During selecting top nominees, the index of the nominees gives higher priority to the nominees with the same weight but lower index.
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L191

Let's say there are two nominees with weight 1000 and one is index 5 in the array `nominees` but the other has index 6.
In the for loop, when reaching to index 5, 
```
packed = 0x0000000000000000000000000000000000000000000000000000000003E80005
```
And assume it will be placed at `topNomineesPacked[0]`.
Then, when the for loop reaches to index 6, we have:
```
packed = 0x0000000000000000000000000000000000000000000000000000000003E80006
```
Then, this packed value will be compared to `topNomineesPacked[0]`.
Since `packed > topNomineesPacked[0]` (actually the weight for both is the same 0x03E8, but one has index 6 and the other has index 5), the `topNomineesPacked[0]` will be replaced by the `packed` value.

This shows that in case of equal weights, the index becomes important and effective.

### Q3

During swapping member or rotating member, better to have a check that the address of new member is different from the address of the old member.
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L218

### Q4

If governor set the full weight duration to zero, no votes will be counted during the election. It means, whoever casts vote, it will have weight equal to zero.
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L77
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L255

A nonzero check is required to add to the function `setFullWeightDuration`.

### Q5

Voters can nor remove their votes to a contender. For example, during voting period, an user casts vote to a contender. Later, the user notices that this contender is not good fit. But, the user can not unvote to this contender as there is no such mechanism. 

Please note that having such mechanism leads to another attack possibility that a malicious user casts vote to a contender until the last moment (so the contender qualifies for the nominee position, and the next votes to this contender will not be counted), then before the voting ends, the malicious user unvotes to the contender.

### Q6

There is no mechanism that a contender/governor be able to cancel contender-ship. 

Attack scenario:

For example, some very well-known address adds himself as a contender (but the foundation knows beforehand that these contenders will not pass the compliance check). Since, they are very well-known, they attract many votes, so that other contenders will not have enough votes left to qualify for nominee position. After, the election, these contenders will be excluded by the nomineeVetter (as they do not pass the compliance check). The issue is that the number of contenders passing the nominee-threshold (0.2%) is very low, because the well-known contenders had attracted all the available votes.

It is better that governor be able to remove a contender even during voting period. Also, a contender should be able to remove himself.

Or it was better to have a pending mode between the time the nominee election is created and the time the voting starts. During this pending mode, all the contenders should be added, and the nomineeVetter should check their compliance. 