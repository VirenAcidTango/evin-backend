# Spec Delta

## Purpose

Da a profesores y técnicos un resumen de la actividad de todos los alumnos: totales, actividad de la semana, alumnos inactivos, juegos más usados y tendencias.

## ADDED Requirements

### Requirement: Resumen de actividad
El sistema SHALL devolver a profesores y técnicos el resumen de actividad y SHALL negarlo a los demás roles.

#### Scenario: Profesor consulta el panel
- **WHEN** un profesor envía `GET /api/v1/dashboard`
- **THEN** la respuesta es 200 con `total_alumnos`, `total_sesiones`, `sesiones_hoy`, `sesiones_semana`, `activos_semana`, `sin_sesiones`, `juegos_populares`, `actividad_reciente` y `tendencias` con `mejorando`, `empeorando` y `estable`

#### Scenario: Los totales reflejan las partidas registradas
- **WHEN** se registra una partida con fecha de hoy y después un profesor consulta el panel
- **THEN** `total_sesiones` y `sesiones_hoy` aumentan en uno y la partida aparece la primera en `actividad_reciente`

#### Scenario: Familiar intenta consultar el panel
- **WHEN** un usuario con rol padre envía `GET /api/v1/dashboard`
- **THEN** la respuesta es 403

#### Scenario: Sin token
- **WHEN** se envía `GET /api/v1/dashboard` sin cabecera `Authorization`
- **THEN** la respuesta es 401
