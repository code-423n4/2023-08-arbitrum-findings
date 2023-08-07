## The `rotateMember` function is annotated to be initiated by the security council

### Description

The `rotateMember` function inherits documentation from its interface, which describes the function as follows:

> /// @notice Security council member can rotate out their address for a new one; _currentAddress and _newAddress should be of the same identity. Functionality is equivalent to replaceMember, tho emits a different event to distinguish the security council's intent (same identity).
    ///         Rotation must be initiated by the security council.

However, there's no check in the function enforcing that a rotation must be initated by the council's member, which is not enforced.

### Impact

An integrating smart contract may assume this function is only callable by the security council when it conflicts with its `onlyRole(MEMBER_ROTATOR_ROLE)` modifier.

### Recommendation

Consider adding an amend note in the implementation or removing such line from the interface.

## Both cohorts can share members after initialization

### Summary

The `initialize` function doesn't check that both cohorts don't share a member

### Description

Currently, the `initialize` function receives the initial cohort members for both `_firstCohort` and `_secondCohort`. Both hold the invariant that no user can be in both cohorts at the same time which is a guarantee used by multiple functions (e.g. the `getBothCohorts` functions).

However, although this invariant is checked in the `_addMemberToCohortArray`, it's not checked during initialization.

### Impact

A misconfigured initialization may render a single member in both arrays, which can cause unexpected issues with any contract relying on this invariant.

## An `address(0)` check is not included in `_removeMemberFromCohortArray`

### Summary

The`_removeMemberFromCohortArray` function doesn't include an `address(0)` check.

### Description

The current implementation of the `SecurityCouncilManager` includes multiple checks for the `address(0)` across multiple methods, including cohort member additions and removals. These checks are applied inconsistently across the contract, leading to other issues such as redundant checks or lack of these.

## Recomendation

For consistency with other internal methods and favoring a consistent single check in the contract, considering applying the following diff:

```diff
diff --git a/src/security-council-mgmt/SecurityCouncilManager.sol b/src/security-council-mgmt/SecurityCouncilManager.sol
index 8ec1308..206a8a0 100644
--- a/src/security-council-mgmt/SecurityCouncilManager.sol
+++ b/src/security-council-mgmt/SecurityCouncilManager.sol
@@ -159,6 +159,9 @@ contract SecurityCouncilManager is
     }
 
     function _removeMemberFromCohortArray(address _member) internal returns (Cohort) {
+        if (_member == address(0)) {
+            revert ZeroAddress();
+        }
         for (uint256 i = 0; i < 2; i++) {
             address[] storage cohort = i == 0 ? firstCohort : secondCohort;
             for (uint256 j = 0; j < cohort.length; j++) {
@@ -181,9 +184,6 @@ contract SecurityCouncilManager is
 
     /// @inheritdoc ISecurityCouncilManager
     function removeMember(address _member) external onlyRole(MEMBER_REMOVER_ROLE) {
-        if (_member == address(0)) {
-            revert ZeroAddress();
-        }
         Cohort cohort = _removeMemberFromCohortArray(_member);
         _scheduleUpdate();
         emit MemberRemoved({member: _member, cohort: cohort});
@@ -219,9 +219,6 @@ contract SecurityCouncilManager is
         internal
         returns (Cohort)
     {
-        if (_addressToRemove == address(0) || _addressToAdd == address(0)) {
-            revert ZeroAddress();
-        }
         Cohort cohort = _removeMemberFromCohortArray(_addressToRemove);
         _addMemberToCohortArray(_addressToAdd, cohort);
         _scheduleUpdate();
```

Also, consider even removing the `address(0)` check in the `_removeMemberFromCohortArray` function if it's not possible to add the `address(0)` either.

### `SecurityCouncilManager` cohort member operations can be bricked (partially or completely) at initialization

## Summary

The lack of checks for cohort members in the `initialize` function may cause the contract to not be able to operate cohorts again or to lock the `address(0)` in the cohort members array.

## Description

The initializing function doesn't perform all of the required invariant checks for adding new cohort members, allowing a misconfigured initialization to bypass the following guarantees:

- A single member can't be added to both cohorts at the same time.
- The `address(0)` can't be part of a cohort.

This scenario although low likely, can come from a configuration bug affecting a deployment setup leaving only the manager contract unusable.

## Impact

Any member added twice to a cohort may lead to a Timelock update that includes a single address twice. Similarly, if the `address(0)` is added accidentally, it can't be removed thereafter because of the `address(0)` checks included in operations removing it.

## Recommendation

Consider adding the mentioned checks in the `initialize` function.