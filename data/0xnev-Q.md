### Findings
| Count | Title |
|:--:|:-------|
| [L-01] |Implement a proposal cap for security council member removal proposals | 
| [L-02] |Sudden Swing of Votes for member removal proposal | 
| [L-03] |`SecurityCouncilNomineeElectionGovernor.includeNominee()`: Missing `onlyVettingPeriod()` modifier | 
| [L-04] |`SecurityCouncilNomineeElectionGovernor.includeNominee()`: Missing check adhering to constitution | 
| [NC-01] |`SecurityCouncilNomineeElectionGovernor._requireLastMemberElectionHasExecuted()`: Cannot create another election unless previous election is executed | 
| [NC-02] |No queuing and vetoer mechanism for proposals |
| [R-01] | SecurityCouncilManager._removeMemberFromCohortArray()`: No need to loop through both cohort | 

| Total Findings | 7 |
|:--:|:--:|

## [L-01] Implement a proposal cap for security council member removal proposals

## Impact
[SecurityCouncilMemberRemovalGovernor.sol#L106-L143](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L106-L143)

In `SecurityCouncilMemberRemovalGovernor.propose()`, There is no proposal cap, so anybody who meets proposal threshold can spam and propose member removal proposals as long as member to remove is currently in a existing cohort. This differs from nominee and member election proposals, where a proposal can only be proposed bi-anually.

While such proposals still need to go through the standard governance voting process, allowing uncapped proposing of proposals can affect the UI/UX of Arbitrum DAO voting site.

## Recommendation
Implement a minimum proposal cap for each proposer. Many DAOS only allow one active proposal per proposer.

## [L-02] Sudden Swing of Votes for member removal proposal

## Impact
[SecurityCouncilMemberRemovalGovernor.sol#L163-L174](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L163-L174)

When casting votes for member removal proposals, against/for voters can wait till the last minute to cast votes, resulting in no time for voters of the other support group to react. This leads to preventing proposal execution/causing immediate execution of proposals. Given there is no queuing and vetoeing mechanism, this scenario could be abused to prevent execution of proposal or cause immediate execution of proposal.

## Recommendation
Consider implementing similar decreasing voting power to disincentvize late voting. 

If not, add a objection period where a sudden swing in votes from Succeeded to Defeated will allow for voters to vote and react to sudden change in state of proposal.

## [L-03] `SecurityCouncilNomineeElectionGovernor.includeNominee()`: Missing `onlyVettingPeriod()` modifier

## Impact 
[SecurityCouncilNomineeElectionGovernor.sol#L290-L317](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L290-L317)

In the nominee election phase, the appointed nominee vetter can manually include nominees via `SecurityCouncilNomineeElectionGovernor.includeNominee()` without having to go through a proposal.

However, the `onlyVettingPeriod()` modifier is missing, and as such the vetting deadline is not checked, meaning the nominee vetter can still include nominees even after vetting deadline has passed.

While it is acknowledged in test file `SecurityCouncilNomineeElectionGovernor.t.sol` that `includeNominee` should succeed even after vetting period, ARB DAO should consider only allowing inclusion of nominees before vetting period ends to ensure fairness, given exclusion can only be performed within vetting period.

## Recommendation
```diff
- function includeNominee(uint256 proposalId, address account) external onlyNomineeVetter {
+ function includeNominee(uint256 proposalId, address account) external onlyNomineeVetter onlyVettingPeriod(proposalId) {    
    ProposalState state_ = state(proposalId);
    if (state_ != ProposalState.Succeeded) {
        revert ProposalNotSucceededState(state_);
    }

    if (isNominee(proposalId, account)) {
        revert NomineeAlreadyAdded(account);
    }

    uint256 cnCount = compliantNomineeCount(proposalId);
    uint256 cohortSize = securityCouncilManager.cohortSize();
    if (cnCount >= cohortSize) {
        revert CompliantNomineeTargetHit(cnCount, cohortSize);
    }

    // can't include nominees from the other cohort (the cohort not currently up for election)
    // this only checks against the current the current other cohort, and against the current cohort membership
    // in the security council, so changes to those will mean this check will be inconsistent.
    // this check then is only a relevant check when the elections are running as expected - one at a time,
    // every 6 months. Updates to the sec council manager using methods other than replaceCohort can effect this check
    // and it's expected that the entity making those updates understands this.
    if (securityCouncilManager.cohortIncludes(otherCohort(), account)) {
        revert AccountInOtherCohort(otherCohort(), account);
    }

    _addNominee(proposalId, account);
}
```

## [L-04] `SecurityCouncilNomineeElectionGovernor.includeNominee()`: Missing check adhering to constitution

## Impact
[SecurityCouncilNomineeElectionGovernor.sol#L290-L317](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L290-L317)

Based on consitution, nominee vetter can only add member of the outgoing security council randomly selected as evident by the code comments and docs:

> In the event that fewer than six candidates are supported by pledged votes representing at least 0.2% of all Votable Tokens, the current Security Council members whose seats are up for election may become candidates (as randomly selected out of their Cohort) until there are 6 candidates.


While nominee vetter is a trusted role and additional nominee added still need to undergo standard governance voting procedure, an additional check could be made to track the current cohort members to ensure that the nominee added to supplement insufficient cohort must be from the current cohort which adheres to the constitution rules. It also prevents malicious nominee vetters from adding other nominees.


## Recommendation
```diff
function includeNominee(uint256 proposalId, address account) external onlyNomineeVetter {
    ProposalState state_ = state(proposalId);
    if (state_ != ProposalState.Succeeded) {
        revert ProposalNotSucceededState(state_);
    }

    if (isNominee(proposalId, account)) {
        revert NomineeAlreadyAdded(account);
    }

    uint256 cnCount = compliantNomineeCount(proposalId);
    uint256 cohortSize = securityCouncilManager.cohortSize();
    if (cnCount >= cohortSize) {
        revert CompliantNomineeTargetHit(cnCount, cohortSize);
    }

    // can't include nominees from the other cohort (the cohort not currently up for election)
    // this only checks against the current the current other cohort, and against the current cohort membership
    // in the security council, so changes to those will mean this check will be inconsistent.
    // this check then is only a relevant check when the elections are running as expected - one at a time,
    // every 6 months. Updates to the sec council manager using methods other than replaceCohort can effect this check
    // and it's expected that the entity making those updates understands this.
    if (securityCouncilManager.cohortIncludes(otherCohort(), account)) {
        revert AccountInOtherCohort(otherCohort(), account);
    }

+   if (!securityCouncilManager.cohortIncludes(currentCohort(), account)) {
+       revert AccountNotInCurrentCohort(otherCohort(), account);
+   }

    _addNominee(proposalId, account);
}
```

## [NC-01] `SecurityCouncilNomineeElectionGovernor._requireLastMemberElectionHasExecuted()`: Cannot create another election unless previous election is executed

## Impact and Recommendation
[SecurityCouncilNomineeElectionGovernor.sol#L163](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L163)

To avoid having to undergo a contract upgrade in the event of previously blocked elections due to a lack of nominees, consider creating another type of proposal specifically for governance to resolve previously blocked proposals by extending proposal vetting deadlines

## [NC-02] No queuing and vetoer mechanism for proposals

## Impact and Recommendation
Consider adding a queuing period and vetoer role before execution as a fail-safe and last check of member election proposal instead of allowing immediate execution after voting and vetting periods. 

## [R-01] `SecurityCouncilManager._removeMemberFromCohortArray()`: No need to loop through both cohort

## Impact and Recommendation
[SecurityCouncilManager.sol#L161-L173](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L161-L173)

Since a member cannot be in both cohorts at the same time, use the enum flag `Cohort` to indicate in `removeMember()`, `replaceMember()` and `rotateMember()` in which cohort the member will be removed via `_removeMemberFromCohortArray()` instead of looping through both cohorts.
