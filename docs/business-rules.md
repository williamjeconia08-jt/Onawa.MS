# Onawa Management System — Business Rules Document

## 1. Document Purpose

This Business Rules Document defines the operational rules for the Onawa Management System before implementation begins. It translates the business workflow into clear rules that can later guide the data model, user screens, APIs, permissions, validation, and reports.

The system is for the main branch of a mini-market/business. Smaller branches are treated only as stock transfer destinations in the MVP.

## 2. Business Scope

### 2.1 In Scope for MVP

The MVP must support:

- Authentication and role-based access.
- User management.
- Product catalogue management.
- Product categories.
- Default currency: Namibian dollars (N$).
- Barcode lookup, including multiple barcodes per product.
- VAT-inclusive product pricing.
- Stock-tracked normal products.
- Stock-tracked internal bakery bread products.
- Non-stock service or digital items such as airtime.
- Inventory stock levels and stock movement history.
- Supplier receiving for normal stock products.
- Internal bakery receiving for bread.
- Sales/POS with cash, card, and mobile payments.
- Unpaid sales with manager approval.
- Refunds with manager approval.
- Damage and expiry write-offs.
- Branch transfers from the main branch to smaller branches.
- Receiving branch confirmation for transfers.
- Printable transfer records.
- Basic business reports.
- Manual fallback and back-entered record flagging.

### 2.2 Out of Scope for MVP

The MVP must not include:

- Full customer account management.
- Customer loyalty.
- Discounts or promotions.
- Split payments.
- Full offline synchronization.
- Full multi-branch inventory.
- Smaller branch sales/POS tracking.
- Supplier debt tracking.
- Expense tracking.
- Full accounting.
- Full bakery manufacturing, recipes, or production costing.
- Advanced analytics dashboards.

## 3. Core Principles

### BR-001: Main Branch Only

The system manages the main branch only. Smaller branches are not independent inventory locations in the MVP.

### BR-002: Branch Transfers Are Not Sales

Stock sent to smaller branches must be recorded as internal branch transfers, not as sales revenue.

### BR-003: Inventory Must Explain Stock Changes

Every change to stock must be recorded as an inventory movement with a clear movement type, responsible user, date/time, quantity, and related business reference where applicable.

### BR-004: Historical Transactions Must Remain Stable

Completed sales, purchases, transfers, inventory movements, refunds, and approvals must not be silently rewritten. Corrections must be traceable.

### BR-005: Deactivate Real Records, Delete Only Mistakes

Records with business history must be deactivated rather than deleted. Deletion is allowed only for unused mistake records.

### BR-006: Manual Fallback Is Allowed During Rollout

If the system is unavailable or during the rollout period, staff may record transactions manually and enter them later as back-entered records.

## 4. Users, Roles, and Permissions

### 4.1 Roles

The system must support these roles:

- Owner
- Manager
- Cashier
- Inventory Clerk

### 4.2 Authentication Rules

- Every user must log in before using protected system functions.
- Deactivated users must not be allowed to log in.
- Authentication confirms identity only.
- Authorization controls permitted actions after login.

### 4.3 Authorization Rules

- Users must only access actions allowed by their role.
- Important business actions must record the responsible user.
- Sensitive actions must require manager or owner authority.

### 4.4 Owner Rules

The owner can:

- View all records and reports.
- View sales, inventory, transfer, purchase, refund, and user activity information.
- Have full visibility over the system.

### 4.5 Manager Rules

Managers can:

- Manage products.
- Approve unpaid sales.
- Mark unpaid sales as paid.
- Approve or process refunds.
- Handle branch transfers.
- Correct branch transfers with traceability.
- Deactivate records.
- Delete unused mistake records.
- View reports.
- Supervise back-entered records.

### 4.6 Cashier Rules

Cashiers can:

- Log in.
- Scan or search products.
- Process normal paid sales.
- Select one payment method per paid sale.
- Print or view receipts.
- Enter backlogged sales only under supervision if the business allows it.

Cashiers must not:

- Process refunds alone.
- Mark unpaid sales as paid.
- Change product prices.
- Adjust stock.
- Manage suppliers.
- Manage users.
- View profit or supplier cost reports.
- Change VAT settings.
- Delete or deactivate records.

### 4.7 Inventory Clerk Rules

Inventory clerks can:

- Record stock received.
- Record branch transfers.
- Correct transfers if allowed by policy.
- Record damage and expiry.
- Perform stock counts and spot checks.
- Enter backlogged inventory records.
- View stock reports.

Sensitive inventory corrections may require manager approval.

## 5. Products and Product Types

### 5.1 Product Master Rules

Each product should have:

- Product name.
- Category.
- Selling price.
- Cost price, which is optional and defaults to N$0.00 when not provided.
- Product type.
- Active or inactive status.
- VAT behavior.
- Stock threshold where applicable.
- Supplier relationship where applicable.

Product quantities should use whole numbers in the MVP. Decimal quantities are not required at launch.

### 5.2 Product Status Rules

- Active products can be used in new sales, purchases, receiving, transfers, and inventory actions according to product type.
- Inactive products must be hidden from normal selection screens.
- Inactive products must remain visible in historical records and reports.
- A product with transaction history should be deactivated, not deleted.

### 5.3 Product Type Rules

The system must distinguish these product behavior types:

1. Normal stock product.
2. Internal production stock product.
3. Non-stock service/digital item.

### 5.4 Normal Stock Product Rules

Normal stock products:

- Are purchased from suppliers.
- Are received into inventory.
- Are sold to customers.
- Can be transferred to smaller branches.
- Can be damaged or expired.
- Must not normally allow negative stock.
- May have an optional cost price that defaults to N$0.00 if not entered.

### 5.5 Internal Production Stock Product Rules

Internal production stock products, especially bread:

- Are produced by the business internally.
- Are received into the shop from the internal bakery.
- Are stock-tracked.
- Use whole-number quantities in the MVP.
- Can be sold to customers.
- Can be transferred to smaller branches.
- Can expire or be written off.
- Must not require full bakery manufacturing features in the MVP.

### 5.6 Non-Stock Service/Digital Item Rules

Non-stock service or digital items, especially airtime:

- Do not increase or decrease physical inventory.
- Can be sold through POS.
- Should be reportable separately from physical product sales.
- May appear on the same receipt as physical products, but must be categorized separately in receipts and reports.
- Should not be received through supplier stock receiving or bakery receiving.

## 6. Barcodes

### 6.1 Barcode Rules

- A product may have zero, one, or many barcodes.
- Each barcode must identify only one active product at a time.
- Product lookup must support barcode scanning.
- Product lookup must support manual barcode entry.
- Product lookup must support product name search for items without barcodes.

### 6.2 Barcode Integrity Rules

- Duplicate barcodes must not be allowed across products.
- Removing or deactivating a barcode must not change old sales history.
- Barcode changes should be traceable when practical.

## 7. VAT and Pricing

### 7.1 VAT Rules

- VAT is required for MVP.
- The default VAT rate is 15%.
- Shelf prices are VAT-inclusive.
- Receipts must show VAT information.
- VAT reports must be available.
- Product-level VAT exemption is not required for MVP.
- VAT rate should be configurable for future legal or business changes.

### 7.2 Tax-Inclusive Pricing Rules

- The selling price shown to customers already includes VAT.
- Prices and totals should use Namibian dollars (N$) by default.
- The customer must not be charged selling price plus VAT.
- For reporting, the system should calculate the VAT portion from the VAT-inclusive total.

### 7.3 Historical Price Rules

- Sale line items must store the price used at the time of sale.
- Sale line items must store the VAT rate or VAT amount used at the time of sale.
- Later product price changes must not affect old sales.

## 8. Sales and POS

### 8.1 Normal Sale Rules

A normal sale:

- Represents products or services given to a customer.
- Usually does not require customer information.
- Uses one payment method in MVP.
- May use cash, card, or mobile payment.
- Must produce a VAT-inclusive receipt.
- Must include receipt number, cashier name, business registration details, and business address once those business details are configured.
- Must reduce stock immediately for stock-tracked products.
- Must not reduce stock for non-stock service/digital items.
- May include airtime on the same receipt as physical products, but airtime must be categorized differently from physical stock items.

### 8.2 Sale Completion Rules

- A completed sale should not be edited like a draft.
- Corrections should use approved refund, void, or correction workflows.
- Completed sales must preserve original item names, quantities, prices, VAT values, and payment method.

### 8.3 Payment Rules

- The MVP supports cash, card, and mobile payments.
- Split payments are not supported in MVP.
- Partial payments for unpaid sales are not supported in MVP.
- Daily reports must separate totals by payment method.

### 8.4 Anonymous Customer Rules

- Normal paid sales must not require customer selection.
- Customer management must not slow down checkout in the MVP.

## 9. Unpaid Sales

### 9.1 Definition

An unpaid sale means goods have left the business, but payment has not yet been received.

### 9.2 Unpaid Sale Creation Rules

An unpaid sale must:

- Reduce stock immediately for stock-tracked products.
- Require customer/person name.
- Require phone number.
- Require manager approval.
- Preserve the original sale details.
- Be clearly separated from paid sales.

### 9.3 Unpaid Sale Payment Rules

- Only a manager can mark an unpaid sale as paid.
- Marking an unpaid sale as paid must record the date/time and responsible manager.
- Marking an unpaid sale as paid must record the payment method used.
- Unpaid sales must be settled in full; partial payment is not supported in the MVP.
- The original unpaid sale details must remain unchanged.

### 9.4 Unpaid Sale Reporting Rules

Reports must distinguish:

- Paid sales.
- Unpaid sales.
- Outstanding unpaid sales.
- Unpaid sales later marked as paid.
- Cash, card, and mobile money actually received.

## 10. Refunds

### 10.1 Refund Approval Rules

- Refunds require manager approval.
- Cashiers cannot process refunds alone.
- Refunds must require a reason.
- Refunds should be linked to the original sale where possible.

### 10.2 Refund Integrity Rules

- A refund must not overwrite or replace the original sale.
- A refund must be recorded as a separate transaction.
- Refunds must appear separately in reports.

### 10.3 Returned Stock Rules

- If returned goods are sellable, they may return to stock.
- If returned goods are damaged, expired, or not sellable, they must not return to available stock.
- Unsellable returned goods should be handled as a write-off.

## 11. Inventory Movements

### 11.1 Movement-Based Inventory Rule

Inventory must be movement-based. Current stock should be explainable from recorded stock movements.

### 11.2 Required Movement Information

Every inventory movement must record:

- Product.
- Quantity.
- Movement direction or signed quantity.
- Movement type.
- Reason.
- Responsible user.
- Event date/time.
- Entry date/time.
- Related reference where applicable.
- Back-entered flag where applicable.

### 11.3 Stock Increase Movement Types

Stock can increase through:

- Supplier receiving.
- Internal bakery receiving.
- Refund return to stock.
- Approved manual adjustment.
- Approved stock count correction.

### 11.4 Stock Decrease Movement Types

Stock can decrease through:

- Customer sale.
- Branch transfer out.
- Damage write-off.
- Expiry write-off.
- Manual adjustment.
- Stock count correction.
- Refund write-off for unsellable returned goods.

### 11.5 Negative Stock Rules

- Stock-tracked products should not normally go negative.
- Non-stock products do not have physical stock quantities.
- Any exceptional stock override, if allowed later, must require manager approval and must be traceable.

## 12. Supplier Receiving

### 12.1 Supplier Rules

Supplier records should include:

- Supplier name.
- Contact details.
- Products supplied where applicable.
- Cost information where useful.
- Active or inactive status.

### 12.2 Receiving Rules

- Supplier receiving applies to normal stock products.
- Receiving stock from a supplier must increase inventory.
- Receiving records must preserve purchase history.
- Receiving records should capture quantity, cost, supplier, date/time, user, and optional document reference.
- Cost is optional and should default to N$0.00 when not entered.
- Bread must not use normal supplier receiving in MVP.

## 13. Internal Bakery Receiving

### 13.1 Bread Receiving Rules

- Bread is a stock-tracked internal production product.
- Bread enters shop stock through internal bakery receiving.
- Internal bakery receiving must increase bread stock.
- The system should capture quantity, date/time, user, and optional note/reference.

### 13.2 Bakery Scope Rule

The MVP must not track bakery recipes, ingredients, oven batches, bakery labor, or full production costing.

## 14. Branch Transfers

### 14.1 Transfer Definition

A branch transfer is an internal movement where stock leaves the main branch and is sent to a smaller branch.

### 14.2 Transfer Rules

A branch transfer:

- Is not a sale.
- Is not paid for by the smaller branch.
- Reduces main branch stock.
- Must capture destination branch.
- Must capture products and quantities transferred.
- Must capture transfer date.
- Must capture responsible user.
- Should capture manager or inventory clerk responsible.
- Should capture optional paper reference or slip number.
- Must support receiving branch confirmation.
- Must be printable.

### 14.3 Transfer Destination Rules

- Smaller branches are transfer destinations only.
- The system must not track smaller branch stock levels in MVP.
- The system must not track smaller branch sales in MVP.
- Inactive branches must not be selectable for new transfers.
- Launch branch destinations are Oshandi, Ondangwa, and Omundaungilo. Additional branch destinations may be added later.

### 14.4 Transfer Correction Rules

- Transfer corrections must be traceable.
- Inventory clerks may record branch transfers, but once a transfer is saved, they must not correct it without manager approval.
- Corrections should record who made the correction, when, and why.
- Corrections must not make branch transfers appear as sales.

## 15. Damage and Expiry

### 15.1 Damage and Expiry Rules

- Damaged and expired goods must be recorded explicitly.
- Damage and expiry must not be hidden inside generic adjustments.
- Damage and expiry records must reduce available stock for stock-tracked products.

### 15.2 Required Damage/Expiry Information

Damage and expiry records should include:

- Product.
- Quantity.
- Reason: damaged or expired.
- Event date/time.
- Entry date/time.
- Person who recorded it.
- Optional note.
- Manager approval, which is always required for damage and expiry records.

### 15.3 Reporting Rules

Reports must show damage and expiry separately.

## 16. Stock Counts and Corrections

### 16.1 Stock Count Types

The system should support:

- Spot count for one product.
- Category count.
- High-value item count.
- Fast-moving product count.
- Exception-based count when stock seems wrong.

### 16.2 Stock Count Correction Rules

- Stock count corrections must be traceable.
- Corrections should record expected quantity, counted quantity, variance, user, date/time, and reason.
- Sensitive corrections may require manager approval.
- Saved branch transfer corrections require manager approval.

## 17. Deactivation and Deletion

### 17.1 Deactivation Rules

- Products, suppliers, users, and branches with history should be deactivated instead of deleted.
- Deactivated records must not be selectable for new transactions.
- Deactivated records must remain available for historical reporting.

### 17.2 Deletion Rules

- Deletion is allowed only for unused mistake records.
- Deletion should require manager authority.
- Deletion should not be allowed if a record is referenced by historical transactions.

## 18. Reports

### 18.1 Sales Reports

Sales reports should include:

- Daily sales.
- Paid sales.
- Unpaid sales.
- Outstanding unpaid sales.
- Sales by payment method.
- Refunds.
- VAT totals.
- Physical product sales versus airtime sales, even when airtime appears on the same receipt.

### 18.2 Inventory Reports

Inventory reports should include:

- Current stock levels.
- Low-stock items.
- Stock movement history.
- Damage and expiry losses.
- Stock adjustments.
- Stock count variances.

### 18.3 Branch Transfer Reports

Branch transfer reports should include:

- Transfers by branch.
- Transfers by product.
- Transfers by date.
- Transfer corrections.
- Transfer confirmation status.

### 18.4 Purchase Reports

Purchase reports should include:

- Stock received.
- Purchases by supplier.
- Cost history where useful, using N$ as the default currency.

### 18.5 User Activity Reports

User activity reports should include:

- Sales by cashier.
- Stock changes by user.
- Transfers recorded by user.
- Refunds approved by manager.

## 19. Manual Fallback and Back-Entered Records

### 19.1 Manual Fallback Rules

If the system is unavailable:

1. Staff record sales, transfers, purchases, and other actions manually.
2. Authorized staff enter those records into the system later.
3. Back-entered records must be flagged.
4. The system must store the actual event time and the system entry time.

### 19.2 Event Time and Entry Time Rules

- Event time means when the business event actually happened.
- Entry time means when the record was entered into the system.
- Reports should be able to distinguish back-entered records when needed.
- Daily reports should use 9:30 PM as the default reporting cutoff time. This cutoff should be configurable because the business may adjust it later.

### 19.3 Rollout Validation Rules

During rollout, the business should compare manual and system records for:

- Sales.
- Stock transfers.
- Purchases.
- VAT totals.
- Payment totals.
- Refunds.
- Unpaid sales.

The purpose is to prove that Onawa is reliable enough to become the official source of truth.

## 20. Key Risks and Controls

| Risk | Control |
| --- | --- |
| Branch transfers accidentally counted as sales | Separate transfer module, movement type, and reports |
| Bread stock becomes inaccurate | Require bread receiving records when bread enters shop stock |
| Unpaid sales are abused | Require customer name, phone, and manager approval |
| Refunds are misused | Require manager approval and separate refund records |
| VAT is calculated incorrectly | Use tax-inclusive calculation and preserve VAT per sale line |
| Barcode workflow is weaker than current business process | Support barcode lookup early and allow multiple barcodes per product |
| Customer features slow checkout | Keep normal sales anonymous in MVP |
| Offline features become overbuilt | Use manual fallback and back-entered records |
| Inventory quantity cannot be explained | Use movement-based inventory with clear movement types |

## 21. Confirmed Configuration Decisions

The following planning questions have been answered and should guide the data model and workflow design:

1. Default currency is Namibian dollars (N$).
2. Product quantities use whole numbers in the MVP. Decimal quantities are not required at launch.
3. Cost price is optional for stock-tracked products and should default to N$0.00 when not provided.
4. Inventory clerks may record branch transfers, but they must not correct already-saved branch transfers without manager approval.
5. Damage and expiry records always require manager approval.
6. Unpaid sales do not support partial later payment in the MVP. They must be settled in full.
7. Receipts should include business registration details, business address, cashier name, and receipt number.
8. Airtime may be included on the same receipt as physical products, but it must be categorized separately.
9. Launch branch destinations are Oshandi, Ondangwa, and Omundaungilo. Additional branch destinations may be added later.
10. The default daily reporting cutoff time is 9:30 PM. This should be adjustable later.

## 22. Remaining Open Questions Before Data Model Design

These questions still need confirmation before implementation details are finalized:

1. What are the official business registration details and address to show on receipts?
2. What exact user accounts should be created first for owner, managers, cashiers, and inventory clerks?
3. Should manager approval happen by manager login/password confirmation at the cashier screen, or only by a manager using their own account separately?
4. Should stock count corrections always require manager approval, or only when the variance is above a certain quantity/value?
5. Should VAT values be rounded per line item or only at receipt total level?

## 23. Next Planning Deliverables

After this Business Rules Document, the recommended next deliverables are:

1. Data model and entity relationship design.
2. Screen and workflow design.
3. API and business operation planning.
4. Phase-by-phase implementation plan.
5. Phase 1 implementation: foundation, authentication, and users.
