1.

See SecurityCouncilMgmtUpgradeLib#areAddressArraysEqual

```solidity
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

        for (uint256 i = 0; i < array2.length; i++) {
            bool found = false;
            for (uint256 j = 0; j < array1.length; j++) {
                if (array2[i] == array1[j]) {
                    found = true;
                    break;
                }
            }
            if (!found) {
                return false;
            }
        }

        return true;
    }
```

can be changed to

```solidity
  function areAddressArraysEqual(address[] memory array1, address[] memory array2)
    public
    pure
    returns (bool)
    {
        return keccak256(abi.encodePacked(array1)) == keccak256(abi.encodePacked(array2));
    }
```

Wrote 4 foundry test

1. Using the old function, array1 and array2 have 5 same elements (Gas spent = 11524)
2. Using the new function, array1 and array2 have 5 same elements (Gas spent = 4012)
3. Using the old function, array1 and array2 have 5 different elements (Gas spent = 4034)
4. Using the new function, array1 and array2 have 5 different elements (Gas spent = 4012)