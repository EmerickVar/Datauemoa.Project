# Sequence Diagram | Interaction Flow

## Interaction Flow of Notification, Sharing and Favorites System

```mermaid
sequenceDiagram
    participant U as User
    participant WEB as Web Application
    participant AUTH as Authentication Service
    participant DB as Database
    participant NOTIF as Notification Service
    participant SHARE as Sharing Service
    participant FAV as Favorites Service
    participant EXT as External Platforms
    participant BG as Background Processes

    Note over U,BG: Authentication and Initial Load Flow
    U->>WEB: Access page (Currency/Actions/Options/etc.)
    WEB->>AUTH: Verify authentication
    AUTH-->>WEB: User authenticated
    WEB->>DB: Load page data
    DB-->>WEB: Market data/news/calendar
    WEB-->>U: Display page with enabled modules

    Note over U,BG: Notification Configuration
    U->>WEB: Configure notifications
    WEB->>NOTIF: Set parameters
    Note right of NOTIF: Frequency: 3h/24h/96h/30d/120d/365d<br/>Variables: prices, changes, news
    NOTIF->>DB: Save user configuration
    NOTIF->>BG: Start background monitoring
    NOTIF-->>WEB: Configuration confirmed
    WEB-->>U: Notifications activated

    Note over U,BG: Background Process for Notifications
    loop Each configured interval
        BG->>DB: Check data changes
        alt There are significant changes
            BG->>NOTIF: Generate notification
            NOTIF->>DB: Register notification
            NOTIF->>U: Send notification
            Note right of U: Email, push, in-app notification
        end
    end

    Note over U,BG: Content Sharing Flow
    U->>WEB: Click share button
    WEB->>SHARE: Request sharing on platform
    Note right of SHARE: Facebook/X/LinkedIn/Email/WhatsApp
    SHARE->>DB: Register sharing event
    SHARE->>EXT: Generate URL and open platform
    Note right of EXT: Page Name + Link + Image
    EXT-->>SHARE: Opening confirmation
    SHARE->>DB: Update click counter
    SHARE-->>WEB: Process completed
    WEB-->>U: Content shared successfully

    Note over U,BG: Favorites Management
    U->>WEB: Add/remove favorite
    WEB->>FAV: Process request
    FAV->>DB: Update user favorites
    Note right of DB: Dedicated account page<br/>Page Name + Link + Image
    FAV-->>WEB: Favorite updated
    WEB-->>U: Change confirmed

    Note over U,BG: Access to Favorites Page
    U->>WEB: Access favorites page
    WEB->>FAV: Load user favorites
    FAV->>DB: Query saved favorites
    DB-->>FAV: Favorites list
    FAV-->>WEB: Favorites data
    WEB-->>U: Display favorites page

    Note over U,BG: Interaction Tracking
    par Parallel tracking
        U->>WEB: Interaction with favorite
        and
        U->>WEB: Click on shared link
        and
        U->>WEB: Response to notification
    end
    WEB->>DB: Register all interactions
    DB-->>WEB: Registration confirmation

    Note over U,BG: Error Cases and Recovery
    U->>WEB: Action that fails
    WEB->>DB: Operation attempt
    DB-->>WEB: Connection error
    WEB->>WEB: Attempt reconnection
    alt Successful reconnection
        WEB->>DB: Retry operation
        DB-->>WEB: Successful operation
        WEB-->>U: Action completed
    else Persistent error
        WEB-->>U: Show error message
        Note right of U: "Please try again later"
    end
```

### Description of Interactions

**Main Actors:**

-   **User**: Interacts with the web application
-   **Web Application**: Frontend that handles requests
-   **Services**: Backend specialized by functionality
-   **Database**: Persistent storage
-   **Background Processes**: Automatic monitoring
-   **External Platforms**: Social networks and third-party services

**Key Flows:**

1. **Authentication and Loading**: User verification and data loading
2. **Notifications**: Configuration and automatic sending
3. **Sharing**: Integration with social platforms
4. **Favorites**: Preferred content management
5. **Tracking**: Interaction monitoring
6. **Error Handling**: Recovery from failures
