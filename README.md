# Golden Coop - Sales, Inventory and Operations

Database design for a small fast-food restaurant business with two branches and one independent warehouse. This is a Database Management Systems course project built with **Java** and **MySQL**.

**Team:** Abdallah Hattab, Sondos Hammad

---

## Contents

1. [What the system does](#1-what-the-system-does)
2. [How to read this file](#2-how-to-read-this-file)
3. [Entity overview](#3-entity-overview)
4. [Entities and attributes](#4-entities-and-attributes)
5. [Relationships](#5-relationships)
6. [Many-to-many relationships](#6-many-to-many-relationships)
7. [Notes for drawing the ER diagram](#7-notes-for-drawing-the-er-diagram)
8. [Design rules](#8-design-rules)
9. [Derived attributes](#9-derived-attributes)
10. [Fixed value lists](#10-fixed-value-lists)
11. [Out of scope](#11-out-of-scope)

---

## 1. What the system does

The business has **two branches and one independent warehouse**. There is no separate head-office site or chicken preparation centre in this scope. Suppliers deliver chicken directly to the branches, where employees season and prepare it. Vegetables and soft drinks are also normally delivered directly to branches; seasonings, packaging and other supplies normally pass through the warehouse. Each purchase order explicitly identifies its delivery site.

The database covers these connected workflows:

| Flow | What happens |
|---|---|
| **Purchases** | An authorised manager orders stock for a branch or the warehouse, records deliveries and pays supplier invoices. |
| **Warehouse** | Warehouses hold stock and send it to branches when a branch asks. Stock is counted regularly. |
| **Sales** | A cashier records dine-in or takeaway orders, and kitchen staff update preparation status. Recipe quantities are deducted when preparation starts. |
| **Staff** | Each employee has a job title, a current monthly salary and recorded salary payments. |
| **Waste and counts** | Staff record known stock waste separately from physical count discrepancies. |
| **Maintenance** | Managers track faults at any site and assign requests to maintenance companies. |
| **Access** | Optional employee login accounts provide fixed application roles and site restrictions. |

How one unit of stock moves through the system:

An authorised manager creates a **purchase order** for one supplier.
The supplier delivers. A **goods receipt** records what was accepted and rejected, and the **stock level** of the receiving site goes up.
The delivery comes with an invoice. **Supplier payments** pay it off.
A branch asks its warehouse for stock with a **stock transfer**. When the warehouse sends it, the warehouse stock level goes down. When the branch confirms it, the branch stock level goes up.
A cashier saves an order. When kitchen staff start preparing it, the **recipe** quantities are deducted once from branch stock. Payment is recorded at handover in this first-version workflow.
Known spoilage of unused stock is recorded as **stock waste**. A **stock count** separately compares expected and actual quantities; a discrepancy is not automatically waste.
Managers record salary payments and follow maintenance requests through to completion.
---

## 2. How to read this file

| Mark | Meaning |
|---|---|
| **ID** | Identifier (primary key). If several attributes of one entity are marked ID, the identifier is the **combination** of them. |
| **FK** | Link to another entity (foreign key). On the ER diagram this is drawn as a **relationship line**, not as an ordinary attribute. |
| **D** | Derived attribute. It is calculated from other data and is **not stored**. Draw it with a dashed outline. |

Naming: every name is lowercase with underscores, and every entity name is singular.

---

## 3. Entity overview

There are **28 entities** and **40 relationships**: 39 one-to-many relationships and one optional one-to-one relationship between employee and user account. Only five entities were added to the original model.

| Group | Entities |
|---|---|
| Organisation and staff (4) | city, site, job_title, employee |
| Menu and sales (5) | menu_category, menu_item, menu_item_availability, sales_order, sales_order_line |
| Purchasing (8) | supplier, stock_item, supplier_item_price, purchase_order, purchase_order_line, goods_receipt, goods_receipt_line, supplier_payment |
| Stock (6) | recipe_line, stock_level, stock_transfer, stock_transfer_line, stock_count, stock_count_line |
| Access, salary payments, waste and maintenance (5) | user_account, salary_payment, stock_waste, maintenance_company, maintenance_request |

Two entities must not be confused:

**menu_item** is something the restaurant **sells** (a sandwich, a combo, a drink).
**stock_item** is something the restaurant **buys and stores** (buns, chicken, oil, cups, cleaning supplies).
recipe_line is the bridge between them.

---

## 4. Entities and attributes

### Organisation and staff

#### city

A city where the company has at least one site. Each city belongs to one operating area.

| Attribute | Key | Description |
|---|---|---|
| city_id | ID | Unique number of the city |
| city_name | | Name of the city, for example Nablus |
| area | | Operating area of the city: North, Centre or South |

#### site

Any place the business operates: a branch or a warehouse. Branches sell, warehouses store, and all of them can hold staff.

| Attribute | Key | Description |
|---|---|---|
| site_id | ID | Unique number of the site |
| site_name | | Name of the site, for example Nablus–Rafidia |
| site_type | | branch or warehouse |
| opening_year | | Year the site opened |
| address | | Street address and location details |
| phone | | Site contact number |
| city_id | FK → city | The city the site is in |
| supplied_by | FK → site | For a branch: the warehouse that supplies it. Empty for warehouses. Direct supplier deliveries to branches are still allowed. |

#### job_title

The list of job titles in the company, for example branch manager, assistant manager, cashier, kitchen employee, cleaner or warehouse keeper.

| Attribute | Key | Description |
|---|---|---|
| job_id | ID | Unique number of the job title |
| job_name | | Name of the job title |

#### employee

A person who works for the company. Each employee has one job title and works at one site.

| Attribute | Key | Description |
|---|---|---|
| employee_id | ID | Unique number of the employee |
| employee_name | | Full name |
| monthly_salary | | Current agreed monthly salary in ILS |
| phone | | Employee contact number |
| hire_date | | Employment start date |
| is_active | | yes / no; inactive employees remain in historical records |
| site_id | FK → site | The site where the employee works |
| job_id | FK → job_title | The employee's job title |

### Menu and sales

#### menu_category

A section of the menu: sandwiches, combo meals, buckets, sides, drinks, desserts, extra sauces.

| Attribute | Key | Description |
|---|---|---|
| category_id | ID | Unique number of the category |
| category_name | | Name of the category |

#### menu_item

One thing a customer can order. Combos and buckets are menu items too. The price is the same in every branch.

| Attribute | Key | Description |
|---|---|---|
| menu_item_id | ID | Unique number of the menu item |
| menu_item_name | | Name, for example Classic Chicken Sandwich |
| item_price | | Current selling price in ILS, VAT included |
| menu_item_category | FK → menu_category | The category the item belongs to |
| item_existence | | Whether the item is on the menu now (yes / no). A removed item is kept, because old orders still refer to it. |

#### menu_item_availability

Says whether one menu item can be ordered at one branch right now. A branch can switch an item off (for example when it runs out) without affecting other branches.

| Attribute | Key | Description |
|---|---|---|
| menu_item_id | ID, FK → menu_item | The menu item |
| site_id | ID, FK → site | The branch |
| availability | | Available or unavailable at this branch |

#### sales_order

One customer order taken at a branch by a cashier.

| Attribute | Key | Description |
|---|---|---|
| order_id | ID | Unique number of the order |
| order_date_time | | Date and time the order was taken |
| payment_method | | cash or card (one method per order) |
| vat_rate | | Configurable VAT percentage copied into the order; confirm the applicable rate with the client, with no hard-coded legal assumption |
| take_away_or_not | | yes = takeaway, no = dine-in |
| order_status | | new, preparing, ready, delivered or cancelled; replaces is_voided to avoid two conflicting status fields |
| preparation_started_at | | Nullable timestamp, set once when stock is deducted |
| paid_at | | Nullable payment timestamp; set at handover in this version |
| cancellation_reason | | Required for cancelled orders |
| cancelled_at | | Nullable cancellation timestamp |
| cancelled_by | FK → employee | Nullable approving manager; required for cancelled orders |
| site_id | FK → site | The branch where the order was taken |
| cashier_id | FK → employee | The employee who entered the order |
| order_total | D | Total of all lines |
| vat_amount | D | VAT part of the total |

#### sales_order_line

One menu item on one order, with how many were bought and the price charged.

| Attribute | Key | Description |
|---|---|---|
| order_id | ID, FK → sales_order | The order this line belongs to |
| menu_item_id | ID, FK → menu_item | The menu item sold |
| quantity | | How many were ordered |
| unit_price_charged | | Price of one item at the moment of sale. It is copied from the menu item so that old orders keep their original price. |

### Purchasing

#### supplier

A company that sells stock to the restaurant, such as a chicken, vegetable, beverage, seasoning or packaging supplier. A supplier may provide several material categories; its actual offerings are represented through supplier_item_price.

| Attribute | Key | Description |
|---|---|---|
| supplier_id | ID | Unique number of the supplier |
| supplier_name | | Name of the supplier |
| contact_details | | Phone number or email |
| address | | Supplier address |
| supplier_type | | local or foreign |
| payment_terms | | When invoices must be paid, for example 30 days from invoice, or paid in advance |

#### stock_item

Anything the company buys and keeps in stock: food ingredients, drinks, packaging, cleaning supplies.

| Attribute | Key | Description |
|---|---|---|
| stock_item_id | ID | Unique number of the stock item |
| item_name | | Name, for example bun or cooking oil |
| purchase_unit | | Unit it is bought in, for example pack, tin, carton |
| usable_unit | | Unit it is used and counted in, for example piece, litre, gram |
| conversion_factor | | How many usable units are in one purchase unit (a pack of 50 buns → 50) |

#### supplier_item_price

The price one supplier charges for one stock item, starting from a given date. Old prices are kept, so the history is never lost.

| Attribute | Key | Description |
|---|---|---|
| stock_item_id | ID, FK → stock_item | The stock item |
| supplier_id | ID, FK → supplier | The supplier |
| valid_date | ID | Date from which this price applies |
| unit_price | | Price of one purchase unit in ILS |

#### purchase_order

An order created by an authorised manager for one supplier, to be delivered to one branch or warehouse.

| Attribute | Key | Description |
|---|---|---|
| purchase_order_id | ID | Unique number of the purchase order |
| order_date | | Date the order was sent |
| supplier_id | FK → supplier | The supplier the order was sent to |
| site_id | FK → site | The site where the goods must be delivered |
| delivery_status | D | Not delivered, partly delivered or fully delivered |

#### purchase_order_line

One stock item on one purchase order.

| Attribute | Key | Description |
|---|---|---|
| purchase_order_id | ID, FK → purchase_order | The purchase order this line belongs to |
| stock_item_id | ID, FK → stock_item | The stock item ordered |
| order_quantity | | Quantity ordered, in purchase units |
| unit_price | | Price of one purchase unit agreed at the time of ordering |

#### goods_receipt

One delivery that arrived for a purchase order, together with the supplier's invoice for that delivery. One purchase order can arrive in several deliveries.

| Attribute | Key | Description |
|---|---|---|
| goods_receipt_id | ID | Unique number of the receipt |
| purchase_order_id | FK → purchase_order | The purchase order this delivery belongs to |
| receipt_date | | Date the goods arrived |
| invoice_number | | Number printed on the supplier's invoice |
| invoice_amount | | Total of the supplier's invoice in ILS |
| due_date | | Date by which the invoice must be paid |
| receipt_status | | draft or posted; posting adds accepted quantities to stock once |
| balance_due | D | What is still unpaid on this invoice |

#### goods_receipt_line

One stock item in one delivery, with how much was accepted and how much was rejected.

| Attribute | Key | Description |
|---|---|---|
| goods_receipt_id | ID, FK → goods_receipt | The delivery this line belongs to |
| stock_item_id | ID, FK → stock_item | The stock item delivered |
| item_accepted | | Quantity accepted, in purchase units |
| item_rejected | | Quantity rejected (damaged or unusable), in purchase units |

#### supplier_payment

One payment made to a supplier for one invoice. An invoice can be paid in several parts.

| Attribute | Key | Description |
|---|---|---|
| payment_id | ID | Unique number of the payment |
| goods_receipt_id | FK → goods_receipt | The delivery (and its invoice) being paid |
| payment_amount | | Amount paid in ILS |
| payment_date | | Date of the payment |

### Stock

#### recipe_line

One ingredient of one menu item: how much of a stock item is used each time the menu item is sold. This is what connects sales to stock.

| Attribute | Key | Description |
|---|---|---|
| stock_item_id | ID, FK → stock_item | The stock item used |
| menu_item_id | ID, FK → menu_item | The menu item it is used in |
| quantity_used | | Quantity used for one menu item, in usable units (for example 20 grams of lettuce) |

#### stock_level

How much of one stock item is at one site right now.

| Attribute | Key | Description |
|---|---|---|
| site_id | ID, FK → site | The site holding the stock |
| stock_item_id | ID, FK → stock_item | The stock item |
| quantity_on_hand | | Quantity at this site now, in usable units |
| reorder_level | | When the quantity falls below this number, more should be ordered |

#### stock_transfer

A movement of stock from a warehouse to a branch. It starts as a request from the branch, is then sent by the warehouse, and is finally confirmed by the branch.

| Attribute | Key | Description |
|---|---|---|
| stock_transfer_id | ID | Unique number of the transfer |
| request_date | | Date the branch asked for the stock |
| sent_date | | Date the warehouse sent it (empty until sent) |
| received_date | | Date the branch confirmed receipt (empty until received) |
| transfer_status | | requested, sent or received |
| transfer_from | FK → site | The warehouse that sends the stock |
| transfer_to | FK → site | The branch that receives the stock |

#### stock_transfer_line

One stock item on one transfer, with the three quantities that can differ.

| Attribute | Key | Description |
|---|---|---|
| stock_transfer_id | ID, FK → stock_transfer | The transfer this line belongs to |
| stock_item_id | ID, FK → stock_item | The stock item moved |
| requested_quantity | | Quantity the branch asked for, in usable units |
| sent | | Quantity the warehouse sent |
| received | | Quantity the branch confirmed on arrival |

#### stock_count

One physical count of stock at one site on one date.

| Attribute | Key | Description |
|---|---|---|
| count_id | ID | Unique number of the count |
| count_date | | Date and time of the physical count and expected-quantity snapshot |
| count_status | | draft or posted; posting applies the approved count adjustment once |
| site_id | FK → site | The site that was counted |

#### stock_count_line

The result for one stock item in one count.

| Attribute | Key | Description |
|---|---|---|
| count_id | ID, FK → stock_count | The count this line belongs to |
| stock_item_id | ID, FK → stock_item | The stock item counted |
| counted_quantity | | Quantity really found, in usable units |
| count_expected_quantity | | Quantity the system expected at the moment of counting |
| difference | D | Expected minus counted: positive means shortage, negative means surplus. This is a count discrepancy, not automatically waste. |

---

### Access, salary payments, waste and maintenance

#### user_account

An optional login account for an employee. Job title describes employment; account_role controls application permissions. Not every employee needs an account.

| Attribute | Key | Description |
|---|---|---|
| account_id | ID | Unique account number |
| employee_id | FK → employee, UNIQUE | Exactly one employee per account, at most one account per employee |
| username | UNIQUE | Unique login name |
| password_hash | | Password hash produced by a suitable password hashing library; never plaintext |
| account_role | | manager, cashier, kitchen or warehouse |
| is_active | | yes / no; disabling access preserves history |

#### salary_payment

One payment towards one employee's salary for one calendar month. Multiple instalments are allowed.

| Attribute | Key | Description |
|---|---|---|
| salary_payment_id | ID | Unique salary payment number |
| employee_id | FK → employee | Employee receiving payment |
| salary_month | | Calendar month, 1 through 12 |
| salary_year | | Calendar year |
| payment_amount | | Amount of this instalment in ILS |
| payment_date | | Date paid |

This entity records actual payments, including instalments, without duplicating the employee's salary in every payment row. Reports show the salary month and total paid. They must not infer historical salary balances from the employee's current salary: historical monthly entitlements and salary changes would require a separate payroll-period model. Full payroll accrual is outside this version.

#### stock_waste

One known loss of one unused stock material at one site. A separate line-detail entity is unnecessary.

| Attribute | Key | Description |
|---|---|---|
| waste_id | ID | Unique waste record number |
| site_id | FK → site | Site where stock was lost |
| stock_item_id | FK → stock_item | Material lost |
| quantity | | Positive quantity in usable units |
| reason | | Explanation, such as spoilage or accidental damage |
| waste_date_time | | Date and time of the loss |
| recorded_by | FK → employee | Employee recording the loss |
| waste_status | | draft or posted; posting deducts stock once |

This records losses of stock not already deducted through a recipe. A cancelled prepared order is recorded in the order history and is not deducted a second time as stock_waste.

#### maintenance_company

| Attribute | Key | Description |
|---|---|---|
| maintenance_company_id | ID | Unique maintenance company number |
| company_name | | Company name |
| contact_details | | Phone number or email |

#### maintenance_request

One fault reported at a branch or warehouse. Equipment is described in the request instead of adding an equipment entity.

| Attribute | Key | Description |
|---|---|---|
| maintenance_request_id | ID | Unique request number |
| site_id | FK → site | Site with the fault |
| maintenance_company_id | FK → maintenance_company | Nullable until the request is assigned |
| equipment_description | | Equipment name, location and identifying details if applicable |
| fault_description | | Description of the fault |
| reported_at | | Report date and time |
| request_status | | reported, assigned, in_progress, completed or cancelled |
| completed_at | | Nullable; required when completed |
| cost | | Nullable until known; nonnegative final cost in ILS when completed |

---

## 5. Relationships

Relationships 1-32 and 34-40 are **one-to-many (1:N)**. Relationship 33 is **one-to-one (1:1)** with an optional account on the employee side. Each FK belongs to exactly one parent unless explicitly nullable. A parent can have zero related rows unless a minimum is stated; for example a newly added menu item may have no sales yet.

Read each row in both directions, as in the "Meaning" column.

### Organisation and staff

| # | One | Many | Link attribute | Meaning |
|---|---|---|---|---|
| 1 | city | site | site.city_id | A city has zero or many sites. A site is in exactly one city. |
| 2 | site (warehouse) | site (branch) | site.supplied_by | A warehouse supplies zero or many branches. A branch is supplied by exactly one warehouse. Warehouses have no supplier site. |
| 3 | site | employee | employee.site_id | A site has many employees. An employee works at exactly one site. |
| 4 | job_title | employee | employee.job_id | A job title is held by many employees. An employee has exactly one job title. |

### Menu and sales

| # | One | Many | Link attribute | Meaning |
|---|---|---|---|---|
| 5 | menu_category | menu_item | menu_item.menu_item_category | A category contains many menu items. A menu item belongs to exactly one category. |
| 6 | menu_item | menu_item_availability | menu_item_id | A menu item has many availability rows (one per branch). Each row is about exactly one menu item. |
| 7 | site | menu_item_availability | site_id | A branch has many availability rows (one per menu item). Each row is about exactly one branch. |
| 8 | site | sales_order | sales_order.site_id | A branch takes many orders. An order is taken at exactly one branch. |
| 9 | employee | sales_order | sales_order.cashier_id | A cashier enters many orders. An order is entered by exactly one cashier. |
| 10 | sales_order | sales_order_line | order_id | An order has one or many lines. A line belongs to exactly one order. |
| 11 | menu_item | sales_order_line | menu_item_id | A menu item appears on many order lines. A line is for exactly one menu item. |

### Purchasing

| # | One | Many | Link attribute | Meaning |
|---|---|---|---|---|
| 12 | supplier | supplier_item_price | supplier_id | A supplier has many price rows. A price row belongs to exactly one supplier. |
| 13 | stock_item | supplier_item_price | stock_item_id | A stock item has many price rows, because several suppliers can sell it and prices change over time. A price row is for exactly one stock item. |
| 14 | supplier | purchase_order | purchase_order.supplier_id | A supplier receives many purchase orders. A purchase order is sent to exactly one supplier. |
| 15 | site | purchase_order | purchase_order.site_id | A site is the delivery place of many purchase orders. A purchase order is delivered to exactly one site. |
| 16 | purchase_order | purchase_order_line | purchase_order_id | A purchase order has one or many lines. A line belongs to exactly one purchase order. |
| 17 | stock_item | purchase_order_line | stock_item_id | A stock item appears on many purchase order lines. A line is for exactly one stock item. |
| 18 | purchase_order | goods_receipt | goods_receipt.purchase_order_id | A purchase order has zero or many receipts, because deliveries can be partial. A receipt belongs to exactly one purchase order. |
| 19 | goods_receipt | goods_receipt_line | goods_receipt_id | A receipt has one or many lines. A line belongs to exactly one receipt. |
| 20 | stock_item | goods_receipt_line | stock_item_id | A stock item appears on many receipt lines. A line is for exactly one stock item. |
| 21 | goods_receipt | supplier_payment | supplier_payment.goods_receipt_id | A receipt (invoice) has zero or many payments: none while unpaid, several if paid in parts. A payment is for exactly one receipt. |

### Stock

| # | One | Many | Link attribute | Meaning |
|---|---|---|---|---|
| 22 | menu_item | recipe_line | menu_item_id | A menu item has one or many recipe lines (one per ingredient). A recipe line belongs to exactly one menu item. |
| 23 | stock_item | recipe_line | stock_item_id | A stock item is used in many recipe lines. A recipe line uses exactly one stock item. |
| 24 | site | stock_level | site_id | A site has many stock levels (one per stock item). A stock level belongs to exactly one site. |
| 25 | stock_item | stock_level | stock_item_id | A stock item has many stock levels (one per site). A stock level is for exactly one stock item. |
| 26 | site | stock_transfer | stock_transfer.transfer_from | A warehouse sends many transfers. A transfer is sent from exactly one site. |
| 27 | site | stock_transfer | stock_transfer.transfer_to | A branch receives many transfers. A transfer goes to exactly one site. |
| 28 | stock_transfer | stock_transfer_line | stock_transfer_id | A transfer has one or many lines. A line belongs to exactly one transfer. |
| 29 | stock_item | stock_transfer_line | stock_item_id | A stock item appears on many transfer lines. A line is for exactly one stock item. |
| 30 | site | stock_count | stock_count.site_id | A site has many stock counts over time. A count is done at exactly one site. |
| 31 | stock_count | stock_count_line | count_id | A count has one or many lines. A line belongs to exactly one count. |
| 32 | stock_item | stock_count_line | stock_item_id | A stock item appears on many count lines. A line is for exactly one stock item. |

---

### Additional operational relationships

| # | Parent | Child | Link attribute | Meaning |
|---|---|---|---|---|
| 33 | employee | user_account | employee_id (UNIQUE) | Employee has zero or one account; every account belongs to exactly one employee. 1:1. |
| 34 | employee | salary_payment | employee_id | Employee has zero or many salary payments; each payment is for one employee. |
| 35 | site | stock_waste | site_id | Site has zero or many waste records; each record belongs to one site. |
| 36 | stock_item | stock_waste | stock_item_id | Material has zero or many waste records; each record identifies one material. |
| 37 | employee | stock_waste | recorded_by | Employee records zero or many losses; each loss has one recording employee. |
| 38 | site | maintenance_request | site_id | Site has zero or many requests; each request belongs to one site. |
| 39 | maintenance_company | maintenance_request | maintenance_company_id | Company handles zero or many requests; a request has zero or one assigned company. |
| 40 | employee | sales_order | cancelled_by | Manager approves zero or many cancellations; an order has zero or one cancellation approver. Separate from cashier relationship 9. |

## 6. Many-to-many relationships

There is no direct many-to-many line in this design. Each many-to-many relationship is carried by an entity that sits in the middle and stores the data that belongs to the pair.

| Many-to-many between | Entity in the middle | Data stored on the pair |
|---|---|---|
| menu_item and site | menu_item_availability | availability |
| sales_order and menu_item | sales_order_line | quantity, unit_price_charged |
| supplier and stock_item | supplier_item_price | valid_date, unit_price |
| purchase_order and stock_item | purchase_order_line | order_quantity, unit_price |
| goods_receipt and stock_item | goods_receipt_line | item_accepted, item_rejected |
| menu_item and stock_item | recipe_line | quantity_used |
| site and stock_item | stock_level | quantity_on_hand, reorder_level |
| stock_transfer and stock_item | stock_transfer_line | requested_quantity, sent, received |
| stock_count and stock_item | stock_count_line | counted_quantity, count_expected_quantity |

---

## 7. Notes for drawing the ER diagram

**19 entities have their own single identifier:** city, site, job_title, employee, menu_category, menu_item, sales_order, supplier, stock_item, purchase_order, goods_receipt, supplier_payment, stock_transfer, stock_count, user_account, salary_payment, stock_waste, maintenance_company, maintenance_request.
**9 entities are identified by a combination** (the nine in section 6). They cannot exist without their parents.
**site has a relationship with itself** (relationship 2, "supplied by"). Draw a line that leaves site and comes back to site.
**site and stock_transfer are connected twice** (relationships 26 and 27). Draw two separate lines and label them "from" and "to".
**employee and sales_order:** draw separate lines labelled "cashier" and "cancellation approved by". The latter is optional until cancellation.
**Do not draw FK attributes as ordinary attributes** if you use the classic notation with ovals. They are shown by the relationship line.
**Account relationship:** enforce UNIQUE(employee_id) in user_account; do not draw it as 1:N.
**Optional assignment:** maintenance_request may have no maintenance company until assigned.
**Draw the five derived attributes** (marked D) with a dashed outline.
---

## 8. Design rules

**Units.** Purchase quantities and supplier prices use purchase units. Recipes, counts, waste, transfers and stock levels use usable units. Accepted receipt quantity × conversion_factor is added on posting. Unit definitions and conversion factors cannot change after use in posted documents; introduce a new stock item for different packaging. Quantities use fixed decimal precision; currency uses decimal, never floating point.
**Historical values.** Sale and purchase lines retain their agreed unit prices. Orders retain their VAT rate and branch; invoices retain due dates. Historical documents are not recalculated using current prices. Salary payments retain their actual amounts and salary periods; they do not claim to preserve historical salary entitlements.
**One item per document.** Each item occurs once in an order, receipt, transfer or count. Sale-line notes apply to the whole line. To support different instructions for two units of the same item later, a line-number key would be needed; that is outside this version.
**Order lifecycle and payment.** Allowed normal transitions are new → preparing → ready → delivered. Orders are unpaid until handover; delivering records paid_at and the selected cash/card method in one transaction. Only delivered, paid orders count as revenue. This pay-at-handover assumption must be validated with the restaurant. No advance, split or partial payments are supported.
**Recipe deduction.** On new → preparing, check all required materials and deduct the aggregate recipe quantities once, setting preparation_started_at in the same transaction. Preparation happens in the branch; there is no prepared-chicken inventory or production entity. Raw chicken and seasonings are consumed according to standard recipe quantities, not measured preparation batches. Do not change recipes while affected orders are awaiting preparation. Never reconstruct old consumption using a changed current recipe.
**Cancellation.** A manager may cancel an undelivered order, recording reason, time and approver. Cancelling a new order has no stock effect. Cancelling after preparation begins does not restore ingredients, since they were already consumed; preparation_started_at preserves that distinction. Do not also deduct these ingredients as stock waste. Delivered/paid orders cannot be cancelled in this version: refunds require a separate future design. Cancelled orders are excluded from revenue.
**Supplier deliveries.** A purchase order targets exactly one site and supplier. Chicken goes directly to branches; vegetables and beverages normally do too. Packaging and seasonings normally go through the warehouse. Receipts may be partial; each receipt item must occur on its purchase order. Reject acceptance beyond the remaining ordered quantity. Rejected material does not enter stock. Receipt posting is immutable and occurs once.
**Invoices and supplier payments.** One delivery has one invoice in this simplified model. The invoice number is unique within the supplier, checked through the purchase-order relationship. Payments apply only to posted receipts, must be positive and must not exceed the unpaid balance. Dates and invoice amounts are validated; this is not full accounting.
**Transfers.** Only warehouse → its assigned branch is supported. At requested → sent, validate available stock and subtract sent quantities once. At sent → received, add confirmed quantities once. Require 0 ≤ received ≤ sent ≤ requested and positive requested quantities. Any sent-minus-received difference is reported as transfer loss, not deducted again. Sent transfers cannot be edited or cancelled. No additional deduction occurs on receipt confirmation.
**Known waste.** Posting a waste record deducts its positive usable-unit quantity once, after a stock check. Posted records are immutable. Waste is separate from unexplained count discrepancies and from ingredients already consumed by prepared orders.
**Physical counts.** A manager counts and approves a site under a short stock-movement freeze, capturing expected quantities. On posting, adjust each counted stock item to the actual counted quantity once. Only listed items are adjusted. An item already recorded as waste must not be deducted again. Count snapshots remain unchanged after posting; the freeze and posting transaction prevent concurrent sales or receipts from being overwritten.
**Atomic operations.** Stock changes and document state transitions commit together in a MySQL transaction, with appropriate row locks and conditional status checks. Repeated clicks cannot repeat an effect. Lock all affected stock rows consistently; reject negative stock. A posted document cannot be silently edited or deleted. Initial stock is established through a documented opening count, not unexplained manual edits.
**Staff and accounts.** Each employee has one current job and site. Deactivation preserves history and blocks account access. At most one account exists per employee; credentials are hashed, and authorization is enforced in Java for every operation, not only hidden buttons. Non-manager users operate only within their assigned site; managers have business-wide access in this small deployment. Only managers can see salaries, record salary payments or approve cancellations/counts. Employee reassignment disables their account until reviewed.
**Salary payments.** Allow instalments and require a valid month/year and positive payment amount. Show all existing payments for the selected employee/month before saving; a manager must verify the amount and avoid duplicate submission. The employee's monthly_salary is the current rate only. No automatic historical balance, overtime, deductions, attendance calculation or unpaid-month accrual is implemented. If the client requires automatic payroll settlement, extend the model rather than duplicating monthly salary entitlements across payment rows.
**Maintenance.** A reported request may be unassigned. Assignment requires a company; completion requires company, completion timestamp and final nonnegative cost. Completion cannot precede reporting. Equipment is described in the request without a separate asset register. Cost is recorded, but settlement of maintenance invoices is outside scope.
**Menu and history.** Combos are ordinary menu items with recipes; free extras may have zero price and their own recipe. Active menu status, branch availability and ingredient sufficiency are checked before preparation. All sellable items, including packaged drinks, have recipes. Historical items and employees are deactivated rather than deleted. Monetary values are in ILS; no legal tax rate is assumed by the design.
---

## 9. Derived attributes

| Attribute | Entity | How it is calculated |
|---|---|---|
| order_total | sales_order | Sum of quantity × unit_price_charged over the order's lines |
| vat_amount | sales_order | order_total × vat_rate / (100 + vat_rate), because prices include VAT |
| delivery_status | purchase_order | Use posted receipts only: not delivered if all accepted totals are zero; fully delivered if every line is satisfied; otherwise partly delivered |
| balance_due | goods_receipt | invoice_amount minus the sum of its payments |
| difference | stock_count_line | count_expected_quantity minus counted_quantity; positive shortage, negative surplus |

---

## 10. Fixed value lists

| Attribute | Allowed values |
|---|---|
| city.area | North, Centre, South |
| site.site_type | branch, warehouse |
| menu_item.item_existence | yes, no |
| menu_item_availability.availability | available, unavailable |
| sales_order.payment_method | cash, card |
| sales_order.take_away_or_not | yes (takeaway), no (dine-in) |
| sales_order.order_status | new, preparing, ready, delivered, cancelled |
| employee.is_active / user_account.is_active | yes, no |
| user_account.account_role | manager, cashier, kitchen, warehouse |
| goods_receipt.receipt_status / stock_count.count_status / stock_waste.waste_status | draft, posted |
| maintenance_request.request_status | reported, assigned, in_progress, completed, cancelled |
| supplier.supplier_type | local, foreign |
| stock_transfer.transfer_status | requested, sent, received |

---

## 11. Out of scope

Delivery orders, online orders and table reservations
Customer accounts and loyalty cards
Discounts and coupons
Full payroll, attendance, overtime, deductions and historical accrual of unpaid salary months (current salary and actual salary payments are included)
Full accounting (supplier invoices/payments, salary payments and maintenance costs are recorded, without a general ledger)
Customer refunds, paid-order cancellation, advance payments and split payments
Cash drawer sessions, equipment registers and detailed maintenance invoicing
Head-office sites, regional managers and separate chicken preparation/production facilities
Expiry dates and batch tracking
Vehicles and drivers
More than one currency (everything is in ILS)
---

## 12. Users, screens and reports

| Account role | Main screens and actions |
|---|---|
| cashier | Own-branch menu, new orders, line notes, order progress, payment at handover and printable receipts; no salary access |
| kitchen | Own-branch preparation queue; start preparation and mark ready; no salary or purchasing access |
| warehouse | Warehouse receipts, requested transfers, dispatch and waste entry; no salary access |
| manager | Employee/job setup, optional accounts, suppliers, purchasing, receipt confirmation at branches, transfer receipt confirmation, stock/count approval, cancellations, salaries, maintenance and reports |

Role permissions are fixed in application code to avoid additional role/permission entities. A screen may be shared by roles while enforcing site restrictions. The core order queue remains available to the manager when no kitchen account is assigned. Physical card processing is external; the application records the method and does not store card credentials.

Useful reports require queries rather than additional entities:

Paid sales by period, branch, category and cashier; top-selling items and order preparation times.
Current stock, low-stock alerts and estimated recipe requirements.
Purchase history, outstanding posted supplier invoices and overdue balances.
Requested/in-transit/completed transfers and sent-versus-received differences.
Recorded waste and physical count discrepancies, displayed separately.
Staff by site/job, current monthly salary totals and actual salary payments by month. Historical salary balances are not inferred.
Maintenance requests by status/site, completed maintenance costs and repeated equipment descriptions.
Implementation deliverables also include a connection configuration file, database setup/sample-data script, runnable Java application and a documented backup/restore procedure. This model is a design specification, not a claim that those features are implemented. Formal sample-query requirements and the drawn ER diagram must be prepared separately
