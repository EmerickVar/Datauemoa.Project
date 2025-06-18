# State Diagram | Functional State of Modules

## Estados Funcionales de los Módulos del Sistema

```mermaid
stateDiagram-v2
    [*] --> SystemInitialization

    state SystemInitialization {
        [*] --> LoadingApplication
        LoadingApplication --> AuthenticationCheck
        AuthenticationCheck --> UserAuthenticated : Valid credentials
        AuthenticationCheck --> LoginRequired : Invalid/Missing credentials
        LoginRequired --> AuthenticationCheck : User logs in
        UserAuthenticated --> ModulesReady
        ModulesReady --> [*]
    }

    SystemInitialization --> ModuleSelection

    state ModuleSelection {
        [*] --> SelectingModule
        SelectingModule --> NotificationModule : User wants notifications
        SelectingModule --> SharingModule : User wants to share
        SelectingModule --> FavoritesModule : User wants favorites
        SelectingModule --> [*] : No action
    }

    state NotificationModule {
        [*] --> NotificationIdle
        NotificationIdle --> ConfiguringNotifications : User configures
        ConfiguringNotifications --> SettingFrequency
        SettingFrequency --> SettingVariables
        SettingVariables --> ActivatingMonitoring
        ActivatingMonitoring --> NotificationActive

        NotificationActive --> MonitoringData : Background process
        MonitoringData --> CheckingThresholds
        CheckingThresholds --> NoChanges : Below threshold
        CheckingThresholds --> SignificantChange : Above threshold

        NoChanges --> MonitoringData : Continue monitoring
        SignificantChange --> GeneratingNotification
        GeneratingNotification --> SendingNotification
        SendingNotification --> NotificationSent
        NotificationSent --> MonitoringData : Continue monitoring

        NotificationActive --> DisablingNotifications : User disables
        DisablingNotifications --> NotificationIdle

        note right of MonitoringData
            Frequencies:
            - 3 hours
            - 24 hours
            - 96 hours
            - 30 days
            - 120 days
            - 365 days
        end note
    }

    state SharingModule {
        [*] --> SharingIdle
        SharingIdle --> SelectingPlatform : User clicks share
        SelectingPlatform --> FacebookSharing : Facebook selected
        SelectingPlatform --> XSharing : X selected
        SelectingPlatform --> LinkedInSharing : LinkedIn selected
        SelectingPlatform --> EmailSharing : Email selected
        SelectingPlatform --> WhatsAppSharing : WhatsApp selected

        FacebookSharing --> GeneratingShareURL
        XSharing --> GeneratingShareURL
        LinkedInSharing --> GeneratingShareURL
        EmailSharing --> GeneratingShareURL
        WhatsAppSharing --> GeneratingShareURL

        GeneratingShareURL --> OpeningExternalPlatform
        OpeningExternalPlatform --> TrackingClick
        TrackingClick --> ShareCompleted
        ShareCompleted --> SharingIdle

        SharingIdle --> SharingError : Platform error
        SharingError --> SharingIdle : Error handled

        note right of GeneratingShareURL
            Content includes:
            - Page Name
            - Page Link
            - Image
            - Subpage Name (if applicable)
        end note
    }

    state FavoritesModule {
        [*] --> FavoritesIdle
        FavoritesIdle --> AddingToFavorites : User adds favorite
        FavoritesIdle --> ViewingFavorites : User views favorites page
        FavoritesIdle --> RemovingFromFavorites : User removes favorite

        AddingToFavorites --> ValidatingFavorite
        ValidatingFavorite --> FavoriteExists : Already favorited
        ValidatingFavorite --> SavingFavorite : New favorite

        FavoriteExists --> FavoritesIdle : No action needed
        SavingFavorite --> FavoriteAdded
        FavoriteAdded --> UpdatingUserAccount
        UpdatingUserAccount --> FavoritesIdle

        ViewingFavorites --> LoadingFavoritesList
        LoadingFavoritesList --> DisplayingFavorites
        DisplayingFavorites --> FavoritesIdle

        RemovingFromFavorites --> DeletingFavorite
        DeletingFavorite --> FavoriteRemoved
        FavoriteRemoved --> UpdatingUserAccount

        note right of SavingFavorite
            Stored data:
            - Page Name
            - Page Link
            - Image URL
            - Timestamp
        end note
    }

    state DataSynchronization {
        [*] --> SyncIdle
        SyncIdle --> SyncingMarketData : Market data update
        SyncIdle --> SyncingNewsData : News update
        SyncIdle --> SyncingCalendarData : Calendar update

        SyncingMarketData --> DataUpdated
        SyncingNewsData --> DataUpdated
        SyncingCalendarData --> DataUpdated

        DataUpdated --> TriggeringNotifications : If notifications active
        DataUpdated --> SyncIdle : No notifications

        TriggeringNotifications --> SyncIdle

        note right of DataUpdated
            Tables updated:
            - Market second + Market minute
            - Market Day
            - News
            - Calendar + Twig function
        end note
    }

    ModuleSelection --> NotificationModule
    ModuleSelection --> SharingModule
    ModuleSelection --> FavoritesModule

    NotificationModule --> DataSynchronization : Monitors data
    SharingModule --> ModuleSelection : Action completed
    FavoritesModule --> ModuleSelection : Action completed
    DataSynchronization --> NotificationModule : Data changed

    state ErrorHandling {
        [*] --> ErrorDetected
        ErrorDetected --> DatabaseError : DB connection issue
        ErrorDetected --> ExternalServiceError : Platform unavailable
        ErrorDetected --> ValidationError : Invalid data

        DatabaseError --> RetryConnection
        ExternalServiceError --> RetryService
        ValidationError --> ShowErrorMessage

        RetryConnection --> ErrorResolved : Success
        RetryConnection --> PersistentError : Multiple failures

        RetryService --> ErrorResolved : Success
        RetryService --> PersistentError : Multiple failures

        ShowErrorMessage --> ErrorResolved : User corrects

        ErrorResolved --> [*]
        PersistentError --> MaintenanceMode
        MaintenanceMode --> [*]
    }

    NotificationModule --> ErrorHandling : Error occurs
    SharingModule --> ErrorHandling : Error occurs
    FavoritesModule --> ErrorHandling : Error occurs
    ErrorHandling --> ModuleSelection : Error resolved
```

### Descripción de Estados

**Estados del Sistema:**

-   **SystemInitialization**: Carga inicial y autenticación
-   **ModuleSelection**: Selección de funcionalidad por el usuario
-   **DataSynchronization**: Sincronización de datos en background
-   **ErrorHandling**: Manejo de errores y recuperación

**Estados de Módulos:**

1. **NotificationModule**: Gestión de notificaciones con monitoreo continuo
2. **SharingModule**: Proceso de compartir en plataformas sociales
3. **FavoritesModule**: Administración de contenido favorito

**Transiciones Clave:**

-   Activación/desactivación de módulos por el usuario
-   Monitoreo automático de datos para notificaciones
-   Manejo de errores con reintentos automáticos
-   Sincronización continua de datos de mercado, noticias y calendario
