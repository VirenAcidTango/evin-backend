# Spec Delta

## Purpose

Permite a profesores y técnicos dar de alta, consultar, modificar y dar de baja a los alumnos, y consultar las estadísticas de actividad de cada uno.

## ADDED Requirements

### Requirement: Listado de alumnos
El sistema SHALL devolver a profesores y técnicos todos los alumnos ordenados por nombre, y a un usuario con rol alumno solo su propia ficha.

#### Scenario: Profesor consulta el listado
- **WHEN** un profesor envía `GET /api/v1/alumnos`
- **THEN** la respuesta es 200 con todos los alumnos ordenados por `nombre`, cada uno con `id`, `nombre`, `edad`, `curso`, `dificultad`, `progreso` e `img`

#### Scenario: Alumno consulta el listado
- **WHEN** un usuario con rol alumno envía `GET /api/v1/alumnos`
- **THEN** la respuesta es 200 con una lista que contiene solo su propia ficha

#### Scenario: Sin token
- **WHEN** se envía `GET /api/v1/alumnos` sin cabecera `Authorization`
- **THEN** la respuesta es 401

### Requirement: Consulta de un alumno
El sistema SHALL devolver la ficha de un alumno a profesores, técnicos y al propio alumno, y SHALL responder 404 si no existe.

#### Scenario: Profesor consulta un alumno
- **WHEN** un profesor envía `GET /api/v1/alumnos/:id` de un alumno existente
- **THEN** la respuesta es 200 con la ficha del alumno

#### Scenario: Alumno consulta su propia ficha
- **WHEN** un usuario con rol alumno envía `GET /api/v1/alumnos/:id` con el id de su ficha
- **THEN** la respuesta es 200

#### Scenario: Alumno consulta otra ficha
- **WHEN** un usuario con rol alumno envía `GET /api/v1/alumnos/:id` con el id de otro alumno
- **THEN** la respuesta es 403

#### Scenario: Alumno inexistente
- **WHEN** un profesor envía `GET /api/v1/alumnos/:id` con un id que no existe
- **THEN** la respuesta es 404

### Requirement: Alta de alumnos
El sistema SHALL permitir crear alumnos solo a profesores y técnicos, y SHALL validar nombre obligatorio, edad entre 1 y 99, dificultad Fácil, Medio o Difícil y progreso entre 0 y 100.

#### Scenario: Profesor crea un alumno
- **WHEN** un profesor envía `POST /api/v1/alumnos` con nombre, edad, curso, dificultad y progreso válidos
- **THEN** la respuesta es 201 con la ficha creada, que aparece después en el listado

#### Scenario: Familiar intenta crear un alumno
- **WHEN** un usuario con rol padre envía `POST /api/v1/alumnos`
- **THEN** la respuesta es 403 y no se crea ningún alumno

#### Scenario: Datos inválidos
- **WHEN** un profesor envía `POST /api/v1/alumnos` sin nombre, con edad 150 o con una dificultad que no existe
- **THEN** la respuesta es 422 con una lista `errors`

### Requirement: Modificación de alumnos
El sistema SHALL permitir modificar alumnos solo a profesores y técnicos, con las mismas validaciones que el alta.

#### Scenario: Profesor modifica un alumno
- **WHEN** un profesor envía `PATCH /api/v1/alumnos/:id` con un curso nuevo
- **THEN** la respuesta es 200 con la ficha actualizada

#### Scenario: Familiar intenta modificar un alumno
- **WHEN** un usuario con rol padre envía `PATCH /api/v1/alumnos/:id`
- **THEN** la respuesta es 403 y la ficha no cambia

#### Scenario: Alumno inexistente
- **WHEN** un profesor envía `PATCH /api/v1/alumnos/:id` con un id que no existe
- **THEN** la respuesta es 404

### Requirement: Baja de alumnos
El sistema SHALL permitir borrar alumnos solo a profesores y técnicos. Si el alumno tiene una cuenta de usuario asociada, SHALL borrar también esa cuenta.

#### Scenario: Profesor borra un alumno sin cuenta
- **WHEN** un profesor envía `DELETE /api/v1/alumnos/:id` de un alumno sin cuenta asociada
- **THEN** la respuesta es 200 y una consulta posterior a ese alumno responde 404

#### Scenario: Profesor borra un alumno con cuenta
- **WHEN** un profesor envía `DELETE /api/v1/alumnos/:id` de un alumno que se registró con su propia cuenta
- **THEN** la respuesta es 200 y el inicio de sesión con esa cuenta deja de funcionar

#### Scenario: Familiar intenta borrar un alumno
- **WHEN** un usuario con rol padre envía `DELETE /api/v1/alumnos/:id`
- **THEN** la respuesta es 403 y el alumno sigue existiendo

### Requirement: Estadísticas de un alumno
El sistema SHALL devolver a profesores y técnicos las estadísticas de actividad de un alumno calculadas a partir de sus sesiones.

#### Scenario: Campos de las estadísticas
- **WHEN** un profesor envía `GET /api/v1/alumnos/:id/estadisticas`
- **THEN** la respuesta es 200 con `sesiones_totales`, `sesiones_semana`, `ultima_sesion`, `juegos_practicados`, `juego_favorito`, `racha_dias` y `tendencia`

#### Scenario: Alumno sin sesiones
- **WHEN** un profesor consulta las estadísticas de un alumno sin sesiones
- **THEN** `sesiones_totales` y `racha_dias` son 0 y `tendencia` es `sin_datos`

#### Scenario: Juego más jugado
- **WHEN** un alumno tiene más sesiones de un juego que de cualquier otro
- **THEN** `juego_favorito` es ese juego y `juegos_practicados` contiene todos los juegos que ha jugado

#### Scenario: Racha de días consecutivos
- **WHEN** un alumno tiene sesiones en tres días consecutivos y ninguna en el día anterior al primero
- **THEN** `racha_dias` es 3, contando hacia atrás desde su última sesión

#### Scenario: Tendencia con pocas sesiones
- **WHEN** un alumno tiene menos de 4 sesiones
- **THEN** `tendencia` es `sin_datos`

#### Scenario: Tendencia de mejora
- **WHEN** un alumno tiene al menos 4 sesiones y el porcentaje medio de acierto de la mitad registrada más recientemente supera en más de 5 puntos al de la mitad registrada antes (por orden de registro, no por fecha de la partida)
- **THEN** `tendencia` es `mejorando`

#### Scenario: Alumno inexistente
- **WHEN** un profesor envía `GET /api/v1/alumnos/:id/estadisticas` con un id que no existe
- **THEN** la respuesta es 404
