Event emit
In the SecurityCouncilNomineeElectionGovernor contract, there should be an event emit for 'includeNominee' function.
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L290C11-L290C11

Also, there should be an event emit for top nominee selection in the SecurityCouncilMemberElectionGovernorCountingUpgradeable contract.
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L177

There should also be an event emit for '_castVote' function under the SecurityCouncilMemberRemovalGovernor contract
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L223

There should be an event emit for '_countVote' function under the SecurityCouncilMemberRemovalGovernor contract
 https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L163
