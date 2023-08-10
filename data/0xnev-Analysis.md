## 1. Analysis of Codebase

### 1.1 Summary 
In Arbritrum, there exists 2 types of actors that can propose to make changes to the Arbritrum ecosystem:

1. Arbritrum DAO: Regular delegatees/holders of ARB token
2. Arbritrum Security Council: A council of 12 members controlling a multisig that can make changes to ARB ecosystem in the event of emergencies.

This analysis focuses on the Arbritrum Security Council Election, where every 6-months, an election is held where 6 council members spots of the existing 12 council members are put up for election, giving delegatee (voters) power to decide on new members of the security council.

### 1.2 Actors
There are 4 main actors in the Arbritrum Security Council Ecosystem:

### 1.2.1 Contender
Contendor is someone who can register themselves during the nominee slection phase and receive votes from delegatee/voter

### 1.2.2 Nominee
Nominee is a contendor who has received the minimum 0.2% threshold of votable tokens in the nominee selection phase

### 1.2.2a Compliant Nominee
A compliant nominee is one who has successfully pass the compliance stage by the nominee vetter after meeting the minimum 0.2% threshold of votable tokens during the nominee selection phase

### 1.2.2b Noncompliant Nominee
A noncompliant nominee is one who has failed to pass the compliance stage by the nominee vetter even after meeting the minimum 0.2% threshold of votable tokens during the nominee selection phase, and is excluded by the nominee vetter to continue to the member election phase

### 1.2.3 Member
After the nominee compliant phase, the top 6 nominees that receive the most votes casted by delegatees/voters during the membership election phase wiill be elected as the new members of the Arbritrum Security Council

### 1.2.4 Delegatee/Voter
Delegatee/voters (holders of ARB token) cast votes during 3 of the 4 existing phases.

First during the nominee selection phase, where contendors are selected to become nominees for election to become members. 

Second, during the member election phase, where votes casted during the nominee selection phase by delegatees/voters is resetted to be recasted during this phase to decide on the top 6 nominees to be elected into the Arbritrum Security Council.

Third, during a separate membership removal proposal phase specifically for removing an existing member.


### 1.3 Phases
### 1.3.1 Nominee Election Phase
This is the phase that spans 7 days, where the top 6 contendors with the most votes voted by delgatees/voters are selected to potentially enter the membership election phase, provided they pass the compliance checks during the nominee compliance phase conducted by the Arbitrum DAO nominee vetter commitee.

### 1.3.2 Nominee Compliance Phase
This phase spans 14 days, where compliance checks are conducted on potential contendors selected as nominees by nominee vetting commitee. If any nominee fails potential compliance check resulting in the fixed cohort size of 6 for member election phase not being met, the nominee vetting committee can manually include nominees to ensure that cohort size is met. 

### 1.3.3 Member Election Phase
### 1.3.3a Full Weight Voting
After the nominee compliance phase, any votes casted during the first 7 days of the member election phase will account for full weight towards nominee selected.

### 1.3.3b Linearly Decreasing Weight Voting
After the first 7 days of the Member election phase has passed, any votes casted during the next 14 days of the member election phase will have a linearly decreasing weight to incentivize early voting and disincentivize late voting. This is presumably to avoid any sudden swings in votes for nominees.

### 1.3.4 Member Removal Phase
This is a separate phase where proposals can be proposed to call exlcusively, `SecurityCouncilManager.removeMember()`, to remove a member from the security council, where thereafter a spot on the council would be guranteed to be opened for election.


## 2. Architecture Improvements

### 2.1 No queuing and vetoer mechanism
Consider adding a queuing period and vetoer role before execution as a fail-safe and last check of proposal instead of allowing immediate execution after voting and vetting periods. 

### 2.2 Implement a proposal cap for security council member removal proposals
In `SecurityCouncilMemberRemovalGovernor.propose()`, There is no proposal cap, so anybody who meets proposal threshold can spam and propose member removal proposals as long as member to remove is currently in a existing cohort. This differs from nominee and member election proposals, where a proposal can only be proposed bi-anually.

While such proposals still need to go through the standard governance voting process, allowing uncapped proposing of proposals can affect the UI/UX of Arbitrum DAO voting site.

### 2.3 `SecurityCouncilNomineeElectionGovernor._requireLastMemberElectionHasExecuted()`: Cannot create another election unless previous election is executed
To avoid having to undergo a contract upgrade in the event of previously blocked elections due to a lack of nominees, consider creating another type of proposal specifically for governance to resolve previously blocked proposals by extending proposal vetting deadlines

### 2.4 Sudden Swing of Votes for member removal proposal

When casting votes for member removal proposals, against/for voters can wait till the last minute to cast votes, resulting in no time for voters of the other support group to react. This leads to preventing proposal execution/causing immediate execution of proposals. Given there is no queuing and vetoeing mechanism, this scenario could be abused to prevent execution of proposal or cause immediate execution of proposal.

### 2.5 Consider adding a check to ensure that inclusion of new nominee is part of current cohort
Based on consitution, nominee vetter can only add member of the outgoing security council randomly selected as evident by the code comments and docs:

> In the event that fewer than six candidates are supported by pledged votes representing at least 0.2% of all Votable Tokens, the current Security Council members whose seats are up for election may become candidates (as randomly selected out of their Cohort) until there are 6 candidates.

While nominee vetter is a trusted role and additional nominee added still need to undergo standard governance voting procedure, an additional check could be made to track the current cohort members to ensure that the nominee added to supplement insufficient cohort must be from the current cohort which adheres to the constitution rules and prevent malicious nominee vetters from adding other nominees.


## 3. Centralization Risks
### 3.1 Nominee Vetter in `SecurityCouncilNomineeElectionGovernor.sol`

- `excludeNominee()`: Appointed nominee vetter can exclude any nomineee to be put up for election even if contendor meet the minimum threshold (0.2% votes) to become nominee

<br/>

- `includeNominee()`: Appointed nominee vetter can include any contendor to become nominee and contest for election even if contendor do not meet the minimum threshold (0.2% votes) to qualify to be a nominee.

### 3.2 Security Council
The 12 member security council is in itself a centralized entity, who are trusted to make emergency changes to the overall Arbritrum system without having to go through DAO proposals.


## 4. Time Spent
- Day 1: Understand protocol flow and mechanisms
- Day 2: Audit contracts based on protocol flow and mechanism stated in 4.1
- Day 3: Finish up analysis and QA report

### 4.1 Contract Flow
1. `SecurityCouncilNomineeElectionGovernor.sol`: Nominee election phase and nominee compliance phase
2. `SecurityCouncilMemberElectionGovernor.sol`: Member election phase
3. `SecurityCouncilMemberRemovalGovernor.sol`: Member removal phase
4. `SecurityCouncilManager.sol`: Executes cross-chain changes to security councils after proposals has passed 



### Time spent:
16 hours