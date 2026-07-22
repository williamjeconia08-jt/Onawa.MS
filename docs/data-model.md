# Onawa Management System — Data Model Design

## 1. Document Purpose

This document converts the approved business rules into a practical MVP data model. It is not source code. It describes the main entities, their responsibilities, important fields, relationships, and validation rules so implementation can happen carefully phase by phase.

The model is designed for the main branch only. Smaller branches are represented as transfer destinations, not as fully managed inventory locations.

## 2. Design Principles

### DM-001: Transaction History Must Be Preserved

Completed business records should not be rewritten. Sales, refunds, stock movements, receiving records, and branch transfers must keep their historical values.

### DM-002: Inventory Is Movement-Based

Current stock should be explainable from stock movements. Each stock change must have a movement type, quantity, user, event time, entry time, and reference to the related business document where applicable.

### DM-003: Products Are Master Data, Transactions Store Snapshots

Products can change over time, but old transactions must keep the product name, selling price, cost, VAT rate, and quantity used at the time of the transaction.

### DM-004: Deactivation Is Preferred Over Deletion

Products, users, suppliers, and branches that have business history should be deactivated instead of deleted.

### DM-005: Main Branch Is the Only Stock Location in MVP

The system tracks stock for the main branch only. Smaller branches are only destinations for branch transfer records.

## 3. Core Reference Data

## 3.1 System Settings

Stores configurable business settings.

Important fields:

- `id`
- `business_name`
- `business_registration_details`
- `business_address`
- `default_currency_code`
- `default_currency_symbol`
- `default_vat_rate`
- `daily_reporting_cutoff_time`
- `created_at`
- `updated_at`

Default MVP values:

- Currency code: `NAD`
- Currency symbol: `N$`
- VAT rate: `15%`
- Daily reporting cutoff: `21:30`

Notes:

- Business registration details and address are required before final receipt printing is enabled.
- VAT rate and reporting cutoff should be configurable by authorized users.

## 3.2 Roles

Represents the high-level permission groups.

Important fields:

- `id`
- `name`
- `description`
- `created_at`
- `updated_at`

Required MVP roles:

- Owner
- Manager
- Cashier
- Inventory Clerk

## 3.3 Users

Represents people who can log in or be linked to business actions.

Important fields:

- `id`
- `full_name`
- `username`
- `password_hash`
- `role_id`
- `is_active`
- `created_at`
- `updated_at`
- `deactivated_at`

Relationships:

- Many users belong to one role.
- A user can create sales, stock movements, receiving records, branch transfers, refunds, and approvals.

Important rules:

- Deactivated users cannot log in.
- User records linked to historical transactions should be deactivated, not deleted.
- Passwords must never be stored as plain text.

## 4. Product Catalogue

## 4.1 Product Categories

Groups products for searching, reporting, and stock counting.

Important fields:

- `id`
- `name`
- `description`
- `is_active`
- `created_at`
- `updated_at`
- `deactivated_at`

Relationships:

- One category can have many products.

## 4.2 Products

Represents items or services sold by the business.

Important fields:

- `id`
- `name`
- `category_id`
- `product_type`
- `selling_price`
- `cost_price`
- `vat_rate`
- `stock_threshold`
- `is_active`
- `created_at`
- `updated_at`
- `deactivated_at`

Allowed `product_type` values:

- `NORMAL_STOCK`
- `INTERNAL_PRODUCTION_STOCK`
- `NON_STOCK_SERVICE`

Default values:

- `cost_price`: `0.00`
- `vat_rate`: current default VAT rate, initially `15%`
- `is_active`: true

Important rules:

- Product quantities use whole numbers in the MVP.
- Normal stock products and internal production stock products are stock-tracked.
- Non-stock service items, such as airtime, do not affect physical stock.
- Deactivated products cannot be selected for new sales, receiving, transfers, or adjustments.
- Historical sale lines must keep their own price and VAT snapshots.

## 4.3 Product Barcodes

Allows products to have zero, one, or many barcodes.

Important fields:

- `id`
- `product_id`
- `barcode_value`
- `is_active`
- `created_at`
- `updated_at`
- `deactivated_at`

Relationships:

- One product can have many barcodes.
- One barcode should point to one product only.

Important rules:

- Duplicate active barcodes are not allowed.
- Products without barcodes must still be searchable by name.
- Barcode changes must not affect old transaction history.

## 5. Suppliers and Receiving

## 5.1 Suppliers

Represents external suppliers for normal stock products.

Important fields:

- `id`
- `name`
- `contact_person`
- `phone_number`
- `email`
- `address`
- `notes`
- `is_active`
- `created_at`
- `updated_at`
- `deactivated_at`

Important rules:

- Inactive suppliers cannot be selected for new receiving records.
- Suppliers linked to receiving history should be deactivated, not deleted.

## 5.2 Supplier Product Links

Optional table for recording which suppliers provide which products.

Important fields:

- `id`
- `supplier_id`
- `product_id`
- `last_cost_price`
- `is_active`
- `created_at`
- `updated_at`

Relationships:

- One supplier can supply many products.
- One product can be supplied by many suppliers.

Notes:

- This table is useful but can be kept simple in the MVP.

## 5.3 Supplier Receiving Headers

Represents a receiving document from a supplier.

Important fields:

- `id`
- `supplier_id`
- `received_by_user_id`
- `approved_by_user_id`
- `document_reference`
- `event_time`
- `entry_time`
- `is_back_entered`
- `notes`
- `status`
- `created_at`
- `updated_at`

Suggested `status` values:

- `DRAFT`
- `COMPLETED`
- `CANCELLED`

Important rules:

- Completing a supplier receiving record increases stock for normal stock products.
- Bread must not be received through supplier receiving.

## 5.4 Supplier Receiving Lines

Represents individual products received from a supplier.

Important fields:

- `id`
- `supplier_receiving_id`
- `product_id`
- `product_name_snapshot`
- `quantity`
- `unit_cost`
- `line_total_cost`
- `created_at`
- `updated_at`

Important rules:

- Quantity must be a positive whole number.
- Unit cost is optional and can default to `0.00`.
- Completing a line creates an inventory movement of type `SUPPLIER_RECEIVING`.

## 6. Internal Bakery Receiving

## 6.1 Bakery Receiving Headers

Represents bread or other internal production stock received into the shop.

Important fields:

- `id`
- `received_by_user_id`
- `approved_by_user_id`
- `event_time`
- `entry_time`
- `is_back_entered`
- `notes`
- `status`
- `created_at`
- `updated_at`

Suggested `status` values:

- `DRAFT`
- `COMPLETED`
- `CANCELLED`

## 6.2 Bakery Receiving Lines

Represents individual internal production products received.

Important fields:

- `id`
- `bakery_receiving_id`
- `product_id`
- `product_name_snapshot`
- `quantity`
- `assumed_unit_cost`
- `created_at`
- `updated_at`

Important rules:

- Product type must be `INTERNAL_PRODUCTION_STOCK`.
- Quantity must be a positive whole number.
- Completing a line creates an inventory movement of type `INTERNAL_BAKERY_RECEIVING`.
- The MVP does not track recipes, ingredients, bakery labor, or manufacturing batches.

## 7. Inventory

## 7.1 Inventory Stock Balances

Stores current stock balances for quick viewing.

Important fields:

- `id`
- `product_id`
- `quantity_on_hand`
- `updated_at`

Relationships:

- One stock-tracked product has one main branch stock balance.

Important rules:

- Only stock-tracked products should have stock balances.
- Non-stock service products should not have stock balances.
- `quantity_on_hand` should not normally go below zero.
- Stock balances should be updated from completed inventory movements.

## 7.2 Inventory Movements

Records every stock change.

Important fields:

- `id`
- `product_id`
- `product_name_snapshot`
- `movement_type`
- `quantity_change`
- `quantity_before`
- `quantity_after`
- `reason`
- `reference_type`
- `reference_id`
- `event_time`
- `entry_time`
- `is_back_entered`
- `recorded_by_user_id`
- `approved_by_user_id`
- `created_at`

Suggested `movement_type` values:

- `SUPPLIER_RECEIVING`
- `INTERNAL_BAKERY_RECEIVING`
- `SALE_DEDUCTION`
- `BRANCH_TRANSFER_OUT`
- `REFUND_RETURN_TO_STOCK`
- `DAMAGE_WRITE_OFF`
- `EXPIRY_WRITE_OFF`
- `MANUAL_ADJUSTMENT`
- `STOCK_COUNT_CORRECTION`
- `REFUND_UNSELLABLE_WRITE_OFF`

Important rules:

- Positive quantity changes increase stock.
- Negative quantity changes decrease stock.
- Every movement must explain why stock changed.
- Every movement should reference the business document that caused it when applicable.

## 7.3 Stock Counts

Represents stock count or spot-check sessions.

Important fields:

- `id`
- `count_type`
- `started_by_user_id`
- `approved_by_user_id`
- `event_time`
- `entry_time`
- `is_back_entered`
- `notes`
- `status`
- `created_at`
- `updated_at`

Suggested `count_type` values:

- `SPOT`
- `CATEGORY`
- `HIGH_VALUE`
- `FAST_MOVING`
- `EXCEPTION`

## 7.4 Stock Count Lines

Represents counted products and variances.

Important fields:

- `id`
- `stock_count_id`
- `product_id`
- `product_name_snapshot`
- `expected_quantity`
- `counted_quantity`
- `variance_quantity`
- `reason`
- `created_at`
- `updated_at`

Important rules:

- Corrections from stock counts should create inventory movements of type `STOCK_COUNT_CORRECTION` after approval when required.

## 7.5 Damage and Expiry Records

Represents explicit damaged or expired goods.

Important fields:

- `id`
- `product_id`
- `product_name_snapshot`
- `quantity`
- `reason_type`
- `note`
- `event_time`
- `entry_time`
- `is_back_entered`
- `recorded_by_user_id`
- `approved_by_user_id`
- `status`
- `created_at`
- `updated_at`

Suggested `reason_type` values:

- `DAMAGED`
- `EXPIRED`

Important rules:

- Damage and expiry always require manager approval.
- Approved damage records create `DAMAGE_WRITE_OFF` inventory movements.
- Approved expiry records create `EXPIRY_WRITE_OFF` inventory movements.

## 8. Branch Transfers

## 8.1 Branch Destinations

Represents smaller branches that can receive stock from the main branch.

Important fields:

- `id`
- `name`
- `is_active`
- `created_at`
- `updated_at`
- `deactivated_at`

Launch values:

- Oshandi
- Ondangwa
- Omundaungilo

Important rules:

- Branches are transfer destinations only.
- The MVP does not track branch inventory or branch sales.
- Inactive branches cannot be selected for new transfers.

## 8.2 Branch Transfer Headers

Represents a transfer of stock from the main branch to a smaller branch.

Important fields:

- `id`
- `destination_branch_id`
- `recorded_by_user_id`
- `approved_by_user_id`
- `paper_reference`
- `event_time`
- `entry_time`
- `is_back_entered`
- `receiving_confirmed`
- `receiving_confirmed_by_name`
- `receiving_confirmed_at`
- `notes`
- `status`
- `created_at`
- `updated_at`

Suggested `status` values:

- `DRAFT`
- `COMPLETED`
- `CORRECTED`
- `CANCELLED`

Important rules:

- Completed transfers reduce main branch stock.
- Branch transfers are not sales.
- A completed transfer should be printable.
- Receiving confirmation should be recorded.

## 8.3 Branch Transfer Lines

Represents products and quantities in a branch transfer.

Important fields:

- `id`
- `branch_transfer_id`
- `product_id`
- `product_name_snapshot`
- `quantity`
- `created_at`
- `updated_at`

Important rules:

- Quantity must be a positive whole number.
- Product must be stock-tracked.
- Completing a line creates an inventory movement of type `BRANCH_TRANSFER_OUT`.

## 8.4 Branch Transfer Corrections

Represents changes made after a transfer has already been saved.

Important fields:

- `id`
- `branch_transfer_id`
- `corrected_by_user_id`
- `approved_by_user_id`
- `correction_reason`
- `created_at`

Important rules:

- Inventory clerks cannot correct already-saved transfers without manager approval.
- Corrections must explain who corrected the transfer, who approved it, when, and why.

## 9. Sales and Payments

## 9.1 Sale Headers

Represents a completed or pending sale.

Important fields:

- `id`
- `receipt_number`
- `cashier_user_id`
- `approved_by_user_id`
- `sale_status`
- `payment_method`
- `customer_name`
- `customer_phone`
- `subtotal_excluding_vat`
- `vat_total`
- `total_including_vat`
- `currency_symbol_snapshot`
- `event_time`
- `entry_time`
- `is_back_entered`
- `created_at`
- `updated_at`

Suggested `sale_status` values:

- `PAID`
- `UNPAID`
- `VOIDED`

Suggested `payment_method` values:

- `CASH`
- `CARD`
- `MOBILE`
- `UNPAID`

Important rules:

- Normal paid sales do not require customer name or phone.
- Unpaid sales require customer name, phone, and manager approval.
- Unpaid sales reduce stock immediately.
- Unpaid sales must be paid in full later; partial payment is not supported in MVP.
- Completed sale headers should not be edited like drafts.

## 9.2 Sale Lines

Represents products or services sold in a sale.

Important fields:

- `id`
- `sale_id`
- `product_id`
- `product_name_snapshot`
- `product_type_snapshot`
- `quantity`
- `unit_price_including_vat`
- `vat_rate_snapshot`
- `vat_amount`
- `line_total_including_vat`
- `created_at`

Important rules:

- Quantity must be a positive whole number.
- Sale lines must preserve price and VAT snapshots.
- Stock-tracked sale lines create `SALE_DEDUCTION` inventory movements.
- Non-stock service lines, such as airtime, do not create stock movements.
- Airtime can appear on the same receipt as physical products but must be categorized separately in reports.

## 9.3 Unpaid Sale Settlement

Represents the later settlement of an unpaid sale.

Important fields:

- `id`
- `sale_id`
- `marked_paid_by_user_id`
- `payment_method`
- `paid_at`
- `notes`
- `created_at`

Important rules:

- Only managers can mark unpaid sales as paid.
- Settlement must be for the full unpaid amount.
- Settlement must not modify the original sale lines.

## 10. Refunds

## 10.1 Refund Headers

Represents a refund transaction linked to an original sale where possible.

Important fields:

- `id`
- `original_sale_id`
- `processed_by_user_id`
- `approved_by_user_id`
- `refund_reason`
- `refund_total`
- `event_time`
- `entry_time`
- `is_back_entered`
- `status`
- `created_at`
- `updated_at`

Suggested `status` values:

- `DRAFT`
- `APPROVED`
- `COMPLETED`
- `CANCELLED`

Important rules:

- Refunds require manager approval.
- Refunds must not overwrite the original sale.
- Refunds must appear separately in reports.

## 10.2 Refund Lines

Represents individual refunded items.

Important fields:

- `id`
- `refund_id`
- `original_sale_line_id`
- `product_id`
- `product_name_snapshot`
- `quantity`
- `unit_price_including_vat_snapshot`
- `vat_rate_snapshot`
- `vat_amount`
- `line_refund_total`
- `return_to_stock`
- `unsellable_reason`
- `created_at`

Important rules:

- If `return_to_stock` is true, a `REFUND_RETURN_TO_STOCK` inventory movement is created.
- If the returned item is unsellable, it must not return to available stock.
- Unsellable returned stock should be tracked as a write-off where applicable.

## 11. Audit and Approval Support

## 11.1 Approval Records

Optional but recommended table for sensitive approvals.

Important fields:

- `id`
- `approval_type`
- `reference_type`
- `reference_id`
- `requested_by_user_id`
- `approved_by_user_id`
- `approved_at`
- `approval_note`
- `created_at`

Suggested `approval_type` values:

- `UNPAID_SALE`
- `REFUND`
- `DAMAGE_EXPIRY`
- `TRANSFER_CORRECTION`
- `STOCK_COUNT_CORRECTION`

Important rules:

- Approval records help prove who authorized sensitive actions.
- The related business record may also store `approved_by_user_id` for easier querying.

## 11.2 Audit Log

Optional table for important system events and administrative actions.

Important fields:

- `id`
- `user_id`
- `action`
- `entity_type`
- `entity_id`
- `old_values_summary`
- `new_values_summary`
- `created_at`

Important rules:

- Audit logging is especially useful for deactivation, deletion of mistake records, price changes, VAT changes, and permission changes.
- Audit logs should not replace business transaction records.

## 12. Entity Relationship Summary

High-level relationships:

- One role has many users.
- One category has many products.
- One product has many barcodes.
- One supplier can supply many products through supplier product links.
- One supplier receiving header has many supplier receiving lines.
- One bakery receiving header has many bakery receiving lines.
- One stock-tracked product has one inventory stock balance.
- One product has many inventory movements.
- One branch destination has many branch transfers.
- One branch transfer header has many branch transfer lines.
- One sale header has many sale lines.
- One unpaid sale can have one unpaid sale settlement.
- One sale can have many refunds.
- One refund header has many refund lines.
- One user can be responsible for many business actions.

## 13. Recommended Build Order From the Data Model

### Phase 1: Foundation, Authentication, and Users

Create:

- Roles
- Users
- Basic system settings

### Phase 2: Product Catalogue and Barcodes

Create:

- Product categories
- Products
- Product barcodes

### Phase 3: Inventory Core

Create:

- Inventory stock balances
- Inventory movements
- Stock counts
- Stock count lines
- Damage and expiry records

### Phase 4: Suppliers and Purchase Receiving

Create:

- Suppliers
- Supplier product links
- Supplier receiving headers
- Supplier receiving lines

### Phase 5: Internal Bakery Receiving

Create:

- Bakery receiving headers
- Bakery receiving lines

### Phase 6: Branch Transfers

Create:

- Branch destinations
- Branch transfer headers
- Branch transfer lines
- Branch transfer corrections

### Phase 7: Sales/POS

Create:

- Sale headers
- Sale lines
- Unpaid sale settlements
- Refund headers
- Refund lines

### Phase 8: Reports

Reports should read from the completed transaction tables, especially sale headers, sale lines, inventory movements, supplier receiving, branch transfers, refunds, and approvals.

## 14. Remaining Design Decisions

Before implementation, confirm:

1. Whether VAT should be rounded per line item or only at receipt total level.
2. Whether manager approval is captured by password confirmation inside a workflow or by the manager logging in separately.
3. Whether stock count corrections always require approval or only above a threshold.
4. The official business registration details and address for receipts.
5. Initial owner, manager, cashier, and inventory clerk accounts.
