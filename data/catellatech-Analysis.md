#  Analysis - Arbitrum Security Council Election System Contest
<img decoding="async" loading="lazy" src="https://arbitrum.io/wp-content/uploads/2023/03/ARB_lockup_white-2-1024x261.png" alt="" class="wp-image-1023" width="461" height="81"  srcset="https://arbitrum.io/wp-content/uploads/2023/03/ARB_lockup_white-2-1024x261.png 1024w, https://arbitrum.io/wp-content/uploads/2023/03/ARB_lockup_white-2-300x76.png 300w, https://arbitrum.io/wp-content/uploads/2023/03/ARB_lockup_white-2-768x196.png 768w, https://arbitrum.io/wp-content/uploads/2023/03/ARB_lockup_white-2-1536x391.png 1536w, https://arbitrum.io/wp-content/uploads/2023/03/ARB_lockup_white-2-2048x522.png 2048w" sizes="(max-width: 161px) 100vw, 161px">

## Summary

Arbitrum is a set of solutions designed to accelerate and make applications in blockchain more cost-effective. This is achieved by processing multiple transactions at once without incurring high costs. These solutions are based on Ethereum technology, a well-known blockchain platform.

Within the Arbitrum environment, there's a group called the **"Arbitrum DAO"** that makes decisions affecting two chains named **"Arbitrum One"** and **"Nova"**. These decisions are crucial to ensure the security and proper functioning of these chains:
When a [new L2 chain is authorized by the DAO](https://docs.arbitrum.foundation/new-arb-chains), the following steps should be carried out for the new chain to become DAO-governed:
1. Deploy a new UpgradeExecutor contract and a new Security Council on the new L2 chain.
1. Initialize the new L2 UpgradeExectutor with the L1 Timelock's aliased addressed and the new Security Council as its executors.
1. Ownership transfer: for a chain deployed whose contract deployment mirrors that of Arbitrum One and Arbitrum Nova (i.e, [Nitro](https://github.com/OffchainLabs/nitro) core contracts and [token bridge contracts](https://github.com/OffchainLabs/token-bridge-contracts)), the following ownership transfer should take place:
     - The L1 Upgrade Executor should be granted the following affordances:
        - L1 core contract Proxy Admin owner
        - L1 token bridge Proxy Admin owner
        - Rollup Admin owner
        - L1 Gateway Router owner 
        - L1 Arb Custom Gateway Owner
    - The new L2 Upgrade Executor should be granted the following affordances:
        - L2 token bridge Proxy Admin Owner 
        - Chain Owner
        - Standard Arb-ERC20 Beacon Proxy owner

In addition, there's the **"Security Council"**, consisting of 12 individuals from various organizations. This council takes on the responsibility of making quick decisions in the event of a significant security issue arising in the Arbitrum chains. They have the authority to implement urgent actions and resolve such problems. This council is divided into two groups, each of which is elected alternately every six months.

The process of selecting these groups is referred to as "elections" and is governed by a system of smart contracts on the chain. Terms like "contenders," "nominees," and "members" are used to describe the different stages and roles within this process.

![Arbitrum](https://github.com/catellaTech/unknow/blob/main/DiagramaZero.drawio.png?raw=true)

### **Scope**
   - The engagement involved auditing twenty different Solidity Smart Contracts:

      - **SecurityCouncilManager.sol**: Serves as the definitive source of the current status for all security councils. It initiates updates across different chains and collaborates with the UpgradeExecRouteBuilder to craft the necessary payload.

      - **SecurityCouncilNomineeElectionGovernor.sol**: Empowers delegates to vote and select nominees from a broader pool of contenders. This module introduces a vetting period, allowing a trusted entity to include or exclude contenders based on their credibility.

      - **SecurityCouncilNomineeElectionGovernorCountingUpgradeable.sol**: This module handles the vote counting mechanism within the nominee governor system.

      - **SecurityCouncilNomineeElectionGovernorTiming.sol**: Provides timing-related functionalities for the nominee election governor module.

      - **SecurityCouncilMemberElectionGovernor.sol**: Enables delegates to vote for 6 candidates from a shortlist of nominees.

      - **SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol**: Manages the vote counting process within the member election governor module.

      - **SecurityCouncilMemberRemovalGovernor.sol**: Facilitates the voting process for the removal of a candidate from the existing security council.

      - **ArbitrumGovernorVotesQuorumFractionUpgradeable.sol**: Core functionality focused on counting only "votable" tokens.

      - **ElectionGovernor.sol**: Provides shared functionalities that are utilized by various election governors.

      - **UpgradeExecRouteBuilder.sol**: Constructs routes that target the upgrade executors on each respective chain.

      - **SecurityCouncilMemberSyncAction.sol**: An action contract responsible for updating the member list of the Gnosis Safe.

      - **SecurityCouncilMgmtUtils.sol**: Houses shared array utilities to enhance code efficiency.

      - **Common.sol**: Contains shared data structures that are widely used.

      - **L2SecurityCouncilMgmtFactory.sol**: Deploys contracts related to the election process.

      - **GovernanceChainSCMgmtActivationAction.sol**: Activates the election process on Arbitrum One.

      - **L1SCMgmtActivationAction.sol**: Initiates the election process on the L1 Ethereum chain.

      - **NonGovernanceChainSCMgmtActivationAction.sol**: Activates the election process on Arbitrum Nova.

      - **SecurityCouncilMgmtUpgradeLib.sol**: Contains shared utilities to streamline the management process.

      - **KeyValueStore.sol**: Functions as a repository for values linked to specific keys, serving as external storage to prevent actions from directly utilizing state.

      - **ActionExecutionRecord.sol**: Stores a record of executed actions for future reference.

## Approach of the code base 

During the analysis, our approach it was to identify key contracts and their interactions within the Security Council management system. To determine which contract was the main one, we examined the hierarchy and responsibilities of the contracts provided. The main contract is the "Security Council Manager (SCM)," as it appears to be the central core of the system that coordinates and manages various governance functions related to the Security Council.

Starting from that main contract, we constructed to show how other contracts interact with it and contribute to different aspects of the governance process, such as member elections, activation on different chains, and action record management. Each contract plays a specific role and contributes to the overall operation of the system.

![arb](https://github.com/catellaTech/unknow/blob/main/Diagram%204%20arb.drawio.png?raw=true)

Click on the diagram links:
   - [Installation](https://github.com/code-423n4/2023-08-arbitrum#tests)
   - [Arbitrum docs](https://docs.arbitrum.foundation/dao-constitution)
   - [Slither](https://github.com/crytic/slither)
   - [Tests](https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5#run-hardhat-integration-tests)
   - [Scope](https://github.com/code-423n4/2023-08-arbitrum#scope)
   - [draw.io](https://app.diagrams.net/)
   - [Areas of concern](https://github.com/code-423n4/2023-08-arbitrum#areas-of-concern)


## Analysis of the main contract
The central and most critical contracts within the project.

### SecurityCouncilManager:

![Arquic](https://github.com/catellaTech/unknow/blob/main/Diagrama%20sin%20t%C3%ADtulo.drawio.png?raw=true)


## Architecture Description overview

**The system architecture comprises various interconnected smart contracts that collaborate to manage and administer the Security Council's governance process. Here is an overview of the components and their roles within the architecture:**:

![Arquic](https://github.com/catellaTech/unknow/blob/main/Diagrama%203.drawio.png?raw=true)

## Codebase Quality
Overall, we consider the quality of Arbitrum codebase to be excellent. The code appears to be very mature and well-developed. Details are explained below:

| Codebase Quality Categories  | Comments |
| --- | --- |
| **Unit Testing**  | Codebase is well-tested it was great to see the protocol using Forundry framework.|
| **Code Comments**  | Comments in general were solid. However is always room for improvement. Some areas could benefit from greater clarity in comments or explanations. Providing more detailed comments and documentation for complex or critical sections of the code can greatly enhance the codebase's overall readability and maintainability. This would not only help the current development team but also make it easier for future contributors to understand and build upon the existing code. |
| **Documentation** | The documentation for Arbitrum is very good.   |
| **Organization** | Codebase is very mature and well organized with clear distinctions between the 20 contracts. |                                                                                                                                                                                                                              
##  Systemic & Centralization Risks

![systemic](https://github.com/ArbitrumFoundation/governance/raw/c18de53820c505fc459f766c1b224810eaeaabc5/docs/roundtrip-governance.png)

The analysis provided highlights several significant systemic and centralization risks present in the present Arbitrum contest.

1. **dependency risk**:
   - It is observed that old and unsafe versions of Openzeppelin are used in the project, this should be updated to the latest one:

   ```solidity
      package.json#L69-L70
      69: "@openzeppelin/contracts": "4.7.3",
      70: "@openzeppelin/contracts-upgradeable": "4.7.3",
   ```
https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts/4.9.3

2. The following is a list of quirks and/or potentially unexpected behaviors in the Arbitrum Governance implementation. General familiarity with the architecture is assumed:  
   - **Abstain Vote** 
   Voting “abstain” on a core-governor or governor proposal does not count as either a “for” or “against” vote, but does count towards reaching quorum (5% or 3% of votable tokens, respectively).
   - **Timelock vs Governor Execution** 
   An operated queued in the core-governor-timelock or the treasury-governor-timelock can be executed permissionlessly on either its associated governor (typical) or on the timelock itself (atypical). The execution will be the same in either case, but in the later case, the governor’s `ProposalExecuted` event will not be emitted.
   - **L1-to-L2 Message Fees in scheduleBatch**
   When executing a batch of operations on the L1ArbitrumTimelock, if more than one operation creates a retryable ticket, the full `msg.value` value will be forwarded to each one. For the execution to be successful, the `msg.value` should be set to `m `and the L1ArbitrumTimelock should be prefunded with at least `m * n` ETH, where `m` is the max out of the costs of each retryable ticket, and `n` is the number of retryable tickets created.
   - **Two L1 Proxy Admins** 
   There are two L1 proxy admins - one for the governance contracts, once for the governed core Nitro contracts. Note that both proxy admins have the same owner (the DAO), and thus this has no material effect on the DAO's affordances.
   - **Non-excluded L2 Timelock**
   In the both treasury timelock and the DAO treasury can be transfered via treasury gov DAO vote; however, only ARB in the DAO treasury is excluded from the quorum numerator calculation. Thus, the DAO’s ARB should ideally all be stored in the DAO Treasury. (Currently, the sweepReceiver in the TokenDistributor is set to the timelock, not the DAO treasury.)
   - **L2ArbitrumGovernoer onlyGovernance behavior**
   Typically, for a timelocked OZ governror, the `onlyGovernance` modifier ensures a call is made from the timelock; in L2ArbitrumGoverner, the _executor() method is overriden such that `onlyGovernance` enforces a call from the governor contract itself. This ensures calls guarded by `onlyGovernance` go through the full core proposal path, as calls from the governor could only be sent via `relay`. See the code comment on `relay` in [L2ArbitrumGoveror](../src/L2ArbitrumGovernor.sol) for more.
   - **L2 Proposal Cancelation** 
   There are two redundant affordances that the security council can use to cancel proposals in the L2 timelock: relaying through core governor, or using  its `CANCELLER_ROLE` affordance. Additionally, the later affordances is granted directly to the security council, not the `UpgradeExecutor` (this is inconsistent with how `UpgradeExecutors` are generally used elsewhere.)
   - **L1 Proposal Cancelation**
   The Security Council — not the L1 `UpgradeExecutor` — has the affordance to cancel proposals in the L1 timelock, inconsistent with how `UpgradeExecutors` are generally used elsewhere.

   **Test Coverage**: the test coverage provide by Arbitrum is the 94%, however we recomende 100% of the tets coverage.


##  Conclusion

In general, Arbitrum project exhibits an interesting and well-developed architecture we believe the team has done a good job regarding the code. Additionally, it is recommended to improve the documentation and comments in the code to enhance understanding and collaboration among developers and auditors. It is also highly recommended that the team continues to invest in security measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project. 

## Time Spent 
A total of 5 days were spent to cover this audit and fully understand the flow of the contracts.

### Time spent:
24 hours