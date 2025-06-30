# Microservicio de Gestión de Calificaciones (Grade Management)

## Detalle del Microservicio
Este microservicio se encarga de la gestión de calificaciones de estudiantes, permitiendo registrar, consultar, actualizar y eliminar calificaciones. Está diseñado siguiendo una arquitectura hexagonal y utiliza programación reactiva con Spring WebFlux y MongoDB como base de datos.

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
│   │   └── SequenceGeneratorService.java // Servicio para generar secuencias de IDs
│   └── impl/
│       └── GradeServiceImpl.java    // Implementación de los servicios de notas
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

## Documentación de API

### Endpoints de Calificaciones (`/api/grades`)

| Método | Path                                      | Descripción                                   | JSON de Request (Ejemplo) | JSON de Response (Ejemplo) |
|--------|-------------------------------------------|-----------------------------------------------|---------------------------|----------------------------|
| GET    | `/`                                       | Obtiene todas las calificaciones              | N/A                       | `[ { "id": "1", "studentId": "S001", "courseId": "C001", "score": 85.5, "deleted": false } ]` |
| GET    | `/{id}`                                   | Obtiene una calificación por su ID            | N/A                       | `{ "id": "1", "studentId": "S001", "courseId": "C001", "score": 85.5, "deleted": false }` |
| GET    | `/student/{studentId}`                    | Obtiene calificaciones por ID de estudiante   | N/A                       | `[ { "id": "1", "studentId": "S001", "courseId": "C001", "score": 85.5, "deleted": false } ]` |
| GET    | `/course/{courseId}`                      | Obtiene calificaciones por ID de curso        | N/A                       | `[ { "id": "1", "studentId": "S001", "courseId": "C001", "score": 85.5, "deleted": false } ]` |
| GET    | `/student/{studentId}/course/{courseId}`  | Obtiene calificaciones por ID de estudiante y curso | N/A                       | `{ "id": "1", "studentId": "S001", "courseId": "C001", "score": 85.5, "deleted": false }` |
| POST   | `/`                                       | Crea una nueva calificación                   | `{ "studentId": "S001", "courseId": "C001", "score": 90.0 }` | `{ "id": "2", "studentId": "S001", "courseId": "C001", "score": 90.0, "deleted": false }` |
| PUT    | `/{id}`                                   | Actualiza una calificación existente          | `{ "studentId": "S001", "courseId": "C001", "score": 92.0 }` | `{ "id": "1", "studentId": "S001", "courseId": "C001", "score": 92.0, "deleted": false }` |
| DELETE | `/{id}`                                   | Elimina lógicamente una calificación          | N/A                       | `{ "id": "1", "studentId": "S001", "courseId": "C001", "score": 85.5, "deleted": true }` |
| PUT    | `/{id}/restore`                           | Restaura una calificación eliminada lógicamente | N/A                       | `{ "id": "1", "studentId": "S001", "courseId": "C001", "score": 85.5, "deleted": false }` |
| GET    | `/inactive`                               | Obtiene todas las calificaciones inactivas (eliminadas lógicamente) | N/A                       | `[ { "id": "3", "studentId": "S002", "courseId": "C002", "score": 70.0, "deleted": true } ]` |

## Ejemplos de JSON para Endpoints

### GradeRequest (POST /api/grades, PUT /api/grades/{id})
```json
{
  "studentId": "S001",
  "courseId": "C001",
  "score": 90.0
}
```

### GradeResponse (GET /api/grades, GET /api/grades/{id}, etc.)
```json
{
  "id": "654321",
  "studentId": "S001",
  "courseId": "C001",
  "score": 90.0,
  "deleted": false
}
```

### Ejemplo de Respuesta para GET /api/grades (Lista de Calificaciones)
```json
[
  {
    "id": "654321",
    "studentId": "S001",
    "courseId": "C001",
    "score": 90.0,
    "deleted": false
  },
  {
    "id": "654322",
    "studentId": "S002",
    "courseId": "C002",
    "score": 85.5,
    "deleted": false
  }
]
```

### Ejemplo de Respuesta para GET /api/grades/{id} (Calificación Única)
```json
{
  "id": "654321",
  "studentId": "S001",
  "courseId": "C001",
  "score": 90.0,
  "deleted": false
}
```

### Ejemplo de Respuesta para DELETE /api/grades/{id} (Eliminación Lógica)
```json
{
  "id": "654321",
  "studentId": "S001",
  "courseId": "C001",
  "score": 90.0,
  "deleted": true
}
```

### Ejemplo de Respuesta para PUT /api/grades/{id}/restore (Restauración)
```json
{
  "id": "654321",
  "studentId": "S001",
  "courseId": "C001",
  "score": 90.0,
  "deleted": false
}
```

### Ejemplo de Respuesta para GET /api/grades/inactive (Calificaciones Inactivas)
```json
[
  {
    "id": "654323",
    "studentId": "S003",
    "courseId": "C003",
    "score": 70.0,
    "deleted": true
  },
  {
    "id": "654324",
    "studentId": "S004",
    "courseId": "C004",
    "score": 65.0,
    "deleted": true
  }
]
```
