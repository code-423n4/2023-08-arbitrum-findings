### Approach taken in evaluating the codebase

- Focused on Activation of elections and the management upgrade lib
- Look through the council Manager contracts

### Mechanism review

L1SCMgmtActivationAction 

- L1SCMgmtActivationAction does not use emergency and non emergency security council unlike L2. In GovernanceChainSCMgmtActivationAction.sol (L2 version), there is a difference between newEmergencySecurityCouncil and newNonEmergencySecurityCouncil. Not sure about the reason between splitting emergency and non-emergency in L2.
- L1SCMgmtActivationAction does not check the EXECUTOR_ROLE unlike the L2 version, so if the lack of check is unintended then it would be a medium problem. Otherwise, the protocol can save gas by not checking the EXECUTOR_ROLE in GovernanceChainSCMgmtActivationAction.sol since the role is already checked in the function `replaceEmergencySecurityCouncil()` in SecurityCouncilMgmtUpgradeLib

SecurityCouncilMgmtUpgradeLib
- Unsure about using `requireSafesEquivalent()` and checking that the addresses of the prevNonEmergencySecurityCouncil() and newNonEmergencySecurityCouncil() is the same. This defeats the purpose of calling it new and old if both arrays must hold the same address due to the `areAddressArrayEqual()` check. 
- If arrays of safe1 and safe2 is too large, may result it out of gas error because of the nested loop when checking.


```
SecurityCouncilMgmtUpgradeLib.sol
        address[] memory prevOwners = _safe1.getOwners();
        address[] memory newOwners = safe2.getOwners();
```

### Codebase quality analysis

Some minor details:

1. safe1 in SecurityCouncilMgmtUpgradeLib.sol uses an underscore whereas safe2 does not use an underscore. Not sure about the significance, or maybe is just to create a obvious separation between safe1 and safe2. Usually, underscores are used to highlight a variable or indicate an internal function. 

```
    function requireSafesEquivalent(
        IGnosisSafe _safe1,
        IGnosisSafe safe2,
```

2. Typo in NonGovernanceChainSCMgmtActivationAction.sol, should be emergency instead of memgency.

```
        // swap in new memgency security council
```

### Systemic risks

- The bridge from Arbitrum to mainnet may not work 100% of the time, which poses as a systemic risk for the system. 
- The use of isContract() is not a reliable way to check if the address is a contract in SecurityCouncilNomineeElectionGovernor.sol. Using assembly to check extcodesize would be safer.

### Time spent:
10 hours