# Architecture Technique | Barre Verticale

## Diagramme d'Architecture du Système de Barre Verticale

```mermaid
graph TB
    subgraph "Frontend - Interface Utilisateur"
        VB[Barre Verticale]
        VB --> VB1[Composant Notifications]
        VB --> VB2[Composant Partage]
        VB --> VB3[Composant Favoris]
        VB --> VB4[Panneau de Gestion]
        VB --> VB5[Invitation Inscription]
    end

    subgraph "Gestion d'État"
        SM[State Manager]
        SM --> SM1[État Utilisateur Connecté]
        SM --> SM2[État Utilisateur Invité]
        SM --> SM3[État Cookies]
        SM --> SM4[État Base de Données]
    end

    subgraph "Services Backend"
        NS[NotificationService]
        FS[FavoriteService]
        SS[SharingService]
        US[UserService]
        CS[CookieService]
        MS[MigrationService]
    end

    subgraph "Base de Données"
        DB[(Base MySQL)]
        DB --> T1[user_notification_configs]
        DB --> T2[user_favorites]
        DB --> T3[users]
        DB --> T4[pages]
        DB --> T5[notification_logs]
        DB --> T6[sharing_logs]
    end

    subgraph "Stockage Temporaire"
        COOKIES[Cookies Navigateur]
        COOKIES --> C1[notification_configs]
        COOKIES --> C2[user_favorites]
        COOKIES --> C3[session_data]
    end

    subgraph "APIs Externes"
        API[APIs Externes]
        API --> API1[API Facebook]
        API --> API2[API Twitter/X]
        API --> API3[API LinkedIn]
        API --> API4[Service Email]
        API --> API5[WhatsApp Business]
    end

    %% Connexions Frontend
    VB1 --> SM1
    VB1 --> SM2
    VB2 --> SS
    VB3 --> SM1
    VB3 --> SM2
    VB4 --> SM1
    VB5 --> US

    %% Connexions State Manager
    SM1 --> NS
    SM1 --> FS
    SM2 --> CS
    SM3 --> COOKIES
    SM4 --> DB

    %% Connexions Services
    NS --> DB
    FS --> DB
    SS --> API
    US --> DB
    CS --> COOKIES
    MS --> DB
    MS --> COOKIES

    %% Flux de Migration
    CS -.->|"Conversion lors inscription"| MS
    MS -.->|"Transférer données"| DB

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

## Flux de Données Détaillé

```mermaid
sequenceDiagram
    participant U as Utilisateur
    participant VB as Barre Verticale
    participant SM as State Manager
    participant CS as Cookie Service
    participant NS as Notification Service
    participant DB as Base de Données
    participant COOK as Cookies Navigateur

    Note over U,COOK: Scénario: Utilisateur Non Connecté

    U->>VB: Configure notification
    VB->>SM: Vérifier état utilisateur
    SM-->>VB: Utilisateur non connecté
    VB->>CS: Sauvegarder configuration
    CS->>COOK: Écrire cookie
    COOK-->>CS: Confirmation
    CS-->>VB: Configuration sauvegardée
    VB-->>U: Afficher confirmation + invitation inscription

    Note over U,COOK: Scénario: Utilisateur s'Inscrit

    U->>VB: Clic sur "S'inscrire"
    VB->>NS: Processus d'inscription
    NS->>DB: Créer utilisateur
    DB-->>NS: Utilisateur créé
    NS->>CS: Migrer cookies existants
    CS->>COOK: Lire configurations
    COOK-->>CS: Données cookies
    CS->>DB: Transférer vers tables BD
    DB-->>CS: Transfert terminé
    CS->>COOK: Nettoyer cookies
    COOK-->>CS: Cookies supprimés
    CS-->>VB: Migration terminée
    VB-->>U: Accès complet activé

    Note over U,COOK: Scénario: Utilisateur Connecté

    U->>VB: Configure notification
    VB->>SM: Vérifier état utilisateur
    SM-->>VB: Utilisateur connecté
    VB->>NS: Sauvegarder configuration
    NS->>DB: Insérer dans table
    DB-->>NS: Configuration sauvegardée
    NS-->>VB: Confirmation
    VB-->>U: Notification configurée
```

## Spécifications Techniques

### Composants Frontend (JavaScript/Twig)

#### Composant Barre Verticale

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
		// Rendre selon l'état de l'utilisateur
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

### Services Backend (Symfony 6.4)

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
            time() + (86400 * 30), // 30 jours
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

### Structure de Base de Données

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

### Considérations de Performance

1. **Cache Redis** : Pour les configurations utilisateur fréquemment accédées
2. **Lazy Loading** : Charger les composants de la barre à la demande
3. **Debouncing** : Pour éviter multiples requêtes dans configurations rapides
4. **Opérations par Lots** : Grouper les opérations de migration de cookies
5. **CDN** : Pour les ressources statiques de la barre verticale
