
```
library SecurityCouncilMgmtUtils {
    function isInArray(address addr, address[] memory arr) internal pure returns (bool) {
        for (uint256 i = 0; i < arr.length; i++) {
            if (arr[i] == addr) {
                return true;
            }
        }
        return false;
    }
```
here we are Reading array length at each iteration of the loop takes 6 gas (three for mload and three to place memory_offset ) in the stack. Caching the array length in the stack saves around 3 gas per iteration. I suggest storing the array’s length in a variable before the for-loop. 
so we should use  
uint length = arr.length; 
before the for loop in order to save some gas also 

this can be done
```
for (uint i; i < length;) {
    unchecked { ++i; }
}
```
as ++i costs less than the i++
