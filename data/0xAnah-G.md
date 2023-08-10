# ARBITRUM GAS OPTIMIZATIONS

## Introduction
Please be aware that some code snippets may be shortened to conserve space, and certain code snippets may include @audit tags in comments to facilitate issue explanations."

Emphasizing the significance of our recommendations, we strive to improve the code's efficiency while maintaining its readability. We recognize the importance of code that is easily maintainable and understandable for both developers and auditors.

## TABLE OF FINDINGS

| Number |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| G-01| Refactor external/internal function to avoid unnecessary SLOAD | 1 | 100 |
| G-02| Using storage instead of memory for structs/arrays saves gas | 1 | 4200 |
| G-03| Move lesser gas costing require checks to the top | 3 |   |
| G-04| Use v4.9.0 or above OpenZeppelin contracts |  |  |
| G-05| Declaring Unnecessary variables | 3 |  6 |
| G-06 | Use the existing Local variable/global variable when equal to a state variable to avoid reading from state | 1 | 100 |
| G-07| Refactor `replaceCohort()` function for batch operations | 1 |  16 per iteration |
| G-08 | Using calldata instead of memory for read-only arguments in external functions saves gas | 11 |  286 |
| G-09| Use assembly to hashes, in order to save gas | 2 |   |








## [G-01] Refactor external/internal function to avoid unnecessary SLOAD
The functions below read storage slots that are previously read in the functions that invoke them. We can refactor the external/internal functions to pass cached storage variables as stack variables and avoid the extra storage reads that would otherwise take place in the internal functions.


### 1 Instance
- https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L161-#L209
```solidity
file: src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

161:    function createElection() external returns (uint256 proposalId) {
162:        // require that the last member election has executed
163:        _requireLastMemberElectionHasExecuted();
164:
165:        // each election has a deterministic start time
166:        uint256 thisElectionStartTs = electionToTimestamp(electionCount);
167:        if (block.timestamp < thisElectionStartTs) {
168:            revert CreateTooEarly(block.timestamp, thisElectionStartTs);
169:        }
170:
171:        (
172:            address[] memory targets,
173:            uint256[] memory values,
174:            bytes[] memory callDatas,
175:            string memory description
176:        ) = getProposeArgs(electionCount);
177:
178:        proposalId = GovernorUpgradeable.propose(targets, values, callDatas, description);
179:
180:        electionCount++;
181:    }
.
.
.
187:    function _requireLastMemberElectionHasExecuted() internal view {
188:        if (electionCount == 0) {
189:            return;
190:        }
191:
192:        (
193:            address[] memory prevTargets,
194:            uint256[] memory prevValues,
195:            bytes[] memory prevCallDatas,
196:            string memory prevDescription
197:        ) = getProposeArgs(electionCount - 1);
198:
199:        uint256 prevProposalId =
200:            hashProposal(prevTargets, prevValues, prevCallDatas, keccak256(bytes(prevDescription)));
201:
202:        if (
203:            IGovernorUpgradeable(address(securityCouncilMemberElectionGovernor)).state(
204:                prevProposalId
205:            ) != ProposalState.Executed
206:        ) {
207:            revert LastMemberElectionNotExecuted(prevProposalId);
208:        }
209:    }
```
The `createElection()` function above invokes the `_requireLastMemberElectionHasExecuted()` but both function reads the `electionCount` state variable. We can refactor both functions such that the `createElection()` function passes the `electionCount` as an argument to the `_requireLastMemberElectionHasExecuted()` function since this function just reads and do not modify the `electionCount` state variable. Refactoring the code this way would save 1 `SLOAD` 100 gas units (cold access). The diff below shows how the functions could be modified: 
```diff
diff --git a/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol b/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol
index 196c2c5..f65723d 100644
--- a/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol
+++ b/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol
@@ -159,11 +159,12 @@ contract SecurityCouncilNomineeElectionGovernor is
     ///         Can be called by anyone every 6 months.
     /// @return proposalId The id of the proposal
     function createElection() external returns (uint256 proposalId) {
+        uint256 electionCount_ = electionCount; 
         // require that the last member election has executed
-        _requireLastMemberElectionHasExecuted();
+        _requireLastMemberElectionHasExecuted(electionCount_);
 
         // each election has a deterministic start time
-        uint256 thisElectionStartTs = electionToTimestamp(electionCount);
+        uint256 thisElectionStartTs = electionToTimestamp(electionCount_);
         if (block.timestamp < thisElectionStartTs) {
             revert CreateTooEarly(block.timestamp, thisElectionStartTs);
         }
@@ -173,7 +174,7 @@ contract SecurityCouncilNomineeElectionGovernor is
             uint256[] memory values,
             bytes[] memory callDatas,
             string memory description
-        ) = getProposeArgs(electionCount);
+        ) = getProposeArgs(electionCount_);
 
         proposalId = GovernorUpgradeable.propose(targets, values, callDatas, description);

@@ -184,8 +185,8 @@ contract SecurityCouncilNomineeElectionGovernor is
     ///      Ensures that there are no unexpected behaviors from multiple elections running at the same time.
     ///      If, for some reason, the previous member election is blocked,
     ///      it is up to the security council or DAO to unblock the previous election before creating a new one.
-    function _requireLastMemberElectionHasExecuted() internal view {
-        if (electionCount == 0) {
+    function _requireLastMemberElectionHasExecuted(uint electionCount_) internal view {
+        if (electionCount_ == 0) {
                 return;
         }
 
         (
             address[] memory prevTargets,
             uint256[] memory prevValues,
             bytes[] memory prevCallDatas,
             string memory prevDescription
-        ) = getProposeArgs(electionCount - 1);
+        ) = getProposeArgs(electionCount_ - 1);
 
         uint256 prevProposalId =
         hashProposal(prevTargets, prevValues, prevCallDatas, keccak256(bytes(prevDescription)));
```
```
Estimated gas saved: 100 gas units
```


## [G-02]  Using storage instead of memory for structs/arrays saves gas

### 1 Instance
- https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L130
```solidity
file: src/UpgradeExecRouteBuilder.sol

105:    function createActionRouteData(
106:        uint256[] memory chainIds,
107:        address[] memory actionAddresses,
108:        uint256[] memory actionValues,
109:        bytes[] memory actionDatas,
110:        bytes32 predecessor,
111:        bytes32 timelockSalt
112:    ) public view returns (address, bytes memory) {
113:        if (chainIds.length != actionAddresses.length) {
114:            revert ParamLengthMismatch(chainIds.length, actionAddresses.length);
.
.
.       
128:        // from the l1 timelock
129:        for (uint256 i = 0; i < chainIds.length; i++) {
130:            UpExecLocation memory upExecLocation = upExecLocations[chainIds[i]]; @audit convert to storage variable
131:            if (upExecLocation.upgradeExecutor == address(0)) { @audit cache upExecLocation.upgradeExecutor
132:                revert UpgadeExecDoesntExist(chainIds[i]);
133:            }
134:
135:            if (actionDatas[i].length == 0) {
136:                revert EmptyActionBytesData(actionDatas);
137:            }
138:
139:            bytes memory executorData = abi.encodeWithSelector(
140:                UpgradeExecutor.execute.selector, actionAddresses[i], actionDatas[i]
141:            );
142:
143:            // for L1, inbox is set to address(0):
144:            if (upExecLocation.inbox == address(0)) { @audit cache upExecLocation.inbox
145:                schedTargets[i] = upExecLocation.upgradeExecutor;
.
.
.         
178:    }
```
When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. Therefore over `4200` gas units could be saved if we change `upExecLocation` from a memory variable to a storage variable and cache `upExecLocation.upgradeExecutor` and `UpExecLocation.inbox` into cheaper stack variables `upgradeExecutorAddress` and `UpExecLocationInbox` respectively.
```diff
diff --git a/src/UpgradeExecRouteBuilder.sol b/src/UpgradeExecRouteBuilder.sol    
index fdc9d38..57d2134 100644
--- a/src/UpgradeExecRouteBuilder.sol
+++ b/src/UpgradeExecRouteBuilder.sol
@@ -127,8 +127,10 @@ contract UpgradeExecRouteBuilder {
         // for each chain create calldata that targets the upgrade executor      
         // from the l1 timelock
         for (uint256 i = 0; i < chainIds.length; i++) {
-            UpExecLocation memory upExecLocation = upExecLocations[chainIds[i]]; 
-            if (upExecLocation.upgradeExecutor == address(0)) {
+            UpExecLocation storage upExecLocation = upExecLocations[chainIds[i]];
+            address upgradeExecutorAddress = upExecLocation.upgradeExecutor;
+            address UpExecLocationInbox = UpExecLocation.inbox;
+            if (upgradeExecutorAddress == address(0)) {
                 revert UpgadeExecDoesntExist(chainIds[i]);
             }
             if (actionDatas[i].length == 0) {
@@ -140,8 +142,8 @@ contract UpgradeExecRouteBuilder {
             );

             // for L1, inbox is set to address(0):
-            if (upExecLocation.inbox == address(0)) {
-                schedTargets[i] = upExecLocation.upgradeExecutor;
+            if (UpExecLocationInbox == address(0)) {
+                schedTargets[i] = upgradeExecutorAddress;        
                 schedValues[i] = actionValues[i];
                 schedData[i] = executorData;
             } else {
@@ -149,8 +151,8 @@ contract UpgradeExecRouteBuilder {
                 schedTargets[i] = RETRYABLE_TICKET_MAGIC;        
                 schedValues[i] = 0;
                 schedData[i] = abi.encode(
-                    upExecLocation.inbox,
-                    upExecLocation.upgradeExecutor,
+                    UpExecLocationInbox,
+                    upgradeExecutorAddress,
                     actionValues[i],       
                     0,
                     0,
                    executorData
                );
            }
        }
```
```
Estimated gas saved: 4200 gas units
```
#### Please note this instance was not included in the bots report



## [G-03] Move lesser gas costing require checks to the top
Require() or revert() statements that check input arguments or cost lesser gas should be at the top of the function.
Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting alot of gas in a function that may ultimately revert in the unhappy case.

### 3 Instances
- https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L112-#L114
```solidity
file: src/security-council-mgmt/SecurityCouncilManager.sol

89:     function initialize(
90:         address[] memory _firstCohort,
91:         address[] memory _secondCohort,
92:         SecurityCouncilData[] memory _securityCouncils,
93:         SecurityCouncilManagerRoles memory _roles,
94:         address payable _l2CoreGovTimelock,
95:         UpgradeExecRouteBuilder _router
96:     ) external initializer {
97:         if (_firstCohort.length != _secondCohort.length) {
98:             revert CohortLengthMismatch(_firstCohort, _secondCohort);
99:         }
100:        firstCohort = _firstCohort;
101:        secondCohort = _secondCohort;
102:        cohortSize = _firstCohort.length;
103:        _grantRole(DEFAULT_ADMIN_ROLE, _roles.admin);
104:        _grantRole(COHORT_REPLACER_ROLE, _roles.cohortUpdator);
105:        _grantRole(MEMBER_ADDER_ROLE, _roles.memberAdder);
106:        for (uint256 i = 0; i < _roles.memberRemovers.length; i++) {
107:            _grantRole(MEMBER_REMOVER_ROLE, _roles.memberRemovers[i]);
108:        }
109:        _grantRole(MEMBER_ROTATOR_ROLE, _roles.memberRotator);
110:        _grantRole(MEMBER_REPLACER_ROLE, _roles.memberReplacer);
111:
112:        if (!Address.isContract(_l2CoreGovTimelock)) {
113:            revert NotAContract({account: _l2CoreGovTimelock});
114:        }
115:        l2CoreGovTimelock = _l2CoreGovTimelock;
116:
117:        _setUpgradeExecRouteBuilder(_router);
118:        for (uint256 i = 0; i < _securityCouncils.length; i++) {
119:            _addSecurityCouncil(_securityCouncils[i]);
120:        }
121:    }
```
The if revert check statement on [line 112 - line 114](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L112-#L114) can be moved higher the function since it checks if the argument `l2CoreGovTimelock` is a contract or not so that in scenarios where `l2CoreGovTimelock` argument is not a function the function can revert without executing gas consuming operations of [line 100 - line 110](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L100-#L110). The code could be refactored to:  
```diff
diff --git a/src/security-council-mgmt/SecurityCouncilManager.sol b/src/security-council-mgmt/SecurityCouncilManager.sol
index 8ec1308..59e9bae 100644
--- a/src/security-council-mgmt/SecurityCouncilManager.sol
+++ b/src/security-council-mgmt/SecurityCouncilManager.sol
@@ -97,6 +97,11 @@ contract SecurityCouncilManager is
         if (_firstCohort.length != _secondCohort.length) {
             revert CohortLengthMismatch(_firstCohort, _secondCohort);
         }
+
+        if (!Address.isContract(_l2CoreGovTimelock)) {
+            revert NotAContract({account: _l2CoreGovTimelock});
+        }
+        
         firstCohort = _firstCohort;
         secondCohort = _secondCohort;
         cohortSize = _firstCohort.length;
@@ -109,9 +114,6 @@ contract SecurityCouncilManager is
         _grantRole(MEMBER_ROTATOR_ROLE, _roles.memberRotator);
         _grantRole(MEMBER_REPLACER_ROLE, _roles.memberReplacer);

-        if (!Address.isContract(_l2CoreGovTimelock)) {
-            revert NotAContract({account: _l2CoreGovTimelock});
-        }
         l2CoreGovTimelock = _l2CoreGovTimelock;
```
- https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L243-#L245
```solidity
file: src/security-council-mgmt/SecurityCouncilManager.sol

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
The if revert check statement on [line 243 - line 245](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L243-#L245) can be moved higher the function since it checks if the argument `_securityCouncilData` member `chainId` is equal to 0. This check is cheaper than the first check of the function which reads from state. The function would be more gas efficient if this check is moved to the top of the contract so that in scenarios where this check fails the function could revert without performing more gas consuming operations. The code could be refactored to:  
```diff
diff --git a/src/security-council-mgmt/SecurityCouncilManager.sol b/src/security-council-mgmt/SecurityCouncilManager.sol
index 8ec1308..fac0ec9 100644
--- a/src/security-council-mgmt/SecurityCouncilManager.sol
+++ b/src/security-council-mgmt/SecurityCouncilManager.sol
@@ -229,6 +229,10 @@ contract SecurityCouncilManager is
     }

     function _addSecurityCouncil(SecurityCouncilData memory _securityCouncilData) internal {
+        if (_securityCouncilData.chainId == 0) {
+            revert SecurityCouncilZeroChainID(_securityCouncilData);
+        }
+        
         if (securityCouncils.length == MAX_SECURITY_COUNCILS) {
             revert MaxSecurityCouncils(securityCouncils.length);
         }
@@ -240,10 +244,6 @@ contract SecurityCouncilManager is
             revert ZeroAddress();
         }

-        if (_securityCouncilData.chainId == 0) {
-            revert SecurityCouncilZeroChainID(_securityCouncilData);
-        }
-
         if (!router.upExecLocationExists(_securityCouncilData.chainId)) {
             revert SecurityCouncilNotInRouter(_securityCouncilData);
```

- https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L226-#L228
```solidity
file: src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol

218:    function addContender(uint256 proposalId) external {
219:        ElectionInfo storage election = _elections[proposalId];
220:
221:        if (election.isContender[msg.sender]) {
222:            revert AlreadyContender(msg.sender);
223:        }
224:
225:        ProposalState state_ = state(proposalId);
226:        if (state_ != ProposalState.Active) {
227:            revert ProposalNotActive(state_);
228:        }
229:
.
.
.        
236:        if (securityCouncilManager.cohortIncludes(otherCohort(), msg.sender)) {
237:            revert AccountInOtherCohort(otherCohort(), msg.sender);
238:        }
239:
240:        election.isContender[msg.sender] = true;
241:
242:        emit ContenderAdded(proposalId, msg.sender);
243:    }
```
The if revert check statement on [line 225 - line 227](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L226-#L228) can be moved higher the function since it checks if the `state` of the `proposalId` argument is equal to `ProposalState.Active`. This check is cheaper than the first check of the function which reads from state. The function would be more gas efficient if this check is moved to the top of the contract so that in scenarios where this check fails the function could revert without performing more gas consuming operations. The code could be refactored to:  
```diff
diff --git a/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol b/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol
index 196c2c5..b47cf2c 100644
--- a/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol
+++ b/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol
@@ -216,16 +216,17 @@ contract SecurityCouncilNomineeElectionGovernor is
     /// @dev    Can be called only while a proposal is active (in voting phase)
     ///         A contender cannot be a member of the opposite cohort.
     function addContender(uint256 proposalId) external {
+        ProposalState state_ = state(proposalId);
+        if (state_ != ProposalState.Active) {
+            revert ProposalNotActive(state_);
+        }
+        
         ElectionInfo storage election = _elections[proposalId];

         if (election.isContender[msg.sender]) {
             revert AlreadyContender(msg.sender);
         }

-        ProposalState state_ = state(proposalId);
-        if (state_ != ProposalState.Active) {
-            revert ProposalNotActive(state_);
-        }
```

## [G-04] Use v4.9.0 or above OpenZeppelin contracts
From your [package.json file](https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/package.json#L69-#L70) it shows that OpenZeppelin v4.7.3 was used I would recommend you use v4.9.0 or above as it provides many gas optimizations that would help make the protocol gas efficient. https://github.com/OpenZeppelin/openzeppelin-contracts/releases/tag/v4.9.0


## [G-05] Declaring Unnecessary variables
some variables were defined even though they are used once. Not defining variables can reduce gas cost and contract size.

### 2 Instances
- https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L126
```solidity
95:      function _countVote(
96:         uint256 proposalId,
97:         address account,
98:         uint8 support,
99:         uint256 availableVotes,
100:        bytes memory params
101:    ) internal virtual override {
102:        if (support != 1) {
103:            revert InvalidSupport(support);
104:        }
.
.
.
126:        uint240 prevWeightReceived = election.weightReceived[nominee]; @audit unnecessary variable
127:        election.votesUsed[account] = prevVotesUsed + votes;
128:        election.weightReceived[nominee] = prevWeightReceived + weight;
.
.
.
140:    }
```
In the `_countVote()` function above `prevWeightReceived` variable was declared on [line 126](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L126) but only used once in the function thereby making its declaration redundant and unnecessary. `3` units of gas can be saved if we remove this declaration and use `election.weightReceived[nominee]` directly. The code could be refactored as shown in the diff below:
```diff
diff --git a/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol b/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol
index d02c47d..0d3347e 100644
--- a/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol
+++ b/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol
@@ -121,13 +121,12 @@ abstract contract SecurityCouncilMemberElectionGovernorCountingUpgradeable is
         uint256 prevVotesUsed = election.votesUsed[account];
         if (prevVotesUsed + votes > availableVotes) {
             revert InsufficientVotes(prevVotesUsed, votes, availableVotes);
         }
 
-        uint240 prevWeightReceived = election.weightReceived[nominee];
         election.votesUsed[account] = prevVotesUsed + votes;
-        election.weightReceived[nominee] = prevWeightReceived + weight;
+        election.weightReceived[nominee] = election.weightReceived[nominee] + weight;
```

- https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L165
```solidity
file: rc/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol

164:    function fullWeightVotingDeadline(uint256 proposalId) public view returns (uint256) {
165:        uint256 startBlock = proposalSnapshot(proposalId);  @audit unnecessary variable
166:        return startBlock + fullWeightDuration;
167:    }
```
In the `fullWeightVotingDeadline()` function above `startBlock` variable was declared on [line 165](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L165) but only used once in the function thereby making its declaration redundant and unnecessary. `3` units of gas can be saved if we remove this declaration and use `proposalSnapshot(proposalId)` directly. The code could be refactored as shown in the diff below:
```diff
diff --git a/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol b/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol
index d02c47d..0d3347e 100644
--- a/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol
+++ b/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol
@@ -162,8 +162,7 @@ abstract contract SecurityCouncilMemberElectionGovernorCountingUpgradeable is
     /// @notice The deadline after which voting weight will linearly decrease
     /// @param proposalId The proposal to check the deadline for
     function fullWeightVotingDeadline(uint256 proposalId) public view returns (uint256) {
-        uint256 startBlock = proposalSnapshot(proposalId);
-        return startBlock + fullWeightDuration;
+        return proposalSnapshot(proposalId) + fullWeightDuration;
     } 
```
```
Estimated gas saved: 6 gas units
```

## [G-06] Use the existing Local variable/global variable when equal to a state variable to avoid reading from state

### 1 Instance
- https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L233
```solidity
file: src/security-council-mgmt/SecurityCouncilManager.sol

231:    function _addSecurityCouncil(SecurityCouncilData memory _securityCouncilData) internal {
232:        if (securityCouncils.length == MAX_SECURITY_COUNCILS) {
233:            revert MaxSecurityCouncils(securityCouncils.length); @audit securityCouncils.length replace with MAX_SECURITY_COUNCILS
234:        }
.
.
.
268:   }        
```
we can avoid 1 `SLOAD` 100 gas units (warm access) if we replace the `securityCouncils.length` in the revert statement `revert MaxSecurityCouncils(securityCouncils.length)` with the immutable variable `MAX_SECURITY_COUNCILS` since they would need to be equal for the revert to occur.
```diff
diff --git a/src/security-council-mgmt/SecurityCouncilManager.sol b/src/security-council-mgmt/SecurityCouncilManager.solindex 8ec1308..21b4eff 100644
--- a/src/security-council-mgmt/SecurityCouncilManager.sol
+++ b/src/security-council-mgmt/SecurityCouncilManager.sol
@@ -230,7 +230,7 @@ contract SecurityCouncilManager is
 
     function _addSecurityCouncil(SecurityCouncilData memory _securityCouncilData) internal {
         if (securityCouncils.length == MAX_SECURITY_COUNCILS) {
-            revert MaxSecurityCouncils(securityCouncils.length);
+            revert MaxSecurityCouncils(MAX_SECURITY_COUNCILS);
         }
```
```
Estimated gas saved: 100 gas units
```

### [G-07] Refactor `replaceCohort()` function for batch operations

### Instances
```solidity
file: src/gov-action-contracts/execution-record/ActionExecutionRecord.sol

124:    function replaceCohort(address[] memory _newCohort, Cohort _cohort)
125:        external
126:        onlyRole(COHORT_REPLACER_ROLE)
127:    {
128:        if (_newCohort.length != cohortSize) {
129:            revert InvalidNewCohortLength({cohort: _newCohort, cohortSize: cohortSize});
130:        }
131:
132:        // delete the old cohort
133:        _cohort == Cohort.FIRST ? delete firstCohort : delete secondCohort;
134:
135:        for (uint256 i = 0; i < _newCohort.length; i++) {
136:            _addMemberToCohortArray(_newCohort[i], _cohort);
137:        }
138:
139:        _scheduleUpdate();
140:        emit CohortReplaced(_newCohort, _cohort);
141:    }
```
In the `replaceCohort()` function above ` _addMemberToCohortArray()` function was invoked this causes the EVM to perform 2 `JUMP` (8 gas units) instructions which would cost `16` gas units per iteration rather than doing this we could refactor the function that logic of adding a member to the new cohort is done in the `replaceCohort()` function without calling another function. The code could be refactored as shown in the diff below:
```diff
diff --git a/src/security-council-mgmt/SecurityCouncilManager.sol b/src/security-council-mgmt/SecurityCouncilManager.sol
index 8ec1308..25bd46d 100644
--- a/src/security-council-mgmt/SecurityCouncilManager.sol
+++ b/src/security-council-mgmt/SecurityCouncilManager.sol
@@ -133,7 +133,22 @@ contract SecurityCouncilManager is
         _cohort == Cohort.FIRST ? delete firstCohort : delete secondCohort;

         for (uint256 i = 0; i < _newCohort.length; i++) {
-            _addMemberToCohortArray(_newCohort[i], _cohort);
+            if (_newCohort[i] == address(0)) {
+                revert ZeroAddress();
+                }
+            address[] storage cohort = _cohort == Cohort.FIRST ? firstCohort : secondCohort;
+            if (cohort.length == cohortSize) {
+            revert CohortFull({cohort: _cohort});
+            }
+            if (firstCohortIncludes(_newCohort[i])) {
+                revert MemberInCohort({member: _newCohort[i], cohort: Cohort.FIRST});
+            }
+            if (secondCohortIncludes(newCohort[i])) {
+                revert MemberInCohort({member: _newCohort[i], cohort: Cohort.SECOND});
+            }
+        
+            cohort.push(_newCohort[i]);
+
         }

         _scheduleUpdate();
```
```
Estimated gas saved: 16 gas units per iteration
```


## [G-08] Using calldata instead of memory for read-only arguments in external functions saves gas

### 11 Instances
- https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L90-#L92
- https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L124
- https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L231
- https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L271
- https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L279
- https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L103
- https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/modules/SecurityCouncilMemberElectionGovernorCountingUpgradeable.sol#L191
- https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilMemberRemovalGovernor.sol#L107-#L110
- https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L106-#L109
- https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/UpgradeExecRouteBuilder.sol#L187-#L189
- https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L127
 


## [G-09] Use assembly to hashes, in order to save gas

### 2 Instances
- https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L200
- https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/execution-record/ActionExecutionRecord.sol#L20



## CONCLUSION 
As you move forward with implementing the suggested optimizations, we urge you to exercise caution and conduct meticulous testing. It is essential to verify that these changes do not introduce any new vulnerabilities and effectively deliver the desired performance improvements. Carefully review the code modifications and perform rigorous testing to validate the security and effectiveness of the refactored code.
