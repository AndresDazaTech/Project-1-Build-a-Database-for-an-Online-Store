# Project-1-Build-a-Database-for-an-Online-Store
### 1. Identify Keys and Attributes
Below are the tables with the requested notation: 
- `#` = Primary Key (Unique Identifier)
- `*` = Required Field
- `o` = Optional Field
- `(FK)` = Foreign Key (Link to another table)

**Customer**
*   # CustomerID
*   * Name
*   * Email
*   o Phone
*   o LoyaltyCardNumber

**Product**
*   # ProductID
*   * Name
*   o Description
*   * Price
*   * Stock

**Order**
*   # OrderID
*   * CustomerID (FK)
*   * OrderDate
*   * TotalAmount

**OrderDetails**
*   # OrderDetailsID (or composite #OrderID, #ProductID)
*   * OrderID (FK)
*   * ProductID (FK)
*   * Quantity
*   * UnitPrice (Price at the time of purchase)

**Payment**
*   # PaymentID
*   * OrderID (FK)
*   * PaymentType
*   * PaymentAmount

**Discount**
*   # DiscountID
*   * MinimumPurchase
*   * DiscountPercent

**OrderDiscounts** (Intermediate table to handle M:M between Orders and Discounts)
*   * OrderID (FK)
*   * DiscountID (FK)

**ItemPriceHistory**
*   # HistoryID
*   * ProductID (FK)
*   * OldPrice
*   * NewPrice
*   * ChangeDate

---

### 2. Entity-Relationship Diagram (ERD) Structure
*When drawing your diagram, use these connections:*

1.  **Customer to Order:** 1-to-Many. (Solid line: One Customer can have many Orders).
2.  **Order to OrderDetails:** 1-to-Many. (Solid line: One Order must have at least one OrderDetail).
3.  **Product to OrderDetails:** 1-to-Many. (Dashed line: A product can exist without being in an order yet).
4.  **Order to Payment:** 1-to-Many. (Dashed/Solid: One Order can have multiple payment attempts).
5.  **Order to Discount:** Many-to-Many. (Requires bridge table `OrderDiscounts`).
6.  **Product to ItemPriceHistory:** 1-to-Many. (Solid line: One product can have many history entries).

---

### 3. Normalization Explanation

**First Normal Form (1NF)**
*   **Definition:** Every field contains a single atomic value, and there are no repeating groups.
*   **Application:** We ensured that the `Customer` table doesn't have multiple phone numbers in one box. Instead of listing multiple products inside the `Order` table, we created the `OrderDetails` table so each row represents only one product per order.

**Second Normal Form (2NF)**
*   **Definition:** The table is in 1NF and all non-key attributes are fully dependent on the primary key.
*   **Application:** In the `OrderDetails` table, the `Quantity` and `UnitPrice` depend on the combination of the `OrderID` and `ProductID`. By moving these to a separate table, we avoid repeating product information (like Name or Description) every time someone buys it.

**Third Normal Form (3NF)**
*   **Definition:** The table is in 2NF and has no transitive dependencies (non-key fields shouldn't depend on other non-key fields).
*   **Application:** In the `Order` table, we only store the `CustomerID`. We do not store the Customer’s Name or Phone there. If a customer changes their phone number, we only have to update it in the `Customer` table, not in every single order they ever placed.

---

### 4. Data Integrity Rules

To ensure the database remains accurate, the following rules must be enforced:
1.  **Price & Amount:** `Product.Price`, `Order.TotalAmount`, and `Payment.PaymentAmount` must always be greater than zero.
2.  **Stock:** `Product.Stock` cannot be a negative number.
3.  **Quantity:** `OrderDetails.Quantity` must be at least 1.
4.  **Discounts:** `DiscountPercent` must be between 0.01 and 100.00.
5.  **Referential Integrity:** An Order cannot be created for a `CustomerID` that does not exist. Similarly, an `OrderDetails` record cannot be created for a `ProductID` that does not exist.
6.  **Dates:** `ChangeDate` in Price History and `OrderDate` cannot be in the future.

---

### 5. ItemPriceHistory Table Description

The **ItemPriceHistory** table is designed to maintain a "paper trail" of the store's pricing strategy. 
*   **How it works:** Whenever a manager updates the `Price` in the **Product** table, a new record is automatically generated in this table. 
*   **Tracking:** It captures the `OldPrice` (the price before the change), the `NewPrice` (the current price), and the `ChangeDate`.
*   **Purpose:** This allows BuyMore to run reports on inflation, seasonal price dips, or to resolve customer disputes regarding what a product cost on a specific date in the past. It links to the **Product** table via the `ProductID`.
