# English-Auction-Contract
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/token/ERC721/IERC721.sol";
import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/access/Ownable.sol";

contract EnglishAuction is Ownable {
    IERC721 public nft;
    uint256 public nftId;
    uint256 public startPrice;
    uint256 public endTime;
    address public highestBidder;
    uint256 public highestBid;

    event BidPlaced(address bidder, uint256 amount);
    event AuctionEnded(address winner, uint256 amount);

    constructor(address _nft, uint256 _nftId, uint256 _startPrice, uint256 _duration) Ownable(msg.sender) {
        nft = IERC721(_nft);
        nftId = _nftId;
        startPrice = _startPrice;
        endTime = block.timestamp + _duration;
        nft.transferFrom(msg.sender, address(this), _nftId);
    }

    function bid() external payable {
        require(block.timestamp < endTime, "Auction ended");
        require(msg.value > highestBid && msg.value >= startPrice, "Bid too low");

        if (highestBidder != address(0)) {
            payable(highestBidder).transfer(highestBid); // Refund previous
        }

        highestBidder = msg.sender;
        highestBid = msg.value;

        emit BidPlaced(msg.sender, msg.value);
    }

    function endAuction() external {
        require(block.timestamp >= endTime, "Not ended yet");
        require(highestBidder != address(0), "No bids");

        nft.safeTransferFrom(address(this), highestBidder, nftId);
        payable(owner()).transfer(address(this).balance);

        emit AuctionEnded(highestBidder, highestBid);
    }
}
