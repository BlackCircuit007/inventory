# Stockroom Nigeria

Stockroom is an inventory, point-of-sale, team, and reporting application for a Nigerian retail business. It is backed by a real SQLite database and serves its browser interface from a small Node.js server.

The application supports:

- Administrator and sales associate accounts
- Product catalog and stock quantities
- Low-stock alerts based on reorder points
- Point-of-sale checkout
- Cash, card, and transfer payments
- Nigerian naira formatting
- Order history and CSV export
- Team account creation by an administrator
- Administrator-only financial and management views
- Real-time administrator alerts for every completed sale
- Daily, monthly, and yearly audit metrics from SQLite
- Administrator reset control for clearing business data
- Isolated developer security console for authentication and session telemetry
- Persistent SQLite storage for users, products, orders, and order items

## 1. Requirements

### Runtime

- Windows, macOS, or Linux
- Node.js `22.5.0` or newer
- npm, which is included with Node.js

This project intentionally has no third-party npm dependencies. It uses Node's built-in `node:sqlite` module.

Check your installed versions:

```powershell
node --version
npm --version
```

The Node version must be at least `22.5.0`.

## 2. Project Files

| File | Purpose |
| --- | --- |
| `index.html` | The complete browser interface and page structure |
| `styles.css` | Layout, responsive design, orange-and-white theme, tables, forms, and modals |
| `app.js` | Browser behavior, navigation, POS cart, rendering, API calls, and admin alerts |
| `server.js` | Node HTTP server, authentication, SQLite access, API routes, sessions, and real-time events |
| `package.json` | Project metadata and the `npm start` command |
| `stockroom.db` | SQLite database created automatically when the server first starts |
| `README.md` | This manual |

`stockroom.db` is the source of truth for business data. The browser does not use localStorage for products, users, orders, passwords, or stock quantities.

## 2.1 Deploy to Render

The repository includes `render.yaml`, which configures a Node web service and a 1 GB persistent disk for SQLite. The persistent disk is important: without it, the database would be lost when Render restarts or redeploys the service.

1. Push this project to GitHub, including `server.js`, `app.js`, `index.html`, `styles.css`, `package.json`, and `render.yaml`.
2. Sign in to [Render](https://render.com).
3. Select `New` -> `Blueprint`.
4. Connect the GitHub repository containing this project.
5. Select the branch to deploy, normally `main`.
6. Review the service name `stockroom-nigeria` and select `Apply`.
7. Wait for the first build and deployment to finish.
8. Open the Render URL shown on the service page.

The blueprint uses:

```text
Build command: npm install
Start command: npm start
Runtime: Node
Database file: /opt/render/project/src/data/stockroom.db
```

The Render Starter plan is required for the persistent disk in this blueprint. A free web service can run the app temporarily, but its SQLite data is not durable without persistent storage.

After deployment, verify the service by opening the Render URL and signing in with:

```text
Username: admin
Password: admin123
```

Change the administrator password before using the deployed service for real sales. Also confirm that `https://your-render-url.onrender.com/api/state` returns an authentication error when opened while signed out; that confirms the API is not publicly exposing business data.

### Render environment variables

The blueprint sets these automatically:

| Variable | Value | Purpose |
| --- | --- | --- |
| `NODE_VERSION` | `22.18.0` | Provides the built-in `node:sqlite` module |
| `DB_PATH` | `/opt/render/project/src/data/stockroom.db` | Stores SQLite on the persistent disk |

### Updating the deployed app

With `autoDeploy: true`, push changes to the selected GitHub branch and Render will build and deploy them automatically. Do not delete the persistent disk when deleting or recreating the service unless you have a verified backup of `stockroom.db`.

### Backups on Render

Before a reset or major change, download or copy the database using a maintenance process. The database is located at `/opt/render/project/src/data/stockroom.db` inside the Render service. The in-app `Reset business data` button intentionally deletes products, orders, and order items, so use it only when you really want a clean business setup.

## 3. Start the Application

Open PowerShell in the project directory:

```powershell
cd C:\Users\user\Desktop\tests
npm start
```

The server starts at:

```text
http://localhost:3000
```

Open that address in a browser. Do not open `index.html` directly with a `file:///` URL when using the database version. The Node server is required because the browser needs the API and SQLite-backed session.

Stop the server with `Ctrl+C` in the terminal running it.

## 4. Default Accounts

The first server start creates only this administrator account if the `users` table is empty:

| Role | Username | Password |
| --- | --- | --- |
| Administrator | `admin` | `admin123` |

This is the initial administrator credential. Change it before using the application for real business operations. Worker accounts are created by the administrator from the Team page; no worker demo accounts are created automatically.

The application authenticates with usernames, not email addresses. Worker credentials are created directly by an administrator from the Team page.

## 5. User Roles and Permissions

### Super administrator / developer security console

The super administrator is a separate platform-security identity. It is not a company administrator and cannot access company state, inventory, POS, orders, revenue, reports, products, workers, or reset controls. A successful login redirects to the separate `/security.html` console, which has its own dark monitoring interface.

The initial security account is created from environment variables:

```text
SUPER_ADMIN_USERNAME=""""
SUPER_ADMIN_PASSWORD=...///''''
```

If the variables are not set, those development defaults are used. Set both variables in Render before production deployment. The security console reports:

- Failed login count
- Failed attempts grouped by attempted username
- Successful login count
- Active authenticated company sessions
- Source IP and user agent for security events
- Login, logout, and inactive/invalid-account events
- Persistent event count stored in SQLite

The console intentionally does not display business information. Security data is served only through `/api/security/state` and `/api/security/events`, which require the `Super administrator` role.

To enter the developer console, use the normal login page with the configured super administrator credentials. The browser redirects that account to `/security.html`. The company administrator account continues to open the orange company workspace instead.


### Administrator

An administrator can:

- View Overview
- View revenue, order totals, and average order value
- View the sales chart and live activity feed
- View the cash drawer summary
- View Inventory
- Add products
- Adjust stock quantities
- Use Point of sale
- View Orders
- Download the orders CSV
- View Team
- Create sales associate accounts
- View Reports
- Receive real-time alerts for all completed sales

### Sales associate

A sales associate can:

- View the stock count and low-stock list
- Search Inventory
- Use Point of sale
- Add products to a cart
- Complete cash, card, or transfer sales
- See their own active workspace identity

A sales associate cannot see:

- Orders navigation
- Reports navigation
- Team management
- Revenue totals
- Average order value
- Cash drawer details
- Administrator-only inventory actions such as adding products or adjusting stock
- Administrator real-time sale alerts

The server also enforces the administrator restriction. Hiding a button in the browser is not the only protection: administrator API routes return HTTP `403` to workers.

## 6. Administrator Workflow

### 6.1 Sign in

1. Open `http://localhost:3000`.
2. Enter the administrator username and password.
3. Select `Enter workspace`.
4. The dashboard loads data from SQLite through `/api/state`.

### 6.2 Add a product

1. Open `Inventory`.
2. Select `+ Add product`.
3. Enter:
   - Product name
   - SKU
   - Category
   - Price in naira
   - Opening stock quantity
   - Reorder point
4. Select `Add product`.

The product is written to the `products` SQLite table. Its price is stored as an integer number of kobo, not a floating-point currency value.

For example:

```text
Input:       ₦4,250.50
Stored:      425050 kobo
Displayed:   ₦4,250.50
```

### 6.3 Adjust stock

1. Open `Inventory`.
2. Find the product.
3. Select its action button.
4. Enter the new stock quantity.
5. Select `Update stock`.

The browser sends a `PATCH /api/products/:id` request. The server updates SQLite and the dashboard refreshes from the database.

### 6.4 Create a worker account

1. Open `Team`.
2. Select `+ Invite teammate`.
3. Enter the worker's name.
4. Choose the username.
5. Set the password.
6. Select `Create account`.

No email is sent and no random password is generated. The server hashes the password and stores it in the `users` table. The worker can use the chosen username and password immediately.

Usernames must be unique. A duplicate username is rejected.

### 6.5 Activate or deactivate a worker

1. Open `Team` as an administrator.
2. Find the worker account.
3. Select `Deactivate worker` to block that account from signing in.
4. Select `Activate worker` to restore access later.

Deactivation changes the account status to `Inactive` in SQLite. The server checks this status during login, so an inactive worker cannot create a new session even if they still know the old password. The administrator account cannot be deactivated.

### 6.6 Review orders

1. Open `Orders`.
2. Review order number, date, customer, staff member, payment method, total, and completion status.
3. Select `Download CSV` to download a report of the current orders.

### 6.7 Review reports

The Reports page shows real values calculated from the database:

- Today's sales and completed orders
- Current month's sales and completed orders
- Current year's sales and completed orders
- Average order value
- Total units sold
- Current inventory value
- Top products by recorded units sold
- Payment mix from completed orders

All report data is derived from the SQLite-backed orders, order items, and products. Tax is not included in the checkout or reports.

### 6.8 Reset business data

Administrators can open `Reports` and select `Reset business data`. After confirmation, the server deletes all products, stock quantities, orders, and order items. The administrator account and worker accounts remain available. This is intended for starting a new business setup or removing test transactions.

The reset action is protected by the administrator role and runs inside a SQLite transaction.

## 7. Worker Point-of-Sale Workflow

1. Sign in with a sales associate account.
2. Open `Point of sale`.
3. Search for a product or choose a category.
4. Select a product to add it to the cart.
5. Adjust quantity with the plus and minus buttons.
6. Select `Charge`.
7. Choose `Card`, `Cash`, or `Transfer`.
8. Optionally enter a customer name.
9. Select `Complete sale`.

When the sale succeeds, the server performs one SQLite transaction that:

1. Checks that every product has enough stock.
2. Creates an order record.
3. Creates order item records.
4. Reduces product stock.
5. Commits all changes together.
6. Broadcasts the completed sale to connected administrators.

If there is not enough stock, the transaction is rolled back and no partial order is created.

## 8. Currency and Tax

The application is configured for Nigerian naira:

```text
Currency: NGN
Display symbol: ₦
Locale: en-NG
```

The application currently does not add tax. Checkout totals are the exact sum of the selected product prices multiplied by their quantities. The server recalculates this total from SQLite product prices before saving the order.

## 9. Real-Time Administrator Alerts

Administrators receive live alerts through Server-Sent Events, also called SSE.

### How it works

1. An administrator signs in.
2. The browser opens `GET /api/events`.
3. The server keeps the connection open.
4. A worker or administrator completes a sale.
5. SQLite commits the order.
6. The server broadcasts the order to every connected administrator stream.
7. Each administrator dashboard displays a toast such as:

```text
New sale ORD-1050: ₦1,948.50 via Cash.
```

8. The dashboard requests fresh state so revenue, stock, order history, and activity update immediately.

Workers do not connect to the administrator event stream. The server checks the session role and returns `403` to non-administrators.

Alerts work across multiple browser tabs and users while they are connected to the same running server. If an administrator is offline, the completed order remains available in the database and appears when they next load the dashboard, but a missed toast is not replayed as a notification history.

## 10. Database Design

The database file is `stockroom.db`.

### `users`

Stores login and account information.

Important columns:

- `id`: Primary key
- `name`: Display name
- `username`: Unique login name
- `email`: Internal placeholder/contact field
- `password_hash`: Scrypt-derived password hash
- `role`: `Administrator` or `Sales associate`
- `initials`: Avatar initials
- `status`: `Active` or `Inactive`
- `color`: Avatar color

Passwords are never returned to the browser by the API.

### `products`

Stores catalog and stock data.

Important columns:

- `id`: Primary key
- `name`: Product name
- `sku`: Unique stock keeping unit
- `category`: Product category
- `price_kobo`: Integer price in kobo
- `stock`: Current available quantity
- `reorder`: Low-stock threshold
- `icon`: Short visual label used by the interface

### `orders`

Stores one record per completed sale.

Important columns:

- `id`: Internal primary key
- `order_number`: Public number such as `ORD-1050`
- `created_at`: ISO timestamp
- `customer`: Customer label
- `staff_user_id`: User who completed the sale
- `payment`: `Cash`, `Card`, or `Transfer`
- `total_kobo`: Final order total in kobo

### `order_items`

Stores the products included in each order.

Important columns:

- `id`: Primary key
- `order_id`: Related order
- `product_id`: Related product
- `quantity`: Quantity sold
- `unit_price_kobo`: Price at the time of sale

The unit price is copied into the order item so historical sales remain accurate if a product price changes later.

## 11. API Reference

All API calls use JSON. Authenticated requests use the HTTP-only `stockroom_session` cookie created during login.

### `POST /api/login`

Request:

```json
{
  "username": "admin",
  "password": "admin123"
}
```

Returns the authenticated application state and creates a session cookie.

### `POST /api/logout`

Ends the current server session and clears the session cookie.

### `GET /api/state`

Returns:

- Products
- Orders
- Sanitized users
- Current user

Requires authentication.

### `GET /api/events`

Opens the administrator-only SSE stream. Requires an administrator session.

### `POST /api/products`

Creates a product. Administrator only.

Example:

```json
{
  "name": "Oak Serving Board",
  "sku": "HP-072",
  "category": "Home",
  "price": 24000,
  "stock": 10,
  "reorder": 5,
  "icon": "OS"
}
```

The `price` value is supplied in naira and converted to kobo by the server.

### `PATCH /api/products/:id`

Updates product stock. Administrator only.

Example:

```json
{
  "stock": 18
}
```

### `POST /api/users`

Creates a sales associate account. Administrator only.

Example:

```json
{
  "name": "Jordan Blake",
  "username": "jordan",
  "password": "jordan123"
}
```

The server stores a password hash, not the plain password.

### `POST /api/orders`

Creates a completed sale and updates stock atomically.

Example:

```json
{
  "customer": "Walk-in customer",
  "payment": "Transfer",
  "total": 4113.5,
  "items": [
    {
      "productId": 1,
      "quantity": 1
    }
  ]
}
```

Requires any authenticated user. The server checks stock, writes the order and items, reduces stock, commits the transaction, and broadcasts the alert.

## 12. Backup and Restore

### Backup

Stop the server first, then copy the database file:

```powershell
Copy-Item .\stockroom.db .\backups\stockroom-$(Get-Date -Format yyyyMMdd-HHmmss).db
```

Create the backup directory once if needed:

```powershell
New-Item -ItemType Directory -Force .\backups
```

A SQLite backup should be copied while the application is stopped to avoid capturing a partially written file.

### Restore

1. Stop the server with `Ctrl+C`.
2. Keep the current database as a safety copy:

```powershell
Move-Item .\stockroom.db .\stockroom-before-restore.db
```

3. Copy the backup into place:

```powershell
Copy-Item .\backups\stockroom-YYYYMMDD-HHMMSS.db .\stockroom.db
```

4. Start the server again:

```powershell
npm start
```

## 13. Reset Business Data

To completely recreate the database:

1. Stop the server.
2. Delete `stockroom.db`.
3. Start the server again.

```powershell
Remove-Item .\stockroom.db
npm start
```

The server will recreate the schema and the single administrator account. It will not recreate products, workers, orders, or test transactions. Add your real products and worker accounts from the admin interface.

This permanently deletes all worker accounts, products, orders, and stock changes. The administrator account remains. Make a backup first if the data matters.

## 14. Troubleshooting

### The page shows an authentication error

Confirm that the server is running:

```powershell
npm start
```

Then open `http://localhost:3000`, not the HTML file directly.

### The database does not start

Check Node:

```powershell
node --version
```

Node must be `22.5.0` or newer because the project uses `node:sqlite`.

### Port 3000 is already in use

Start on another port:

```powershell
$env:PORT=3001
npm start
```

Then open `http://localhost:3001`.

### An administrator is not receiving live alerts

Check:

- The administrator is logged in.
- The browser is using the Node server URL.
- The server process is still running.
- The browser is not opened from `file:///`.
- The administrator tab has not been suspended by the browser.

Reloading the dashboard reconnects the SSE stream.

### A worker sees administrator content

Log out and back in after changing permissions. The server enforces administrator API permissions independently of the browser UI. A worker request to administrator-only endpoints should return HTTP `403`.

### The database has old records

The database persists by design. Existing orders and stock changes remain after server restarts. Use the reset process above when starting a new business setup or removing test data.

## 15. Production Hardening Checklist

This project is a functional local business application. Before exposing it publicly, address these items:

- Change the default administrator password.
- Use a per-user random password salt instead of the shared local development salt.
- Add password change and password reset workflows.
- Use HTTPS so session cookies and credentials are encrypted in transit.
- Add secure cookie flags such as `Secure` in HTTPS deployment.
- Add CSRF protection for state-changing requests.
- Add rate limiting for login attempts.
- Move sessions from server memory to a persistent session store if multiple server processes will run.
- Validate and recalculate order totals on the server.
- Add server-side audit logs for stock changes, user creation, refunds, and price changes.
- Add explicit refund and cancellation workflows.
- Add database migrations instead of relying only on startup table creation.
- Schedule automated database backups.
- Restrict access to the Node server with a firewall or reverse proxy.
- Add a proper production process manager and monitoring.

## 16. Development Checks

Syntax checks:

```powershell
node --check .\server.js
node --check .\app.js
```

Diagnostics can also be checked in VS Code for:

- `index.html`
- `styles.css`
- `app.js`
- `server.js`

A basic database inspection command:

```powershell
node -e "const {DatabaseSync}=require('node:sqlite'); const db=new DatabaseSync('stockroom.db'); console.log(db.prepare('select name from sqlite_master where type=\"table\"').all())"
```

Inspect current orders:

```powershell
node -e "const {DatabaseSync}=require('node:sqlite'); const db=new DatabaseSync('stockroom.db'); console.log(db.prepare('select order_number, total_kobo, payment from orders order by id desc').all())"
```

Inspect a product's stock:

```powershell
node -e "const {DatabaseSync}=require('node:sqlite'); const db=new DatabaseSync('stockroom.db'); console.log(db.prepare('select sku, stock from products').all())"
```

## 17. Current Limitations

The following behaviors are intentionally simple in this first database-backed version:

- The dashboard's chart currently uses a fixed visual series rather than a date-grouped sales query.
- The report period selectors are visual controls and do not yet change the query range.
- The opening cash float starts at `₦0.00` and can be extended with a register-opening workflow later.
- Missed real-time alerts are not stored in a notification inbox; the order itself remains in SQLite.
- The server stores active sessions in memory, so restarting the server logs everyone out.
- The order total should be recalculated on the server before public deployment.

These limitations do not affect the core SQLite persistence, POS transaction, role enforcement, or live administrator alert behavior.
#   i n v e n t o r y 
 
 