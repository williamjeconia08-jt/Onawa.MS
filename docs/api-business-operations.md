# Onawa Management System — API and Business Operations Plan

## 1. Document Purpose

This document defines the main business operations the Onawa Management System MVP should support. It is not implementation code. It describes the operations, permissions, inputs, validation rules, outputs, and side effects that should later guide backend API design and service-layer implementation.

The operations are grouped by business module and should be implemented phase by phase.

## 2. API Design Principles

### API-001: Business Operations First

APIs should represent real business actions, not only database table operations. For example, `completeSale` is more important than a generic `insertSale` action because it must validate stock, calculate VAT, create sale records, and create inventory movements.

### API-002: Separate Drafts From Completed Transactions

Where a workflow has drafts, such as receiving or transfers, draft records may be edited. Once completed, records should be corrected through traceable correction workflows rather than silently changed.

### API-003: Use Role-Based Authorization

Every operation must check the logged-in user's permissions before executing.

### API-004: Preserve Historical Snapshots

Transaction operations must snapshot important values such as product name, product type, selling price, VAT rate, currency symbol, and user identity.

### API-005: Create Inventory Movements Inside Business Operations

Stock-changing operations must create inventory movements as part of the same business operation.

### API-006: Support Event Time and Entry Time

Operations that can be back-entered must accept event time and must also store system entry time.

## 3. Common Request Context

Most protected operations should receive or derive:

- Authenticated user ID
- User role
- Request timestamp
- Business event time, where applicable
- Back-entered flag, where applicable

Common validation:

- User must be authenticated.
- User must be active.
- User role must be authorized for the operation.
- Referenced records must exist.
- Deactivated records must not be used for new business transactions unless explicitly allowed for historical viewing.

## 4. Authentication and User Operations

## 4.1 `login`

Purpose:

- Authenticate a user and start a session.

Allowed roles:

- Owner
- Manager
- Cashier
- Inventory Clerk

Inputs:

- Username
- Password

Validation:

- Username is required.
- Password is required.
- User must exist.
- User must be active.
- Password must match the stored password hash.

Outputs:

- Authenticated user profile
- Role
- Session/token information, depending on implementation choice

Side effects:

- May record login activity in audit log.

## 4.2 `logout`

Purpose:

- End the user's session.

Allowed roles:

- Any authenticated active user

Inputs:

- Current session/token

Outputs:

- Logout success response

Side effects:

- May record logout activity in audit log.

## 4.3 `createUser`

Purpose:

- Create a new user account.

Allowed roles:

- Owner
- Manager, if final policy allows manager user administration

Inputs:

- Full name
- Username
- Password
- Role
- Active status

Validation:

- Full name is required.
- Username is required and unique.
- Password is required.
- Role must be valid.

Outputs:

- Created user summary

Side effects:

- Password is stored only as a secure hash.
- Audit log records who created the user.

## 4.4 `updateUser`

Purpose:

- Update user profile or role information.

Allowed roles:

- Owner
- Manager, if final policy allows manager user administration

Inputs:

- User ID
- Full name
- Username
- Role
- Active status

Validation:

- User must exist.
- Username must remain unique.
- Role must be valid.

Outputs:

- Updated user summary

Side effects:

- Audit log records changes.

## 4.5 `deactivateUser`

Purpose:

- Prevent a user from logging in without deleting historical records.

Allowed roles:

- Owner
- Manager, if final policy allows manager user administration

Inputs:

- User ID
- Reason/note

Validation:

- User must exist.
- User must not already be inactive.

Outputs:

- Deactivated user summary

Side effects:

- User can no longer log in.
- Audit log records deactivation.

## 5. Settings Operations

## 5.1 `getSystemSettings`

Purpose:

- Read current business settings.

Allowed roles:

- Owner
- Manager
- Cashier, read-only settings needed for receipt display
- Inventory Clerk, read-only settings needed for operational screens

Outputs:

- Business name
- Business registration details
- Business address
- Currency code and symbol
- VAT rate
- Daily reporting cutoff time

## 5.2 `updateSystemSettings`

Purpose:

- Update configurable business settings.

Allowed roles:

- Owner
- Manager, only if final policy allows

Inputs:

- Business name
- Business registration details
- Business address
- Currency code
- Currency symbol
- VAT rate
- Daily reporting cutoff time

Validation:

- VAT rate must be valid.
- Daily reporting cutoff must be valid time.
- Currency defaults should remain `NAD` and `N$` unless intentionally changed.

Outputs:

- Updated settings

Side effects:

- Audit log records settings changes.
- Historical transactions keep their original VAT and currency snapshots.

## 6. Product Catalogue Operations

## 6.1 `createCategory`

Purpose:

- Create a product category.

Allowed roles:

- Owner
- Manager

Inputs:

- Name
- Description

Validation:

- Name is required.
- Name should be unique among active categories.

Outputs:

- Created category

## 6.2 `updateCategory`

Purpose:

- Update category details.

Allowed roles:

- Owner
- Manager

Inputs:

- Category ID
- Name
- Description
- Active status

Validation:

- Category must exist.
- Name is required.

Outputs:

- Updated category

## 6.3 `createProduct`

Purpose:

- Add a new product or service to the catalogue.

Allowed roles:

- Owner
- Manager

Inputs:

- Product name
- Category ID
- Product type
- Selling price
- Cost price
- VAT rate
- Stock threshold
- Active status

Validation:

- Product name is required.
- Category must exist and be active.
- Product type must be valid.
- Selling price is required and cannot be negative.
- Cost price is optional and defaults to N$0.00.
- VAT rate defaults to 15% unless changed by authorized configuration.
- Stock threshold applies to stock-tracked products.

Outputs:

- Created product

Side effects:

- If product is stock-tracked, a stock balance record may be initialized at zero.
- Audit log may record product creation.

## 6.4 `updateProduct`

Purpose:

- Update product catalogue details.

Allowed roles:

- Owner
- Manager

Inputs:

- Product ID
- Product name
- Category ID
- Product type, if allowed before transactions exist
- Selling price
- Cost price
- VAT rate
- Stock threshold
- Active status

Validation:

- Product must exist.
- Product type changes should be restricted after transactions exist.
- Selling price cannot be negative.
- Cost price can default to N$0.00.

Outputs:

- Updated product

Side effects:

- Audit log records price, VAT, status, and important product changes.
- Old sales remain unchanged because they use snapshots.

## 6.5 `deactivateProduct`

Purpose:

- Stop a product from being used in new transactions while preserving history.

Allowed roles:

- Owner
- Manager

Inputs:

- Product ID
- Reason/note

Validation:

- Product must exist.
- Product must not already be inactive.

Outputs:

- Deactivated product

Side effects:

- Product is hidden from normal new transaction selection.
- Historical reports still include the product.

## 6.6 `addProductBarcode`

Purpose:

- Add a barcode to a product.

Allowed roles:

- Owner
- Manager

Inputs:

- Product ID
- Barcode value

Validation:

- Product must exist and be active.
- Barcode value is required.
- Barcode value must be unique among active barcodes.

Outputs:

- Created barcode record

## 6.7 `deactivateProductBarcode`

Purpose:

- Stop using a barcode for future lookup without changing historical transactions.

Allowed roles:

- Owner
- Manager

Inputs:

- Barcode ID
- Reason/note

Validation:

- Barcode must exist.
- Barcode must be active.

Outputs:

- Deactivated barcode record

## 6.8 `searchProducts`

Purpose:

- Find products by name, barcode, category, type, or active status.

Allowed roles:

- Owner
- Manager
- Cashier
- Inventory Clerk

Inputs:

- Search text
- Barcode
- Category filter
- Product type filter
- Active-only flag

Outputs:

- Matching product summaries

Business rules:

- POS lookup should normally return only active products.
- Inventory and historical screens may allow inactive products to be searched.

## 7. Supplier Operations

## 7.1 `createSupplier`

Purpose:

- Create an external supplier.

Allowed roles:

- Owner
- Manager

Inputs:

- Supplier name
- Contact person
- Phone number
- Email
- Address
- Notes

Validation:

- Supplier name is required.

Outputs:

- Created supplier

## 7.2 `updateSupplier`

Purpose:

- Update supplier details.

Allowed roles:

- Owner
- Manager

Inputs:

- Supplier ID
- Supplier details
- Active status

Validation:

- Supplier must exist.

Outputs:

- Updated supplier

## 7.3 `deactivateSupplier`

Purpose:

- Prevent a supplier from being used for new receiving while preserving history.

Allowed roles:

- Owner
- Manager

Inputs:

- Supplier ID
- Reason/note

Validation:

- Supplier must exist.
- Supplier must be active.

Outputs:

- Deactivated supplier

## 8. Supplier Receiving Operations

## 8.1 `createSupplierReceivingDraft`

Purpose:

- Start a supplier receiving document.

Allowed roles:

- Owner
- Manager
- Inventory Clerk

Inputs:

- Supplier ID
- Document reference
- Event time
- Back-entered flag
- Notes

Validation:

- Supplier must exist and be active.
- Event time is required.

Outputs:

- Supplier receiving draft

## 8.2 `addSupplierReceivingLine`

Purpose:

- Add a product line to a supplier receiving draft.

Allowed roles:

- Owner
- Manager
- Inventory Clerk

Inputs:

- Receiving draft ID
- Product ID
- Quantity
- Unit cost

Validation:

- Draft must exist and still be editable.
- Product must be active.
- Product type must be `NORMAL_STOCK`.
- Quantity must be a positive whole number.
- Unit cost is optional and defaults to N$0.00.

Outputs:

- Receiving line

## 8.3 `completeSupplierReceiving`

Purpose:

- Finalize supplier receiving and increase stock.

Allowed roles:

- Owner
- Manager
- Inventory Clerk, if final policy allows completion without separate approval

Inputs:

- Receiving draft ID

Validation:

- Draft must exist.
- Draft must have at least one line.
- All products must be valid normal stock products.
- Quantities must be positive whole numbers.

Outputs:

- Completed receiving record
- Created inventory movements
- Updated stock balances

Side effects:

- Creates `SUPPLIER_RECEIVING` inventory movements.
- Increases stock balances.
- Locks the receiving record from ordinary editing.

## 9. Internal Bakery Receiving Operations

## 9.1 `createBakeryReceivingDraft`

Purpose:

- Start an internal bakery receiving document.

Allowed roles:

- Owner
- Manager
- Inventory Clerk

Inputs:

- Event time
- Back-entered flag
- Notes

Validation:

- Event time is required.

Outputs:

- Bakery receiving draft

## 9.2 `addBakeryReceivingLine`

Purpose:

- Add bread or another internal production stock product to a bakery receiving draft.

Allowed roles:

- Owner
- Manager
- Inventory Clerk

Inputs:

- Bakery receiving draft ID
- Product ID
- Quantity
- Assumed unit cost

Validation:

- Draft must exist and still be editable.
- Product must be active.
- Product type must be `INTERNAL_PRODUCTION_STOCK`.
- Quantity must be a positive whole number.

Outputs:

- Bakery receiving line

## 9.3 `completeBakeryReceiving`

Purpose:

- Finalize internal bakery receiving and increase shop stock.

Allowed roles:

- Owner
- Manager
- Inventory Clerk, if final policy allows completion without separate approval

Inputs:

- Bakery receiving draft ID

Validation:

- Draft must exist.
- Draft must have at least one line.
- All products must be internal production stock products.

Outputs:

- Completed bakery receiving record
- Created inventory movements
- Updated stock balances

Side effects:

- Creates `INTERNAL_BAKERY_RECEIVING` inventory movements.
- Increases stock balances.
- Locks the receiving record from ordinary editing.

## 10. Inventory Operations

## 10.1 `getCurrentStock`

Purpose:

- View current stock balances.

Allowed roles:

- Owner
- Manager
- Inventory Clerk
- Cashier, if POS stock visibility is allowed

Inputs:

- Product filter
- Category filter
- Low-stock-only flag

Outputs:

- Product stock summaries

Business rules:

- Non-stock service products should not show physical stock balances.

## 10.2 `getInventoryMovements`

Purpose:

- View movement history explaining stock changes.

Allowed roles:

- Owner
- Manager
- Inventory Clerk

Inputs:

- Product ID
- Movement type
- Date range
- User ID
- Back-entered flag
- Reference type/reference ID

Outputs:

- Inventory movement list

## 10.3 `recordManualStockAdjustment`

Purpose:

- Record an approved manual stock correction outside ordinary receiving, sale, transfer, refund, or damage workflows.

Allowed roles:

- Owner
- Manager
- Inventory Clerk, only with manager approval if final policy allows

Inputs:

- Product ID
- Quantity change
- Reason
- Event time
- Back-entered flag
- Manager approval

Validation:

- Product must be active and stock-tracked.
- Quantity change cannot be zero.
- Reason is required.
- Stock should not normally become negative.
- Manager approval is required for sensitive adjustments.

Outputs:

- Inventory movement
- Updated stock balance

Side effects:

- Creates `MANUAL_ADJUSTMENT` inventory movement.

## 11. Damage and Expiry Operations

## 11.1 `recordDamageOrExpiry`

Purpose:

- Record damaged or expired goods and reduce stock after approval.

Allowed roles:

- Owner
- Manager
- Inventory Clerk, with manager approval

Inputs:

- Product ID
- Quantity
- Reason type: damaged or expired
- Event time
- Back-entered flag
- Note
- Manager approval

Validation:

- Product must be active and stock-tracked.
- Quantity must be a positive whole number.
- Reason type must be valid.
- Manager approval is always required.
- Stock should not normally become negative.

Outputs:

- Damage/expiry record
- Inventory movement
- Updated stock balance

Side effects:

- Creates `DAMAGE_WRITE_OFF` or `EXPIRY_WRITE_OFF` inventory movement.

## 12. Stock Count Operations

## 12.1 `createStockCount`

Purpose:

- Start a stock count or spot check.

Allowed roles:

- Owner
- Manager
- Inventory Clerk

Inputs:

- Count type
- Category, if applicable
- Event time
- Notes

Validation:

- Count type must be valid.

Outputs:

- Stock count record

## 12.2 `addStockCountLine`

Purpose:

- Record counted quantity for a product.

Allowed roles:

- Owner
- Manager
- Inventory Clerk

Inputs:

- Stock count ID
- Product ID
- Counted quantity
- Reason/note

Validation:

- Stock count must exist and be editable.
- Product must be stock-tracked.
- Counted quantity must be a whole number and cannot be negative.

Outputs:

- Stock count line with calculated variance

## 12.3 `approveStockCountCorrection`

Purpose:

- Approve and apply stock count variance corrections.

Allowed roles:

- Owner
- Manager

Inputs:

- Stock count ID
- Approval note

Validation:

- Stock count must have variance lines.
- Approval policy must be satisfied.

Outputs:

- Approved stock count
- Inventory movements
- Updated stock balances

Side effects:

- Creates `STOCK_COUNT_CORRECTION` inventory movements.

## 13. Branch Transfer Operations

## 13.1 `createBranchDestination`

Purpose:

- Add a branch destination.

Allowed roles:

- Owner
- Manager

Inputs:

- Branch name

Validation:

- Name is required.
- Name should be unique among active branch destinations.

Outputs:

- Created branch destination

Initial launch destinations:

- Oshandi
- Ondangwa
- Omundaungilo

## 13.2 `deactivateBranchDestination`

Purpose:

- Stop a branch destination from being used in new transfers while preserving history.

Allowed roles:

- Owner
- Manager

Inputs:

- Branch destination ID
- Reason/note

Validation:

- Branch destination must exist and be active.

Outputs:

- Deactivated branch destination

## 13.3 `createBranchTransferDraft`

Purpose:

- Start a branch transfer document.

Allowed roles:

- Owner
- Manager
- Inventory Clerk

Inputs:

- Destination branch ID
- Paper reference/slip number
- Event time
- Back-entered flag
- Notes

Validation:

- Destination branch must exist and be active.
- Event time is required.

Outputs:

- Branch transfer draft

## 13.4 `addBranchTransferLine`

Purpose:

- Add a product and quantity to a transfer draft.

Allowed roles:

- Owner
- Manager
- Inventory Clerk

Inputs:

- Branch transfer draft ID
- Product ID
- Quantity

Validation:

- Draft must exist and still be editable.
- Product must be active and stock-tracked.
- Quantity must be a positive whole number.
- Available stock should normally be enough.

Outputs:

- Branch transfer line

## 13.5 `completeBranchTransfer`

Purpose:

- Finalize a branch transfer and reduce main branch stock.

Allowed roles:

- Owner
- Manager
- Inventory Clerk, if final policy allows completion

Inputs:

- Branch transfer draft ID

Validation:

- Draft must exist.
- Draft must have at least one line.
- All products must be stock-tracked.
- Available stock should normally be enough.

Outputs:

- Completed branch transfer
- Created inventory movements
- Updated stock balances

Side effects:

- Creates `BRANCH_TRANSFER_OUT` inventory movements.
- Locks transfer from ordinary editing.

## 13.6 `confirmBranchTransferReceived`

Purpose:

- Record that the destination branch confirmed receiving the stock.

Allowed roles:

- Owner
- Manager
- Inventory Clerk, if final policy allows

Inputs:

- Branch transfer ID
- Receiving confirmed by name
- Confirmation date/time
- Note

Validation:

- Transfer must exist and be completed.
- Confirmation name is required.

Outputs:

- Updated transfer confirmation status

## 13.7 `correctCompletedBranchTransfer`

Purpose:

- Correct an already-saved transfer with traceability.

Allowed roles:

- Owner
- Manager
- Inventory Clerk only with manager approval

Inputs:

- Branch transfer ID
- Corrected lines
- Correction reason
- Manager approval

Validation:

- Transfer must exist and be completed.
- Correction reason is required.
- Manager approval is required.
- Resulting stock changes must be valid.

Outputs:

- Transfer correction record
- Inventory correction movements, if quantities changed
- Updated transfer status

Side effects:

- Records who corrected, who approved, when, and why.

## 14. Sales and POS Operations

## 14.1 `startSale`

Purpose:

- Start a new POS sale session/cart.

Allowed roles:

- Owner
- Manager
- Cashier

Inputs:

- Event time, if back-entered
- Back-entered flag

Outputs:

- Sale draft/cart identifier

## 14.2 `addSaleLine`

Purpose:

- Add a product or service to the sale cart.

Allowed roles:

- Owner
- Manager
- Cashier

Inputs:

- Sale draft/cart ID
- Product ID or barcode
- Quantity

Validation:

- Product must exist and be active.
- Quantity must be a positive whole number.
- Stock-tracked product must have enough stock.

Outputs:

- Updated sale cart

## 14.3 `completePaidSale`

Purpose:

- Complete a normal paid sale.

Allowed roles:

- Owner
- Manager
- Cashier

Inputs:

- Sale draft/cart ID
- Payment method: cash, card, or mobile
- Event time
- Back-entered flag

Validation:

- Cart must have at least one line.
- Payment method must be valid.
- Split payments are not allowed in MVP.
- Stock-tracked items must still have enough stock.

Outputs:

- Completed sale
- Receipt data
- Created inventory movements
- Updated stock balances

Side effects:

- Snapshots product names, product types, prices, VAT rates, VAT amounts, and currency symbol.
- Creates `SALE_DEDUCTION` movements for stock-tracked items.
- Does not create stock movements for airtime/non-stock items.

## 14.4 `createUnpaidSale`

Purpose:

- Complete a sale as unpaid after manager approval.

Allowed roles:

- Owner
- Manager
- Cashier only with manager approval

Inputs:

- Sale draft/cart ID
- Customer/person name
- Phone number
- Reason/note
- Manager approval
- Event time
- Back-entered flag

Validation:

- Cart must have at least one line.
- Customer/person name is required.
- Phone number is required.
- Manager approval is required.
- Stock-tracked items must have enough stock.

Outputs:

- Completed unpaid sale
- Receipt data
- Created inventory movements
- Updated stock balances

Side effects:

- Reduces stock immediately for stock-tracked items.
- Marks sale status as `UNPAID`.
- Creates approval record.

## 14.5 `markUnpaidSalePaid`

Purpose:

- Mark an outstanding unpaid sale as fully paid.

Allowed roles:

- Owner
- Manager

Inputs:

- Sale ID
- Payment method: cash, card, or mobile
- Paid date/time
- Notes

Validation:

- Sale must exist.
- Sale status must be `UNPAID`.
- Payment method must be valid and cannot be unpaid.
- Partial payment is not supported.

Outputs:

- Unpaid sale settlement record
- Updated sale status/payment status

Side effects:

- Does not change original sale lines.
- Records manager who marked it paid.

## 14.6 `voidSale`

Purpose:

- Void a sale only if business policy allows and only under strict controls.

Allowed roles:

- Owner
- Manager

Inputs:

- Sale ID
- Void reason

Validation:

- Sale must exist.
- Void reason is required.
- Sale must meet final void policy.

Outputs:

- Voided sale status

Side effects:

- If stock had moved, reversing inventory movements may be required.
- This operation should be carefully limited and may be deferred if refunds are enough for MVP.

## 15. Refund Operations

## 15.1 `createRefundDraft`

Purpose:

- Start a refund linked to an original sale where possible.

Allowed roles:

- Owner
- Manager

Inputs:

- Original sale ID
- Refund reason
- Event time
- Back-entered flag

Validation:

- Refund reason is required.
- Original sale should exist where possible.

Outputs:

- Refund draft

## 15.2 `addRefundLine`

Purpose:

- Add a refunded item to the refund draft.

Allowed roles:

- Owner
- Manager

Inputs:

- Refund draft ID
- Original sale line ID
- Quantity
- Return-to-stock flag
- Unsellable reason, if not returned to stock

Validation:

- Refund draft must exist and be editable.
- Quantity must be a positive whole number.
- Quantity should not exceed original sale line quantity minus previous refunds.
- Unsellable reason is required if returned stock is not sellable.

Outputs:

- Refund line

## 15.3 `approveAndCompleteRefund`

Purpose:

- Approve and finalize refund.

Allowed roles:

- Owner
- Manager

Inputs:

- Refund draft ID
- Manager approval

Validation:

- Refund must have at least one line.
- Manager approval is required.

Outputs:

- Completed refund
- Inventory movements, where applicable

Side effects:

- Does not overwrite the original sale.
- Creates `REFUND_RETURN_TO_STOCK` movement for sellable stock-tracked returns.
- Unsellable items do not return to available stock.

## 16. Reporting Operations

## 16.1 `getDailySalesReport`

Purpose:

- Show daily sales totals using the configured reporting cutoff.

Allowed roles:

- Owner
- Manager

Inputs:

- Report date
- Optional custom cutoff time

Outputs:

- Paid sales total
- Unpaid sales total
- Refund total
- VAT total
- Sales by payment method
- Physical product sales
- Airtime/non-stock sales

## 16.2 `getOutstandingUnpaidSalesReport`

Purpose:

- Show unpaid sales that have not been settled.

Allowed roles:

- Owner
- Manager

Inputs:

- Date range
- Customer/person search

Outputs:

- Outstanding unpaid sale list
- Totals

## 16.3 `getInventoryReport`

Purpose:

- Show current stock, low stock, and movement information.

Allowed roles:

- Owner
- Manager
- Inventory Clerk

Inputs:

- Product/category filter
- Movement type filter
- Date range

Outputs:

- Current stock levels
- Low-stock items
- Movement history summaries

## 16.4 `getBranchTransferReport`

Purpose:

- Report transfers by branch, product, date, and confirmation status.

Allowed roles:

- Owner
- Manager
- Inventory Clerk

Inputs:

- Branch destination
- Product
- Date range
- Confirmation status

Outputs:

- Transfer summaries
- Transfer line details

## 16.5 `getPurchaseReceivingReport`

Purpose:

- Report stock received from suppliers.

Allowed roles:

- Owner
- Manager
- Inventory Clerk, limited if supplier cost visibility is allowed

Inputs:

- Supplier
- Product
- Date range

Outputs:

- Receiving records
- Quantity received
- Cost totals, only for authorized roles

## 16.6 `getUserActivityReport`

Purpose:

- Show user-linked business activity.

Allowed roles:

- Owner
- Manager

Inputs:

- User
- Date range
- Activity type

Outputs:

- Sales by cashier
- Stock changes by user
- Transfers recorded by user
- Refunds approved by manager

## 17. Error and Validation Response Guidance

Common validation messages should be clear and business-friendly:

- Product is inactive and cannot be used for new transactions.
- Supplier is inactive and cannot be used for receiving.
- Branch destination is inactive and cannot receive new transfers.
- Quantity must be a positive whole number.
- Not enough stock available.
- Manager approval is required.
- Customer name and phone number are required for unpaid sales.
- This completed record cannot be edited directly. Use the correction workflow.

## 18. Recommended Implementation Order

Recommended operation build order:

1. Authentication and user operations.
2. Settings operations.
3. Product category, product, and barcode operations.
4. Inventory stock and movement read operations.
5. Supplier receiving operations.
6. Internal bakery receiving operations.
7. Branch destination and branch transfer operations.
8. Damage/expiry and stock count operations.
9. POS paid sale operations.
10. Unpaid sale operations.
11. Refund operations.
12. Reporting operations.
13. Audit log and advanced approval refinements.

## 19. Remaining API Decisions

Before implementation, confirm:

1. Whether the first version should expose APIs as REST endpoints, server actions, or another backend pattern depending on the chosen framework.
2. Whether manager approval is captured by password confirmation inside a workflow or by a separate manager session.
3. Whether stock count correction approval is always required or threshold-based.
4. Whether voiding sales is needed in MVP or whether refunds/corrections are sufficient.
5. Whether receipt numbers should be globally sequential, daily sequential, or include a date prefix.
