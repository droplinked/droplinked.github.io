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

## Mermaid Diagram

To view the relationships and structure of the `Cart`, `Item`, and related models, you can access the rendered diagram via the following link:

[View Mermaid Diagram](https://mermaid.live/view#pako:eNqdV1tz2jgY_SsaP6UzJBMgNMAbBdp6mgDFJLOX7HSEJUCLLXklOS1N89_3kyw7Jogss7zodr5j6ei7iKcgFoQG_YDKEcNridMHjuA3xFKjp6JvfkpLxtfQYJ0r9BBERUeskN5QFBv0Gb1YXzTQYLgI78cNNPw8Hn4Zj75N7xbvHoIDJr3LKPAsTONhmUznt4MbYJtMJ7_fTu-iPY7p8m8a65AgtRFZOAKeOV1RSXlMkRYoglnlxYvvnMpDgztFpRcfb2i8FbkeECKpUoeWbuGDENu9Lxr9Qk3TP_9CDBoj2Q1T2hy1GDNendmjToZ3KeV6UYg0K0aFZk6gaDEPZ-M9VYTdNVCqjEpnE3JtLF84WDGxEhL2aIAeBrjleKtOYIgs0MPwQ2aJYoTGG8z4CUS_zWc3KCoNPISZSHZrcQrVrEB6ODjFp8gyGQ_mHusNJVTiE-w_W6CHYck4Brc5geJDgfRwSEqWNEl2J5DMHdSn5ivrTxTCAiflvGPxecaGZQYxx5oanx48YpbgZUKrFSTNUt2U5-mSSqRoAgyURDUKk0jc9D6BJyR89jYe_QwoHL2x_xHW2JiWFgTGh3AIXlweMHp18jKa8QkKVGy-M9TJlPcop3OFo_9mA2GOEDpUyMF7UqyZ4OaGCWGmC95R0bAXhOeiaAqKgOXYtrhIkHs53mZhz-2s2UpD4iTGI6FrwAQRqoHnILl-kiLPKDE5VsHm17WhMS-Gb-faGVaKknmeUCDIqkFdwmIWSTPtOSoX1oenmVPIjOuwpRAJxRwxFfJHwWID_pjgNeyIsBgEBA5WE4YpZNAF1vM9txL5q7BbRWfD6e3sZrwYjxCkgXBSDX11OBY513IHZEPXgyOLmGHjON-Z3nilI8aRYkkNamAyyIKlFMpGmpnN2IVXzmEt8oz4LRIMeherpdHzAy865k7rT5GX4r_NPbX_y50n-fyTY66ZNsf8WnZL1YD-6Nti6n8vGDuzfOjJ-y8TcszQ4_xpK_1GirR0i_nuXIvzFNqD1OROpIXGyYgpe38zKmMITLy2TyqzgohbQlm15r39BFzE3L1pjyhSJmD20yZs0xxBVuWljCVbmaoQ8sVyJRhf6UO9Jh8X6Ey48Hp34BqvE0HdTe44GN1jtaBploBfQYw_YhM094MIaTepbKVc17PF_3nFWYI3yo3aqzWFCPbLbxnj1NyfrbKuJ2liw9I50p7hnibo16_zc_FUhE4fGGIBHsB4dbg66EBEY2C5vXD7uLaYV4nCBzYPa4s1gULQcucD1d_QxW7dq7ssHqWRPU65jW1usRu401rM70GqnSqR1D9eBx3dYR00gWJkMc4Tkx3chXNTu7ugEaQUSiIj8FfKOuFDAFeUQswZO4Ll1sCeAYdzLaIdj4O-ljltBCD0ehP0VzhRMCpyoPsrVkIyzP8QIq1AMA76T8GPoH_e7nUvOp2rbuvyutvrXDa77Uawg_nmVfui-b7Z67Wa181O77r33Ah-Wo7WRfuyfdlptjvN963L1lX3-V-UH5vQ)

This link opens the Mermaid diagram in the Mermaid Live Editor, allowing you to view, edit, or export the diagram as needed.


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
