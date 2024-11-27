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

| Field                  | Type       | Description                                  | Required |
| ---------------------- | ---------- | -------------------------------------------- | -------- |
| `type`                 | enum       | Type of the RuleSet (`GATING`, `DISCOUNT`).  | Yes      |
| `discountPercentage`   | number     | Discount percentage (if type is `DISCOUNT`). | No       |
| `nftPurchaseLink`      | string     | Link for NFT purchase.                       | No       |
| `network`              | enum       | Blockchain network (e.g., `ETH`).            | Yes      |
| `blockchainType`       | enum       | Blockchain type (e.g., `NFT`).               | Yes      |
| `nftContractAddresses` | string[]   | List of NFT contract addresses.              | Yes      |
| `minimumNftRequired`   | number     | Minimum NFTs required.                       | Yes      |
| `description`          | string     | Description of the RuleSet.                  | No       |
| `ownerID`              | ObjectId   | ID of the owner (user).                      | Yes      |
| `shopId`               | ObjectId   | ID of the shop associated with the RuleSet.  | Yes      |
| `collectionID`         | ObjectId   | ID of the collection linked to the RuleSet.  | Yes      |
| `passedNftDetails`     | string[][] | List of passed NFT details.                  | Yes      |
| `_id`                  | ObjectId   | Unique identifier of the RuleSet.            | Yes      |

## Example RuleSet

````json
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



# Mint to Merch (M2M) Feature Documentation

The Mint to Merch (M2M) feature enables users to seamlessly integrate NFTs into physical merchandise. This feature allows users to select NFTs from their wallet, create a preview of the product, and add it to their cart for purchase.

## Core Functionalities

1. **Fetch Supported Networks**
   - **Purpose**: Provide the frontend with a list of blockchain networks that support NFTs for the M2M feature.
   - **Endpoint**: `GET /product/public/print-services`

2. **Fetch NFTs and Domains**
   - **Purpose**: Retrieves NFTs and domain addresses associated with a given wallet address.
   - **Endpoint**: `POST /nft-images`
   - **Input**:
     - **FetchNFTDTO**: Contains the wallet address and other necessary details.

     Example Input:
     ```json
     {
       "walletAddress": "0x123abc456def7890"
     }
     ```

     Example Output:
     ```json
     {
       "nfts": [
         "https://example.com/nft1.png",
         "https://example.com/nft2.png"
       ],
       "domains": [
         "user.eth",
         "user.wallet"
       ]
     }
     ```

3. **Generate Preview Image**
   - **Purpose**: Creates a preview image by merging an NFT image with a product mockup.
   - **Endpoint**: `POST /preview`
   - **Input**:
     - **NFT details** (image URL, position, etc.)

     Example Request:
     ```json
     {
       "nftImage": "https://example.com/nft1.png",
       "position": "front",
       "productTemplate": "https://example.com/product.png"
     }
     ```

     Example Response:
     ```json
     {
       "previewImage": "https://example.com/preview.png"
     }
     ```

4. **Add M2M Data to Cart**
   - **Purpose**: Allows optional M2M customization data to be included in the cart when adding a product.
   - **Input**:
     - `m2m_data` field in the cart body:
       - `m2m_position`: Position of the print on the product.
       - `print_url`: URL of the NFT print.
       - `preview`: URL of the preview image.

     Example Input:
     ```json
     {
       "product_id": "123",
       "quantity": 1,
       "m2m_data": {
         "m2m_position": "front",
         "print_url": "https://example.com/nft1.png",
         "preview": "https://example.com/preview.png"
       }
     }

## Technical Flow

1. **Fetch Supported Networks**
   - The frontend queries the available networks for M2M.
   - Supported networks like Ethereum, Polygon, and others are listed in the response from the `GET /product/public/print-services` endpoint.

2. **Fetch NFTs and Domains**
   - The user inputs their wallet address and selects a network from the available options.
   - The frontend sends the wallet address to the `POST /nft-images` endpoint to retrieve the NFTs and domain addresses associated with the wallet.
   - The response includes a list of NFTs and associated domain names.

3. **Generate Preview**
   - After selecting an NFT and a product, the backend generates a preview image by merging the selected NFT with a product template.
   - The preview image is sent back to the frontend, allowing the user to view how their NFT will appear on the product.

4. **Add M2M Data to Cart**
   - If the user is satisfied with the preview, they can add the customized product to the cart.
   - The M2M data, including NFT position, print URL, and preview image URL, is added to the cart in the `m2m_data` field.

## API Endpoints Summary

| Endpoint           | HTTP Method | Description |
|--------------------|-------------|-------------|
| `/product/public/print-services` | `GET` | Fetch a list of blockchain networks that support NFTs for M2M. |
| `/nft-images`      | `POST`      | Fetch NFTs and domain addresses for a wallet. |
| `/preview`         | `POST`      | Generate an NFT-based preview image. |

## Example Workflow

### Step 1: Fetch Supported Networks
The frontend makes a request to `/product/public/print-services` to get a list of supported blockchain networks for M2M integration. The list of networks (such as Ethereum, Polygon) is displayed to the user.

### Step 2: Fetch NFTs and Domains
The user enters their wallet address and selects a blockchain network. The frontend sends a request to `/nft-images` with the wallet address, and the response includes a list of NFTs owned by the wallet and associated domain addresses.

### Step 3: Generate Preview Image
Once the user selects an NFT and a product template, a request is sent to `/preview` to generate a preview image. The backend merges the NFT image with the product template and sends back the generated preview image URL.

### Step 4: Add M2M Data to Cart
The user customizes the product by selecting the position of the print and the NFT to be applied. The product, along with the M2M data (print URL, preview image, and position), is added to the cart for checkout.

## Conclusion

The Mint to Merch (M2M) feature allows users to seamlessly integrate their NFTs into physical products. By following the steps outlined above, developers can implement this feature and provide an engaging user experience that combines digital assets with physical goods.

````
