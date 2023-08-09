# L-01: L1SCMgmtActivationAction.perform() lacks confirming updates for EXECUTOR_ROLE

Both [NonGovernanceChainSCMgmtActivationAction.perform()](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/NonGovernanceChainSCMgmtActivationAction.sol#L35-L43) and [GovernanceChainSCMgmtActivationAction.perform()](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol#L114-L122) do confirm updates for EXECUTOR_ROLE. However, L1SCMgmtActivationAction.perform() doesn't confirm such updates.

## Recommended Mitigation Steps

```
File: governance\src\gov-action-contracts\AIPs\SecurityCouncilMgmt\L1SCMgmtActivationAction.sol
30:     function perform() external {
......
61:         );
+++	    bytes32 EXECUTOR_ROLE = l1UpgradeExecutor.EXECUTOR_ROLE();
+++	    require(
+++	        upgradeExecutor.hasRole(EXECUTOR_ROLE, address(newEmergencySecurityCouncil)),
+++	        "L1SCMgmtActivationAction: new emergency security council not set"
+++	    );
+++	    require(
+++	        !upgradeExecutor.hasRole(EXECUTOR_ROLE, address(prevEmergencySecurityCouncil)),
+++	        "L1SCMgmtActivationAction: prev emergency security council still set"
+++	    );
62:     }
```
# L-02: The relay function in multiple contracts lacks the payable keyword

```solidity
function relay(address target, uint256 value, bytes calldata data)
        external
        virtual
        override
        onlyOwner
    {
        AddressUpgradeable.functionCallWithValue(target, data, value);
    }
```

The `relay` function internally uses `AddressUpgradeable.functionCallWithValue` to make call. If the `value` argument is greater than 0, tx will revert due to having not enough native token. These contracts does not implement `receive() payable` or `fallback() payable`, so it is unable to receive native token.

These `relay` functions are located at [[1](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L254)]/[[2](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L203)]/[[3](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L103)].

## Recommended Mitigation Steps

Add `payable` keyword for `relay` function.

# N-01: In GovernanceChainSCMgmtActivationAction.sol/L1SCMgmtActivationAction.sol, multiple error messages are misspelled

In L1SCMgmtActivationAction.sol:

```fix
File: governance\src\gov-action-contracts\AIPs\SecurityCouncilMgmt\L1SCMgmtActivationAction.sol
30:     function perform() external {
......
41:         require(
42:             l1Timelock.hasRole(TIMELOCK_CANCELLER_ROLE, address(prevEmergencySecurityCouncil)),
43:---          "GovernanceChainSCMgmtActivationAction: prev emergency security council should have cancellor role"
43:+++          "L1SCMgmtActivationAction: prev emergency security council should have cancellor role"
44:         );
45:         require(
46:             !l1Timelock.hasRole(TIMELOCK_CANCELLER_ROLE, address(l1UpgradeExecutor)),
47:---          "GovernanceChainSCMgmtActivationAction: l1UpgradeExecutor already has cancellor role"
47:+++          "L1SCMgmtActivationAction: l1UpgradeExecutor already has cancellor role"
48:         );
......
54:         require(
55:             l1Timelock.hasRole(TIMELOCK_CANCELLER_ROLE, address(l1UpgradeExecutor)),
56:---          "GovernanceChainSCMgmtActivationAction: l1UpgradeExecutor canceller role not set"
56:+++          "L1SCMgmtActivationAction: l1UpgradeExecutor canceller role not set"
57:         );
58:         require(
59:             !l1Timelock.hasRole(TIMELOCK_CANCELLER_ROLE, address(prevEmergencySecurityCouncil)),
60:---          "GovernanceChainSCMgmtActivationAction: prevEmergencySecurityCouncil canceller role not revoked"
60:+++          "L1SCMgmtActivationAction: prevEmergencySecurityCouncil canceller role not revoked"
61:         );
62:     }
```

In GovernanceChainSCMgmtActivationAction.sol:

```fix
File: governance\src\gov-action-contracts\AIPs\SecurityCouncilMgmt\GovernanceChainSCMgmtActivationAction.sol
114:         bytes32 EXECUTOR_ROLE = upgradeExecutor.EXECUTOR_ROLE();
115:         require(
116:             upgradeExecutor.hasRole(EXECUTOR_ROLE, address(newEmergencySecurityCouncil)),
117:---          "NonGovernanceChainSCMgmtActivationAction: new emergency security council not set"
117:+++          "GovernanceChainSCMgmtActivationAction: new emergency security council not set"
118:         );
119:         require(
120:             !upgradeExecutor.hasRole(EXECUTOR_ROLE, address(prevEmergencySecurityCouncil)),
121:---          "NonGovernanceChainSCMgmtActivationAction: prev emergency security council still set"
121:+++          "GovernanceChainSCMgmtActivationAction: prev emergency security council still set"
122:         );
```