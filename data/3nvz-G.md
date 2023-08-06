# Safe Gas on for-loop in SecurityCouncilManager.sol
## Issue was found in line 284 in the SecurityCouncilManager.sol contract
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L284

## The loop is living in the removeSecurityCouncil
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L279

## Here is my attempt to gas optimize the function
function removeSecurityCouncil(SecurityCouncilData memory _securityCouncilData)
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
    returns (bool)
{
    uint256 len = securityCouncils.length;

    for (uint256 i = 0; i < len; i++) {
        SecurityCouncilData storage securityCouncilData = securityCouncils[i];
        if (
            securityCouncilData.securityCouncil == _securityCouncilData.securityCouncil &&
            securityCouncilData.chainId == _securityCouncilData.chainId &&
            securityCouncilData.updateAction == _securityCouncilData.updateAction
        ) {
            SecurityCouncilData storage lastSecurityCouncil = securityCouncils[len - 1];

            securityCouncils[i] = lastSecurityCouncil;
            securityCouncils.pop();
            emit SecurityCouncilRemoved(
                securityCouncilData.securityCouncil,
                securityCouncilData.updateAction,
                len - 1 // Use len - 1 because we're about to remove an element
            );
            return true;
        }
    }
    revert SecurityCouncilNotInManager(_securityCouncilData);
}

## Here are the optimizations applied
- Single Length Access: Access securityCouncils.length once and store it (cache it) in a variable "len" to avoid accessing it repeatedly in the loop.

- Use len - 1: In the emit statement, use len - 1 as the new length value, since we're about to remove an element from the array. This prevents recalculating the length after the pop() operation.

##Final Note
Please provide some feedback how I can improve the report for the next time and if I have to write a seperate report for each issue I found or if it's fine if I put all the gas optimization issues for example into one report, all medium severity issues into another etc.
Thank you