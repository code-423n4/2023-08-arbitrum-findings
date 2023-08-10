## Summary

### Low Risk Issues
|Number|Issue|Instances| |
|-|:-|:-:|:-:|
| [L&#x2011;01] | blockNumber passed in quorum() and getPastCirculatingSupply() must be already mined | 1 |
| [L&#x2011;02] | Add msg.sender in generateSalt() to prevent DOS/front-running | 1 |
| [L&#x2011;03] | `_get()` is expecting uint256 argument but uint160 argument is passed | 1 |
| [L&#x2011;04] | `_set()` is expecting uint256 argument but uint160 argument is passed | 1 |
| [L&#x2011;05] | Misleading comment in NonGovernanceChainSCMgmtActivationAction.sol | 1 |






### [L&#x2011;01]  blockNumber passed in quorum() and getPastCirculatingSupply() must be already mined 
In ArbitrumGovernorVotesQuorumFractionUpgradeable.sol contract, quorum() and getPastCirculatingSupply() takes blocknumber as an argument to calculate quorum size and to get "circulating" votes supply. It has used openzeppelin contracts for governance and one such contract which is used for voting is VotesUpgradeable.sol where the Natspec for both function states,
```* - `timepoint` must be in the past. If operating using block numbers, the block must be already mined.```
[Reference link](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/f6401d9eaca362b90350a023473cb71964e1b9ef/contracts/governance/utils/VotesUpgradeable.sol#L112C1-L112C109). 
Usually in governance blocknumber is passed in future for voting etc. However this case is different and must be taken care per recommendation.

There are [2 instances](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/ArbitrumGovernorVotesQuorumFractionUpgradeable.sol#L32-L41) of this issue:

### Recommended Mitigation Steps
1) Add Natspec in contract ```blockNumbers passed in quorum() and getPastCirculatingSupply() must be already mined.```
2) Also document this Natspec in protocol documentation.

### [L&#x2011;02]  Add msg.sender in generateSalt() to prevent DOS/front-running
generateSalt() function is used to generate the random salt however this can be front-run or Dos in which attacker/hacker can pass the members and nonce which cause the transaction fails as the same nonce can not be used. It is therefore recommended to add the msg.sender while generating salt.

There is [1 instance](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L370) of this issue:

```Solidity
File: src/security-council-mgmt/SecurityCouncilManager.sol

    function generateSalt(address[] memory _members, uint256 nonce)
        external
        pure
        returns (bytes32)
    {
        return keccak256(abi.encodePacked(_members, nonce));
    }
```

### Recommended Mitigation Steps
```diff

    function generateSalt(address[] memory _members, uint256 nonce)
        external
        pure
        returns (bytes32)
    {
-        return keccak256(abi.encodePacked(_members, nonce));
+        return keccak256(abi.encodePacked(_members, nonce, msg.sender));
    }
```
### [L&#x2011;03]  _get() is expecting uint256 argument but uint160 argument is passed 
In SecurityCouncilMemberSyncAction.sol, getUpdateNonce() is used to get the updated nonce. It pass the address argument to _get() function further type cast it to uint160.

```Solidity
File: src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol

    function getUpdateNonce(address securityCouncil) public view returns (uint256) {
        return _get(uint160(securityCouncil));
    }
```
_get() function is given as below,

```Solidity
File: src/gov-action-contracts/execution-record/ActionExecutionRecord.sol

    function _get(uint256 key) internal view returns (uint256) {
        return store.get(computeKey(key));
    }
```

As seen, _get() is expecting uint256 argument(key) however it is being passed as uint160. This needs to be corrected to avoid the unexpected behaviour.

### Recommended Mitigation Steps
[Use](https://docs.soliditylang.org/en/v0.8.20/080-breaking-changes.html) `uint(uint160(address))` instead of `uint160(address)`

```Solidity
File: src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol

    function getUpdateNonce(address securityCouncil) public view returns (uint256) {
-        return _get(uint160(securityCouncil));
+        return _get(uint(uint160(securityCouncil)));
    }
```

### [L&#x2011;04]  `_set()` is expecting uint256 argument but uint160 argument is passed 
In SecurityCouncilMemberSyncAction.sol, _setUpdateNonce() is used to set nonce. It pass the address argument to _set() function further type cast it to uint160.

```Solidity
File: src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol

    function _setUpdateNonce(address securityCouncil, uint256 nonce) internal {
        _set(uint160(securityCouncil), nonce);
    }
```
_set() function is given as below,

```Solidity
File: src/gov-action-contracts/execution-record/ActionExecutionRecord.sol

    function _set(uint256 key, uint256 value) internal {
        store.set(computeKey(key), value);
    }
```

As seen, _set() is expecting uint256 argument(key) however it is being passed as uint160. This needs to be corrected to avoid the unexpected behaviour.

### Recommended Mitigation Steps
[Use](https://docs.soliditylang.org/en/v0.8.20/080-breaking-changes.html) `uint(uint160(address))` instead of `uint160(address)`

```Solidity
File: src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol

    function getUpdateNonce(address securityCouncil) public view returns (uint256) {
    function _setUpdateNonce(address securityCouncil, uint256 nonce) internal {
-        _set(uint160(securityCouncil), nonce);
+        _set(uint(uint160(securityCouncil), nonce));
    }
```
### [L&#x2011;05]  Misleading comment in NonGovernanceChainSCMgmtActivationAction.sol 
This comment in contract should be corrected

```diff
    function perform() external {
-        // swap in new memgency security council
+        // swap in new emergency security council
```

