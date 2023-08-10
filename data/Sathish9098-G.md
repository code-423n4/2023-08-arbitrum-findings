# GAS OPTIMIZATION

All gas optimizations is calculated based on opcodes

| Issue Count| Issues | Instances | Gas Saved |
|-----------------|-----------------|-----------------|-----------------|
| [G-1]  | Using calldata instead of memory for read-only arguments in external functions saves gas   | 10   | 2820   |
| [G-2]  | Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate   | 4  | 8000   |
| [G-3]  |  State variables can be packed into fewer storage slots  | 2  | 4000   |
| [G-4]  | IF’s/require() statements that check input arguments should be at the top of the function   | 5   | 10000   |
| [G-5]  | Don't emit state variable when stack variable available  | 1  | 100   |
| [G-6]  | Using bools for storage incurs overhead  | 3   | 60000  |
| [G-7]  | Structs can be packed into fewer storage slots   | 2   | 4000 |


##

## [G-1] Using calldata instead of memory for read-only arguments in external functions saves gas

### Saved ``2820 GAS``

When a function with a memory array is called externally, the abi.decode ()  step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution . Saves at least ``282 GAS`` per instance .

### ``_firstCohort``,``_secondCohor``,``_securityCouncils``,``_roles`` can be calldata instead of memory since all parameters are read only : Saves ``1974`` GAS


https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L89-L96

```diff
FILE: governance/src/security-council-mgmt/SecurityCouncilManager.sol

89: function initialize(
- 90:        address[] memory _firstCohort,
+ 90:        address[] calldata _firstCohort,
- 91:        address[] memory _secondCohort,
+ 91:        address[] calldata _secondCohort,
- 92:        SecurityCouncilData[] memory _securityCouncils,
+ 92:        SecurityCouncilData[] calldata _securityCouncils,
- 93:        SecurityCouncilManagerRoles memory _roles,
+ 93:        SecurityCouncilManagerRoles calldata _roles,
94:        address payable _l2CoreGovTimelock,
95:        UpgradeExecRouteBuilder _router
96:    ) external initializer {


- 124: function replaceCohort(address[] memory _newCohort, Cohort _cohort)
+ 124: function replaceCohort(address[] memory calldata, Cohort _cohort)
125:        external
126:        onlyRole(COHORT_REPLACER_ROLE)
127:    {

- 271: function addSecurityCouncil(SecurityCouncilData memory _securityCouncilData)
+ 271: function addSecurityCouncil(SecurityCouncilData calldata _securityCouncilData)
272:        external
273:        onlyRole(DEFAULT_ADMIN_ROLE)
274:    {

- 279: function removeSecurityCouncil(SecurityCouncilData memory _securityCouncilData)
+ 279: function removeSecurityCouncil(SecurityCouncilData calldata _securityCouncilData)
280:        external
281:        onlyRole(DEFAULT_ADMIN_ROLE)
282:        returns (bool)

```

### ``_updatedMembers`` can be ``calldata`` instead of memory : Saves ``282 GAS``

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L31-L33

```diff
FILE: Breadcrumbsgovernance/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol

- 31: function perform(address _securityCouncil, address[] memory _updatedMembers, uint256 _nonce)
+ 31: function perform(address _securityCouncil, address[] calldata _updatedMembers, uint256 _nonce)
32:        external
33:        returns (bool res)
34:    {

```

### ``dp,impls`` can be ``calldata`` instead of ``memory`` : Saves ``564`` GAS

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol#L81

```diff
FILE: governance/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol

81: function deploy(DeployParams memory dp, ContractImplementations memory impls)
82:        external
83:        onlyOwner
84:        returns (DeployedContracts memory)

```

## [G-2] Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate

### Saves ``8000 GAS, 4 SLOT``

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.

###

```diff
FILE: governance/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

+    struct UserStatus {
+        bool isContender;
+        bool isExcluded;
+    }


54: struct ElectionInfo {
- 55:        mapping(address => bool) isContender;
- 56:        mapping(address => bool) isExcluded;
+           UserStatus userStatus;
57:        uint256 excludedNomineeCount;
58:    }

```
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L55-L56

```diff
FILE: governance/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol

+ struct NomineeCountingInfo {
+        uint256 votesUsed;
+        uint256 votesReceived;
+        bool isNominee;
+    }

18: struct NomineeElectionCountingInfo {
+    NomineeCountingInfo _nomineeCountingInfo ;
- 19:        mapping(address => uint256) votesUsed;
- 20:        mapping(address => uint256) votesReceived;
21:        address[] nominees;
- 22:        mapping(address => bool) isNominee;
23:    }

```
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L19-L20

##

## [G-3] State variables can be packed into fewer storage slots

### Saves ``4000 GAS, 2 SLOT``

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables can also be cheaper.


### ``voteSuccessNumerator`` can be uint96 instead of uint256 because of this check ``_voteSuccessNumerator <= voteSuccessDenominator)``. The value always less than ``10000`` because ``voteSuccessDenominator`` constant : Saves ``2000 GAS, 1 SLOT ``

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L28C1-L30


```diff
FILE: Breadcrumbsgovernance/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol

28:  uint256 public constant voteSuccessDenominator = 10_000;
- 29:    uint256 public voteSuccessNumerator;
+ 29:    uint96 public voteSuccessNumerator;
30:    ISecurityCouncilManager public securityCouncilManager;

```

### ``cohortSize`` can be changed uint96 from uint256. ``cohortSize`` saves only ``_firstCohort.length `` _firstCohort parameter length. The array length is not going to exceeds uint96 range ``18446744073709551615``
: Saves ``2000 GAS, 1 SLOT``


https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L55-L73

```diff
FILE: Breadcrumbsgovernance/src/security-council-mgmt/SecurityCouncilManager.sol


 /// @notice Address of the l2 timelock used by core governance
    address payable public l2CoreGovTimelock;
+   uint96 public cohortSize;

    /// @notice The list of Security Councils under management. Any changes to the cohorts in this manager
    ///         will be pushed to each of these security councils, ensuring that they all stay in sync
    SecurityCouncilData[] public securityCouncils;

    /// @notice Address of UpgradeExecRouteBuilder. Used to help create security council updates
    UpgradeExecRouteBuilder public router;

    /// @notice Maximum possible number of Security Councils to manage
    /// @dev    Since the councils array will be iterated this provides a safety check to make too many Sec Councils
    ///         aren't added to the array.
    uint256 public immutable MAX_SECURITY_COUNCILS = 500;

    /// @notice Nonce to ensure that scheduled updates create unique entries in the timelocks
    uint256 public updateNonce;

    /// @notice Size of cohort under ordinary circumstancces
-    uint256 public cohortSize;


```

##

## [G-4] IF’s/require() statements that check input arguments should be at the top of the function

### Saves ``10000 GAS``

FAIL CHEAPLY INSTEAD OF COSTLY 

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case. 

###  ``!Address.isContract(address(params.securityCouncilManager)``,``!Address.isContract(address(params.securityCouncilMemberElectionGovernor)``, ``(quorumDenominator() / params.quorumNumeratorValue) > 500)`` should performed top of the the functions. This will avoid the unwanted storage writes : Saves around ``6000 GAS`` . If any revert after storage writes this wastes the computation cost 


https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L111-L130

```diff
FILE: governance/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

+        if (!Address.isContract(address(params.securityCouncilManager))) {
+            revert NotAContract(address(params.securityCouncilManager));
+        }
+        if (!Address.isContract(address(params.securityCouncilMemberElectionGovernor))) {
+            revert NotAContract(address(params.securityCouncilMemberElectionGovernor));
+        }

+        if ((quorumDenominator() / params.quorumNumeratorValue) > 500) {
+            revert QuorumNumeratorTooLow(params.quorumNumeratorValue);
+        }


      _transferOwnership(params.owner);

        nomineeVetter = params.nomineeVetter;
-        if (!Address.isContract(address(params.securityCouncilManager))) {
-            revert NotAContract(address(params.securityCouncilManager));
-        }
        securityCouncilManager = params.securityCouncilManager;
-        if (!Address.isContract(address(params.securityCouncilMemberElectionGovernor))) {
-            revert NotAContract(address(params.securityCouncilMemberElectionGovernor));
-        }
        securityCouncilMemberElectionGovernor = params.securityCouncilMemberElectionGovernor;

        // elsewhere we make assumptions that the number of nominees
        // is not greater than 500
        // This value can still be updated via updateQuorumNumerator to a lower value
        // if it is deemed ok, however we put a quick check here as a reminder
-        if ((quorumDenominator() / params.quorumNumeratorValue) > 500) {
-            revert QuorumNumeratorTooLow(params.quorumNumeratorValue);
-        }

```

### ``!Address.isContract(address(_nomineeElectionGovernor))``,``!Address.isContract(address(_securityCouncilManager)`` should be checked top of the function : Saves ``4000 GAS``

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L68-L74


```diff
FILE: governance/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol


if (_fullWeightDuration > _votingPeriod) {
            revert InvalidDurations(_fullWeightDuration, _votingPeriod);
        }

+        if (!Address.isContract(address(_nomineeElectionGovernor))) {
+            revert NotAContract(address(_nomineeElectionGovernor));
+        }

+        if (!Address.isContract(address(_securityCouncilManager))) {
+            revert NotAContract(address(_securityCouncilManager));
+        }

        __Governor_init("SecurityCouncilMemberElectionGovernor");
        __GovernorVotes_init(_token);
        __SecurityCouncilMemberElectionGovernorCounting_init({
            initialFullWeightDuration: _fullWeightDuration
        });
        __GovernorSettings_init(0, _votingPeriod, 0);
        _transferOwnership(_owner);

-        if (!Address.isContract(address(_nomineeElectionGovernor))) {
-            revert NotAContract(address(_nomineeElectionGovernor));
-        }
        nomineeElectionGovernor = _nomineeElectionGovernor;
-        if (!Address.isContract(address(_securityCouncilManager))) {
-            revert NotAContract(address(_securityCouncilManager));
-        }
        securityCouncilManager = _securityCouncilManager;
    }

```

##

## [G-5] Don't emit state variable when stack variable available 

### Saves ``100 GAS, 1 SLOD``

Emitting stack variable is more gas efficient than state variable . Saves ``100 GAS ``

### ``prevWeightReceived + weight`` should be used : ``saves 100 GAS, 1 SLOD ``

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L138

```solidity
FILE: Breadcrumbsgovernance/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

 election.weightReceived[nominee] = prevWeightReceived + weight;

        emit VoteCastForNominee({
            voter: account,
            proposalId: proposalId,
            nominee: nominee,
            votes: votes,
            weight: weight,
            totalUsedVotes: prevVotesUsed + votes,
            usableVotes: availableVotes,
-          weightReceived: election.weightReceived[nominee]
+          weightReceived: prevWeightReceived + weight
        });

```

## [G-6] Using bools for storage incurs overhead

### Saves ``60000 GAS, 3 Instances ``

```

   // Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.

```

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27 Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from false to true, after having been true in the past

```solidity
FILE: Breadcrumbsgovernance/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

55:        mapping(address => bool) isContender;
56:        mapping(address => bool) isExcluded;
 
```
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L55-L56

```diff
FILE: Breadcrumbsgovernance/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol

22:        mapping(address => bool) isNominee;

```
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol#L22

##

## [G-7] Structs can be packed into fewer storage slots

### Saves ``4000 GAS, 2 SLOT``

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct.

### ``quorumNumeratorValue`` can be ``uint96`` instead of ``uint256``. As per this check if ((quorumDenominator() / params.quorumNumeratorValue) > 500) { ``uint96`` is safe. Since ``quorumDenominator()`` function always return ``10000`` . So the ``numerator`` not exceeds ``denominator ``. If exceeds this will revert with divide by zero exception : Saves ``2000 GAS, 1 SLOT``

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L38-L48

```diff
FILE: Breadcrumbsgovernance/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

38: struct InitParams {
39:        Date firstNominationStartDate;
40:        uint256 nomineeVettingDuration;
41:        address nomineeVetter;
42:        ISecurityCouncilManager securityCouncilManager;
43:        ISecurityCouncilMemberElectionGovernor securityCouncilMemberElectionGovernor;
44:        IVotesUpgradeable token;
45:        address owner;
+ 46:        uint96 quorumNumeratorValue;
- 46:        uint256 quorumNumeratorValue;
47:        uint256 votingPeriod;
48:    }

```

### ``chainId`` can be uint96 instead of uint256. Saves ``2000 GAS, 1 SLOT``

https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L22-L25

The ``chainId`` is a parameter that uniquely identifies different ``Ethereum networks``. The ``chainId`` is typically used to prevent ``replay attacks`` when creating transactions or signing messages that are intended for a specific network. Ethereum chains generally have values in the range of ``1 to 4294967295`` (32-bit unsigned integer range).

Using a ``uint96`` instead of a ``uint256`` for the ``chainId`` could potentially save some gas cost in storage and execution, as smaller data types consume less storage space and require less computational effort to manipulate.


```diff
FILE: governance/src/UpgradeExecRouteBuilder.sol

17: struct UpExecLocation {
18:    address inbox; // Inbox should be set to address(0) to signify that the upgrade executor is on the L1/host chain
19:    address upgradeExecutor;
20: }


22: struct ChainAndUpExecLocation {
- 23:    uint256 chainId;
+ 23:    uint96 chainId;
24:    UpExecLocation location;
25: }

```



