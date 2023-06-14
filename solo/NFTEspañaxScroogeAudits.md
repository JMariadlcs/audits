![](https://imgur.com/FxVcZQM.jpeg)

# Introduction

JMaria conducted a security review of the NFTEspaña marketplace, which was limited to a specific time frame. The review primarily focused on the security aspects of the application's implementation.

# About JMariadlcs - devScrooge

JMaria, also known as devScrooge on social networks, is a Telecommunications and Blockchain Engineer who currently works full-time as a DeFi developer and also performs security audits for smart contracts. He strives to make valuable contributions to the blockchain ecosystem and its protocols by dedicating his time and effort to conducting security research and performing thorough security reviews.

Socials:
[Webpage](https://www.devscrooge.com/)
[Twitter](https://twitter.com/notifications)
[LinkedIn](https://www.linkedin.com/in/jmariadlcs/)
[Github](https://github.com/JMariadlcs)
[ResearchGate](https://www.researchgate.net/profile/Jose-Maria-De-La-Cruz-Sanchez)

[Contact by email](mailto:0xdevscrooge@gmail.com)

# About NFTEspaña

NFTespaña, enables the purchase of NFTs without the need for cryptocurrencies, thus democratizing the trade of these digital assets for any user.

The main objective of NFTespaña is to facilitate the understanding of blockchain technology for collectors, creators, galleries, and brands through a simple and accessible language.

Within their marketplace, you will find the most prominent releases from Spanish-speaking artists and content creators, as well as stunning digital art and collectibles. The NFTespaña Marketplace has been specifically designed to cater to the Spanish-speaking community, providing a welcoming environment for those venturing into the world of NFTs.

# Severity classification

| Vulnerability Level   | Classification                                                                                                                     |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| [Critical](#Critical) | Easily exploitable by anyone, causing loss/manipulation of assets or data.                                                         |
| [High](#High)         | Arduously exploitable by a subset of addresses, causing loss/manipulation of assets or data.                                       |
| [Medium](#Medium)     | Inherent risk of future exploits that may or may not impact the smart contract execution due to current or future implementations. |
| [Low](#Low)           | Minor deviation from best practices.                                                                                               |
| [Gas](#Gas)           | Gas Optimization                                                                                                                   |

# Security Assessment Summary

| Project Name | NFTEspaña                                                                    |
| ------------ | ---------------------------------------------------------------------------- |
| Language     | Solidity                                                                     |
| Codebase     | https://github.com/minteandome/nft-espana/tree/main/contracts-project-finals |
| Commit       | e962ead                                                                      |

| Delivery Date     | 05/25/2023                     |
| ----------------- | ------------------------------ |
| Audit Methodology | Static Analysis, Manual Review |

**_review commit hash_ - [e962ead43c0391635198df4ff1a66d678d8d4d50](https://github.com/minteandome/nft-espana/commit/e962ead43c0391635198df4ff1a66d678d8d4d50)**

### Scope

The following smart contracts were in scope of the audit:

- `BidsNFTes.sol`

The following number of issues were found, categorized by their severity:

- Critical: 1 issues
- High: 2 issues
- Medium: 3 issues
- Low: 2 issues
- Informational: 6 issues
- Gas: 10 issues

# Findings Summary

| ID                | Title                                                                                                 |   Severity    |
| ----------------- | :---------------------------------------------------------------------------------------------------- | :-----------: |
| [C-01](#C-01)     | ETH sent can become stuck indefinitely                                                                |   Critical    |
| [H-01](#H-01)     | ERC721 tokens can become stuck indefinitely when sent to a non-receiver ERC721 contract               |     High      |
| [H-02](#H-02)     | Token transfer may fail but bidder's funds could still be transferred                                 |     High      |
| [M-01](#M-01)     | Centralization Risk for trusted owners                                                                |    Medium     |
| [M-02](#M-02)     | Seller can receive less funds than expected                                                           |    Medium     |
| [M-03](#M-03)     | Certain values for royalty fee can lead to DOS                                                        |    Medium     |
| [L-01](#L-01)     | Initialize function could be front-run                                                                |      Low      |
| [L-02](#L-02)     | Royalty fee is unbounded                                                                              |      Low      |
| [I-01](#I-01)     | Make sure to set the right value for `_wrappedToken`                                                  | Informational |
| [I-02](#I-02)     | Event is missing `indexed` fields                                                                     | Informational |
| [I-03](#I-03)     | Functions not used internally could be marked external                                                | Informational |
| [I-04](#I-04)     | Use `nonReentrant modifier`                                                                           | Informational |
| [I-05](#I-05)     | Consider incrementing `_bidId` after creating the bid                                                 | Informational |
| [I-06](#I-06)     | Consider checking that the current NFT Contract supports IERC2981 when creating the bid               | Informational |
| [GAS-01](#GAS-01) | Use assembly to check for `address(0)`                                                                |      Gas      |
| [GAS-02](#GAS-02) | `array[index] += amount` is cheaper than `array[index] = array[index] + amount` (or related variants) |      Gas      |
| [GAS-03](#GAS-03) | State variables should be cached in stack variables rather than re-reading them from storage          |      Gas      |
| [GAS-04](#GAS-04) | Use Custom Errors                                                                                     |      Gas      |
| [GAS-05](#GAS-05) | Don't initialize variables with default value                                                         |      Gas      |
| [GAS-06](#GAS-06) | Long revert strings                                                                                   |      Gas      |
| [GAS-07](#GAS-07) | Pre-increments and pre-decrements are cheaper than post-increments and post-decrements                |      Gas      |
| [GAS-08](#GAS-08) | Use `storage` instead of `memory` for structs/arrays                                                  |      Gas      |
| [GAS-09](#GAS-09) | Increments can be `unchecked` in for-loops                                                            |      Gas      |
| [GAS-10](#GAS-10) | Use != 0 instead of > 0 for unsigned integer comparison                                               |      Gas      |

---

# Detailed Findings

# Critical Issues

## <a name="C-01"></a>[C-01] ETH sent can be stuck forever

### Vulnerability description

It is possible for any user to send ETH to the contract so it becomes stuck in the contract indefinitely without the ability to withdraw the funds

### Proof of concept (POC)

Some functions like [addBidItem](https://github.com/minteandome/nft-espana/blob/main/contracts-project-finals/contracts/BidsNFTes.sol#L100) and [acceptBid](https://github.com/minteandome/nft-espana/blob/main/contracts-project-finals/contracts/BidsNFTes.sol#L138) are being declared as `payable` functions.

Although the ETH sent by any user using msg.value is never used in the contract, it is still possible for users to unintentionally or intentionally send ETH through these functions. In such cases, the sent amount of ETH would become permanently trapped in the contract since no function has been implemented for withdrawal, and the sent ETH remains unused throughout the contract.

```solidity=
function addBidItem(address nftContract, uint256 tokenId, uint256 bid, uint256 expiresAt) public payable  whenNotPaused returns (uint256){
```

```solidity=
function acceptBid(address nftContract, uint256 tokenId, uint256 bidId) public payable  whenNotPaused returns (uint256,uint256,uint256){
```

### Recommendations

As ETH is never used in the contract, remove the payable declaration from these functions.

---

# High Issues

## <a name="H-01"></a>[H-01] ERC721 tokens can become stuck indefinitely when sent to a non-receiver ERC721 contract

### Vulnerability description

It is possible that when accepting a bid, the ERC721 token is sent to a contract that does not support ERC-721. In such cases, the token can become permanently frozen inside the contract, and the transaction does not revert because it uses `transferFrom` instead of `safeTransferFrom`.

### Proof of concept (POC)

To purchase an ERC721 token, it is necessary for the user to create a bid. However, this bid does not necessarily have to be made by an Externally Owned Account (EOA) and can be created using a smart contract. In some cases, it is possible that the smart contract creating the bid does not support ERC721 for various reasons.

In the acceptBid function, the ERC721 token is transferred from the owner to the bidder using the transferFrom function. If the receiving contract does not support ERC721, the transaction will not revert, and the token will become permanently stuck inside the contract.

```solidity=
IERC721(bidItem.nftContract).transferFrom(msg.sender, bidItem.bider, bidItem.tokenId);
```

[Link to the code](https://github.com/minteandome/nft-espana/blob/main/contracts-project-finals/contracts/BidsNFTes.sol#L153)

### Recommendations

To ensure that the contract is only sent to receiers that accept ERC-721 tokens use the `safeTransferFrom` function instead of `transferFrom`.

## <a name="H-02"></a>[H-02] Token transfer may fail but bidder's funds could still be transferred

### Vulnerability description

The return value of the token transfer from the owner's address to the bidder is not checked, and it may fail without triggering a revert

### Proof of concept (POC)

Considering that ERC721 tokens implement `before` and `after` transfer hooks in `_transfer` by default, any behavior could really be expected from them, unless they are being deployed by the protocol itself, which is not the case. It is possible that the transfer of the token fails but does not revert. This would cause the owner to receive the amount from the bidder but not transfer the ERC721 token, resulting in a loss of funds.

```solidity=
IERC721(bidItem.nftContract).transferFrom(msg.sender, bidItem.bider, bidItem.tokenId);
```

[Link to the code](https://github.com/minteandome/nft-espana/blob/main/contracts-project-finals/contracts/BidsNFTes.sol#L153)

### Recommendations

To ensure that the transaction reverts if the ERC-721 transfer fails use the `safeTransferFrom` function instead of `transferFrom`.

---

# Medium Issues

## <a name="M-01"></a>[M-01] Centralization Risk for trusted owners

### Vulnerability description

Contracts have owners with privileged rights to perform admin tasks and need to be trusted not to perform malicious updates or modify important contract variables that can lead to unexpected contract behavior or a drain of funds.

### Impact

The owner is able to change the value for the `_royalty` variable without needing the approval of the protocol users by calling the `modifyRoyalties` function:

```solidity=
function modifyRoyalties(uint96 _royalty) public onlyOwner {
        royalty = _royalty;
    }
```

[Link to code](https://github.com/minteandome/nft-espana/blob/main/contracts-project-finals/contracts/BidsNFTes.sol#L62-L64)

The owner is also able to stop or enable the market at any point without needing the approval of the protocol users by calling the `stopMarket` and `enableMarket` functions.

```solidity=
function stopMarket() public onlyOwner{
        _pause();
        emit MarketDisabled();
    }
```

[Link to code](https://github.com/minteandome/nft-espana/blob/main/contracts-project-finals/contracts/BidsNFTes.sol#L66-L69)

```solidity=
function enableMarket() public onlyOwner {
        _unpause();
        emit MarketEnabled();
    }
```

[Link to code](https://github.com/minteandome/nft-espana/blob/main/contracts-project-finals/contracts/BidsNFTes.sol#L71-L74)

## <a name="M-02"></a>[M-02] Seller can receive less funds than expected

### Vulnerability description

The owner is able to change the royalty fees at any time. It is possible that the seller receives a less amount of tokens than expected due to a front-run by the owner to receive more funds or due to the transaction ordering inside a block.

### Proof of concept (POC)

To complete a 'bid transaction,' two transactions must be performed:

1. A user must create a bid for a specific token.
2. The owner of the token must accept the bid.

It is possible that when a user executes a bid offer transaction, the royalties are set to 'x' price. The owner of the token may expect to receive the `bid amount - x`, which is designated for royalties, and accept the bid. However, when the acceptance transaction is in the mempool, and the owner realizes it is a substantial bid, he may decide to front-run the transaction by creating a 'modify royalties fees' transaction and paying more gas. As a result, when the bid acceptance transaction is executed, the new royalties will be charged. Consequently, the owner of the token will receive less funds than expected.

The described situation could also occur unintentionally. The royalty fee can be changed at the same time that the bid is accepted, but the transaction ordering in the block, caused due to a higher paid amount of gas, may cause the variable to be changed before the acceptance transaction is executed.

### Recommendations

To ensure that the token owner receives the expected amount for their token, a modification can be made. Instead of calculating the royalty fees when the bid is accepted, they can be calculated when the bid offer is executed and included as a parameter in the bid struct. When the token owner accepts the bid, the royalties charged must not be recalculated, and the previous calculated royalty fees must be used. This modification guarantees that the token owner receives the same amount for their token as expected.

## <a name="M-03"></a>[M-03] Certain values for royalty fee can lead to DOS

### Vulnerability description

Setting certain values, particularly high ones, when accepting a bid can lead to a Denial-of-Service (DoS) scenario.

### Proof of concept (POC)

When a seller decides to accept a bid, executing the `acceptBid` function, the `splitPayment` function is internally called.

One of the tasks of the `splitPayment` is to calculate the corresponding royalty funds for the marketplay. This is done in the following way:

```solidity=
uint256 amountSeller = currentBid.bid - royaltyMarketPlace;
```

The variable `royaltyMarketPlace` is calculated by the `getRoyaltiesMarketPlace` function:

```solidity=
function getRoyaltiesMarketPlace(address nftContract, uint256 tokenId, uint256 bidId) public view returns (uint256){
        require(bidId <= _bidId , "Item not available");

        Bid storage currentBid = bidsByToken[nftContract][tokenId][bidId];
        uint256 royaltyAmount = (currentBid.bid * royalty) / _feeDenominator();//(PRICE * PERCENTE) / 10000

        return royaltyAmount;
    }
```

As it can be seen the variable `royalty` is used as an operand in a multiplication. The `royalty` variable can be set to really high values as there is no upper bounds for controlling this case:

```solidity=
 function modifyRoyalties(uint96 _royalty) public onlyOwner {
        royalty = _royalty;
    }
```

If the variable `royalty` is set to a high enough value for the `royaltyAmount` variable to be higher than `currentBid.bid`, the transaction will revert:

```solidity=
uint256 amountSeller = currentBid.bid - royaltyMarketPlace;
```

### Recommendations

Define an upper bound for the variable `royalty` so that it can not be set to a high value that can lead to a Denial of Service (DOS) on most transactions.

---

# Low Issues

## <a name="L-01"></a>[L-01] Initializers could be front-run

Initializers could be front-run, allowing an attacker to either set their own values, take ownership of the contract, and in the best case forcing a re-deployment

_Instances (3)_:

```solidity
File: BidsNFTes.sol

50:    function initialize (address _wrappedToken) public initializer {
```

[Link to code](https://github.com/minteandome/nft-espana/blob/main/contracts-project-finals/contracts/BidsNFTes.sol#L50-L57)

### Recommendations

Consider adding a `constructor` and calling the `_disableInitializers` function in the contract constructor. To do so, consider adding the [Initializable dependency](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/proxy/utils/Initializable.sol) from Openzeppelin.

```solidity=
constructor () {
    _disableInitializers () ;
}

```

## <a name="L-02"></a>[L-02] Royalty fee is unbounded

### Vulnerability description

The royalty fee for the marketplace is unbounded and can be set to a 100% or even higher. This can result on the seller not receiving any funds.

### Proof of concept (POC)

The modifyRoyalties function which is used to set the royalty fee is defined in the following way:

```solidity=
function modifyRoyalties(uint96 _royalty) public onlyOwner {
        royalty = _royalty;
    }
```

### Recommendations

Set a maximum upper bound for the royalty fee that can be set, ensuring it meets the requirement of not resulting in the user receiving 0 funds from the sale.

---

# Informational Issues

## <a name="I-01"></a>[I-01] Make sure to set the right value for `_wrappedToken`

### Vulnerability description

`_wrappedToken` is a key variable for the proper performance of the protocol. Please ensure that the correct value for the address token is set before deploying. As indicated by the team, this variable should contain the address value of `WMATIC`, which is `0x0d500b1d8e8ef31e21c99d1db9a6444d3adf1270`.

```solidity=
function initialize (address _wrappedToken) public initializer {
```

[Link to the code](https://github.com/minteandome/nft-espana/blob/main/contracts-project-finals/contracts/BidsNFTes.sol#L50)

## <a name="I-02"></a>[I-02] Event is missing `indexed` fields

Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

_Instances (3)_:

```solidity
File: BidsNFTes.sol

43:    event BidCreated(address indexed nftContract, uint256 tokenId, uint256 indexed bidId);

44:    event BidUpdated(address indexed nftContract, uint256 tokenId, uint256 indexed bidId);

45:    event BidDisactivated(address indexed nftContract, uint256 tokenId, uint256 indexed bidId);

```

[Link to code](https://github.com/minteandome/nft-espana/blob/main/contracts-project-finals/contracts/BidsNFTes.sol#L43.L45)

## <a name="I-03"></a>[I-03] Functions not used internally could be marked external instead of public

_Instances (11)_:

```solidity
File: BidsNFTes.sol

50:    function initialize (address _wrappedToken) public initializer {

62:    function modifyRoyalties(uint96 _royalty) public onlyOwner {

66:    function stopMarket() public onlyOwner{

71:    function enableMarket() public onlyOwner {

76:    function getRoyaltiesTotal() public view onlyOwner returns (uint256) {

80:    function getTotalSold() public view onlyOwner returns (uint256) {

100:    function addBidItem(address nftContract, uint256 tokenId, uint256 bid, uint256 expiresAt) public payable  whenNotPaused returns (uint256){

138:    function acceptBid(address nftContract, uint256 tokenId, uint256 bidId) public payable  whenNotPaused returns (uint256,uint256,uint256){

207:    function getBidsByToken(address nftContract, uint256 tokenId) public view returns (Bid[] memory){

220:    function modifyBid(address nftContract, uint256 tokenId, uint256 bidId, uint256 newBid, uint256 newExpiresAt) public  whenNotPaused{

269:    function getTimeStamp () public view returns (uint256){

```

[Link to code](https://github.com/minteandome/nft-espana/blob/main/contracts-project-finals/contracts/BidsNFTes.sol)

## <a name="I-04"></a>[I-04] Use `nonReentrant modifier`

### Description

A reentrancy scenario has not been detected, but I would recommend using the `nonReentrant` modifier in some of the main functions, as in `addBidItem` and `acceptBid` to ensure that reentrancy does not occur under any circumstances.

## <a name="I-05"></a>[I-05] Consider incrementing `_bidId` after creating the bid

### Description

The variable `_bidId` is currently incremented before creating each bid by calling `_incrementBidId()`. As a result, the first bid of the protocol will have an `Id` of 1. To align with best practices and facilitate future integrations, it is recommended to increment the `Id` before creating the bid, so that the first bid would have an `Id` of 0.

## <a name="I-06"></a>[I-06] Consider checking that the current NFT Contract supports IERC2981 when creating the bid

### Description

When accepting a bid for an NFT, it is currently checked whether the current NFT supports the IERC2981 type. If it doesn't, the execution reverts. To improve the user experience and avoid unnecessary waiting, it is recommended to perform this check when creating the bid for the current NFT instead of when accepting the bid. This way, if the NFT does not support IERC2981, the transaction will immediately revert upon creating the bid, and the user won't have to wait until the bid is accepted to encounter the revert.

```solidity=
require(IERC2981(currentBid.nftContract).supportsInterface(type(IERC2981).interfaceId), "Smart Contract does not support ERC2981 Interface");
```

---

# Gas Issues

## <a name="GAS-01"></a>[GAS-01] Use assembly to check for `address(0)`

_Saves 6 gas per instance_

_Instances (1)_:

```solidity
File: BidsNFTes.sol

187:    if(address(creator) != address(0)){

```

[Link to code](https://github.com/minteandome/nft-espana/blob/main/contracts-project-finals/contracts/BidsNFTes.sol#L187)

## <a name="GAS-02"></a>[GAS-02] `array[index] += amount` is cheaper than `array[index] = array[index] + amount` (or related variants)

When updating a value in an array with arithmetic, using `array[index] += amount` is cheaper than `array[index] = array[index] + amount`.
This is because you avoid an additonal `mload` when the array is stored in memory, and an `sload` when the array is stored in storage.
This can be applied for any arithmetic operation including `+=`, `-=`,`/=`,`*=`,`^=`,`&=`, `%=`, `<<=`,`>>=`, and `>>>=`.
This optimization can be particularly significant if the pattern occurs during a loop.

_Saves 28 gas for a storage array, 38 for a memory array_

_Instances (1)_:

```solidity
File: BidsNFTes.sol

113:    counterBidsByToken[nftContract][tokenId] = counterBidsByToken[nftContract][tokenId] + 1;

```

[Link to code](https://github.com/minteandome/nft-espana/blob/main/contracts-project-finals/contracts/BidsNFTes.sol#L113)

## <a name="GAS-03"></a>[GAS-03] State variables should be cached in stack variables rather than re-reading them from storage

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

_Saves 100 gas per instance_

_Instances (1)_:

```solidity
File: BidsNFTes.sol

115:    bidsIdsByToken[nftContract][tokenId].push(_bidId);

```

[Link to code](https://github.com/minteandome/nft-espana/blob/main/contracts-project-finals/contracts/BidsNFTes.sol#L115)

## <a name="GAS-04"></a>[GAS-04] Use Custom Errors

[Source](https://blog.soliditylang.org/2021/04/21/custom-errors/)
Instead of using error strings, to reduce deployment and runtime cost, you should use Custom Errors. This would save both deployment and runtime cost.

_Instances (14)_:

```solidity
File: BidsNFTes.sol

102:    require(block.timestamp < expiresAt, "Expire date must be after than current time");

140:    require( wrappedToken.balanceOf(bidsByToken[nftContract][tokenId][bidId].bider) >= bidsByToken[nftContract][tokenId][bidId].bid , "Insufficient funds");

141:    require( wrappedToken.allowance(bidsByToken[nftContract][tokenId][bidId].bider, address(this)) >= bidsByToken[nftContract][tokenId][bidId].bid , "Not authorized for move funds");

145:    require(bidItem.expiresAt >= block.timestamp, "Bid deprecated");

177:    require(amountSeller + royaltyMarketPlace + royaltyCreator == currentBid.bid, "Payment failed");

226:    require(bidsByToken[nftContract][tokenId][bidId].active , " Bid doesnt exists or inactive");

227:    require(bidsByToken[nftContract][tokenId][bidId].bider == payable(msg.sender) , "You are not original bider");

235:    require(block.timestamp < newExpiresAt, "Expire date must be after than current time");

236:    require(bidsByToken[nftContract][tokenId][bidId].active , " Bid doesnt exists or inactive");

237:    require(bidsByToken[nftContract][tokenId][bidId].bider == payable(msg.sender) , "You are not original bider");

238:    require(bidsByToken[nftContract][tokenId][bidId].expiresAt != newExpiresAt, "Bid with same expires date");

248:    require(bidId <= _bidId , "Item not available");

259:    require(bidId <= _bidId , "Item not available");

263:    require(IERC2981(currentBid.nftContract).supportsInterface(type(IERC2981).interfaceId), "Smart Contract does not support ERC2981 Interface");
```

[Link to code](https://github.com/minteandome/nft-espana/blob/main/contracts-project-finals/contracts/BidsNFTes.sol)

## <a name="GAS-05"></a>[GAS-05] Don't initialize variables with default value

_Instances (1)_:

```solidity
File: BidsNFTes.sol

211:    for( uint i = 0; i < counterBidsByToken[nftContract][tokenId] ; i++){

```

[Link to code](https://github.com/minteandome/nft-espana/blob/main/contracts-project-finals/contracts/BidsNFTes.sol#L211)

## <a name="GAS-06"></a>[GAS-06] Long revert strings

_Instances (3)_:

```solidity
File: BidsNFTes.sol

102:    require(block.timestamp < expiresAt, "Expire date must be after than current time");

235:    require(block.timestamp < newExpiresAt, "Expire date must be after than current time");

263:    require(IERC2981(currentBid.nftContract).supportsInterface(type(IERC2981).interfaceId), "Smart Contract does not support ERC2981 Interface");
```

[Link to code](https://github.com/minteandome/nft-espana/blob/main/contracts-project-finals/contracts/BidsNFTes.sol#L102)
[Link to code](https://github.com/minteandome/nft-espana/blob/main/contracts-project-finals/contracts/BidsNFTes.sol#L235)
[Link to code](https://github.com/minteandome/nft-espana/blob/main/contracts-project-finals/contracts/BidsNFTes.sol#L263)

## <a name="GAS-07"></a>[GAS-07] Pre-increments and pre-decrements are cheaper than post-increments and post-decrements

_Saves 5 gas per iteration_

_Instances (1)_:

```solidity
File: BidsNFTes.sol

211:    for( uint i = 0; i < counterBidsByToken[nftContract][tokenId] ; i++){

```

[Link to code](https://github.com/minteandome/nft-espana/blob/main/contracts-project-finals/contracts/BidsNFTes.sol#L211)

## <a name="GAS-08"></a>[GAS-08] Use `storage` instead of `memory` for structs/arrays

Using `memory` copies the struct or array in memory. Use `storage` to save the location in storage and have cheaper reads:

_Instances (1)_:

```solidity
File: BidsNFTes.sol

209:    Bid[] memory bids = new Bid[](counterBidsByToken[nftContract][tokenId]);

```

[Link to code](https://github.com/minteandome/nft-espana/blob/main/contracts-project-finals/contracts/BidsNFTes.sol#L209)

## <a name="GAS-09"></a>[GAS-09] Increments can be `unchecked` in for-loops

_Instances (1)_:

```solidity
File: BidsNFTes.sol

211:    for( uint i = 0; i < counterBidsByToken[nftContract][tokenId] ; i++){

```

[Link to code](https://github.com/minteandome/nft-espana/blob/main/contracts-project-finals/contracts/BidsNFTes.sol#L211)

## <a name="GAS-10"></a>[GAS-10] Use != 0 instead of > 0 for unsigned integer comparison

_Instances (4)_:

```solidity
File: BidsNFTes.sol

106:    if(bidersByToken[nftContract][tokenId][msg.sender] > 0){

156:    require (IERC1155(bidItem.nftContract).balanceOf(msg.sender, bidItem.tokenId) > 0 , "Seller is not the owner of NFT");

178:    if(royaltyMarketPlace > 0){

193:    if(amountSeller > 0 ){

```

[Link to code](https://github.com/minteandome/nft-espana/blob/main/contracts-project-finals/contracts/BidsNFTes.sol)

---

## Disclaimer

A comprehensive evaluation of smart contract security can never guarantee the absolute absence of vulnerabilities. Such an evaluation is subject to limitations in terms of time, resources, and expertise, and aims to identify as many potential vulnerabilities as possible. Therefore, even after the evaluation, it cannot be ensured that 100% security has been achieved, or that any potential issues with the smart contracts have been identified. It is strongly recommended to conduct subsequent security reviews, bug bounty programs, and on-chain monitoring to ensure continued security.
