## GAS Summary<a name="GAS Summary">

### Gas Optimizations
| |Issue|Contexts|Estimated Gas Saved|
|-|:-|:-|:-:|
| [GAS&#x2011;1](#GAS&#x2011;1) | `abi.encode()` is less efficient than `abi.encodepacked()` | 4 | 212 |
| [GAS&#x2011;2](#GAS&#x2011;2) | Use assembly in place of `abi.decode` to extract calldata values more efficiently | 2 | 224 |
| [GAS&#x2011;3](#GAS&#x2011;3) | Avoid emitting event on every iteration | 1 | 375 |
| [GAS&#x2011;4](#GAS&#x2011;4) | Counting down in `for` statements is more gas efficient | 28 | 7196 |
| [GAS&#x2011;5](#GAS&#x2011;5) | Multiple accesses of a mapping/array should use a local variable cache | 30 | 2400 |
| [GAS&#x2011;6](#GAS&#x2011;6) | The result of a function call should be cached rather than re-calling the function | 6 | 300 |
| [GAS&#x2011;7](#GAS&#x2011;7) | Use do while loops instead of for loops | 18 | 72 |
| [GAS&#x2011;8](#GAS&#x2011;8) | Use nested `if` and avoid multiple check combinations | 1 | 6 |
| [GAS&#x2011;9](#GAS&#x2011;9) | Using XOR (^) and AND (&) bitwise equivalents | 30 | 390 |

Total: 158 contexts over 9 issues

## Gas Optimizations

### <a href="#gas-summary">[GAS&#x2011;1]</a><a name="GAS&#x2011;1"> `abi.encode()` is less efficient than `abi.encodepacked()`

See for more information: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison 

#### <ins>Proof Of Concept</ins>


```solidity
File: UpgradeExecRouteBuilder.sol

151: schedData[i] = abi.encode(
```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L151

```solidity
File: ActionExecutionRecord.sol

38: return uint256(keccak256(abi.encode(actionContractId, key)));
```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/execution-record/ActionExecutionRecord.sol#L38

```solidity
File: KeyValueStore.sol

27: return uint256(keccak256(abi.encode(owner, key)));
```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/execution-record/KeyValueStore.sol#L27

```solidity
File: ElectionGovernor.sol

23: electionData[0] = abi.encode(electionIndex);
```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/ElectionGovernor.sol#L23



#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }
    contract Contract0 {
        string a = "Code4rena";
        function not_optimized() public returns(bytes32){
            return keccak256(abi.encode(a));
        }
    }
    contract Contract1 {
        string a = "Code4rena";
        function optimized() public returns(bytes32){
            return keccak256(abi.encodePacked(a));
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 101871                                    | 683             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| not_optimized                             | 2661            | 2661 | 2661   | 2661 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 99465                                     | 671             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| optimized                                 | 2608            | 2608 | 2608   | 2608 | 1       |



### <a href="#gas-summary">[GAS&#x2011;2]</a><a name="GAS&#x2011;2"> Use assembly in place of `abi.decode` to extract calldata values more efficiently

Instead of using `abi.decode`, we can use assembly to decode our desired calldata values directly. This will allow us to avoid decoding calldata values that we will not use.

For example, for a generic `abi.decode` call:
```solidity
(bytes32 varA, bytes32 varB, uint256 varC, uint256 varD) = abi.decode(metadata, (bytes32, bytes32, uint256, uint256));
```

We can use the following assembly call to extract the relevant metadata:

```solidity
        bytes32 varA;
        bytes32 varB;
        uint256 varC;
        uint256 varD;
        { // used to discard `data` variable and avoid extra stack manipulation
            bytes calldata data = metadata;
            assembly {
                varA := calldataload(add(data.offset, 0x00))
                varB := calldataload(add(data.offset, 0x20))
                varC := calldataload(add(data.offset, 0x40))
                varD := calldataload(add(data.offset, 0x60))
            }
        }
```

#### <ins>Proof Of Concept</ins>

```solidity
File: SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

110: (address nominee, uint256 votes) = abi.decode(params, (address, uint256));

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L110

```solidity
File: SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol

78: (address contender, uint256 votes) = abi.decode(params, (address, uint256));

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L78




### <a href="#gas-summary">[GAS&#x2011;3]</a><a name="GAS&#x2011;3"> Avoid emitting event on every iteration

Expensive operations should always try to be avoided within loops. Such operations include: reading/writing to storage, heavy calculations, external calls, and emitting events. In this instance, an event is being emitted every iteration. Events have a base cost of `Glog (375 gas)` per emit and `Glogdata (8 gas) * number of bytes in event`. We can avoid incurring those costs each iteration by emitting the event outside of the loop.

#### <ins>Proof Of Concept</ins>

```solidity
File: SecurityCouncilManager.sol

284: for (uint256 i = 0; i < securityCouncils.length; i++) {
            SecurityCouncilData storage securityCouncilData = securityCouncils[i];
            if (
                securityCouncilData.securityCouncil == _securityCouncilData.securityCouncil
                    && securityCouncilData.chainId == _securityCouncilData.chainId
                    && securityCouncilData.updateAction == _securityCouncilData.updateAction
            ) {
                SecurityCouncilData storage lastSecurityCouncil =
                    securityCouncils[securityCouncils.length - 1];

                securityCouncils[i] = lastSecurityCouncil;
                securityCouncils.pop();
                emit SecurityCouncilRemoved(
                    securityCouncilData.securityCouncil,
                    securityCouncilData.updateAction,
                    securityCouncils.length
                );
                return true;
            }

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L284




### <a href="#gas-summary">[GAS&#x2011;4]</a><a name="GAS&#x2011;4"> Counting down in `for` statements is more gas efficient

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: UpgradeExecRouteBuilder.sol

76: for (uint256 i = 0; i < _upgradeExecutors.length; i++) {

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L76

```solidity
File: UpgradeExecRouteBuilder.sol

129: for (uint256 i = 0; i < chainIds.length; i++) {

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L129

```solidity
File: UpgradeExecRouteBuilder.sol

193: for (uint256 i = 0; i < chainIds.length; i++) {

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L193

```solidity
File: SecurityCouncilMgmtUpgradeLib.sol

61: for (uint256 i = 0; i < array1.length; i++) {
63: for (uint256 j = 0; j < array2.length; j++) {
74: for (uint256 i = 0; i < array2.length; i++) {
76: for (uint256 j = 0; j < array1.length; j++) {

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L61

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L63

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L74

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L76



```solidity
File: SecurityCouncilManager.sol

106: for (uint256 i = 0; i < _roles.memberRemovers.length; i++) {
118: for (uint256 i = 0; i < _securityCouncils.length; i++) {

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L106

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L118



```solidity
File: SecurityCouncilManager.sol

135: for (uint256 i = 0; i < _newCohort.length; i++) {

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L135

```solidity
File: SecurityCouncilManager.sol

162: for (uint256 i = 0; i < 2; i++) {
164: for (uint256 j = 0; j < cohort.length; j++) {

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L162

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L164



```solidity
File: SecurityCouncilManager.sol

251: for (uint256 i = 0; i < securityCouncils.length; i++) {

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L251

```solidity
File: SecurityCouncilManager.sol

284: for (uint256 i = 0; i < securityCouncils.length; i++) {

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L284

```solidity
File: SecurityCouncilManager.sol

339: for (uint256 i = 0; i < firstCohort.length; i++) {
342: for (uint256 i = 0; i < secondCohort.length; i++) {

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L339

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L342



```solidity
File: SecurityCouncilManager.sol

392: for (uint256 i = 0; i < securityCouncils.length; i++) {

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L392

```solidity
File: SecurityCouncilMemberSyncAction.sol

60: for (uint256 i = 0; i < _updatedMembers.length; i++) {
67: for (uint256 i = 0; i < previousOwners.length; i++) {

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L60

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L67



```solidity
File: SecurityCouncilMemberSyncAction.sol

105: for (uint256 i = 0; i < owners.length; i++) {

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L105

```solidity
File: SecurityCouncilMgmtUtils.sol

6: for (uint256 i = 0; i < arr.length; i++) {

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol#L6

```solidity
File: SecurityCouncilMgmtUtils.sol

22: for (uint256 i = 0; i < input.length; i++) {
31: for (uint256 i = 0; i < intermediateLength; i++) {

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol#L22

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol#L31



```solidity
File: L2SecurityCouncilMgmtFactory.sol

111: for (uint256 i = 0; i < dp.firstCohort.length; i++) {
117: for (uint256 i = 0; i < dp.secondCohort.length; i++) {

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol#L111

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol#L117



```solidity
File: SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

181: for (uint256 i = 0; i < nominees.length; i++) {

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L181

```solidity
File: SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

205: for (uint16 i = 0; i < nominees.length; i++) {
215: for (uint16 i = 0; i < k; i++) {

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L205

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L215





</details>


#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.AddNum();
            c1.AddNum();
        }
    }
    
    contract Contract0 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=0;i<=9;i++){
                _num = _num +1;
            }
            num = _num;
        }
    }
    
    contract Contract1 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=9;i>=0;i--){
                _num = _num +1;
            }
            num = _num;
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 77011                                     | 311             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 7040            | 7040 | 7040   | 7040 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 73811                                     | 295             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 3819            | 3819 | 3819   | 3819 | 1       |




### <a href="#gas-summary">[GAS&#x2011;5]</a><a name="GAS&#x2011;5"> Multiple accesses of a mapping/array should use a local variable cache

Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times saves ~42 gas per access due to not having to perform the same offset calculation every time. Help the Optimizer by saving a storage variable's reference instead of repeatedly fetching it

To help the optimizer,declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array. As an example, instead of repeatedly calling `someMap[someIndex]`, save its reference like this: `SomeStruct storage someStruct = someMap[someIndex]` and use it.

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: UpgradeExecRouteBuilder.sol

144: schedTargets[i] = upExecLocation.upgradeExecutor;
146: schedData[i] = executorData;
149: schedTargets[i] = RETRYABLE_TICKET_MAGIC;
151: schedData[i] = abi.encode(

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L144

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L146

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L149

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L151



```solidity
File: L2SecurityCouncilMgmtFactory.sol

112: if (!govChainEmergencySCSafe.isOwner(dp.firstCohort[i])) {
113: revert AddressNotInCouncil(owners, dp.firstCohort[i]);
118: if (!govChainEmergencySCSafe.isOwner(dp.secondCohort[i])) {
119: revert AddressNotInCouncil(owners, dp.secondCohort[i]);

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol#L112

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol#L113

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol#L118

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol#L119



```solidity
File: SecurityCouncilMemberRemovalGovernor.sol

117: if (targets[0] != address(securityCouncilManager)) {
118: revert TargetNotManager(targets[0]);
120: if (values[0] != 0) {
121: revert ValueNotZero(values[0]);
124: if (calldatas[0].length != 36) {
125: revert UnexpectedCalldataLength(calldatas[0].length);
128: (bytes4 selector, bytes memory rest) = this.separateSelector(calldatas[0]);

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L117

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L118

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L120

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L121

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L124

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L125

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L128



```solidity
File: SecurityCouncilNomineeElectionGovernor.sol

221: if (election.isContender[msg.sender]) {
240: election.isContender[msg.sender] = true;

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L221

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L240



```solidity
File: SecurityCouncilNomineeElectionGovernor.sol

272: if (election.isExcluded[nominee]) {
279: election.isExcluded[nominee] = true;

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L272

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L279



```solidity
File: SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

121: uint256 prevVotesUsed = election.votesUsed[account];
126: uint240 prevWeightReceived = election.weightReceived[nominee];
127: election.votesUsed[account] = prevVotesUsed + votes;
128: election.weightReceived[nominee] = prevWeightReceived + weight;
138: weightReceived: election.weightReceived[nominee]

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L121

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L126

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L127

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L128

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L138



```solidity
File: SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

208: if (topNomineesPacked[0] < packed) {
209: topNomineesPacked[0] = packed;

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L208-L209

```solidity
File: SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol

88: uint256 prevVotesUsed = election.votesUsed[account];
94: uint256 prevVotesReceived = election.votesReceived[contender];
107: election.votesUsed[account] = prevVotesUsed + actualVotes;
108: election.votesReceived[contender] = prevVotesReceived + actualVotes;

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L88

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L94

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L107

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L108





</details>


### <a href="#gas-summary">[GAS&#x2011;6]</a><a name="GAS&#x2011;6"> The result of a function call should be cached rather than re-calling the function

External calls are expensive. Results of external function calls should be cached rather than call them multiple times. Consider caching the following:

#### <ins>Proof Of Concept</ins>


```solidity
File: SecurityCouncilNomineeElectionGovernor.sol

236: if (securityCouncilManager.cohortIncludes(otherCohort(), msg.sender)) {
237: revert AccountInOtherCohort(otherCohort(), msg.sender);

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L236-L237

```solidity
File: SecurityCouncilNomineeElectionGovernor.sol

312: if (securityCouncilManager.cohortIncludes(otherCohort(), account)) {
313: revert AccountInOtherCohort(otherCohort(), account);

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L312-L313

```solidity
File: SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

78: if (newFullWeightDuration > votingPeriod()) {
79: revert FullWeightDurationGreaterThanVotingPeriod(newFullWeightDuration, votingPeriod());

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L78-L79



### <a href="#gas-summary">[GAS&#x2011;7]</a><a name="GAS&#x2011;7"> Use do while loops instead of for loops

A do while loop will cost less gas since the condition is not being checked for the first iteration.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: UpgradeExecRouteBuilder.sol

49: function perform

for (uint256 i = 0; i < _upgradeExecutors.length; i++) {
            ChainAndUpExecLocation memory chainAndUpExecLocation = _upgradeExecutors[i];
            if (chainAndUpExecLocation.location.upgradeExecutor == address(0)) {
                revert ZeroAddress();
            }

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L49

```solidity
File: UpgradeExecRouteBuilder.sol

105: function createActionRouteData

for (uint256 i = 0; i < chainIds.length; i++) {
            UpExecLocation memory upExecLocation = upExecLocations[chainIds[i]];
            if (upExecLocation.upgradeExecutor == address(0)) {
                revert UpgadeExecDoesntExist(chainIds[i]);
            }

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L106

```solidity
File: UpgradeExecRouteBuilder.sol

186: function createActionRouteDataWithDefaults

for (uint256 i = 0; i < chainIds.length; i++) {
            actionDatas[i] = DEFAULT_GOV_ACTION_CALLDATA;
            values[i] = DEFAULT_VALUE;
        }

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L187

```solidity
File: SecurityCouncilMgmtUpgradeLib.sol

52: function areAddressArraysEqual

for (uint256 i = 0; i < array1.length; i++) {
            bool found = false;
            for (uint256 j = 0; j < array2.length; j++) {
                if (array1[i] == array2[j]) {
                    found = true;
                    break;
                }

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L52



```solidity
File: SecurityCouncilManager.sol

89: function initialize

for (uint256 i = 0; i < _roles.memberRemovers.length; i++) {
            _grantRole(MEMBER_REMOVER_ROLE, _roles.memberRemovers[i]);
        }

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L89




```solidity
File: SecurityCouncilManager.sol

124: function replaceCohort

for (uint256 i = 0; i < _newCohort.length; i++) {
            _addMemberToCohortArray(_newCohort[i], _cohort);
        }

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L124

```solidity
File: SecurityCouncilManager.sol

161: function _removeMemberFromCohortArray

for (uint256 i = 0; i < 2; i++) {
            address[] storage cohort = i == 0 ? firstCohort : secondCohort;
            for (uint256 j = 0; j < cohort.length; j++) {
                if (_member == cohort[j]) {
                    cohort[j] = cohort[cohort.length - 1];
                    cohort.pop();
                    return i == 0 ? Cohort.FIRST : Cohort.SECOND;
                }

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L161

```solidity
File: SecurityCouncilManager.sol

231: function _addSecurityCouncil

for (uint256 i = 0; i < securityCouncils.length; i++) {
            SecurityCouncilData storage existantSecurityCouncil = securityCouncils[i];

            if (
                existantSecurityCouncil.chainId == _securityCouncilData.chainId
                    && existantSecurityCouncil.securityCouncil == _securityCouncilData.securityCouncil
            ) {
                revert SecurityCouncilAlreadyInRouter(_securityCouncilData);
            }

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L231

```solidity
File: SecurityCouncilManager.sol

279: function removeSecurityCouncil

for (uint256 i = 0; i < securityCouncils.length; i++) {
            SecurityCouncilData storage securityCouncilData = securityCouncils[i];
            if (
                securityCouncilData.securityCouncil == _securityCouncilData.securityCouncil
                    && securityCouncilData.chainId == _securityCouncilData.chainId
                    && securityCouncilData.updateAction == _securityCouncilData.updateAction
            ) {
                SecurityCouncilData storage lastSecurityCouncil =
                    securityCouncils[securityCouncils.length - 1];

                securityCouncils[i] = lastSecurityCouncil;
                securityCouncils.pop();
                emit SecurityCouncilRemoved(
                    securityCouncilData.securityCouncil,
                    securityCouncilData.updateAction,
                    securityCouncils.length
                );
                return true;
            }

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L279

```solidity
File: SecurityCouncilManager.sol

337: function getBothCohorts

for (uint256 i = 0; i < firstCohort.length; i++) {
            members[i] = firstCohort[i];
        }

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L337




```solidity
File: SecurityCouncilManager.sol

379: function getScheduleUpdateInnerData

for (uint256 i = 0; i < securityCouncils.length; i++) {
            SecurityCouncilData memory securityCouncilData = securityCouncils[i];
            actionAddresses[i] = securityCouncilData.updateAction;
            chainIds[i] = securityCouncilData.chainId;
            actionDatas[i] = abi.encodeWithSelector(
                SecurityCouncilMemberSyncAction.perform.selector,
                securityCouncilData.securityCouncil,
                newMembers,
                nonce
            );
        }

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L379

```solidity
File: SecurityCouncilMemberSyncAction.sol

31: function perform

for (uint256 i = 0; i < _updatedMembers.length; i++) {
            address member = _updatedMembers[i];
            if (!securityCouncil.isOwner(member)) {
                _addMember(securityCouncil, member, threshold);
            }

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L31




```solidity
File: SecurityCouncilMemberSyncAction.sol

97: function getPrevOwner

for (uint256 i = 0; i < owners.length; i++) {
            address currentOwner = owners[i];
            if (currentOwner == _owner) {
                return previousOwner;
            }

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L97

```solidity
File: SecurityCouncilMgmtUtils.sol

5: function isInArray

for (uint256 i = 0; i < arr.length; i++) {
            if (arr[i] == addr) {
                return true;
            }

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol#L5

```solidity
File: SecurityCouncilMgmtUtils.sol

15: function filterAddressesWithExcludeList

for (uint256 i = 0; i < input.length; i++) {
            address nominee = input[i];
            if (!excludeList[nominee]) {
                intermediate[intermediateLength] = nominee;
                intermediateLength++;
            }

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol#L15




```solidity
File: L2SecurityCouncilMgmtFactory.sol

81: function deploy

for (uint256 i = 0; i < dp.firstCohort.length; i++) {
            if (!govChainEmergencySCSafe.isOwner(dp.firstCohort[i])) {
                revert AddressNotInCouncil(owners, dp.firstCohort[i]);
            }

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol#L81




```solidity
File: SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

177: function topNominees

for (uint256 i = 0; i < nominees.length; i++) {
            weights[i] = election.weightReceived[nominees[i]];
        }

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L177

```solidity
File: SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

191: function selectTopNominees

for (uint16 i = 0; i < nominees.length; i++) {
            uint256 packed = (uint256(weights[i]) << 16) | i;

            if (topNomineesPacked[0] < packed) {
                topNomineesPacked[0] = packed;
                LibSort.insertionSort(topNomineesPacked);
            }

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L191






</details>




### <a href="#gas-summary">[GAS&#x2011;8]</a><a name="GAS&#x2011;8"> Use nested `if` and avoid multiple check combinations

Using nested `if`, is cheaper than using `&&` multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

#### <ins>Proof Of Concept</ins>

```solidity
File: SecurityCouncilMemberRemovalGovernor.sol

183: if (!(0 < _voteSuccessNumerator && _voteSuccessNumerator <= voteSuccessDenominator)) {

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L183




#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
    
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
    
        function testGas() public {
            c0.checkAge(19);
            c1.checkAgeOptimized(19);
        }
    }
    
    contract Contract0 {
    
        function checkAge(uint8 _age) public returns(string memory){
            if(_age>18 && _age<22){
                return "Eligible";
            }
        }
    
    }
    
    contract Contract1 {
    
        function checkAgeOptimized(uint8 _age) public returns(string memory){
            if(_age>18){
                if(_age<22){
                    return "Eligible";
                }
            }
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 76923                                     | 416             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| checkAge                                  | 651             | 651 | 651    | 651 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 76323                                     | 413             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| checkAgeOptimized                         | 645             | 645 | 645    | 645 | 1       |



### <a href="#gas-summary">[GAS&#x2011;9]</a><a name="GAS&#x2011;9"> Using XOR (^) and AND (&) bitwise equivalents

Given 4 variables a, b, c and d represented as such:

    0 0 0 0 0 1 1 0 <- a
    0 1 1 0 0 1 1 0 <- b
    0 0 0 0 0 0 0 0 <- c
    1 1 1 1 1 1 1 1 <- d

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: UpgradeExecRouteBuilder.sol

72: if (_l1ArbitrumTimelock == address(0)) {
78: if (chainAndUpExecLocation.location.upgradeExecutor == address(0)) {

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L72

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L78



```solidity
File: UpgradeExecRouteBuilder.sol

131: if (upExecLocation.upgradeExecutor == address(0)) {
134: if (actionDatas[i].length == 0) {
143: if (upExecLocation.inbox == address(0)) {

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L131

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L134

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L143



```solidity
File: SecurityCouncilMgmtUpgradeLib.sol

40: newSecurityCouncilThreshold == _expectedThreshold,

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L40

```solidity
File: SecurityCouncilManager.sol

133: _cohort == Cohort.FIRST ? delete firstCohort : delete secondCohort;

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L133

```solidity
File: SecurityCouncilManager.sol

144: if (_newMember == address(0)) {
147: address[] storage cohort = _cohort == Cohort.FIRST ? firstCohort : secondCohort;
148: if (cohort.length == cohortSize) {

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L144

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L147

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L148



```solidity
File: SecurityCouncilManager.sol

163: address[] storage cohort = i == 0 ? firstCohort : secondCohort;
165: if (_member == cohort[j]) {
168: return i == 0 ? Cohort.FIRST : Cohort.SECOND;

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L163

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L165

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L168



```solidity
File: SecurityCouncilManager.sol

184: if (_member == address(0)) {

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L184

```solidity
File: SecurityCouncilManager.sol

222: if (_addressToRemove == address(0) || _addressToAdd == address(0)) {

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L222

```solidity
File: SecurityCouncilManager.sol

232: if (securityCouncils.length == MAX_SECURITY_COUNCILS) {
237: _securityCouncilData.updateAction == address(0)
238: || _securityCouncilData.securityCouncil == address(0)
243: if (_securityCouncilData.chainId == 0) {
255: existantSecurityCouncil.chainId == _securityCouncilData.chainId
256: && existantSecurityCouncil.securityCouncil == _securityCouncilData.securityCouncil

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L232

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L237

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L238

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L243

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L255

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L256



```solidity
File: SecurityCouncilManager.sol

287: securityCouncilData.securityCouncil == _securityCouncilData.securityCouncil
288: && securityCouncilData.chainId == _securityCouncilData.chainId
289: && securityCouncilData.updateAction == _securityCouncilData.updateAction

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L287-L289

```solidity
File: SecurityCouncilManager.sol

365: address[] memory cohortMembers = cohort == Cohort.FIRST ? firstCohort : secondCohort;

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L365

```solidity
File: SecurityCouncilMemberSyncAction.sol

107: if (currentOwner == _owner) {

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L107

```solidity
File: L2SecurityCouncilMgmtFactory.sol

102: if (dp.nomineeVetter == address(0)) {

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol#L102

```solidity
File: SecurityCouncilNomineeElectionGovernor.sol

188: if (electionCount == 0) {

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L188

```solidity
File: SecurityCouncilNomineeElectionGovernor.sol

390: return electionCount == 0 ? Cohort.FIRST : electionIndexToCohort(electionCount - 1);

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L390

```solidity
File: SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

116: if (weight == 0) {

```

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L116



</details>


#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(1,2);
            c1.optimized(1,2);
        }
    }
    
    contract Contract0 {
        function not_optimized(uint8 a,uint8 b) public returns(bool){
            return ((a==1) || (b==1));
        }
    }
    
    contract Contract1 {
        function optimized(uint8 a,uint8 b) public returns(bool){
            return ((a ^ 1) & (b ^ 1)) == 0;
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 46099                                     | 261             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 456             | 456 | 456    | 456 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 42493                                     | 243             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 430             | 430 | 430    | 430 | 1       |
