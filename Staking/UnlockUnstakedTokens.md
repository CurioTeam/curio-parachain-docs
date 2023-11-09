# Unlock Unstaked Tokens

Before you can unlock your previously staked tokens, you have to wait 7 days (in block time).

In the Polkadot JS Apps go to `Developer -> Extrinsics -> Submission`.

![Untitled](UnlockUnstakedTokens/unlockUnstaked.png)

## Collator

1. Select any account with enough balance to cover the transaction fee, which is around 0.005 CGT (the *using the selected account* field)
2. Select the appropriate extrinsic: `parachainStaking -> unlockUnstaked(target)`
3. Select the `Id` option (the *MultiAddress (LookupSource) field*)
4. Select your collator's CURIO address (the *Id: AccountId* field)
5. Sign and submit the extrinsic (the *Submit Transaction* button)

> 💡 Info\
You have unstaked tokens if you have either reduced your stake without increasing it for (at least) same amount afterwards or executing your exit request.

## Delegator

1. Select any account with enough balance to cover the transaction fee, which is around 0.005 CHT (the *using the selected account* field)
2. Select the appropriate extrinsic: `parachainStaking -> unlockUnstaked(target)`
3. Select the `Id` option (the *MultiAddress (LookupSource) field*)
4. Select the CURIO address you delegated from (the *Id: AccountId* field)
5. Sign and submit the extrinsic (the *Submit Transaction* button)


> 💡 Info\
Even if you have not exited, reduced or removed your delegation, you can still have unstaked tokens.
This can happen if either of the following events occurred:\
    - You were kicked out of your collator candidate's delegation pool because all current delegators have a higher stake\
    - Your collator candidate intentionally left the collator pool.