# Database Schema

Domain: **e-commerce store** — customers place orders containing product line items.

## Entity-relationship overview

```
customers (1) ──< orders (1) ──< order_items >── (1) products
```

## Tables

### `customers`

| Column | Type | Notes |
|--------|------|-------|
| id | INTEGER PK | Auto-increment |
| name | TEXT | Full name |
| email | TEXT | Unique-style emails in seed data |
| city | TEXT | |
| country | TEXT | Mostly India in sample data |
| created_at | TEXT | ISO date `YYYY-MM-DD` |

### `products`

| Column | Type | Notes |
|--------|------|-------|
| id | INTEGER PK | |
| name | TEXT | e.g. "Wireless Mouse" |
| category | TEXT | Electronics, Furniture, etc. |
| unit_price | REAL | INR-style prices |
| stock_qty | INTEGER | Inventory count |

### `orders`

| Column | Type | Notes |
|--------|------|-------|
| id | INTEGER PK | |
| customer_id | INTEGER FK | → `customers.id` |
| order_date | TEXT | ISO date |
| status | TEXT | `pending`, `paid`, `shipped`, `cancelled` |
| total_amount | REAL | Order header total |

### `order_items`

| Column | Type | Notes |
|--------|------|-------|
| id | INTEGER PK | |
| order_id | INTEGER FK | → `orders.id` |
| product_id | INTEGER FK | → `products.id` |
| quantity | INTEGER | Units ordered |
| line_total | REAL | `unit_price * quantity` (stored) |

## Design rationale

- **Normalized model**: Customers and products are referenced by FKs so spend and popularity queries join through `orders` / `order_items` without duplicating names.
- **Separate line items**: Supports “most ordered product” (aggregate `order_items`) vs “top customer by spend” (aggregate `orders` or line totals).
- **Status on orders**: Enables “unpaid” / pending filters without a separate payments table, keeping the schema small but realistic.
- **ISO date strings**: SQLite-friendly and easy for the LLM to filter with `date()` functions.

## Seed data volume

| Table | ~Rows |
|-------|-------|
| customers | 25 |
| products | 15 |
| orders | 80 |
| order_items | ~160 |
| **Total** | **~280** |

Seed script: `python -m knowledge_agent.db.seed`
