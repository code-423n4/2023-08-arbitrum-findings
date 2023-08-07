## Cache cohort's length storage reads in `getBothCohorts` function

### Summary

The `getBothCohorts` makes unnecessary storage reads when accessing array's length.

### Description

Currently, the `getBothCohorts` makes use of both `firstCohort` and `secondCohort` lengths in order to create a new memory array with the content of both. However, the function is defined as follows:

```solidity
function getBothCohorts() public view returns (address[] memory) {
    address[] memory members = new address[](firstCohort.length + secondCohort.length);
    for (uint256 i = 0; i < firstCohort.length; i++) {
        members[i] = firstCohort[i];
    }
    for (uint256 i = 0; i < secondCohort.length; i++) {
        members[firstCohort.length + i] = secondCohort[i];
    }
    return members;
}
```

The function reads both lengths while creating the new member's array and in every iteration through each cohort.

### Recommendation

Consider applying the following diff:

```diff
diff --git a/src/security-council-mgmt/SecurityCouncilManager.sol b/src/security-council-mgmt/SecurityCouncilManager.sol
index 8ec1308..4a32fc3 100644
--- a/src/security-council-mgmt/SecurityCouncilManager.sol
+++ b/src/security-council-mgmt/SecurityCouncilManager.sol
@@ -335,12 +335,14 @@ contract SecurityCouncilManager is
 
     /// @inheritdoc ISecurityCouncilManager
     function getBothCohorts() public view returns (address[] memory) {
-        address[] memory members = new address[](firstCohort.length + secondCohort.length);
-        for (uint256 i = 0; i < firstCohort.length; i++) {
+        uint256 firstCohortLength = firstCohort.length;
+        uint256 secondCohortLength = secondCohort.length;
+        address[] memory members = new address[](firstCohortLength + secondCohortLength);
+        for (uint256 i = 0; i < firstCohortLength; i++) {
             members[i] = firstCohort[i];
         }
-        for (uint256 i = 0; i < secondCohort.length; i++) {
-            members[firstCohort.length + i] = secondCohort[i];
+        for (uint256 i = 0; i < secondCohortLength; i++) {
+            members[firstCohortLength + i] = secondCohort[i];
         }
         return members;
     }
```

## Duplicated `_addressToAdd` check in `_swapMembers`

### Summary

The `_swapMembers` function makes an unnecessary `address(0)` check for `_addressToAdd`.

### Description

The `_swapMembers` removes a member from a cohort while adding another one. It does this by using the internal `_removeMemberFromCohortArray` and `_addMemberToCohortArray` functions. However, it checks that the zero address is neither added nor removed from the cohort, but this check is [already performed by the `_addAnotherMemberToCohortArray` function](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L144)

### Recommendation

Consider applying the following diff:

```diff
diff --git a/src/security-council-mgmt/SecurityCouncilManager.sol b/src/security-council-mgmt/SecurityCouncilManager.sol
index 8ec1308..7287627 100644
--- a/src/security-council-mgmt/SecurityCouncilManager.sol
+++ b/src/security-council-mgmt/SecurityCouncilManager.sol
@@ -219,7 +219,7 @@ contract SecurityCouncilManager is
         internal
         returns (Cohort)
     {
-        if (_addressToRemove == address(0) || _addressToAdd == address(0)) {
+        if (_addressToRemove == address(0)) {
             revert ZeroAddress();
         }
         Cohort cohort = _removeMemberFromCohortArray(_addressToRemove);
```

## Merge `rotateMember` and `replaceMember` regardless of their semantical difference.

### Summary

Make both functions just one by using a flag as a 3rd parameter to check access control and emit events.

### Description

Because both `rotateMember` and `replaceMember` behave the same but emit a different event and have a different Access Control mechanism it makes sense that both functions are separated. However, both the `MEMBER_REPLACER_ROLE` and `MEMBER_ROTATOR_ROLE` are given to the 9/12 multisig without any further intervention of the user that's rotating their address.

Since both roles are given to the same entity and the events can be selectively emitted, a flag to the `replaceMember` function can be added so that it's the only function exposed, reducing bytecode size.

### Recommendation

Apply the following diff to the SecurityCouncilManager contract:

```diff
diff --git a/src/security-council-mgmt/SecurityCouncilManager.sol b/src/security-council-mgmt/SecurityCouncilManager.sol
index 8ec1308..3ebeab1 100644
--- a/src/security-council-mgmt/SecurityCouncilManager.sol
+++ b/src/security-council-mgmt/SecurityCouncilManager.sol
@@ -190,29 +190,24 @@ contract SecurityCouncilManager is
     }
 
     /// @inheritdoc ISecurityCouncilManager
-    function replaceMember(address _memberToReplace, address _newMember)
-        external
-        onlyRole(MEMBER_REPLACER_ROLE)
-    {
+    function replaceMember(address _memberToReplace, address _newMember, bool rotation) external {
+        _checkRole(rotation ? MEMBER_ROTATOR_ROLE : MEMBER_REPLACER_ROLE, msg.sender);
+
         Cohort cohort = _swapMembers(_memberToReplace, _newMember);
-        emit MemberReplaced({
-            replacedMember: _memberToReplace,
-            newMember: _newMember,
-            cohort: cohort
-        });
-    }
 
-    /// @inheritdoc ISecurityCouncilManager
-    function rotateMember(address _currentAddress, address _newAddress)
-        external
-        onlyRole(MEMBER_ROTATOR_ROLE)
-    {
-        Cohort cohort = _swapMembers(_currentAddress, _newAddress);
-        emit MemberRotated({
-            replacedAddress: _currentAddress,
-            newAddress: _newAddress,
-            cohort: cohort
-        });
+        if(rotation) {
+            emit MemberRotated({
+                replacedAddress: _memberToReplace,
+                newAddress: _newMember,
+                cohort: cohort
+            });
+        } else {
+            emit MemberReplaced({
+                replacedMember: _memberToReplace,
+                newMember: _newMember,
+                cohort: cohort
+            });
+        }
     }
 
     function _swapMembers(address _addressToRemove, address _addressToAdd)
```