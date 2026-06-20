# 🖥️ PC Parts Ecommerce Database

A MySQL database for a computer parts ecommerce platform — includes `users` and `products` tables with realistic seed data (200 users, 500 products with real-world researched Indian pricing).

---

## 🚀 Getting Started — Setup MySQL on Your System

Follow these steps **in order** before running any SQL file in this repo.

### Step 1 — Install MySQL (Server + Workbench)

You need the actual MySQL database engine installed on your machine first.

1. Go to https://dev.mysql.com/downloads/installer/
2. Download **MySQL Installer for Windows**
3. Run the installer and choose **"Developer Default"** — this installs:
   - MySQL Server (the actual database engine)
   - MySQL Workbench (GUI tool to manage your database)
   - MySQL Shell
4. During setup, you'll be asked to set a **root password** — remember this, you'll need it every time you connect.
5. Finish the installation and let it configure the MySQL Server as a Windows service (so it runs automatically).
6. If prompted, also install the official **MySQL certificate / SSL files**, and choose the default authentication method (**Use Strong Password Encryption**) when asked.

✅ At this point, MySQL Server is running in the background and MySQL Workbench is installed as a desktop app.

---

### Step 2 — Install VS Code

1. Go to https://code.visualstudio.com/
2. Download and install **VS Code** for your OS.

---

### Step 3 — Install the MySQL Shell for VS Code Extension

1. Open **VS Code**
2. Click the **Extensions** icon on the left sidebar (or press `Ctrl+Shift+X`)
3. Search for: **MySQL Shell for VS Code**
4. Make sure you install the **genuine plugin published by Oracle Corporation** ✅ — verify the publisher name before installing, since similarly named extensions exist
5. Click **Install**

---

### Step 4 — Connect to Your Database

1. After installation, a new icon will appear in the **bottom-left sidebar** of VS Code — click it to open **MySQL Shell for VS Code**
2. Click **"Start Database"** (or the **+** icon under Database Connections)
3. Select **New Connection**
4. Enter your connection details:
   - **Host:** `127.0.0.1` (or `localhost`)
   - **Port:** `3306` (default)
   - **Username:** `root`
   - **Password:** the root password you set during MySQL installation
5. Click **OK**, then double-click the new connection to open a **DB Notebook**

✅ You're now ready to run SQL queries directly inside VS Code.

---

### ⚠️ If You Don't Have MySQL Workbench Installed

If you skipped Step 1 or only have VS Code without the actual MySQL engine, **the extension alone will not work** — VS Code's MySQL Shell extension is just a client; it needs a real MySQL Server to connect to.

1. Install **MySQL Workbench** + **MySQL Server** first from https://dev.mysql.com/downloads/installer/ (see Step 1 above)
2. Also install the official **MySQL certificate / SSL files** if prompted during setup
3. Once MySQL Server is running, go back to Step 3 and continue from there

---

## 👤 Users Table

### Schema

```sql
CREATE TABLE users (
  id                  CHAR(36)        NOT NULL DEFAULT (UUID()),
  email               VARCHAR(255)    NOT NULL,
  password_hash       VARCHAR(255)    NOT NULL,
  first_name          VARCHAR(100)    NOT NULL,
  last_name           VARCHAR(100)    NOT NULL,
  phone               VARCHAR(20)     DEFAULT NULL,
  role                ENUM('customer', 'admin', 'vendor') NOT NULL DEFAULT 'customer',
  is_active           BOOLEAN         NOT NULL DEFAULT TRUE,
  is_email_verified   BOOLEAN         NOT NULL DEFAULT FALSE,
  email_verified_at   TIMESTAMP       DEFAULT NULL,
  last_login_at       TIMESTAMP       DEFAULT NULL,
  created_at          TIMESTAMP       NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at          TIMESTAMP       NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  deleted_at          TIMESTAMP       DEFAULT NULL,

  PRIMARY KEY (id),
  UNIQUE KEY uq_users_email (email),
  INDEX idx_users_role (role),
  INDEX idx_users_is_active (is_active),
  INDEX idx_users_deleted_at (deleted_at)
) DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### Column Reference

| Column | Type | Description |
|--------|------|-------------|
| `id` | `CHAR(36)` | UUID v4 primary key, auto-generated |
| `email` | `VARCHAR(255)` | Unique user email |
| `password_hash` | `VARCHAR(255)` | Bcrypt/Argon2 hashed password |
| `first_name` / `last_name` | `VARCHAR` | User's name |
| `phone` | `VARCHAR(20)` | Optional phone number |
| `role` | `ENUM` | `customer`, `admin`, or `vendor` |
| `is_active` | `BOOLEAN` | Account active status |
| `is_email_verified` | `BOOLEAN` | Email verification status |
| `email_verified_at` | `TIMESTAMP` | When email was verified |
| `last_login_at` | `TIMESTAMP` | Last login timestamp |
| `created_at` / `updated_at` | `TIMESTAMP` | Record timestamps |
| `deleted_at` | `TIMESTAMP` | Soft delete marker (`NULL` = active) |

### Seed Data — 200 rows

| Property | Detail |
|----------|--------|
| Roles | ~85% customer, ~12% vendor, ~3% admin |
| Active users | 184 / 200 (92%) |
| Email verified | ~78% |
| Has phone | ~70% |
| Soft deleted | ~4% |

---

## 🛒 Products Table

### Schema

```sql
CREATE TABLE products (
  id                  CHAR(36)        NOT NULL DEFAULT (UUID()),

  -- Identity
  name                VARCHAR(255)    NOT NULL,
  slug                VARCHAR(280)    NOT NULL,
  sku                 VARCHAR(100)    NOT NULL,
  barcode             VARCHAR(100)    DEFAULT NULL,
  description         TEXT            DEFAULT NULL,
  short_description   VARCHAR(500)    DEFAULT NULL,
  brand               VARCHAR(100)    DEFAULT NULL,
  category            VARCHAR(100)    DEFAULT NULL,

  -- Pricing
  price               DECIMAL(10,2)   NOT NULL,
  discount_price      DECIMAL(10,2)   DEFAULT NULL,
  cost_price          DECIMAL(10,2)   DEFAULT NULL,
  tax_rate            DECIMAL(5,2)    NOT NULL DEFAULT 0.00,
  currency            VARCHAR(3)      NOT NULL DEFAULT 'USD',

  -- Inventory
  stock_quantity      INT UNSIGNED    NOT NULL DEFAULT 0,
  low_stock_threshold INT UNSIGNED    NOT NULL DEFAULT 5,

  -- Shipping / Physical attributes
  weight_kg           DECIMAL(8,3)    DEFAULT NULL,
  length_cm           DECIMAL(8,2)    DEFAULT NULL,
  width_cm            DECIMAL(8,2)    DEFAULT NULL,
  height_cm           DECIMAL(8,2)    DEFAULT NULL,

  -- Media
  thumbnail_url       VARCHAR(500)    DEFAULT NULL,
  image_urls          JSON            DEFAULT NULL,

  -- Status / Visibility
  status              ENUM('draft', 'active', 'inactive', 'out_of_stock', 'discontinued')
                                      NOT NULL DEFAULT 'draft',
  is_featured         BOOLEAN         NOT NULL DEFAULT FALSE,

  -- Ratings
  rating_average      DECIMAL(3,2)    NOT NULL DEFAULT 0.00,
  rating_count        INT UNSIGNED    NOT NULL DEFAULT 0,

  -- SEO
  meta_title          VARCHAR(255)    DEFAULT NULL,
  meta_description    VARCHAR(500)    DEFAULT NULL,

  -- Timestamps
  created_at          TIMESTAMP       NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at          TIMESTAMP       NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  deleted_at          TIMESTAMP       DEFAULT NULL,

  PRIMARY KEY (id),
  UNIQUE KEY uq_products_slug (slug),
  UNIQUE KEY uq_products_sku (sku),
  INDEX idx_products_brand (brand),
  INDEX idx_products_category (category),
  INDEX idx_products_status (status),
  INDEX idx_products_price (price),
  INDEX idx_products_deleted_at (deleted_at)
) DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### Column Reference

| Column | Type | Description |
|--------|------|-------------|
| `id` | `CHAR(36)` | UUID v4 primary key |
| `name` / `slug` | `VARCHAR` | Product name and URL-friendly slug |
| `sku` / `barcode` | `VARCHAR` | Internal SKU and scannable barcode |
| `description` / `short_description` | `TEXT` / `VARCHAR` | Full and short product description |
| `brand` / `category` | `VARCHAR` | Brand name and category (e.g. "NVIDIA", "Graphics Cards") |
| `price` | `DECIMAL(10,2)` | Selling price |
| `discount_price` | `DECIMAL(10,2)` | Optional discounted price |
| `cost_price` | `DECIMAL(10,2)` | Internal buying cost (never shown to customers) |
| `tax_rate` | `DECIMAL(5,2)` | Tax % applied at checkout (e.g. 18.00 for India GST) |
| `currency` | `VARCHAR(3)` | ISO currency code (e.g. INR, USD) |
| `stock_quantity` | `INT` | Units in stock |
| `low_stock_threshold` | `INT` | Triggers low-stock warning |
| `weight_kg`, `length_cm`, `width_cm`, `height_cm` | `DECIMAL` | Shipping dimensions |
| `thumbnail_url` / `image_urls` | `VARCHAR` / `JSON` | Product images |
| `status` | `ENUM` | `draft`, `active`, `inactive`, `out_of_stock`, `discontinued` |
| `is_featured` | `BOOLEAN` | Shown in featured sections |
| `rating_average` / `rating_count` | `DECIMAL` / `INT` | Denormalized review stats |
| `meta_title` / `meta_description` | `VARCHAR` | SEO fields |
| `deleted_at` | `TIMESTAMP` | Soft delete marker |

### Seed Data — 500 rows

Real-world PC component data researched from Amazon.in, EliteHubs, PrimeABGB, VisionIT, Vedant Computers, and pricehistory.app (Indian pricing, mid-2026).

| Category | Count |
|----------|-------|
| Graphics Cards | 120 |
| Processors | 95 |
| Storage | 73 |
| Motherboards | 44 |
| Power Supplies | 43 |
| Memory (RAM) | 49 |
| Cases | 39 |
| Cooling | 37 |

> **Note:** 113 entries are real, verified products with researched market prices (e.g. RTX 4090 Founders Edition ₹1,77,000, Core i9-14900K ₹57,675). The remaining 387 are realistic edition variants of those same product lines with proportionally adjusted pricing — included to reach 500 rows while keeping prices grounded in real market data. Treat this as high-quality seed/demo data, not a live price feed.

---

## 🚀 Setup — Running the SQL Files

Run files in this exact order:

```bash
mysql -u root -p your_database < schema/users.sql
mysql -u root -p your_database < schema/products.sql
mysql -u root -p your_database < seeds/users_insert_200.sql
mysql -u root -p your_database < seeds/products_insert_500.sql
```

Or, using the **MySQL Shell for VS Code DB Notebook**:
1. Open each `.sql` file in VS Code
2. Click the ▶️ **Run** icon at the top of the notebook
3. Run them top to bottom, in the order listed above

---

## 🔍 Useful Queries

```sql
-- Total users and products
SELECT COUNT(*) AS total_users FROM users;
SELECT COUNT(*) AS total_products FROM products;

-- Active users
SELECT * FROM users WHERE is_active = 1;

-- Active, in-stock products
SELECT * FROM products WHERE status = 'active' AND stock_quantity > 0;

-- Products by category
SELECT category, COUNT(*) AS total FROM products GROUP BY category;

-- All 5 core aggregate functions on products
SELECT 
  COUNT(*)             AS total_products,
  SUM(stock_quantity)   AS total_stock,
  ROUND(AVG(price), 2) AS average_price,
  MAX(price)            AS highest_price,
  MIN(price)            AS lowest_price
FROM products;
```

---

## 🪟 Storefront View

A ready-to-use view that exposes only customer-safe fields (hides `cost_price`, adds computed `final_price`, `discount_percent`, and `stock_status`):

```sql
CREATE VIEW vw_storefront_products AS
SELECT
  id, name, slug, sku, brand, category, short_description,
  price, discount_price,
  COALESCE(discount_price, price) AS final_price,
  CASE WHEN discount_price IS NOT NULL
       THEN ROUND(((price - discount_price) / price) * 100, 0)
       ELSE 0 END AS discount_percent,
  currency,
  CASE
    WHEN stock_quantity = 0 THEN 'Out of Stock'
    WHEN stock_quantity <= low_stock_threshold THEN 'Low Stock'
    ELSE 'In Stock'
  END AS stock_status,
  stock_quantity > 0 AS is_purchasable,
  thumbnail_url, image_urls, is_featured,
  rating_average, rating_count,
  meta_title, meta_description, created_at
FROM products
WHERE status = 'active' AND deleted_at IS NULL;
```

```sql
-- Usage
SELECT * FROM vw_storefront_products;
SELECT * FROM vw_storefront_products WHERE is_featured = 1;
SELECT * FROM vw_storefront_products WHERE discount_price IS NOT NULL;
```

---

## ⚙️ Requirements

- MySQL **8.0.13+** (required for `DEFAULT (UUID())`)
- utf8mb4 charset support
- VS Code + MySQL Shell for VS Code extension (Oracle Corporation), or MySQL Workbench

---

## 📌 Notes

- **Passwords** are stored as bcrypt hashes only — never plain text.
- **Soft deletes** — always filter with `WHERE deleted_at IS NULL` for active records.
- **UUID primary keys** are auto-generated by MySQL — don't pass `id` manually on insert.
- **DDL statements** (`CREATE`, `DROP`, `ALTER`) auto-commit in MySQL and **cannot be rolled back** inside a transaction — only `INSERT`/`UPDATE`/`DELETE` are transactional.

---
