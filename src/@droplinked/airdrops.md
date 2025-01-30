# Airdrop Feature Implementation Document

## Overview

This document outlines the implementation of a multi-chain Airdrop feature including:

1. Smart Contract (Solidity)
2. Web3 Package (NPM)
3. Web3 Service (Web3 project)
4. Backend Service (NestJS)
5. Frontend Integration (shopBuilder)

---

## 1. Smart Contract Specification

There are 3 main functions on the airdrop smart contract, which are these:

```solidity
function distributeERC721(
        address token,
        address[] calldata recipients,
        uint256[] calldata tokenIds,
        string memory airdropId
    )
```

```solidity
function distributeERC1155(
        address token,
        uint256 tokenId,
        address[] calldata recipients,
        uint256 amount,
        string memory airdropId
    )
```

```solidity
function distributeERC20(
        address token,
        address[] calldata recipients,
        uint256[] calldata amounts,
        string memory airdropId
    )
```

Each call to the contract can transfer up to 300 assets.

---

## 2. Web3 package specification

A new Web3Action will be added to web3 package, with which you can execute an airdrop of any asset within the standards of `ERC20`-`ERC721`-`ERC1155`.

Usage of the web3 package for airdrop execution:

```typescript
const provider = web3.web3Instance({
    method: Web3Actions.AIRDROP,
    chain: Chain[chain],
    preferredWallet: ChainWallet.Metamask,
});

const txHashes: string[] = provider.airdrop({
    type: TokenStandard.ERC1155,
    tokenAddress: "0x....",
    tokenId: 1,
    receivers: [
        {
            receiver: "0x....",
            amount: 1
        },
        {
            receiver: "0x....",
            amount: 2
        },
        ...
    ],
    airdropId: '14231264ab698db6a9sd6c',
});
```

The other variants of calling the package:

```typescript
{
    type: TokenStandard.ERC721;
    tokenAddress: EthAddress;
    tokenIds: number[];
    receivers: string[];
    airdropId: string;
},
{
    type: TokenStandard.ERC1155;
    tokenAddress: EthAddress;
    tokenId: number;
    receivers: { receiver: string; amount: number }[];
    airdropId: string;
},
{
    type: TokenStandard.ERC20;
    tokenAddress: EthAddress;
    receivers: { receiver: string; amount: number }[];
    airdropId: string;
}
```

All of this variants, will return a list of transaction hashes of the airdrop as the output which must be sent to backend for verification.

---

## 3. Web3 service

- Fetch nfts route in web3 services: `GET /fetch-nft/fetchNFTs`
- inputs:

     ```js
     {
     	address: string;
     	chain: NETWORK; // ex.: BINANCE, POLYGON, ...
     	network: NETWORKTYPE; // TESTNET || MAINNET
     	cursor: string;
     }
     ```

- outputs:

     ```js
     {
     	collectionName: string;
     	imageUrl: string;
     	description: string;
     	tokenContractAddress: string;
     	tokenId: number;
     }
     [];
     ```

---

## 4. Backend Service

Backend needs 3 routes, one for fetching the list of NFTs of the user given the wallet address, one route for getting the list of transaction hashes from front-end, and verifying them using web3 services and one for the callback from the web3.

First route is the `POST /nfts/airdrop` which gets the detail of the airdrop (tokenAddress, tokenId, receivers and the amounts each receiver will get), and returns an airdropId.

Second route is the `GET /nfts/:chain/:walletAddress` which will return the list of NFTs by calling the web3 services (the output of this route is the same as the web3 services one!).

Third route is the `POST /nfts/airdrop/:airdropId` which gets the list of transaction hash in it's body like this:

```js
{
    transactionId: [ "tx1_hash", "tx2_hash", ... ],
    airdropId: 'a126489b96d61ef96f'
}
```

Then it will call the transaction checking route of web3 as usual and wait for its callback.

Fourth route is the callback route `POST /nfts/callback/airdrop/:airdropId/:transactionId` which will be called by the web3 services. This route will save the airdrop information in a collection by the airdropId.
