// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract Project {
    struct Campaign {
        address payable creator;
        string title;
        string description;
        uint256 goalAmount;
        uint256 raisedAmount;
        uint256 deadline;
        bool isActive;
        bool goalReached;
    }
    
    mapping(uint256 => Campaign) public campaigns;
    mapping(uint256 => mapping(address => uint256)) public contributions;
    mapping(uint256 => address[]) public contributors;
    
    uint256 public campaignCounter;
    uint256 public constant MINIMUM_CONTRIBUTION = 0.01 ether;
    
    event CampaignCreated(
        uint256 indexed campaignId,
        address indexed creator,
        string title,
        uint256 goalAmount,
        uint256 deadline
    );
    
    event ContributionMade(
        uint256 indexed campaignId,
        address indexed contributor,
        uint256 amount
    );
    
    event FundsWithdrawn(
        uint256 indexed campaignId,
        address indexed creator,
        uint256 amount
    );
    
    event RefundIssued(
        uint256 indexed campaignId,
        address indexed contributor,
        uint256 amount
    );
    
    modifier onlyCreator(uint256 _campaignId) {
        require(campaigns[_campaignId].creator == msg.sender, "Only campaign creator can call this");
        _;
    }
    
    modifier campaignExists(uint256 _campaignId) {
        require(_campaignId < campaignCounter, "Campaign does not exist");
        _;
    }
    
    modifier campaignActive(uint256 _campaignId) {
        require(campaigns[_campaignId].isActive, "Campaign is not active");
        require(block.timestamp < campaigns[_campaignId].deadline, "Campaign deadline has passed");
        _;
    }
    
    // Core Function 1: Create Campaign
    function createCampaign(
        string memory _title,
        string memory _description,
        uint256 _goalAmount,
        uint256 _durationInDays
    ) external returns (uint256) {
        require(_goalAmount > 0, "Goal amount must be greater than 0");
        require(_durationInDays > 0, "Duration must be greater than 0");
        require(bytes(_title).length > 0, "Title cannot be empty");
        
        uint256 deadline = block.timestamp + (_durationInDays * 1 days);
        
        campaigns[campaignCounter] = Campaign({
            creator: payable(msg.sender),
            title: _title,
            description: _description,
            goalAmount: _goalAmount,
            raisedAmount: 0,
            deadline: deadline,
            isActive: true,
            goalReached: false
        });
        
        emit CampaignCreated(campaignCounter, msg.sender, _title, _goalAmount, deadline);
        
        campaignCounter++;
        return campaignCounter - 1;
    }
    
    // Core Function 2: Contribute to Campaign
    function contribute(uint256 _campaignId) 
        external 
        payable 
        campaignExists(_campaignId) 
        campaignActive(_campaignId) 
    {
        require(msg.value >= MINIMUM_CONTRIBUTION, "Contribution too small");
        require(campaigns[_campaignId].creator != msg.sender, "Creator cannot contribute to own campaign");
        
        Campaign storage campaign = campaigns[_campaignId];
        
        // Add to contributions mapping
        if (contributions[_campaignId][msg.sender] == 0) {
            contributors[_campaignId].push(msg.sender);
        }
        contributions[_campaignId][msg.sender] += msg.value;
        
        // Update campaign
        campaign.raisedAmount += msg.value;
        
        // Check if goal is reached
        if (campaign.raisedAmount >= campaign.goalAmount) {
            campaign.goalReached = true;
        }
        
        emit ContributionMade(_campaignId, msg.sender, msg.value);
    }
    
    // Core Function 3: Withdraw Funds (Creator) or Get Refund (Contributors)
    function withdrawFunds(uint256 _campaignId) 
        external 
        campaignExists(_campaignId) 
        onlyCreator(_campaignId) 
    {
        Campaign storage campaign = campaigns[_campaignId];
        require(campaign.goalReached || block.timestamp >= campaign.deadline, "Campaign still active and goal not reached");
        require(campaign.raisedAmount > 0, "No funds to withdraw");
        
        uint256 amount = campaign.raisedAmount;
        campaign.raisedAmount = 0;
        campaign.isActive = false;
        
        campaign.creator.transfer(amount);
        
        emit FundsWithdrawn(_campaignId, campaign.creator, amount);
    }
    
    function getRefund(uint256 _campaignId) external campaignExists(_campaignId) {
        Campaign storage campaign = campaigns[_campaignId];
        require(block.timestamp >= campaign.deadline, "Campaign still active");
        require(!campaign.goalReached, "Campaign goal was reached, no refunds");
        require(contributions[_campaignId][msg.sender] > 0, "No contribution found");
        
        uint256 refundAmount = contributions[_campaignId][msg.sender];
        contributions[_campaignId][msg.sender] = 0;
        
        payable(msg.sender).transfer(refundAmount);
        
        emit RefundIssued(_campaignId, msg.sender, refundAmount);
    }
    
    // View Functions
    function getCampaignDetails(uint256 _campaignId) 
        external 
        view 
        campaignExists(_campaignId) 
        returns (
            address creator,
            string memory title,
            string memory description,
            uint256 goalAmount,
            uint256 raisedAmount,
            uint256 deadline,
            bool isActive,
            bool goalReached
        ) 
    {
        Campaign storage campaign = campaigns[_campaignId];
        return (
            campaign.creator,
            campaign.title,
            campaign.description,
            campaign.goalAmount,
            campaign.raisedAmount,
            campaign.deadline,
            campaign.isActive,
            campaign.goalReached
        );
    }
    
    function getContributorCount(uint256 _campaignId) 
        external 
        view 
        campaignExists(_campaignId) 
        returns (uint256) 
    {
        return contributors[_campaignId].length;
    }
    
    function getMyContribution(uint256 _campaignId) 
        external 
        view 
        campaignExists(_campaignId) 
        returns (uint256) 
    {
        return contributions[_campaignId][msg.sender];
    }
}
