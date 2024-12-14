# Cart Module Documentation

## Overview

The Cart module manages the shopping cart functionality, including creating anonymous carts, managing cart items, and handling payment details. It includes the following key components:

1. **Controller**: Handles HTTP requests and API endpoints.
2. **Service**: Contains the business logic for managing carts.


---

## Database Model

## Mermaid Diagram

To view the relationships and structure of the `Cart`, `Item`, and related models, you can access the rendered diagram via the following link:

[View Mermaid Diagram](https://mermaid.live/view#pako:eNqdV1tz2jgY_SsaP6UzJBMgNMAbBdp6mgDFJLOX7HSEJUCLLXklOS1N89_3kyw7Jogss7zodr5j6ei7iKcgFoQG_YDKEcNridMHjuA3xFKjp6JvfkpLxtfQYJ0r9BBERUeskN5QFBv0Gb1YXzTQYLgI78cNNPw8Hn4Zj75N7xbvHoIDJr3LKPAsTONhmUznt4MbYJtMJ7_fTu-iPY7p8m8a65AgtRFZOAKeOV1RSXlMkRYoglnlxYvvnMpDgztFpRcfb2i8FbkeECKpUoeWbuGDENu9Lxr9Qk3TP_9CDBoj2Q1T2hy1GDNendmjToZ3KeV6UYg0K0aFZk6gaDEPZ-M9VYTdNVCqjEpnE3JtLF84WDGxEhL2aIAeBrjleKtOYIgs0MPwQ2aJYoTGG8z4CUS_zWc3KCoNPISZSHZrcQrVrEB6ODjFp8gyGQ_mHusNJVTiE-w_W6CHYck4Brc5geJDgfRwSEqWNEl2J5DMHdSn5ivrTxTCAiflvGPxecaGZQYxx5oanx48YpbgZUKrFSTNUt2U5-mSSqRoAgyURDUKk0jc9D6BJyR89jYe_QwoHL2x_xHW2JiWFgTGh3AIXlweMHp18jKa8QkKVGy-M9TJlPcop3OFo_9mA2GOEDpUyMF7UqyZ4OaGCWGmC95R0bAXhOeiaAqKgOXYtrhIkHs53mZhz-2s2UpD4iTGI6FrwAQRqoHnILl-kiLPKDE5VsHm17WhMS-Gb-faGVaKknmeUCDIqkFdwmIWSTPtOSoX1oenmVPIjOuwpRAJxRwxFfJHwWID_pjgNeyIsBgEBA5WE4YpZNAF1vM9txL5q7BbRWfD6e3sZrwYjxCkgXBSDX11OBY513IHZEPXgyOLmGHjON-Z3nilI8aRYkkNamAyyIKlFMpGmpnN2IVXzmEt8oz4LRIMeherpdHzAy865k7rT5GX4r_NPbX_y50n-fyTY66ZNsf8WnZL1YD-6Nti6n8vGDuzfOjJ-y8TcszQ4_xpK_1GirR0i_nuXIvzFNqD1OROpIXGyYgpe38zKmMITLy2TyqzgohbQlm15r39BFzE3L1pjyhSJmD20yZs0xxBVuWljCVbmaoQ8sVyJRhf6UO9Jh8X6Ey48Hp34BqvE0HdTe44GN1jtaBploBfQYw_YhM094MIaTepbKVc17PF_3nFWYI3yo3aqzWFCPbLbxnj1NyfrbKuJ2liw9I50p7hnibo16_zc_FUhE4fGGIBHsB4dbg66EBEY2C5vXD7uLaYV4nCBzYPa4s1gULQcucD1d_QxW7dq7ssHqWRPU65jW1usRu401rM70GqnSqR1D9eBx3dYR00gWJkMc4Tkx3chXNTu7ugEaQUSiIj8FfKOuFDAFeUQswZO4Ll1sCeAYdzLaIdj4O-ljltBCD0ehP0VzhRMCpyoPsrVkIyzP8QIq1AMA76T8GPoH_e7nUvOp2rbuvyutvrXDa77Uawg_nmVfui-b7Z67Wa181O77r33Ah-Wo7WRfuyfdlptjvN963L1lX3-V-UH5vQ)

This link opens the Mermaid diagram in the Mermaid Live Editor, allowing you to view, edit, or export the diagram as needed.


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

This document ensures clarity and fairness in the distribution of profits and proper handling of taxes, commissions, and discounts while maintaining Droplinked’s share.
