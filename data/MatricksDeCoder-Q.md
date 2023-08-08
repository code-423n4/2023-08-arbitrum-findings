## NC-1 Unused Imports 

Some files have imports that are not used

[import "../UpgradeExecutor.sol" //SecurityCouncilManager.sol line 5](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L5)
Above import is a contract and there appears to be no contract calls, usage, logic associated with it, it has functions initialize and execute. Therefore it appears this imported contract is not used. 

[import "../../interfaces/ISecurityCouncilManager.sol"; //SecurityCouncilNomineeElectionGovernorTiming.sol line 4](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol#L4C37-L4C37)
Above Interface imported but never used 

[import "../../../interfaces/IArbitrumDAOConstitution.sol"; //GovernanceChainSCMgmtActivationAction.sol line 7 ](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol#L7)
Above interface imported but never used 

**Impact:** --> The above leads to bloated code, code readability, code auditability, code maintanability issues as readers may think it implies missing code of functionality.

**This doubles as a Gas Issue so will be reported in Gas Report too** Most importantly it adds to gas costs as it increases the size of the contract file as the unused import is extra bytecode for no reason which increases deployment costs for no reason 

**Recommendation:** --> It is recommended to remove all unused imports in above cases and any other relevant cases that may have been missed

