# Microservicio de Gestión de Calificaciones (Grade Management)

## Detalle del Microservicio
Este microservicio se encarga de la gestión de calificaciones de estudiantes, permitiendo registrar, consultar, actualizar y eliminar calificaciones. Está diseñado siguiendo una arquitectura hexagonal y utiliza programación reactiva con Spring WebFlux y MongoDB como base de datos.

**Característica especial**: El microservicio incluye funcionalidad transaccional que genera notificaciones automáticas cuando se crean o actualizan calificaciones. Los mensajes y campos de notificación están personalizados en español y muestran los nombres reales de estudiante y curso.

## Estructura del Proyecto
```
src/main/java/pe/edu/vallegrande/vg_ms_grade_management/
├── domain/
│   ├── model/
│   │   └── Grade.java                 // Entidad que representa una nota
│   │   └── DatabaseSequence.java      // Entidad para la secuencia de IDs
│   ├── enums/
│   │   └── ... (otros enums del dominio)
│   └── repository/
│       └── GradeRepository.java       // Interfaz para la persistencia de notas
├── application/
│   ├── service/
│   │   ├── GradeService.java          // Interfaz para los servicios de notas
│   │   ├── GradeNotificationService.java // Servicio para notificaciones transaccionales
│   │   └── SequenceGeneratorService.java // Servicio para generar secuencias de IDs
│   └── impl/
│       ├── GradeServiceImpl.java    // Implementación de los servicios de notas
│       └── GradeNotificationServiceImpl.java // Implementación de notificaciones transaccionales
├── infrastructure/
│   ├── config/
│   │   └── ... (configuraciones específicas de infraestructura)
│   ├── document/
│   │   └── GradeDocument.java         // Documento MongoDB para notas
│   ├── dto/
│   │   ├── request/
│   │   │   └── GradeRequest.java    // DTO para la creación/actualización de notas
│   │   └── response/
│   │       └── GradeResponse.java   // DTO para las respuestas de notas
│   ├── mapper/
│   │   └── GradeMapper.java           // Mapper entre Grade y GradeDocument
│   ├── repository/
│   │   └── GradeRepository.java       // Repositorio Spring Data MongoDB para notas
│   ├── rest/
│   │   └── GradeRest.java             // Controlador REST para las notas
│   └── service/
│       └── ApiService.java            // Servicio de API genérico
└── VgMsGradeManagementApplication.java  // Clase principal de la aplicación
```

## Funcionalidad Transaccional de Notificaciones

### Comportamiento Automático
Cuando se crea o actualiza una calificación, el sistema automáticamente genera notificaciones:

1. **Al crear una calificación** (`POST /api/grades`):
   - Genera notificación tipo `Calificación Publicada` para el estudiante (con su nombre real y el nombre real del curso)
   - Si la calificación es menor a 11, genera notificación `Bajo Rendimiento` para los padres

2. **Al actualizar una calificación** (`PUT /api/grades/{id}`):
   - Genera notificación tipo `Calificación Actualizada` para el estudiante
   - Si la calificación es menor a 11, genera notificación `Bajo Rendimiento` para los padres

### Tipos de Notificaciones Generadas
- **Calificación Publicada**: Cuando se publica una nueva calificación
- **Calificación Actualizada**: Cuando se actualiza una calificación existente
- **Bajo Rendimiento**: Cuando la calificación es menor a 11 puntos (para padres)

## Documentación de API

### Endpoints de Calificaciones (`/api/grades`)

| Método | Path                                      | Descripción                                   | JSON de Request (Ejemplo) | JSON de Response (Ejemplo) |
|--------|-------------------------------------------|-----------------------------------------------|---------------------------|----------------------------|
| GET    | `/`                                       | Obtiene todas las calificaciones              | N/A                       | `[ { "id": "1", "studentId": "4", "courseId": "C001", "grade": 20.0, "deleted": false } ]` |
| GET    | `/{id}`                                   | Obtiene una calificación por su ID            | N/A                       | `{ "id": "1", "studentId": "4", "courseId": "C001", "grade": 20.0, "deleted": false }` |
| GET    | `/student/{studentId}`                    | Obtiene calificaciones por ID de estudiante   | N/A                       | `[ { "id": "1", "studentId": "4", "courseId": "C001", "grade": 20.0, "deleted": false } ]` |
| GET    | `/course/{courseId}`                      | Obtiene calificaciones por ID de curso        | N/A                       | `[ { "id": "1", "studentId": "4", "courseId": "C001", "grade": 20.0, "deleted": false } ]` |
| GET    | `/student/{studentId}/course/{courseId}`  | Obtiene calificaciones por ID de estudiante y curso | N/A                       | `{ "id": "1", "studentId": "4", "courseId": "C001", "grade": 20.0, "deleted": false }` |
| **GET** | **`/{id}/notifications`**                 | **Obtiene notificaciones relacionadas con una calificación** | N/A | `[ { "id": "1", "recipientId": "4", "recipientType": "Amelia Flores Ascencio", "message": "Hola Amelia Flores Ascencio, tu calificación de Matemáticas ha sido publicada. Obtuviste 20.0/20 puntos.", "notificationType": "Calificación Publicada", "status": "Pendiente", "channel": "Correo" } ]` |
| POST   | `/`                                       | Crea una nueva calificación                   | `{ "studentId": "4", "courseId": "C001", "grade": 20.0 }` | `{ "id": "2", "studentId": "4", "courseId": "C001", "grade": 20.0, "deleted": false }` |
| PUT    | `/{id}`                                   | Actualiza una calificación existente          | `{ "studentId": "4", "courseId": "C001", "grade": 20.0 }` | `{ "id": "1", "studentId": "4", "courseId": "C001", "grade": 20.0, "deleted": false }` |
| DELETE | `/{id}`                                   | Elimina lógicamente una calificación          | N/A                       | `{ "id": "1", "studentId": "4", "courseId": "C001", "grade": 20.0, "deleted": true }` |
| PUT    | `/{id}/restore`                           | Restaura una calificación eliminada lógicamente | N/A                       | `{ "id": "1", "studentId": "4", "courseId": "C001", "grade": 20.0, "deleted": false }` |
| GET    | `/inactive`                               | Obtiene todas las calificaciones inactivas (eliminadas lógicamente) | N/A                       | `[ { "id": "3", "studentId": "4", "courseId": "C002", "grade": 8.5, "deleted": true } ]` |

## Ejemplos de JSON para Endpoints

### GradeRequest (POST /api/grades, PUT /api/grades/{id})
```json
{
  "studentId": "4",
  "courseId": "C001",
  "grade": 20.0,
  "academicPeriod": "Bimester",
  "evaluationType": "Examen",
  "evaluationDate": "2024-01-15",
  "remarks": "Excelente trabajo en el examen"
}
```

### GradeResponse (GET /api/grades, GET /api/grades/{id}, etc.)
```json
{
  "id": "654321",
  "studentId": "4",
  "courseId": "C001",
  "grade": 20.0,
  "academicPeriod": "Bimester",
  "evaluationType": "Examen",
  "evaluationDate": "2024-01-15",
  "remarks": "Excelente trabajo en el examen",
  "deleted": false
}
```

### Ejemplo de Respuesta para GET /api/grades/{id}/notifications (Notificaciones de Calificación)
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
    "sentAt": null
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
    "sentAt": null
  }
]
```

### Ejemplo de Respuesta para GET /api/grades (Lista de Calificaciones)
```json
[
  {
    "id": "654321",
    "studentId": "4",
    "courseId": "C001",
    "grade": 20.0,
    "academicPeriod": "Bimester",
    "evaluationType": "Examen",
    "evaluationDate": "2024-01-15",
    "remarks": "Excelente trabajo en el examen",
    "deleted": false
  },
  {
    "id": "654322",
    "studentId": "4",
    "courseId": "C002",
    "grade": 8.5,
    "academicPeriod": "Bimester",
    "evaluationType": "Examen",
    "evaluationDate": "2024-01-16",
    "remarks": "Necesita mejorar",
    "deleted": false
  }
]
```

### Ejemplo de Respuesta para GET /api/grades/{id} (Calificación Única)
```json
{
  "id": "654321",
  "studentId": "4",
  "courseId": "C001",
  "grade": 20.0,
  "academicPeriod": "Bimester",
  "evaluationType": "Examen",
  "evaluationDate": "2024-01-15",
  "remarks": "Excelente trabajo en el examen",
  "deleted": false
}
```

### Ejemplo de Respuesta para DELETE /api/grades/{id} (Eliminación Lógica)
```json
{
  "id": "654321",
  "studentId": "4",
  "courseId": "C001",
  "grade": 20.0,
  "academicPeriod": "Bimester",
  "evaluationType": "Examen",
  "evaluationDate": "2024-01-15",
  "remarks": "Excelente trabajo en el examen",
  "deleted": true
}
```

### Ejemplo de Respuesta para PUT /api/grades/{id}/restore (Restauración)
```json
{
  "id": "654321",
  "studentId": "4",
  "courseId": "C001",
  "grade": 20.0,
  "academicPeriod": "Bimester",
  "evaluationType": "Examen",
  "evaluationDate": "2024-01-15",
  "remarks": "Excelente trabajo en el examen",
  "deleted": false
}
```

## Casos de Uso con Notificaciones Automáticas

### 1. Crear Calificación con Notificación Automática
**POST** `/api/grades`

```json
{
  "studentId": "4",
  "courseId": "C001",
  "grade": 20.0,
  "academicPeriod": "Bimester",
  "evaluationType": "Examen",
  "evaluationDate": "2024-01-15",
  "remarks": "Excelente trabajo"
}
```

**Resultado**: Se crea la calificación y automáticamente se genera una notificación `Calificación Publicada` para el estudiante con su nombre real y el nombre real del curso.

### 2. Crear Calificación con Bajo Rendimiento
**POST** `/api/grades`

```json
{
  "studentId": "4",
  "courseId": "C002",
  "grade": 8.5,
  "academicPeriod": "Bimester",
  "evaluationType": "Examen",
  "evaluationDate": "2024-01-16",
  "remarks": "Necesita mejorar"
}
```

**Resultado**: Se crea la calificación y automáticamente se generan:
- Notificación `Calificación Publicada` para el estudiante
- Notificación `Bajo Rendimiento` para los padres

### 3. Actualizar Calificación
**PUT** `/api/grades/654321`

```json
{
  "studentId": "4",
  "courseId": "C001",
  "grade": 20.0,
  "academicPeriod": "Bimester",
  "evaluationType": "Examen",
  "evaluationDate": "2024-01-15",
  "remarks": "Calificación corregida"
}
```

**Resultado**: Se actualiza la calificación y automáticamente se genera una notificación `Calificación Actualizada` para el estudiante.

### 4. Consultar Notificaciones de una Calificación
**GET** `/api/grades/654321/notifications`

**Resultado**: Retorna todas las notificaciones relacionadas con esa calificación específica, mostrando los nombres reales y los mensajes en español.

## Notas Técnicas
- Los mensajes y campos de notificación están personalizados en español.
- El campo `recipientType` muestra el nombre real del estudiante (excepto para padres, que sigue siendo "PARENT").
- El campo `message` incluye el nombre real del estudiante y del curso.
- Los valores de `notificationType`, `status` y `channel` están en español.
