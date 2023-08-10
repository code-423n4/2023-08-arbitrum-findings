|     |
| --- |
| \[L-01\] Use Ownable2StepUpgradeable instead of OwnableUpgradeable contract |
| \[L-02\] Keccak Constant values should used to immutable rather than constant |
| \[L-03\] Function Calls in Loop Could Lead to Denial of Service |
| \[L-04\] Insufficient coverage |
| \[N-01\] For modern and more readable code; update import usages |
| \[N-02\] Empty/Unused Function Parameters |
| \[N-03\] Generate perfect code headers every time |
| \[N-04\] Use SMTChecker |
| \[N-05\] NatSpec comments should be increased in |
| \[N-06\] Use a single file for all system-wide constants |
| \[S-01\] You can explain the operation of critical functions in NatSpec with an infographic. |

* * *

## \[L-01\] Use Ownable2StepUpgradeable instead of OwnableUpgradeable contract

```
contract SecurityCouncilNomineeElectionGovernor is
    Initializable,
    GovernorUpgradeable,
    GovernorVotesUpgradeable,
    SecurityCouncilNomineeElectionGovernorCountingUpgradeable,
    ArbitrumGovernorVotesQuorumFractionUpgradeable,
    GovernorSettingsUpgradeable,
    OwnableUpgradeable, // <= FOUND
    SecurityCouncilNomineeElectionGovernorTiming,
    ElectionGovernor,
    ISecurityCouncilNomineeElectionGovernor
{
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L17C1-L26C2

```
contract SecurityCouncilMemberRemovalGovernor is
    Initializable,
    GovernorUpgradeable,
    GovernorVotesUpgradeable,
    GovernorPreventLateQuorumUpgradeable,
    GovernorCountingSimpleUpgradeable,
    ArbitrumGovernorVotesQuorumFractionUpgradeable,
    GovernorSettingsUpgradeable,
    ArbitrumGovernorProposalExpirationUpgradeable,
    OwnableUpgradeable
{
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L17C1-L27C2

```
abstract contract SecurityCouncilNomineeElectionGovernorTiming is
    Initializable,
    GovernorUpgradeable
{
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol#L12C1-L15C2

## \[L-02\] Keccak Constant values should used to immutable rather than constant

There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts.

While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand.

```
bytes32 public constant COHORT_REPLACER_ROLE = keccak256("COHORT_REPLACER");
    bytes32 public constant MEMBER_ADDER_ROLE = keccak256("MEMBER_ADDER");
    bytes32 public constant MEMBER_REPLACER_ROLE = keccak256("MEMBER_REPLACER");
    bytes32 public constant MEMBER_ROTATOR_ROLE = keccak256("MEMBER_ROTATOR");
    bytes32 public constant MEMBER_REMOVER_ROLE = keccak256("MEMBER_REMOVER");
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L79C5-L83C79

## \[L-03\] Function Calls in Loop Could Lead to Denial of Service

Function calls made in unbounded loop are error-prone with potential resource exhaustion as it can trap the contract due to the gas limitations or failed transactions. Here are some of the instances entailed:

```
 for (uint256 i = 0; i < securityCouncils.length; i++) {
            SecurityCouncilData memory securityCouncilData = securityCouncils[i];
            actionAddresses[i] = securityCouncilData.updateAction;
            chainIds[i] = securityCouncilData.chainId;
            actionDatas[i] = abi.encodeWithSelector(   //  <=FOUND
                SecurityCouncilMemberSyncAction.perform.selector,
                securityCouncilData.securityCouncil,
                newMembers,
                nonce
            );
        }
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L392C9-L401C15

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L89C4-L121C6

Consider bounding the loop where possible to avoid unnecessary gas wastage and denial of service.

## \[L-04\] Insufficient coverage

Description
The test coverage rate of the project is ~94%. Testing all functions is best practice in terms of security criteria.

Due to its capacity, test coverage is expected to be 100%.

## \[N-01\] For modern and more readable code; update import usages

> Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.
> This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.
> 
> ### *Recommendation*
> 
> `import {contract1 , contract2} from "filename.sol";`

> All contest

## \[N-02\] Empty/Unused Function Parameters

Empty or unused function parameters should be commented out as a better and declarative way to silence runtime warning messages. As an example, the following function may have these parameters refactored to:

```
 function _execute(
        uint256 proposalId,
        address[] memory, /* targets */
        uint256[] memory, /* values */
        bytes[] memory callDatas,
        bytes32 /*descriptionHash*/
```

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L324C4-L329C36

## \[N-03\] Generate perfect code headers every time

Description:
I recommend using header for Solidity code layout and readability

```
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

## \[N-04\] Use SMTChecker

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

## \[N-05\] NatSpec comments should be increased in contracts

> all contest

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity Official documentation

in complext project such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

## \[N-06\] Use a single file for all system-wide constants

There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values).

This will help with readability and easier maintenance for future changes. This also helps with any issues, as some of these hard-coded values are admin addresses.

constants.sol
Use and import this file in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution.

## \[S-01\] You can explain the operation of critical functions in NatSpec with an infographic.