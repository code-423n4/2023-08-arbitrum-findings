## The `rotateMember` function is annotated to be initiated by the security council

### Description

The `rotateMember` function inherits documentation from its interface, which describes the function as follows:

> /// @notice Security council member can rotate out their address for a new one; _currentAddress and _newAddress should be of the same identity. Functionality is equivalent to replaceMember, tho emits a different event to distinguish the security council's intent (same identity).
    ///         Rotation must be initiated by the security council.

However, there's no check in the function enforcing that a rotation must be initated by the council's member, which is not enforced.

### Impact

An integrating smart contract may assume this function is only callable by the security council when it conflicts with its `onlyRole(MEMBER_ROTATOR_ROLE)` modifier.

### Recommendation

Consider adding an amend note in the implementation or remove such line from the interface.

## Both cohorts can share members after initialization

### Summary

The `initialize` function doesn't check that both cohorts don't share a member

### Description

Currently, the `initialize` function receives the initial cohort members for both `_firstCohort` and `_secondCohort`. Both hold the invariant that no user can be in both cohorts at the same time which is a guarantee used by multiple functions (e.g. the `getBothCohorts` functions).

However, although this invariant is checked in the `_addMemberToCohortArray`, it's not checked during initialization.

### Impact

A misconfigured initialization may render both arrays different length, which can cause unexpected issues with any contract relying on this invariant.