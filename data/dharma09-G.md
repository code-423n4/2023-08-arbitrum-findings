## **Gas Optimizations**

| Number | Issue | Instances |
| --- | --- | --- |
| [[G-01]](#) | Using storage instead of memory for structs/arrays saves gas | 1 |
| [[G-02]](#) | The result of a function call should be cached rather than re-calling the function | 1 |
| [[G-03]](#)  | Avoid emitting event on every iteration | 1 |
| [[G-04]](#) | Can Make The Variable Outside The Loop To Save Gas | 5 |
| [[G-05]](#) | Use calldata instead of memory for function parameters | 10 |
| [[G-06]](#)  | Compute hashing off-chain to save gas at deployment  | 5 |
| [[G-07]](#) | Assigning state variables to default value can be omitted to save gas | 1 |

### [G-01] Using `storage` instead of `memory` for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a `memory` variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (**2100 gas**) for *each* field of the struct/array. If the fields are read from the new memory variable, they incur an additional `MLOAD` rather than a cheap stack read. Instead of declearing the variable with the `memory` keyword, declaring the variable with the `storage` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a `memory` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires `memory`, or if the array/struct is being read from another `memory` array/struct

[UpgradeExecRouteBuilder.sol#L130](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L130)

```solidity
File: /UpgradeExecRouteBuilder.sol
130: UpExecLocation memory upExecLocation = upExecLocations[chainIds[i]];
```

### [G-02] The result of a function call should be cached rather than re-calling the function

In `SecurityCouncilMemberElectionGovernorCountingUpgradeable.so`there is function called `votingPeriod()` which called two time. so its better to cached rather than recalling the function

[SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L77C3-L84C6](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L77C3-L84C6)

```solidity
File: /security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol
77: function setFullWeightDuration(uint256 newFullWeightDuration) public onlyGovernance {
78:        if (newFullWeightDuration > votingPeriod()) { //1st call
79:            revert FullWeightDurationGreaterThanVotingPeriod(newFullWeightDuration, votingPeriod()); //2nd call
80:        }
81:
82:        fullWeightDuration = newFullWeightDuration;
83:        emit FullWeightDurationSet(newFullWeightDuration);
84:    }
```

### [G-03] Avoid emitting event on every iteration

Expensive operations should always try to be avoided within loops. Such operations include: reading/writing to storage, heavy calculations, external calls, and emitting events. In this instance, an event is being emitted every iteration. Events have a base cost of `Glog (375 gas)` per emit and `Glogdata (8 gas) * number of bytes in event`. We can avoid incurring those costs each iteration by emitting the event outside of the loop.

`Gas save 375`

[SecurityCouncilManager.sol#L296C16-L300C19](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L296C16-L300C19)

```solidity
File: /security-council-mgmt/SecurityCouncilManager.sol
296: emit SecurityCouncilRemoved(
297:                    securityCouncilData.securityCouncil,
298:                    securityCouncilData.updateAction,
299:                    securityCouncils.length
300:                );
```

### [G-04] Can Make The Variable Outside The Loop To Save Gas

When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. Here's an example:

```solidity
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        uint256 total = 0;
        
        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }
        
        return total;
    }
}
```

[SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L206](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L206)

```solidity
File: /governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol
206: uint256 packed = (uint256(weights[i]) << 16) | i;
```

[SecurityCouncilMemberSyncAction.sol#L61](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L61)

```solidity
File: /security-council-mgmt/SecurityCouncilMemberSyncAction.sol
61: address member = _updatedMembers[i];
68: address owner = previousOwners[i];
106: address currentOwner = owners[i];
```

[security-council-SecurityCouncilMgmtUtils.sol#L23C12-L23C40](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol#L23C12-L23C40)

```solidity
File: /security-council-mgmt/SecurityCouncilMgmtUtils.sol
23: address nominee = input[i];
```

### [G-05] **Use calldata instead of memory for function parameters**

[SecurityCouncilManager.sol#L89C5-L96C29](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L89C5-L96C29)

```solidity
File: /security-council-mgmt/SecurityCouncilManager.sol
89: function initialize(
90:        address[] memory _firstCohort,
91:        address[] memory _secondCohort,
92:        SecurityCouncilData[] memory _securityCouncils,
93:        SecurityCouncilManagerRoles memory _roles,
94:        address payable _l2CoreGovTimelock,
95:        UpgradeExecRouteBuilder _router
96:    ) external initializer {
```

[SecurityCouncilMemberSyncAction.sol#L127](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L127)

```solidity
File: /security-council-mgmt/SecurityCouncilMemberSyncAction.sol
127: function _execFromModule(IGnosisSafe securityCouncil, bytes memory data) internal {
```

[SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L191C2-L195C6](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L191C2-L195C6)

```solidity
File: /security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol
191: function selectTopNominees(address[] memory nominees, uint240[] memory weights, uint256 k)
192:        public
193:        pure
194:        returns (address[] memory)
195:    {
```

[SecurityCouncilManager.sol#L124C4-L127C6](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L124C4-L127C6)

```solidity
File: /security-council-mgmt/SecurityCouncilManager.sol
124: function replaceCohort(address[] memory _newCohort, Cohort _cohort)
125:        external
126:        onlyRole(COHORT_REPLACER_ROLE)
127:    {

271: function addSecurityCouncil(SecurityCouncilData memory _securityCouncilData)
272:        external
273:        onlyRole(DEFAULT_ADMIN_ROLE)
274:    {

370: function generateSalt(address[] memory _members, uint256 nonce)
371:        external
372:        pure
373:        returns (bytes32)
374:    {
```

[SecurityCouncilNomineeElectionGovernor.sol#L324C4-L330C34](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L324C4-L330C34)

```solidity
File: /security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol
324: function _execute(
325:        uint256 proposalId,
326:        address[] memory, /* targets */
327:        uint256[] memory, /* values */
328:        bytes[] memory callDatas,
329:        bytes32 /*descriptionHash*/
330:    ) internal virtual override {

423: function propose(address[] memory, uint256[] memory, bytes[] memory, string memory)
424:        public
425:        virtual
426:        override
427:        returns (uint256)
428:    {
```

[SecurityCouncilMemberSyncAction.sol#L31C5-L34C6](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L31C5-L34C6)

```solidity
File: /security-council-mgmt/SecurityCouncilMemberSyncAction.sol
31: function perform(address _securityCouncil, address[] memory _updatedMembers, uint256 _nonce)
32:        external
33:        returns (bool res)
34:    {
```

[SecurityCouncilMgmtFactory.sol#L81C5-L86C1](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol#L81C5-L86C1)

```solidity
File: /security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol
81: function deploy(DeployParams memory dp, ContractImplementations memory impls)
82:        external
83:        onlyOwner
84:        returns (DeployedContracts memory)
85:    {
```

### [G-06] Compute hashing off-chain to save gas at deployment

we can compute these values off-chain to save gas at deployment as hashing is expensive 

[SecurityCouncilManager.sol#](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L79C3-L83C79)L79-L83

for example:

```solidity
File: /security-council-mgmt/SecurityCouncilManager.sol
79:    bytes32 public constant COHORT_REPLACER_ROLE = 0x9605363ac419df002bafe0cc1dcbfb19cf2a2e2afa1c7812ee5dec3c3b19090625d;
80:    bytes32 public constant MEMBER_ADDER_ROLE = 0xdf8520ffe197c5343c6f5aec59570151ef9a492f2b2d3f6c624fd45ddde6135ec42;
81:    bytes32 public constant MEMBER_REPLACER_ROLE = 0xa08df8e9779f89161e9d4aa6eaffa8e1f95bfbc9b9812ee5dec3c3b19090625d;
82:    bytes32 public constant MEMBER_ROTATOR_ROLE = 0x9605363ac419d89161e9d4aa6eaffa8e1f95bfbc9b9812ee5dec3c3b19090625d;
83:    bytes32 public constant MEMBER_REMOVER_ROLE = 0xa08df8e977c6f5aec59570151ef9a4bafe0cc192f2c624fd45ddde6135ec42;
```

### [G-07] Assigning state variables to default value can be omitted to save gas

In the `UpgradeExecRouteBuilder.sol` contract the `DEFAULT_VALUE` state variable is assigned to the default value of `0` as shown below:

```solidity
File: /src/UpgradeExecRouteBuilder.sol
52: uint256 public constant DEFAULT_VALUE = 0;
```

This can be omitted to save gas, since by default  variable will have the value `0`. This will save an extra `SSTORE` operation.