## GAS OPTIMIZATIONS

1. ## Reducing Bytecode Size With Modifiers

We could consider refactoring  modifiers so it calls an internal function. This approach is used in OpenZeppelin's Ownable.sol:

``` solidity
modifier onlyOwner() {
    _checkOwner();
    _;
}

function _checkOwner() internal view virtual {
    if (owner() != _msgSender()) {
        revert OwnableUnauthorizedAccount(_msgSender());
    }
}
```
Here, the onlyOwner() modifier calls checkOwner() to validate whether the msg.sender is the owner. This is a more optimized version of:
``` solidity
modifier onlyOwner() {
    require(owner() == msg.sender, "Not owner");
    ;
}
```

2. ## Cache securityCouncils.length in SecurityCouncilManager#getScheduleUpdateInnerData

As we can see, securityCouncils.length is used multiple times here and is computed each time: 

```solidity
        // build batch call to L1 timelock
        //@audit GAS: cache securityCouncils.length
        address[] memory actionAddresses = new address[](securityCouncils.length);
        bytes[] memory actionDatas = new bytes[](securityCouncils.length);
        uint256[] memory chainIds = new uint256[](securityCouncils.length);

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

Here, it should be cached and used instead of being computed each time. 

3. ## Unnecessary copying to new array in SecurityCouncilMgmtUtils#filterAddressesWithExcludeList

In the above mentioned function, there is unnecessary copying done to 'output' array when the intermediate array could itself be returned directly. Here is the code: 

```solidity 
    function filterAddressesWithExcludeList(
        address[] memory input,
        mapping(address => bool) storage excludeList
    ) internal view returns (address[] memory) {
        address[] memory intermediate = new address[](input.length);
        uint256 intermediateLength = 0;

        for (uint256 i = 0; i < input.length; i++) {
            address nominee = input[i];
            if (!excludeList[nominee]) {
                intermediate[intermediateLength] = nominee;
                intermediateLength++;
            }
        }
    //@audit GAS: copying for no reason
        address[] memory output = new address[](intermediateLength);
        for (uint256 i = 0; i < intermediateLength; i++) {
            output[i] = intermediate[i];
        }

        return output;
    }
```
The intermediate array can be returned directly here instead of being copied to a new array. 

4. ## SecurityCouncilMgmtUpgradeLib#areAddressArraysEqual is inefficient

The method used to check if arrays are equal is inefficient and is gas consuming. Instead, this code can be used to check if arrays are equal: 

```solidity
function areAddressArraysEqual(address[] memory _array1, address[] memory _array2) public pure returns (bool) {
        if(_array1.length != _array2.length) {
            return false;
        }

        uint sum1;
        uint sum2;

        for(uint i = 0; i < _array1.length; i++) {
            sum1 += uint(keccak256(abi.encodePacked(_array1[i])));
            sum2 += uint(keccak256(abi.encodePacked(_array2[i])));
        }

        return sum1 == sum2;
    }
```

We observed a gas decrease of more than around 20% with the newly implemented code. 
