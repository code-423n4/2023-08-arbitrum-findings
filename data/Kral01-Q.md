## 001-QA 

In [SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol) the function [selectTopNominees](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L191C4-L212C10) has no way to mitigate issues when there are `no votes` for any members or there are only less than 6 members with `non-zero` votes.

This can cause issues and make the voting system faulty. 

Recommendation is to ensure proper logic is implemented to catch these faults.

## 002-QA

In [SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol) the function [selectTopNominees](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L191C4-L212C10) has no way to mitigate issues when there are same number of votes for members in top places or if there are `two candidates with equal number of votes in the 6th position`, then there is no error handling mechanism to decide who will be in the elected council.

This can cause issues and make the voting system faulty. 

Recommendation is to ensure proper logic is implemented to catch these type of issues to decide who will move to the newly elected council.

