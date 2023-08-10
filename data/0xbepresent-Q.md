1 - Contender and Nominee can vote to themselves
==

In the [SecurityCouncilNomineeElectionGovernorCountingUpgradeable:_countVote()](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L62) there is not any validation that the contenders can vote to themselves.

Additionally the nominees can vote to themselves since there is not any restriction in the [SecurityCouncilMemberElectionGovernorCountingUpgradeable._countVote()](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L95) function.

Add a restriction that contender or nominee can not vote to themselves.

2 - Validates only the votes number that will be used
==

The [SecurityCouncilNomineeElectionGovernorCountingUpgradeable::_countVote()](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L62C14-L62C24) allows the users to count the votes for multiple `contenders` in order to nominee them.

The next code block helps to calculate the `actualVotes` variable which is the exact number that helps the contenteder to be a `nominee`, therefore the voter's votes are not wasted and it is only used the necessary amount:

```solidity
File: SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol
093: 
094:         uint256 prevVotesReceived = election.votesReceived[contender];
095:         uint256 votesThreshold = quorum(proposalSnapshot(proposalId));
096: 
097:         uint256 actualVotes = votes;
098:         if (prevVotesReceived + votes >= votesThreshold) {
099:             // we pushed the contender over the line
100:             // we should only give the contender enough votes to get to the line so that we don't waste votes
101:             actualVotes = votesThreshold - prevVotesReceived;
102: 
103:             // push the contender to the nominees
104:             _addNominee(proposalId, contender);
105:         }
106: 
107:         election.votesUsed[account] = prevVotesUsed + actualVotes;
108:         election.votesReceived[contender] = prevVotesReceived + actualVotes;
```

The problem is in the next code block helps to ensure the voter has enough avaliabale vote weight. Since not all the voter's votes will be used, it is not necessary to validates all the `votes` number:

```solidity
File: SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol
87:         NomineeElectionCountingInfo storage election = _elections[proposalId];
88:         uint256 prevVotesUsed = election.votesUsed[account];
89: 
90:         if (votes + prevVotesUsed > weight) {
91:             revert InsufficientTokens(votes, prevVotesUsed, weight);
92:         }
```

Consider validates only the votes that will be necessary to cover the contender threshold.

```diff
--      if (votes + prevVotesUsed > weight) {
++      if (actualVotes + prevVotesUsed > weight) {
            revert InsufficientTokens(votes, prevVotesUsed, weight);
        }
```

3 - The `relay()` functions don't have the `payable` modifier
==

The [SecurityCouncilMemberElectionGovernor.relay()](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L103) and [SecurityCouncilNomineeElectionGovernor.relay()](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L254) functions don't have the `payable` modifier.

The requirements of the [AddressUpgradeable.functionCallWithValue()](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/f6401d9eaca362b90350a023473cb71964e1b9ef/contracts/utils/AddressUpgradeable.sol#L83C14-L83C35) are:

- the calling contract must have an ETH balance of at least `value`.
- the called Solidity function must be `payable`.

Consider adding the `payable` modifier in the `relay()` functions.