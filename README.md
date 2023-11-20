# Solidity-bootcamp-5

### Task-5(Final)

In the previous lesson, you saw the final version of the contract we created. You also created a similar one with some differences. For an example your proposal also has a title, and you have a different logic for proposal state calculation

1. Your final task is to make sure that your final version of the contract is fully functioning. 
2. You should test it using the Remix IDE 
3. You should also deploy it to one of the blockchain's testnet that is compatible with EVM. The most popular choices in general are Goerli and Rinkby testnets in Ethereum and BNB Chain Testnet other than Ethereum, but again you can deploy in any testnet that uses EVM. 
4. After the deployment, submit the address of your first smart contract! 
5. If you haven't already created, create a GitHub repo with your code and submit the link of your repo.

Solution:

```sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;
import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Counters.sol";

contract ProposalContract {
    // ****************** Data ***********************

    //Owner
    address owner;

    using Counters for Counters.Counter;
    Counters.Counter private _counter;

    struct Proposal {
        string title; // Title of the proposal
        string description; // Description of the proposal
        uint256 approve; // Number of approve votes
        uint256 reject; // Number of reject votes
        uint256 pass; // Number of pass votes
        uint256 total_vote_to_end; // When the total votes in the proposal reaches this limit, proposal ends
        bool current_state; // This shows the current state of the proposal, meaning whether if passes or fails
        bool is_active; // This shows if others can vote to our contract
    }

    mapping(uint256 => Proposal) proposal_history; // Recordings of previous proposals
    address[] private voted_addresses;

    //constructor
    constructor() {
        owner = msg.sender;
        voted_addresses.push(msg.sender);
    }

    modifier onlyOwner() {
        require(msg.sender == owner);
        _;
    }

    modifier active() {
        require(proposal_history[_counter.current()].is_active == true);
        _;
    }

    modifier newVoter(address _address) {
        require(!isVoted(_address), "Address has not voted yet");
        _;
    }

    // ****************** Execute Functions ***********************

    function setOwner(address new_owner) external onlyOwner {
        owner = new_owner;
    }

    function create(
        string calldata _title,
        string calldata _description,
        uint256 _total_vote_to_end
    ) external onlyOwner {
        _counter.increment();
        proposal_history[_counter.current()] = Proposal(
            _title,
            _description,
            0,
            0,
            0,
            _total_vote_to_end,
            false,
            true
        );
    }

    function vote(uint8 choice) external active newVoter(msg.sender) {
        Proposal storage proposal = proposal_history[_counter.current()];
        uint256 total_vote = proposal.approve + proposal.reject + proposal.pass;

        voted_addresses.push(msg.sender);

        if (choice == 1) {
            proposal.approve += 1;
            proposal.current_state = calculateCurrentState();
        } else if (choice == 2) {
            proposal.reject += 1;
            proposal.current_state = calculateCurrentState();
        } else if (choice == 0) {
            proposal.pass += 1;
            proposal.current_state = calculateCurrentState();
        }

        if (
            (proposal.total_vote_to_end - total_vote == 1) &&
            (choice == 1 || choice == 2 || choice == 0)
        ) {
            proposal.is_active = false;
            voted_addresses = [owner];
        }
    }

    function terminateProposal() external onlyOwner active {
        proposal_history[_counter.current()].is_active = false;
    }

    function calculateCurrentState() private view returns (bool) {
    Proposal storage proposal = proposal_history[_counter.current()];

    uint256 totalVotes = proposal.approve + proposal.reject + proposal.pass;

    // Calculate the approval rate as a percentage
    uint256 approvalRate = (proposal.approve * 100) / totalVotes;

    // Determine the success threshold based on the total votes
    uint256 successThreshold;
    if (totalVotes <= 10) {
        successThreshold = 60; // 60% approval required for proposals with 10 or fewer votes
    } else {
        successThreshold = 50; // 50% approval required for proposals with more than 10 votes
    }

    // Check if the approval rate meets the success threshold
    return approvalRate >= successThreshold;
}


    // ****************** Query Functions ***********************

    function isVoted(address _address) public view returns (bool) {
        for (uint256 i = 0; i < voted_addresses.length; i++) {
            if (voted_addresses[i] == _address) {
                return true;
            }
        }
        return false;
    }

    function getCurrentProposal() external view returns (Proposal memory) {
        return proposal_history[_counter.current()];
    }

    function getProposal(uint256 number) external view returns (Proposal memory) {
        return proposal_history[number];
    }
}

```

![image](https://github.com/Arya-0902/Solidity-bootcamp-5/assets/99527147/8d26b85c-11e7-4eac-9b8b-fa328b711c09)

Contract Address: 0x9396B453Fad71816cA9f152Ae785276a1D578492
Successfully Deployed Smart contract on the "Remix VM - Mainnet Fork"
