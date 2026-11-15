# Proposal

## Why

El backend no tiene ninguna prueba que funcione: los 6 tests existentes son los que generó Rails al crear los controladores, llaman a rutas que no existen y sus fixtures usan columnas que ya se eliminaron. La CI de GitHub Actions falla en sus tres trabajos desde agosto de 2026 y no bloquea nada. Antes de refactorizar el backend o añadir juegos nuevos hace falta una red de seguridad que detecte cualquier cambio en el comportamiento actual de la API.

## What Changes

- Se eliminan los tests y fixtures generados por Rails que no prueban nada (controladores, modelos vacíos, mailer y ActionCable).
- La CI vuelve a funcionar:
  - el trabajo de tests define la contraseña de la base de datos que exige `config/database.yml` y deja de instalar Chrome y de ejecutar tests de sistema, que una API no tiene;
  - Brakeman deja de fallar por no ser la última versión (la opción `--ensure-latest` de `bin/brakeman`);
  - se corrigen los 16 avisos de estilo de RuboCop.
- Se añade una batería de tests de integración que recorre la API por HTTP, sin depender de la estructura interna, y cubre el camino feliz de cada endpoint y sus errores principales (sin token, rol sin permiso, recurso inexistente y datos inválidos).
- Las especificaciones de este cambio documentan el comportamiento actual de la API como línea base. No cambia ningún comportamiento observable.
- Queda fuera de este cambio público un conjunto de casos de autorización pendientes de una revisión de seguridad que se tratará en privado con los responsables del proyecto.

## Capabilities

### New Capabilities

- `autenticacion`: inicio de sesión, registro, perfil propio y recuperación de contraseña.
- `catalogo-juegos`: consulta pública del catálogo de juegos.
- `gestion-alumnos`: alta, consulta, modificación, baja y estadísticas de alumnos.
- `registro-sesiones`: registro y consulta de las partidas jugadas.
- `panel-profesor`: resumen de actividad para profesionales.
- `gestion-usuarios`: listado de usuarios y asignación de roles por un administrador.

### Modified Capabilities

Ninguna: el proyecto no tenía especificaciones previas.

## Impact

- Código de pruebas: `test/` (se sustituye por completo).
- Integración continua: `.github/workflows/ci.yml` y `bin/brakeman`.
- Estilo: los ficheros con avisos de RuboCop (solo comillas y espacios, sin cambios de lógica).
- Documentación: `README.md` con los pasos para ejecutar los tests en local.
- Sin cambios en la API, la base de datos ni el frontend.
