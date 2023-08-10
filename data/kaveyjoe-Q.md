## 1 . Target : https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol



- Description: 
The function revokeRole  does not check to make sure that the role to be revoked is actually held by the specified address. This could allow an attacker to revoke a role from an address that does not actually have it, which could have unintended consequences.

- Steps to reproduce:
            1 . Create a new instance of the GovernanceChainSCMgmtActivationAction contract.
            2 . Pass in a value for the securityCouncilManager property that is not a contract address.
            3 . Call the revokeRole function, specifying the TIMELOCK_PROPOSAL_ROLE for the role to be revoked 
                and the securityCouncilManager address as the address to revoke the role from.

Expected behavior:
 The revokeRole function should fail, and throw an error indicating that the specified address does not have the role to be revoked.
Actual behavior:
 The revokeRole function does not fail, and the role is revoked from the specified address, even though the address does not actually have the role.

- Recommended fix:

 The revokeRole function should be updated to check to make sure that the specified address actually has the role to be revoked. This can be done by adding the following code to the function:
require(
    l2CoreGovTimelock.hasRole(role, address),
    "Cannot revoke role from address that does not have it"
);
This code will check to make sure that the specified address actually has the role to be revoked. If the address does not have the role, the revokeRole function will fail, and throw an error.