# Spec Delta

## Purpose

Ofrece a cualquier visitante, con o sin cuenta, la lista de juegos de estimulación visual disponibles en EVIN con su descripción, nivel y tipo.

## ADDED Requirements

### Requirement: Consulta pública del catálogo
El sistema SHALL devolver todos los juegos ordenados por nombre, sin exigir autenticación.

#### Scenario: Sin token
- **WHEN** se envía `GET /api/v1/juegos` sin cabecera `Authorization`
- **THEN** la respuesta es 200 con una lista de juegos ordenada por `nombre`

#### Scenario: Campos de cada juego
- **WHEN** se envía `GET /api/v1/juegos`
- **THEN** cada elemento incluye `id`, `nombre`, `descripcion`, `img`, `nivel` y `tipo`
