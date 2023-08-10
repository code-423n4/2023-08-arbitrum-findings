
# Summary
|        | Issue | Instances | Gas Saved |
|--------|-------|-----------|-----------|
|[G-01]|Use `calldata` instead of `memory`|4|-341 522|
|[G-02]|The creation of an intermediary array can be avoided|1|-323 094|
|[G-03]|Combine multiple for loop|2|-285 015|
|[G-04]|Optimize array comparison|1|-136 722|
|[G-05]|Activating the  `--via-ir` with foundry|-|-19 932 210|
|[G-06]|Set the number of optimizer runs individually|-|-|


The gas saved column simply adds up the evolution in the snapshot, using the method described in the next section.

# Benchmark
A benchmark is performed on each optimization, using the tests snapshot provided by foundry. This snapshot is based on tests and therefore does not take into account all functions, this loss is accepted. But it is also subject to intrinsic variance. This means that some tests are not relevant to the comparison because they vary too much and are thus not taken into account. These are listed below:
>testNoopUpdate
>testRemoveOne
>testAddOne
>testUpdateCohort
>testCantDropBelowThreshhold
>

## [G-01] Use `calldata` instead of `memory`
Using `calldata` instead of `memory` for function parameters can save gas if the argument is only read in the function. Only instances compilable with a simple type swap are considered valid.

*4 instances*

- [SecurityCouncilManager.sol#L93](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L93)
- [SecurityCouncilManager.sol#L370](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L370)
- [SecurityCouncilNomineeElectionGovernor.sol#L103](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/governors/SecurityCouncilNomineeElectionGovernor.sol#L103)
- [L2SecurityCouncilMgmtFactory.sol#L81](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol#L81) (only `ContractImplementations memory impls`)

Applying this optimisation, those changes appear in the snapshot:

```
testCantUpdateCohortWithADup() (gas: 20 (0.016%))
testE2E() (gas: -51950 (-0.062%))
testSecurityCouncilManagerDeployment() (gas: -47927 (-0.160%))
testNomineeElectionGovDeployment() (gas: -47927 (-0.160%))
testRemovalGovDeployment() (gas: -47927 (-0.160%))
testMemberElectionGovDeployment() (gas: -47927 (-0.160%))
testOnlyOwnerCanDeploy() (gas: -49346 (-0.196%))
testInvalidInit() (gas: -25509 (-0.364%))
testInitialization() (gas: -1263 (-0.626%))
testUpdateSecondCohort() (gas: -2014 (-0.657%))
testUpdateFirstCohort() (gas: -2014 (-0.657%))
testReplaceMemberInSecondCohort() (gas: -2034 (-0.744%))
testRotateMember() (gas: -2034 (-0.753%))
testReplaceMemberInFirstCohort() (gas: -2034 (-0.754%))
testAddMemberAffordances() (gas: -1892 (-0.758%))
testRemoveMember() (gas: -1892 (-0.872%))
testAddMemberToSecondCohort() (gas: -3926 (-1.109%))
testAddMemberToFirstCohort() (gas: -3926 (-1.119%))
Overall gas change: -341522 (-0.024%)
```

## [G-02] The creation of an intermediary array can be avoided
Sometimes, it's not necessary to create an intermediate array to store  values.

*1 instance*

- [SecurityCouncilMgmtUtils.sol#L15-L36](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMgmtUtils.sol#L15-L36)

In this case, an array of maximum size is created because we don't yet know what size the final array will be. This is not useful, as it's more efficient to keep this maximum-size array, fill it and then reduce its size using assembly.

The function can be changed like that:

```diff
function filterAddressesWithExcludeList(
address[]  memory input,
mapping(address => bool)  storage excludeList
)  internal  view  returns  (address[]  memory)  {
- /* */
+	address[]  memory output =  new  address[](input.length);
+	uint256 outputLength =  0;
+	for  (uint256 i =  0; i < input.length; i++)  {
+		if  (!excludeList[input[i]])  {
+			output[outputLength]  = input[i];
+			outputLength++;
+		}
+	}
+	assembly {
+		mstore(output, outputLength) //Set the appropriate size
+	}
+	return output;
}
```

Applying this optimisation, those changes appear in the snapshot:

```
testE2E() (gas: -31712 (-0.038%))
testSecurityCouncilManagerDeployment() (gas: -30294 (-0.101%))
testNomineeElectionGovDeployment() (gas: -30294 (-0.101%))
testRemovalGovDeployment() (gas: -30294 (-0.101%))
testMemberElectionGovDeployment() (gas: -30294 (-0.101%))
testOnlyOwnerCanDeploy() (gas: -30294 (-0.120%))
testInvalidInit() (gas: -30293 (-0.433%))
testTopNomineesGas() (gas: -109619 (-2.434%))
Overall gas change: -323094 (-0.023%)
```

## [G-03] Combine multiple for loop
Whenever possible, it's best to avoid running several for loops one after the other. Most of the time, they can be combined. This avoids certain initialization and counting operations. It's even more interesting when you can take advantage of values that have already been calculated (in this case, a sum of lengths).

*2 instances*

- [L2SecurityCouncilMgmtFactory.sol#L107-L121](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/factories/L2SecurityCouncilMgmtFactory.sol#L107-L121) 

```diff
+		uint256 totalCohortLength = dp.firstCohort.length + dp.secondCohort.length;
-       if (owners.length != (dp.firstCohort.length + dp.secondCohort.length)) {
+       if  (owners.length !=  (totalCohortLength))  {
            revert InvalidCohortsSize(owners.length, dp.firstCohort.length, dp.secondCohort.length);
        }

-        for (uint256 i = 0; i < dp.firstCohort.length; i++) {
-            if (!govChainEmergencySCSafe.isOwner(dp.firstCohort[i])) {
-                revert AddressNotInCouncil(owners, dp.firstCohort[i]);
-            }
-        }

-        for (uint256 i = 0; i < dp.secondCohort.length; i++) {
-            if (!govChainEmergencySCSafe.isOwner(dp.secondCohort[i])) {
-                revert AddressNotInCouncil(owners, dp.secondCohort[i]);
-           }
-       }
+		for (uint256 i = 0; i < totalCohortLength; i++) {
+			address cohortMember; 
+			if (i < dp.firstCohort.length) { 
+				cohortMember = dp.firstCohort[i]; 
+			} 
+			else { 
+				cohortMember = dp.secondCohort[i - dp.firstCohort.length];
+			} 
+			if (!govChainEmergencySCSafe.isOwner(cohortMember)) { 
+				revert AddressNotInCouncil(owners, cohortMember); 
+			} 
+		}
```

Applying this optimisation, those changes appear in the snapshot:

```
testE2E() (gas: -38080 (-0.046%))
testSecurityCouncilManagerDeployment() (gas: -38605 (-0.129%))
testNomineeElectionGovDeployment() (gas: -38605 (-0.129%))
testRemovalGovDeployment() (gas: -38605 (-0.129%))
testMemberElectionGovDeployment() (gas: -38605 (-0.129%))
testOnlyOwnerCanDeploy() (gas: -38667 (-0.154%))
Overall gas change: -231167 (-0.016%)
```

- [SecurityCouncilMemberSyncAction.sol#L60-L72](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilMemberSyncAction.sol#L60-L72)

```diff
-        for (uint256 i = 0; i < _updatedMembers.length; i++) {
-            address member = _updatedMembers[i];
-            if (!securityCouncil.isOwner(member)) {
-                _addMember(securityCouncil, member, threshold);
-            }
-        }
-        
-        for (uint256 i = 0; i < previousOwners.length; i++) {
-            address owner = previousOwners[i];
-            if (!SecurityCouncilMgmtUtils.isInArray(owner, _updatedMembers)) {
-                _removeMember(securityCouncil, owner, threshold);
-            }
-        }

+        for (uint256 i = 0; i < _updatedMembers.length || i < previousOwners.length; i++) {
+            if (i < _updatedMembers.length) {
+                address member = _updatedMembers[i];
+                if (!securityCouncil.isOwner(member)) {
+                    _addMember(securityCouncil, member, threshold);
+                }
+            }
+            
+            if (i < previousOwners.length) {
+                address owner = previousOwners[i];
+                if (!SecurityCouncilMgmtUtils.isInArray(owner, _updatedMembers)) {
+                    _removeMember(securityCouncil, owner, threshold);
+                }
+            }
+        }

```

Applying this optimisation, those changes appear in the snapshot:

```
testNonces() (gas: -944 (-0.011%))
testE2E() (gas: -52904 (-0.064%))
Overall gas change: -53848 (-0.004%)
```

## [G-04] Optimize array comparison

The following function is not well optimized. First of all, there's no need to check the array twice. This alone reduces the cost by half.


*1 instance*

- [SecurityCouncilMgmtUpgradeLib.sol#L52-L88](https://github.com/arbitrumfoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/gov-action-contracts/AIPs/SecurityCouncilMgmt/SecurityCouncilMgmtUpgradeLib.sol#L52-L88)

```diff
function areAddressArraysEqual(address[] memory array1, address[] memory array2) 
	public 
	pure 
	returns (bool)
{
	if (array1.length != array2.length) {
		return false;
	} 
	 for (uint256 i = 0; i < array1.length; i++) {  
		 bool found = false;  
		 for (uint256 j = 0; j < array2.length; j++) {  
			 if (array1[i] == array2[j]) {  
				 found = true;  
				 break;  
			 }  
		 }  
		 if (!found) {  
			 return false;  
		 }  
	 } 
-	 for (uint256 i = 0; i < array2.length; i++) {  
-		 bool found = false;  
-		 for (uint256 j = 0; j < array1.length; j++) {  
-			 if (array2[i] == array1[j]) {  
-				 found = true;  
-				 break;  
-			 }  
-		 }  
-		 if (!found) {  
-			 return false;  
-		 } 
-	 } 
	 return true; 
```

Applying this optimisation, those changes appear in the snapshot:

```
testE2E() (gas: -136722 (-0.164%))
Overall gas change: -136722 (-0.010%)
```

But at this point the function is not yet perfectly optimized. it has a complexity of O(n^2). This can be increased by hashing the arrays, but the elements must be ordered. For this, quicksort is the best algorythm (this is obviously not recalled) (address can be compared by converting them to uint160).
```diff
function areAddressArraysEqual(address[] memory array1, address[] memory array2){
-    /* */
+  	 quicksort(array1);
+        quicksort(array2);
+	 bytes32 hash1 = keccak256(abi.encode(array1));  
+	 bytes32 hash2 = keccak256(abi.encode(array2));
+	 return hash1 == hash2; 
+ }
```

In this way, the function would have a theoretical complexity of O(n log n). But in reality this will be more expensive for small arrays than the first alternative. Another way would be to introduce a hash table to compare elements more efficiently without looping (only one array need to be hashed), but this is again uneconomical on small arrays.
That's why only a benchmark for the simplest optimization is provided. On tests, the other two worsen costs. But it's useful to know that they exist and could be implemented if this function were to become more important.


## [G-05] Activating the  `--via-ir` with foundry

This option allows YUL optimization. It is currently in heavy development and will allow the biggest gain.  

With this activated, those changes appear in the snapshot:
```
testNoVoteForNonCompliantNominee() (gas: 87 (0.070%))
testMiscVotesViews() (gas: 183 (0.080%))
testIncludeNominee() (gas: 978 (0.152%))
testExecute() (gas: -1396 (-0.210%))
testProposalCreationCallRestriction() (gas: -108 (-0.217%))
testProposalExpirationDeadline() (gas: -297 (-0.220%))
testAddContender() (gas: -483 (-0.224%))
testForceSupport() (gas: 500 (0.302%))
testNoZeroWeightVotes() (gas: 568 (0.335%))
testUpdateRouter() (gas: -266 (-0.349%))
testProposalCreationTargetRestriction() (gas: -169 (-0.360%))
testForceSupport() (gas: -638 (-0.361%))
testCannotUseMoreVotesThanAvailable() (gas: -927 (-0.377%))
testAddSC() (gas: 493 (0.417%))
testInvalidParams() (gas: 721 (0.436%))
testProposalCreationUnexpectedCallDataLen() (gas: -191 (-0.459%))
testProposalCreationValuesRestriction() (gas: -301 (-0.486%))
testUpdateSecondCohort() (gas: -1516 (-0.495%))
testRegisterTokenOnL2NotEnoughVal() (gas: 22178 (0.501%))
testUpdateFirstCohort() (gas: -1547 (-0.505%))
testRegisterTokenOnL2() (gas: 23576 (0.516%))
testReplaceMemberInFirstCohort() (gas: -1405 (-0.521%))
testReplaceMemberInSecondCohort() (gas: -1449 (-0.530%))
testSelectTopNomineesFails() (gas: -1613 (-0.598%))
testRemoveAllOutboxes() (gas: 4345 (0.627%))
testZeroToken() (gas: 509 (0.708%))
testCantUpdateCohortWithADup() (gas: 903 (0.733%))
testProposalCreationTargetLen() (gas: 297 (0.839%))
testCountVote() (gas: 5034 (0.934%))
testRotateMember() (gas: -2537 (-0.940%))
testSetRecipientsTwice() (gas: 60317 (0.944%))
testSuccessfulProposalAndCantAbstain() (gas: -1351 (-0.947%))
testProposalCreationCallParamRestriction() (gas: -630 (-1.122%))
testClaimAndDelegate() (gas: 67720 (1.157%))
testClaimAndDelegateFailsWrongNonce() (gas: 67376 (1.161%))
testClaimAndDelegateFailsForWrongSender() (gas: 67642 (1.166%))
testSetRecipientsFailsWhenAddingTwice() (gas: 66686 (1.167%))
testSetRecipientsFailsNotEnoughDeposit() (gas: 66676 (1.176%))
testClaimAndDelegateFailsForExpired() (gas: 67870 (1.181%))
testSweepFailsTwice() (gas: 67968 (1.182%))
testSetRecipients() (gas: 67564 (1.185%))
testClaimFailsBeforeStart() (gas: 67688 (1.187%))
testSweepAfterClaim() (gas: 68723 (1.187%))
testWithdrawFailsNotOwner() (gas: 68257 (1.189%))
testWithdraw() (gas: 68257 (1.189%))
testSweep() (gas: 68426 (1.190%))
testClaimFailsForTwice() (gas: 68459 (1.192%))
testSetSweepReceiver() (gas: 68050 (1.193%))
testClaimFailsForUnknown() (gas: 68198 (1.195%))
testSetSweepReceiverFailsNullAddress() (gas: 68221 (1.196%))
testClaimFailsForFalseTransfer() (gas: 68071 (1.197%))
testClaimFailsAfterEnd() (gas: 68372 (1.199%))
testClaim() (gas: 68864 (1.199%))
testSweepFailsBeforeClaimPeriodEnd() (gas: 68404 (1.199%))
testSweepFailsForFailedTransfer() (gas: 68586 (1.202%))
testWithdrawFailsTransfer() (gas: 68731 (1.205%))
testSetSweepReceiverFailsOwner() (gas: 68791 (1.206%))
testSetRecipientsFailsWrongAmountCount() (gas: 68869 (1.270%))
testSetRecipientsFailsWrongRecipientCount() (gas: 69167 (1.276%))
testSetRecipientsFailsNotOwner() (gas: 69302 (1.279%))
testSuccessNumeratorSufficientVotes() (gas: -4743 (-1.313%))
testReplaceMemberAffordances() (gas: -2763 (-1.325%))
testDoesDeployAndDeposit() (gas: 73458 (1.359%))
testSuccessNumeratorInsufficientVotes() (gas: -4916 (-1.372%))
testExecutorPermissions() (gas: -191009 (-1.394%))
testDoesDeploy() (gas: 74634 (1.398%))
testExecutorPermissionsFail() (gas: -191646 (-1.401%))
testPastCirculatingSupplyExclude() (gas: -195020 (-1.412%))
testPastCirculatingSupplyMint() (gas: -194775 (-1.418%))
testCantReinit() (gas: -194401 (-1.422%))
testPastCirculatingSupply() (gas: -194587 (-1.423%))
testProperlyInitialized() (gas: -194591 (-1.424%))
testProposalDoesExpire() (gas: -3916 (-1.437%))
testRemoveSeC() (gas: -602 (-1.581%))
testVotesToWeight() (gas: 2467 (1.614%))
testCastVoteReverts() (gas: 544 (1.634%))
testE2E() (gas: -1367483 (-1.645%))
testOnlyNomineeElectionGovernorCanPropose() (gas: 1859 (1.687%))
testSetFullWeightDuration() (gas: 593 (1.697%))
testCohortMethods() (gas: 3006 (1.922%))
testNonces() (gas: -162554 (-1.955%))
testAddMemberToSecondCohort() (gas: -6982 (-1.972%))
testRemoveMember() (gas: -4312 (-1.987%))
testAddMemberToFirstCohort() (gas: -7523 (-2.144%))
testUpgraderCanCancel() (gas: -665066 (-2.321%))
testSanityCheckValues() (gas: -659682 (-2.322%))
testRoles() (gas: -662154 (-2.332%))
testSetMinDelay() (gas: -662603 (-2.336%))
testContractsDeployed() (gas: -662956 (-2.338%))
testContractsInitialized() (gas: -664183 (-2.339%))
testDeploySteps() (gas: -663740 (-2.340%))
testProxyAdminOwnership() (gas: -663818 (-2.340%))
testSetMinDelayRevertsForCoreAddress() (gas: -673708 (-2.371%))
testGetPrevOwner() (gas: -189384 (-2.392%))
testGetPrevOwner() (gas: -189429 (-2.393%))
testExcludeNominee() (gas: 10353 (2.396%))
testAddressRegistryAddress() (gas: 1162 (2.470%))
testUpgradeAndCall() (gas: 3567 (2.494%))
testCreateElection() (gas: 6424 (2.572%))
testRelay() (gas: 1133 (2.683%))
testBridgeBurnNotGateway() (gas: -91388 (-2.696%))
testBridgeBurn() (gas: -91715 (-2.701%))
testBridgeMint() (gas: -91658 (-2.703%))
testInit() (gas: -91270 (-2.720%))
testProposeReverts() (gas: 873 (2.724%))
testBridgeMintNotGateway() (gas: -91691 (-2.744%))
testInitialization() (gas: 5564 (2.757%))
testAction() (gas: 17747 (2.820%))
testInitZeroNovaGateway() (gas: -91312 (-2.874%))
testInitZeroNovaRouter() (gas: -91425 (-2.878%))
testInitZeroGateway() (gas: -91430 (-2.878%))
testL2() (gas: 20363 (2.956%))
testProperlyInits() (gas: -498030 (-3.048%))
testRelease() (gas: -501792 (-3.051%))
testMigrationTargetMustBeContract() (gas: -498768 (-3.053%))
testBeneficiaryCanSetBeneficiary() (gas: -498785 (-3.054%))
testOnlyOwnerCanMigrate() (gas: -498812 (-3.055%))
testOwnerCanSetBeneficiary() (gas: -498941 (-3.055%))
testRandomAddressCantSetBeneficiary() (gas: -499262 (-3.057%))
testOnlyBeneficiaryCanRelease() (gas: -499210 (-3.057%))
testRelay() (gas: 1304 (3.083%))
testUpgrade() (gas: 4253 (3.102%))
testInitFails() (gas: -310887 (-3.154%))
testAddOutbxesAction() (gas: -21773 (-3.342%))
testAddSCAffordances() (gas: -3777 (-3.377%))
testNoLogicContractInit() (gas: 92072 (3.419%))
testTopNomineesGas() (gas: 155981 (3.464%))
testCastVote() (gas: -572625 (-3.534%))
testCastVoteFailsForNonBeneficiary() (gas: -571395 (-3.538%))
testDelegate() (gas: -571417 (-3.553%))
testClaim() (gas: -570936 (-3.567%))
testDelegateFailsForNonBeneficiary() (gas: -571142 (-3.568%))
testReleaseAffordance() (gas: -571476 (-3.570%))
testDoesDeploy() (gas: -570820 (-3.574%))
testClaimFailsForNonBeneficiary() (gas: -571228 (-3.577%))
testVestedAmountStart() (gas: -579489 (-3.605%))
testCastVoteReverts() (gas: 1205 (3.620%))
testCanTransferAndCallContract() (gas: 155618 (3.695%))
testAIP1Point2() (gas: -50235 (-3.798%))
testRemoveMemberAffordances() (gas: -3991 (-4.028%))
testCannotTransferAndCallReverter() (gas: 168980 (4.067%))
testOldClaimStart() (gas: 168880 (4.084%))
testZeroOwner() (gas: 169312 (4.097%))
testClaimStartAfterClaimEnd() (gas: 169462 (4.098%))
testZeroDelegateTo() (gas: 169641 (4.105%))
testZeroReceiver() (gas: 169664 (4.105%))
testCanMintLessThan2Percent() (gas: 168658 (4.112%))
testCanTransferAndCallEmpty() (gas: 168579 (4.115%))
testCanMintTwiceWithWarp() (gas: 337439 (4.120%))
testCanMint2Percent() (gas: 169018 (4.121%))
testCanMintZero() (gas: 168505 (4.128%))
testCannotMintTwice() (gas: 337579 (4.138%))
testCannotTransferAndCallNonReceiver() (gas: 169517 (4.140%))
testCanBurn() (gas: 168585 (4.145%))
testCannotMintWithoutFastForward() (gas: 169207 (4.158%))
testCannotMintNotOwner() (gas: 169434 (4.164%))
testCannotMintMoreThan2Percent() (gas: 169620 (4.166%))
testMigrateEthToNewWalletWithSlowerVesting() (gas: -804645 (-4.181%))
testMigrateTokensToNewWalletWithSlowerVesting() (gas: -804858 (-4.182%))
testMigrateTokensToNewWalletWithFasterVesting() (gas: -804957 (-4.182%))
testIsInitialised() (gas: 170701 (4.191%))
testAddressRegistryAddress() (gas: 2294 (4.197%))
testAdminCanChangeExecutor() (gas: 109468 (4.237%))
testTransfer() (gas: 253081 (4.266%))
testAddMemberAffordances() (gas: -10695 (-4.286%))
testTransferNotOwner() (gas: 253427 (4.297%))
testCantAddEOA() (gas: -41996 (-4.333%))
testInitZeroToken() (gas: 252706 (4.344%))
testCantReAddOutbox() (gas: -42359 (-4.346%))
testInit() (gas: 253551 (4.355%))
testExecuteFailsForAdmin() (gas: 118121 (4.435%))
testExecuteFailsForNobody() (gas: 118350 (4.439%))
testDoesNotInitialiseZeroL1Token() (gas: 169811 (4.468%))
testDoesNotInitialiseZeroInitialSup() (gas: 169869 (4.469%))
testDoesNotInitialiseZeroOwner() (gas: 170361 (4.482%))
testSetNomineeVetter() (gas: 1813 (4.545%))
testRemoveOutboxes() (gas: -39070 (-4.574%))
testInit() (gas: 111581 (4.596%))
testCantExecuteEOA() (gas: 112213 (4.599%))
testSetVoteSuccessNumeratorAffordance() (gas: 2198 (4.615%))
testProposeFails() (gas: 925 (4.708%))
testUpdateRouterAffordacnes() (gas: -5340 (-4.752%))
testSetVoteSuccessNumerator() (gas: 1429 (4.759%))
testExecute() (gas: 127628 (4.766%))
testSeparateSelector() (gas: 1142 (4.852%))
testInitFailsZeroAdmin() (gas: 112840 (4.931%))
testRemoveSCAffordances() (gas: -4049 (-5.047%))
testRelay() (gas: 2143 (5.053%))
testIsInArray() (gas: 107 (5.090%))
testSelectTopNominees(uint256) (gas: 18748 (5.516%))
testProperInitialization() (gas: 2735 (5.535%))
testUpdateCohortAffordances() (gas: -4660 (-5.615%))
testProperInitialization() (gas: 4756 (6.087%))
testExecuteInbox() (gas: 369596 (6.436%))
testRouteBuilderErrors() (gas: -72037 (-6.451%))
testCancelFailsBadSender() (gas: 352395 (6.568%))
testSchedule() (gas: 361753 (6.757%))
testExecuteInboxNotEnoughVal() (gas: 369341 (6.784%))
testCancel() (gas: 362558 (6.814%))
testExecuteInboxInvalidData() (gas: 370823 (6.839%))
testExecute() (gas: 370566 (6.861%))
testScheduleFailsBadL2Timelock() (gas: 362455 (6.862%))
testScheduleFailsBadSender() (gas: 362652 (6.872%))
testDoesDeploy() (gas: 363553 (6.900%))
testDoesNotDeployZeroL2Timelock() (gas: 363572 (7.311%))
testDoesNotDeployZeroInbox() (gas: 363824 (7.313%))
testSecurityCouncilManagerDeployment() (gas: -2297814 (-7.671%))
testNomineeElectionGovDeployment() (gas: -2298257 (-7.677%))
testRemovalGovDeployment() (gas: -2298788 (-7.679%))
testMemberElectionGovDeployment() (gas: -2298868 (-7.680%))
testOwnerCanSetHashTwice() (gas: 21267 (8.061%))
testOwnerCanSetHash() (gas: 21075 (8.070%))
testMonOwnerCannotSetHash() (gas: 21303 (8.105%))
testConstructor() (gas: 21188 (8.169%))
testAddAndRemoveSequencer() (gas: 39930 (8.261%))
testCantAddZeroAddress() (gas: 20311 (8.620%))
testOnlyOwnerCanDeploy() (gas: -2410046 (-9.571%))
testL1() (gas: 30668 (11.800%))
testSetMinDelayRevertsForCoreAddress() (gas: 1356671 (12.572%))
testL1GovernanceFactory() (gas: 1368102 (12.711%))
testSetMinDelay() (gas: 1367888 (12.739%))
testInvalidInit() (gas: -902461 (-12.888%))
testInitReverts() (gas: -845212 (-17.426%))
testOnlyOwnerCanCreateWallets() (gas: -305406 (-20.314%))
testPauseAndUpauseInbox() (gas: 78574 (21.217%))
testDeploy() (gas: -1148592 (-25.025%))
Overall gas change: -19932210 (-1.389%)
```
Not all tests have lower costs, so this change can be widely discussed.

## [G-06] Set the number of optimizer runs individually
Some development environments allow the number of optimizer runs to be defined individually for each contract. This is difficult to achieve in foundry. But hardhat is the best example. The main limitation to the number of runs in the case of contracts that will be heavily executed is the size of the deployment, which is limited. With foundry, a single contract completely blocks this number. Here, the maximum number is 1915 (set to 1900 in foundry) for the `SecurityCouncilNomineeElectionGovernor.sol` contract. By only setting this one to this value and greatly increase for the other, a huge saving can be achieved.

We therefore suggest moving to a development/deployment environment where this option is available. No contract in scope expects the limit, even for 100,000 runs.

This optimization certainly has the biggest impact, but as the tests are written with foundry, it's impossible to provide a benchmark in a short space of time.
