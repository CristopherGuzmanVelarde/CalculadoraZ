# Microservicio de Gestión de Notificaciones (Notification Management)

## Detalle del Microservicio
Este microservicio se encarga de la gestión de notificaciones del sistema educativo, permitiendo crear, consultar, actualizar y eliminar notificaciones para estudiantes, profesores y padres. Está diseñado siguiendo una arquitectura hexagonal y utiliza programación reactiva con Spring WebFlux y MongoDB como base de datos.

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

## Documentación de API

### Endpoints de Notificaciones (`/notifications`)

| Método | Path                | Descripción                                   | JSON de Request (Ejemplo) | JSON de Response (Ejemplo) |
|--------|---------------------|-----------------------------------------------|---------------------------|----------------------------|
| GET    | `/`                 | Obtiene todas las notificaciones              | N/A                       | `[ { "id": "1", "recipientId": "S001", "recipientType": "STUDENT", "message": "Tu calificación ha sido publicada", "notificationType": "GRADE_PUBLISHED", "status": "PENDING", "channel": "EMAIL", "createdAt": "2024-01-15T10:30:00", "sentAt": null } ]` |
| GET    | `/{id}`             | Obtiene una notificación por su ID            | N/A                       | `{ "id": "1", "recipientId": "S001", "recipientType": "STUDENT", "message": "Tu calificación ha sido publicada", "notificationType": "GRADE_PUBLISHED", "status": "SENT", "channel": "EMAIL", "createdAt": "2024-01-15T10:30:00", "sentAt": "2024-01-15T10:35:00" }` |
| POST   | `/`                 | Crea una nueva notificación                   | `{ "recipientId": "S001", "recipientType": "STUDENT", "message": "Tu calificación ha sido publicada", "notificationType": "GRADE_PUBLISHED", "status": "PENDING", "channel": "EMAIL" }` | `{ "id": "2", "recipientId": "S001", "recipientType": "STUDENT", "message": "Tu calificación ha sido publicada", "notificationType": "GRADE_PUBLISHED", "status": "PENDING", "channel": "EMAIL", "createdAt": "2024-01-15T11:00:00", "sentAt": null }` |
| PUT    | `/{id}`             | Actualiza una notificación existente          | `{ "recipientId": "S001", "recipientType": "STUDENT", "message": "Tu calificación ha sido actualizada", "notificationType": "GRADE_UPDATED", "status": "SENT", "channel": "SMS" }` | `{ "id": "1", "recipientId": "S001", "recipientType": "STUDENT", "message": "Tu calificación ha sido actualizada", "notificationType": "GRADE_UPDATED", "status": "SENT", "channel": "SMS", "createdAt": "2024-01-15T10:30:00", "sentAt": "2024-01-15T11:00:00" }` |
| DELETE | `/{id}`             | Elimina una notificación                      | N/A                       | `204 No Content` |

## Tipos de Datos

### RecipientType (Tipo de Destinatario)
- `STUDENT` - Estudiante
- `TEACHER` - Profesor
- `PARENT` - Padre/Madre

### NotificationType (Tipo de Notificación)
- `GRADE_PUBLISHED` - Calificación Publicada
- `GRADE_UPDATED` - Calificación Actualizada
- `LOW_PERFORMANCE` - Bajo Rendimiento

### Status (Estado)
- `PENDING` - Pendiente
- `SENT` - Enviada
- `FAILED` - Fallida

### Channel (Canal)
- `EMAIL` - Correo Electrónico
- `SMS` - Mensaje de Texto
- `PUSH` - Notificación Push

## Ejemplos de JSON para Endpoints

### NotificationRequest (POST /notifications, PUT /notifications/{id})
```json
{
  "recipientId": "S001",
  "recipientType": "STUDENT",
  "message": "Tu calificación de Matemáticas ha sido publicada. Obtuviste 18/20 puntos.",
  "notificationType": "GRADE_PUBLISHED",
  "status": "PENDING",
  "channel": "EMAIL"
}
```

### NotificationResponse (GET /notifications, GET /notifications/{id}, etc.)
```json
{
  "id": "65d2a7f1b3e9c8a7b6c5d4e3",
  "recipientId": "S001",
  "recipientType": "STUDENT",
  "message": "Tu calificación de Matemáticas ha sido publicada. Obtuviste 18/20 puntos.",
  "notificationType": "GRADE_PUBLISHED",
  "status": "SENT",
  "channel": "EMAIL",
  "createdAt": "2024-01-15T10:30:00",
  "sentAt": "2024-01-15T10:35:00"
}
```

### Ejemplo de Respuesta para GET /notifications (Lista de Notificaciones)
```json
[
  {
    "id": "65d2a7f1b3e9c8a7b6c5d4e3",
    "recipientId": "S001",
    "recipientType": "STUDENT",
    "message": "Tu calificación de Matemáticas ha sido publicada. Obtuviste 18/20 puntos.",
    "notificationType": "GRADE_PUBLISHED",
    "status": "SENT",
    "channel": "EMAIL",
    "createdAt": "2024-01-15T10:30:00",
    "sentAt": "2024-01-15T10:35:00"
  },
  {
    "id": "65d2a7f1b3e9c8a7b6c5d4e4",
    "recipientId": "T001",
    "recipientType": "TEACHER",
    "message": "Nueva calificación pendiente de revisión para el curso de Física.",
    "notificationType": "GRADE_PUBLISHED",
    "status": "PENDING",
    "channel": "PUSH",
    "createdAt": "2024-01-15T11:00:00",
    "sentAt": null
  },
  {
    "id": "65d2a7f1b3e9c8a7b6c5d4e5",
    "recipientId": "P001",
    "recipientType": "PARENT",
    "message": "Su hijo/a ha obtenido una calificación baja en Historia. Por favor revise el progreso académico.",
    "notificationType": "LOW_PERFORMANCE",
    "status": "SENT",
    "channel": "SMS",
    "createdAt": "2024-01-15T09:15:00",
    "sentAt": "2024-01-15T09:20:00"
  }
]
```

### Ejemplo de Respuesta para GET /notifications/{id} (Notificación Única)
```json
{
  "id": "65d2a7f1b3e9c8a7b6c5d4e3",
  "recipientId": "S001",
  "recipientType": "STUDENT",
  "message": "Tu calificación de Matemáticas ha sido publicada. Obtuviste 18/20 puntos.",
  "notificationType": "GRADE_PUBLISHED",
  "status": "SENT",
  "channel": "EMAIL",
  "createdAt": "2024-01-15T10:30:00",
  "sentAt": "2024-01-15T10:35:00"
}
```

### Ejemplo de Respuesta para POST /notifications (Creación)
```json
{
  "id": "65d2a7f1b3e9c8a7b6c5d4e6",
  "recipientId": "S002",
  "recipientType": "STUDENT",
  "message": "Tu calificación de Ciencias ha sido actualizada. Nueva calificación: 16/20 puntos.",
  "notificationType": "GRADE_UPDATED",
  "status": "PENDING",
  "channel": "EMAIL",
  "createdAt": "2024-01-15T12:00:00",
  "sentAt": null
}
```

### Ejemplo de Respuesta para PUT /notifications/{id} (Actualización)
```json
{
  "id": "65d2a7f1b3e9c8a7b6c5d4e6",
  "recipientId": "S002",
  "recipientType": "STUDENT",
  "message": "Tu calificación de Ciencias ha sido actualizada. Nueva calificación: 16/20 puntos.",
  "notificationType": "GRADE_UPDATED",
  "status": "SENT",
  "channel": "SMS",
  "createdAt": "2024-01-15T12:00:00",
  "sentAt": "2024-01-15T12:05:00"
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
  "recipientId": "S001",
  "recipientType": "STUDENT",
  "message": "Tu calificación de Matemáticas ha sido publicada. Obtuviste 18/20 puntos.",
  "notificationType": "GRADE_PUBLISHED",
  "status": "PENDING",
  "channel": "EMAIL"
}
```

### 2. Notificación de Bajo Rendimiento para Padres
```json
{
  "recipientId": "P001",
  "recipientType": "PARENT",
  "message": "Su hijo/a ha obtenido una calificación baja en Historia. Por favor revise el progreso académico.",
  "notificationType": "LOW_PERFORMANCE",
  "status": "PENDING",
  "channel": "SMS"
}
```

### 3. Notificación de Calificación Actualizada
```json
{
  "recipientId": "S002",
  "recipientType": "STUDENT",
  "message": "Tu calificación de Ciencias ha sido actualizada. Nueva calificación: 16/20 puntos.",
  "notificationType": "GRADE_UPDATED",
  "status": "PENDING",
  "channel": "PUSH"
}
```

### 4. Notificación para Profesores
```json
{
  "recipientId": "T001",
  "recipientType": "TEACHER",
  "message": "Nueva calificación pendiente de revisión para el curso de Física.",
  "notificationType": "GRADE_PUBLISHED",
  "status": "PENDING",
  "channel": "PUSH"
}
```

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
