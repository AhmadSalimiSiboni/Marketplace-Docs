# Shopping System Flowchart - Login & cart

```
┌─────────────────────────────┐
│          User               │
└─────────────┬───────────────┘
              │ Login Request
              ▼
┌─────────────────────────────┐
│       Auth Service           │
│  - Validate Credentials      │
│  - Single Session Check      │
│  - Sliding Expiration        │
└─────────────┬───────────────┘
              │
              ├─ Active Session Exists?
              │      ├─ No → Create Session → Continue
              │      └─ Yes → Show Warning → Option: Force Logout
              │
              ▼
┌─────────────────────────────┐
│ Middleware: SlidingSession  │
│ - Check ActiveSessionId      │
│ - Check SessionExpiresAt     │
│ - Read Session Duration from │
│   SystemSettings             │
│ - Refresh ExpiresAt if Active│
│ - Update LastActivity        │
│ - Logout if Expired          │
└─────────────┬───────────────┘
              │
              ▼
┌─────────────────────────────┐
│       Cart Service           │
│ - Check Active Cart (!IsDeleted) │
│     ├─ No → Create Cart      │
│     └─ Yes → Resume / Expire / Soft Delete │
│ - Add Item → Reserve Inventory │
│ - Remove Item → Soft Delete & Release │
│ - Checkout → Commit Order & Soft Delete │
└─────────────┬───────────────┘
              │
              ▼
┌─────────────────────────────┐
│      Inventory Service       │
│ - Transaction-safe Reserve   │
│ - Transaction-safe Release   │
└─────────────┬───────────────┘
              │
              ▼
┌─────────────────────────────┐
│  Background Service          │
│ - Expire Carts (Soft Delete) │
│ - Expire Sessions (Sliding Expiration) │
└─────────────┬───────────────┘
              │
              ▼
┌─────────────────────────────┐
│   SystemSettings Table       │
│ - Key / Value / Description  │
│ - SessionTimeoutMinutes      │
│ - MaxCartItems / Other Params│
└─────────────────────────────┘
```

