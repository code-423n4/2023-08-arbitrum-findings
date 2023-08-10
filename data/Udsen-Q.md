## 1. LACK OF INPUT VALIDATION FOR ADDRESSES WHEN ASSIGNING IN THE `initialize` FUNCTION OF THE `SecurityCouncilManager` CONTRACT

The `SecurityCouncilManager.initialize` function initializes the `firstCohort` and `secondCohort` arrays. But there is no input address validation performed inside the `initialize` function. Hence it is recommended to perform `address(0)` check for each of the `cohort member` addresses before the assignment to the state array variable.

Similarly the `SecurityCouncilNomineeElectionGovernor.setNomineeVetter` function, there is no check to verify that the `_nomineeVetter != address(0)`. Since the `nomineeVetter` is a critical role in this protocol it is recommeneded to add this check to the `setNomineeVetter` function.

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L100-L101
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L246-250

## 2. USE TWO-STEP OWNERSHIP TRANSFER

The `SecurityCouncilNomineeElectionGovernor` contract inherits from the `openzeppelin` `OwnableUpgradeable` contract to transfer the ownership to a particular user. But the ownership transfer process of the `OwnableUpgradeable` is single step and prone to `DoS` if `errorneous address` is set as the `owner`.

Hence it is recommended to use the `openzeppelin` `Ownable2StepUpgradeable`contract to use two step ownership transfer process for safe ownership transfers.

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L23
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L112

## 3. `includeNominee` FUNCTION DOES NOT IMPLEMENT THE `onlyVettingPeriod` MODIFIER

The `SecurityCouncilNomineeElectionGovernor.includeNominee` function is used to allow the nomineeVetter to explicitly include a nominee if there are fewer nominees than the target. The function has the `onlyNomineeVetter` modifier which limits the function to be called only by the `nomineeVetter`. But this function lacks the time limitation which is imposed via the `onlyVettingPeriod` modifier. Hence this function can be called by the `nomineeVetter` any time as long as the `state_ == ProposalState.Succeeded` condition is true. But the `nomineeVetter` modifier functionality is added to the `SecurityCouncilNomineeElectionGovernor.excludeNominee` function.

Hence it is recommended to add the `nomineeVetter` modifier to the `includeNominee` function as well to restrict the authority of the `nomineeVetter` and reduce the centralization risk.

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L290

## 4. DISCREPENCY IN THE `NATSPEC` COMMENTS AND FUNCTION IMPLEMENTATION

In the `SecurityCouncilMemberElectionGovernorCountingUpgradeable.votesToWeight` function the following `natspec` comment is given.

    ///         Each vote has weight 1 until the fullWeightVotingDeadline is reached, after which each vote has linearly
    ///         deacreasing weight, reaching 0 at the proposalDeadline.

It mentions that at the proposalDeadline the weight will reach 0.

But it is not correctly implemented in the logic implementation as shown below. The `weight` is returned as `0` only after the proposal deadline has passed and not at the proposal deadline.

        uint256 endBlock = proposalDeadline(proposalId);
        if (blockNumber > endBlock) {
            return 0;
        }

Above should be corrected as follows:

        uint256 endBlock = proposalDeadline(proposalId);
        if (blockNumber => endBlock) {
            return 0;
        }

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L223-L224
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L236-L239

## 5. THE `initialize` FUNCTIONS COULD BE FRONT RUN BY A MALICIOUS USER

The critical contracts of this system are upgradeable and hence have implemented the `initialize` function. For example the critical contracts such as `SecurityCouncilMemberElectionGovernor`, `SecurityCouncilNomineeElectionGovernor` uses the initialize functions.

But these initialize functions do not have any access control and hence can be front run by a malicious user thus prompting a redeployment of the contracts. 

Hence it is recommended to add access control to the `initialize` functions of these critical functions.

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L103-L131
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L48-L76

## 6. `isContract` COULD RETURN `false` WHEN WORKING WITH `CREATE2` TO DETERMINE A CONTRACT ADDRESS 

The `isContract` function of the `Openzeppeling` `Address.sol` library is used in multiple locations of this system to verify whether the passed in address to a function is indeed a contract. For example the following code snippet of the `SecurityCouncilManager._setUpgradeExecRouteBuilder` function shows how the `isContract` is used.

    function _setUpgradeExecRouteBuilder(UpgradeExecRouteBuilder _router) internal {
        address routerAddress = address(_router);

        if (!Address.isContract(routerAddress)) {
            revert NotAContract({account: routerAddress});
        }

        router = _router;
        emit UpgradeExecRouteBuilderSet(routerAddress);
    }

But the issue here is that if the `routerAddress` is created via the `CREATE2` and passed the address to this function the contract might not yet be deployed. Hence the `isContract` will revert since the `EXTCODESIZE` opcode will return `0`. But the contract to the `routerAddress` will be deployed at a later point in time.

Due to the transaction revert the users will be confused. The same issue could exist in other contracts such as `SecurityCouncilMemberRemovalGovernor`, `SecurityCouncilMemberElectionGovernor` etc ...

Hence it is recommended to inform the users of this protocol of the possibility of reverts when `CREATE2` is used to determine the contract addresses before the contract is deployed to the blockchain. This will enhance the user experience when interacting with the function which use the `isContract` functionality.

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L112-L114
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L318-L320

## 7. TYPO'S IN THE NATSPEC COMMENTS SHOULD BE CORRECTED

Following typos in the `natspec` comments should be corrected for ease of understanding and readability.

        // system will not be blocked if a retryable ticket is `expires`
		
Above comment should be corrected as follows:

        // system will not be blocked if a retryable ticket is `expired`

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L41

        // when nonce is too `now`, we simply return, we don't revert.
		
Above comment should be corrected as follows:

        // when nonce is too `low`, we simply return, we don't revert.

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol.sol#L44