# Onawa Management System — Screen and Workflow Design

## 1. Document Purpose

This document describes the main screens and user workflows for the Onawa Management System MVP. It uses the approved business rules and data model as inputs, but it does not contain implementation code.

The goal is to define how users will move through the system before designing APIs or writing application code.

## 2. User Experience Principles

### UX-001: Keep Cashier Work Fast

Cashier workflows must be simple and fast. Normal sales should not require customer selection or unnecessary steps.

### UX-002: Separate Sales From Stock Transfers

Branch transfers must have their own screens and workflows so they are not confused with customer sales.

### UX-003: Make Sensitive Actions Clearly Approved

Unpaid sales, refunds, damage/expiry records, saved transfer corrections, and sensitive stock corrections must clearly show manager approval.

### UX-004: Show Business Meaning, Not Just Data

Screens should explain business status clearly, such as unpaid, paid, confirmed, awaiting approval, damaged, expired, low stock, or back-entered.

### UX-005: Support Manual Fallback Entry

Where relevant, screens should allow authorized users to mark records as back-entered and capture the real event time separately from the entry time.

## 3. Navigation Structure

The MVP navigation should be grouped around business tasks:

- Dashboard
- POS / Sales
- Products
- Inventory
- Suppliers / Receiving
- Bakery Receiving
- Branch Transfers
- Refunds
- Reports
- Users and Roles
- Settings

Role visibility should control which navigation items appear to each user.

## 4. Login Workflow

### 4.1 Login Screen

Purpose:

- Allow authorized users to access the system.

Fields:

- Username
- Password

Actions:

- Log in
- Show validation error for incorrect credentials
- Block login for deactivated users

Business rules:

- Authentication confirms identity.
- Authorization controls what the logged-in user can access.
- Deactivated users must not log in.

Success result:

- User is taken to the correct home/dashboard view for their role.

## 5. Dashboard Workflow

### 5.1 Dashboard Screen

Purpose:

- Give users a quick summary of the most relevant information for their role.

Owner and manager dashboard may show:

- Today sales total
- Sales by payment method
- Outstanding unpaid sales
- Low-stock count
- Pending approvals
- Recent branch transfers
- Damage/expiry summary

Cashier dashboard may show:

- Start sale button
- Recent receipts handled by the cashier
- Any supervised/back-entry tasks if allowed

Inventory clerk dashboard may show:

- Low-stock items
- Recent receiving records
- Recent branch transfers
- Pending stock count tasks
- Damage/expiry records awaiting approval

Business rules:

- Cashiers should not see profit or supplier cost information.
- Owners and managers should have broader visibility.

## 6. User Management Workflow

### 6.1 User List Screen

Purpose:

- View system users and their current status.

Visible to:

- Owner
- Manager, if allowed by final policy

Displayed information:

- Full name
- Username
- Role
- Active/inactive status
- Created date

Actions:

- Create user
- Edit user
- Deactivate user
- View user activity summary

Business rules:

- Deactivated users cannot log in.
- Users linked to history should be deactivated, not deleted.

### 6.2 Create/Edit User Screen

Fields:

- Full name
- Username
- Role
- Password or reset password action
- Active status

Validation:

- Username is required.
- Role is required.
- Password is required for new users.

## 7. Product Catalogue Workflow

### 7.1 Product List Screen

Purpose:

- Search and manage products.

Search/filter options:

- Product name
- Barcode
- Category
- Product type
- Active/inactive status
- Low-stock status

Displayed information:

- Product name
- Category
- Product type
- Selling price
- Cost price, where user role is allowed
- Current stock for stock-tracked products
- Stock threshold
- Active/inactive status

Actions:

- Create product
- Edit product
- Manage barcodes
- Deactivate product
- View stock movement history

Business rules:

- Cashiers can search products but cannot change product setup.
- Deactivated products should be hidden from normal sale selection.

### 7.2 Create/Edit Product Screen

Fields:

- Product name
- Category
- Product type
- Selling price
- Cost price, optional and defaulting to N$0.00
- VAT rate, defaulting to 15%
- Stock threshold
- Active status

Product type options:

- Normal stock product
- Internal production stock product
- Non-stock service/digital item

Validation:

- Product name is required.
- Selling price is required.
- Quantity-related fields apply only to stock-tracked products.
- Cost price can be blank and should default to N$0.00.

### 7.3 Product Barcode Screen

Purpose:

- Add and manage one or many barcodes for a product.

Fields:

- Barcode value
- Active status

Actions:

- Add barcode
- Deactivate barcode

Validation:

- Barcode value must be unique among active barcodes.
- Products can exist without barcodes.

## 8. Inventory Workflow

### 8.1 Current Stock Screen

Purpose:

- Show current main-branch stock levels.

Search/filter options:

- Product name
- Barcode
- Category
- Product type
- Low-stock only

Displayed information:

- Product
- Category
- Product type
- Quantity on hand
- Stock threshold
- Low-stock indicator

Actions:

- View movement history
- Record adjustment, if authorized
- Start stock count or spot check
- Record damage/expiry

Business rules:

- Non-stock service products should not show physical stock balances.
- Stock should not normally go negative.

### 8.2 Inventory Movement History Screen

Purpose:

- Explain why stock changed.

Filters:

- Product
- Movement type
- Date range
- User
- Reference type
- Back-entered only

Displayed information:

- Product
- Movement type
- Quantity before
- Quantity change
- Quantity after
- Reason
- User
- Event time
- Entry time
- Related reference

Business rules:

- Every stock change should be traceable.
- Movement history should be read-only except through correction workflows.

## 9. Supplier Receiving Workflow

### 9.1 Supplier List Screen

Purpose:

- Manage supplier records.

Displayed information:

- Supplier name
- Contact person
- Phone number
- Active/inactive status

Actions:

- Create supplier
- Edit supplier
- Deactivate supplier
- View supplier receiving history

Business rules:

- Inactive suppliers cannot be used for new receiving records.

### 9.2 Supplier Receiving Screen

Purpose:

- Record stock received from external suppliers.

Header fields:

- Supplier
- Document reference
- Event time
- Back-entered flag
- Notes

Line fields:

- Product
- Quantity
- Unit cost, optional and defaulting to N$0.00

Actions:

- Save draft
- Complete receiving
- Cancel draft

Validation:

- Supplier is required.
- Product must be a normal stock product.
- Quantity must be a positive whole number.
- Bread/internal production products cannot be received here.

Completion result:

- Stock increases.
- Inventory movement records are created.

## 10. Internal Bakery Receiving Workflow

### 10.1 Bakery Receiving Screen

Purpose:

- Record bread or other internal production stock received into the shop.

Header fields:

- Event time
- Back-entered flag
- Notes

Line fields:

- Product
- Quantity
- Assumed unit cost, optional

Actions:

- Save draft
- Complete receiving
- Cancel draft

Validation:

- Product must be an internal production stock product.
- Quantity must be a positive whole number.

Completion result:

- Bread/shop stock increases.
- Inventory movement records are created.

Business warning:

- Bread stock reports are only reliable if staff record bread received from the bakery consistently.

## 11. Branch Transfer Workflow

### 11.1 Branch Transfer List Screen

Purpose:

- View and manage transfers from the main branch to smaller branches.

Filters:

- Destination branch
- Date range
- Confirmation status
- Recorded by user
- Back-entered only

Displayed information:

- Transfer number
- Destination branch
- Transfer date
- Recorded by
- Confirmation status
- Status

Actions:

- Create transfer
- View transfer
- Print transfer
- Confirm receiving
- Correct transfer, if authorized

Business rules:

- Branch transfers are not sales.
- Transfers reduce main branch stock.

### 11.2 Create Branch Transfer Screen

Header fields:

- Destination branch
- Paper reference/slip number
- Event time
- Back-entered flag
- Notes

Line fields:

- Product
- Quantity

Actions:

- Save draft
- Complete transfer
- Cancel draft
- Print transfer document

Validation:

- Destination branch is required and must be active.
- Product must be stock-tracked.
- Quantity must be a positive whole number.
- Available stock must normally be enough for the transfer.

Completion result:

- Main branch stock decreases.
- Inventory movement records are created with movement type `BRANCH_TRANSFER_OUT`.

### 11.3 Transfer Confirmation Workflow

Purpose:

- Record that the receiving branch confirmed the stock.

Fields:

- Receiving confirmed flag
- Receiving confirmed by name
- Confirmation date/time
- Optional note

Business rules:

- Confirmation records what was received by the destination branch.
- The MVP does not create branch inventory records.

### 11.4 Transfer Correction Workflow

Purpose:

- Correct already-saved transfers with traceability.

Fields:

- Correction reason
- Corrected lines
- Manager approval

Business rules:

- Inventory clerks cannot correct already-saved transfers without manager approval.
- Corrections must record who corrected the transfer, who approved it, when, and why.

## 12. POS Sales Workflow

### 12.1 POS Sale Screen

Purpose:

- Allow cashiers to sell quickly.

Main areas:

- Barcode/manual entry box
- Product search
- Cart/sale lines
- Payment method selection
- Sale total and VAT summary
- Complete sale button
- Unpaid sale request option, if manager approval is available

Line information:

- Product name
- Quantity
- Unit price including VAT
- Line total
- Product category/type indicator, especially for airtime

Actions:

- Scan barcode
- Search product by name
- Add item
- Change quantity
- Remove item before completion
- Complete paid sale
- Request unpaid sale approval
- Print/view receipt

Validation:

- Product must be active.
- Stock-tracked products must have enough available stock.
- Quantity must be a positive whole number.
- One payment method is allowed per paid sale.

Completion result:

- Sale is recorded.
- Stock decreases for stock-tracked products.
- Airtime/non-stock lines do not reduce inventory.
- Receipt is generated with receipt number, cashier name, VAT information, business registration details, and business address.

### 12.2 Receipt View/Print Screen

Purpose:

- Show and print completed sale receipt.

Receipt should show:

- Business name
- Business registration details
- Business address
- Receipt number
- Date/time
- Cashier name
- Items sold
- Airtime/category distinction where applicable
- VAT-inclusive totals
- VAT amount
- Payment method

Business rules:

- Receipt values must come from sale snapshots, not current product data.

## 13. Unpaid Sale Workflow

### 13.1 Create Unpaid Sale From POS

Purpose:

- Allow goods to leave before payment only under manager approval.

Required fields:

- Customer/person name
- Phone number
- Manager approval
- Reason/note, recommended

Business rules:

- Unpaid sales are rare.
- Unpaid sales reduce stock immediately.
- Customer name and phone number are required.
- Partial later payment is not supported in MVP.

### 13.2 Outstanding Unpaid Sales Screen

Purpose:

- Let managers find and settle unpaid sales.

Filters:

- Customer/person name
- Phone number
- Date range
- Outstanding only

Displayed information:

- Sale/receipt number
- Customer/person name
- Phone number
- Amount outstanding
- Sale date
- Approved by

Actions:

- View original sale
- Mark as paid, manager only

### 13.3 Mark Unpaid Sale as Paid Workflow

Fields:

- Payment method
- Paid date/time
- Notes

Business rules:

- Only manager can mark unpaid sales as paid.
- Settlement must be for the full unpaid amount.
- Original sale lines must not be changed.

## 14. Refund Workflow

### 14.1 Refund List Screen

Purpose:

- View refunds separately from sales.

Filters:

- Date range
- Original receipt number
- Approved by
- Processed by

Displayed information:

- Refund number
- Original sale/receipt number
- Refund amount
- Reason
- Approved by
- Status

### 14.2 Create Refund Screen

Purpose:

- Record manager-approved refunds.

Fields:

- Original sale/receipt reference, where possible
- Refund reason
- Refunded items
- Return-to-stock decision per item
- Unsellable reason if not returned to stock
- Manager approval

Business rules:

- Refunds require manager approval.
- Refunds must not overwrite original sales.
- Sellable returned goods may return to stock.
- Unsellable returned goods must not return to available stock.

Completion result:

- Refund record is created.
- Stock is increased only for returned sellable stock-tracked items.
- Refund appears separately in reports.

## 15. Damage and Expiry Workflow

### 15.1 Damage/Expiry List Screen

Purpose:

- View damage and expiry records.

Filters:

- Product
- Reason type
- Date range
- Approval status
- Recorded by

Displayed information:

- Product
- Quantity
- Reason type
- Recorded by
- Approved by
- Event time
- Status

### 15.2 Record Damage/Expiry Screen

Fields:

- Product
- Quantity
- Reason type: damaged or expired
- Event time
- Back-entered flag
- Note
- Manager approval

Business rules:

- Damage and expiry always require manager approval.
- Approved records reduce stock.
- Damage and expiry should report separately.

## 16. Stock Count Workflow

### 16.1 Start Stock Count Screen

Fields:

- Count type
- Category, if category count
- Notes
- Event time

Count types:

- Spot
- Category
- High-value
- Fast-moving
- Exception

### 16.2 Enter Count Lines Screen

Fields:

- Product
- Expected quantity
- Counted quantity
- Variance
- Reason/note

Business rules:

- Corrections should be traceable.
- Approval policy for stock count corrections still needs final confirmation.

## 17. Reports Workflow

### 17.1 Reports Home Screen

Purpose:

- Give owner and manager access to business reports.

Report groups:

- Sales reports
- Inventory reports
- Branch transfer reports
- Purchase reports
- User activity reports

Business rules:

- Cashiers should not see profit or supplier cost reports.
- Reports should use the default 9:30 PM cutoff unless changed in settings.

### 17.2 Sales Reports

Reports:

- Daily sales
- Paid sales
- Unpaid sales
- Outstanding unpaid sales
- Sales by payment method
- Refunds
- VAT totals
- Physical product sales versus airtime sales

### 17.3 Inventory Reports

Reports:

- Current stock levels
- Low-stock items
- Stock movement history
- Damage/expiry losses
- Stock adjustments
- Stock count variances

### 17.4 Branch Transfer Reports

Reports:

- Transfers by branch
- Transfers by product
- Transfers by date
- Transfer corrections
- Transfer confirmation status

### 17.5 Purchase Reports

Reports:

- Stock received
- Purchases by supplier
- Cost history where useful

### 17.6 User Activity Reports

Reports:

- Sales by cashier
- Stock changes by user
- Transfers recorded by user
- Refunds approved by manager

## 18. Settings Workflow

### 18.1 Business Settings Screen

Fields:

- Business name
- Business registration details
- Business address
- Default currency code
- Default currency symbol
- VAT rate
- Daily reporting cutoff time

Default values:

- Currency code: NAD
- Currency symbol: N$
- VAT rate: 15%
- Daily reporting cutoff: 21:30

Business rules:

- VAT rate changes should be restricted to authorized users.
- Reporting cutoff changes should be restricted to authorized users.
- Historical sales must keep the VAT rate used at the time of sale.

## 19. Role-Based Screen Access Summary

| Screen/Area | Owner | Manager | Cashier | Inventory Clerk |
| --- | --- | --- | --- | --- |
| Dashboard | Full | Full operational | Limited cashier | Inventory-focused |
| POS / Sales | View/use | View/use/approve | Use normal sales | View if needed |
| Unpaid Sales | View | Approve/settle | Request only | View if allowed |
| Refunds | View | Approve/process | No independent access | View if allowed |
| Products | Full | Manage | Search only | View/search |
| Inventory | Full | Manage/approve | No stock adjustment | Record/manage allowed tasks |
| Suppliers | Full | Manage | No access | Record receiving if allowed |
| Bakery Receiving | Full | Manage | No access | Record receiving |
| Branch Transfers | Full | Manage/correct | No access | Record, but saved corrections need manager approval |
| Reports | Full | Operational reports | Limited/no sensitive reports | Stock reports |
| Users/Roles | Full | Manage if allowed | No access | No access |
| Settings | Full | Limited authorized settings | No access | No access |

## 20. MVP Workflow Build Order

Recommended screen/workflow implementation order:

1. Login and role-based navigation.
2. Basic dashboard.
3. Users and roles.
4. Product categories, products, and barcodes.
5. Current stock and movement history.
6. Supplier receiving.
7. Internal bakery receiving.
8. Branch transfers.
9. Damage and expiry.
10. POS paid sales.
11. Unpaid sales.
12. Refunds.
13. Reports.
14. Settings and rollout/back-entry refinements.

## 21. Remaining Workflow Decisions

Before implementation, confirm:

1. Should manager approval be done by entering manager credentials inside the same screen, or should managers approve from their own logged-in session?
2. Should stock count corrections always require manager approval, or only when above a threshold?
3. Should the POS allow cashier quantity changes by typing only, plus/minus buttons, or both?
4. Should receipt printing happen automatically after sale completion, or should the cashier choose print/view?
5. Should branch transfer receiving confirmation be entered by main-branch staff after paper confirmation returns, or by someone from the receiving branch later?
