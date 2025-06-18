# Flowchart | Logical Functionality

## Flujo Lógico del Sistema de Notificaciones, Compartir y Favoritos

```mermaid
flowchart TD
    A[Usuario accede al sistema] --> B{¿Está autenticado?}
    B -->|No| C[Redirigir a login]
    B -->|Sí| D[Cargar página solicitada]

    D --> E{¿Qué tipo de página?}
    E -->|Currency/Actions/Options/Cryptos/Metals| F[Cargar datos de mercado]
    E -->|News| G[Cargar noticias]
    E -->|Economic Calendar| H[Cargar eventos calendario]
    E -->|Market Square| I[Cargar proyectos]
    E -->|Otros| J[Cargar datos específicos]

    F --> K[Mostrar datos con precios promedio]
    G --> L[Mostrar noticias por categoría]
    H --> M[Mostrar eventos por país/fecha]
    I --> N[Mostrar lista de proyectos]
    J --> O[Mostrar contenido específico]

    K --> P[Habilitar módulos]
    L --> P
    M --> P
    N --> P
    O --> P

    P --> Q{¿Módulo de Notificaciones?}
    Q -->|Sí| R[Configurar notificaciones]
    R --> R1[Definir frecuencia 3h/24h/96h/30d/120d/365d]
    R1 --> R2[Configurar variables a monitorear]
    R2 --> R3[Establecer umbrales de notificación]
    R3 --> R4[Activar servicio de background]

    P --> S{¿Módulo de Compartir?}
    S -->|Sí| T[Mostrar botones de compartir]
    T --> T1[Facebook/X/LinkedIn/Email/WhatsApp]
    T1 --> T2{¿Usuario hace clic?}
    T2 -->|Sí| T3[Generar URL personalizada]
    T3 --> T4[Abrir plataforma de destino]
    T4 --> T5[Registrar evento de compartir]

    P --> U{¿Módulo de Favoritos?}
    U -->|Sí| V[Mostrar botón de favoritos]
    V --> V1{¿Usuario añade a favoritos?}
    V1 -->|Sí| V2[Guardar en página de cuenta dedicada]
    V2 --> V3[Generar enlace de acceso rápido]

    R4 --> W[Proceso de background ejecutándose]
    W --> X[Monitorear cambios según frecuencia]
    X --> Y{¿Hay cambios significativos?}
    Y -->|No| X
    Y -->|Sí| Z[Generar notificación]
    Z --> AA[Enviar al usuario]
    AA --> AB[Registrar en historial]
    AB --> X

    T5 --> AC[Fin del proceso de compartir]
    V3 --> AD[Fin del proceso de favoritos]

    AC --> AE[Usuario continúa navegando]
    AD --> AE
    AB --> AE
    AE --> D

    style A fill:#e1f5fe
    style W fill:#fff3e0
    style R fill:#f3e5f5
    style T fill:#e8f5e8
    style V fill:#fce4ec
```

### Descripción del Flujo

**Flujo Principal:**

1. **Autenticación**: Verificación de usuario
2. **Carga de Página**: Determinación del tipo de contenido
3. **Carga de Datos**: Obtención de información específica según el tipo
4. **Habilitación de Módulos**: Activación de funcionalidades

**Módulo de Notificaciones:**

-   Configuración de frecuencias (3h, 24h, 96h, 30d, 120d, 365d)
-   Monitoreo continuo en background
-   Generación automática de notificaciones

**Módulo de Compartir:**

-   Soporte para múltiples plataformas
-   Tracking de clicks y engagement
-   Generación de URLs personalizadas

**Módulo de Favoritos:**

-   Gestión en página de cuenta dedicada
-   Acceso rápido a contenido favorito
-   Persistencia de preferencias de usuario
