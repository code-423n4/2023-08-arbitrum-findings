### [G-1] Unnecessary addresses array creation

The function `filterAddressesWithExcludeList` creates an unnecessary `output` array of addresses in memory costing extra gas and adding complexity.[src](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol#L15-L36)

#### Vulnerability details

```solidity
    function filterAddressesWithExcludeList(
        address[] memory input,
        mapping(address => bool) storage excludeList
    ) internal view returns (address[] memory) {
        address[] memory intermediate = new address[](input.length);
        uint256 intermediateLength = 0;

        for (uint256 i = 0; i < input.length; i++) {
            address nominee = input[i];
            if (!excludeList[nominee]) {
                intermediate[intermediateLength] = nominee;
                intermediateLength++;
            }
        }

        address[] memory output = new address[](intermediateLength);
        for (uint256 i = 0; i < intermediateLength; i++) {
            output[i] = intermediate[i];
        }

        return output;
    }
```

#### Recommendation

Remove either `intermediate` or `output`.

### [G-2] Duplicate checks in the `_countVote` function 

In the function above there is a check that is duplicated costing unnecessary gas and adding repetition.

#### Vulnerability details

[src](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L121-L127)

```solidity
        uint256 prevVotesUsed = election.votesUsed[account];
        if (prevVotesUsed + votes > availableVotes) {
            revert InsufficientVotes(prevVotesUsed, votes, availableVotes);
        }

        uint240 prevWeightReceived = election.weightReceived[nominee];
        election.votesUsed[account] = prevVotesUsed + votes;
```

#### Recommendation

Change to:

```solidity
        uint256 prevVotesUsed = election.votesUsed[account];
++      uint256 totalVotesUsed = prevVotesUsed + votes;
++      if (totalVotesUsed > availableVotes) {
--      if (prevVotesUsed + votes > availableVotes) {
            revert InsufficientVotes(prevVotesUsed, votes, availableVotes);
        }

        uint240 prevWeightReceived = election.weightReceived[nominee];
--      election.votesUsed[account] = prevVotesUsed + votes;
++      election.votesUsed[account] = totalVotesUsed;
```