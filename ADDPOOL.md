# MCN Multi-farming Integration

If you would like to integrate your project into the MCN farming ecosystem, you can submit a proposal [here](https://snapshot.org/#/mcnventures.eth).

This document contains steps for MCN's tech team.

## Gather specific data

Ensure that you have the following information before you begin:

1. Project token address
2. Amount of each type of token to be supplied to each pool
3. Reward emission rate
4. Reward start time
5. Token source address(es) (wallets that will be transferring the tokens to the contract)
6. Token price source

## Add the new tokens to the contract

Make the following contract calls for the relevant pools (see [this section](#contract-interaction-documentation) for detailed documentation).
1. `addPoolsAndAllowBonus`
2. `approve` the contract as a spender for your new token
3. `addBonus`

## Update the front-end

1. Update supportedPools in [constants.js](https://github.com/MCN-Ventures/mcn-farming-frontend/blob/main/src/sushi/lib/constants.js) to reflect the state of the updated/new pool(s).
2. Update getPrices in [utils.js](https://github.com/MCN-Ventures/mcn-farming-frontend/blob/main/src/sushi/utils.js) to fetch the price for the new token for APR calculations.

## No-Code Walkthrough Example

Project XYZ has been approved for integration into the MCN farming ecosystem. They have worked with the MCN partners and agreed to the following:
- XYZ will provide 50,000 XYZ tokens to the MCN staking pool to be distributed over 180 days.
- XYZ will provide 15,000 XYZ tokens to the MCN-USDC staking pool to be distributed over 180 days.
- MCN will provide 150 MCN tokens to a new XYZ staking pool to be distributed over 21 days.

**TODO: Finish writing out this example with screenshots from Etherscan**

## Contract interaction documentation

### `addPoolsAndAllowBonus`

This function must be called by the contract owner if you would like to create a new pool or use a new token source address to fund an existing pool.

**Parameters**

*_lpTokens* - a list of token addresses for the pool(s) that you would like to create / edit

*_bonusTokenAddrs* - a list of token addresses for the rewards that you would like to add / edit

*_authorizers* - a list of source addresses that will be transferring the reward tokens to the contract / adjusting the pool(s)

**Example**

This call creates a new pool to stake USDC and receive MCN and WETH as rewards funded by the MCN GP multisig wallet.

```
addPoolsAndAllowBonus(
  "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
  ["0xD91E9a0fEf7C0fa4EBdAF4d0aCF55888949A2a9b", "0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2"],
  "0x01a628361BB8378675C60Af503dd6199876746D7"
)
```

### `addBonus`

This function is used to add rewards to a new or existing pool that is not already emitting the given reward. Should be called by the reward token source wallet.

*Note: the caller must have been previously authorized to call this function via `addPoolsAndAllowBonus`*

**Parameters**

*_lpToken* - the stake token address for the pool you would like to add / edit

*_bonusTokenAddr* - the reward token address

*_startTime* - the time you would like to start the rewards in epoch time (must be a future time)

*_weeklyRewards* - the number of tokens to be emitted every week (in wei)

*_transferAmount* - the number of tokens to be transferred to the contract and used for this reward (in wei)

**Example**

This call adds 300 MCN to the USDC staking pool to be emitted over the course of 21 days starting Mon, 27 Sep 2021 8:00:00 GMT.

```
addBonus(
  "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
  "0xD91E9a0fEf7C0fa4EBdAF4d0aCF55888949A2a9b",
  1632729600,
  100000000000000000000,
  300000000000000000000
)
```

Be sure to note the number of decimals in the reward token and adjust the parameters accordingly.

weeklyRewards = transferAmount / rewardDurationInDays * 7

### `updateBonus`

This function can be used to change a pool's start time (if not already started) or change a pool's reward emission rate.

*Note: the caller must have been previously authorized to call this function via `addPoolsAndAllowBonus`*

**Parameters**

*_lpToken* - the stake token address for the pool you would like edit

*_bonusTokenAddr* - the reward token address

*_weeklyRewards* - the number of tokens to be emitted every week (in wei)

*_startTime* - set to 0 if you are changing weeklyRewards, otherwise set to desired start time in epoch time (must be a future time)

### `extendBonus`

This function can be used to add additional rewards to a pool that is a pool that is already emitting rewards.

*Note: the caller must have been previously authorized to call this function via `addPoolsAndAllowBonus`*

**Parameters**

*_lpToken* - the stake token address for the pool you would like to edit

*_poolBonusId* - the index of the bonus token to be added for this pool

*_bonusTokenAddr* - the reward token address

*_transferAmount* - the number of tokens to be transferred to the contract and used for this reward (in wei)

The poolBonusId can be discerned by passing *_lpToken* to `getPool` and getting the nth address in the returned tuple that corresponds to *_bonusTokenAddr* (where n is indexed from 0)
