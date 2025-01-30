# Airdrop Feature Implementation Document

## Overview

This document outlines the implementation of a multi-chain Airdrop feature, covering the following components:

1. **Smart Contract (Solidity)** - On-chain contract for distributing ERC20, ERC721, and ERC1155 tokens.
2. **Web3 Package (NPM)** - A package to abstract blockchain and wallet interactions for the frontend.
3. **Web3 Service (NestJS)** - Backend service for retrieving NFT details.
4. **Backend Service (NestJS)** - Storing transaction details and verifying airdrop status.
5. **Frontend Integration (ShopBuilder)** - User interface to manage and perform airdrops.

---

## 1. Smart Contract Specification

A dedicated smart contract will be deployed with the following functions:

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

- Each function executes a batch airdrop with a limit of **300 recipients per transaction**.
- If the number of recipients exceeds 300, multiple transactions will be created.

---

## 2. Web3 Package Specification

A new `Web3Action` will be introduced for airdrop execution. The package will facilitate token transfers based on predefined standards (`ERC20`, `ERC721`, `ERC1155`).

### **Usage Example**

```typescript
const provider = web3.web3Instance({
	method: Web3Actions.AIRDROP,
	chain: Chain[chain],
	preferredWallet: ChainWallet.Metamask,
});

const txHashes: string[] = provider.airdrop({
	type: TokenStandard.ERC1155,
	tokenAddress: '0x....',
	tokenId: 1,
	receivers: [
		{ receiver: '0x....', amount: 1 },
		{ receiver: '0x....', amount: 2 },
	],
	airdropId: '14231264ab698db6a9sd6c',
});
```

### **Airdrop Input Variants**

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

- Each call returns a list of **transaction hashes**.
- The frontend will send these transaction hashes to the backend for verification.

---

## 3. Web3 Service (NFT Fetching)

A new API route will be introduced to fetch all NFTs owned by a given wallet on different chains.

### **Endpoint: Fetch NFTs**

**Request:**

- **Method:** `GET`
- **Route:** `/fetch-nft/fetchNFTs`
- **Payload:**

     ```js
     {
     	address: string;
     	chain: NETWORK; // ex.: BINANCE, POLYGON, ...
     	network: NETWORKTYPE; // TESTNET || MAINNET
     	cursor: string;
     }
     ```

**Response:**

```js
[
  {
      collectionName: string;
      imageUrl: string;
      description: string;
      tokenContractAddress: string;
      tokenId: number;
  }
]
```

---

## 4. Backend Service (Airdrop Management)

The backend will handle:

- Fetching NFTs from web3 services.
- Storing airdrop transaction details.
- Verifying transactions.

### **API Routes**

#### **1. Create Airdrop**

- **Method:** `POST`
- **Route:** `/nfts/airdrop`
- **Payload:**

```js
{
    tokenAddress: string;
    tokenId?: number;
    receivers: { receiver: string; amount?: number }[];
}
```

- **Response:**

```js
{
	airdropId: 'a126489b96d61ef96f';
}
```

#### **2. Fetch NFTs for User**

- **Method:** `GET`
- **Route:** `/nfts/:chain/:walletAddress`
- **Response:** (Same as Web3 service response)

#### **3. Submit Transaction Hashes**

- **Method:** `POST`
- **Route:** `/nfts/airdrop/:airdropId`
- **Payload:**

```js
{
    transactionId: ["tx1_hash", "tx2_hash", ...]
}
```

#### **4. Transaction Verification Callback**

- **Method:** `POST`
- **Route:** `/nfts/callback/airdrop/:airdropId/:transactionId`
- **Action:** Saves airdrop status and updates transaction history.

---

## 5. Frontend (ShopBuilder)

Frontend will allow users to:

- **Connect Wallet** and **Fetch NFTs**.
- **Airdrop NFTs** to multiple addresses by:
     - **Manual Input**
     - **Uploading an Excel file (Predefined Template)**
- **Monitor Airdrop Status**

### **Steps in UI:**

1. **User connects their wallet** → Fetches owned NFTs.
2. **User selects an NFT** → Inputs recipient addresses or uploads a file.
3. **User initiates airdrop** → Calls Web3 package.
4. **Frontend receives transaction hashes** → Sends to the backend.
5. **Backend verifies transactions** → Confirms airdrop completion.

---

## Summary

- **Smart Contract** enables airdrop for ERC20, ERC721, and ERC1155 tokens.
- **Web3 Package** abstracts blockchain interactions.
- **Web3 Service** provides NFT ownership details.
- **Backend** stores and verifies transactions.
- **Frontend** allows users to select NFTs and initiate airdrops.

This document serves as a guideline for developers to implement the airdrop feature efficiently.
