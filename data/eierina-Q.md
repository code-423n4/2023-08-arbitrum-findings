# [NC-01] Error in SecurityCouncilMgmtUpgradeLib areAddressArraysEqual
## Impact
The [areAddressArraysEqual](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L52-L88) incorrectly flags different arrays as equals, as in the case of [A, B, A] and [A, B, B],

## Proof of Concept
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L61-L72

## Tools Used
n/a

## Recommended Mitigation Steps
Ensure no (or remove) dubplicates in the arrays 1 and 2.

# [NC-02] Code with no effects in SecurityCouncilMgmtUpgradeLib areAddressArraysEqual
## Impact
The [areAddressArraysEqual](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L52-L88) unnecessarily scan the arrays from both sides in two separates loops but it is redundant to do so because the two input arrays are required to have the same size.

## Proof of Concept
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L57-L59

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L74-L85

## Tools Used
n/a

## Recommended Mitigation Steps
Remove second loop.

# [NC-03] Inconsistent check of EXECUTOR_ROLE during replace of EmergencySecurityCouncil
## Impact
The SecurityCouncilMgmtUpgradeLib's [replaceEmergencySecurityCouncil](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L8) check EXECUTOR_ROLE on prev and new EmergencySecurityCouncil(s), then revoke from prev and add to new. Some contracts that use replaceEmergencySecurityCouncil do check whether the EXECUTOR_ROLE was swapped correctly, other do not. It would be correct to check the POST effects (swap correctly applied) inside the replaceEmergencySecurityCouncil function itself so to avoid repesting the check and in every place after the replaceEmergencySecurityCouncil function is called, or forgetting to do so.

## Proof of Concept
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L57-L59

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L74-L85

## Tools Used
n/a

## Recommended Mitigation Steps
Remove second loop.