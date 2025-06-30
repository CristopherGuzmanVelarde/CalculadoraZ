# Gestión de Reportes en vg-ms-grade-management

## Detalle del Microservicio
Este microservicio se encarga de la generación de reportes relacionados con las calificaciones de estudiantes y cursos. Permite obtener el promedio de calificaciones de un estudiante y la distribución de calificaciones para un curso específico. Está diseñado siguiendo una arquitectura hexagonal y utiliza programación reactiva con Spring WebFlux y MongoDB como base de datos.

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
│   │   ├── ReportService.java         // Interfaz para los servicios de reportes
│   │   └── SequenceGeneratorService.java // Servicio para generar secuencias de IDs
│   └── impl/
│       ├── GradeServiceImpl.java    // Implementación de los servicios de notas
│       └── ReportServiceImpl.java   // Implementación de los servicios de reportes
├── infrastructure/
│   ├── config/
│   │   └── ... (configuraciones específicas de infraestructura)
│   ├── document/
│   │   └── GradeDocument.java         // Documento MongoDB para notas
│   ├── dto/
│   │   ├── request/
│   │   │   └── GradeRequest.java    // DTO para la creación/actualización de notas
│   │   └── response/
│   │       ├── GradeResponse.java   // DTO para las respuestas de notas
│   │       ├── ReportAverageGradeResponse.java // DTO para el promedio de calificaciones
│   │       └── ReportGradeDistributionResponse.java // DTO para la distribución de calificaciones
│   ├── mapper/
│   │   └── GradeMapper.java           // Mapper entre Grade y GradeDocument
│   ├── repository/
│   │   └── GradeRepository.java       // Repositorio Spring Data MongoDB para notas
│   ├── rest/
│   │   ├── GradeRest.java             // Controlador REST para las notas
│   │   └── ReportRest.java            // Controlador REST para los reportes
│   └── service/
│       └── ApiService.java            // Servicio de API genérico
└── VgMsGradeManagementApplication.java  // Clase principal de la aplicación
```

## Documentación de API

### Endpoints de Reportes (`/api/v1/reports`)

| Método | Path                                      | Descripción                                                                 | JSON de Request (Ejemplo) | JSON de Response (Ejemplo) |
|--------|-------------------------------------------|-----------------------------------------------------------------------------|---------------------------|----------------------------|
| GET    | `/student/{studentId}/average-grade`      | Obtiene el promedio de calificaciones de un estudiante específico.          | N/A                       | `{ "studentId": 1, "averageGrade": 15.5 }` |
| GET    | `/course/{courseId}/grade-distribution`   | Obtiene la distribución de calificaciones para un curso específico.         | N/A                       | `{ "courseId": 101, "distribution": { "0-5": 2, "6-10": 5, "11-15": 10, "16-20": 8 } }` |

## Ejemplos de JSON para Endpoints

### ReportAverageGradeResponse (GET /api/v1/reports/student/{studentId}/average-grade)
```json
{
  "studentId": 1,
  "averageGrade": 15.5
}
```

### ReportGradeDistributionResponse (GET /api/v1/reports/course/{courseId}/grade-distribution)
```json
{
  "courseId": 101,
  "distribution": {
    "0-5": 2,
    "6-10": 5,
    "11-15": 10,
    "16-20": 8
  }
}
```
