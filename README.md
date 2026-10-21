# GeStock

**GeStock** is a stock and inventory management **REST API** built with **Laravel 11**. It manages products, categories, suppliers, customers, orders, and inventories, and secures everything with **Sanctum** token authentication and a **role & permission** system.

A UML class diagram of the domain is included in [`UML class Diagram.pdf`](UML%20class%20Diagram.pdf).

## Features

- **Authentication** with Laravel Sanctum (register, login, logout)
- **Role-based access control**: roles, permissions, and role ↔ permission assignment, enforced by custom `role` and `permission` middleware
  - The **first registered user becomes `Admin`**; later users get the `User` role
- **Inventory domain** with full CRUD:
  - Products (linked to a category and a supplier)
  - Categories, Suppliers, Customers
  - Orders (status: `processing` / `completed` / `cancelled`) with order line items (quantity, price)
  - Inventories (capacity, current stock, location) with per-product quantities
- **Factories & seeders** for quick demo data
- **Auto-generated API docs** via [laravel-request-docs](https://github.com/rakutentech/laravel-request-docs)

## Tech Stack

- PHP 8.2+, Laravel 11
- Laravel Sanctum
- MySQL (or SQLite)
- rakutentech/laravel-request-docs

## Data Model

```
Role ──< User ──< Order >── Customer
 │                 │
 └─< PermissionRole >── Permission
                   │
              OrderProduct >── Product >── Category
                                  │  └──── Supplier
                     InventoryProduct >── Inventory
```

| Table               | Key fields |
| ------------------- | ---------- |
| `users`             | `username`, `password`, `role_id` |
| `roles`             | `name`, `description` |
| `permissions`       | `label`, `description` |
| `customers`         | `name`, `email`, `phone`, `address` |
| `suppliers`         | `name`, `email`, `phone`, `address` |
| `categories`        | `name`, `description` |
| `products`          | `name`, `description`, `price`, `category_id`, `supplier_id` |
| `orders`            | `total_price`, `status`, `customer_id`, `user_id` |
| `order_product`     | `order_id`, `product_id`, `quantity`, `price` |
| `inventories`       | `capacity`, `current_stock`, `location` |
| `inventory_product` | `inventory_id`, `product_id`, `quantity` |

## Getting Started

### Prerequisites

- PHP 8.2+ and [Composer](https://getcomposer.org/)
- [Node.js](https://nodejs.org/) and npm
- MySQL (or use SQLite)

> Prefer containers? See [dockerize-gestock](https://github.com/SifeYassine/dockerize-gestock) for a ready-made Docker Compose setup.

### Installation

```bash
git clone https://github.com/SifeYassine/gestock.git
cd gestock

composer install
npm install && npm run build

cp .env.example .env
php artisan key:generate
```

### Configure the database

Edit `.env`:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=gestock
DB_USERNAME=root
DB_PASSWORD=
```

### Migrate

```bash
php artisan migrate
```

### Create the default roles

Registration assigns the `Admin` or `User` role by name, so both must exist before the first sign-up:

```bash
php artisan tinker
>>> \App\Models\Role::create(['name' => 'Admin', 'description' => 'Administrator']);
>>> \App\Models\Role::create(['name' => 'User',  'description' => 'Standard user']);
```

Optionally seed demo data:

```bash
php artisan db:seed
```

### Run

```bash
php artisan serve
```

- API base URL: `http://127.0.0.1:8000/api`
- API docs: `http://127.0.0.1:8000/request-docs`

## API Overview

All routes are prefixed with `/api`. Except for `register` and `login`, every route requires `Authorization: Bearer <token>`.

### Auth

| Method | Endpoint         | Description |
| ------ | ---------------- | ----------- |
| POST   | `/auth/register` | `{ "username", "password" }` |
| POST   | `/auth/login`    | Returns a Sanctum token |
| POST   | `/auth/logout`   | Revokes the current token |

### Resources

Customers, orders, categories, suppliers, products, and inventories all follow the same pattern:

| Method | Endpoint                      |
| ------ | ----------------------------- |
| POST   | `/{resource}/create`          |
| GET    | `/{resource}/index`           |
| PUT    | `/{resource}/update/{id}`     |
| DELETE | `/{resource}/delete/{id}`     |

### Pivots

| Method | Endpoint |
| ------ | -------- |
| POST / PUT / DELETE | `/orders/{orderId}/products/{productId}/{create\|update\|delete}` |
| GET    | `/orders/products/index` |
| POST / PUT / DELETE | `/inventories/{inventoryId}/products/{productId}/{create\|update\|delete}` |
| GET    | `/inventories/products/index` |

### Administration

| Endpoint group | Access |
| -------------- | ------ |
| `/roles/*`, `/permissions/*`, `/roles/{roleId}/permissions/{permissionId}/*` | `Admin` role |
| `/users/index`, `/users/update/{id}`, `/users/delete/{id}` | Permissions `Index users`, `Update user`, `Delete user` |

## Related Repositories

- [gestock-frontend](https://github.com/SifeYassine/gestock-frontend): Vue.js frontend
- [dockerize-gestock](https://github.com/SifeYassine/dockerize-gestock): Docker setup

## Author

**Sifeddine Yassine** · [@SifeYassine](https://github.com/SifeYassine)
