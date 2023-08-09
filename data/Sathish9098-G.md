# GAS OPTIMIZATION

##

## [G-1] Using calldata instead of memory for read-only arguments in external functions saves gas

When a function with a memory array is called externally, the abi.decode ()  step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution . Saves at least ``282 GAS`` per instance .

### ``_firstCohort``,``_secondCohor``,``_securityCouncils``,``_roles`` can be calldata instead of memory since all parameters are read only : Saves ``1128`` GAS


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

##

## [G-] State variables can be packed into fewer storage slots

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables can also be cheaper.


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



Cache function calls instead of calling repeat 

don't use external calls inside the loops 

Don't emit state variable 

Combine two emit in single 

struct can be packed in fewer slots 

+= not used 

Multiple accesses of a mapping/array should use a local variable cache

require/if checks should be top of the function 


Structs can be used instead of mapping 

bool incurs overhead

Don't initialize default values 

## [G-] Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declaring the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incurring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

