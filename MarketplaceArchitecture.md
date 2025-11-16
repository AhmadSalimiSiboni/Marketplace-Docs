
# Marketplace System Architecture

این فایل شامل تمام دیاگرام‌ها و جریان سیستم فروشگاه/Marketplace است.  

---

## 1️⃣ DFD Level 0 — نمای کلی سیستم

```mermaid
flowchart TD
    C[Customer] -->|1. انتخاب محصولات / ثبت سفارش| O[Order System]
    O -->|2. ارسال عملیات به سیستم لجستیک| S[Shipment System]
    S -->|3. وضعیت ارسال و تحویل| C
```

## 2️⃣ DFD Level 1 — زیرسیستم سفارش

```mermaid
flowchart TD
    C[Customer] -->|A. انتخاب محصول| PC[Product Catalog]
    PC -->|B. افزودن به سبد| SC[Shopping Cart]
    SC -->|C. نهایی کردن سفارش| O[Order System]
    O -->|D. ارسال اطلاعات سفارش به لجستیک| S[Shipment System]
```

## 3️⃣ DFD Level 2 — جریان سفارش تا تحویل

```mermaid
flowchart TD
    C[Customer] -->|1. انتخاب کالا + تعداد + بسته‌بندی| PC[Product Catalog]
    PC -->|2. ثبت در سبد| SC[Shopping Cart]
    SC -->|3. ایجاد Order و OrderItem| O[Order System]
    O -->|4. بررسی روش ارسال برای هر OrderItem| Dec[Shipment Decision Engine]
    Dec -->|5. ساخت گروه مرسوله‌ها| S[Shipment]
    S -->|6. ساخت ShipmentItem برای هر OrderItem| SI[ShipmentItem]
    SI -->|7. ارسال، رهگیری، تحویل| D[Delivery]
    D -->|8. به‌روزرسانی وضعیت| C
```

## 4️⃣ ERD — روابط دیتابیس

```mermaid
erDiagram
    CATEGORY ||--o{ PRODUCT : has
    BRAND ||--o{ PRODUCT : has
    PRODUCT ||--o{ PRODUCTIMAGE : has
    PRODUCT ||--o{ PRODUCTPACKAGING : has
    PACKAGINGTYPE ||--o{ PRODUCTPACKAGING : defines

    VENDOR ||--o{ VENDORPRODUCT : offers
    PRODUCT ||--o{ VENDORPRODUCT : offered_in
    VENDORPRODUCT ||--o{ VENDORPRODUCTPLAN : has

    "ORDER" ||--o{ ORDERITEM : contains
    PRODUCT ||--o{ ORDERITEM : included_in
    PRODUCTPACKAGING ||--o{ ORDERITEM : packaged_as
    VENDORPRODUCTPLAN ||--o{ ORDERITEM : priced_by

    "ORDER" ||--o{ SHIPMENT : creates
    SHIPMENT ||--o{ SHIPMENTITEM : includes
    ORDERITEM ||--o{ SHIPMENTITEM : shipped_as
    SHIPMENTITEM ||--o{ DELIVERY : delivered_as

    SHIPMENTMETHOD ||--o{ SHIPMENTITEM : method
    SHIPMENTSTATUS ||--o{ SHIPMENT : status
    ORDERSTATUS ||--o{ "ORDER" : status
    PAYMENTSTATUS ||--o{ "ORDER" : payment_status
    ORDERITEMSTATUS ||--o{ ORDERITEM : status
    DELIVERYSTATUS ||--o{ DELIVERY : status
```

## 5️⃣ Class Diagram — مدل EF Core

```mermaid
classDiagram
    class Product {
        +int ProductId
        +string TechnicalCode
        +string Name
        +int SubCategoryId
        +int BrandId
    }
    class Vendor {
        +int VendorId
        +string BusinessName
        +decimal CommissionRate
    }
    class VendorProduct {
        +int VendorProductId
        +int VendorId
        +int ProductId
    }
    class VendorProductPlan {
        +int VendorProductPlanId
        +int VendorProductId
        +decimal Price
        +int QuantityAvailable
        +DateTime StartDate
        +DateTime? EndDate
    }
    class Order {
        +int OrderId
        +int CustomerId
        +DateTime OrderDate
        +decimal TotalAmount
        +int OrderStatusId
    }
    class OrderItem {
        +int OrderItemId
        +int OrderId
        +int ProductId
        +int ProductPackagingId
        +int Quantity
        +decimal UnitPrice
        +int VendorProductPlanId
        +int ShipmentMethodId
    }
    class Shipment {
        +int ShipmentId
        +int OrderId
        +DateTime CreatedAt
        +int ShipmentStatusId
    }
    class ShipmentItem {
        +int ShipmentItemId
        +int ShipmentId
        +int OrderItemId
        +int QuantityShipped
        +string TrackingNumber
        +int ShipmentMethodId
    }

    Product "1" -- "*" VendorProduct : offered_in
    Vendor "1" -- "*" VendorProduct : offers
    VendorProduct "1" -- "*" VendorProductPlan : has
    Order "1" -- "*" OrderItem : contains
    Order "1" -- "*" Shipment : creates
    Shipment "1" -- "*" ShipmentItem : includes
    OrderItem "1" -- "*" ShipmentItem : shipped_as
```

## 6️⃣ Sequence Diagram — جریان سفارش تا تحویل

```mermaid
sequenceDiagram
    participant C as Customer
    participant UI as Frontend
    participant Svc as OrderService
    participant DB as DB
    participant Dec as ShipmentDecisionEngine
    participant Sh as ShipmentService
    participant Carrier as CarrierAPI

    C->>UI: انتخاب محصولات و نهایی‌سازی سبد
    UI->>Svc: CreateOrder(request with OrderItems + preferred ShipmentMethod)
    Svc->>DB: INSERT Order, OrderItems
    Svc->>Dec: DecideShipmentGrouping(OrderItems)
    Dec->>DB: Create Shipment + ShipmentItem
    Dec-->>Svc: ShipmentCreated
    Svc-->>UI: Order Created
    Sh->>Carrier: request pickup / create tracking
    Carrier-->>Sh: trackingNumber
    Sh->>DB: update ShipmentItem (TrackingNumber, ShippedAt, Status=Shipped)
    Carrier-->>Sh: Delivered callback
    Sh->>DB: update ShipmentItem (DeliveredAt, Status=Delivered)
    Sh-->>Svc: notify OrderService
    Svc->>DB: update OrderItemStatus, OrderStatus
    Svc-->>UI: notify Customer
```
