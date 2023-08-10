# QA
(by SolSaver)

## Typos

### Instance 1:

`counicl` should be `council`

```
File: src/gov-action-contracts/AIPs/SecurityCouncilMgmt/L1SCMgmtActivationAction.sol

39:        // swap in new emergency security counicl canceller role

```

Link: https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/L1SCMgmtActivationAction.sol#L39

### Instance 2:

`now` should be `low`

```
File: src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol

44:        // when nonce is too now, we simply return, we don't revert.

```

Link: https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L44

## Redundant checks

There are two loops below:
* Loop 1 (Line 61): Iterates over array 1, and then checks if the element exists in array 2.
* Loop 2 (Line 74): Iterates over array 2, and then checks if the element exists in array 1.

Both the loops are doing the same things, and only one of the loops is needed, as the other loop is just redundant.

```
File: src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol

61:     for (uint256 i = 0; i < array1.length; i++) {
            bool found = false;
            for (uint256 j = 0; j < array2.length; j++) {
                if (array1[i] == array2[j]) {
                    found = true;
                    break;
                }
            }
            if (!found) {
                return false;
            }
        }

74:     for (uint256 i = 0; i < array2.length; i++) {
            bool found = false;
            for (uint256 j = 0; j < array1.length; j++) {
                if (array2[i] == array1[j]) {
                    found = true;
                    break;
                }
            }
            if (!found) {
                return false;
            }
        }

```

Link: https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L74-L85

## Use `SecurityCouncilMgmtUtils.isInArray()` for consistency

In the instances below, `SecurityCouncilMgmtUtils.isInArray()` can be used, instead of writing the same logic every time. Rewriting the logic increases the chances of bugs.

### Instance 1:

```
File: src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol

        for (uint256 i = 0; i < array1.length; i++) {
            bool found = false;
63:         for (uint256 j = 0; j < array2.length; j++) {
                if (array1[i] == array2[j]) {
                    found = true;
                    break;
                }
            }
            if (!found) {
                return false;
            }
        }
```

Link: https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L63-L68

### Instance 1 Suggested Fix:

```
for (uint256 i = 0; i < array1.length; i++) {
+   bool found = isInArray(array1[i], array2);
-   bool found = false;
-   for (uint256 j = 0; j < array2.length; j++) {
-       if (array1[i] == array2[j]) {
-           found = true;
-           break;
-       }
-   }
    if (!found) {
        return false;
    }
}
```

### Instance 2:

```
File: src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol

        for (uint256 i = 0; i < array2.length; i++) {
            bool found = false;
76:         for (uint256 j = 0; j < array1.length; j++) {
                if (array2[i] == array1[j]) {
                    found = true;
                    break;
                }
            }
            if (!found) {
                return false;
            }
        }
```

Link: https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L76-L81

### Instance 2 Suggested Fix:

```
for (uint256 i = 0; i < array2.length; i++) {
+   bool found = isInArray(array2[i], array1);
-   for (uint256 j = 0; j < array1.length; j++) {
-       if (array2[i] == array1[j]) {
-           found = true;
-           break;
-       }
-   }
    if (!found) {
        return false;
    }
}
```

## Missing `updateAction` equality check

When a security council is added, the existing council list is iterated on to check if the council is already in the router. In this check, the `chainId` is matched, the `securityCouncil` is matched, but the `updateAction` is not.
In contrast, the `removeSecurityCouncil` actually performs the check before the security council is removed: `securityCouncilData.updateAction == _securityCouncilData.updateAction`

```
File: src/security-council-mgmt/SecurityCouncilManager.sol

254:        if (
                existantSecurityCouncil.chainId == _securityCouncilData.chainId
                    && existantSecurityCouncil.securityCouncil == _securityCouncilData.securityCouncil
            ) {
                revert SecurityCouncilAlreadyInRouter(_securityCouncilData);
            }
```

Link: https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L254-L259

### Suggested fix:

```
    if (
        existantSecurityCouncil.chainId == _securityCouncilData.chainId
            && existantSecurityCouncil.securityCouncil == _securityCouncilData.securityCouncil
+           && existantSecurityCouncil.updateAction == _securityCouncilData.updateAction
    ) {
        revert SecurityCouncilAlreadyInRouter(_securityCouncilData);
    }
```

## `SecurityCouncilNomineeElectionGovernorTiming.electionToTimestamp()` can underflow if not initialized properly

There are no checks in place to ensure that `firstNominationStartDate` is initialized. This may lead to underflows if the method is called prematurely.

```
File: src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol

77:     uint256 month = firstNominationStartDate.month - 1;
```

Link: https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol#L77