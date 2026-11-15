# Spec Delta

## Purpose

Guarda el resultado de cada partida jugada (alumno, juego, fecha, aciertos e intentos) y permite consultarlas para el seguimiento de los alumnos.

## ADDED Requirements

### Requirement: Registro de una partida
El sistema SHALL guardar una partida enviada por un usuario autenticado y devolverla con su porcentaje de acierto. Si no se indica fecha, SHALL usar la fecha del día.

#### Scenario: Partida de un juego del catálogo
- **WHEN** un profesor envía `POST /api/v1/sesiones` con `alumno`, `alumno_id`, `juego` igual al nombre de un juego del catálogo, `aciertos` 3 e `intentos` 4
- **THEN** la respuesta es 201 con `id`, `alumno_nombre`, `juego_nombre`, `fecha` de hoy, `aciertos`, `intentos`, `alumno_id`, `juego_id` del juego y `porcentaje` 75

#### Scenario: Partida de un juego que no está en el catálogo
- **WHEN** un profesor envía `POST /api/v1/sesiones` con un nombre de juego que no existe en el catálogo
- **THEN** la respuesta es 201, la partida guarda ese `juego_nombre` y su `juego_id` queda vacío

#### Scenario: Más aciertos que intentos
- **WHEN** se envía `POST /api/v1/sesiones` con `aciertos` mayor que `intentos`
- **THEN** la respuesta es 422 con una lista `errors`

#### Scenario: Datos incompletos o inválidos
- **WHEN** se envía `POST /api/v1/sesiones` sin nombre de alumno, sin juego, con `intentos` 0 o con `aciertos` negativos
- **THEN** la respuesta es 422 con una lista `errors`

#### Scenario: Sin token
- **WHEN** se envía `POST /api/v1/sesiones` sin cabecera `Authorization`
- **THEN** la respuesta es 401 y no se guarda ninguna partida

### Requirement: Consulta de partidas
El sistema SHALL devolver las partidas ordenadas de la más reciente a la más antigua, con filtros opcionales por alumno y por juego.

#### Scenario: Profesor consulta todas las partidas
- **WHEN** un profesor envía `GET /api/v1/sesiones`
- **THEN** la respuesta es 200 con las partidas ordenadas por `fecha` descendente

#### Scenario: Filtro por alumno
- **WHEN** un profesor envía `GET /api/v1/sesiones?alumno_id=<id>`
- **THEN** la respuesta contiene solo las partidas de ese alumno

#### Scenario: Filtro por juego
- **WHEN** un profesor envía `GET /api/v1/sesiones?juego=<texto>`
- **THEN** la respuesta contiene solo las partidas cuyo nombre de juego incluye ese texto, sin distinguir mayúsculas

#### Scenario: Sin token
- **WHEN** se envía `GET /api/v1/sesiones` sin cabecera `Authorization`
- **THEN** la respuesta es 401
