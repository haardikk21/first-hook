# Building your first hook

A really really simple "points" program onchain

Assume you have some sort of ETH/XYZ pool on Uni v4, as you sort of trade in this pool

if you are spending ETH to purchase XYZ token, we're gonna give you some "points" for that

an incentive for people to purchase your XYZ token with ETH

> NOTE: this is a barebones proof of concept. you should not use this as it is in a production environment.

### How?

points themselves will also be represented as an ERC-1155 token

(1155 instead of 20 so we can have unique points tokens per pool)

"earning points" => "minting $POINTS tokens to the user"

### How many points to give out?

for every swap that happens, we're gonna give out 20% of the amount of ETH that was spent as $POINTS tokens

e.g. if someone spends 5 ETH, we give them 1 $POINT token

### High level design

Alice does a swap on our hooked pool

Inside `afterSwap`, our hook is gonna check a few things

1. we are in an ETH/XYZ pool of some sort (i.e. ETH is one of the tokens in the pool)
2. Alice is spending ETH to purchase XYZ
3. Figure out exactly how much ETH alice spent and calculate amount of points to give her accordingly
4. Mint $POINTS tokens

we're gonna create a little helper function called `_assignPoints` that does the actual token minting

Detour: how do we know who to mint the POINT tokens to? what address?

### hookData

we're gonna have the user specify an address to mint points to

this enables a user to "donate" their points to someone else if they wish to

MOST hook functions (including \_afterSwap) have a `hookData` argument present as the last argument of the function

users can pass in arbitrary data to your hook contract through this parameter

### BalanceDelta numeric signs

in uniswap by convention, whenever we talk about money changing hands

token transfers are always represented from the perspective of the "caller" (Alice)

if a token transfer amount is a negative number

negative implies money coming out of Alice's wallet and going into Uniswap

if amount is a positive number

Money going into Alice's wallet and coming out of Uniswap

### KYC Pools

e.g. you're only allowed to swap if you can present some sort of KYC proof (ZK proof)

your hook ideally wants to verify that ZK proof. if it is valid, let you do the swap. otherwise, dont let you do the swap.

how do we give the ZK proof to the hook contract? we pass it in via `hookData`

---

Flow of a swap in a hooked pool

Alice
-> Router contract (initiates a swap)
-> Uniswap PoolManager contract
-> has no way of knowing here who Alice is
-> msg.sender = Router contract

            -> Hook contract
                (msg.sender = address(uniswap poolmanager))
                (address sender)

`tx.origin`

that is true, but only in this very simple case

in an alternative case what might be happening is

Alice is using a smart contract wallet(account abstraction)
A paymaster or relayer of some sort is the one actually putting the transaction onchain

`tx.origin` = address(relayer) != address(alice)

GENERALLY speaking, both Uniswap and by extension our hook, has no inherent way of knowing the address of Alice
