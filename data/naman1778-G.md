## [G-01] Using *calldata* instead of *memory* for read-only arguments in *external* functions saves gas

When a function with a *memory* array is called externally, the *abi.decode()* step has to use a for-loop to copy each index of the *calldata* to the *memory* index. Each iteration of this for-loop costs at least 60 gas (i.e. *60 * <mem_array>.length*). Using *calldata* directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having *memory* arguments, it’s still valid for implementation contracs to use *calldata* arguments instead.

If the array is passed to an *internal* function which passes the array to another internal function where the array is modified and therefore *memory* is used in the *external* call, it’s still more gass-efficient to use *calldata* when the *external* function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

There are 7 instances of this issue in 3 files:

    File: SecurityCouncilManager.sol	

    89: function initialize(
    90:     address[] memory _firstCohort,
    91:     address[] memory _secondCohort,
    92:     SecurityCouncilData[] memory _securityCouncils,
    93:     SecurityCouncilManagerRoles memory _roles,
    94:     address payable _l2CoreGovTimelock,
    95:     UpgradeExecRouteBuilder _router
    96: ) external initializer {

    124: function replaceCohort(address[] memory _newCohort, Cohort _cohort)
    125:     external
    126:     onlyRole(COHORT_REPLACER_ROLE)

    271: function addSecurityCouncil(SecurityCouncilData memory _securityCouncilData)
    272:     external
    273:     onlyRole(DEFAULT_ADMIN_ROLE)

    279: function removeSecurityCouncil(SecurityCouncilData memory _securityCouncilData)
    280:     external
    281:     onlyRole(DEFAULT_ADMIN_ROLE)
    282:     returns (bool)

    370: function generateSalt(address[] memory _members, uint256 nonce)
    371:     external
    372:     pure
    373:     returns (bytes32)

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol

    File: SecurityCouncilMemberSyncAction.sol	

    31: function perform(address _securityCouncil, address[] memory _updatedMembers, uint256 _nonce)
    32:     external
    33:     returns (bool res)

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol

    File: L2SecurityCouncilMgmtFactory.sol	

    81: function deploy(DeployParams memory dp, ContractImplementations memory impls)
    82:     external
    83:     onlyOwner
    84:     returns (DeployedContracts memory)

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized("Naman");
            c1.optimized("Naman");
        }
    }
    
    contract Contract0 {
    
         function not_optimized(string memory a) public returns(string memory){
             return a;
         }
    }
    
    contract Contract1 {
    
         function optimized(string calldata a) public returns(string calldata){
             return a;
         }
    }

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 100747                                    | 535             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 790             | 790 | 790    | 790 | 1       |

| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 66917                                     | 366             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 556             | 556 | 556    | 556 | 1       |

## [G-02] Variables that are set only at time of construction should be set as immutable

The variables that are only set at the time of construction of contract should be set as immutable to save gas cost.

There are 5 instances of this issue 4 files:

    File: SecurityCouncilManager.sol	

    115: l2CoreGovTimelock = _l2CoreGovTimelock;

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol

    File: SecurityCouncilNomineeElectionGovernor.sol	

    114: nomineeVetter = params.nomineeVetter;

    248: nomineeVetter = _nomineeVetter;

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

    File: UpgradeExecRouteBuilder.sol	

    87: l1TimelockAddr = _l1ArbitrumTimelock;

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol

    File: GovernanceChainSCMgmtActivationAction.sol	

    44: securityCouncilManager = _securityCouncilManager;

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol

#### Test Contract

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
    
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
    
        function testGas() public {
            c0.getNum();
            c1.getNum();
        }
    }
    
    contract Contract0 {
        uint256 num;
    
        constructor(){
            num = 10;
        }
    
        function getNum() public returns(uint256){
            return num;
        }
    }
    
    contract Contract1 {
        uint256 immutable num;
    
        constructor(){
            num = 10;
        }
    
        function getNum() public returns(uint256){
            return num;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 46181                                     | 154             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| getNum                                    | 2246            | 2246 | 2246   | 2246 | 1       |

| Contract1 contract                        |                 |     |        |     |         |
|----------------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                                    | Deployment Size |     |        |     |         |
| 30108                                              | 195             |     |        |     |         |
| Function Name                                      | min             | avg | median | max | # calls |
| getNum                                             | 146             | 146 | 146    | 146 | 1       |

## [G-03] Using XOR (^) and AND (&) bitwise equivalents

Given 4 variables a, b, c and d represented as such:

    0 0 0 0 0 1 1 0 <- a
    0 1 1 0 0 1 1 0 <- b
    0 0 0 0 0 0 0 0 <- c
    1 1 1 1 1 1 1 1 <- d

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.

There are 2 instances of this issue in 1 file:

    File: SecurityCouncilManager.sol	

    222: if (_addressToRemove == address(0) || _addressToAdd == address(0)) {

    236: if (
    237:     _securityCouncilData.updateAction == address(0)
    238:         || _securityCouncilData.securityCouncil == address(0)
    239: ) {

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(1,2);
            c1.optimized(1,2);
        }
    }
    
    contract Contract0 {
        function not_optimized(uint8 a,uint8 b) public returns(bool){
            return ((a==1) || (b==1));
        }
    }
    
    contract Contract1 {
        function optimized(uint8 a,uint8 b) public returns(bool){
            return ((a ^ 1) & (b ^ 1)) == 0;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 46099                                     | 261             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 456             | 456 | 456    | 456 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 42493                                     | 243             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 430             | 430 | 430    | 430 | 1       |

## [G-04] *<x> += <y>* costs more gas than *<x> = <x> + <y>* for state variables

Using the Addition operator instead of plus-equals is gas efficient. 

There are 2 instances of this issue in 1 file:

    File: SecurityCouncilNomineeElectionGovernorTiming.sol	 

    79: month += 6 * electionIndex;

    84: month += 1;

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilNomineeElectionGovernorTiming.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;

        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }

        function testGas() public {
            c0.add();
            c1.add1();
        }
    }

    contract Contract0 {

        uint8 num1 = 1;

        function add() public{
            num1 += 1;
        }

    }

    contract Contract1 {

        uint8 num1 = 1;

        function add1() public{
            num1 = num1 + 1;
        }

    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 67017                                     | 268             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| add                                       | 5405            | 5405 | 5405   | 5405 | 1       |

| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 70623                                     | 286             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| add1                                      | 5363            | 5363 | 5363   | 5363 | 1       |

## [G-06] Instead of counting down in for statements, count up

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

There are 28 instances of this issue in 7 files:

    File: SecurityCouncilManager.sol	

    106: for (uint256 i = 0; i < _roles.memberRemovers.length; i++) {

    118: for (uint256 i = 0; i < _securityCouncils.length; i++) {

    135: for (uint256 i = 0; i < _newCohort.length; i++) {

    162: for (uint256 i = 0; i < 2; i++) {

    164: for (uint256 j = 0; j < cohort.length; j++) {

    251: for (uint256 i = 0; i < securityCouncils.length; i++) {

    284: for (uint256 i = 0; i < securityCouncils.length; i++) {

    339: for (uint256 i = 0; i < firstCohort.length; i++) {

    342: for (uint256 i = 0; i < secondCohort.length; i++) {

    392: for (uint256 i = 0; i < securityCouncils.length; i++) {

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol

    File: SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol	

    181: for (uint256 i = 0; i < nominees.length; i++) {

    205: for (uint16 i = 0; i < nominees.length; i++) {

    215: for (uint16 i = 0; i < k; i++) {

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

    File: UpgradeExecRouteBuilder.sol	

    76: for (uint256 i = 0; i < _upgradeExecutors.length; i++) {

    129: for (uint256 i = 0; i < chainIds.length; i++) {

    193: for (uint256 i = 0; i < chainIds.length; i++) {

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol

    File: SecurityCouncilMemberSyncAction.sol	

    60: for (uint256 i = 0; i < _updatedMembers.length; i++) {

    67: for (uint256 i = 0; i < previousOwners.length; i++) {

    105: for (uint256 i = 0; i < owners.length; i++) {

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol

    File: SecurityCouncilMgmtUtils.sol		

    6: for (uint256 i = 0; i < arr.length; i++) {

    22: for (uint256 i = 0; i < input.length; i++) {

    31: for (uint256 i = 0; i < intermediateLength; i++) {

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol

    File: L2SecurityCouncilMgmtFactory.sol	

    111: for (uint256 i = 0; i < dp.firstCohort.length; i++) {

    117: for (uint256 i = 0; i < dp.secondCohort.length; i++) {

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol

    File: SecurityCouncilMgmtUpgradeLib.sol	

    61: for (uint256 i = 0; i < array1.length; i++) {

    63: for (uint256 j = 0; j < array2.length; j++) {

    74: for (uint256 i = 0; i < array2.length; i++) {

    76: for (uint256 j = 0; j < array1.length; j++) {

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.AddNum();
            c1.AddNum();
        }
    }
    
    contract Contract0 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=0;i<=9;i++){
                _num = _num +1;
            }
            num = _num;
        }
    }
    
    contract Contract1 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=9;i>=0;i--){
                _num = _num +1;
            }
            num = _num;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 77011                                     | 311             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 7040            | 7040 | 7040   | 7040 | 1       |

| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 73811                                     | 295             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 3819            | 3819 | 3819   | 3819 | 1       |