# Building your first hook

A really simple onchain "points program"

Assume you launch some sort of token - $XYZ

You set up a pool on Uniswap for ETH/XYZ

your goal is to incentivize people to buy XYZ for ETH in that pool

- everytime someone buys XYZ for ETH in that pool, our hook will assign points to an address the user says they want to receive the points on

points themselves will act as an ERC-20 token onchain, we're gonna mint $POINTS tokens to people who buy XYZ for ETH on Uniswap

> NOTE: this is a proof of concept and not a production ready design

## How many points to give out per trade?

For every swap that happens, we're gonna give out 20% of the amount of ETH that was used to purchase XYZ as points.

For e.g. if someone swaps 5 ETH for XYZ, they get 1 POINT token

## General purpose vs. specific hooks

As developers, you have a choice to build hooks that either only work for one specific pool, or you can build a hook that is more general and can be used by other teams and other pool creators and such.

We're gonna build our hook today as a somewhat general purpose hook, where anybody who has an ETH/TOKEN pool of any sort, can incentivize their community to purchase their TOKEN for ETH by issuing them points.

## hookdata

the user is going to specify which address they want their points to go to

and the way they tell us this information is through `hookData`

we expect the user to send an address to us through `hookData`

and if `hookData` is empty, or it contains an invalid address, nobody gets points for that swap

## Improvements

- we are NOT creating different $POINT tokens for each pool. in a better implementation, you want to not use a single ERC-20 for $POINTs and instead either deploy a new ERC-20 to represent points for each individual pool, or you want to use something like ERC-1155/ERC-6909 multi-token standards to create different point tokens for each pool

- someone can add liquidity, remove liquidity, add liquidity, remove liquidity, and keep doing this over and over to farm $POINT tokens. in a more sophisticated implementation, instead of giving points while adding liquidity. when somenoe adds liquidity you jsut "take note" of that, and then later when they remove liquidity you give them points based on how long they had that liquidity locked up.
