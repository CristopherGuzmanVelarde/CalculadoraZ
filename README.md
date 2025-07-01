# Microservicio de Gestión de Notificaciones (Notification Management)

## Detalle del Microservicio
Este microservicio se encarga de la gestión de notificaciones del sistema educativo, permitiendo crear, consultar, actualizar, eliminar lógicamente y restaurar notificaciones para estudiantes, profesores y padres. Está diseñado siguiendo una arquitectura hexagonal y utiliza programación reactiva con Spring WebFlux y MongoDB como base de datos.

## Estructura del Proyecto
```
src/main/java/pe/edu/vallegrande/vg_ms_grade_management/
├── domain/
│   ├── model/
│   │   └── Notification.java              // Entidad que representa una notificación
│   └── enums/
│       └── ... (otros enums del dominio)
├── application/
│   ├── service/
│   │   └── NotificationService.java       // Interfaz para los servicios de notificaciones
│   └── impl/
│       └── NotificationServiceImpl.java   // Implementación de los servicios de notificaciones
├── infrastructure/
│   ├── config/
│   │   └── ... (configuraciones específicas de infraestructura)
│   ├── document/
│   │   └── NotificationDocument.java      // Documento MongoDB para notificaciones
│   ├── dto/
│   │   ├── request/
│   │   │   └── NotificationRequest.java   // DTO para la creación/actualización de notificaciones
│   │   └── response/
│   │       └── NotificationResponse.java  // DTO para las respuestas de notificaciones
│   ├── mapper/
│   │   └── NotificationMapper.java        // Mapper entre Notification y NotificationDocument
│   ├── repository/
│   │   └── NotificationRepository.java    // Repositorio Spring Data MongoDB para notificaciones
│   └── rest/
│       └── NotificationRest.java          // Controlador REST para las notificaciones
└── VgMsGradeManagementApplication.java    // Clase principal de la aplicación
```

## Cambios recientes
- Los valores de los campos `recipientType`, `notificationType`, `status` y `channel` ahora están en español.
- Los mensajes y el campo `recipientType` muestran el nombre real del estudiante y el curso.
- **Nuevo:** Ahora el eliminado de notificaciones es lógico (campo `deleted`). Puedes restaurar notificaciones eliminadas.

## Documentación de API

### Endpoints de Notificaciones (`/notifications`)

| Método | Path                | Descripción                                   | JSON de Request (Ejemplo) | JSON de Response (Ejemplo) |
|--------|---------------------|-----------------------------------------------|---------------------------|----------------------------|
| GET    | `/`                 | Obtiene todas las notificaciones no eliminadas| N/A                       | `[ { "id": "1", "recipientId": "4", "recipientType": "Amelia Flores Ascencio", "message": "Hola Amelia Flores Ascencio, tu calificación de Matemáticas ha sido publicada. Obtuviste 20.0/20 puntos.", "notificationType": "Calificación Publicada", "status": "Pendiente", "channel": "Correo", "createdAt": "2024-01-15T10:30:00", "sentAt": null, "deleted": false } ]` |
| GET    | `/{id}`             | Obtiene una notificación por su ID            | N/A                       | `{ "id": "1", "recipientId": "4", "recipientType": "Amelia Flores Ascencio", "message": "Hola Amelia Flores Ascencio, tu calificación de Matemáticas ha sido publicada. Obtuviste 20.0/20 puntos.", "notificationType": "Calificación Publicada", "status": "Pendiente", "channel": "Correo", "createdAt": "2024-01-15T10:30:00", "sentAt": null, "deleted": false }` |
| POST   | `/`                 | Crea una nueva notificación                   | `{ "recipientId": "4", "recipientType": "Amelia Flores Ascencio", "message": "Hola Amelia Flores Ascencio, tu calificación de Matemáticas ha sido publicada. Obtuviste 20.0/20 puntos.", "notificationType": "Calificación Publicada", "status": "Pendiente", "channel": "Correo" }` | `{ "id": "2", "recipientId": "4", "recipientType": "Amelia Flores Ascencio", "message": "Hola Amelia Flores Ascencio, tu calificación de Matemáticas ha sido publicada. Obtuviste 20.0/20 puntos.", "notificationType": "Calificación Publicada", "status": "Pendiente", "channel": "Correo", "createdAt": "2024-01-15T11:00:00", "sentAt": null, "deleted": false }` |
| PUT    | `/{id}`             | Actualiza una notificación existente          | `{ "recipientId": "4", "recipientType": "Amelia Flores Ascencio", "message": "Hola Amelia Flores Ascencio, tu calificación de Matemáticas ha sido actualizada. Nueva calificación: 20.0/20 puntos.", "notificationType": "Calificación Actualizada", "status": "Pendiente", "channel": "Correo" }` | `{ "id": "1", "recipientId": "4", "recipientType": "Amelia Flores Ascencio", "message": "Hola Amelia Flores Ascencio, tu calificación de Matemáticas ha sido actualizada. Nueva calificación: 20.0/20 puntos.", "notificationType": "Calificación Actualizada", "status": "Pendiente", "channel": "Correo", "createdAt": "2024-01-15T10:30:00", "sentAt": null, "deleted": false }` |
| DELETE | `/{id}`             | Elimina lógicamente una notificación          | N/A                       | `{ "id": "1", "recipientId": "4", "recipientType": "Amelia Flores Ascencio", "message": "Hola Amelia Flores Ascencio, tu calificación de Matemáticas ha sido publicada. Obtuviste 20.0/20 puntos.", "notificationType": "Calificación Publicada", "status": "Pendiente", "channel": "Correo", "createdAt": "2024-01-15T10:30:00", "sentAt": null, "deleted": true }` |
| PUT    | `/{id}/restore`     | Restaura una notificación eliminada lógicamente | N/A                     | `{ "id": "1", "recipientId": "4", "recipientType": "Amelia Flores Ascencio", "message": "Hola Amelia Flores Ascencio, tu calificación de Matemáticas ha sido publicada. Obtuviste 20.0/20 puntos.", "notificationType": "Calificación Publicada", "status": "Pendiente", "channel": "Correo", "createdAt": "2024-01-15T10:30:00", "sentAt": null, "deleted": false }` |

## Tipos de Datos

### recipientType (Tipo de Destinatario)
- Nombre completo del estudiante (ejemplo: "Amelia Flores Ascencio")
- "PARENT" (para notificaciones de bajo rendimiento, hasta que se implemente el nombre real del padre)

### notificationType (Tipo de Notificación)
- `Calificación Publicada`
- `Calificación Actualizada`
- `Bajo Rendimiento`

### status (Estado)
- `Pendiente`
- `Enviada`
- `Fallida`

### channel (Canal)
- `Correo`
- `SMS`
- `Notificación`

### deleted (Eliminado Lógico)
- `false`: Notificación activa
- `true`: Notificación eliminada lógicamente

## Ejemplos de JSON para Endpoints

### NotificationRequest (POST /notifications, PUT /notifications/{id})
```json
{
  "recipientId": "4",
  "recipientType": "Amelia Flores Ascencio",
  "message": "Hola Amelia Flores Ascencio, tu calificación de Matemáticas ha sido publicada. Obtuviste 20.0/20 puntos.",
  "notificationType": "Calificación Publicada",
  "status": "Pendiente",
  "channel": "Correo"
}
```

### NotificationResponse (GET /notifications, GET /notifications/{id}, etc.)
```json
{
  "id": "65d2a7f1b3e9c8a7b6c5d4e3",
  "recipientId": "4",
  "recipientType": "Amelia Flores Ascencio",
  "message": "Hola Amelia Flores Ascencio, tu calificación de Matemáticas ha sido publicada. Obtuviste 20.0/20 puntos.",
  "notificationType": "Calificación Publicada",
  "status": "Pendiente",
  "channel": "Correo",
  "createdAt": "2024-01-15T10:30:00",
  "sentAt": null,
  "deleted": false
}
```

### Ejemplo de Respuesta para GET /notifications (Lista de Notificaciones)
```json
[
  {
    "id": "65d2a7f1b3e9c8a7b6c5d4e3",
    "recipientId": "4",
    "recipientType": "Amelia Flores Ascencio",
    "message": "Hola Amelia Flores Ascencio, tu calificación de Matemáticas ha sido publicada. Obtuviste 20.0/20 puntos.",
    "notificationType": "Calificación Publicada",
    "status": "Pendiente",
    "channel": "Correo",
    "createdAt": "2024-01-15T10:30:00",
    "sentAt": null,
    "deleted": false
  },
  {
    "id": "65d2a7f1b3e9c8a7b6c5d4e4",
    "recipientId": "4",
    "recipientType": "Amelia Flores Ascencio",
    "message": "Hola Amelia Flores Ascencio, tu calificación de Matemáticas ha sido actualizada. Nueva calificación: 20.0/20 puntos.",
    "notificationType": "Calificación Actualizada",
    "status": "Pendiente",
    "channel": "Correo",
    "createdAt": "2024-01-15T11:00:00",
    "sentAt": null,
    "deleted": false
  }
]
```

### Ejemplo de Respuesta para GET /notifications/{id} (Notificación Única)
```json
{
  "id": "65d2a7f1b3e9c8a7b6c5d4e3",
  "recipientId": "4",
  "recipientType": "Amelia Flores Ascencio",
  "message": "Hola Amelia Flores Ascencio, tu calificación de Matemáticas ha sido publicada. Obtuviste 20.0/20 puntos.",
  "notificationType": "Calificación Publicada",
  "status": "Pendiente",
  "channel": "Correo",
  "createdAt": "2024-01-15T10:30:00",
  "sentAt": null,
  "deleted": false
}
```

### Ejemplo de Respuesta para POST /notifications (Creación)
```json
{
  "id": "65d2a7f1b3e9c8a7b6c5d4e6",
  "recipientId": "4",
  "recipientType": "Amelia Flores Ascencio",
  "message": "Hola Amelia Flores Ascencio, tu calificación de Matemáticas ha sido publicada. Obtuviste 20.0/20 puntos.",
  "notificationType": "Calificación Publicada",
  "status": "Pendiente",
  "channel": "Correo",
  "createdAt": "2024-01-15T12:00:00",
  "sentAt": null,
  "deleted": false
}
```

### Ejemplo de Respuesta para PUT /notifications/{id} (Actualización)
```json
{
  "id": "65d2a7f1b3e9c8a7b6c5d4e6",
  "recipientId": "4",
  "recipientType": "Amelia Flores Ascencio",
  "message": "Hola Amelia Flores Ascencio, tu calificación de Matemáticas ha sido actualizada. Nueva calificación: 20.0/20 puntos.",
  "notificationType": "Calificación Actualizada",
  "status": "Pendiente",
  "channel": "Correo",
  "createdAt": "2024-01-15T12:00:00",
  "sentAt": null,
  "deleted": false
}
```

### Ejemplo de Respuesta para DELETE /notifications/{id} (Eliminación)
```http
HTTP/1.1 204 No Content
```

## Casos de Uso Comunes

### 1. Notificación de Calificación Publicada
```json
{
  "recipientId": "4",
  "recipientType": "Amelia Flores Ascencio",
  "message": "Hola Amelia Flores Ascencio, tu calificación de Matemáticas ha sido publicada. Obtuviste 20.0/20 puntos.",
  "notificationType": "Calificación Publicada",
  "status": "Pendiente",
  "channel": "Correo"
}
```

### 2. Notificación de Calificación Actualizada
```json
{
  "recipientId": "4",
  "recipientType": "Amelia Flores Ascencio",
  "message": "Hola Amelia Flores Ascencio, tu calificación de Matemáticas ha sido actualizada. Nueva calificación: 20.0/20 puntos.",
  "notificationType": "Calificación Actualizada",
  "status": "Pendiente",
  "channel": "Correo"
}
```

### 3. Notificación de Bajo Rendimiento para Padres
```json
{
  "recipientId": "P4",
  "recipientType": "PARENT",
  "message": "Su hijo/a ha obtenido una calificación baja (8.5/20) en Matemáticas. Por favor revise el progreso académico.",
  "notificationType": "Bajo Rendimiento",
  "status": "Pendiente",
  "channel": "SMS"
}
```

## Notas Técnicas
- Los mensajes y campos de notificación están personalizados en español.
- El campo `recipientType` muestra el nombre real del estudiante (excepto para padres, que sigue siendo "PARENT").
- El campo `message` incluye el nombre real del estudiante y del curso.
- Los valores de `notificationType`, `status` y `channel` están en español.
- El campo `deleted` permite implementar un eliminado lógico de notificaciones.
- Los métodos GET solo devuelven notificaciones activas (`deleted: false`).
- Puedes restaurar notificaciones eliminadas usando el endpoint correspondiente.

## Códigos de Estado HTTP

- `200 OK` - Operación exitosa
- `201 Created` - Recurso creado exitosamente
- `204 No Content` - Operación exitosa sin contenido de respuesta
- `400 Bad Request` - Solicitud malformada
- `404 Not Found` - Recurso no encontrado
- `500 Internal Server Error` - Error interno del servidor

## Notas Técnicas

- El microservicio utiliza **Spring WebFlux** para programación reactiva
- **MongoDB** como base de datos NoSQL
- **MapStruct** para mapeo entre entidades y documentos
- **Lombok** para reducir código boilerplate
- Las fechas se manejan en formato **ISO 8601** (LocalDateTime)
- El campo `createdAt` se establece automáticamente al crear una notificación
- El campo `sentAt` se actualiza cuando la notificación es enviada exitosamente 
