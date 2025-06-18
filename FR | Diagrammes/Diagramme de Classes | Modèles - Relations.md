# Diagramme de Classes | Modèles et Relations

## Système de Notifications, Partage et Favoris pour Symfony 6.4

```mermaid
classDiagram
    class User {
        +int id
        +string email
        +string username
        +DateTime created_at
        +DateTime updated_at
        +getFavorites()
        +getNotifications()
        +getSharedItems()
    }

    class Page {
        +int id
        +string name
        +string type
        +string subpage
        +string url
        +string image_url
        +DateTime created_at
        +DateTime updated_at
        +getNotifications()
        +getFavorites()
        +getShares()
    }

    class NotificationModule {
        +int id
        +string module_type
        +string label
        +string variables
        +string values
        +string frequency
        +string interval
        +bool is_active
        +DateTime created_at
        +sendNotification()
        +updateFrequency()
    }

    class Notification {
        +int id
        +int user_id
        +int page_id
        +int module_id
        +string title
        +string content
        +string type
        +bool is_read
        +DateTime sent_at
        +DateTime read_at
        +markAsRead()
        +getContent()
    }

    class SharingModule {
        +int id
        +string platform
        +string share_url
        +string share_content
        +int click_count
        +DateTime created_at
        +shareToFacebook()
        +shareToX()
        +shareToLinkedIn()
        +shareToEmail()
        +shareToWhatsApp()
    }

    class SharedItem {
        +int id
        +int user_id
        +int page_id
        +string platform
        +string shared_url
        +DateTime shared_at
        +trackClick()
    }

    class FavoritesModule {
        +int id
        +string page_name
        +string page_link
        +string image_url
        +DateTime created_at
        +addToFavorites()
        +removeFromFavorites()
    }

    class Favorite {
        +int id
        +int user_id
        +int page_id
        +DateTime added_at
        +bool is_active
        +remove()
    }

    class MarketData {
        +int id
        +string market_type
        +string symbol
        +decimal spot_price
        +decimal high_price
        +decimal low_price
        +decimal variance
        +DateTime updated_at
        +getAveragePrice()
        +getPriceChange()
    }

    class NewsData {
        +int id
        +string category
        +string title
        +string content
        +string author
        +DateTime published_at
        +string tags
        +getByCategory()
    }

    class CalendarData {
        +int id
        +string country
        +string event
        +DateTime event_date
        +string importance
        +string forecast
        +getUpcomingEvents()
    }

    User "1" *-- "0..*" Notification : reçoit
    User "1" *-- "0..*" SharedItem : crée
    User "1" *-- "0..*" Favorite : possède

    Page "1" *-- "0..*" Notification : génère
    Page "1" o-- "0..*" SharedItem : peut_être_partagée
    Page "1" o-- "0..*" Favorite : peut_être_favorite

    NotificationModule "1" *-- "0..*" Notification : crée
    SharingModule "1" *-- "0..*" SharedItem : gère
    FavoritesModule "1" *-- "0..*" Favorite : traite

    Page "1" --> "0..*" MarketData : affiche
    Page "1" --> "0..*" NewsData : montre
    Page "1" --> "0..*" CalendarData : présente

    NotificationModule "1" --> "0..*" MarketData : surveille
    NotificationModule "1" --> "0..*" NewsData : suit
    NotificationModule "1" --> "0..*" CalendarData : observe
```

### Description des Relations

-   **User** : Utilisateur du système qui peut recevoir des notifications, partager du contenu et marquer des favoris
-   **Page** : Pages du système (Currency, Actions, Options, Cryptos, Metals, News, etc.)
-   **NotificationModule** : Gère les notifications avec différentes fréquences et intervalles
-   **SharingModule** : Gère le partage sur les réseaux sociaux et autres plateformes
-   **FavoritesModule** : Administre le système de favoris de l'utilisateur
-   **MarketData/NewsData/CalendarData** : Données spécifiques qui alimentent les notifications
