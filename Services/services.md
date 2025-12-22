# 🧱 Microservices Capstone Project

A production-style microservices architecture with **clear service boundaries**, **JWT-based security**, **API Gateway**, and **fault-tolerant inter-service communication using Feign and Resilience4j**.

---

## 🧱 SERVICES OVERVIEW

| Layer | Service |
|------|--------|
| Entry | API Gateway |
| Identity | Auth Service |
| Business (Read) | Catalog Service |
| Business (Core) | Order Service |
| Finance | Payment Service |

---

## 🔐 AUTH SERVICE (Identity & Token Issuance)

### Responsibility
Authenticate users and issue JWT tokens only.

### Features
- User registration
- Login (username/password)
- Password hashing
- Role assignment (USER / ADMIN)
- JWT creation (claims + expiry)
- Refresh token (optional)

### Database
- users
- roles

### APIs

- POST /auth/register
- POST /auth/login


### Rules
- Does NOT validate JWT for other services
- Does NOT participate in request flow after login

---

## 🚪 API GATEWAY (Single Entry Point)

### Responsibility
Routing and first security checkpoint.

### Features
- Route requests to services
- Validate JWT (signature + expiry)
- Block unauthenticated requests
- Add user headers (optional)

### Routes
- /auth/** → Auth Service
- /catalog/** → Catalog Service
- /orders/** → Order Service
- /payments/** → Payment Service

  
### Rules
- No business logic
- No database

---

## 🛍️ CATALOG SERVICE (Read-Only Business Data)

### Responsibility
Serve product data to users.

### Features
- List products
- Product details
- Categories
- Images
- Display prices

### Database
- products
- categories
- product_images

### APIs
- GET /catalog/products
- GET /catalog/products/{id}
- GET /catalog/categories

  
### Rules
- No cart
- No users
- No orders
- Does NOT call other services

---

## 🧠 ORDER SERVICE (Core Business Orchestrator)

### Responsibility
Manage user intent and order lifecycle.

### Features
- Cart management
- Order creation
- Order status management
- Checkout orchestration
- Inter-service calls using Feign

### Database
- carts
- cart_items
- orders
- order_items

### APIs
- POST /orders/cart/add
- POST /orders/cart/remove
- POST /orders
- GET /orders/{id}
 
### Service Calls
- Catalog Service (product validation)
- Payment Service (payment initiation)

### Order Lifecycle
- CREATED → PAYMENT_PENDING → PAID / FAILED

  
---

## 💳 PAYMENT SERVICE (Money Isolation)

### Responsibility
Handle payments independently and securely.

### Features
- Initiate payments
- Payment gateway integration (Mock / Razorpay / Stripe)
- Handle callbacks
- Maintain payment records

### Database
- payments
- transactions

### APIs
- POST /payments/initiate
- POST /payments/callback

  
### Rules
- No products
- No cart
- No order creation

---

## 🔁 COMPLETE FLOW

1. Login  
   Frontend → Gateway → Auth Service → JWT

2. Browse Products  
   Frontend → Gateway → Catalog Service

3. Cart Actions  
   Frontend → Gateway → Order Service

4. Checkout  
   Frontend → Gateway → Order Service  
   Order → Catalog (Feign + Circuit Breaker)  
   Order → Payment (Feign + Circuit Breaker)

5. Payment Callback  
   Payment Gateway → Payment Service → Order Service

6. Order Status  
   Frontend → Gateway → Order Service

---

## ⚡ FEIGN & RESILIENCE4J

| Call | Feign | Circuit Breaker |
|-----|------|----------------|
| Order → Catalog | Yes | Yes |
| Order → Payment | Yes | Yes |
| Gateway → Services | No | No |
| Auth Validation | No | No |

---

## 🔐 JWT SECURITY MODEL

### Frontend
- Stores JWT
- Sends JWT with every request

### API Gateway
- Validates JWT signature and expiry

### Each Service
- Validates claims and roles
- Uses Spring Security filters

Auth Service is NOT called again after login.

---

## 🪜 BUILD ORDER

1. Auth Service  
2. API Gateway  
3. Catalog Service  
4. Order Service  
5. Payment Service  
6. Add Feign  
7. Add Resilience4j  
8. Integrate Frontend  

---

## 🚫 COMMON MISTAKES

- Shared database ❌
- Auth in business flow ❌
- Catalog calling Order ❌
- Payment creating orders ❌
- Gateway with business logic ❌

---

## 🎯 INTERVIEW ONE-LINER

**“Auth issues identity, Gateway secures entry, Catalog serves read-only data, Order orchestrates business flow, and Payment handles money — secured using JWT, Feign, and Resilience4j.”**
