# Zero address check for cohort address when calling initialisation function for scm contract

### Severity
Gas Optimization / Informational

### Relevant GitHub Links
https://github.com/ArbitrumFoundation/governance/blob/c18de53820c505fc459f766c1b224810eaeaabc5/src/security-council-mgmt/SecurityCouncilManager.sol#L89C4-L89C5

## Summary
There is currently no check for the cohort max size, as per the documentation the current expected size between both cohorts is 12 (6 each). Also, there is no zero address check for all cohort members that are added.

- My solution will check for both the max per cohort and then any zero addresses in the array
- The `cohortSize` is set based on the `initialize()` require statement, however, if the size is changed later then the check in this would be useful to ensure its enforcing the max - At present its not required.
- Adding the `cohortMaxZ()` function to the `SecurityCouncilManager.sol` file
- Adjusting the `initializer()` function to call `cohortMaxZ()`
- The code could be integrated into the `initializer()` function, but it felt more appropriate to create a function and then declare as a view.

## Vulnerability Details
The `initialize()` function used to initialiase the SecurityCouncilManager contract has no check for the max size of the cohorts, as per the constitution (currently 12 members, 6 per cohort), and no check that any addresses in the cohort are not zero addresses.

## Impact
Cohort voting will be deemed impossible due to cohort members having a 0 address, unexpected behaviour when using the contracts of the protocol for governance, such as voting 9/12 as per the multisig requirements if more than 3 addresses are zero.

## Tools Used
Manual review, Remix, Foundry test

## RESULT
**FOUNDRY TESTS:**
- using 6 addresses with no zero addresses present

```bash
[PASS] test_C4Initialization() (gas: 761460)
Logs:
  first cohort member set: 0x0000000000000000000000000000000000000457
  second cohort member set: 0x00000000000000000000000000000000000008ad
  first cohort member set: 0x0000000000000000000000000000000000000458
  second cohort member set: 0x00000000000000000000000000000000000008ae
  first cohort member set: 0x0000000000000000000000000000000000000459
  second cohort member set: 0x00000000000000000000000000000000000008af
  first cohort member set: 0x000000000000000000000000000000000000045a
  second cohort member set: 0x00000000000000000000000000000000000008b0
  first cohort member set: 0x000000000000000000000000000000000000045b
  second cohort member set: 0x00000000000000000000000000000000000008b1
  first cohort member set: 0x000000000000000000000000000000000000045c
  second cohort member set: 0x00000000000000000000000000000000000008b2

Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 3.83ms
Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)
```

- using 6 address with 2 random address(0) in both cohort arrays
- expected failure, used CohortLengthMismatch() customer error to display the addresses

```bash
Running 1 test for test/security-council-mgmt/pxng0lin_SecurityCouncilManager.t.sol:SecurityCouncilManagerTest
[FAIL. Reason: CohortLengthMismatch([0x0000000000000000000000000000000000000000, 0x0000000000000000000000000000000000000458, 0x0000000000000000000000000000000000000459, 0x0000000000000000000000000000000000000000, 0x000000000000000000000000000000000000045B, 0x000000000000000000000000000000000000045C], [0x00000000000000000000000000000000000008aD, 0x00000000000000000000000000000000000008AE, 0x0000000000000000000000000000000000000000, 0x00000000000000000000000000000000000008b0, 0x00000000000000000000000000000000000008B1, 0x0000000000000000000000000000000000000000])] test_C4Initialization() (gas: 361795)
Test result: FAILED. 0 passed; 1 failed; 0 skipped; finished in 2.96ms
Ran 1 test suites: 0 tests passed, 1 failed, 0 skipped (1 total tests)

Encountered a total of 1 failing tests, 0 tests succeeded
```

## Recommendations
implement a check for max size and cohort member zero addresses upon initialiasation
```
function cohortMaxZ(address[] memory _firstC, address[] memory _secondC) internal view returns(bool) {
        bool pass = true;
        // check the max of the first and second cohort
        require(_firstC.length == cohortSize && _secondC.length == cohortSize, "cohort max check failed"); // gas 265627
        if (_firstC.length != cohortSize || _secondC.length != cohortSize) {
            revert ZeroAddress(); // gas 265503, less gas
        }
        // assign the counter var
        uint256 i;
        // do while loop to check for address(0x0)
        do {
            if (_firstC[i] != address(0x0) && _secondC[i] != address(0x0)) {
                ++i; // && used for either
            } else {
                pass = false;
                break;
            }
           
        } while (i < cohortSize);

        return pass;
}
```
- Then adjusting the initialize() function
```
...
        firstCohort = _firstCohort;
        secondCohort = _secondCohort;
        cohortSize = _firstCohort.length;
        // MAX COHORT MEMBERS AND ADDR(0) CHECK
        bool success = cohortMaxZ(firstCohort, secondCohort);
        if (!success) {
            revert CohortLengthMismatch(firstCohort, secondCohort); // New custom error could be implemented here
        }
```