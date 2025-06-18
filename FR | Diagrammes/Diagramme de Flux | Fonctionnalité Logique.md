# Diagramme de Flux | Fonctionnalité Logique

## Flux Logique du Système de Notifications, Partage et Favoris

```mermaid
flowchart TD
    A[L'utilisateur accède au système] --> B{Est-il authentifié ?}
    B -->|Non| C[Rediriger vers la connexion]
    B -->|Oui| D[Charger la page demandée]

    D --> E{Quel type de page ?}
    E -->|Currency/Actions/Options/Cryptos/Metals| F[Charger les données de marché]
    E -->|News| G[Charger les actualités]
    E -->|Economic Calendar| H[Charger les événements du calendrier]
    E -->|Market Square| I[Charger les projets]
    E -->|Autres| J[Charger les données spécifiques]

    F --> K[Afficher les données avec prix moyens]
    G --> L[Afficher les actualités par catégorie]
    H --> M[Afficher les événements par pays/date]
    I --> N[Afficher la liste des projets]
    J --> O[Afficher le contenu spécifique]

    K --> P[Activer les modules]
    L --> P
    M --> P
    N --> P
    O --> P

    P --> Q{Module de Notifications ?}
    Q -->|Oui| R[Configurer les notifications]
    R --> R1[Définir la fréquence 3h/24h/96h/30j/120j/365j]
    R1 --> R2[Configurer les variables à surveiller]
    R2 --> R3[Établir les seuils de notification]
    R3 --> R4[Activer le service en arrière-plan]

    P --> S{Module de Partage ?}
    S -->|Oui| T[Afficher les boutons de partage]
    T --> T1[Facebook/X/LinkedIn/Email/WhatsApp]
    T1 --> T2{L'utilisateur clique ?}
    T2 -->|Oui| T3[Générer une URL personnalisée]
    T3 --> T4[Ouvrir la plateforme de destination]
    T4 --> T5[Enregistrer l'événement de partage]

    P --> U{Module de Favoris ?}
    U -->|Oui| V[Afficher le bouton favoris]
    V --> V1{L'utilisateur ajoute aux favoris ?}
    V1 -->|Oui| V2[Sauvegarder dans la page de compte dédiée]
    V2 --> V3[Générer un lien d'accès rapide]

    R4 --> W[Processus en arrière-plan en cours d'exécution]
    W --> X[Surveiller les changements selon la fréquence]
    X --> Y{Y a-t-il des changements significatifs ?}
    Y -->|Non| X
    Y -->|Oui| Z[Générer une notification]
    Z --> AA[Envoyer à l'utilisateur]
    AA --> AB[Enregistrer dans l'historique]
    AB --> X

    T5 --> AC[Fin du processus de partage]
    V3 --> AD[Fin du processus de favoris]

    AC --> AE[L'utilisateur continue la navigation]
    AD --> AE
    AB --> AE
    AE --> D

    style A fill:#e1f5fe
    style W fill:#fff3e0
    style R fill:#f3e5f5
    style T fill:#e8f5e8
    style V fill:#fce4ec
```

### Description du Flux

**Flux Principal :**

1. **Authentification** : Vérification de l'utilisateur
2. **Chargement de Page** : Détermination du type de contenu
3. **Chargement de Données** : Obtention d'informations spécifiques selon le type
4. **Activation des Modules** : Activation des fonctionnalités

**Module de Notifications :**

-   Configuration des fréquences (3h, 24h, 96h, 30j, 120j, 365j)
-   Surveillance continue en arrière-plan
-   Génération automatique de notifications

**Module de Partage :**

-   Support pour multiples plateformes
-   Suivi des clics et engagement
-   Génération d'URLs personnalisées

**Module de Favoris :**

-   Gestion dans une page de compte dédiée
-   Accès rapide au contenu favori
-   Persistance des préférences utilisateur
