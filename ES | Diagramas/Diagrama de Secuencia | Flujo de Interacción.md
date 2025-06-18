# Sequence Diagram | Interaction Flow

## Flujo de Interacciones del Sistema de Notificaciones, Compartir y Favoritos

```mermaid
sequenceDiagram
    participant U as Usuario
    participant WEB as Aplicación Web
    participant AUTH as Servicio de Autenticación
    participant DB as Base de Datos
    participant NOTIF as Servicio de Notificaciones
    participant SHARE as Servicio de Compartir
    participant FAV as Servicio de Favoritos
    participant EXT as Plataformas Externas
    participant BG as Procesos Background

    Note over U,BG: Flujo de Autenticación y Carga Inicial
    U->>WEB: Accede a página (Currency/Actions/Options/etc.)
    WEB->>AUTH: Verificar autenticación
    AUTH-->>WEB: Usuario autenticado
    WEB->>DB: Cargar datos de página
    DB-->>WEB: Datos de mercado/noticias/calendario
    WEB-->>U: Mostrar página con módulos habilitados

    Note over U,BG: Configuración de Notificaciones
    U->>WEB: Configurar notificaciones
    WEB->>NOTIF: Establecer parámetros
    Note right of NOTIF: Frecuencia: 3h/24h/96h/30d/120d/365d<br/>Variables: precios, cambios, noticias
    NOTIF->>DB: Guardar configuración de usuario
    NOTIF->>BG: Iniciar monitoreo background
    NOTIF-->>WEB: Configuración confirmada
    WEB-->>U: Notificaciones activadas

    Note over U,BG: Proceso de Background para Notificaciones
    loop Cada intervalo configurado
        BG->>DB: Verificar cambios en datos
        alt Hay cambios significativos
            BG->>NOTIF: Generar notificación
            NOTIF->>DB: Registrar notificación
            NOTIF->>U: Enviar notificación
            Note right of U: Email, push, in-app notification
        end
    end

    Note over U,BG: Flujo de Compartir Contenido
    U->>WEB: Clic en botón compartir
    WEB->>SHARE: Solicitar compartir en plataforma
    Note right of SHARE: Facebook/X/LinkedIn/Email/WhatsApp
    SHARE->>DB: Registrar evento de compartir
    SHARE->>EXT: Generar URL y abrir plataforma
    Note right of EXT: Página Name + Link + Image
    EXT-->>SHARE: Confirmación de apertura
    SHARE->>DB: Actualizar contador de clics
    SHARE-->>WEB: Proceso completado
    WEB-->>U: Contenido compartido exitosamente

    Note over U,BG: Gestión de Favoritos
    U->>WEB: Añadir/remover favorito
    WEB->>FAV: Procesar solicitud
    FAV->>DB: Actualizar favoritos de usuario
    Note right of DB: Página dedicada de cuenta<br/>Page Name + Link + Image
    FAV-->>WEB: Favorito actualizado
    WEB-->>U: Cambio confirmado

    Note over U,BG: Acceso a Página de Favoritos
    U->>WEB: Acceder a página de favoritos
    WEB->>FAV: Cargar favoritos del usuario
    FAV->>DB: Consultar favoritos guardados
    DB-->>FAV: Lista de favoritos
    FAV-->>WEB: Datos de favoritos
    WEB-->>U: Mostrar página de favoritos

    Note over U,BG: Tracking de Interacciones
    par Tracking paralelo
        U->>WEB: Interacción con favorito
        and
        U->>WEB: Clic en enlace compartido
        and
        U->>WEB: Respuesta a notificación
    end
    WEB->>DB: Registrar todas las interacciones
    DB-->>WEB: Confirmación de registro

    Note over U,BG: Casos de Error y Recuperación
    U->>WEB: Acción que falla
    WEB->>DB: Intento de operación
    DB-->>WEB: Error de conexión
    WEB->>WEB: Intentar reconexión
    alt Reconexión exitosa
        WEB->>DB: Reintentar operación
        DB-->>WEB: Operación exitosa
        WEB-->>U: Acción completada
    else Error persistente
        WEB-->>U: Mostrar mensaje de error
        Note right of U: "Por favor, intente más tarde"
    end
```

### Descripción de las Interacciones

**Actores Principales:**

-   **Usuario**: Interactúa con la aplicación web
-   **Aplicación Web**: Frontend que maneja las solicitudes
-   **Servicios**: Backend especializado por funcionalidad
-   **Base de Datos**: Almacenamiento persistente
-   **Procesos Background**: Monitoreo automático
-   **Plataformas Externas**: Redes sociales y servicios de terceros

**Flujos Clave:**

1. **Autenticación y Carga**: Verificación de usuario y carga de datos
2. **Notificaciones**: Configuración y envío automático
3. **Compartir**: Integración con plataformas sociales
4. **Favoritos**: Gestión de contenido preferido
5. **Tracking**: Seguimiento de interacciones
6. **Manejo de Errores**: Recuperación ante fallos
