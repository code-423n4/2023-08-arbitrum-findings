# Approach taken in evaluating the codebase
- The documentation was read thoroughly which includes:
. The Arbitrum consitution: https://docs.arbitrum.foundation/dao-constitution
. The "Security Council Elections Proposed Implementation Spec" proposal: https://forum.arbitrum.foundation/t/proposal-security-council-elections-proposed-implementation-spec/15425
. The architectural overview graphic (which was very helpful): https://raw.githubusercontent.com/ArbitrumFoundation/governance/c18de53820c505fc459f766c1b224810eaeaabc5/docs/security-council-colors.png
- After that, all contracts were sorted by number of lines of code and approached from the smallest number
- During the review the code was heavily documented
- Arbitrum-related issues (e.g. block reorgs, usage of block.number in Arbitrum, etc. were inspected via https://solodit.xyz/)
- Unclear behavior of the protocol was discussed via Discord with protocol members who were very supportive
- Audit markers were left in code to indicate findings and their severity

# Codebase quality analysis
- The codebase was in total deemed as mature
- Some inconsistencies in naming variables (e.g. TIMELOCK_PROPOSAL_ROLE vs. PROPOSER_ROLE) were encountered
- Enums were not consistently used throughout the code base (VoteType)
- The excessive amount of inheritance and the associated amount of initializations within a contract makes it hard to keep an overview and easy to miss required initializations (bot results already discovered some of these issues)
- Some parts of the code could use more comments to explain what happens. One good example is the packing of weights and appending the nominee index in the "selectTopNominees" function in SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol (https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L191). Here the appended index actually influences the ranking for same weights in nominees which is undocumented.
- The test suite seemed well organized but can be enhanced (currently 94%)
- Several typos in the codebase deminished the impression of a very mature codebase a bit
- There was no sign in the protocol that a static analyzer like slither was used. This is recommened.
- The Makefile was helpful to get started and run tests

# Centralization risks
- After the contract deployer admin account the security council with the power for emergency activities (circumventing DAO vote) is the greatest centralization risk
- Some processes happen off-chain and are not transparent for the DAO members (e.g. randomization of council members if a cohort could not be filled). A list of processes that happen off-chain could help establishing trust from DAO members.

# Mechanism review
- The implementation of the full election process as a state machine through multiple porposals is good to undertand and transparent
- Enforcing the time delay through a roundtrip from L1 to L2 and back to L1 is a good solution to enable users to opt out.
- Potentially a final time window of 3 days to opt out from the protocol is too low. One would always need to have an ear on what is happening on Arbitrum to not miss this timeframe.
- No precautions are implemented in code against the vetter making mistakes that he cannot correct any more (see the QA findings for details):
. removing a nominee cannot be re-added
. after the vetting period a nominee cannot be removed anymore
These edge cases should be documented. It also implies that the vetter acts with an enourmous level of care. A vetter is also human and human make mistakes. Potentially the implementation of a mechanism to have a small amount of time after an action would allow the vetter to revert his faulty action while maintaining trust from the DAO as it is clear that this is only for mistakes in a small time window.

# Learnings
- This was the auditor's first audit of a protocol where a deeper look was taken in the L1 <-> L2 communication
- This was also the auditor's first participation in a contest fully focused on governance
- The way votes are packaged was seen the first time
- Looking into previous audit contest findings around the topics of Arbitrum, Gnosis safes and governance in general helped gather some new knowledge

### Time spent:
50 hours