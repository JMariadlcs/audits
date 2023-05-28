![](https://imgur.com/FxVcZQM.jpeg)

---

# Introduction

JMaria conducted a security review of the [undisclosed name] protocol, which was limited to a specific time frame. The review primarily focused on the security aspects of the application's implementation.

# About JMariadlcs - devScrooge

JMaria, also known as devScrooge on social networks, is a Telecommunications and Blockchain Engineer who currently works full-time as a DeFi developer and also performs security audits for smart contracts. He strives to make valuable contributions to the blockchain ecosystem and its protocols by dedicating his time and effort to conducting security research and performing thorough security reviews.

Socials:
[Webpage](https://www.devscrooge.com/)
[Twitter](https://twitter.com/notifications)
[LinkedIn](https://www.linkedin.com/in/jmariadlcs/)
[Github](https://github.com/JMariadlcs)
[ResearchGate](https://www.researchgate.net/profile/Jose-Maria-De-La-Cruz-Sanchez)

[Contact by email](mailto:0xdevscrooge@gmail.com)

# About [Undisclosed] protocol

The audited protocol main functionallity is to allow user to mint and transfer NFTs automatically using their frontend.

# Severity classification

| Vulnerability Level   | Classification                                                                                                                     |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| [Critical](#Critical) | Easily exploitable by anyone, causing loss/manipulation of assets or data.                                                         |
| [High](#High)         | Arduously exploitable by a subset of addresses, causing loss/manipulation of assets or data.                                       |
| [Medium](#Medium)     | Inherent risk of future exploits that may or may not impact the smart contract execution due to current or future implementations. |
| [Low](#Low)           | Minor deviation from best practices.                                                                                               |
| [Gas](#Gas)           | Gas Optimization                                                                                                                   |

### Scope

The following smart contracts were in scope of the audit:

- [Undisclosed name]

The following number of issues were found, categorized by their severity:

- Critical: 2 issues
- High: 1 issues
- Medium: 0 issues
- Low: 1 issues
- Informational: 1 issues
- Gas: 6 issues

# Findings Summary

| ID                | Title                                                                                  |   Severity    |
| ----------------- | :------------------------------------------------------------------------------------- | :-----------: |
| [C-01](#C-01)     | Any user can mint unlimited amount of tokens for free                                  |   Critical    |
| [C-02](#C-02)     | Seller is receiving funds without transfering any token                                |   Critical    |
| [H-01](#H-01)     | Reentrancy attack is possible                                                          |     High      |
| [L-01](#L-01)     | Unsafe use of `.transfer` for sending ETH                                              |      Low      |
| [I-01](#I-01)     | Functions not used internally could be marked external                                 | Informational |
| [GAS-01](#GAS-01) | Cache array length outside of loop                                                     |      Gas      |
| [GAS-01](#GAS-01) | Cache array length outside of loop                                                     |      Gas      |
| [GAS-02](#GAS-02) | Use calldata instead of memory for function arguments that do not get mutated          |      Gas      |
| [GAS-03](#GAS-03) | Use Custom Errors                                                                      |      Gas      |
| [GAS-04](#GAS-04) | Don't initialize variables with default value                                          |      Gas      |
| [GAS-05](#GAS-05) | Pre-increments and pre-decrements are cheaper than post-increments and post-decrements |      Gas      |
| [GAS-06](#GAS-06) | Increments can be `unchecked` in for-loops                                             |      Gas      |

---

# Detailed Findings

# Critical issues

## <a name="C-01"></a>[C-01] Any user can mint unlimited amount of tokens for free

### Description

The function `mintTokenByCreator` can be called by any user without any restriction and mint an unlimited amount of tokens to the caller without paying any amount for them.

### Proof of concept (POC)

The function `mintTokenByCreator` should be used only by the creator as the NatSpec comment indicates:`@notice only creator can mint private nfts`. However, the function does not include any restriction access modifier, there fore any user can execute the function and mint tokens themselves for free.

```solidity=
function mintTokenByCreator(
        uint256[] memory tokenIds,
        string[] memory tokenUris
    ) public checklengthOfTokenIdsAnduris(tokenIds, tokenUris) {
        for (uint256 i = 0; i < tokenIds.length; i++) {
            _mintToken(msg.sender, tokenIds[i], tokenUris[i]);
        }
    }
```

### Recommendations

Add an `onlyOwner` modifier if the desired behaviour is that only the owner can mint tokens for free.

## <a name="C-02"></a>[C-02] Seller is receiving funds without transfering any token

### Description

The behaviour of the `mintTokenAndTransfer` should be exchanging NFTs from the seller with funds from the buyer. However, the seller is receiving funds without transfering any token

### Proof of concept (POC)

The function `mintTokenAndTransfer` allows the seller to receive funds without the need to transfer his tokens.
The function implement the following loop:

```solidity=
 for (uint256 i = 0; i < mintTokenTransfer.tokenIds.length; i++) {
            _mintToken(
                mintTokenTransfer.seller,
                mintTokenTransfer.tokenIds[i],
                mintTokenTransfer.tokenUris[i]
            );
            _distributeRoyality(
                mintTokenTransfer.seller,
                mintTokenTransfer.price,
                mintTokenTransfer.adminCommission,
                mintTokenTransfer.tokenIds[i],
                mintTokenTransfer.adminWallet
            );
            _safeTransfer(
                mintTokenTransfer.seller,
                msg.sender,
                mintTokenTransfer.tokenIds[i],
                ""
            );
        }
```

As it can be seen, the first executed function is used to mint tokens directly to the seller, later on, the same minted tokens are transfered from the seller to the buyer, in this case `msg.sender`. This means that the transfered tokens are previously free minted to the seller which actually is not providing any value to the buyer but he is receiving funds when `_distributeRoyality` is executed.

### Recommendations

Change the implementation of the function to execute itself as the described behaviour.

---

# High issues

## <a name="H-01"></a>[H-01] Reentrancy attack is possible

### Description

The function `mintTokenAndTransfer` is used to mint tokens to the user and distribute the paid funds. The problem is that the tokens are minted to the user before the funds are paid and as the function does not implement a `nonReentrant` modifier it is possible that an attacker can execute a reentrancy attack.

### Recommendations

As the contract is already defined as `ReentrancyGuard`, add the `nonReentrant` modifier to the functions to avoid reentrancy attacks.

---

# Low Issues

## <a name="L-01"></a>[L-01] Unsafe use of `.transfer` for sending ETH

### Vulnerability description

The contract employs a low-level `.transfer` method to send ETH, but there are certain drawbacks when the recipient is a smart contract. Specifically, the transfer will fail under the following conditions:

1. The withdrawer smart contract does not have a payable fallback or receive function implemented.

2. The smart contract's payable fallback function consumes more than 2300 gas, which can occur when using a proxy that increases gas usage during the call.

### Recommendations

It is worth noting that the sendValue function in the Address library of OpenZeppelin Contract can be utilized to transfer Ether without the restriction of 2300 gas units. To mitigate the risks of reentrancy that may arise from using this function, it is important to closely adhere to the "Check-effects-interactions" pattern and employ OpenZeppelin Contract's ReentrancyGuard contract. For additional information on why the use of Solidity's transfer is no longer recommended, you can refer to the following articles:

1. [Stop using Solidity’s transfer now](https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/)
2. [Reentrancy after Istanbul](https://blog.openzeppelin.com/reentrancy-after-istanbul)

---

# Informational Issues

## <a name="I-01"></a>[I-01] Functions not used internally could be marked external

---

# Gas issues

## <a name="GAS-01"></a>[GAS-01] Cache array length outside of loop

If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

## <a name="GAS-02"></a>[GAS-02] Use calldata instead of memory for function arguments that do not get mutated

Mark data types as `calldata` instead of `memory` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as `calldata`. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies `memory` storage.

## <a name="GAS-03"></a>[GAS-03] Use Custom Errors

[Source](https://blog.soliditylang.org/2021/04/21/custom-errors/)
Instead of using error strings, to reduce deployment and runtime cost, you should use Custom Errors. This would save both deployment and runtime cost.

## <a name="GAS-04"></a>[GAS-04] Don't initialize variables with default value

## <a name="GAS-05"></a>[GAS-05] Pre-increments and pre-decrements are cheaper than post-increments and post-decrements

## <a name="GAS-06"></a>[GAS-06] Increments can be `unchecked` in for-loops

---

## Disclaimer

A comprehensive evaluation of smart contract security can never guarantee the absolute absence of vulnerabilities. Such an evaluation is subject to limitations in terms of time, resources, and expertise, and aims to identify as many potential vulnerabilities as possible. Therefore, even after the evaluation, it cannot be ensured that 100% security has been achieved, or that any potential issues with the smart contracts have been identified. It is strongly recommended to conduct subsequent security reviews, bug bounty programs, and on-chain monitoring to ensure continued security.
