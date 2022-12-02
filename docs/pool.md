---
sidebar_position: 4
---

# Pool

The [Pool](https://github.com/DyadStablecoin/contracts/blob/main/src/Pool.sol)
holds all the deposited ETH.

It has functions to redeem DYAD for ETH, claim liquidated dNFTs and to 
synchronize the protocol. 

### Sync

Burn or mint DYAD depending on the delta between the ETH price from the last 
time sync was called to the current price and update all dNFTs accordingly.

IMPORTANT: One dNFT of the EOA calling this function will get an XP boost.

```javascript
function sync() public returns (uint)
```

Returns the amount burned/minted

NOTES:
- Can be called by anyone
- XP accrual happens only if DYAD is burned

### Claim

Claim a dNFT.

A dNFT is claimable if its deposit attribute is negative.
The claimed dNFT is burned and a new one is minted. The newly minted dNFT xp and
withdrawn attributes will be the same as the burned dNFT. They are copied over.

IMPORTANT: Enough ETH must be sent to cover the negative deposit of the old 
dNFT. If it is more than to cover it, it will get attributed to the new dNFT.

```javascript
function claim(uint id, address receiver) external payable returns (uint)
```

Returns the id of the newly minted dNFT.

NOTES:
- Can be called by anyone
