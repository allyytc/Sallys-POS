# Sally's Coffee Shop POS

A full-stack point-of-sale web app for a small coffee shop. Baristas can manage customers, place and queue orders, track inventory, and run a loyalty program where customers earn and redeem points on drinks.

Built for Oregon State University's databases course, January–March 2026.

**Stack:** Node.js, Express, MySQL (mysql2), vanilla JavaScript, HTML/CSS

## Database design

Five tables, defined in `PL.sql`:

| Table | Purpose |
|---|---|
| `Customers` | Name, phone number, and loyalty point balance |
| `Orders` | Date, total cost, status (`queued` or `completed`), and points redeemed |
| `Products` | Drinks and food items, with an optional substitute product |
| `OrderHasProducts` | Many-to-many link between orders and products, with quantity |
| `Inventory` | Stock count and storage location for each product |

Foreign keys use `ON DELETE CASCADE` where a parent row owns its children: deleting a customer removes their orders, and deleting an order removes its line items.

### Stored procedures

| Procedure | What it does |
|---|---|
| `sp_reset_sally_coffee` | Drops and recreates all tables, then loads sample data |
| `sp_delete_customer` | Deletes a customer; cascades remove their orders and line items |
| `sp_place_order` | Inserts a queued order, records any points redeemed, and returns the new order ID |
| `sp_complete_order` | Marks an order completed and awards loyalty points (+1 per drink, minus points redeemed) |
| `sp_cancel_order` | Restores inventory for every item in the order, then deletes it |

Loyalty points are awarded only when an order is completed, not when it is placed, so a canceled order never earns points.

## API

All routes are in `app.js`.

| Method | Route | Description |
|---|---|---|
| POST | `/api/reset` | Reset the database to sample data |
| GET | `/api/customers` | List customers |
| POST | `/api/customers` | Create a customer |
| PUT | `/api/customers/:id` | Update a customer |
| DELETE | `/api/customers/:id` | Delete a customer |
| GET | `/api/customers/:id/orders` | List a customer's orders |
| GET | `/api/products` | List products |
| GET | `/api/orders` | List orders |
| GET | `/api/orders/:id/items` | List the items in an order |
| POST | `/api/orders` | Place an order |
| PUT | `/api/orders/:id/complete` | Complete an order and award points |
| PUT | `/api/orders/:id/items` | Update the items in an order |
| DELETE | `/api/orders/:id` | Cancel an order and restore inventory |
| GET | `/api/inventory` | List inventory |
| PUT | `/api/inventory/:id` | Update stock or location |

## Run locally

1. Install dependencies: `npm install`
2. Add your MySQL credentials to `db-connector.js`.
3. Run `PL.sql` against your database to create the tables and procedures.
4. Start the server: `npm run development`, then open `http://localhost:6777`.

## Team

- **Ally Chen** ([@allyytc](https://github.com/allyytc)): designed the database schema and wrote all stored procedures in `PL.sql`, including the order placement, completion, cancellation, and loyalty-points logic. Wrote the order endpoints in `app.js` that call these procedures (placing, completing, canceling, and editing orders), co-built the customer, product, and inventory endpoints, and built the initial UI for the Items, History, and Inventory pages.
- **Seth Gleason** ([@Gleason-o7](https://github.com/Gleason-o7)): built most of the front-end logic in `public/main.js` and styling in `public/style.css`, including the Customers tab, customer editing and deletion, order placement and new-customer creation from the Items tab, inventory stock and location editing, and the order queue. Co-built the customer, product, and inventory endpoints in `app.js`, prevented inventory from going below zero, and added phone number validation. See citations below.

## Citations

Gleason, S (January 2026) Activity 2 - Connect webapp to database https://canvas.oregonstate.edu/courses/2031764/assignments/10323319

Gleason, S (February 2026) bsg_sample_data_manipulation_queries.sql https://canvas.oregonstate.edu/courses/2031764/assignments/10323336?module_item_id=26243423

Chen, A (March 2026) sp_moviedb.sql https://canvas.oregonstate.edu/courses/2031764/assignments/10323341?module_item_id=26243449

Gleason, S (March 2026) "Regular Expression to reformat a US phone number in Javascript" https://stackoverflow.com/questions/8358084/regular-expression-to-reformat-a-us-phone-number-in-javascript

    - Many of my comments were AI generated

Gleason, S (January - March 2026) ./public/main.js claude.com

    - Many of my comments were AI generated

Gleason, S (Febrary - March 2026) ./public/app.js claude.com
