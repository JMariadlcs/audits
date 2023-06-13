# Ajna Finance

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

# [H-01] User's funds can be unclaimed forever

## Lines of Code

- https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L114-L125
- https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L412
- https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol#L811-L821

## Vulnerability Details

### Description

If a user tries to claim his earned rewards during an epoch in which the `RewardsManager.sol` contract holds an amount of Ajna tokens that is less than the amount of earned rewards, the difference will be unclaimable for the user forever.

### Impact

The supossed behaviour of the contract is that each user can only claim his earned rewards related with a `tokenId_` once in each epoch. In order to ensure this behaviour when a user calls to the external `claimRewards` function the following is checked:

```bash=
if (isEpochClaimed[tokenId_][epochToClaim_]) revert AlreadyClaimed();
```

In the case that the user has not claimed before in the same epoch his earned rewards related with `tokenId_`, the execution continues and the `_claimRewards` function is called.
Inside `_claimRewards` the amount of `rewardsEarned` is computed by calling the function `_calculateAndClaimRewards`. This function also sets to true the above mentioned mapping to mark that the user can no longer claim rewards during this epoch:

```bash=
isEpochClaimed[tokenId_][epoch] = true;
```

The execution of `_claimRewards` continues and the function `_transferAjnaRewards` is executed with the `rewardsEarned` as parameter, which refers to the total amount of earned rewards by the user.

The implementation of `_transferAjnaRewards` is the following one:

```solidity
function _transferAjnaRewards(uint256 rewardsEarned_) internal {
        // check that rewards earned isn't greater than remaining balance
         //if remaining balance is greater, set to remaining balance
        uint256 ajnaBalance = IERC20(ajnaToken).balanceOf(address(this));
        if (rewardsEarned_ > ajnaBalance) rewardsEarned_ = ajnaBalance;

        if (rewardsEarned_ != 0) {
            // transfer rewards to sender
            IERC20(ajnaToken).safeTransfer(msg.sender, rewardsEarned_);
        }
    }
```

We can see that if the amount of `rewardsEarned_` is less than `IERC20(ajnaToken).balanceOf(address(this))` the user will receive only the balance of the ajnaTokens held by the contract. The difference of the amount between the rewardsEarned and amount is not recorded anywhere so the user can not claim this amount in later epochs. The user can neither call the `_claimRewards` function later in the same epoch if the contract receives enough ajnaToken balance to cover the remaining rewards because `isEpochClaimed[tokenId_][epoch] = true` was set to true before.

### Proof of concept

Imagine the situation where a user has 100 Ajna tokens earned as rewards.
Imagine that user is claiming at epoch 1. The contract balance of Ajna tokens at epoch 1 is 50 Ajna tokens.

When the user claims his rewards, `isEpochClaimed[tokenId_][epoch]` is set to true so he can not longer claim at epoch 1.

When executing `_transferAjnaRewards`, as `rewardsEarned_ > ajnaBalance` he will be transfered the total balance of Ajna Tokens in the contract, which is 50 tokens. He is receiving 50 tokens less than the corresponding. If the contract receives 50 more Ajna tokens in epoch 1 but after the first claim, the user can not claim again as `isEpochClaimed[tokenId_][epoch]` is equal to `true`.

If the user later claims in other epoch. The computation of the rewards is done without taking into account that he had received 50 less tokens than expected in epoch 1. So he has lost 50 Ajna tokens as reward.

### Tools Used

Manual review

### Recommended Mitigation Steps

Implement a mechanism that, when the described situation takes place and the user has received less rewards than the total corresponding amount, records the difference so that when the computation of rewards is done in later epochs and the user has claimed again, the unreceived amount from the first epoch is taken into account so that he can receive it.
