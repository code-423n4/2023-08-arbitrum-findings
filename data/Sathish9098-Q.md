
Division by zero not prevented
The divisions below take an input parameter which does not have any zero-value checks, which may lead to the functions reverting when zero is passed.

Lack of integer validation 

use latest version of openzheplin 

Hard coded fees structure 

[L‑04]	Missing contract-existence checks before low-level calls

Low-level calls return success if there is no code present at the specified address. In addition to the zero-address checks, add a check to verify that <address>.code.length > 0

444              (success[i], result[i]) = mc[i].call(data[i]);

module.delegatecall(
175              abi.encodeWithSelector(
176                  this.exerciseInternal.selector,
177                  optionsData.from,
178                  optionsData.oTAPTokenID,
179:                 optionsData.paymentToken,

[L‑05]	External call recipient may consume all transaction gas	12
There is no limit specified on the amount of gas used, so the recipient can use up all of the transaction's gas, causing it to revert. Use addr.call{gas: <amount>}("") or this library instead.

/// @audit `executeMarketFn()`
444:             (success[i], result[i]) = mc[i].call(data[i]);

## Multiplication before division

State variables not capped at reasonable values
Consider adding minimum/maximum value checks to ensure that the state variables below can never be used to excessively harm users, including via griefing

Allowed fees/rates should be capped by smart contracts
Fees/rates should be required to be below 100%, preferably at a much lower limit, to ensure users don't have to monitor the blockchain for changes prior to using the protocol

There are 2 instances of this issue:

File: contracts/Penrose.sol

256      function setBigBangEthMarketDebtRate(uint256 _rate) external onlyOwner {
257          bigBangEthDebtRate = _rate;
258          emit BigBangEthMarketDebtRate(_rate);
259:     }

[L‑12] Solidity version 0.8.20 may not work on other chains due to PUSH0

The compiler for Solidity 0.8.20 switches the default target EVM version to Shanghai, which includes the new PUSH0 op code. This op code may not yet be implemented on all L2s, so deployment on these chains will fail. To work around this issue, use an earlier EVM version. While the project itself may or may not compile with 0.8.20, other projects with which it integrates, or which extend this project may, and those projects will have problems deploying these contracts/libraries.

File: contracts/Penrose.sol

2:   pragma solidity ^0.8.18;

Array lengths not checked
If the length of the arrays are not required to be of the same length, user operations may not be fully executed due to a mismatch in the number of items iterated over, versus the number of items provided in the second array

Empty receive()/payable fallback() function does not authorize requests
If the intention is for the Ether to be used, the function should call another function, otherwise it should revert (e.g. require(msg.sender == address(weth))). Having no access control on the function means that someone may send Ether to the contract, and have no way to get anything back out, which is a loss of funds. If the concern is having to spend a small amount of gas to check the sender against an immutable address, the code should at least have a function to rescue unused Ether.

Signature use at deadlines should be allowed
According to EIP-2612, signatures used on exactly the deadline timestamp are supposed to be allowed. While the signature may or may not be used for the exact EIP-2612 use case (transfer approvals), for consistency's sake, all deadlines should follow this semantic. If the timestamp is an expiration rather than a deadline, consider whether it makes more sense to include the expiration timestamp as a valid timestamp, as is done for deadlines.

There are 3 instances of this issue:

File: contracts/governance/twTAP.sol

159:         if (participant.expiry < block.timestamp) {
https://github.com/code-423n4/2023-07-tapioca/blob/e6eef060495b31173578570215e80f9e95330b9a/tap-token-audit/contracts/governance/twTAP.sol#L159-L159

File: contracts/option-airdrop/AirdropBroker.sol

158:         require(aoTapOption.expiry > block.timestamp, "adb: Option expired");

233:         require(aoTapOption.expiry > block.timestamp, "adb: Option expired");
https://github.com/code-423n4/2023-07-tapioca/blob/e6eef060495b31173578570215e80f9e95330b9a/tap-token-audit/contracts/option-airdrop/AirdropBroker.sol#L158-L158

Open TODOs
Code architecture, incentives, and error handling/reporting questions/issues should be resolved before deployment

There are 10 instances of this issue:

File: tap-token-audit/contracts/governance/twTAP.sol

233:              //    (TODO: Word better?)

289:                  // TODO: Strongly suspect this is never less. Prove it.

347:          // TODO: Mint event?

398:          // TODO: Make whole function unchecked

411:              // TODO: Prove that math is safe

444:          // TODO: Word this better

Constructor should be disableInitializer() when contract is upgradable 

Check that authorizeUpgrade() is properly secured if UUPS







