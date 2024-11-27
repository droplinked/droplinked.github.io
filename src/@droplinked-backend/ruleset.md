# <u>Ruleset</u>
# Feature Overview: RuleSet V2

The **RuleSet V2** feature allows users to:

- **Create** a RuleSet.
- **Retrieve**, **edit**, or **delete** existing RuleSets.
- **Check** if a RuleSet passes validation using a wallet address and RuleSet ID.

This feature has been refactored and implemented on a new route: `/v2`.

The core functionality involves creating a RuleSet and associating it with a collection. A RuleSet includes various parameters such as type, discount percentage, blockchain information, and ownership details.

## Core Functionalities

### 1. CRUD Operations
- **Create a RuleSet**: Users can create a RuleSet by providing the required data. Each RuleSet is linked to a specific collection, owner, and shop.
- **Retrieve RuleSets**: Fetch a list of RuleSets or a specific RuleSet by ID.
- **Edit a RuleSet**: Update details of an existing RuleSet.
- **Delete a RuleSet**: Remove a RuleSet from the system.

### 2. RuleSet Validation
- **Endpoint**: `POST /v2/check-ruleset`
- **Purpose**: Verifies whether a wallet address satisfies a specific RuleSet.
  
#### Input:
- `rulesetId`: ID of the RuleSet.
- `walletAddress`: The wallet address to check.

#### Process:
The system communicates with Web3 to validate the RuleSet against the wallet.

#### Returns:
- `true` if the RuleSet is satisfied.
- `false` if it is not satisfied.

## Data Model

The `RuleSetV2` schema defines the structure of a RuleSet. Below is a description of its properties:

| Field                  | Type       | Description                                          | Required |
|------------------------|------------|------------------------------------------------------|----------|
| `type`                 | enum       | Type of the RuleSet (`GATING`, `DISCOUNT`).           | Yes      |
| `discountPercentage`   | number     | Discount percentage (if type is `DISCOUNT`).          | No       |
| `nftPurchaseLink`      | string     | Link for NFT purchase.                               | No       |
| `network`              | enum       | Blockchain network (e.g., `ETH`).                     | Yes      |
| `blockchainType`       | enum       | Blockchain type (e.g., `NFT`).                        | Yes      |
| `nftContractAddresses` | string[]   | List of NFT contract addresses.                       | Yes      |
| `minimumNftRequired`   | number     | Minimum NFTs required.                               | Yes      |
| `description`          | string     | Description of the RuleSet.                          | No       |
| `ownerID`              | ObjectId   | ID of the owner (user).                              | Yes      |
| `shopId`               | ObjectId   | ID of the shop associated with the RuleSet.           | Yes      |
| `collectionID`         | ObjectId   | ID of the collection linked to the RuleSet.           | Yes      |
| `passedNftDetails`     | string[][] | List of passed NFT details.                          | Yes      |
| `_id`                  | ObjectId   | Unique identifier of the RuleSet.                    | Yes      |

## Example RuleSet

```json
{
  "type": "DISCOUNT",
  "discountPercentage": 20,
  "nftPurchaseLink": "https://example.com",
  "network": "ETH",
  "blockchainType": "NFT",
  "nftContractAddresses": ["0x123...", "0x456..."],
  "minimumNftRequired": 1,
  "description": "A discount RuleSet for NFT holders",
  "ownerID": "6332a65d26038728b5aa9e43",
  "shopId": "6332a65d26038728b5aa9e43",
  "collectionID": "6332a65d26038728b5aa9e43",
  "passedNftDetails": []
}
