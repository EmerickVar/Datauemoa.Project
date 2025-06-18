# Arquitectura Técnica | Barra Vertical

## Diagrama de Arquitectura del Sistema de Barra Vertical

```mermaid
graph TB
    subgraph "Frontend - Interfaz de Usuario"
        VB[Barra Vertical]
        VB --> VB1[Componente Notificaciones]
        VB --> VB2[Componente Compartir]
        VB --> VB3[Componente Favoritos]
        VB --> VB4[Panel de Gestión]
        VB --> VB5[Invitación Registro]
    end

    subgraph "Gestión de Estado"
        SM[State Manager]
        SM --> SM1[Estado Usuario Loggeado]
        SM --> SM2[Estado Usuario Invitado]
        SM --> SM3[Estado Cookies]
        SM --> SM4[Estado Base Datos]
    end

    subgraph "Servicios Backend"
        NS[NotificationService]
        FS[FavoriteService]
        SS[SharingService]
        US[UserService]
        CS[CookieService]
        MS[MigrationService]
    end

    subgraph "Base de Datos"
        DB[(MySQL Database)]
        DB --> T1[user_notification_configs]
        DB --> T2[user_favorites]
        DB --> T3[users]
        DB --> T4[pages]
        DB --> T5[notification_logs]
        DB --> T6[sharing_logs]
    end

    subgraph "Almacenamiento Temporal"
        COOKIES[Browser Cookies]
        COOKIES --> C1[notification_configs]
        COOKIES --> C2[user_favorites]
        COOKIES --> C3[session_data]
    end

    subgraph "APIs Externas"
        API[External APIs]
        API --> API1[Facebook API]
        API --> API2[Twitter/X API]
        API --> API3[LinkedIn API]
        API --> API4[Email Service]
        API --> API5[WhatsApp Business]
    end

    %% Conexiones Frontend
    VB1 --> SM1
    VB1 --> SM2
    VB2 --> SS
    VB3 --> SM1
    VB3 --> SM2
    VB4 --> SM1
    VB5 --> US

    %% Conexiones State Manager
    SM1 --> NS
    SM1 --> FS
    SM2 --> CS
    SM3 --> COOKIES
    SM4 --> DB

    %% Conexiones Servicios
    NS --> DB
    FS --> DB
    SS --> API
    US --> DB
    CS --> COOKIES
    MS --> DB
    MS --> COOKIES

    %% Flujo de Migración
    CS -.->|"Conversión al registrarse"| MS
    MS -.->|"Transferir datos"| DB

    %% Estilos
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

## Flujo de Datos Detallado

```mermaid
sequenceDiagram
    participant U as Usuario
    participant VB as Barra Vertical
    participant SM as State Manager
    participant CS as Cookie Service
    participant NS as Notification Service
    participant DB as Base de Datos
    participant COOK as Browser Cookies

    Note over U,COOK: Escenario: Usuario No Loggeado

    U->>VB: Configura notificación
    VB->>SM: Verificar estado usuario
    SM-->>VB: Usuario no loggeado
    VB->>CS: Guardar configuración
    CS->>COOK: Escribir cookie
    COOK-->>CS: Confirmación
    CS-->>VB: Configuración guardada
    VB-->>U: Mostrar confirmación + invitación registro

    Note over U,COOK: Escenario: Usuario Se Registra

    U->>VB: Clic en "Registrarse"
    VB->>NS: Proceso de registro
    NS->>DB: Crear usuario
    DB-->>NS: Usuario creado
    NS->>CS: Migrar cookies existentes
    CS->>COOK: Leer configuraciones
    COOK-->>CS: Datos cookies
    CS->>DB: Transferir a tablas BD
    DB-->>CS: Transferencia completada
    CS->>COOK: Limpiar cookies
    COOK-->>CS: Cookies eliminadas
    CS-->>VB: Migración completada
    VB-->>U: Acceso completo habilitado

    Note over U,COOK: Escenario: Usuario Loggeado

    U->>VB: Configura notificación
    VB->>SM: Verificar estado usuario
    SM-->>VB: Usuario loggeado
    VB->>NS: Guardar configuración
    NS->>DB: Insertar en tabla
    DB-->>NS: Configuración guardada
    NS-->>VB: Confirmación
    VB-->>U: Notificación configurada
```

## Especificaciones Técnicas

### Componentes Frontend (JavaScript/Twig)

#### Barra Vertical Component

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
		// Renderizar según estado del usuario
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

### Servicios Backend (Symfony 6.4)

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
            time() + (86400 * 30), // 30 días
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

### Estructura de Base de Datos

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

### Consideraciones de Rendimiento

1. **Cache Redis**: Para configuraciones de usuario frecuentemente accedidas
2. **Lazy Loading**: Cargar componentes de la barra bajo demanda
3. **Debouncing**: Para evitar múltiples requests en configuraciones rápidas
4. **Batch Operations**: Agrupar operaciones de migración de cookies
5. **CDN**: Para recursos estáticos de la barra vertical
