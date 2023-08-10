## 1. Its better not to return a constant address of precompiled contract.

```Solidity
return (
            address(100),
            abi.encodeWithSelector(ArbSys.sendTxToL1.selector, l1TimelockAddr, timelockCallData)
        );
```
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L174

`UpgradeExecRouteBuilder.sol/createActionRouteData(...)` has a return value of address which is a constant and never change. This will be better not to return the constant value as this will save gas of extra interaction with memory while returning the output and extra calldata interaction while using the function.

## 2. No need to introduce a new local variable as it is used ony one time and is only increasing the memory interaction by adding MSTORE and MLOAD.
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L291
`lastSecurityCouncil` variable is introduced pointing to the storage but is only used one time which defeats the purpose of its declaration in the first place.

## 3. Better to declare the names of return variables inside the `returns()` argument of the function
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L379

Function can be declared like this 
```Solidity
 function getScheduleUpdateInnerData(uint256 nonce)
        public
        view
        returns (address[] memory newMembers, address to , bytes memory data)
```

## 4. `securityCouncils.length` can be cached in memory to be used inside the function `SecurityCouncilManager.sol/getScheduleUpdateInnerData(...)` as it is called 4 times and gas could be saved by caching it.
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L388-L394