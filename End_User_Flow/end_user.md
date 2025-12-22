# 🛒 E-Commerce Microservices Architecture – Communication & Flow

This document explains **which service communicates with which**, **why that communication exists**, and the **complete end-user flow** of the system.

The goal is **clear separation of responsibilities**, **secure access**, and **correct business orchestration**.

---

## 🔁 WHO COMMUNICATES WITH WHOM (AND WHY)

### ✅ Frontend → API Gateway
**Why**
- Single entry point to backend
- No direct access to internal services
- Central JWT validation

📌 Frontend never calls services directly.

---

### ✅ API Gateway → Auth Service
**Why**
- User login & registration
- JWT token issuance

📌 Happens **only during login / registration**

---

### ✅ API Gateway → Catalog Service
**Why**
- Product browsing is read-only
- No business orchestration needed

📌 Direct routing, no Feign required

---

### ✅ API Gateway → Order Service
**Why**
- Cart operations
- Order creation
- Order status & history

📌 Order Service is the **core business brain**

---

### ✅ Order Service → Catalog Service (Feign + Circuit Breaker)
**Why**
- Validate product existence
- Validate latest price during checkout

📌 Catalog Service is the **source of product truth**  
📌 Catalog **never** calls Order Service

---

### ✅ Order Service → Payment Service (Feign + Circuit Breaker)
**Why**
- Initiate payment
- Isolate money-related logic

📌 Order controls the flow, Payment handles money

---

### ✅ Payment Service → Order Service (Callback)
**Why**
- Update order status after payment result

📌 Payment **never touches Order DB directly**

---

## 🚫 ILLEGAL COMMUNICATION (ARCHITECTURE VIOLATIONS)

The following interactions are **not allowed**:

- ❌ Catalog → Order  
- ❌ Payment → Catalog  
- ❌ Auth → Any business service  
- ❌ Frontend → Services directly  

If any of these exist, the architecture is **incorrect**.

---

## 👣 END-USER FLOW (STEP-BY-STEP)

### 🟢 Step 1: User opens the app
- Frontend loads
- Checks if JWT exists
- If not logged in → redirect to login

---

### 🟢 Step 2: User logs in
- User → Frontend → API Gateway → Auth Service

  
- Credentials verified
- JWT issued
- User is authenticated

📌 Auth Service is **done** after this step

---

### 🟢 Step 3: User browses products
- User → Frontend → API Gateway → Catalog Service


- Product list
- Product details
- Categories

📌 Fast, read-only, safe

---

### 🟢 Step 4: User adds items to cart
- User → Frontend → API Gateway → Order Service

  
- Cart created
- Items added / removed

📌 Cart lives **inside Order Service**

---

### 🟢 Step 5: User clicks “Checkout”
- Frontend → API Gateway → Order Service

  
Inside **Order Service**:
1. Calls Catalog Service to validate products & prices
2. Creates order
3. Sets status → `PAYMENT_PENDING`
4. Calls Payment Service

---

### 🟢 Step 6: User completes payment
- Payment Gateway → Payment Service


- Payment success / failure
- Transaction recorded

---

### 🟢 Step 7: Payment callback updates order
- Payment Service → Order Service

  
- Order updated to `PAID` or `FAILED`

📌 Order Service owns the **order lifecycle**

---

### 🟢 Step 8: User checks order status
- Frontend → API Gateway → Order Service


- Order status shown
- Order history displayed

---

## 🧠 FINAL MENTAL MODEL

- 🔐 **Auth** = Identity & token issuance  
- 🚪 **API Gateway** = Gatekeeper  
- 🛍️ **Catalog** = Read-only product truth  
- 🧠 **Order** = Brain & orchestrator  
- 💳 **Payment** = Money specialist  

---

## ❗ ARCHITECTURE PRINCIPLES

- No service chains
- No shared databases
- No business logic in Gateway
- Order Service orchestrates everything
- JWT validated on backend

**No confusion. No shortcuts. No fake microservices.**
