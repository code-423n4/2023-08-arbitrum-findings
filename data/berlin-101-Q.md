# L-01 SecurityCouncilManager.sol "_addSecurityCouncil" only checks for chain id and security council address

When adding a security council only chain id and security council address are checked to detect whether adding the council is allowed: https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L255-L256

When removing a security council also the address of the update action is checked to make the match: https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L255-L256.

This can mean 2 things. 1. When removing a security council the check for the update action address is not necessary and can be removed. 2. Not checking for the address of the update action when adding a security council prevents adding more granular associations between a security council and upgrade actions.

