## 1. DEFAULT VALUE ASSIGNMENT TO VARIABLES CAN BE OMITTED TO SAVE GAS

In the `SecurityCouncilManager` contract there multiple occasions inside the `for` loop where the default value of `0` assigned to the `i` variable. Since `i` is by default initialzied to `0` it is not required explicitly assign the `0` value to `i` variable inside the `for` loops. This will save gas.

        for (uint256 i = 0; i < _newCohort.length; i++) {
            _addMemberToCohortArray(_newCohort[i], _cohort);
        }

Above can be modified as follows:

        for (uint256 i; i < _newCohort.length; i++) {
            _addMemberToCohortArray(_newCohort[i], _cohort);
        }

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L135-L137
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L118-L120
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L164
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L162
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L392

## 2. VARIABLES CAN BE DECLARED OUTSIDE THE `for` LOOPS TO SAVE GAS

In the `SecurityCouncilManager` contract, there are multiple occassions where the local variables are declared inside the `for` loops. But it is recommedded to declare these variables outside the `for` loops and use them inside later.

        for (uint256 i = 0; i < _securityCouncils.length; i++) {
            _addSecurityCouncil(_securityCouncils[i]);
        }

Above `for` loop can be modified as follows:

        uint256 i;
        for ( i; i < _securityCouncils.length; i++) {
            _addSecurityCouncil(_securityCouncils[i]);
        }

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L118-L120

## 3.  THE ENTIRE `for` LOOP FOR SECURITY COUNCIL EXISTENCE CHECK CAN BE REPLACED WITH A SINGLE TWO DIMENSIONAL MAPPING

The `SecurityCouncilManager._addSecurityCouncil` function is used to add a new `SecurityCouncilData` element to the `securityCouncils` array. Before adding the `SecurityCouncilData` element there is a check performed to verify that `new SecurityCouncil` does no already exist in the `securityCouncils` array as show below:

        for (uint256 i = 0; i < securityCouncils.length; i++) { //@audit-issue - this entire for loop can be replaced with a two dimensional mapping(uint256 => mapping(address => bool))
            SecurityCouncilData storage existantSecurityCouncil = securityCouncils[i];
            if (
                existantSecurityCouncil.chainId == _securityCouncilData.chainId
                    && existantSecurityCouncil.securityCouncil == _securityCouncilData.securityCouncil
            ) {
                revert SecurityCouncilAlreadyInRouter(_securityCouncilData);
            }
        }

As it is seen above this check requires to iterate through an entire `for` loop for `securityCouncils.length` number of iteration. Hence this operation is extremly gas consuming. Further there are multiple `SLOAD` operations happening inside a single iteration as well.

Hence it is recommended to replace the entire for loop with a two dimensional `mapping(uint256 => mapping(address => bool))`. Here for each new `SecurityCouncilData` element added to the `securityCouncils` array the `chainId` and `securityCouncil` will be stored in the `mapping` with boolean value of `true`. 

So when adding a new `SecurityCouncilData` element later, this mapping can be checked for the boolean value. If the boolean value is `true`, it means the `new SecurityCouncilData` element is already in the `secuirtyCouncils` array and hence the transaction should revert.

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L251-L260

## 4. `storage` VARIABLE DECLARATIONS CAN BE REPLACED WITH `memory` variables.

In the `SecurityCouncilManager._addSecurityCouncil` the `existantSecurityCouncil` variable is declared as a `storage` variable. But this variable is used only to read from and used to modify and value. Hence the `existantSecurityCouncil` variable can be declared as a `memory` variable to save gas on multiple `SLOAD` and `SSTORE` operations.

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L255-L256

## 5. RECOMMENDED TO CACHE THE STORAGE ARRAY LENGTH AND USE THE VALUE FROM MEMORY

The `SecurityCouncilManager.getScheduleUpdateInnerData` function uses the `securityCouncils.length` value multiple times within the function scope. This is multiple calls to the `SLOAD` operation. Hence it is recommended to cache the `securityCouncils.length` value into memory variable and use it from memory for the operations inside the `getScheduleUpdateInnerData` function.

This will save gas on multiple `SLOAD` operations.

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L388-L413

## 6. EXEUCTION FLOW OF THE `SecurityCouncilMemberSyncAction.perform` FUNCTION SHOULD BE CHANGED TO SAVE GAS

The `SecurityCouncilMemberSyncAction.perform` function is used to update members of security council multisig to match the provided array. There are two `for` loops in the function to modify the members of the security council.

        for (uint256 i = 0; i < _updatedMembers.length; i++) {
            address member = _updatedMembers[i];
            if (!securityCouncil.isOwner(member)) {
                _addMember(securityCouncil, member, threshold);
            }
        }

As shown above the first `for` loop is used to add the new members which are non-existent in the current owner array to the security council.

        for (uint256 i = 0; i < previousOwners.length; i++) {
            address owner = previousOwners[i];
            if (!SecurityCouncilMgmtUtils.isInArray(owner, _updatedMembers)) {
                _removeMember(securityCouncil, owner, threshold);
            }
        }

The second `for` loop as shown above is used to remove the current `owner` in the security array who are not in the updated members list.

The `_removeMember` function call of the second `for` loop calls the `getPrevOwner` function, which gets the current list of the owners of the security council as shown below:

        address[] memory owners = securityCouncil.getOwners(); 

But this `owners` list includes the newly added members through the first `for` loop. Hence the `owner.length` is higher which consumes more gas. Hence the execution flow of the two `for` loops should interchange. Which would remove the existing owners who are not in the updated members list first. This will make the `owner.lenght` comparatively smaller thus saving gas since less number of iterations to execute.

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L60-L65
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L67-L72
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L103-L111

## 7. FOR READ ONLY OPERATIONS IT IS RECOMMENDED TO USE `memory` VARIABLE INSTEAD OF `storage` VARIABLE

In the `SecurityCouncilMemberElectionGovernorCountingUpgradeable.topNominees` function the respective `election` struct from the `_elections` mapping is retrieved as follows:

        ElectionInfo storage election = _elections[proposalId]; 


Even though the `election` is a storage variable it is only readfrom within the `topNominees` function and not written into. Hence a `memory variable` can be used in place of the `storage variable` to save gas as shown below:

        ElectionInfo memory election = _elections[proposalId]; 

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L180