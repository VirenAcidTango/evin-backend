# Spec Delta

## Purpose

Permite al administrador de la plataforma (rol técnico) ver todas las cuentas y asignar roles, que es la única vía para obtener los roles profesor y técnico.

## ADDED Requirements

### Requirement: Listado de usuarios
El sistema SHALL devolver al administrador todas las cuentas ordenadas por nombre, sin datos de contraseña, y SHALL negarlo a los demás roles.

#### Scenario: Administrador consulta las cuentas
- **WHEN** un usuario con rol técnico envía `GET /api/v1/users`
- **THEN** la respuesta es 200 con todas las cuentas ordenadas por `nombre`, cada una solo con `id`, `nombre`, `email` y `rol`

#### Scenario: Profesor intenta consultar las cuentas
- **WHEN** un profesor envía `GET /api/v1/users`
- **THEN** la respuesta es 403

### Requirement: Asignación de roles
El sistema SHALL permitir al administrador cambiar el rol de una cuenta a alumno, profesor, técnico o padre, y SHALL impedir que se quite a sí mismo el rol de técnico.

#### Scenario: Administrador asigna el rol profesor
- **WHEN** un usuario con rol técnico envía `PATCH /api/v1/users/:id` con `rol` igual a `profesor`
- **THEN** la respuesta es 200 con la cuenta y su rol nuevo, y esa cuenta puede consultar después el listado de alumnos

#### Scenario: Rol inexistente
- **WHEN** un usuario con rol técnico envía `PATCH /api/v1/users/:id` con un rol que no existe
- **THEN** la respuesta es 422 y el rol no cambia

#### Scenario: Administrador intenta quitarse su propio rol
- **WHEN** un usuario con rol técnico envía `PATCH /api/v1/users/:id` con su propio id y un rol distinto de `tecnico`
- **THEN** la respuesta es 422 y sigue siendo técnico

#### Scenario: Cuenta inexistente
- **WHEN** un usuario con rol técnico envía `PATCH /api/v1/users/:id` con un id que no existe
- **THEN** la respuesta es 404

#### Scenario: Profesor intenta asignar un rol
- **WHEN** un profesor envía `PATCH /api/v1/users/:id`
- **THEN** la respuesta es 403 y el rol no cambia
