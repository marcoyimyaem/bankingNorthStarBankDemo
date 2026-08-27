# Banking App Starter — Admin Dashboard Edition

A teaching/demo banking application split into two independently run projects:

- `backend/` — Java 21 + Spring Boot REST API, Spring Security session login, MySQL/JPA, Swagger/OpenAPI, Thymeleaf + Bootstrap UI, customer/admin roles, and unit tests.
- `frontend/` — React + Vite responsive banking UI consuming the backend API.

> **Demo only:** this is intentionally small and is not production banking software.

## Demo users

| User | Password | Role | Account | Starting balance |
|---|---|---|---|---:|
| `alice` | `password123` | USER | `100000001` | 5000.00 |
| `bob` | `password123` | USER | `100000002` | 2500.00 |
| `admin` | `password123` | ADMIN | `900000001` | 100000.00 |

The admin still owns a normal bank account, so it can deposit into its own account and transfer money just like a regular banking user. When the `admin` user logs into React, the application automatically switches to the dedicated Admin Dashboard.

## Admin Dashboard features

The React admin dashboard includes:

- Customer listing
- Search by full name, username, or account number
- Customer balance display
- Admin top-up of a customer account
- Admin debit of a customer account, with insufficient-funds validation
- Customer transaction inspection
- Separate transaction types for `ADMIN_TOPUP` and `ADMIN_DEBIT`
- Admin's own deposit and cash-transfer controls
- Responsive mobile/tablet layout

Admin top-ups/debits can only target accounts owned by `USER` accounts. The server also protects `/api/admin/**` with `ROLE_ADMIN`; hiding buttons in React is not the security mechanism.

## Mobile React UI

The customer and admin interfaces are responsive.

At tablet/mobile widths:

- The desktop sidebar is removed.
- The admin customer table becomes stacked customer cards.
- Top-up, debit, and history actions use large touch targets.
- The admin statistics stack vertically on narrow phones.
- Transaction history reflows to a single-column-friendly layout.
- Admin adjustment forms become bottom-sheet-style dialogs.
- The design supports widths down to roughly 320px without requiring page-level horizontal scrolling.

To test mobile mode in Chrome/Edge, open DevTools, enable **Toggle device toolbar**, then try widths such as 375px or 390px.

## Prerequisites

- Java 21
- Maven 3.9+
- Node.js 20.19+ or 22.12+
- npm
- MySQL 8.x, or Docker for the included compose file

## 1. Start MySQL

From the project root:

```bash
docker compose up -d
```

This starts MySQL on `localhost:3306` with database `banking_app`, username `root`, password `root`.

Or use your own MySQL and configure:

```bash
export DB_URL='jdbc:mysql://localhost:3306/banking_app?createDatabaseIfNotExist=true&useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC'
export DB_USERNAME='root'
export DB_PASSWORD='your-password'
```

## 2. Run Spring Boot

Terminal 1:

```bash
cd backend
mvn spring-boot:run
```

Backend URLs:

- API: `http://localhost:8080/api`
- Swagger: `http://localhost:8080/swagger-ui.html`
- Thymeleaf UI: `http://localhost:8080/server-ui`

Run tests:

```bash
cd backend
mvn test
```

The test suite includes the original banking service tests plus admin top-up/debit tests.

## 3. Run React

Terminal 2:

```bash
cd frontend
npm install
npm run dev
```

Open:

```text
http://localhost:5173
```

Vite proxies `/api/*` to Spring Boot on port `8080`, so Spring and React run independently.

## Admin test flow

1. Log in with `admin / password123`.
2. The React Admin Dashboard should load automatically.
3. Search for `alice`, `bob`, `100000001`, etc.
4. Use **Top up** to add money directly to a selected customer.
5. Use **Debit** to subtract money from that customer. The debit cannot exceed the current balance.
6. Click **Transactions** (desktop) or **History** (mobile) to inspect the customer's activity.
7. You should see admin adjustments as `Admin top-up` or `Admin debit` in transaction history.
8. Use the **Admin account** section to deposit to or transfer from account `900000001`.

## Main endpoints

| Method | Endpoint | Access | Purpose |
|---|---|---|---|
| GET | `/api/auth/csrf` | Public | Get CSRF token |
| POST | `/api/auth/login` | Public | Login |
| POST | `/api/auth/logout` | Authenticated | Logout |
| GET | `/api/auth/me` | Authenticated | Current user |
| GET | `/api/account` | USER / ADMIN | Own account summary |
| POST | `/api/account/deposit` | USER / ADMIN | Deposit into own account |
| POST | `/api/account/transfer` | USER / ADMIN | Transfer from own account |
| GET | `/api/account/transactions` | USER / ADMIN | Own transaction history |
| GET | `/api/admin/customers?search=` | ADMIN only | List/search customers |
| POST | `/api/admin/customers/{accountNumber}/top-up` | ADMIN only | Add customer balance |
| POST | `/api/admin/customers/{accountNumber}/debit` | ADMIN only | Subtract customer balance |
| GET | `/api/admin/customers/{accountNumber}/transactions` | ADMIN only | Inspect customer transactions |

Example admin top-up JSON:

```json
{
  "amount": 500.00,
  "description": "Branch cash adjustment"
}
```

Example debit JSON:

```json
{
  "amount": 100.00,
  "description": "Balance correction"
}
```

POST endpoints use the same Spring Security CSRF flow as the rest of the application. The React `api.js` fetches the token and sends it automatically.

## How the two projects connect

```text
Browser
  |
  v
React/Vite http://localhost:5173
  |
  | /api/*
  v
Spring Boot http://localhost:8080
  |
  v
MySQL localhost:3306
```

## Production note

This is still the first-version teaching project with added admin functionality. The separate hardened version should be used as the basis for production-oriented controls such as migrations, immutable ledgers, audit systems, rate limits, idempotency, external secrets, strict production CORS, monitoring, fraud controls, and stronger authorization policies.
