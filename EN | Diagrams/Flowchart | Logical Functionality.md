# Flowchart | Logical Functionality

## Logical Flow of Notification, Sharing and Favorites System

```mermaid
flowchart TD
    A[User accesses system] --> B{Is authenticated?}
    B -->|No| C[Redirect to login]
    B -->|Yes| D[Load requested page]

    D --> E{What type of page?}
    E -->|Currency/Actions/Options/Cryptos/Metals| F[Load market data]
    E -->|News| G[Load news]
    E -->|Economic Calendar| H[Load calendar events]
    E -->|Market Square| I[Load projects]
    E -->|Others| J[Load specific data]

    F --> K[Display data with average prices]
    G --> L[Display news by category]
    H --> M[Display events by country/date]
    I --> N[Display project list]
    J --> O[Display specific content]

    K --> P[Enable modules]
    L --> P
    M --> P
    N --> P
    O --> P

    P --> Q{Notifications Module?}
    Q -->|Yes| R[Configure notifications]
    R --> R1[Define frequency 3h/24h/96h/30d/120d/365d]
    R1 --> R2[Configure variables to monitor]
    R2 --> R3[Set notification thresholds]
    R3 --> R4[Activate background service]

    P --> S{Sharing Module?}
    S -->|Yes| T[Display sharing buttons]
    T --> T1[Facebook/X/LinkedIn/Email/WhatsApp]
    T1 --> T2{User clicks?}
    T2 -->|Yes| T3[Generate custom URL]
    T3 --> T4[Open target platform]
    T4 --> T5[Register sharing event]

    P --> U{Favorites Module?}
    U -->|Yes| V[Display favorites button]
    V --> V1{User adds to favorites?}
    V1 -->|Yes| V2[Save to dedicated account page]
    V2 --> V3[Generate quick access link]
    R4 --> W[Background process running]
    W --> X[Monitor changes according to frequency]
    X --> Y{Are there significant changes?}
    Y -->|No| X
    Y -->|Yes| Z[Generate notification]
    Z --> AA[Send to user]
    AA --> AB[Register in history]
    AB --> X

    T5 --> AC[End of sharing process]
    V3 --> AD[End of favorites process]

    AC --> AE[User continues browsing]
    AD --> AE
    AB --> AE
    AE --> D

    style A fill:#e1f5fe
    style W fill:#fff3e0
    style R fill:#f3e5f5
    style T fill:#e8f5e8
    style V fill:#fce4ec
```

### Flow Description

**Main Flow:**

1. **Authentication**: User verification
2. **Page Loading**: Content type determination
3. **Data Loading**: Obtaining specific information according to type
4. **Module Enablement**: Functionality activation

**Notifications Module:**

-   Frequency configuration (3h, 24h, 96h, 30d, 120d, 365d)
-   Continuous background monitoring
-   Automatic notification generation

**Sharing Module:**

-   Multi-platform support
-   Click and engagement tracking
-   Custom URL generation

**Favorites Module:**

-   Management in dedicated account page
-   Quick access to favorite content
-   User preference persistence
