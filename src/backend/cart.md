# Cart Module Documentation

## Overview

The Cart module manages the shopping cart functionality, including creating anonymous carts, managing cart items, and handling payment details. It includes the following key components:

1. **Controller**: Handles HTTP requests and API endpoints.
2. **Service**: Contains the business logic for managing carts.

---

# Cart Logic in Droplinked Platform

The **Cart** system in Droplinked handles product types, shipping costs, taxes, commissions, and discounts. This document outlines the logic and calculations for the cart.

---

## Product Types

Droplinked supports the following product types:

### 1. **Physical Products**

- Products uploaded by sellers.
- **Shipping Methods**:
  - **EasyPost Shipping**: Managed by Droplinked.
  - **Custom Shipping**: Managed by the seller; products remain with the producer.

### 2. **Print-on-Demand Products**

- User-designed products manufactured after purchase.
- **Shipping Method**: Only **EasyPost Shipping** is used.
- Includes **production costs** (e.g., $30 production cost, $50 selling price).

### 3. **Digital Products**

- Non-physical products, such as files or software.
- No shipping required.

### 4. **Events**

- Tickets or access to events.
- No shipping required.

---

## Key Cart Logic

### 1. **Shipping Costs**

- **EasyPost & Print-on-Demand Products**:

  - Shipping costs are calculated by Droplinked and belong to Droplinked.

- **Custom Shipping**:
  - Sellers define shipping costs, and these belong to the seller.

---

### 2. **Taxes**

Taxes are calculated based on the buyer's location.

#### Tax Distribution:

- **Physical Products, Digital Products, and Events**: Taxes belong to the seller.
- **Print-on-Demand Products**: Taxes belong to Droplinked.

---

### 3. **Commissions**

Droplinked applies the following commissions:

#### **Platform Commission**

- 1% of the cart's total value is charged as a platform fee.

#### **Stripe Commission**

- If Stripe is used for payment, 3% of the cart value is charged.

#### **Affiliate Sales Commission**

- Sellers can set a commission percentage for affiliates.
- The affiliate's share is calculated based on the seller's profit after deducting production costs (if applicable).

---

### 4. **Discount Logic**

Droplinked supports three types of discounts, but **only one** can be applied at a time:

#### **RuleSet Discount** (e.g., NFT Discounts):

- Applied to the seller’s profit only.
- Droplinked’s share, shipping, and taxes remain unaffected.

#### **Percentage Coupons**:

- Applied to the total cart value and split proportionally among items based on their profit contribution.

#### **Gift Cards** (Fixed Amount Coupons):

- A fixed dollar amount deducted from the cart total.
- Distributed among items proportionally based on their profit contribution.

---

### 5. **Key Calculations**

#### **Total Item Price**

`Unit Price × Quantity`

#### **Profit Calculations**

- **Print-on-Demand Products**:  
  `Total Item Price - (Production Cost × Quantity)`
- **Other Products**:  
  `Total Item Price`

#### **Affiliate Commission**

`Profit × Affiliate Commission Percentage`

#### **Discount Calculations**

- **RuleSet Discount**:  
  `Profit After Affiliate Commission × Discount Percentage`
- **Percentage Coupons & Gift Cards**:  
  Split proportionally among items based on profit.

#### **Seller's Adjusted Profit**

`Profit - Affiliate Commission - Discounts - Platform Commissions`

#### **Droplinked’s Share**

- Platform Commission: 1% of the item price.
- Stripe Commission: 3% (if applicable).
- Shipping Costs: For EasyPost and Print-on-Demand.
- Taxes: For Print-on-Demand.

---

### 6. **Final Total Payable by Customer**

`Sum of Item Prices + Shipping Costs + Taxes - Total Discounts`

#### Constraints:

- The total amount must at least cover Droplinked’s share, shipping costs, and applicable taxes.
- If insufficient, the deficit is deducted from the seller's credit.

---

## Example Calculation

### Scenario:

A customer buys a Print-on-Demand product for $50 with a production cost of $30. The customer uses an NFT for a 10% discount, and the affiliate commission is 50%.

### Steps:

1. **Item Price**: $50
2. **Profit**: $50 - $30 = $20
3. **Affiliate Commission**: $20 × 50% = $10
4. **NFT Discount**: ($20 - $10) × 10% = $1
5. **Adjusted Seller Profit**: $20 - $10 - $1 = $9
6. **Droplinked’s Share**:
   - Platform Fee: $50 × 1% = $0.50
   - Stripe Fee: $50 × 3% = $1.50
7. **Final Seller Profit**: $9
8. **Customer Total Payment**:  
   $50 (item price) - $1 (NFT discount) = $49

---

## Important Notes

1. **Discount Exclusivity**: Only one discount can be applied at a time.
2. **Droplinked's Protections**: Discounts do not affect Droplinked’s share, shipping costs, or applicable taxes.
3. **Credit Adjustment**: If the cart value is insufficient, the seller's credit is used to cover Droplinked’s share.

---

# Cart Module Schema Documentation

## Cart Schema

| Field               | Type                           | Description                                                        |
| ------------------- | ------------------------------ | ------------------------------------------------------------------ |
| `_id`               | `ObjectId`                     | Unique identifier for the cart.                                    |
| `status`            | `string`                       | Status of the cart (e.g., `ACTIVE`, `CHECKED_OUT`).                |
| `type`              | `string`                       | Type of the cart (e.g., `NORMAL`).                                 |
| `shopID`            | `ObjectId` (ref: Shops)        | ID of the shop associated with the cart.                           |
| `ownerID`           | `ObjectId` (ref: User)         | ID of the user who owns the cart.                                  |
| `checkoutAddressID` | `ObjectId` (ref: AddressBooks) | ID of the address book for checkout.                               |
| `items`             | `Array<CartItems>`             | List of items in the cart.                                         |
| `paymentType`       | `string`                       | Payment type (e.g., `STRIPE`).                                     |
| `email`             | `string`                       | Email of the user associated with the cart.                        |
| `giftCard`          | `object`                       | Gift card details (if applied).                                    |
| `groupedItems`      | `Array<CartGroupedItems>`      | Grouped items based on shipping and additional costs.              |
| `passedRules`       | `Array<PassedRule>`            | Rules applied to the cart.                                         |
| `note`              | `string`                       | Optional note for the cart.                                        |
| `isInvoice`         | `boolean`                      | Indicates if the cart is an invoice.                               |
| `invoiceStatus`     | `string`                       | Status of the invoice (`COMPLETED` or `INCOMPLETED`).              |
| `country`           | `string`                       | Country of the user.                                               |
| `costs`             | `Costs`                        | Financial breakdown of the cart (subtotal, shipping, taxes, etc.). |
| `createdAt`         | `Date`                         | Timestamp when the cart was created.                               |
| `updatedAt`         | `Date`                         | Timestamp when the cart was last updated.                          |

---

## CartGroupedItems Schema

| Field       | Type                      | Description                                   |
| ----------- | ------------------------- | --------------------------------------------- |
| `vas`       | `Array<UnionVasTemplate>` | Value-added services (e.g., shipping, taxes). |
| `items`     | `Array<CartItem>`         | List of items in the group.                   |
| `shipments` | `object`                  | Shipping details for the group.               |
| `amounts`   | `object`                  | Financial amounts for the group.              |

---

## Item Schema

| Field                     | Type                    | Description                                             |
| ------------------------- | ----------------------- | ------------------------------------------------------- |
| `_id`                     | `ObjectId`              | Unique identifier for the item.                         |
| `skuID`                   | `ObjectId` (ref: Sku)   | ID of the SKU associated with the item.                 |
| `quantity`                | `number`                | Quantity of the item.                                   |
| `shopOwnerID`             | `ObjectId` (ref: User)  | ID of the shop owner.                                   |
| `shopId`                  | `ObjectId` (ref: Shops) | ID of the shop.                                         |
| `m2m_data`                | `M2MData`               | Metadata for print-on-demand items.                     |
| `totalDiscountPercentage` | `number`                | Total discount percentage applied to the item.          |
| `color`                   | `string`                | Color of the item (if applicable).                      |
| `size`                    | `string`                | Size of the item (if applicable).                       |
| `passedRule`              | `PassedRule`            | Rule applied to the item.                               |
| `nftId`                   | `ObjectId` (ref: Nft)   | ID of the NFT associated with the item (if applicable). |

---

## Costs Schema

| Field                  | Type       | Description                   |
| ---------------------- | ---------- | ----------------------------- |
| `subtotal`             | `number`   | Subtotal of the cart.         |
| `totalWithoutDiscount` | `number`   | Total cost without discounts. |
| `shipping`             | `Shipping` | Shipping cost breakdown.      |
| `taxes`                | `Taxes`    | Tax breakdown.                |
| `discountApplied`      | `number`   | Total discount applied.       |
| `totalCartAmount`      | `number`   | Total amount of the cart.     |
| `droplinkedCommission` | `number`   | Commission for Droplinked.    |
| `stripeCommission`     | `number`   | Commission for Stripe.        |
| `droplinkedTotalShare` | `number`   | Total share for Droplinked.   |
| `producerNetProfit`    | `number`   | Net profit for the producer.  |

---

## Shipping Schema

| Field        | Type     | Description                     |
| ------------ | -------- | ------------------------------- |
| `droplinked` | `number` | Shipping cost for Droplinked.   |
| `producer`   | `number` | Shipping cost for the producer. |
| `total`      | `number` | Total shipping cost.            |

---

## Taxes Schema

| Field        | Type     | Description             |
| ------------ | -------- | ----------------------- |
| `droplinked` | `number` | Taxes for Droplinked.   |
| `producer`   | `number` | Taxes for the producer. |
| `total`      | `number` | Total taxes.            |

---

## M2MData Schema

| Field          | Type     | Description                         |
| -------------- | -------- | ----------------------------------- |
| `m2m_position` | `string` | Position for print-on-demand items. |
| `print_url`    | `string` | URL of the print file.              |
| `preview`      | `string` | Preview image URL.                  |

---

## Relationships

- **CART** has many **ITEM**.
- **CART** has many **GROUPED_ITEM**.
- **GROUPED_ITEM** has many **ITEM**.
- **GROUPED_ITEM** has one **SHIPMENT**.
- **GROUPED_ITEM** has many **VAS**.
- **ITEM** references **SKU**, **USER**, **SHOPS**, and optionally **NFT**.
- **CART** references **USER**, **SHOPS**, and **ADDRESS_BOOKS**.
- **CART** has one **COSTS**.
- **COSTS** has one **SHIPPING** and one **TAXES**.

---
For more details, visit the [https://mermaid.live/view#pako:eNqtVktz2jAQ_isen5NLjtwcIAmT4THYaTsdZhjFWrCKLanSipSS_PfKNgHbEoFDfWDE7qdvtU9pH6aCQtgLQQ0YWStSLHhgv340T4L399tbsQ9GyXAc9IJUcCSMaxfwOJ--zIaD5QGYkQOmJW-TMZ7mhsIXwPhpNBsPJ8klwm9R3IQ0VeO78XIQJZFjrwmaPJQmhEQmOMldffz8YvUKVqCApz6Gl3g4vwCJn6az2INpRvEMTRNyDU00GMyHcby8n06fL2H70zhpRa8WNDIwG00ezwKS6MfwuL3Bvq_X5adRMb4OlowGs2dHrJGg0Y4YdxIcoSS7AjgmPp3OhBwNggfXgnjjoPyqNIN0IwxGlCrQ2g-CgrD8JP2lBQ_WbIV9omhXrISRQEcIhe6oJNEa6Nzk4DrLBTYcehUiB8IDpkd8K1jq-spqeewPXSoMR7Xr2E-FxgaWEgRkBQSpArukEXp0RtKm7uMzw1VZX5_hjekElnEMfhvCkeHOm8jp-ZRVeaYtTeVfcVcs7WnJSbzKBcEABZJ8wHQVlRmo1BYQWYMnarlQrjX2F84m0s3jChtHO4arNbL2Hbot6VYK81SPzpgsS78rJ0Xplu4YPM7N_YW-qjhOUTsSlAPV3ctJ4Yub7jRkHfctyQ1c19YKUmBbUJ1DHEe3e5Iy2VJoVg5sd0rYX1walXs0sGXw1rFTTv-vi_kUWHsRXAmtRvm1tNVMvxLcHu5Xbqpn9r6bJG1eq_7wNs13hpmdjZ-946lIae11xEj-NAdcTUcPFJGUOQPqtWanKUZF29BhtxJ2G98A7YuiYFq3kn5wxPou4bz-xJGUxuKMKKdipRLUpKAmgDMlVgw9XVXdhfvz7Oc4vS53-Our9D-ThzdhAcreYNQ-8CruRYgZ2EYOe3ZJidoswgUvccSgiHc8DXuoDNyE9i5bZ2FvRXJt_9V3weGB-AmRhP8UojiCgDIUalw_J6tX5cc_NNIkaQ](#).
