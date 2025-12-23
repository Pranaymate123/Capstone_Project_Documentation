🔐 AUTH SERVICE (Identity only) Entities

✅ User

✅ Role

User \-\-\-- id email password_hash status

Role \-\-\-- id name (USER / ADMIN)

❌ Must NOT contain

Cart

Orders

Payments

Products

Auth = who the user is, nothing else.

🛍️ CATALOG SERVICE (Product truth) Entities

✅ Product

✅ Category

Category 1 ────\< Product

Product \-\-\-\-\-\-- id name price active category_id

❌ Must NOT contain

Cart

Orders

Users

Catalog = what can be sold.

🧠 ORDER SERVICE (Core business) Entities

✅ Cart

✅ CartItem

✅ Order

✅ OrderItem

Cart 1 ────\< CartItem Order 1 ────\< OrderItem

CartItem.product_id → Catalog.Product.id (ID only) OrderItem.product_id
→ Catalog.Product.id (ID only)

Important

user_id comes from JWT

No FK to Auth DB

Cascade allowed inside Order Service only

Order = user intent + lifecycle.

💳 PAYMENT SERVICE (Money only) Entities

✅ Payment

(Optional) Transaction

Payment \-\-\-\-\-\-- id order_id (Order Service ID) amount status

Payment = money handling, nothing else.
