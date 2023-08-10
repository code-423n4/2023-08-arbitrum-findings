## [Low] Insufficient Member Verification during Initialization

It is possible to add same members to a council (also possible to add same address to two councils) twice or more times.
Also there is no check for array length (12 or not). 

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L97-L101

## [Low] Erroneous Data Logged in Events

The original intention of the `SecurityCouncilRemoved` event is to record the removed council. However, the data submitted here consists of references to `securityCouncils[i]`. 
This results in the remove event consistently recording the last element of the 'securityCouncils' array.

Consider emitting the `SecurityCouncilRemoved` event before the array undergoes shifting and popping.

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L296-L300

## [Info] Repetition in Comments

> // this only checks against ~~the current~~ the current other cohort, and against the current cohort membership

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L231

> // revoke old security council cancel role; it is unnecessary ~~to grant it~~ to explicitly grant it to new security council since the security council can already cancel via the core governor's relay method.

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol#L103