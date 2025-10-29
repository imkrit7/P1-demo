
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SimpleAuction {
    address payable public seller;
    uint public auctionEndTime;

    address public highestBidder;
    uint public highestBid;

    bool public ended;

    mapping(address => uint) public pendingReturns;

    event HighestBidIncreased(address bidder, uint amount);
    event AuctionEnded(address winner, uint amount);

    constructor() {
        seller = payable(msg.sender);           // The deployer is the seller
        auctionEndTime = block.timestamp + 1 days;  // Auction lasts 1 day
    }

    // Bid with ETH
    function bid() external payable {
        require(block.timestamp < auctionEndTime, "Auction ended");
        require(msg.value > highestBid, "Bid too low");

        if (highestBid != 0) {
            // Refund previous highest bidder
            pendingReturns[highestBidder] += highestBid;
        }

        highestBidder = msg.sender;
        highestBid = msg.value;
        emit HighestBidIncreased(msg.sender, msg.value);
    }

    // Withdraw any overbid amount
    function withdraw() external {
        uint amount = pendingReturns[msg.sender];
        require(amount > 0, "No funds to withdraw");

        pendingReturns[msg.sender] = 0;
        payable(msg.sender).transfer(amount);
    }

    // End the auction and send funds to the seller
    function endAuction() external {
        require(block.timestamp >= auctionEndTime, "Auction not yet ended");
        require(!ended, "Auction already ended");

        ended = true;
        emit AuctionEnded(highestBidder, highestBid);

        seller.transfer(highestBid);
    }
}
