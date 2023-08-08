# Analysis Report  


## Executive Summary
This audit was initiated to ensure the election's implementation aligns with the Constitution and maintains governance security without granting undue power or compromising voting integrity.

-----

## Acknowledgment
Thanks so much to the development team for allowing me to audit this critical implementation. Your transparency, cooperation, and dedication to upholding the highest standards have made this process both insightful and constructive. I am but one person, trying to help make web3 more secure.

Despite many working parts, I found the entire system to be robust and extremely well thought out. The team has considered a wide variety of potential corner cases and has implemented security at every level while staying true to the Constitution to which it adheres.

-----

## Approach
Understanding the governance lifecycle flow of Arbitrum 

Having read the [Arbitrum DAO’s Constitution](https://docs.arbitrum.foundation/dao-constitution), listened to the development team on `Twitter spaces`, and read [The Comprehensive Guide](https://medium.com/code4rena/the-security-council-elections-within-the-arbitrum-dao-a-comprehensive-guide-aa6d001aae60), I’m able to grasp their primary concerns:
- The implementation should not deviate from the Constitution.
- Maintain existing Governance security with new elections.
- Restrict election contract powers to the security council and DAO.
- Limit election contract upgrade powers to the DAO and Security councils.
- Allow any party to manage elections according to the set schedule.
- Reserve election process changes exclusively for the DAO or Security Council.
- Prevent double-counting votes in governors, adhering to the Constitution's schedule.

Per Twitter spaces.
- Main concern is, “it touches the upgrade executor.”
- Consider “race conditions” of other upgrades 
- Via `SecurityCouncilManager:` 21-22: “Care must be taken in the timing of these updates to avoid race conditions, as well as to avoid `invalidating` other operations”

-----

## Audit concerns
- My initial concern is both the audit and implementation timeline. 
- While seven days is certainly ample time in most cases, the way in which these contracts interact with the broader ecosystem may require a deeper understanding. Given the hopes of an election starting on the 15th of September, the timeline seems ambitious. Moreover if critical/high risk vulnerabilities are found, a delay to adequately fix them would most certainly be required. 
And finally, the most recent audit by [Trail of Bits](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/audits/trail_of_bits_aips1-1and1-2__5_6_2023.pdf), while different in scope (an assessment of AIP 1.1 and AIP 1.2”) maintains the protocol is getting thoroughly reviewed at all steps of deployment. 

-----

## Constitutional concerns/deviations
Immediate Execution
- As stated above with the given timeline. The Constitution states: “At T+28 days: The 6 candidates who have received the most votes are elected and immediately join the Council, replacing the Cohort that was up for re-election.” They have to go through time locks, L2 to L1 message transfers, and other processes before they can update the Council membership. So the timing of the implementation may not adhere to the specifics, but it all seems within good technical reasons.

Compliance Checks by the Foundation:
- The proposed implementation introduces a 14-day compliance check stage between `Nominee Selection` and `Member Election.` While the Constitution gives an affordance for the Arbitrum Foundation to conduct a compliance check, it introduces some centralization concerns. But this is known by all entities and doesn’t seem to be a major issue.

And finally, an initial thought, the phrase "except in a true security emergency, such as a critical vulnerability that could significantly compromise the integrity, confidentiality, or availability of a chain governed by the ArbitrumDAO," seems somewhat vague in terms of `integrity, confidentiality, or availability` and could be interpreted differently in certain cases. Without specific criteria for what constitutes an emergency, it may be challenging for the community to hold the Security Council accountable for its decisions to invoke emergency powers. (This is my opinion; I'm obviously not going to change the constitution.)

-----

## Manual review
The code quality adheres to quality standards which made the architecture easy to follow. Upon first review, no glaring issues stood out to me.

There is a lot to digest, and my goal was to gain a mid-high level understanding of all the working parts. 
- Knowing that, and given the finite time, there were some `addresses` I assumed would simply work: `RETRYABLE_TICKET_MAGIC` etc. 
- The hardcoded `address(100)` in the `UpgradeExecRouteBuilder` did confuse me however, is that an address or a timelock mechanism? Perhaps a comment there. (Later discovering this is the precompile address for [ArbSys](https://github.com/OffchainLabs/nitro-contracts/blob/97cfbe00ff0eea4d7f5f5f3afb01598c19ddabc4/src/precompiles/ArbSys.sol))
- I am currently unaware of how the `multisig wallet` functionality works, as it's out of scope.

Additionally, I understand the `timelock` mechanisms, but the lack of any `reentrancy guard` piqued my interest as a first potential vulnerability. For example if an arbitrary call can reenter the `TimelockControllerUpgradeable`, it could lead to some permission escalations. There is a reentrancy guard in the `UpgradeExecutor`.

It's clear the lack of reentrancy guards are due to the quality access control and limited access of functions to verified users. However if a malicious actor were to gain access, there may be other areas of reentrancy concern given the cascading effect of privilege escalation. Overall, it seems if a malicious actor gains control of the `UpgradeExecutor` as `EXECUTOR_ROLE` the entire system is compromised. 

Albeit slim, another initial thought of mine would be a malicious actor gaining access via OpenZeppelin’s upgradeable contracts as OpenZeppelin contracts tend to be assumed as well vetted by most, if not all auditors. 

And finally, while a malicious actor gaining control is the highest concern, invalidating operations is also a high concern knowing it could similalrly lead to cascading failures. 

-----

## Organizational Upgrades

Overall it would be helpful to add more detailed top-level comments or `NatSpec` explaining a library's purposes and how each fits into the system. This would help auditors get up to speed quickly and upgrade the efficiency of time.

Additionally, some contracts, like [SecurityCouncilMgmtUpgradeLib.sol](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol) for example, lack `custom errors` and given the potential `gas savings` it may be best to create a `utils` file in the `src folder`. Creating a file of custom errors can then be imported into each contract. For example: 
```
import "./utils/Errors.sol";
```
```
Errors.sol
pragma solidity ^0.8.19;

error IsElected();
error NotElected();
error NotEnoughTime();
// etc…
```

-----

## Testing
The testing coverage seems quite thorough and extensive.
- I reviewed the `SecurityCouncilManagerTest` which has full coverage and. 
- I'm unsure if tests have been run on a testnet...
- Would like to see some sort of Foundry `warping` to simulate transactions that occur at the same time or in rapid succession.

-----

## Low-risk and non-critical findings 

Using both Slither as a static analysis tool and manual review, I found the following:

1. A warning in the `TimelockControllerUpgradeable.executeBatch` via [openzeppelin-contracts-upgradeable](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/governance/extensions/GovernorTimelockControlUpgradeable.sol).
```
 * WARNING: Setting up the TimelockController to have additional proposers besides the governor is very risky, as it
 * grants them powers that they must be trusted or known not to use: 1) {onlyGovernance} functions like {relay} are
 * available to them through the timelock, and 2) approved governance proposals can be blocked by them, effectively
 * executing a Denial of Service attack. This risk will be mitigated in a future release.
```
This is known and as stated hopefully mitigated in a future release.

2. Shadow variable in [SecurityCouncilManager.sol](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol)   
The parameter ` _roles` in the initialize function shadows the `_roles` state variable defined in the AccessControlUpgradeable contract. [Slither Reference](https://github.com/crytic/slither/wiki/Detector-Documentation#local-variable-shadowing) 
```
93:     SecurityCouncilManagerRoles memory _roles,
```
```
61:     mapping(bytes32 => RoleData) private _roles;
```
3. Calls within a loop.   
Several contracts use external calls in a loop which may lead to a denial-of-service attack (which may be the largest security threat). [Slither Reference](https://github.com/crytic/slither/wiki/Detector-Documentation/#calls-inside-a-loop)

[SecurityCouncilManager.sol](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol)
#231-268 –  #247 `!router.upExecLocationExists`
```
function _addSecurityCouncil(SecurityCouncilData memory _securityCouncilData) internal {
if (securityCouncils.length == MAX_SECURITY_COUNCILS) {
revert MaxSecurityCouncils(securityCouncils.length);
}

if (
_securityCouncilData.updateAction == address(0)
|| _securityCouncilData.securityCouncil == address(0)
) {
revert ZeroAddress();
}

if (_securityCouncilData.chainId == 0) {
revert SecurityCouncilZeroChainID(_securityCouncilData);
}

if (!router.upExecLocationExists(_securityCouncilData.chainId)) {
revert SecurityCouncilNotInRouter(_securityCouncilData);
}

for (uint256 i = 0; i < securityCouncils.length; i++) {
SecurityCouncilData storage existantSecurityCouncil = securityCouncils[i];

if (
existantSecurityCouncil.chainId == _securityCouncilData.chainId
&& existantSecurityCouncil.securityCouncil == _securityCouncilData.securityCouncil
) {
revert SecurityCouncilAlreadyInRouter(_securityCouncilData);
}
}

securityCouncils.push(_securityCouncilData);
emit SecurityCouncilAdded(
_securityCouncilData.securityCouncil,
_securityCouncilData.updateAction,
securityCouncils.length
);
}

```
[SecurityCouncilMemberSyncAction.sol](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol) #31-74 – #62 `securityCouncil.isOwner`
```
function perform(address _securityCouncil, address[] memory _updatedMembers, uint256 _nonce)
external
returns (bool res)
{
// make sure that _nonce is greater than the last nonce
// we do this to ensure that a previous update does not occur after a later one
// the mechanism just checks greater, not n+1, because the Security Council Manager always
// sends the latest full list of members so it doesn't matter if some updates are missed
// Additionally a retryable ticket could be used to execute the update, and since tickets
// expire if not executed after some time, then allowing updates to be skipped means that the
// system will not be blocked if a retryable ticket is expires
uint256 updateNonce = getUpdateNonce(_securityCouncil);
if (_nonce <= updateNonce) {
// when nonce is too now, we simply return, we don't revert.
// this way an out of date update will actual execute, rather than remaining in an unexecuted state forever
emit UpdateNonceTooLow(_securityCouncil, updateNonce, _nonce);
return false;
}

// store the nonce as a record of execution
// use security council as the key to ensure that updates to different security councils are kept separate
_setUpdateNonce(_securityCouncil, _nonce);

IGnosisSafe securityCouncil = IGnosisSafe(_securityCouncil);
// preserve current threshold, the safe ensures that the threshold is never lower than the member count
uint256 threshold = securityCouncil.getThreshold();

address[] memory previousOwners = securityCouncil.getOwners();

for (uint256 i = 0; i < _updatedMembers.length; i++) {
address member = _updatedMembers[i];
if (!securityCouncil.isOwner(member)) {
_addMember(securityCouncil, member, threshold);
}
}

for (uint256 i = 0; i < previousOwners.length; i++) {
address owner = previousOwners[i];
if (!SecurityCouncilMgmtUtils.isInArray(owner, _updatedMembers)) {
_removeMember(securityCouncil, owner, threshold);
}
}
return true;
}
```
And #129-131 `!securityCouncil.execTransactionFromModule`
```
function _execFromModule(IGnosisSafe securityCouncil, bytes memory data) internal {
if (
!securityCouncil.execTransactionFromModule(
address(securityCouncil), 0, data, OpEnum.Operation.Call
)
) {
revert ExecFromModuleError({data: data, securityCouncil: address(securityCouncil)});
}
```
And #97-116 #103 `owners = securityCouncil.getOwners`
```
function getPrevOwner(IGnosisSafe securityCouncil, address _owner)
public
view
returns (address)
{
// owners are stored as a linked list and removal requires the previous owner
address[] memory owners = securityCouncil.getOwners();
address previousOwner = SENTINEL_OWNERS;
for (uint256 i = 0; i < owners.length; i++) {
address currentOwner = owners[i];
if (currentOwner == _owner) {
return previousOwner;
}
previousOwner = currentOwner;
}
revert PreviousOwnerNotFound({
targetOwner: _owner,
securityCouncil: address(securityCouncil)
});
}
```

-----

## Threat Model

- If `high or medium` vulnerabilites are found, they will be submitted separately.

#### Overview
 - Because of [ArbitrumGovernance’s](https://github.com/ArbitrumFoundation/governance/blob/main/docs/overview.md) complexity, a successful attack would most likely involve a `mult-step process` as executing a passed proposal takes a number of transactions.
 - Furthermore, in an election system, gaining votes to control the council is the primary concern, and would most likely involve manipulating the `timing of transactions` related to the `Security Council Manager`. 
- Per `bytes032,` they have included a [diagram via Twitter](https://twitter.com/bytes032/status/1688566753444429826?s=43&t=ezf5V_RX8d4d4zdIpUUrWQ) of the election timeline process
- The potential attacks below are listed from, what I believe to be, most likely to least likely. 

`Race conditions`
- A governance proposal and a Security Council action could both try to upgrade the system at the same time, conflicting with each other.
- An election result could come in while a governance proposal to remove a Security Council member is in progress.
- Upgrades coming from different chains could arrive out of order and conflict.
- Potential block conflicts: block stuffing, replay attacks involving the nonce, transaction-ordering dependence.
- As of `2023-08-07` there are no relevant `race condition` reports on [Solodit](https://solodit.xyz/auth)

`Access Control`
- Given the complexity of deploying and initializing multiple proxy contracts and the use of low level calls, this is a concern. But nothing apparent can be found on my end; address checks seem to be in place correctly.

`Uninitialized Proxy`
- Using proxy contracts without proper initialization can introduce vulnerabilities, as uninitialized storage variables can be manipulated. Attackers can exploit these vulnerabilities to gain unauthorized access or execute unintended actions.
- See [[L‑03] Upgradeable contract not initialized](https://github.com/code-423n4/2023-08-arbitrum/blob/main/bot-report.md#l03-upgradeable-contract-not-initialized)

`Front-Running`
- Actions like adding or replacing members in the cohort could be subject to front-running, where an attacker observes a pending transaction and attempts to preempt it with their own, potentially leading to unexpected behavior.
- See [Initialization can be front-run](https://github.com/code-423n4/2023-08-arbitrum/blob/main/bot-report.md#l07-initialization-can-be-front-run)

`DDoS`
- Attacks targeting the Arbitrum network or nodes during the election could disrupt the process. Additionally if a malicious actor were to somehow gain access to an intensive loop function, this could delay certain timelocks. But the likelihood of this seems extremely low.

-----

## Relevant Solodit Findings

The uniqueness of the Arbitrum Dao's voting mechanism makes finding similar audits a bit difficult. However, and although different, these are some I thought relevent and have thus included:
- [DOS by Frontrunning NoteERC20 initialize() Function](https://solodit.xyz/issues/h-08-dos-by-frontrunning-noteerc20-initialize-function-code4rena-notional-notional-contest-git)   
- [The total community voting power is updated incorrectly when a user delegates](https://solodit.xyz/issues/h-1-the-total-community-voting-power-is-updated-incorrectly-when-a-user-delegates-sherlock-3d-frankenpunks-frankendao-git)
- [Decrementing the quorum in Oracle in some scenarios can open up a frontrunning/backrunning](https://solodit.xyz/issues/decrementing-the-quorum-in-oracle-in-some-scenarios-can-open-up-a-frontrunningbackrunning-spearbit-liquid-collective-pdf)
- [Unable to use L2GatewayRouter to withdraw LPT from L2 to L1, as L2LPTGateway does not implement L2GatewayRouter expected method](https://solodit.xyz/issuesm-07-wp-m4-unable-to-use-l2gatewayrouter-to-withdraw-lpt-from-l2-to-l1-as-l2lptgateway-does-not-implement-l2gatewayrouter-expected-method-code4rena-livepeer-livepeer-contest-git)

-----

## Spelling and informational
[SecurityCouncilManager.sol](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol)
```
72:     /// @notice Size of cohort under ordinary circumstancces (circumstances)
```

[SecurityCouncilMemberRemovalGovernor.sol](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol)
```
101:     ///         but enforces that only calls to securityCouncilManager's removeMember can be propsoed (proposed)
```

[ArbitrumGovernorVotesQuorumFractionUpgradeable.sol](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/ArbitrumGovernorVotesQuorumFractionUpgradeable.sol)
```
16:     ///         tokens will still count towards the total supply, and will therefore affect the quorom. (quorum)

```
[UpgradeExecRouteBuilder.sol](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol)
```
40:     error UpgadeExecDoesntExist(uint256 chainId); (Upgrade)
```

Additionally, there may be a redundant check.    
When iterating through `_upgradeExecutors` in the `constructor` the code checks if an `upgradeExecutor` for a chainId already exists, then sets it. It might be more efficient to skip the existence check and directly overwrite the value. This is more of a design choice, but overwriting could simplify the code and be more gas efficient.   

Magic Number   
`address(100)` is hard-coded. On initial inspection I am unsure why? A comment would be good to better explain.
```
175:          address(100),
```

[SecurityCouncilMemberSyncAction.sol](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol)
```
41:         // system will not be blocked if a retryable ticket is expires (expired)
44:         // when nonce is too now, we simply return, we don't revert. (two or too low?)
```

[Common.sol](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/Common.sol)
```
6:	 /// and the are members replaced with new ones. (^the old members are)
```

[GovernanceChainSCMgmtActivationAction.sol](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/GovernanceChainSCMgmtActivationAction.sol)    
Sentence is confusing, perhaps:    
”Revoke the old security council's cancel role. The new council doesn't need it explicitly granted since they can cancel through the core governor's relay method.”
```
103:         // revoke old security council cancel role; it is unnecessary to grant it to explicitly grant it to new security council since the security council can already cancel via the core governor's relay method.

```

[L1SCMgmtActivationAction.sol](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/L1SCMgmtActivationAction.sol)
```
9: counicl (council) 
```
[ActionExecutionRecord.sol](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/execution-record/ActionExecutionRecord.sol)
```
8-9: it (x2)
```

-----

## Conclusion

The system exhibits a strong focus on security and demonstrates a commendable alignment with the Constitution within the constraints of technical feasibility. The robustness of the `Access Control and Permissions` is evident, contributing to the integrity of the framework. In the course of my audit, I identified no immediate security threats or critical vulnerabilities. As I believe the likelihood of a successful attack would involve a multi-step process and would require more time and understanding to execute. 

Given the complexity of the system and the ambitious timeline, it might be of interest to extend testing once the audit findings are revealed. Running the implementation on a testnet could provide additional confidence and allow for a thorough evaluation of potential edge cases: simultaneous transactions, front-running, etc.

Overall, the development team's dedication to upholding the democratic principles at the core of Arbitrum Dao's ecosystem is reassuring. Their commitment to transparency and strict adherence to the Constitution lays a solid foundation for trust, transparency and collaboration.

### Time spent:
20 hours