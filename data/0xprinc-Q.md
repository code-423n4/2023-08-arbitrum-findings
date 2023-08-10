## 1. Wrong implememtation of the code of `SecurityCouncilMgmtUpgradeLib.sol/areAddressArraysEqual(...)` as it is missing an edge case
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L52

This function checks whether the two addresses array have all same elements or not. <br>This function contains two loops, first loop check whether all elements of first array are present in the second array, second for loop does vice versa. <br>
This is wrong implementation as the input of type `([a,a,b,c], [a,b,b,c])` will bypass where a,b,c are three different addresses. <br>
This is considered `low` as the function is used with arrays with all addresses in the array to be different and no repetition. <br>
But if that is the case then there is no point of having two for loops as if the arrays have all elements different then this will not require two for loops.

## 2. Minor typo in `ActionExecutionRecod.sol`
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/execution-record/ActionExecutionRecord.sol#L8-L9
`it` appears two times
```Solidity
/// @dev    This contract is designed to be inherited by action contracts, so it
///         it must not use any local storage
```

## 3. No input validation in `ActionExecutionRecord.sol/constructor(...)`
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/execution-record/ActionExecutionRecord.sol#L19

there should be the input validation of `isContract` to validate the `KeyValueStore` contract.

## 4. Better not to use same name for the two functions 
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/execution-record/ActionExecutionRecord.sol#L37-L39
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/execution-record/KeyValueStore.sol#L26-L28
`ActionExecutionRecord.sol/computeKey()` and `KeyValueStore.sol/computeKey()` have the same function signatures which is not recommended.

## 5. Redundant public constant present in `SecurityCouncilManager.sol`
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L77

constant named as the `RETRYABLE_TICKET_MAGIC` present in the contract is redundant as it is not called anytime in that contract and is not even used by any other contract calling this contract i.e. `SecurityCouncilManager.sol`.
`RETRYABLE_TICKET_MAGIC` is also declared in `L1ArbitrumTimelock.sol` and `UpgradeExecRouteBuilder.sol` where it is used but not here.<BR>

## 6. `SecurityCouncilManager.sol/replaceCohort(...)` should been recommended to have the functionality to replace the new cohort of any size in the limits of (0 -> cohortSize)
This functionality will be useful while making a big change in the security council with change in the new cohort length, all this in a single transaction. otherwise it will need more than a single transaction.

## 7. `SecurityCouncilManager.sol/_swapMembers(...)` should include the validation that the address to add should not equal to the address to remove.
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L218-L229

This validation is recommended to avoid not necessary swapping.

## 8. Visibility of `SecurityCouncilMgmtUtils.sol/filterAddressesWithExcludeList(...)` can be marked as pure as it is not reading any state.
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol#L15

## 9. No need to daclare the return variable in `SecurityCouncilMemberSyncAction.sol/perform(...)` as the name of the variable is never used 
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L31-L35

`res` variable name is never used so better not to name the retuern variable
```Solidity
    function perform(address _securityCouncil, address[] memory _updatedMembers, uint256 _nonce)
        external                                                                                  
        returns (bool res)  
```

## 10. `zeroAddress` validation missing in function `perform(...)`
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L60-L65


```Solidity
for (uint256 i = 0; i < _updatedMembers.length; i++) {
            address member = _updatedMembers[i];                    // @low should also have a check of that the new addresses are not zero addresses
            if (!securityCouncil.isOwner(member)) {
                _addMember(securityCouncil, member, threshold);
            }
        }
```