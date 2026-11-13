# Golden Coop – Sales and Inventory Database

Database design for a fast-food restaurant chain. This is a Database Management Systems course project built with **Java** and **MySQL**.

**Team:** Abdallah Hattab, Sondos

> Golden Coop is a fictional company invented for this project. It is not a real brand.

This file describes the data model: every entity, its attributes, and every relationship. It is written so that anyone can draw the ER diagram from it without asking questions.

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

Golden Coop has 6 branches, 2 warehouses and a head office. The database covers three flows that are connected to each other:

| Flow | What happens |
|---|---|
| **Purchases** | Head office orders stock from suppliers, receives the deliveries and pays the invoices. |
| **Warehouse** | Warehouses hold stock and send it to branches when a branch asks. Stock is counted regularly. |
| **Sales** | Customers order at a branch (dine-in or takeaway). Each sale uses up stock at that branch. |

How one unit of stock moves through the system:

1. The purchasing officer creates a **purchase order** for one supplier.
2. The supplier delivers. A **goods receipt** records what was accepted and rejected, and the **stock level** of the receiving site goes up.
3. The delivery comes with an invoice. **Supplier payments** pay it off.
4. A branch asks its warehouse for stock with a **stock transfer**. When the warehouse sends it, the warehouse stock level goes down. When the branch confirms it, the branch stock level goes up.
5. A customer buys something. The **sales order** lines name the menu items sold, and the **recipe** of each menu item says how much stock to take out of the branch.
6. A **stock count** compares what should be there with what is really there. The difference is the waste.

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

There are **23 entities** and **32 relationships**.

| Group | Entities |
|---|---|
| Organisation and staff (4) | `city`, `site`, `job_title`, `employee` |
| Menu and sales (5) | `menu_category`, `menu_item`, `menu_item_availability`, `sales_order`, `sales_order_line` |
| Purchasing (8) | `supplier`, `stock_item`, `supplier_item_price`, `purchase_order`, `purchase_order_line`, `goods_receipt`, `goods_receipt_line`, `supplier_payment` |
| Stock (6) | `recipe_line`, `stock_level`, `stock_transfer`, `stock_transfer_line`, `stock_count`, `stock_count_line` |

Two entities must not be confused:

- **`menu_item`** is something the chain **sells** (a sandwich, a combo, a drink).
- **`stock_item`** is something the chain **buys and stores** (buns, chicken, oil, cups, cleaning supplies).

`recipe_line` is the bridge between them.

---

## 4. Entities and attributes

### Organisation and staff

#### `city`

A city where the company has at least one site. Each city belongs to one operating area.

| Attribute | Key | Description |
|---|---|---|
| city_id | ID | Unique number of the city |
| city_name | | Name of the city, for example Nablus |
| area | | Operating area of the city: North, Centre or South |

#### `site`

Any place the company operates: a branch, a warehouse or the head office. Branches sell, warehouses store, and all of them can hold staff.

| Attribute | Key | Description |
|---|---|---|
| site_id | ID | Unique number of the site |
| site_name | | Name of the site, for example Nablus–Rafidia |
| site_type | | branch, warehouse or head office |
| opening_year | | Year the site opened |
| city_id | FK → city | The city the site is in |
| supplied_by | FK → site | For a branch: the warehouse that supplies it. Empty for warehouses and the head office. |

#### `job_title`

The list of job titles in the company, for example branch manager, assistant manager, cashier, cook.

| Attribute | Key | Description |
|---|---|---|
| job_id | ID | Unique number of the job title |
| job_name | | Name of the job title |

#### `employee`

A person who works for the company. Each employee has one job title and works at one site.

| Attribute | Key | Description |
|---|---|---|
| employee_id | ID | Unique number of the employee |
| employee_name | | Full name |
| monthly_salary | | Monthly salary in ILS |
| site_id | FK → site | The site where the employee works |
| job_id | FK → job_title | The employee's job title |

### Menu and sales

#### `menu_category`

A section of the menu: sandwiches, combo meals, buckets, sides, drinks, desserts, extra sauces.

| Attribute | Key | Description |
|---|---|---|
| category_id | ID | Unique number of the category |
| category_name | | Name of the category |

#### `menu_item`

One thing a customer can order. Combos and buckets are menu items too. The price is the same in every branch.

| Attribute | Key | Description |
|---|---|---|
| menu_item_id | ID | Unique number of the menu item |
| menu_item_name | | Name, for example Classic Chicken Sandwich |
| item_price | | Current selling price in ILS, VAT included |
| menu_item_category | FK → menu_category | The category the item belongs to |
| item_existence | | Whether the item is on the menu now (yes / no). A removed item is kept, because old orders still refer to it. |

#### `menu_item_availability`

Says whether one menu item can be ordered at one branch right now. A branch can switch an item off (for example when it runs out) without affecting other branches.

| Attribute | Key | Description |
|---|---|---|
| menu_item_id | ID, FK → menu_item | The menu item |
| site_id | ID, FK → site | The branch |
| availability | | Available or unavailable at this branch |

#### `sales_order`

One customer order taken at a branch by a cashier.

| Attribute | Key | Description |
|---|---|---|
| order_id | ID | Unique number of the order |
| order_date_time | | Date and time the order was taken |
| payment_method | | cash or card (one method per order) |
| vat_rate | | VAT percentage that applied when the order was taken (16) |
| take_away_or_not | | yes = takeaway, no = dine-in |
| is_voided | | yes if the order was cancelled after entry. Voided orders stay in the database but do not count as sales. |
| site_id | FK → site | The branch where the order was taken |
| cashier_id | FK → employee | The employee who entered the order |
| order_total | D | Total of all lines |
| vat_amount | D | VAT part of the total |

#### `sales_order_line`

One menu item on one order, with how many were bought and the price charged.

| Attribute | Key | Description |
|---|---|---|
| order_id | ID, FK → sales_order | The order this line belongs to |
| menu_item_id | ID, FK → menu_item | The menu item sold |
| quantity | | How many were ordered |
| unit_price_charged | | Price of one item at the moment of sale. It is copied from the menu item so that old orders keep their original price. |

### Purchasing

#### `supplier`

A company that sells stock to Golden Coop.

| Attribute | Key | Description |
|---|---|---|
| supplier_id | ID | Unique number of the supplier |
| supplier_name | | Name of the supplier |
| contact_details | | Phone number or email |
| supplier_type | | local or foreign |
| payment_terms | | When invoices must be paid, for example 30 days from invoice, or paid in advance |

#### `stock_item`

Anything the company buys and keeps in stock: food ingredients, drinks, packaging, cleaning supplies.

| Attribute | Key | Description |
|---|---|---|
| stock_item_id | ID | Unique number of the stock item |
| item_name | | Name, for example bun or cooking oil |
| purchase_unit | | Unit it is bought in, for example pack, tin, carton |
| usable_unit | | Unit it is used and counted in, for example piece, litre, gram |
| conversion_factor | | How many usable units are in one purchase unit (a pack of 50 buns → 50) |

#### `supplier_item_price`

The price one supplier charges for one stock item, starting from a given date. Old prices are kept, so the history is never lost.

| Attribute | Key | Description |
|---|---|---|
| stock_item_id | ID, FK → stock_item | The stock item |
| supplier_id | ID, FK → supplier | The supplier |
| valid_date | ID | Date from which this price applies |
| unit_price | | Price of one purchase unit in ILS |

#### `purchase_order`

An order sent by head office to one supplier, to be delivered to one site.

| Attribute | Key | Description |
|---|---|---|
| purchase_order_id | ID | Unique number of the purchase order |
| order_date | | Date the order was sent |
| supplier_id | FK → supplier | The supplier the order was sent to |
| site_id | FK → site | The site where the goods must be delivered |
| delivery_status | D | Not delivered, partly delivered or fully delivered |

#### `purchase_order_line`

One stock item on one purchase order.

| Attribute | Key | Description |
|---|---|---|
| purchase_order_id | ID, FK → purchase_order | The purchase order this line belongs to |
| stock_item_id | ID, FK → stock_item | The stock item ordered |
| order_quantity | | Quantity ordered, in purchase units |
| unit_price | | Price of one purchase unit agreed at the time of ordering |

#### `goods_receipt`

One delivery that arrived for a purchase order, together with the supplier's invoice for that delivery. One purchase order can arrive in several deliveries.

| Attribute | Key | Description |
|---|---|---|
| goods_receipt_id | ID | Unique number of the receipt |
| purchase_order_id | FK → purchase_order | The purchase order this delivery belongs to |
| receipt_date | | Date the goods arrived |
| invoice_number | | Number printed on the supplier's invoice |
| invoice_amount | | Total of the supplier's invoice in ILS |
| due_date | | Date by which the invoice must be paid |
| balance_due | D | What is still unpaid on this invoice |

#### `goods_receipt_line`

One stock item in one delivery, with how much was accepted and how much was rejected.

| Attribute | Key | Description |
|---|---|---|
| goods_receipt_id | ID, FK → goods_receipt | The delivery this line belongs to |
| stock_item_id | ID, FK → stock_item | The stock item delivered |
| item_accepted | | Quantity accepted, in purchase units |
| item_rejected | | Quantity rejected (damaged or unusable), in purchase units |

#### `supplier_payment`

One payment made to a supplier for one invoice. An invoice can be paid in several parts.

| Attribute | Key | Description |
|---|---|---|
| payment_id | ID | Unique number of the payment |
| goods_receipt_id | FK → goods_receipt | The delivery (and its invoice) being paid |
| payment_amount | | Amount paid in ILS |
| payment_date | | Date of the payment |

### Stock

#### `recipe_line`

One ingredient of one menu item: how much of a stock item is used each time the menu item is sold. This is what connects sales to stock.

| Attribute | Key | Description |
|---|---|---|
| stock_item_id | ID, FK → stock_item | The stock item used |
| menu_item_id | ID, FK → menu_item | The menu item it is used in |
| quantity_used | | Quantity used for one menu item, in usable units (for example 20 grams of lettuce) |

#### `stock_level`

How much of one stock item is at one site right now.

| Attribute | Key | Description |
|---|---|---|
| site_id | ID, FK → site | The site holding the stock |
| stock_item_id | ID, FK → stock_item | The stock item |
| quantity_on_hand | | Quantity at this site now, in usable units |
| reorder_level | | When the quantity falls below this number, more should be ordered |

#### `stock_transfer`

A movement of stock from a warehouse to a branch. It starts as a request from the branch, is then sent by the warehouse, and is finally confirmed by the branch.

| Attribute | Key | Description |
|---|---|---|
| stock_transfer_id | ID | Unique number of the transfer |
| request_date | | Date the branch asked for the stock |
| sent_date | | Date the warehouse sent it (empty until sent) |
| transfer_status | | requested, sent or received |
| transfer_from | FK → site | The warehouse that sends the stock |
| transfer_to | FK → site | The branch that receives the stock |

#### `stock_transfer_line`

One stock item on one transfer, with the three quantities that can differ.

| Attribute | Key | Description |
|---|---|---|
| stock_transfer_id | ID, FK → stock_transfer | The transfer this line belongs to |
| stock_item_id | ID, FK → stock_item | The stock item moved |
| requested_quantity | | Quantity the branch asked for, in usable units |
| sent | | Quantity the warehouse sent |
| received | | Quantity the branch confirmed on arrival |

#### `stock_count`

One physical count of stock at one site on one date.

| Attribute | Key | Description |
|---|---|---|
| count_id | ID | Unique number of the count |
| count_date | | Date of the count |
| site_id | FK → site | The site that was counted |

#### `stock_count_line`

The result for one stock item in one count.

| Attribute | Key | Description |
|---|---|---|
| count_id | ID, FK → stock_count | The count this line belongs to |
| stock_item_id | ID, FK → stock_item | The stock item counted |
| counted_quantity | | Quantity really found, in usable units |
| count_expected_quantity | | Quantity the system expected at the moment of counting |
| difference | D | Expected minus counted. This is the waste. |

---

## 5. Relationships

All 32 relationships are **one-to-many (1:N)**. In every row below, the entity in the "One" column is the parent and the entity in the "Many" column holds the link attribute.

Read each row in both directions, as in the "Meaning" column.

### Organisation and staff

| # | One | Many | Link attribute | Meaning |
|---|---|---|---|---|
| 1 | city | site | site.city_id | A city has one or many sites. A site is in exactly one city. |
| 2 | site (warehouse) | site (branch) | site.supplied_by | A warehouse supplies many branches. A branch is supplied by exactly one warehouse. Warehouses and the head office have no supplier site. |
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

## 6. Many-to-many relationships

There is no direct many-to-many line in this design. Each many-to-many relationship is carried by an entity that sits in the middle and stores the data that belongs to the pair.

| Many-to-many between | Entity in the middle | Data stored on the pair |
|---|---|---|
| menu_item and site | `menu_item_availability` | availability |
| sales_order and menu_item | `sales_order_line` | quantity, unit_price_charged |
| supplier and stock_item | `supplier_item_price` | valid_date, unit_price |
| purchase_order and stock_item | `purchase_order_line` | order_quantity, unit_price |
| goods_receipt and stock_item | `goods_receipt_line` | item_accepted, item_rejected |
| menu_item and stock_item | `recipe_line` | quantity_used |
| site and stock_item | `stock_level` | quantity_on_hand, reorder_level |
| stock_transfer and stock_item | `stock_transfer_line` | requested_quantity, sent, received |
| stock_count and stock_item | `stock_count_line` | counted_quantity, count_expected_quantity |

---

## 7. Notes for drawing the ER diagram

- **14 entities have their own single identifier:** `city`, `site`, `job_title`, `employee`, `menu_category`, `menu_item`, `sales_order`, `supplier`, `stock_item`, `purchase_order`, `goods_receipt`, `supplier_payment`, `stock_transfer`, `stock_count`.
- **9 entities are identified by a combination** (the nine in section 6). They cannot exist without their parents.
- **`site` has a relationship with itself** (relationship 2, "supplied by"). Draw a line that leaves `site` and comes back to `site`.
- **`site` and `stock_transfer` are connected twice** (relationships 26 and 27). Draw two separate lines and label them "from" and "to".
- **`employee` and `sales_order`:** label the line "cashier" or "entered by".
- **Do not draw FK attributes as ordinary attributes** if you use the classic notation with ovals. They are shown by the relationship line.
- **Draw the five derived attributes** (marked D) with a dashed outline.

---

## 8. Design rules

1. **Two units for stock.** Supplier prices, purchase order lines and receipt lines use the **purchase unit** (packs, tins). Stock levels, transfers, counts and recipes use the **usable unit** (pieces, grams, litres). The conversion happens once, when goods are received: accepted quantity × conversion_factor.
2. **A transaction keeps its own copy of anything that can change later.** This is why the price is stored on the order line and the purchase order line, the VAT rate on the order, and the due date on the receipt. It is also why the order stores its branch (a cashier can be moved to another branch) and the transfer stores its source warehouse.
3. **One item appears once per document.** The same menu item cannot be on two lines of one order; entering it again increases the quantity. The same rule applies to purchase orders, receipts, transfers and counts.
4. **Nothing that has history is deleted.** A removed menu item gets item_existence = no. A cancelled order gets is_voided = yes.
5. **Combos and buckets are ordinary menu items** with their own recipe lines.
6. **Free extra sauce is a menu item with price 0** and its own recipe line, so the stock is still reduced.
7. **All purchasing is done by head office.** Chicken, vegetables and soft drinks are delivered straight to branches. Everything else is delivered to a warehouse.

---

## 9. Derived attributes

| Attribute | Entity | How it is calculated |
|---|---|---|
| order_total | sales_order | Sum of quantity × unit_price_charged over the order's lines |
| vat_amount | sales_order | order_total × vat_rate / (100 + vat_rate), because prices include VAT |
| delivery_status | purchase_order | Compare order_quantity on each line with the total item_accepted in all receipts of the order |
| balance_due | goods_receipt | invoice_amount minus the sum of its payments |
| difference | stock_count_line | count_expected_quantity minus counted_quantity |

---

## 10. Fixed value lists

| Attribute | Allowed values |
|---|---|
| city.area | North, Centre, South |
| site.site_type | branch, warehouse, head office |
| menu_item.item_existence | yes, no |
| menu_item_availability.availability | available, unavailable |
| sales_order.payment_method | cash, card |
| sales_order.take_away_or_not | yes (takeaway), no (dine-in) |
| sales_order.is_voided | yes, no |
| supplier.supplier_type | local, foreign |
| stock_transfer.transfer_status | requested, sent, received |

---

## 11. Out of scope

- Delivery orders, online orders and table reservations
- Customer accounts and loyalty cards
- Discounts and coupons
- Payroll and attendance (only the monthly salary is stored)
- Full accounting (only supplier invoices and payments)
- Expiry dates and batch tracking
- Vehicles and drivers
- More than one currency (everything is in ILS)
