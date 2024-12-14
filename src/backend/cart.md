# Cart Module Documentation

## Overview

The Cart module manages the shopping cart functionality, including creating anonymous carts, managing cart items, and handling payment details. It includes the following key components:

1. **Controller**: Handles HTTP requests and API endpoints.
2. **Service**: Contains the business logic for managing carts.

---

## Cart Controller

### `createAnonymousCart` Endpoint

- **Description**: Allows public users to create an anonymous cart.
- **Route**: `POST /public/anonymous-cart`
- **Functionality**:
  - Calls the `createAnonymousCart` method from the `CartService`.
  - Returns the created cart object in the response.
- **Swagger Metadata**:
  - `@ApiOperation`: Describes the API operation.
  - `@ApiResponse`: Defines the expected HTTP response structure.

---

## Cart Service

### `createAnonymousCart`

- **Description**: 
  - Interacts with the Cart repository to create an anonymous cart.
  - Sets the cart type to `ANONYMOUS`.
- **Returns**: The newly created anonymous cart object.

---

## Database Model

### Mermaid Diagram

The following diagram visualizes the relationships and structure of the `Cart`, `Item`, and related models:

```mermaid
erDiagram
    Cart {
        string status "Status of the cart (e.g., ACTIVE, CHECKED_OUT)"
        string type "Type of the cart (e.g., NORMAL, ANONYMOUS)"
        ObjectId shopID "Reference to Shops"
        ObjectId ownerID "Reference to User"
        ObjectId checkoutAddressID "Reference to AddressBooks"
        CartItem[] items "List of items in the cart"
        string paymentType "Payment type (e.g., STRIPE)"
        object casperPaymentIntent "Payment intent for Casper"
        object stacksPaymentIntent "Payment intent for Stacks"
        object xrplsidechainPaymentIntent "Payment intent for XRPL Sidechain"
        object polygonPaymentIntent "Payment intent for Polygon"
        object nearPaymentIntent "Payment intent for NEAR"
        object hederaPaymentIntent "Payment intent for Hedera"
        object binancePaymentIntent "Payment intent for Binance"
        object redbellyPaymentIntent "Payment intent for Redbelly"
        object paymentIntent "General payment intent"
        object shipmentRates "Available shipment rates"
        number selectedShipmentRate "Selected shipment rate"
        string selectedShipmentRateID "Selected shipment rate ID"
        object shipmentData "Shipment data"
        object[] availableShipmentRates "List of available shipment rates"
        object[] selectedShipmentRates "List of selected shipment rates"
        object[] selectedShipmentRateIDs "List of selected shipment rate IDs"
        object[] shipmentInformation "Additional shipment information"
        string email "Email address of the cart owner"
        object giftCard "Gift card details"
        CartGroupedItems[] groupedItems "Grouped items in the cart"
        PassedRule[] passedRules "List of passed rules"
        string note "Optional note"
        boolean isInvoice "Flag indicating if the cart is an invoice"
        string invoiceStatus "Status of the invoice (COMPLETED or INCOMPLETED)"
        string country "Country associated with the cart"
        date createdAt "Timestamp of creation"
        date updatedAt "Timestamp of last update"
    }

    Item {
        ObjectId skuID "Reference to SKU"
        number quantity "Quantity of the item"
        ObjectId shopOwnerID "Reference to the shop owner"
        ObjectId shopId "Reference to the shop"
        object m2m_data "Many-to-many data"
        number totalDiscountPercentage "Total discount percentage"
        string color "Color of the item"
        string size "Size of the item"
        object passedRule "Passed rule details"
        ObjectId nftId "Reference to NFT (optional)"
    }

    CartGroupedItems {
        UnionVasTemplate[] vas "VAS templates for grouped items"
        CartItem[] items "List of items in the group"
        object shipments "Shipment details for the group"
        object amounts "Amounts related to the group"
    }

    Cart ||--o{ Item : "contains"
    Cart ||--o{ CartGroupedItems : "groups"
    Cart ||--o{ Shops : "associated with"
    Cart ||--o{ User : "owned by"
    Cart ||--o{ AddressBooks : "checkout address"
    Item ||--o{ Sku : "has SKU"
    Item ||--o{ Shops : "sold by"
    Item ||--o{ User : "owned by"
    Item ||--o{ Nft : "optionally references"
```

## addProductToCartV2 Service
## Step 1: Initial Validation

### Description
The first step ensures the integrity of input data before proceeding with adding a product to the cart. This is done by calling the `addToCartInitialValidation` function, which checks:

- `skuID` and `shopID` are valid ObjectIds.
- Anonymous carts cannot include a wallet address.
- Authenticated users can only add products from shops they own.

### Code
```typescript
await this.addToCartInitialValidation(
  query.skuID,
  anonymous,
  query?.wallet,
  query?.ownerID,
  query?.shopID,
);
