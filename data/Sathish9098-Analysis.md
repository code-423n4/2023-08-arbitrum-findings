# Analysis - Arbitrum Security Council Election System  
# Summary

| List |Head |Details|
|:--|:----------------|:------|
|1 | Overview of Arbitrum Security Council Election System  | overview of the key components and features of Arbitrum Security Council Election System   |
|2 |Audit approach | Process and steps i followed  |
|3 |Learnings | Learnings from this protocol|
|4 |Possible Systemic Risks | The possible systemic risks based on my analysis |
|5 |Code Commentary | Suggestions for existing code base |
|6 | Centralization risks | Concerns associated with centralized systems |
|7 |Gas Optimizations | Details about my gas optimizations findings and gas savings  |
|8 |Risks as per Analysis | Possible risks |
|9 |Non-functional aspects | General suggestions |
|10 |Time spent on analysis  | The Over all time spend for this reports |

## Overview

``Arbitrum`` is a suite of scaling solutions providing environments with ``high-throughput``, ``low-cost smart contracts``, ``backed by industry-leading`` proving technology rooted in ``Ethereum``.

### ``Arbitrum DAO -  Governs the Arbitrum One and Nova chains``
The Arbitrum DAO is a decentralized autonomous organization (DAO) built on the ``Ethereum blockchain``. At its core, the ``Arbitrum DAO`` is a community-driven governance mechanism that allows ``$ARB`` token holders to propose and ``vote`` on changes to the ``organization`` and the ``technologies`` it governs.

The Arbitrum DAO's governance smart contracts are implemented on the Arbitrum One ``rollup chain``, which is a ``Layer 2`` scaling solution for the ``Ethereum blockchain``. These smart contracts include the DAO's governance token, ``$ARB``. DAO members use ``$ARB`` tokens to vote on ``Arbitrum DAO proposals (AIPs)``. The weight of any given voter's vote is proportional to the amount of ``$ARB`` they hold (or represent)1.

The Arbitrum DAO has a built-in ``treasury system`` (implemented as a smart contract); the ``DAO's treasury`` is used to fund ongoing development and ``maintenance`` of the organization and its technologies. Token holders can propose and vote on how to use the treasury's funds

### ``Security Council ``
This is responsible for making ``emergency response`` decisions when the community needs to address a critical security risk to the protocol. They are a ``12 member`` council from independent organisations that hold the ability to conduct emergency actions to the ``Arbitrum chains``. The ``12 member`` council is separated into ``2 cohorts`` which are elected alternatively every ``6 months``. 

> The main contributers in over all protocols 

#### Contender
Someone who can receive votes in the nominee selection stage but who has received less than ``0.2%`` of votable tokens
    
#### Nominee
Someone who has received ``0.2%`` of votable tokens in the nominee selection stage
``compliant / noncompliant`` indicates whether they’ve been excluded by the ``nominee vetter``.

#### Member
A member of the ``Security council``

 
## Audit approach

I followed below steps while analyzing and auditing the code base.

1. Read the contest Readme.md and took the required notes.

  - Arbitrum Security Council Election System 
    - Inheritance
    - Test Coverage - 94%
    - Use rollups
    - Multi-Chain
    - Timelocks have code associated with them to handle crosschain messages
    - smart contracts deployed in Ethereum and Arbitrum
      
2. Analyzed the over all codebase one iterations very fast

3. Study of documentation to understand each contract purposes, its functionality, how it is connected with other contracts, etc.

4. Then i read old audits and already known findings. Then go through the bot races findings 

5. Then setup my testing environment things. Run the tests to checks all test passed. I used ``foundryup`` and ``make`` to test Arbitrum Security Council Election System . 

#### Commands Used for testing: 

  - make install
  - make build
  - make sc-election-test
  - make coverage, make gas
  
6. Finally, I started with the auditing the code base in depth way I started understanding line by line code and took the necessary notes to ask some questions to sponsors.

## Stages of audit

- ``The first stage of the audit``

During the initial stage of the audit for Arbitrum Security Council Election System, the primary focus was on analyzing gas usage and quality assurance (QA) aspects. This phase of the audit aimed to ensure the efficiency of gas consumption and verify the robustness of the platform.

#### Gas Optimizations
  Found 7 different gas optimizations and totally saved ``88920 GAS``
  
#### QA Analysis
 Found ``10`` ``Low`` risk ``findings ``

- ``The second stage of the audit``

In the second stage of the audit for Arbitrum Security Council Election System, the focus shifted towards understanding the protocol usage in more detail. This involved identifying and analyzing the important contracts and functions within the system. By examining these key components, the audit aimed to gain a comprehensive understanding of the protocol's functionality and potential risks. 

- ``The third stage of the audit``

During the third stage of the audit for Arbitrum Security Council Election System, the focus was on thoroughly examining and marking any doubtful or vulnerable areas within the protocol. This stage involved conducting comprehensive vulnerability assessments and identifying potential weaknesses in the system. Found ``60-70`` ``vulnerable`` and ``weakness`` code parts all marked with ``@audit tags``.

- ``The fourth stage of the audit``

During the fourth stage of the audit for Arbitrum Security Council Election System, a comprehensive analysis and testing of the previously identified doubtful and vulnerable areas were conducted. This stage involved diving deeper into these areas, performing in-depth examinations, and subjecting them to rigorous testing, including fuzzing with various inputs. Finally concluded findings after all research's and tests. Then i reported C4 with proper formats 


## Learnings

As per protocols docs i learned following things 

- ``Decentralized Governance``: The Arbitrum DAO plays a crucial role in governing both the Arbitrum One and Nova chains. This demonstrates a decentralized approach to decision-making and management of the protocol.

- ``Emergency Response with Security Council`` : The Security Council serves as a mechanism for addressing critical security risks promptly. This highlights the importance of having a specialized group with the authority to conduct emergency actions when needed.

- ``Cohort-Based Council`` : The Security Council consists of 12 members from independent organizations, divided into two cohorts. This approach ensures regular alternation of council members, promoting diversity and preventing concentration of power.

- ``Elections and Constitution`` : The elections for the Security Council adhere to a predefined constitution, emphasizing the importance of transparency, fairness, and predictable processes.

- ``On-Chain Governance Implementation`` : The Arbitrum Governance subsystem is implemented using on-chain smart contracts written in Solidity. This ensures that governance processes are transparent, tamper-resistant, and enforceable.

- ``Roles and Terminology`` : The usage of terms like "contender," "nominee," "compliant," "noncompliant," and "member" provides clear distinctions for different stages and statuses within the governance process.


## Possible Systemic Risks

1. ``Operational Clashes``: As noted, there's a possibility of operational clashes between Core Governance and election operations due to overlapping timelocks. This could potentially cause delays or block certain actions if not managed properly
2. ``Security Council Updates``: The Security Council's ability to create updates that might impact election execution could lead to failures if these updates aren't carefully managed. This risk highlights the importance of responsible and thorough decision-making within the Security Council.
3. ``Election Contract Logic Errors``: Errors or vulnerabilities in the election contract logic could lead to unexpected behavior, security breaches, or unintended outcomes during the election process
4. ``Constitution and Code Misalignment``: There's a risk that the implemented code doesn't perfectly align with the Arbitrum Constitution. Such discrepancies could lead to confusion, disputes, or even security vulnerabilities if not promptly addressed.
5. ``Security Council Concentration of Power``: While the Security Council holds the responsibility for emergency actions, there's a potential risk of centralization of power within this group. Ensuring diversity, transparency, and accountability within the council can help mitigate this risk.
6. ``Limited Participation``: The requirement for candidates to have a certain percentage of votable tokens (0.2%) might exclude potential contributors who have valid viewpoints but lack the required threshold, leading to limited representation.

## Centralization risks

A single point of failure is not acceptable for this project Centrality risk is high in the project, the role of ``onlyOwner``detailed below has very critical and important powers

Project and funds may be compromised by a malicious or stolen private key ``onlyOwner`` ``msg.sender``

```solidity

File: src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol

81       function deploy(DeployParams memory dp, ContractImplementations memory impls)
82           external
83           onlyOwner
84           returns (DeployedContracts memory)
85:      {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol#L81-L85

```solidity
File: src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol

103      function relay(address target, uint256 value, bytes calldata data)
104          external
105          virtual
106          override
107          onlyOwner
108:     {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberElectionGovernor.sol#L103-L108

```solidity
File: src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol

178:     function setVoteSuccessNumerator(uint256 _voteSuccessNumerator) public onlyOwner {

203      function relay(address target, uint256 value, bytes calldata data)
204          external
205          virtual
206          override
207          onlyOwner
208:     {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L178-L178

```solidity
File: src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

254      function relay(address target, uint256 value, bytes calldata data)
255          external
256          virtual
257          override
258          onlyOwner
259:     {

```
https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L254-L259


## Gas Optimizations

In the pursuit of ``gas-efficient`` smart contracts, several key optimizations have been identified that can enhance performance and resource utilization. These optimizations focus on improving gas efficiency in various aspects of contract ``design`` and ``execution``.

One prominent optimization involves the strategic use of ``calldata`` over ``memory`` for read-only arguments in external functions. By directly utilizing ``calldata``, the need for a ``loop-driven process``, such as ``abi.decode()``, to copy calldata to memory is circumvented. This results in streamlined contract code and more efficient runtime execution.

Another significant optimization suggests consolidating ``multiple address/ID mappings`` into a single mapping that associates addresses/IDs with corresponding data ``structures``. This approach not only simplifies data management but also leads to reduced ``storage usag``e and ``more efficient write operations``. The resulting optimization enhances both ``gas efficiency`` and ``contract performance``.

A third optimization entails the careful ``packing of state variables`` into fewer storage ``slots``. This practice leads to minimized ``gas costs`` for both ``reading`` and ``writing variables``, especially when variables within the same slot need to be updated in a single function. This strategy optimizes storage space and contributes to efficient contract execution.

Furthermore, organizing if conditions and ``require()`` statements that validate input arguments at the top of functions is advised. This prudent arrangement helps in early detection of invalid inputs, resulting in swift reverts and avoiding unnecessary ``computational overhead``. This optimization is a proactive measure to enhance contract ``reliability`` and ``execution efficiency``.

In the context of ``event emission``, emitting ``stack variables`` instead of state variables is recommended. Emitting stack variables conserves gas and improves execution efficiency during contract operations that involve emitting events.

The optimization of storage usage is also extended to the choice of data types. Replacing bool with ``uint256(1)`` and ``uint256(2)`` for representing true and false avoids additional storage read and write operations, thus contributing to gas savings and efficient contract execution.

Lastly, the optimization of ``storage slots`` is achieved by packing ``structs`` into fewer slots. This practice leads to more cost-effective ``write operations`` and minimizes the need for additional ``storage-related operations``, thus enhancing both ``gas efficiency`` and ``contract performance``.

These optimizations collectively contribute to the creation of gas-efficient smart contracts that operate smoothly and effectively on the blockchain. By integrating these strategies into contract design and development, developers can harness improved performance, reduced gas costs, and optimal resource utilization.

## Code Commentary

### General Code suggestions for all contracts

1. Use more recent version of solidity
2. Add specific imports instead of over all 
3. Try to reduce the contract names size . Now the contract names really very hard to understand and very big names 
4. Follow the namings as per solidity naming standards . internal/private function and variable names always start with ``_`` . Example ``address[] internal firstCohort;`` ``address[] internal secondCohort;``
5. Don't initailize default values in loops . Like ``for (uint256 i = 0; i < _securityCouncils.length; i++) {``
6. ``address.iscontract()`` checks can use modifiers 
7. Hard coded values should be avoided
8. Provide meaningful error messages in require statements to enhance debugging
9. Add comments to explain complex logic, calculations, and the purpose of functions

#### Only wrote 2 contracts reports. This covers the majority of the suggestions.

``SecurityCouncilManager.sol``

- There's some code duplication between ``_addMemberToCohortArray`` and ``_removeMemberFromCohortArray``. You could potentially refactor this logic into a separate private function to reduce redundancy

-  In ``_removeMemberFromCohortArray``, you're iterating twice through both cohorts to remove the member. Instead, you can optimize this by using a single loop that checks both cohorts and removes the member when found.

-  Emitting events is a good practice for keeping track of contract state changes. Ensure that all relevant state changes are accompanied by corresponding event emissions. Some critical functions like ``_addMemberToCohortArray``,``_removeMemberFromCohortArray`` not emit any events .


-  If the ``SecurityCouncilData`` structure is used in multiple places, consider using a struct to define it. This can help centralize the structure and make it easier to manage changes


-   Consider defining interfaces for ``access control roles``. This can make it clearer what each role is responsible for and can improve the overall readability of your code.


-  Instead of using ``if`` statements followed by ``revert``, consider using ``require`` statements. It makes the code ``cleaner`` and ``clearer``


-  If you anticipate the need for upgrades in the future, you might want to consider using a pattern like the ``OpenZeppelin's Upgrades Plugins`` to enable seamless contract upgrades while preserving storage data.

- Create a ``modifier`` to check for ``zero addresses`` and reuse it in functions that require this check

``SecurityCouncilNomineeElectionGovernor.sol``

- If possible, order the ``struct members based on their size`` (from largest to smallest) to minimize gas usage due to ``potential padding``.

- ``initialize()`` function should be ``external ``

- Reduce the ``inheritance list``

- To improve readability, organize functions and modifiers in a consistent order. For example, you can list constructors first, followed by external functions, internal functions, and then modifiers

- In the ``initialize`` function, add checks to ensure that address parameters ``(nomineeVetter, securityCouncilManager, securityCouncilMemberElectionGovernor)`` are not set to the zero address.

-  In ``_requireLastMemberElectionHasExecuted`` instead of directly reverting, you could emit a custom event when the requirement is not met. This event can be useful for monitoring the state of the contract and providing more context on the error.

- If some of your functions are ``complex``, consider breaking them down into smaller functions with descriptive names. This can improve code ``readability`` and ``maintainability``.

- Instead of scattering the initialization steps throughout the ``initialize`` function, consider consolidating them into a ``separate internal function``. This can improve the overall readability of the initialize function.

- Replace magic numbers ``(e.g., 500)`` with named constants or variables to improve code readability and maintainability. This makes it easier to understand the purpose of these numbers in context.


## Risks as per Analysis

``SecurityCouncilManager.sol``

- The contract manipulates arrays ``(firstCohort and secondCohort)`` to add, remove, and replace members. Incorrect ``array manipulation`` can result in ``unintended state changes`` or ``even data corruption``.

- If the arrays ``firstCohort`` and ``secondCohort`` grow too large, there could be a risk of hitting gas limits when manipulating these arrays, leading to transactions failing or becoming too expensive.

- The contract relies on external contracts like ``UpgradeExecRouteBuilder`` and ``ArbitrumTimelock``. Any vulnerabilities or misconfigurations in these contracts can impact the ``security of this contract``.

- Add bounds checking when accessing arrays to prevent out-of-bounds errors.

- Replace the ``custom role-based access control`` with OpenZeppelin's ``AccessControl`` contract for more standardized and tested access control.

``SecurityCouncilNomineeElectionGovernor.sol``

- The ``replaceCohort`` function replaces a cohort with a new set of members. It is crucial to ensure that this operation is atomic and prevents any race conditions to maintain data consistency across the contract's state.

- The contract relies on ``UpgradeExecRouteBuilder`` to construct security council updates. If there are vulnerabilities or malicious behaviors in this external contract, it could compromise the entire process.

- If this nonce is not managed correctly or is predictable, attackers could manipulate timelock actions and execute unauthorized updates

- The interaction with the ``L2 timelock`` contract is crucial. If there are vulnerabilities in the ``ArbitrumTimelock`` contract or if the contract's interaction with it is not secure, it could result in ``unauthorized changes`` being scheduled.

- The contract relies on external data, such as router and ``l2CoreGovTimelock``. If these external contracts change their behavior or state unexpectedly, it could disrupt the contract's operations.

### Risks Found Based on QA Reports

1. If the condition ``block.timestamp < thisElectionStartTs`` is used and ``block.timestamp`` is equal to ``thisElectionStartTs``, then the condition would evaluate to ``false``, and the transaction would not revert. This means that the election could be created even if the desired start time ``thisElectionStartTs`` has already been reached but not yet ended. 

2. The ``threshold`` is an important parameter that controls how many members of the security council must approve an update before it can be executed. If the threshold is too low, it could be easier for an attacker to take control of the ``security council``

3. The ``perform`` function does not check if the caller is authorized to perform the update. This is a potential security problem, as it could allow an attacker to call the function and add or remove members from the security council without authorization.

4. This could allow an ``COHORT_REPLACER_ROLE`` to replace a cohort that is still being used, which could cause the security council to be inoperable.

5.  Even if the ``updateNonce``  value is increased every time a new proposal is scheduled, the ``generateSalt ``function is still not a secure function. This is because the ``generateSalt`` function simply concatenates the ``newMembers`` array and the ``updateNonce`` variable together to create a salt. This salt is not very secure, and it is possible for an attacker to generate a collision and create a valid proposal with the same salt as another proposal.

6. The divisions below take an input parameter which does not have any zero-value checks, which may lead to the functions reverting when zero is passed. ``params.quorumNumeratorValue`` should be checked before division operation

7. The contract's does not have any sanity checks when assigning ``uint`` values to critical variables in the ``constructor`` and ``initializer`` function. This could lead to human errors that would require the contract to be redeployed.

8. The Protocol contract uses the ``vulnerable`` version of the ``openzeppelin`` library. This means that the contract is susceptible to a number of security vulnerabilities that have been patched in newer versions of the library.Protocol uses the contracts": "4.7.3", contracts-upgradeable": "4.7.3"  there are known vulnerability in this version 

9. The ``MAX_SECURITY_COUNCILS`` constant in the Protocol contract is hardcoded to a value of ``500``. This means that there can only be a maximum of 500 security councils in the protocol. This could be a problem in the future if the protocol needs to support more than 10 security councils.If any problem need to ``increase/decrease`` ``security councils`` means its not possible. Need to redeploy the over all contract .

10. At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation. This can also be called an ” EMERGENCY STOP (CIRCUIT BREAKER) PATTERN “.


## Non-functional aspects

- Aim for high test coverage to validate the contract's behavior and catch potential bugs or vulnerabilities
- The protocol execution flow is not explained in efficient way. 
- Its best to explain over all protocol with architecture is easy to understandings 

## Time spent on analysis 
``15 Hours``

































































### Time spent:
15 hours