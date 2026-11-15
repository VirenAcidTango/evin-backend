# Tasks

## 1. Base de pruebas limpia

- [ ] 1.1 Borrar los tests generados por Rails (`test/controllers/`, `test/models/`, `test/mailers/` y `test/channels/`) y verificar que no queda ningún fichero `*_test.rb` fuera de `test/integration/`
- [ ] 1.2 Sustituir los fixtures por los mínimos del diseño (una cuenta por rol y los juegos del catálogo) y verificar que `bin/rails db:fixtures:load RAILS_ENV=test` los carga sin errores
- [ ] 1.3 Añadir a `test/test_helper.rb` las ayudas genéricas (token por la API, cabecera `Authorization`, JSON de la respuesta) y el envío de correo `:test`, y verificar que `bin/rails test` termina en verde con 0 tests
- [ ] 1.4 Documentar en `README.md` cómo ejecutar los tests en local con Docker Compose y verificar que los pasos funcionan tal como están escritos

## 2. CI funcional

- [ ] 2.1 Definir `DB_PASSWORD` en el trabajo `test`, quitar la instalación de Chrome, `libvips` y `sqlite3` y ejecutar `bin/rails db:test:prepare test` sin `test:system`; verificar que el trabajo pasa en el PR
- [ ] 2.2 Quitar `--ensure-latest` de `bin/brakeman` y verificar que el trabajo `scan_ruby` pasa sin avisos
- [ ] 2.3 Aplicar `bin/rubocop -a` y verificar que el trabajo `lint` pasa y que el diff solo cambia comillas y espacios

## 3. Autenticación

- [ ] 3.1 Tests de inicio de sesión, registro y perfil propio según `specs/autenticacion/spec.md` y verificar que pasan
- [ ] 3.2 Tests de solicitud y restablecimiento de contraseña según `specs/autenticacion/spec.md`, con `travel_to` para la caducidad, y verificar que pasan sin contactar con Resend

## 4. Catálogo de juegos

- [ ] 4.1 Tests de `specs/catalogo-juegos/spec.md` y verificar que pasan sin token

## 5. Alumnos

- [ ] 5.1 Tests de listado, consulta, alta, modificación y baja según `specs/gestion-alumnos/spec.md` y verificar que pasan
- [ ] 5.2 Tests de estadísticas (campos, sin sesiones, juego más jugado, racha y tendencia) con `travel_to`, y verificar que pasan con cualquier fecha del sistema

## 6. Partidas

- [ ] 6.1 Tests de registro y consulta de partidas según `specs/registro-sesiones/spec.md` y verificar que pasan

## 7. Panel y usuarios

- [ ] 7.1 Tests de `specs/panel-profesor/spec.md` y verificar que pasan
- [ ] 7.2 Tests de `specs/gestion-usuarios/spec.md` y verificar que pasan

## 8. Integración

- [ ] 8.1 Verificar que cada escenario de las seis especificaciones tiene un test con su mismo nombre y que la CI pasa entera en el PR
- [ ] 8.2 Proteger `develop` y `main` en el fork para exigir los trabajos `scan_ruby`, `lint` y `test` antes de fusionar, y verificar que un PR con un test roto no se puede fusionar

## Workflow follow-up

- Archivar el cambio con OpenSpec cuando el PR de implementación esté fusionado en `develop`.
- Comprobar que `openspec/specs/` contiene las seis especificaciones archivadas.
