# Diagramme de Séquence | Flux d'Interaction

## Flux d'Interactions du Système de Notifications, Partage et Favoris

```mermaid
sequenceDiagram
    participant U as Utilisateur
    participant WEB as Application Web
    participant AUTH as Service d'Authentification
    participant DB as Base de Données
    participant NOTIF as Service de Notifications
    participant SHARE as Service de Partage
    participant FAV as Service de Favoris
    participant EXT as Plateformes Externes
    participant BG as Processus Arrière-plan

    Note over U,BG: Flux d'Authentification et Chargement Initial
    U->>WEB: Accède à la page (Currency/Actions/Options/etc.)
    WEB->>AUTH: Vérifier l'authentification
    AUTH-->>WEB: Utilisateur authentifié
    WEB->>DB: Charger les données de la page
    DB-->>WEB: Données de marché/actualités/calendrier
    WEB-->>U: Afficher la page avec modules activés

    Note over U,BG: Configuration des Notifications
    U->>WEB: Configurer les notifications
    WEB->>NOTIF: Établir les paramètres
    Note right of NOTIF: Fréquence : 3h/24h/96h/30j/120j/365j<br/>Variables : prix, changements, actualités
    NOTIF->>DB: Sauvegarder la configuration utilisateur
    NOTIF->>BG: Initialiser la surveillance arrière-plan
    NOTIF-->>WEB: Configuration confirmée
    WEB-->>U: Notifications activées

    Note over U,BG: Processus Arrière-plan pour Notifications
    loop Chaque intervalle configuré
        BG->>DB: Vérifier les changements dans les données
        alt Il y a des changements significatifs
            BG->>NOTIF: Générer une notification
            NOTIF->>DB: Enregistrer la notification
            NOTIF->>U: Envoyer la notification
            Note right of U: Email, push, notification in-app
        end
    end

    Note over U,BG: Flux de Partage de Contenu
    U->>WEB: Clic sur le bouton partager
    WEB->>SHARE: Demander le partage sur la plateforme
    Note right of SHARE: Facebook/X/LinkedIn/Email/WhatsApp
    SHARE->>DB: Enregistrer l'événement de partage
    SHARE->>EXT: Générer l'URL et ouvrir la plateforme
    Note right of EXT: Nom de la page + Lien + Image
    EXT-->>SHARE: Confirmation d'ouverture
    SHARE->>DB: Mettre à jour le compteur de clics
    SHARE-->>WEB: Processus terminé
    WEB-->>U: Contenu partagé avec succès

    Note over U,BG: Gestion des Favoris
    U->>WEB: Ajouter/supprimer un favori
    WEB->>FAV: Traiter la demande
    FAV->>DB: Mettre à jour les favoris de l'utilisateur
    Note right of DB: Page dédiée de compte<br/>Nom de la page + Lien + Image
    FAV-->>WEB: Favori mis à jour
    WEB-->>U: Changement confirmé

    Note over U,BG: Accès à la Page de Favoris
    U->>WEB: Accéder à la page de favoris
    WEB->>FAV: Charger les favoris de l'utilisateur
    FAV->>DB: Consulter les favoris sauvegardés
    DB-->>FAV: Liste des favoris
    FAV-->>WEB: Données des favoris
    WEB-->>U: Afficher la page de favoris

    Note over U,BG: Suivi des Interactions
    par Suivi parallèle
        U->>WEB: Interaction avec favori
        and
        U->>WEB: Clic sur lien partagé
        and
        U->>WEB: Réponse à notification
    end
    WEB->>DB: Enregistrer toutes les interactions
    DB-->>WEB: Confirmation d'enregistrement

    Note over U,BG: Cas d'Erreur et Récupération
    U->>WEB: Action qui échoue
    WEB->>DB: Tentative d'opération
    DB-->>WEB: Erreur de connexion
    WEB->>WEB: Tenter la reconnexion
    alt Reconnexion réussie
        WEB->>DB: Réessayer l'opération
        DB-->>WEB: Opération réussie
        WEB-->>U: Action terminée
    else Erreur persistante
        WEB-->>U: Afficher message d'erreur
        Note right of U: "Veuillez réessayer plus tard"
    end
```

### Description des Interactions

**Acteurs Principaux :**

-   **Utilisateur** : Interagit avec l'application web
-   **Application Web** : Frontend qui gère les demandes
-   **Services** : Backend spécialisé par fonctionnalité
-   **Base de Données** : Stockage persistant
-   **Processus Arrière-plan** : Surveillance automatique
-   **Plateformes Externes** : Réseaux sociaux et services tiers

**Flux Clés :**

1. **Authentification et Chargement** : Vérification de l'utilisateur et chargement des données
2. **Notifications** : Configuration et envoi automatique
3. **Partage** : Intégration avec les plateformes sociales
4. **Favoris** : Gestion du contenu préféré
5. **Suivi** : Suivi des interactions
6. **Gestion des Erreurs** : Récupération en cas de défaillances
