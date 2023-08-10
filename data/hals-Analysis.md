# 1. Audit Scope

The Arbitrum DAO incorporates a security council that can take certain emergency and non-emergency actions, and the scope of the audit was to review the codebase related to the election process of the security council members.
The audited codebase consits of appx. 2184 nSLoC distributed over 20 contracts.

# 2. Learnings

From reviewing the Arbitrum election related codebase,I was able to achieve the following:

1. Learning about Arbitrum DAO and the cycle of the security council members election.
2. Learning about synchronizing actions between chains.
3. Learning more about on-chain voting management and timelocks.
4. Gaining valuable foundry testing skills; as the project uses mocking alot in their tests, which enables interacting with the required parts of some contracts without the need to deploy and initialize them (has its pros and cons though).

# 3. Approach Taken

1. first, I gained a high-level understanding of the protocol's objectives and architecture from the provided documentation.
2. then conducted a detailed manual review of the code, trying to identify potential vulnerabilities,and where the code deviates from the intended design (as it's considered a bug).
3. then tried to match-pattern with known vulnerabilities reported in previous election/governance projects,mainly referred to solodit website for past related vulnerabilities found in similar projects.
4. I referred to the tests,changing the test parameters to analyse the what-if scenarios and to write PoCs.
5. finally, I documented my findings with PoCs.

# 4. Codebase Analysis

- The codebase was very distinguished in terms of:

  - The flow of election process and how it’s synced between the contracts.
  - Synced deployment of the election management contracts, as each one is heavily dependent on the other as an actor; they are deployed together in one transaction.

- The in-scope **main** contracts can be divided into three groups:

#### 1. Security Council Governors (Election):

- `SecurityCouncilManager` contract:
  where all the security council members management operations are done; replace cohorts,add,remove & swap members.

- `SecurityCouncilNomineeElectionGovernor` contract:
  where the election process for a new cohort members starts;it has the logic of adding contenders,excluding and adding nominees within the vetting period, and starting the next phase (member election) once deadline has passed and a minimum number of nominees has been met.

- `SecurityCouncilMemberElectionGovernor` contract:
  where the voting process for a new cohort members is continued; it has the logic of voting on nominees, then after the deadline of the member election process has passed; a new cohort members are set.

- `SecurityCouncilMemberRemovalGovernor` contract:
  where a proposal is set to initiate the voting process to remove a cohort members.

#### 2. Deployment:

- `L2SecurityCouncilMgmtFactory` contract:
  where security council management contracts are deployed and initialized on the governance chain (Arbitrum).

#### 3. Activation:

- `GovernanceChainSCMgmtActivationAction`,`L1SCMgmtActivationAction` & `NonGovernanceChainSCMgmtActivationAction`contracts:
  where elections are activated on the three chains; Arbitrum,Ethereum & Nova.

#### 4. Routes Builder:

- `UpgradeExecRouteBuilder` contract:
  where it builds routes to target the upgrade executors on each of the three chain.

# 5. Architecture Feedback

1. Deployment:  
   in `L2SecurityCouncilMgmtFactory`: it deploys the four security governors contracts and syncs the relations between these contracts: as it sets the cohort replacer role in the `SecurityCouncilManager` to the address of `SecurityCouncilMemberElectionGovernor` which makes it less prone to any mis-alignmnet that might occure if each of these contracts are deployed individually.

   ![Deployment](https://drive.google.com/uc?id=1GwBPahhPslo6mqCanmE0WXCNQBrIgGqn)

2. Election flow:  
   in nomineeElection contract ⇒ createElection ⇒ 0-7 days to add contenders ⇒ from day 7 to day 21 (14 days) nominees are vetted: added/excluded ⇒ then anyone can execute the proposal to be sent to the membersElection contract ⇒ from day21 + 21 days are added for voting on the nominees ⇒ then the top 6 nominees are added to the security council while the old cohort members are removed:  
   
   ![Election Flow](https://drive.google.com/uc?id=11zEsVTOTKwDlh4TX19TfIMWdBWEmEfgo)

3. Timelock:  
   after the security council members election ends, users are given some time to withdraw their asset if they don't agree with the proposed change.

   ![Timelock](https://drive.google.com/uc?id=13r9IOD5dO372z1oI3s2YiDdh50bdX30-)

# 6. Centralization Risks

#### `SecurityCouncilMemberElectionGovernor` contract:

1.  The owner of this contract can use `relay` function to call any function in any contract that's accessible only by the address of `SecurityCouncilMemberElectionGovernor` contract; this gives the owner the ability to replace the members of any cohort any time without election.

2.  Also the owner can change the `fullWeightDuration` of the memberElection contract even if there's a running/active proposal; which makes the owner in control of the result of members voting by increaing and decreasing the weight in favor of some members.

#### `SecurityCouncilMemberRemovalGovernor` contract:

1.  The owner can change the `voteSuccessNumerator` of the memberRemoval contract even if there's a running/active proposal; which makes a malicious owner controls the result of member removing voting by increaing or decreasing the `voteSuccessNumerator` in favor of/against this member.

2.  The owner of this contract can use `relay` function to remove any cohort member any time without election.

# 7. Systemic Risks

- In general, the codebase is robust and follows the security best practices and implementations.
- Some low/medium vulnerabilites were detected:
  1. Time as per design is not respected in all functions; for example: in `SecurityCouncilNomineeElectionGovernor`, no checks are made on nomineeVettingDuration if it complies with the design value or not (14 days).
  2. Setting the values of `fullWeightDuration` and `votingPeriod()` equal in `SecurityCouncilMemberElectionGovernor` will break the voting as `votesToWeight` will always revert (due to dividion by zero).
  3. No check for the minimum number of security members upon removal/which makes executing emergency contracts un-executable if it reaches below minimum threshold of 9 members (emergency proposals needs the approval of at least 9 members to be executed); recommended adding separate functions for swap and remove members.
  4. If the removed cohort member vacancy is to be filled; it can't be filled via nomineeElection contract, only directly by the `MEMBER_ADDER_ROLE` role which is assigned to the govChainEmergencySecurityCouncil.
  5. Security management contracts are typically deployed on the governance chain via `L2SecurityCouncilMgmtFactory` contract; but these contracts can be deployed by anyone maliciously and used to trick users; so it must be ensured that the deployer of these contrats is the `L2SecurityCouncilMgmtFactory` contract address.
  6. `relay` function can't send native tokens to the target address when called; as it's non-payable.

# 8. Other Recommendations

1. In `SecurityCouncilManager`: check that cohorts sizes comply with the designed value (6 members for each cohort) upon contract initialization, and if it's deemed acceptable to have less than 6 members/cohort at any time (assuming that some cohort members were removed and their vacancies were not filled); then set cohortSize to a constant value of 6.

2. In `SecurityCouncilNomineeElectionGovernor`: assign the designed `nomineeVettingDuration` upon initialization.

3. In `SecurityCouncilNomineeElectionGovernor`: add a mechanism to enable creating elections for cohort vacancies between the main elections (which occurs each 6-months).

# 9. Time Spent

Approximately 35 hours ; divided between manually reviewing the codebase, reading documentation, foundry testing, and documenting my findings.


### Time spent:
35 hours