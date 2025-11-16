# Marketplace 7-Layer Architecture

این فایل شامل دیاگرام و توضیح لایه‌های هفت‌گانه پروژه Marketplace است. تمام دیاگرام‌ها با **Mermaid** نوشته شده و قابل رندر در GitHub هستند.

---

## 1️⃣ Mermaid Diagram — 7 Layer Architecture

```mermaid
flowchart TD
    %% Presentation Layer
    subgraph L1[1. Presentation/UI]
        UI[Frontend Web / Blazor / API]
    end

    %% Application Layer
    subgraph L2[2. Application]
        App[Application Services / Use Cases]
    end

    %% Domain Layer
    subgraph L3[3. Domain]
        Domain[Entities, Value Objects, Business Rules]
    end

    %% Infrastructure Layer
    subgraph L4[4. Infrastructure]
        DB[(Database / EF Core)]
        Payment[Payment Gateway / External APIs]
        Repo[Repositories]
    end

    %% Reporting & Logging
    subgraph L5[5. Reporting & Logging]
        SalesReports[Sales Reports]
        VendorReports[Vendor Reports]
        AuditLogs[Audit Logs]
    end

    %% ML & Analytics
    subgraph L6[6. ML & Analytics]
        CustomerBehavior[Customer Behavior Analysis]
        Recommendation[Recommendation Engine]
    end

    %% Cross-Cutting
    subgraph L7[7. Cross-Cutting]
        Logging[Logging / Exception Handling]
        Security[Security / Validation]
        Helpers[Helpers]
    end

    %% Flow connections
    UI --> App
    App --> Domain
    App --> Repo
    Domain --> Repo
    Repo --> DB
    App --> Payment

    Domain --> SalesReports
    Domain --> VendorReports
    Domain --> AuditLogs
    SalesReports --> CustomerBehavior
    VendorReports --> CustomerBehavior
    AuditLogs --> CustomerBehavior
    CustomerBehavior --> Recommendation
    Recommendation --> UI

    App --> Logging
    Domain --> Logging
    Repo --> Logging
    Payment --> Logging
    UI --> Security
```

