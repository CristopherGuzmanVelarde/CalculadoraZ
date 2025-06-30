# Microservicio de Gestión de Calificaciones (Grade Management)

## Detalle del Microservicio
Este microservicio se encarga de la gestión de calificaciones de estudiantes, permitiendo registrar, consultar, actualizar y eliminar calificaciones. Está diseñado siguiendo una arquitectura hexagonal y utiliza programación reactiva con Spring WebFlux y MongoDB como base de datos.

**Característica especial**: El microservicio incluye funcionalidad transaccional que genera notificaciones automáticas cuando se crean o actualizan calificaciones.

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
   - Genera notificación tipo `GRADE_PUBLISHED` para el estudiante
   - Si la calificación es menor a 11, genera notificación `LOW_PERFORMANCE` para los padres

2. **Al actualizar una calificación** (`PUT /api/grades/{id}`):
   - Genera notificación tipo `GRADE_UPDATED` para el estudiante
   - Si la calificación es menor a 11, genera notificación `LOW_PERFORMANCE` para los padres

### Tipos de Notificaciones Generadas
- **GRADE_PUBLISHED**: Cuando se publica una nueva calificación
- **GRADE_UPDATED**: Cuando se actualiza una calificación existente
- **LOW_PERFORMANCE**: Cuando la calificación es menor a 11 puntos (para padres)

## Documentación de API

### Endpoints de Calificaciones (`/api/grades`)

| Método | Path                                      | Descripción                                   | JSON de Request (Ejemplo) | JSON de Response (Ejemplo) |
|--------|-------------------------------------------|-----------------------------------------------|---------------------------|----------------------------|
| GET    | `/`                                       | Obtiene todas las calificaciones              | N/A                       | `[ { "id": "1", "studentId": "S001", "courseId": "C001", "grade": 85.5, "deleted": false } ]` |
| GET    | `/{id}`                                   | Obtiene una calificación por su ID            | N/A                       | `{ "id": "1", "studentId": "S001", "courseId": "C001", "grade": 85.5, "deleted": false }` |
| GET    | `/student/{studentId}`                    | Obtiene calificaciones por ID de estudiante   | N/A                       | `[ { "id": "1", "studentId": "S001", "courseId": "C001", "grade": 85.5, "deleted": false } ]` |
| GET    | `/course/{courseId}`                      | Obtiene calificaciones por ID de curso        | N/A                       | `[ { "id": "1", "studentId": "S001", "courseId": "C001", "grade": 85.5, "deleted": false } ]` |
| GET    | `/student/{studentId}/course/{courseId}`  | Obtiene calificaciones por ID de estudiante y curso | N/A                       | `{ "id": "1", "studentId": "S001", "courseId": "C001", "grade": 85.5, "deleted": false }` |
| **GET** | **`/{id}/notifications`**                 | **Obtiene notificaciones relacionadas con una calificación** | N/A | `[ { "id": "1", "recipientId": "S001", "recipientType": "STUDENT", "message": "Tu calificación ha sido publicada", "notificationType": "GRADE_PUBLISHED", "status": "PENDING", "channel": "EMAIL" } ]` |
| POST   | `/`                                       | Crea una nueva calificación                   | `{ "studentId": "S001", "courseId": "C001", "grade": 90.0 }` | `{ "id": "2", "studentId": "S001", "courseId": "C001", "grade": 90.0, "deleted": false }` |
| PUT    | `/{id}`                                   | Actualiza una calificación existente          | `{ "studentId": "S001", "courseId": "C001", "grade": 92.0 }` | `{ "id": "1", "studentId": "S001", "courseId": "C001", "grade": 92.0, "deleted": false }` |
| DELETE | `/{id}`                                   | Elimina lógicamente una calificación          | N/A                       | `{ "id": "1", "studentId": "S001", "courseId": "C001", "grade": 85.5, "deleted": true }` |
| PUT    | `/{id}/restore`                           | Restaura una calificación eliminada lógicamente | N/A                       | `{ "id": "1", "studentId": "S001", "courseId": "C001", "grade": 85.5, "deleted": false }` |
| GET    | `/inactive`                               | Obtiene todas las calificaciones inactivas (eliminadas lógicamente) | N/A                       | `[ { "id": "3", "studentId": "S002", "courseId": "C002", "grade": 70.0, "deleted": true } ]` |

## Ejemplos de JSON para Endpoints

### GradeRequest (POST /api/grades, PUT /api/grades/{id})
```json
{
  "studentId": "S001",
  "courseId": "C001",
  "grade": 90.0,
  "academicPeriod": "Bimester",
  "evaluationType": "Exam",
  "evaluationDate": "2024-01-15",
  "remarks": "Excelente trabajo en el examen"
}
```

### GradeResponse (GET /api/grades, GET /api/grades/{id}, etc.)
```json
{
  "id": "654321",
  "studentId": "S001",
  "courseId": "C001",
  "grade": 90.0,
  "academicPeriod": "Bimester",
  "evaluationType": "Exam",
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
    "recipientId": "S001",
    "recipientType": "STUDENT",
    "message": "Tu calificación de Curso C001 ha sido publicada. Obtuviste 90.0/20 puntos.",
    "notificationType": "GRADE_PUBLISHED",
    "status": "PENDING",
    "channel": "EMAIL",
    "createdAt": "2024-01-15T10:30:00",
    "sentAt": null
  },
  {
    "id": "65d2a7f1b3e9c8a7b6c5d4e4",
    "recipientId": "PS001",
    "recipientType": "PARENT",
    "message": "Su hijo/a ha obtenido una calificación baja (8.5/20) en Curso C001. Por favor revise el progreso académico.",
    "notificationType": "LOW_PERFORMANCE",
    "status": "PENDING",
    "channel": "SMS",
    "createdAt": "2024-01-15T10:30:00",
    "sentAt": null
  }
]
```

### Ejemplo de Respuesta para GET /api/grades (Lista de Calificaciones)
```json
[
  {
    "id": "654321",
    "studentId": "S001",
    "courseId": "C001",
    "grade": 90.0,
    "academicPeriod": "Bimester",
    "evaluationType": "Exam",
    "evaluationDate": "2024-01-15",
    "remarks": "Excelente trabajo en el examen",
    "deleted": false
  },
  {
    "id": "654322",
    "studentId": "S002",
    "courseId": "C002",
    "grade": 85.5,
    "academicPeriod": "Bimester",
    "evaluationType": "Project",
    "evaluationDate": "2024-01-16",
    "remarks": "Buen trabajo en el proyecto",
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
  "grade": 90.0,
  "academicPeriod": "Bimester",
  "evaluationType": "Exam",
  "evaluationDate": "2024-01-15",
  "remarks": "Excelente trabajo en el examen",
  "deleted": false
}
```

### Ejemplo de Respuesta para DELETE /api/grades/{id} (Eliminación Lógica)
```json
{
  "id": "654321",
  "studentId": "S001",
  "courseId": "C001",
  "grade": 90.0,
  "academicPeriod": "Bimester",
  "evaluationType": "Exam",
  "evaluationDate": "2024-01-15",
  "remarks": "Excelente trabajo en el examen",
  "deleted": true
}
```

### Ejemplo de Respuesta para PUT /api/grades/{id}/restore (Restauración)
```json
{
  "id": "654321",
  "studentId": "S001",
  "courseId": "C001",
  "grade": 90.0,
  "academicPeriod": "Bimester",
  "evaluationType": "Exam",
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
  "studentId": "S001",
  "courseId": "C001",
  "grade": 18.5,
  "academicPeriod": "Bimester",
  "evaluationType": "Exam",
  "evaluationDate": "2024-01-15",
  "remarks": "Excelente trabajo"
}
```

**Resultado**: Se crea la calificación y automáticamente se genera una notificación `GRADE_PUBLISHED` para el estudiante.

### 2. Crear Calificación con Bajo Rendimiento
**POST** `/api/grades`

```json
{
  "studentId": "S002",
  "courseId": "C001",
  "grade": 8.5,
  "academicPeriod": "Bimester",
  "evaluationType": "Exam",
  "evaluationDate": "2024-01-15",
  "remarks": "Necesita mejorar"
}
```

**Resultado**: Se crea la calificación y automáticamente se generan:
- Notificación `GRADE_PUBLISHED` para el estudiante
- Notificación `LOW_PERFORMANCE` para los padres

### 3. Actualizar Calificación
**PUT** `/api/grades/654321`

```json
{
  "studentId": "S001",
  "courseId": "C001",
  "grade": 19.0,
  "academicPeriod": "Bimester",
  "evaluationType": "Exam",
  "evaluationDate": "2024-01-15",
  "remarks": "Calificación corregida"
}
```

**Resultado**: Se actualiza la calificación y automáticamente se genera una notificación `GRADE_UPDATED` para el estudiante.

### 4. Consultar Notificaciones de una Calificación
**GET** `/api/grades/654321/notifications`

**Resultado**: Retorna todas las notificaciones relacionadas con esa calificación específica.

### 5. Obtener Notas Inactivas

**GET** `/api/v1/grades/inactive`

```json
[
  {
    "id": "65d2a7f1b3e9c8a7b6c5d4e3",
    "studentId": "S001",
    "courseId": "C001",
    "grade": 12.5,
    "deleted": true
  },
  {
    "id": "65d2a7f1b3e9c8a7b6c5d4e4",
    "studentId": "S002",
    "courseId": "C002",
    "grade": 18.0,
    "deleted": true
  }
]
```

### 6. Reporte de Promedio de Notas por Estudiante

**GET** `/api/v1/reports/student/{studentId}/average-grade`

```json
{
  "studentId": "S001",
  "averageGrade": 15.5,
  "message": "This is a placeholder for student average grade report."
}
```

### 7. Reporte de Distribución de Notas por Curso

**GET** `/api/v1/reports/course/{courseId}/grade-distribution`

```json
{
  "courseId": "C001",
  "gradeDistribution": {
    "0-10": 10,
    "11-15": 25,
    "16-20": 15
  },
  "message": "This is a placeholder for course grade distribution report."
}
```

## Flujo Transaccional

### Al Crear una Calificación:
1. Se valida la calificación
2. Se guarda en la base de datos
3. Se genera notificación `GRADE_PUBLISHED` para el estudiante
4. Si la calificación < 11, se genera notificación `LOW_PERFORMANCE` para los padres
5. Se retorna la calificación creada

### Al Actualizar una Calificación:
1. Se valida la calificación
2. Se actualiza en la base de datos
3. Se genera notificación `GRADE_UPDATED` para el estudiante
4. Si la calificación < 11, se genera notificación `LOW_PERFORMANCE` para los padres
5. Se retorna la calificación actualizada

## Notas Técnicas

- **Transaccionalidad**: Las operaciones de calificaciones y notificaciones son transaccionales
- **Reactividad**: Todo el flujo utiliza programación reactiva con Project Reactor
- **Automatización**: Las notificaciones se generan automáticamente sin intervención manual
- **Flexibilidad**: Los mensajes se generan dinámicamente basados en los datos de la calificación
- **Canalización**: Diferentes tipos de notificaciones usan diferentes canales (EMAIL, SMS)
