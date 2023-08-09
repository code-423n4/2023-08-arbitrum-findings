1.

There are some relay functions in several contracts. See

```solidity
    function relay(address target, uint256 value, bytes calldata data)
        external
        virtual
        override
        onlyOwner
    {
        AddressUpgradeable.functionCallWithValue(target, data, value);
    }
```

functionCallWithValue is a function that allow user to send ether. However, the current relay function is unable to allow user to do so because it is not marked as payable and we need to modify the relay function. 

Suggestion:

```solidity
function relay(address target, uint256 value, bytes calldata data)
    external
    +++ payable
    virtual
    override
    onlyOwner
{
    +++ require(msg.value == value, "Incorrect Ether value sent");
    AddressUpgradeable.functionCallWithValue(target, data, value);
}
```

2.

See SecurityCouncilNomineeElectionGovernor#createElection()

Election suppose to be created every 6 month. However, it can be created too late.

Below is the foundry test modified from the existing SecurityCouncilNomineeElectionGovernorTest#testCreateElection

```solidity
function testCreateElection() public {
 // ... existing code

 +++    vm.warp(expectedStartTimestamp + 365 days); // Next election can be created after 1 year
 +++    governor.createElection();

 +++    vm.warp(expectedStartTimestamp + 365 days * 10);// Next election can be created after 10 year
 +++    governor.createElection();

 +++    vm.warp(expectedStartTimestamp + 365 days * 100);// Next election can be created after 100 year
 +++    governor.createElection();
}
```

Suggestion:

Change SecurityCouncilNomineeElectionGovernor#createElection

```solidity
function createElection() external returns (uint256 proposalId) {
        // require that the last member election has executed
        _requireLastMemberElectionHasExecuted();

        // each election has a deterministic start time
        uint256 thisElectionStartTs = electionToTimestamp(electionCount);
        if (block.timestamp < thisElectionStartTs) {
            revert CreateTooEarly(block.timestamp, thisElectionStartTs);
        }

        +++ uint256 thisElectionEndTs = electionToTimestamp(electionCount + 1);
        +++ if (block.timestamp >= thisElectionEndTs) {
        +++  revert CreateTooLate(block.timestamp, thisElectionStartTs);
        +++}

        (
            address[] memory targets,
            uint256[] memory values,
            bytes[] memory callDatas,
            string memory description
        ) = getProposeArgs(electionCount);

        proposalId = GovernorUpgradeable.propose(targets, values, callDatas, description);

        electionCount++;
    }
```