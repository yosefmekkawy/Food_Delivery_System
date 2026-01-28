# Food Delivery System Analysis

## 1. Actor: Restaurant
**Feature: Menu Management**
*  Manage items (Create, update, delete dishes).
*  Manage item details (Prices, descriptions, images).
*  Manage availability (Mark items as in/out of stock).

**Feature: Restaurant Profile**
*  Manage restaurant information (Address, logo, name).
*  Manage settings (Opening/closing hours, preparation time).

**Feature: Business Intelligence**
*  View order analytics (Total revenue, order counts).
*  View order history (Past orders logs).

**Feature: Marketing**
*  Create and manage restaurant-specific promo codes.

---

## 2. Actor: Customer
**Feature: Authentication & Account**
*  Login and Sign up (Email, phone, social auth).
*  Manage Profile (Update name, email).
*  Address Management (Add, edit, delete delivery addresses).
*  Save Favourite Restaurants.

**Feature: Browsing Restaurants**
*  Browse restaurants (List view, categories).
*  View Restaurant Details (Menu, info).

**Feature: Cart & Ordering**
*  Add items to cart.
*  Edit cart (Update quantities, remove items).
*  Checkout (Review order, select address).
*  Place order.

**Feature: Order Tracking**
*  Track active order status (Pending, Preparing, On the way).
*  View order history.

---

## 3. Actor: Payment Gateway
**Feature: Transaction Processing**
*  Manage user payment methods.
*  Validate transaction details.
*  Approve or decline transactions.

---

## 4. Actor: System Admin
**Feature: System Marketing**
*  Create system-level promo codes.
*  Push notifications (Offers, restaurant alerts).

**Feature: Content Management**
*  Manage system categories (Define global tags like Pizza, Burgers, Sushi).
*  Vendor Approval (Verify and approve new restaurant sign-ups).

**Feature: Refund Management**
*  Refunds (Manage disputes and issue refunds).

---

## Flow Chart For "Place Order" Use Case

```mermaid
flowchart TD
    A[Add Items to Cart] --> B[Checkout Order]
    B --> C{Validate User Data \n & Address}
    C -- Invalid --> D[Show Error]
    C -- Valid --> E[Check Restaurant Availability]
    E -- Closed --> F[Show Error]
    E -- Available --> G[Calculate Total \n + Delivery Fee]
    G --> H[Select Payment Type]
    
    H --> I{Payment Method?}
    I -- Cash --> J[Set Status: COD]
    I -- Visa --> K[Process Gateway]
    
    K --> L{Transaction Success?}
    L -- No --> H
    L -- Yes --> M[Set Status: Paid]
    
    J --> N[Save Order]
    M --> N
    N --> O[Clear Cart]
    O --> P[Notify User & Restaurant]
```
---

## Sequence Diagram For "Place Order" Use Case

```mermaid
sequenceDiagram
    actor User
    participant System
    participant Gateway

    User->>System: Add Items to Cart
    System-->>User: Cart Updated

    User->>System: Click "Place Order"

%% Validation Step
    System->>System: Validate Address & Inventory

    alt Data Invalid or Out of Stock
        System-->>User: Show Error Message
    else Data Valid
        System-->>User: Request Payment Method
        User->>System: Select Payment Method

        alt Payment Method is Visa
            System->>Gateway: Initiate Transaction
            Gateway-->>System: Transaction Result

            alt Payment Declined
                System-->>User: Show "Payment Failed"
            else Payment Success
                System->>System: Create Order (Status: Paid)
                System-->>User: Show "Order Confirmed"
            end

        else Payment Method is Cash (Cash On Delivery)
            System->>System: Create Order (Status: Pending Payment)
            System-->>User: Show "Order Confirmed"
        end
    end
```

## Pseudo Code — Place Order Use Case

```text
if (restaurant is closed OR address is out of range)
    decline

if (user data or address is invalid)
    decline

if (any item in cart is out of stock)
    decline

calculate total price

if (payment type is Visa) {
    if (transaction fails)
        fail
    else 
        mark payment as paid
} 
else {
    mark payment as cash_on_delivery
}

create order()
set order status (pending_delivery)
empty user cart
```
## Er Diagram — Place Order Use Case
```mermaid
erDiagram
User ||--o{ Order : places
User ||--|| Cart : has

    Restaurant ||--o{ Order : receives
    Restaurant ||--o{ Product : owns
    
    Order ||--|{ OrderItem : contains
    Cart ||--|{ CartItem : contains

    User {
        int id
        string name
        string phone
        string email
    }

    Product {
        int id
        string name
        decimal price
        int stock_quantity
        boolean is_active
        int restaurant_id
    }

    Cart {
        int id
        int user_id
        int restaurant_id "Enforces single-restaurant orders"
    }

    CartItem {
        int id
        int cart_id
        int product_id
        int quantity
    }

    Order {
        int id
        int user_id
        int restaurant_id
        decimal total_amount
        decimal delivery_fee
        string status "pending, paid, cooking, delivering, completed"
        string payment_method "cash, visa"
        string delivery_address "Snapshot of address at time of order"
        datetime created_at
    }

    OrderItem {
        int id
        int order_id
        int product_id
        decimal price_at_purchase 
        int quantity
    }
```
---