# Findings Summary

| ID     | Title                                           | Severity     |
| ------ | ----------------------------------------------- | ------------ |
| [L-01] | The relay function should be marked as payable  | Low          |

# Detailed Findings

# [L-01] The relay function should be marked as payable

## Description

```solidity
    function relay(address target, uint256 value, bytes calldata data)
        external
        virtual
        override
        onlyOwner
    {
        AddressUpgradeable.functionCallWithValue(target, data, value);
    }
```

The relay function support value param, but contract does not have any payable method, can't transfer-in msg.value.   

## Recommendations

1. Hard-coded value to 0
2. Mark the relay function as payable
