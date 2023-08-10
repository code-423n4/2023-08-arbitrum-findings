# Since some contract will be deployed on Arbitrum, so I think it's better to distinguish between L1 and L2 when get `block.number`
File:
```text
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L151-L153
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L333-L335
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/ArbitrumGovernorProposalExpirationUpgradeable.sol#L36
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L115-L118
```
I think it better to use a distinguished method to get `block.number`, for example:
```solidity
    function currentBlockNumber() internal view returns (uint256) {
        if (block.chainid == ARBITRUM_CHAIN_ID) {
            return arbSys.arbBlockNumber();
        }

        return block.number;
    }
```

