#Gas Optimisation

## L2SecurityCouncilMgmtFactory.sol line 111, Use the pre-increment operator `++i` instead of the post-increment operator `i++` in for loop to savs gas 

for (uint256 i = 0; i < dp.firstCohort.length; i++) {
            if (!govChainEmergencySCSafe.isOwner(dp.firstCohort[i])) {
                revert AddressNotInCouncil(owners, dp.firstCohort[i]);
            }
        }

### L2SecurityCouncilMgmtFactory.sol line 117 

for (uint256 i = 0; i < dp.secondCohort.length; i++) {
            if (!govChainEmergencySCSafe.isOwner(dp.secondCohort[i])) {
                revert AddressNotInCouncil(owners, dp.secondCohort[i]);
            }
        }