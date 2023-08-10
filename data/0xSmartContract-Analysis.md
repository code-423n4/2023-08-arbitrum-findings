# Analysis  - Arbitrum Security Council Elections
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|b) |Test analysis | Test scope of the project and quality of tests |
|c) |Architectural | Architecture feedback |
|d) |Documents  | What is the scope and quality of documentation for Users and Administrators? |
|e) |Systemic risks | Potential systemic risks in the project |
|f) |Competition analysis| What are similar projects? |
|g) |Security Approach of the Project | Audit approach of the Project |
|h) |Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis |
|i) |Gas Optimization | Gas usage approach of the project and alternative solutions to it |
|j) |New insights and learning from this audit | Things learned from the project |


## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2023-08-arbitrum#scope

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2023-08-arbitrum#tests)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review| [Arbitrum Governancel](https://github.com/ArbitrumFoundation/governance/blob/main/docs/overview.md) |Provides a basic architectural teaching for General Architecture|
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither](https://github.com/crytic/slither)| |
|5|Test Suits|[Tests](https://github.com/code-423n4/2023-08-arbitrum#tests)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2023-08-arbitrum#scope)||
|7|Infographic|[Figma](https://www.figma.com/)|I made Visual drawings to understand the hard-to-understand mechanisms|
|8|Special focus on Areas of  Concern|[Areas of Concern](https://github.com/code-423n4/2023-08-arbitrum#areas-of-concern)||




## b) Test analysis

The audit scope of the contracts to be audited is 94% and it should be aimed to be 100%.

```

- What is the overall line coverage percentage provided by your tests?: 94%
```


### What could they have done better?;
There are many unit tests in the project, integration tests in which the interaction of contracts with each other are modeled should be increased.

### Test suites do not test for attack vectors, especially re-entrancy

Test teams are testing many functions and variables, but recently, due to the vulnerability in the Vyper Compiler, the hacking of the projects using certain Vyper compiler and losing 50 million $ has revealed the security weakness here.
https://cointelegraph.com/news/curve-finance-pools-exploited-over-24-reentrancy-vulnerability

Attack vectors, especially re-entrancy, seem untested and trusted

```solidity

governance/src/UpgradeExecutor.sol:
  46      ///         that do not touch local state should be used.
  47:     function execute(address upgrade, bytes memory upgradeCallData)
  48:         public
  49:         payable
  50:         onlyRole(EXECUTOR_ROLE)
  51:         nonReentrant
  52:     {


```

## c) Architectural 


How the elections work
The Security Council is a 12-member council divided into two groups. Every six months, there are elections to fill the seats in these two groups, respectively.

Every elected Security Council member’s term lasts one year, excluding the first cohort’s member terms, which are truncated by the amount of time between the date of the first election specified in the Constitution and the date of the DAO's launch.

![image](https://github.com/code-423n4/2023-08-arbitrum/assets/104318932/fe78e7d2-d838-482b-83f5-f294a58823ad)


To become a candidate for the Security Council, you must be a member of the Arbitrum DAO. You must also have support from at least 0.2% of all votable tokens. Once the candidates have been chosen, all members of the Arbitrum DAO can vote for the candidates. The 6 candidates who receive the most votes will be elected to the Security Council. Additionally, the Arbitrum Foundation may set forth further guidelines and procedures for ensuring a fair, transparent and effective elections process.

No more than 3 candidates from the same organization should be elected into the Security Council. Also, candidates shouldn't have any conflicts of interest that would prevent them from acting in the best interests of the Arbitrum DAO.

The rules for the Security Council elections can be changed by the members of the Arbitrum DAO, but these changes can't be made during an ongoing election. Security Council members can also be removed from their position prior to their term ending if at least 10% of all votable tokens participate in a removal vote, and at least 5/6 of the votes are in favor of removal. A member can also be removed if at least 9 members of the Security Council vote in favor of removal.





## d) Documents 
- Documentation should be increased further, it is recommended to add the architectural design to the documents as infographic

- I would also recommend adding quality Medium articles, it's a great way to provide an in-depth look at many of the topics in the project and is used by many blockchain projects.



## f) Competition analysis


![image](https://github.com/code-423n4/2023-08-arbitrum/assets/104318932/66057a5a-fca1-48cc-8fde-24d235ebca85)


<img width="613" alt="image" src="https://github.com/code-423n4/2023-08-arbitrum/assets/104318932/3655ce71-f60d-4181-913b-d9a9df452594">




## g) Security Approach of the Project
Successful current security understanding of the project;
1- The first audit was conducted in a reputable audit firm such as Trail of Bits and all security concerns were resolved, this is the case in both audit reports;

https://github.com/ArbitrumFoundation/governance/tree/c18de53820c505fc459f766c1b224810eaeaabc5/audits

2- They manage the 2nd audit process with an innovative audit such as Code4rena, in which many auditors examine the codes.

3- After the inspection, the concept of "continuous inspection" should be adhered to with bug bounty programs such as ImmuneFi.


What the project should add in the understanding of Security;
1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)
2- After the project is published on the mainnet, there should be emergency action plans (not found in the documents)


## h) Other Audit Reports and Automated Findings 

**Automated Findings:**
https://github.com/code-423n4/2023-08-arbitrum/blob/main/bot-report.md


Especially  Medium detections in the Automated Finding Report should be taken into account;

### Medium Risk Issues

| |Issue|Instances|
|-|:-|:-:|
| [[M&#x2011;01](#m01-the-owner-is-a-single-point-of-failure-and-a-centralization-risk)] | The `owner` is a single point of failure and a centralization risk | 5 | 

**Other Audit Reports:**

While investigating the potential risks in the project, the past audit reports can give us serious insights, so they should be taken into account and analyzed in detail. 


https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/audits/trail_of_bits_aips1-1and1-2__5_6_2023.pdf
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/audits/trail_of_bits_governance_report_1_6_2023.pdf

<img width="810" alt="image" src="https://github.com/code-423n4/2023-08-arbitrum/assets/104318932/db69e453-f193-497b-83a3-b2c2b6527f33">


## i) Gas Optimization

The project is generally efficient in terms of gas optimizations, many generally accepted gas optimizations have been implemented, gas optimizations with minor effects are already mentioned in automatic finding, but gas optimizations will not be a priority considering code readability and code base size

Highest gas optimization: It is Storage Packed, high gas gain will be achieved in case of packaging in the following structs

https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L393

```solidity
File: src/security-council-mgmt/SecurityCouncilManager.sol

393:              SecurityCouncilData memory securityCouncilData = securityCouncils[i];

```

## j) New insights and learning from this audit 

- I learned the concept of "Security Council" and the point of use in Layer2, also I understood the architecture of this in the codes
- I learned the details of The Security Council elections.

- I learned the details of DAO Proposals and Voting Procedures




### Time spent:
12 hours