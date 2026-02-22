# GYM Microservices — Documentación

Repositorio de documentación del proyecto **GYM Microservices**, un sistema de gestión de gimnasio construido con arquitectura de microservicios en Spring Boot. Aquí se centraliza toda la evidencia del proyecto: colección de pruebas, diagramas de arquitectura, video de demostración y presentación.

> **Repositorio de código fuente:** [Ing-Daron11/Microservices-GYM](https://github.com/Ing-Daron11/Microservices-GYM)

---

## Contenido del repositorio

```
docs/
├── postman/
│   └── GYM_Microservices.postman_collection.json
├── diagramas/
│   ├── arquitectura-general.png
│   ├── diagrama-clases.png
│   ├── flujo-integridad-referencial.png
│   └── diagrama-secuencia.png
├── video/
│   └── demo-microservices-gym.mp4
├── presentacion/
│   └── GYM_Microservices_Presentacion.pdf
└── README.md
```

---

## Descripción del proyecto

**GYM Microservices** es un sistema backend compuesto por cuatro microservicios independientes que gestionan las entidades principales de un gimnasio. Cada servicio tiene su propia base de datos en memoria (H2) y se comunica con los demás mediante HTTP a través de `RestTemplate`.

### Microservicios

| Microservicio            | Puerto | Responsabilidad                                         |
|--------------------------|--------|---------------------------------------------------------|
| `microservicio-miembros`     | `8081` | Gestión de miembros inscritos en el gimnasio            |
| `microservicio-clases`       | `8082` | Programación de clases y orquestación de relaciones     |
| `microservicio-entrenadores` | `8083` | Gestión de entrenadores y sus especialidades            |
| `microservicio-equipos`      | `8084` | Inventario de equipos y materiales del gimnasio         |

### Principios aplicados

- **Domain-Driven Design (DDD):** uso de Value Objects (`ClaseId`, `Horario`, `Email`, `Especialidad`, etc.) para encapsular conceptos del dominio.
- **Integridad referencial distribuida:** antes de eliminar cualquier entidad, el servicio correspondiente consulta al microservicio de clases para verificar que no existan referencias activas.
- **Validación de entrada:** uso de Bean Validation (`@NotBlank`, `@Positive`) para garantizar la consistencia de los datos.
- **Manejo centralizado de errores:** cada servicio cuenta con un `GlobalExceptionHandler` que traduce excepciones de dominio a respuestas HTTP apropiadas.

---

## Colección de Postman

**Archivo:** [`postman/GYM_Microservices.postman_collection.json`](postman/GYM Microservices.postman_collection.json)

La colección contiene **27 requests** organizados en 5 carpetas:

| Carpeta                                        | Requests | Descripción                                              |
|------------------------------------------------|----------|----------------------------------------------------------|
| Miembros (puerto 8081)                         | 6        | CRUD completo + verificación de existencia               |
| Entrenadores (puerto 8083)                     | 6        | CRUD completo + verificación de existencia               |
| Equipos (puerto 8084)                          | 6        | CRUD completo + verificación de existencia               |
| Clases (puerto 8082)                           | 11       | Programar, asignar/remover entidades, eliminar, verificar|
| 🔄 Flujo de Prueba - Eliminar con referencias  | 17       | Prueba completa del flujo de integridad referencial      |

### Cómo importar

1. Abrir **Postman**
2. Clic en **Import**
3. Seleccionar el archivo `GYM_Microservices.postman_collection.json`
4. La colección aparecerá lista con todas las variables configuradas

### Variables de colección

Las URLs base y los IDs se gestionan con variables. Un script de pre-request genera IDs únicos automáticamente en cada sesión:

| Variable          | Valor por defecto          |
|-------------------|----------------------------|
| `base_miembros`   | `http://localhost:8081`    |
| `base_clases`     | `http://localhost:8082`    |
| `base_entrenadores` | `http://localhost:8083`  |
| `base_equipos`    | `http://localhost:8084`    |
| `miembro_id`      | Generado automáticamente   |
| `entrenador_id`   | Generado automáticamente   |
| `equipo_id`       | Generado automáticamente   |
| `clase_id`        | Generado automáticamente   |

### Flujo de prueba de integridad referencial

La carpeta **"Flujo de Prueba"** demuestra el comportamiento del sistema ante eliminaciones con referencias activas:

```
Pasos 1–4:   Crear miembro, entrenador, equipo y clase
Pasos 5–7:   Asignar todos a la clase
Pasos 8–10:  ❌ Intentar eliminar cada entidad → 400 Bad Request
Pasos 11–13: ✅ Remover entrenador, equipo y miembro de la clase
Pasos 14–17: ✅ Eliminar cada entidad → 204 No Content
```

---

## Diagramas

**Carpeta:** [`diagramas/`](diagramas/)

| Archivo                              | Descripción                                                              |
|--------------------------------------|--------------------------------------------------------------------------|
| `arquitectura-general.png`           | Vista general de los 4 microservicios y sus dependencias HTTP            |
| `diagrama-clases.png`                | Diagrama de clases UML con entidades y Value Objects de cada servicio    |
| `flujo-integridad-referencial.png`   | Diagrama de flujo del mecanismo de verificación antes de eliminaciones   |
| `diagrama-secuencia.png`             | Diagrama de secuencia del flujo completo: crear → asignar → eliminar     |

---

## Video de demostración

**Carpeta:** [`video/`](video/)

El video cubre:

1. Arranque de los 4 microservicios en orden (miembros → entrenadores → equipos → clases)
2. Ejecución de la colección de Postman paso a paso
3. Demostración del flujo de integridad referencial (intentos fallidos y exitosos de eliminación)
4. Revisión de la consola H2 para verificar el estado de la base de datos

---

## Presentación

**Carpeta:** [`presentacion/`](presentacion/)

La presentación cubre los siguientes temas:

1. Contexto y objetivo del proyecto
2. Arquitectura de microservicios adoptada
3. Decisiones de diseño: DDD, Value Objects, integridad referencial distribuida
4. Descripción de cada microservicio y sus responsabilidades
5. Flujo de comunicación entre servicios
6. Demostración de endpoints y resultados
7. Conclusiones y aprendizajes

---

## Cómo ejecutar el proyecto

### Pre-requisitos

- Java 17+
- Maven 3.8+

### Orden de arranque recomendado

Los microservicios sin dependencias externas deben iniciarse primero:

```bash
# Terminal 1 — Sin dependencias
cd microservicio-miembros && ./mvnw spring-boot:run

# Terminal 2 — Sin dependencias
cd microservicio-entrenadores && ./mvnw spring-boot:run

# Terminal 3 — Sin dependencias
cd microservicio-equipos && ./mvnw spring-boot:run

# Terminal 4 — Depende de los tres anteriores
cd microservicio-clases && ./mvnw spring-boot:run
```

### Verificar que todos están activos

| Servicio      | Health check                              |
|---------------|-------------------------------------------|
| Miembros      | `GET http://localhost:8081/api/miembros`  |
| Entrenadores  | `GET http://localhost:8083/api/entrenadores` |
| Equipos       | `GET http://localhost:8084/api/equipos`   |
| Clases        | `GET http://localhost:8082/api/clases`    |

---

## Autores

| Nombre                        | Código      | GitHub                                                   |
|-------------------------------|-------------|----------------------------------------------------------|
| Nicolás Cuéllar Molina        | A00394970   | [@Nicolas-CM](https://github.com/Nicolas-CM)             |
| Miguel Angel Martinez Vidal   | A00396327   | [@Miguel-23-ing](https://github.com/Miguel-23-ing)       |
| Davide Flamini Cazaran        | A00381665   | [@davidone007](https://github.com/davidone007)           |
| Andres Cabezas Guerrero       | A00394772   | [@andrescabezas26](https://github.com/andrescabezas26)   |
| Daron Mercado                 | A00395421   | [@Ing-Daron11](https://github.com/Ing-Daron11)           |

---

*Universidad — Ingeniería de Sistemas — Microservicios — Semestre IX — 2026*
