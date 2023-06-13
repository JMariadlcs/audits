# [L-01] Overflow during safeCasting can lead to DOS

## Lines of Code

- https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L142
- https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L143

## Vulnerability Details

### Description

These lines perform a safe casting of `block.number` to `uint48`. If this `block.number` is ever reached, the protocol will become useless.

### Proof of concept

This `SafeCast checks` that block.number is not higher than `2^48`, which is `281474976710655`. Ajna is a multichain project. The current block height, for example on Fantom, is `61857099`. So, a lot of time has to pass for an overflow and revert to occur. However, if time passes or the chain changes the way it operates and the block creation speed increases such that block.number reaches `2^48`, the protocol will become useless due to this revert.

### Tools Used

Manual review

### Recommended Mitigation Steps

Do not downcast `block.number` to `uint48`.

# [L-02] Maximum number of iterations in NatSpec comment is not implemented

## Lines of Code

- https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol#L412

## Vulnerability Details

### Description

The `NatSpec` comment of the function `_validateSlate` states the following:

`Only iterates through a maximum of 10 proposals that made it through both voting stages.`

But the implementation of the `_validSlate` function does not check the number of iterations. The number of iterations performed are the corresponding to the `uint256 numProposalsInSlate_` parameter of the function, there is not a maximum number for the iterations.

### Tools Used

Manual review

### Recommended Mitigation Steps

Include a check to ensure that `numProposalsInSlate_` is a maximum of `10`:

```bash=
require(numProposalsInSlate_ < 10);
```
