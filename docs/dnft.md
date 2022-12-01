---
sidebar_position: 4
---

# dNFT

A [dNFT](https://github.com/DyadStablecoin/contracts/blob/main/src/dNFT.sol)
gives the holder the ability to mint new DYAD. We like to think about it as its
own central bank.

At one point in time there can be no more than 300 dNFTs. There are three ways 
to obtain one. 

- At its official launch
- Claiming a liquidated dNFT
- Trough the secondary market

### Mint dNFT

To mint a new NFT at least $5k worth of ETH have to be included.

```javascript
function mintNft(address receiver) external payable returns (uint)
```

Returns the id of the newly minted dNFT.

NOTES:
- Can only be called until the max supply of 300 is reached
- Every newly minted dNFT will start with an XP of 900k
- The DYAD is automatically deposited in the pool

### Withdraw

Withdraw DYAD from the pool to the EOA, calling this function.

```javascript
function withdraw(uint id, uint amount) external
```

NOTES:
- Can only be called by the dNFT owner
- Will decrease the dNFT deposit attribute by the amount
- Will increase the dNFT withdrawn attribute by the amount

### Deposit

Deposit DYAD from the EOA into the pool.

```javascript
function deposit(uint id, uint amount) external
```

NOTES:
- Can be called by anyone
- Amount must be smaller or equal to the dNFT withdrawn attribute
- Will increase the dNFT deposit attribute by the amount
- Will decrease the dNFT withdrawn attribute by the amount

### Mint DYAD

Mint new DYAD and deposit it in the pool.

```javascript
function mintDyad(uint id) payable public
```

NOTES:
- Can only be called by the dNFT owner
- Will deposit the minted DYAD directly into the pool
- Will increase the dNFT deposit attribute by the amount
