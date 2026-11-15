# Design

## Context

Ver proposal.md (Why). Estado de partida que condiciona el enfoque:

- Rails 7.2 en modo API con Minitest, que es el marco de pruebas que trae Rails. `test/test_helper.rb` carga todos los fixtures y ejecuta en paralelo.
- `config/database.yml` exige la variable `DB_PASSWORD` y usa `host: localhost`. El trabajo `test` de la CI arranca PostgreSQL 16 en el puerto 5432 con la contraseña `evin1234`, pero no define `DB_PASSWORD`, así que falla con `KeyError` antes de ejecutar nada.
- `bin/brakeman` añade `--ensure-latest`, que termina con código 5 cuando hay una versión más nueva de Brakeman, aunque no haya avisos de seguridad.
- La lógica de negocio vive en los controladores, así que hoy solo se puede probar a través de HTTP. Está previsto refactorizarla en cambios posteriores.
- `config/initializers/resend.rb` fija SMTP como método de envío en todos los entornos; los correos se envían con `deliver_later`.
- La ruta `GET /api/v1/juegos/:id` está declarada en `config/routes.rb`, pero el controlador no tiene la acción `show`.

## Goals / Non-Goals

**Goals:**

- Que los tests sobrevivan sin cambios al refactor posterior de controladores, servicios y modelos.
- Que `bin/rails test` funcione igual en local (Docker Compose) y en la CI.
- CI en verde y obligatoria para fusionar en `develop` y `main`.

**Non-Goals:**

- Tests del frontend o de extremo a extremo con navegador (cambio posterior).
- Tests unitarios de modelos o servicios: llegarán cuando el refactor extraiga la lógica.
- Corregir comportamientos dudosos: este cambio los fija tal como están. Cualquier corrección será un cambio propio que modifique la especificación correspondiente.
- La ruta `GET /api/v1/juegos/:id` sin acción: no se especifica ni se prueba. Se decidirá en otro cambio si se implementa o se quita la ruta.

## Decisions

### Tests de integración de Rails en vez de una suite externa

Se usa `ActionDispatch::IntegrationTest`, que hace peticiones HTTP a la aplicación completa en el mismo proceso. Los tests solo usan rutas, cabeceras y JSON; nunca llaman a controladores, modelos ni clases internas.

- Alternativa: suite externa en TypeScript (por ejemplo Playwright) contra el entorno en Docker. Independiente del lenguaje, pero más lenta, con otra pila que mantener y sin necesidad real: el refactor previsto es dentro de Rails.
- Alternativa: RSpec. Muy extendido, pero añade dependencias y el proyecto ya trae Minitest.

### Datos de prueba creados a través de la API

Los fixtures se reducen al mínimo que la API no permite crear: una cuenta por rol (técnico, profesor, padre) y los juegos del catálogo. Alumnos, partidas y cuentas de alumno se crean en cada test llamando a la propia API.

- Motivo: los fixtures se escriben contra las tablas. Cuando un cambio posterior migre `sesions` a claves foráneas, solo habrá que tocar estos pocos fixtures y no cientos de líneas.
- Alternativa: FactoryBot. Más flexible, pero sigue acoplado al modelo de datos y añade una dependencia.
- Los datos de prueba no usan las credenciales de `db/seeds.rb`.

### Ayudas de test pequeñas y explícitas

`test/test_helper.rb` incluye solo ayudas genéricas: obtener un token iniciando sesión por la API, construir la cabecera `Authorization` y leer el JSON de la respuesta. Nada de lógica de negocio en las ayudas, para que un test se entienda leyéndolo.

### Tiempo controlado

Los escenarios que dependen de la fecha (racha, sesiones de hoy y de la semana, caducidad del token de recuperación) usan `travel_to` para fijar el día, de modo que no fallen según cuándo se ejecuten.

### Correos sin envío real

Rails incluye en los tests las ayudas de Active Job, que encolan los trabajos sin ejecutarlos, y las de Action Mailer. Los escenarios de solicitud comprueban que el correo se encola (o no). Para el restablecimiento, el test ejecuta el trabajo encolado y saca el token del enlace del correo entregado, sin leer la base de datos. Como `config/initializers/resend.rb` fija SMTP también en test, `test/test_helper.rb` fija el método de envío `:test`, de modo que ningún test contacta con Resend.

### Organización

Un fichero por capacidad en `test/integration/api/v1/` (`autenticacion_test.rb`, `catalogo_juegos_test.rb`, etc.), con un test por escenario de la especificación y el nombre del escenario como nombre del test. Así cada escenario se puede rastrear hasta su test.

### CI

- Trabajo `test`: se define `DB_PASSWORD` con la contraseña del servicio, se elimina la instalación de Chrome, `libvips` y `sqlite3`, y se ejecuta `bin/rails db:test:prepare test` sin `test:system`.
- Brakeman: se quita `--ensure-latest` de `bin/brakeman`. Mantener Brakeman al día queda en manos de Dependabot, que ya está configurado.
- RuboCop: `bin/rubocop -a` (solo correcciones seguras) para los 16 avisos actuales.
- Ramas: en el fork se protegen `develop` y `main` para exigir los tres trabajos en verde antes de fusionar.

## Risks / Trade-offs

- [Los tests fijan comportamientos que quizá sean errores, como que la racha cuente hasta la última sesión y no hasta hoy] → Es intencionado: el objetivo es detectar cambios. Corregirlos será un cambio explícito que modifique la especificación y su test.
- [Casos de autorización no cubiertos en este cambio público] → Se tratarán en privado y se añadirán tests cuando estén corregidos.
- [Los tests en paralelo comparten la base de datos de cada proceso] → Cada test corre en su transacción; los tests no dependen de recuentos absolutos salvo en datos que crean ellos mismos.
- [El inicializador de Resend fuerza SMTP también en test] → `test/test_helper.rb` fija el envío `:test`. Si se olvidara, la entrega fallaría por falta de clave en vez de enviar un correo real.

## Migration Plan

Sin migración: no cambian datos ni API. Para volver atrás basta con revertir el PR.
