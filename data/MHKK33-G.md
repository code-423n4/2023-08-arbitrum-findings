CACHED VARIABLE MUST BE USED FOR LOW GAS USAGE

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L98C10-L98C10

SUMMARY:
    In the given code, 'if' statement reads the value 'votes' directly from global storage which cost gas, the value of 'votes' must be read from variable 'actualVotes' for low gas usage.

ACTUAL CODE:

       uint256 actualVotes = votes;
        if (prevVotesReceived + votes >= votesThreshold) {
            // we pushed the contender over the line
            // we should only give the contender enough votes to get to the line so that we don't waste votes
            actualVotes = votesThreshold - prevVotesReceived;

            // push the contender to the nominees
            _addNominee(proposalId, contender);
        }

MODIFIED CODE:

      uint256 actualVotes = votes;
        if (prevVotesReceived + actualVotes >= votesThreshold) {
            // we pushed the contender over the line
            // we should only give the contender enough votes to get to the line so that we don't waste votes
            actualVotes = votesThreshold - prevVotesReceived;

            // push the contender to the nominees
            _addNominee(proposalId, contender);
        }
