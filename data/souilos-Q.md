
# VULN 1 

## [LOW] Keccak Constant values should used to immutable rather than constant
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 79 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

    bytes32 public constant COHORT_REPLACER_ROLE = keccak256("COHORT_REPLACER");


Found in line 80 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

    bytes32 public constant MEMBER_ADDER_ROLE = keccak256("MEMBER_ADDER");


Found in line 81 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

    bytes32 public constant MEMBER_REPLACER_ROLE = keccak256("MEMBER_REPLACER");


Found in line 82 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

    bytes32 public constant MEMBER_ROTATOR_ROLE = keccak256("MEMBER_ROTATOR");


Found in line 83 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

    bytes32 public constant MEMBER_REMOVER_ROLE = keccak256("MEMBER_REMOVER");

------------------------------------------------------------------------ 

### Mitigation 

There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts. While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand.










# VULN 2 

## [LOW] Use Ownable2Step rather than Ownable
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 26 at 2023-08-arbitrum/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol:

    OwnableUpgradeable


Found in line 23 at 2023-08-arbitrum/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol:

    OwnableUpgradeable,


Found in line 23 at 2023-08-arbitrum/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol:

    OwnableUpgradeable,

------------------------------------------------------------------------ 

### Mitigation 

Ownable2Step and Ownable2StepUpgradeable prevent the contract ownership from mistakenly being transferred to an address that cannot handle it (e.g. due to a typo in the address), by requiring that the recipient of the owner permissions actively accept via a contract call of its own.










# VULN 3 

## [LOW] Keccak Constant values should used to immutable rather than constant
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 79 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

    bytes32 public constant COHORT_REPLACER_ROLE = keccak256("COHORT_REPLACER");


Found in line 80 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

    bytes32 public constant MEMBER_ADDER_ROLE = keccak256("MEMBER_ADDER");


Found in line 81 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

    bytes32 public constant MEMBER_REPLACER_ROLE = keccak256("MEMBER_REPLACER");


Found in line 82 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

    bytes32 public constant MEMBER_ROTATOR_ROLE = keccak256("MEMBER_ROTATOR");


Found in line 83 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

    bytes32 public constant MEMBER_REMOVER_ROLE = keccak256("MEMBER_REMOVER");

------------------------------------------------------------------------ 

### Mitigation 

By using immutable variables, the Keccak256 hash is computed at contract deployment time rather than at compilation time. This means that the value can be updated if the algorithm changes in a future compiler version, without breaking backward compatibility. Additionally, it provides better gas optimization, as the Keccak256 hash is computed only once at contract deployment instead of every time the variable is accessed during execution.










# VULN 4 

## [LOW] Immutables should be in uppercase
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 57 at 2023-08-arbitrum/src/UpgradeExecRouteBuilder.sol:

    address public immutable l1TimelockAddr;


Found in line 60 at 2023-08-arbitrum/src/UpgradeExecRouteBuilder.sol:

    uint256 public immutable l1TimelockMinDelay;


Found in line 67 at 2023-08-arbitrum/src/security-council-mgmt/SecurityCouncilManager.sol:

    uint256 public immutable MAX_SECURITY_COUNCILS = 500;


Found in line 13 at 2023-08-arbitrum/src/gov-actions-contracts/execution-record/ActionExecutionRecord.sol:

    KeyValueStore public immutable store;


Found in line 16 at 2023-08-arbitrum/src/gov-actions-contracts/execution-record/ActionExecutionRecord.sol:

    bytes32 public immutable actionContractId;


Found in line 13 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol:

    IGnosisSafe public immutable newEmergencySecurityCouncil;


Found in line 14 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol:

    IGnosisSafe public immutable newNonEmergencySecurityCouncil;


Found in line 16 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol:

    IGnosisSafe public immutable prevEmergencySecurityCouncil;


Found in line 17 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol:

    IGnosisSafe public immutable prevNonEmergencySecurityCouncil;


Found in line 19 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol:

    uint256 public immutable emergencySecurityCouncilThreshold;


Found in line 20 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol:

    uint256 public immutable nonEmergencySecurityCouncilThreshold;


Found in line 22 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol:

    address public immutable securityCouncilManager;


Found in line 23 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol:

    IL2AddressRegistry public immutable l2AddressRegistry;


Found in line 10 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/L1SCMgmtActivationAction.sol:

    IGnosisSafe public immutable newEmergencySecurityCouncil;


Found in line 11 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/L1SCMgmtActivationAction.sol:

    IGnosisSafe public immutable prevEmergencySecurityCouncil;


Found in line 12 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/L1SCMgmtActivationAction.sol:

    uint256 public immutable emergencySecurityCouncilThreshold;


Found in line 13 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/L1SCMgmtActivationAction.sol:

    IUpgradeExecutor public immutable l1UpgradeExecutor;


Found in line 14 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/L1SCMgmtActivationAction.sol:

    ICoreTimelock public immutable l1Timelock;


Found in line 8 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/NonGovernanceChainSCMgmtActivationAction.sol:

    IGnosisSafe public immutable newEmergencySecurityCouncil;


Found in line 9 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/NonGovernanceChainSCMgmtActivationAction.sol:

    IGnosisSafe public immutable prevEmergencySecurityCouncil;


Found in line 10 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/NonGovernanceChainSCMgmtActivationAction.sol:

    uint256 public immutable emergencySecurityCouncilThreshold;


Found in line 11 at 2023-08-arbitrum/src/gov-actions-contracts/AIPs/SecurityCouncilMgmt/NonGovernanceChainSCMgmtActivationAction.sol:

    IUpgradeExecutor public immutable upgradeExecutor;

------------------------------------------------------------------------ 

### Mitigation 

Immutables should be in uppercase, it helps to distinguish immutables from other types of variables and provides better code readability.
