# 🪙 SimpleAuction Smart Contract

A simple **on-chain auction system** built with Solidity for learning and demonstration purposes.  
This contract enables users to participate in a basic timed auction using Ethereum (ETH) directly on the blockchain.

---


<img width="1827" height="853" alt="image" src="https://github.com/user-attachments/assets/3e269718-8505-46be-8c40-81527c8f1e65" />


## 📖 Project Description

**SimpleAuction** is a beginner-friendly smart contract that demonstrates the fundamentals of building a decentralized auction system on Ethereum.  
It allows users to place bids in ETH, tracks the highest bidder, manages refunds for overbid participants, and automatically ends after a set duration.

This project is ideal for developers who want to understand:
- ETH transactions within smart contracts  
- Safe fund withdrawal patterns  
- Event-driven design in Solidity  

---
![Uploading image.png…]()

## ⚙️ What It Does

1. **Deploy** — The contract is deployed by a seller (the deployer), and the auction lasts for **1 day** by default.  
2. **Bid** — Anyone can place a bid with ETH.  
3. **Refunds** — If a new higher bid is placed, the previous highest bidder can withdraw their ETH safely.  
4. **End Auction** — After the time expires, the seller can end the auction and receive the highest bid.

---

## 🌟 Features

- ✅ Fixed 1-day auction duration (no inputs required)  
- ✅ Tracks the **highest bid** and **highest bidder**  
- ✅ Secure **withdraw** pattern to prevent re-entrancy  
- ✅ Emits events for `HighestBidIncreased` and `AuctionEnded`  
- ✅ Fully transparent and verifiable on-chain logic  

---

## 🔗 Deployed Smart Contract

- **Network:** Ethereum  
- **Verified Source:** [View on Sourcify](https://repo.sourcify.dev/11142220/0xA17048cc69F67C8A05226049718109F15fbD21d9)  
- **Address:** `0xA17048cc69F67C8A05226049718109F15fbD21d9`

---

## 🧠 Smart Contract Code

```solidity
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
