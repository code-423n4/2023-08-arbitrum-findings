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

3. 

It is possible to have same weight for 2 nominees. If two addresses have the same weight, the one that appears earlier in the nominees array will be selected.

```soldiity
 function selectTopNominees(address[] memory nominees, uint240[] memory weights, uint256 k)
        public
        pure
        returns (address[] memory)
    {
        if (nominees.length != weights.length) {
            revert LengthsDontMatch(nominees.length, weights.length);
        }
        if (nominees.length < k) {
            revert NotEnoughNominees(nominees.length, k);
        }

        uint256[] memory topNomineesPacked = new uint256[](k);

        for (uint16 i = 0; i < nominees.length; i++) {
            uint256 packed = (uint256(weights[i]) << 16) | i;

            if (topNomineesPacked[0] < packed) {
                topNomineesPacked[0] = packed;
                LibSort.insertionSort(topNomineesPacked);
            }
        }

        address[] memory topNomineesAddresses = new address[](k);
        for (uint16 i = 0; i < k; i++) {
            topNomineesAddresses[i] = nominees[uint16(topNomineesPacked[i])];
        }

        return topNomineesAddresses;
    }
```

Let's say k = 6

A gets 200 (Selected)
B gets 180 (Selected)
C gets 160 (Selected)
D gets 140 (Selected)
E gets 130 (Selected)
F gets 120 (Selected)
G gets 120 (Not get selected)

F and G have the same weight, yet G won't get selected because he was added later than F. 

There are some solution would fix this. One of them is to request NomineeElectionGovernor to determine who gets elected manually if this situation happened.

4.

```solidity
if (VoteType(support) == VoteType.Abstain) {
  revert AbstainDisallowed();
}
```

can be removed from SecurityCouncilMemberRemovalGovernor#_countVote because the vote can never be abstain

See

```solidity
  function COUNTING_MODE()
        public
        pure
        virtual
        override(GovernorCountingSimpleUpgradeable, IGovernorUpgradeable)
        returns (string memory)
    {
        return "support=for,against&quorum=for";
    }
```

where the counting mode support for and against votes only