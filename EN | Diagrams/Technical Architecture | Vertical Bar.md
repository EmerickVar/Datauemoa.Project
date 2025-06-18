# Technical Architecture | Vertical Bar

## Vertical Bar System Architecture Diagram

```mermaid
graph TB
    subgraph "Frontend - User Interface"
        VB[Vertical Bar]
        VB --> VB1[Notifications Component]
        VB --> VB2[Share Component]
        VB --> VB3[Favorites Component]
        VB --> VB4[Management Panel]
        VB --> VB5[Registration Invitation]
    end

    subgraph "State Management"
        SM[State Manager]
        SM --> SM1[Logged User State]
        SM --> SM2[Guest User State]
        SM --> SM3[Cookies State]
        SM --> SM4[Database State]
    end

    subgraph "Backend Services"
        NS[NotificationService]
        FS[FavoriteService]
        SS[SharingService]
        US[UserService]
        CS[CookieService]
        MS[MigrationService]
    end

    subgraph "Database"
        DB[(MySQL Database)]
        DB --> T1[user_notification_configs]
        DB --> T2[user_favorites]
        DB --> T3[users]
        DB --> T4[pages]
        DB --> T5[notification_logs]
        DB --> T6[sharing_logs]
    end

    subgraph "Temporary Storage"
        COOKIES[Browser Cookies]
        COOKIES --> C1[notification_configs]
        COOKIES --> C2[user_favorites]
        COOKIES --> C3[session_data]
    end

    subgraph "External APIs"
        API[External APIs]
        API --> API1[Facebook API]
        API --> API2[Twitter/X API]
        API --> API3[LinkedIn API]
        API --> API4[Email Service]
        API --> API5[WhatsApp Business]
    end

    %% Frontend Connections
    VB1 --> SM1
    VB1 --> SM2
    VB2 --> SS
    VB3 --> SM1
    VB3 --> SM2
    VB4 --> SM1
    VB5 --> US

    %% State Manager Connections
    SM1 --> NS
    SM1 --> FS
    SM2 --> CS
    SM3 --> COOKIES
    SM4 --> DB

    %% Services Connections
    NS --> DB
    FS --> DB
    SS --> API
    US --> DB
    CS --> COOKIES
    MS --> DB
    MS --> COOKIES

    %% Migration Flow
    CS -.->|"Convert on registration"| MS
    MS -.->|"Transfer data"| DB

    %% Styles
    classDef frontend fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    classDef backend fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    classDef database fill:#e8f5e8,stroke:#388e3c,stroke-width:2px
    classDef storage fill:#fff8e1,stroke:#f57c00,stroke-width:2px
    classDef external fill:#fce4ec,stroke:#c2185b,stroke-width:2px

    class VB,VB1,VB2,VB3,VB4,VB5 frontend
    class SM,SM1,SM2,SM3,SM4,NS,FS,SS,US,CS,MS backend
    class DB,T1,T2,T3,T4,T5,T6 database
    class COOKIES,C1,C2,C3 storage
    class API,API1,API2,API3,API4,API5 external
```

## Detailed Data Flow

```mermaid
sequenceDiagram
    participant U as User
    participant VB as Vertical Bar
    participant SM as State Manager
    participant CS as Cookie Service
    participant NS as Notification Service
    participant DB as Database
    participant COOK as Browser Cookies

    Note over U,COOK: Scenario: Non-Logged User

    U->>VB: Configure notification
    VB->>SM: Check user state
    SM-->>VB: User not logged in
    VB->>CS: Save configuration
    CS->>COOK: Write cookie
    COOK-->>CS: Confirmation
    CS-->>VB: Configuration saved
    VB-->>U: Show confirmation + registration invitation

    Note over U,COOK: Scenario: User Registers

    U->>VB: Click "Register"
    VB->>NS: Registration process
    NS->>DB: Create user
    DB-->>NS: User created
    NS->>CS: Migrate existing cookies
    CS->>COOK: Read configurations
    COOK-->>CS: Cookies data
    CS->>DB: Transfer to DB tables
    DB-->>CS: Transfer completed
    CS->>COOK: Clear cookies
    COOK-->>CS: Cookies deleted
    CS-->>VB: Migration completed
    VB-->>U: Full access enabled

    Note over U,COOK: Scenario: Logged User

    U->>VB: Configure notification
    VB->>SM: Check user state
    SM-->>VB: User logged in
    VB->>NS: Save configuration
    NS->>DB: Insert in table
    DB-->>NS: Configuration saved
    NS-->>VB: Confirmation
    VB-->>U: Notification configured
```

## Technical Specifications

### Frontend Components (JavaScript/Twig)

#### Vertical Bar Component

```javascript
class VerticalBar {
	constructor() {
		this.userState =
			null;
		this.cookieService =
			new CookieService();
		this.notificationService =
			new NotificationService();
		this.favoriteService =
			new FavoriteService();
	}

	async initialize() {
		this.userState =
			await this.checkUserState();
		this.render();
		this.attachEventListeners();
	}

	render() {
		// Render according to user state
		if (
			this
				.userState
				.isLoggedIn
		) {
			this.renderLoggedUserBar();
		} else {
			this.renderGuestUserBar();
		}
	}
}
```

### Backend Services (Symfony 6.4)

#### NotificationService

```php
class NotificationService
{
    public function saveConfiguration(User $user, array $config): void
    {
        $notificationConfig = new UserNotificationConfig();
        $notificationConfig->setUser($user);
        $notificationConfig->setPageId($config['page_id']);
        $notificationConfig->setModuleId($config['module_id']);
        $notificationConfig->setFrequency($config['frequency']);
        $notificationConfig->setVariables($config['variables']);

        $this->entityManager->persist($notificationConfig);
        $this->entityManager->flush();
    }
}
```

#### CookieService

```php
class CookieService
{
    public function saveNotificationConfig(array $config): void
    {
        $existingConfigs = $this->getNotificationConfigs();
        $existingConfigs[] = $config;

        setcookie(
            'notification_configs',
            json_encode(['configs' => $existingConfigs]),
            time() + (86400 * 30), // 30 days
            '/',
            null,
            true,
            true
        );
    }

    public function migrateCookiesToDatabase(User $user): void
    {
        $notificationConfigs = $this->getNotificationConfigs();
        $favorites = $this->getFavoriteConfigs();

        foreach ($notificationConfigs as $config) {
            $this->notificationService->saveConfiguration($user, $config);
        }

        foreach ($favorites as $favorite) {
            $this->favoriteService->saveFavorite($user, $favorite);
        }

        $this->clearAllCookies();
    }
}
```

### Database Structure

#### user_notification_configs

```sql
CREATE TABLE user_notification_configs (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    page_id VARCHAR(50) NOT NULL,
    module_id VARCHAR(50) NOT NULL,
    frequency ENUM('3h', '24h', '96h', '30d', '120d', '365d') NOT NULL,
    variables JSON NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    INDEX idx_user_active (user_id, is_active),
    INDEX idx_page_module (page_id, module_id)
);
```

#### user_favorites

```sql
CREATE TABLE user_favorites (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    page_id VARCHAR(50) NOT NULL,
    page_name VARCHAR(255) NOT NULL,
    page_url VARCHAR(500) NOT NULL,
    image_url VARCHAR(500),
    category VARCHAR(50),
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    UNIQUE KEY unique_user_page (user_id, page_id),
    INDEX idx_user_category (user_id, category),
    INDEX idx_user_active (user_id, is_active)
);
```

### Performance Considerations

1. **Redis Cache**: For frequently accessed user configurations
2. **Lazy Loading**: Load bar components on demand
3. **Debouncing**: To avoid multiple requests in rapid configurations
4. **Batch Operations**: Group cookie migration operations
5. **CDN**: For vertical bar static resources
