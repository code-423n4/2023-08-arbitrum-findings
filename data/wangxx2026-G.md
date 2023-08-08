## Found can be declared once outside the loop

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L62

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L75

## Array comparison can be optimized using the map method

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L52-L89

The following implementation, the worst case time complexity is O(N*N), the best case time complexity is O(N)
``` solidity 
    function areAddressArraysEqual(address[] memory array1, address[] memory array2)
        public
        pure
        returns (bool)
    {
        if (array1.length != array2.length) {
            return false;
        }

        for (uint256 i = 0; i < array1.length; i++) {
            bool found = false;
            for (uint256 j = 0; j < array2.length; j++) {
                if (array1[i] == array2[j]) {
                    found = true;
                    break;
                }
            }
            if (!found) {
                return false;
            }
        }
       ...
    }
}
```

The following implementation, Time complexity is O(N)
``` solidity
    function areAddressArraysEqual(address[] memory array1, address[] memory array2)
        public
        pure
        returns (bool)
    {
        if (array1.length != array2.length) {
            return false;
        }
        mapping(uint256 => bool) seenElements;

        for (uint256 i = 0; i < array1.length; i++) {
            seenElements[array1[i]] = true;
        }

        for (uint256 i = 0; i < array2.length; i++) {
            if (!seenElements[array2[i]]) {
                return false;
            }
        }
        return true;
}

```
