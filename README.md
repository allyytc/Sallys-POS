# Sally's Coffee Shop

A point-of-sale console for a small coffee shop: take orders, run a barista queue, track stock, and keep customer loyalty accounts in MySQL. Built as an Oregon State University CS340 database course project.

It runs locally against MySQL. This repository does not include a deployment.

## What it does

The browser UI has four screens.

- **Create Order.** Drinks and food come from `Products`. Names that start with "oat milk" stay off the menu grid and are offered with a substitute button, using `substituteProductID`. An item can be added only while `Inventory` still has stock. Customer names autocomplete from `Customers`. An unknown name asks for a phone number in the form `###-###-####` and creates the account. With 10 or more points and a drink on the ticket, the cheapest drink can be marked free. The point balance changes when that order is completed, not when it is placed.
- **Order History.** Placed orders enter an active queue (`status = 'queued'`) and stock is reduced immediately. A queued ticket can be edited (change quantities, add a product, or remove a line by setting quantity to 0), cancelled, or completed. Cancel restores stock and deletes the order. Complete sets `status = 'completed'` and updates points: one point per drink quantity, minus the points redeemed on that order. Completed orders are listed with their line items.
- **Inventory.** Each product shows stock and a storage location, both editable. A status list colors every item by quantity: red at 0, yellow under 100, and green otherwise.
- **Customers.** Search by name and phone, add an account, edit name, phone, and points, or delete an account. Deleting a customer removes that customer's orders through the foreign keys.

**Reset** asks for confirmation, then calls `sp_reset_sally_coffee()`, which drops the tables, recreates them, and reloads the sample customers, menu, orders, and stock.

## Tech stack

- Frontend: HTML, CSS, and vanilla JavaScript in `public/`
- Server: Node.js and Express 4 (`app.js`), listening on port 6777
- Database: MySQL through a `mysql2` connection pool (`db-connector.js`)
- SQL: tables, seed data, and stored procedures in `PL.sql`
- Local scripts: `nodemon` (`npm run development`) and `forever` (`npm run production`, `npm run stop_production`)

## Architecture

`public/main.js` loads and updates data with `fetch`. `app.js` runs parameterized SQL and the stored procedures through the pool in `db-connector.js`.

```
public/index.html + main.js
        |  fetch
        v
app.js (Express, port 6777)
        |  mysql2 pool
        v
MySQL   Customers, Products, Orders, OrderHasProducts, Inventory
        stored procedures in PL.sql
```

Placing an order is split between SQL and the route. `sp_place_order` inserts the queued `Orders` row, stores `redeemedPoints`, and returns the new id. The route then writes `OrderHasProducts` and decrements `Inventory`. `sp_complete_order` sets the status and awards points. `sp_cancel_order` adds stock back and deletes the order. `sp_delete_customer` deletes the customer row. `Orders.customerID` and `OrderHasProducts.orderID` are `ON DELETE CASCADE`.

## Database schema

Tables are created and seeded inside `sp_reset_sally_coffee` in `PL.sql`.

### Customers

| Column | Type | Notes |
| --- | --- | --- |
| customerID | int | Primary key, auto-increment |
| customerName | varchar(45) | Required |
| customerNumber | varchar(15) | Phone, optional |
| numPoints | int | Required, default 0 |

### Orders

| Column | Type | Notes |
| --- | --- | --- |
| orderID | int | Primary key, auto-increment |
| orderDate | datetime | Set with `NOW()` when an order is placed |
| orderCost | decimal(10,2) | Required |
| status | varchar(20) | Required, default `completed`. The app uses `queued` and `completed` |
| redeemedPoints | int | Required, default 0. Points spent on this order, applied on completion |
| customerID | int | Required. Foreign key to `Customers`, `ON DELETE CASCADE` |

### Products

| Column | Type | Notes |
| --- | --- | --- |
| productID | int | Primary key, auto-increment |
| productName | varchar(45) | Required |
| productType | varchar(45) | Required. Seed rows use `drink` and `food` |
| productCost | decimal(10,2) | |
| substituteProductID | int | Nullable foreign key to `Products`. Links a drink to its substitute |

### OrderHasProducts

| Column | Type | Notes |
| --- | --- | --- |
| orderID | int | Primary key, foreign key to `Orders`, `ON DELETE CASCADE` |
| productID | int | Primary key, foreign key to `Products` |
| quantity | int | Required |

### Inventory

| Column | Type | Notes |
| --- | --- | --- |
| inventoryID | int | Primary key, auto-increment |
| productID | int | Unique foreign key to `Products` |
| productStock | int | Required |
| productLocation | varchar(45) | Storage location |

### Stored procedures

| Procedure | What it does |
| --- | --- |
| `sp_reset_sally_coffee` | Drops the five tables, recreates them, and inserts the sample data |
| `sp_delete_customer(customerID)` | Deletes that customer |
| `sp_place_order(customerID, totalCost, redeemedPoints, OUT orderID)` | Inserts a `queued` order and returns the new `orderID` |
| `sp_complete_order(orderID)` | Sets status to `completed` and adds drink quantity minus `redeemedPoints` to the customer's points |
| `sp_cancel_order(orderID)` | Adds each line's quantity back to `Inventory`, then deletes the order |

## Setup

Node.js, npm, and a MySQL server are required. There is no `.env` file and no environment variables. The connection is the four placeholder strings in `db-connector.js`: `HOSTNAME`, `USER`, `PASSWORD`, and `DATABASAE`.

1. Install dependencies:

   ```bash
   npm install
   ```

2. Create a MySQL database. In `db-connector.js`, replace `HOSTNAME`, `USER`, `PASSWORD`, and `DATABASAE` with the host, user, password, and database name. The database-name placeholder is spelled `DATABASAE` in the source.

3. Execute `PL.sql` in that database. It creates the stored procedures. It does not create the database itself.

4. Build the tables and sample data:

   ```sql
   CALL sp_reset_sally_coffee();
   ```

   After the server is running, the Reset button calls the same procedure.

5. Start the app:

   ```bash
   npm run development
   ```

   Open [http://localhost:6777](http://localhost:6777).

`npm run production` starts `app.js` under `forever` on this machine. `npm run stop_production` stops that process.

## Team

Roles are drafts inferred from git history. They need confirmation.

| Name | Role (draft) |
| --- | --- |
| Ally Chen | frontend, Express server, MySQL stored procedures |
| Gleason-o7 | frontend, customer and inventory flows, order queue, API integration |

## Course citations

These lines are the citations previously kept in this file.

- Gleason, S (January 2026) Activity 2 - Connect webapp to database https://canvas.oregonstate.edu/courses/2031764/assignments/10323319
- Gleason, S (February 2026) bsg_sample_data_manipulation_queries.sql https://canvas.oregonstate.edu/courses/2031764/assignments/10323336?module_item_id=26243423
- Chen, A (March 2026) sp_moviedb.sql https://canvas.oregonstate.edu/courses/2031764/assignments/10323341?module_item_id=26243449
- Gleason, S (March 2026) "Regular Expression to reformat a US phone number in Javascript" https://stackoverflow.com/questions/8358084/regular-expression-to-reformat-a-us-phone-number-in-javascript
- Gleason, S (January - March 2026) ./public/main.js claude.com
  - Many of my comments were AI generated
- Gleason, S (Febrary - March 2026) ./public/app.js claude.com
  - Many of my comments were AI generated

The server file cited as `./public/app.js` is `app.js` in the project root. Matching citation comments are in `db-connector.js`, `app.js`, `public/main.js`, and the header of `PL.sql`.
