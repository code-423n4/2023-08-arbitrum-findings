# LOW FINDINGS

##

## [L-1] ``generateSalt()`` function is not secure 

### Impact
 Even if the ``updateNonce``  value is increased every time a new proposal is scheduled, the ``generateSalt ``function is still not a secure function. This is because the ``generateSalt`` function simply concatenates the ``newMembers`` array and the ``updateNonce`` variable together to create a salt. This salt is not very secure, and it is possible for an attacker to generate a collision and create a valid proposal with the same salt as another proposal

### POC
```solidity

440: salt: this.generateSalt(newMembers, updateNonce),

370: function generateSalt(address[] memory _members, uint256 nonce)
371:        external
372:        pure
373:        returns (bytes32)
374:    {
375:        return keccak256(abi.encodePacked(_members, nonce));
376:    }

```
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L440

### Recommended Mitigation
To make the generateSalt function more secure, you could use a more secure hash function, such as the ``sha3 ``function

##

## [L-] Division by zero not prevented

### Impact
The divisions below take an input parameter which does not have any zero-value checks, which may lead to the functions reverting when zero is passed. ``params.quorumNumeratorValue`` should be checked before division operation

### POC
```solidity
FILE: governance/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

128: if ((quorumDenominator() / params.quorumNumeratorValue) > 500) {

```
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L128

```solidity
FILE: Breadcrumbsgovernance/src/security-council-mgmt/governors/modules
/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

252: uint256 decreaseAmount =
253:            votes * (blockNumber - fullWeightVotingDeadline_) / decreasingWeightDuration;

```
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L252-L253


### Recommended Mitigation
Add the ``non zero check`` before perform division 

##

## [L-] Lack of sanity checks when assigning ``uint`` values to critical variables in ``constructor`` and ``initializer`` function

### Impact
The contract's does not have any sanity checks when assigning ``uint`` values to critical variables in the ``constructor`` and ``initializer`` function. This could lead to human errors that would require the contract to be redeployed.

### POC

```solidity
FILE: governance/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol

41: emergencySecurityCouncilThreshold = _emergencySecurityCouncilThreshold;
42: nonEmergencySecurityCouncilThreshold = _nonEmergencySecurityCouncilThreshold;

```
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol#L41-L42

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L63-L75


### Recommended Mitigation
Add ``> 0``, ``MAX``, ``MIN`` value checks 


##

## [L-] ``Protocol`` uses the ``vulnerable`` version of ``openzeppelin version``

### Impact
The Protocol contract uses the ``vulnerable`` version of the ``openzeppelin`` library. This means that the contract is susceptible to a number of security vulnerabilities that have been patched in newer versions of the library.

Protocol uses the contracts": "4.7.3", contracts-upgradeable": "4.7.3"  there are known vulnerability in this version 

  - Improper Input Validation
  - Missing Authorization
  - Denial of Service (DoS)
  - Improper Verification of Cryptographic Signature

### POC
```solidity
FILE: package.json

69: "@openzeppelin/contracts": "4.7.3",
70: "@openzeppelin/contracts-upgradeable": "4.7.3",

```

### Recommended Mitigation
Use at least ``openzeppelin : 4.9.2 ``

##

## [L-] Hardcoded ``MAX_SECURITY_COUNCILS`` may cause problem in future 

### Impact
The ``MAX_SECURITY_COUNCILS`` constant in the Protocol contract is hardcoded to a value of ``500``. This means that there can only be a maximum of 500 security councils in the protocol. This could be a problem in the future if the protocol needs to support more than 10 security councils.

If any problem need to ``increase/decrease`` ``security councils`` means its not possible. Need to redeploy the over all contract 

### POC

```solidity
FILE: governance/src/security-council-mgmt/SecurityCouncilManager.sol

67: uint256 public immutable MAX_SECURITY_COUNCILS = 500;

```
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L67

### Recommended Mitigation
Make ``MAX_SECURITY_COUNCILS`` value configurable 

##

## [L-] Project Upgrade and Stop Scenario should be

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation. This can also be called an ” EMERGENCY STOP (CIRCUIT BREAKER) PATTERN “.

https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol

##






## [L-] Expressions for constant values such as a call to ``keccak256()``, should use ``immutable`` rather than ``constant``

### Impact

While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand. There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts. constants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor.

```solidity
FILE: Breadcrumbsgovernance/src/security-council-mgmt/SecurityCouncilManager.sol

79:    bytes32 public constant COHORT_REPLACER_ROLE = keccak256("COHORT_REPLACER");
80:    bytes32 public constant MEMBER_ADDER_ROLE = keccak256("MEMBER_ADDER");
81:    bytes32 public constant MEMBER_REPLACER_ROLE = keccak256("MEMBER_REPLACER");
82:    bytes32 public constant MEMBER_ROTATOR_ROLE = keccak256("MEMBER_ROTATOR");
83:    bytes32 public constant MEMBER_REMOVER_ROLE = keccak256("MEMBER_REMOVER");

```
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L79-L83

### Recommended Mitigation
Use immutable instead of constant 

##

## [L-] Shorter the inheritance list 

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L16-L27

##

## [L-] initialize() function visibility should be ``external ``

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L58-L69

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L55

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L103




