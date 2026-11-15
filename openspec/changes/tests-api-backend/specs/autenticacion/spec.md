# Spec Delta

## Purpose

Permite a profesionales, familias y alumnos crear una cuenta, iniciar sesión con un token, consultar y editar su perfil y recuperar la contraseña por correo.

## ADDED Requirements

### Requirement: Inicio de sesión
El sistema SHALL devolver un token de acceso y los datos del usuario cuando el correo y la contraseña son correctos, sin distinguir mayúsculas en el correo, y SHALL rechazar las credenciales incorrectas.

#### Scenario: Credenciales correctas
- **WHEN** se envía `POST /api/v1/login` con el correo y la contraseña de un usuario existente
- **THEN** la respuesta es 200 con un `token` y un `user` con `id`, `nombre`, `email`, `rol` y `alumno_id`

#### Scenario: Correo con mayúsculas
- **WHEN** se envía `POST /api/v1/login` con el correo de un usuario existente escrito en mayúsculas y su contraseña
- **THEN** la respuesta es 200 con un `token`

#### Scenario: Contraseña incorrecta
- **WHEN** se envía `POST /api/v1/login` con un correo existente y una contraseña errónea
- **THEN** la respuesta es 401 con un mensaje de error y sin `token`

#### Scenario: Usuario inexistente
- **WHEN** se envía `POST /api/v1/login` con un correo que no está registrado
- **THEN** la respuesta es 401

### Requirement: Registro público
El sistema SHALL crear una cuenta nueva y devolver su token. El registro público SHALL admitir solo los roles alumno y padre; cualquier otro rol solicitado SHALL quedar como alumno.

#### Scenario: Registro como familiar
- **WHEN** se envía `POST /api/v1/register` con nombre, correo nuevo, contraseña y rol `padre`
- **THEN** la respuesta es 201 con un `token` y un `user` con rol `padre`

#### Scenario: Registro como alumno crea su ficha
- **WHEN** se envía `POST /api/v1/register` con un nombre que no existe, correo nuevo, contraseña y rol `alumno`
- **THEN** la respuesta es 201 y el `user` devuelto tiene rol `alumno` y un `alumno_id` no vacío

#### Scenario: Rol privilegiado solicitado
- **WHEN** se envía `POST /api/v1/register` con rol `profesor` o `tecnico`
- **THEN** la respuesta es 201 y el `user` devuelto tiene rol `alumno`

#### Scenario: Correo ya registrado
- **WHEN** se envía `POST /api/v1/register` con un correo que ya existe
- **THEN** la respuesta es 422 con una lista `errors`

#### Scenario: Falta el nombre
- **WHEN** se envía `POST /api/v1/register` sin nombre
- **THEN** la respuesta es 422 con una lista `errors`

### Requirement: Consulta del perfil propio
El sistema SHALL devolver los datos del usuario identificado por el token y SHALL rechazar las peticiones sin token válido.

#### Scenario: Token válido
- **WHEN** se envía `GET /api/v1/me` con un token válido
- **THEN** la respuesta es 200 con un `user` cuyos datos son los del dueño del token

#### Scenario: Sin token
- **WHEN** se envía `GET /api/v1/me` sin cabecera `Authorization`
- **THEN** la respuesta es 401

#### Scenario: Token manipulado o caducado
- **WHEN** se envía `GET /api/v1/me` con un token mal formado, firmado con otra clave o caducado
- **THEN** la respuesta es 401

### Requirement: Edición del perfil propio
El sistema SHALL permitir al usuario cambiar su nombre y su correo, y SHALL exigir la contraseña actual para cambiar la contraseña.

#### Scenario: Cambio de nombre
- **WHEN** se envía `PATCH /api/v1/me` con un token válido y un nombre nuevo
- **THEN** la respuesta es 200 con un `token` nuevo y el `user` con el nombre cambiado

#### Scenario: Cambio de contraseña con la contraseña actual
- **WHEN** se envía `PATCH /api/v1/me` con `password` nueva y `password_actual` correcta
- **THEN** la respuesta es 200 y a partir de ese momento el inicio de sesión funciona con la contraseña nueva y no con la antigua

#### Scenario: Cambio de contraseña sin la contraseña actual correcta
- **WHEN** se envía `PATCH /api/v1/me` con `password` nueva y `password_actual` errónea o ausente
- **THEN** la respuesta es 422 y la contraseña no cambia

### Requirement: Solicitud de recuperación de contraseña
El sistema SHALL responder siempre con el mismo mensaje genérico, exista o no el correo, y SHALL enviar el correo de recuperación solo si la cuenta existe.

#### Scenario: Correo registrado
- **WHEN** se envía `POST /api/v1/password/forgot` con el correo de un usuario existente
- **THEN** la respuesta es 200 con el mensaje genérico y se encola un correo de recuperación para ese usuario

#### Scenario: Correo no registrado
- **WHEN** se envía `POST /api/v1/password/forgot` con un correo que no existe
- **THEN** la respuesta es 200 con el mismo mensaje genérico y no se encola ningún correo

### Requirement: Restablecimiento de contraseña
El sistema SHALL cambiar la contraseña cuando el token de recuperación existe y tiene menos de 2 horas, y SHALL invalidar el token tras usarlo.

#### Scenario: Token vigente
- **WHEN** se envía `POST /api/v1/password/reset` con un token de recuperación emitido hace menos de 2 horas y una contraseña nueva
- **THEN** la respuesta es 200, el inicio de sesión funciona con la contraseña nueva y el mismo token ya no se acepta

#### Scenario: Token caducado
- **WHEN** se envía `POST /api/v1/password/reset` con un token emitido hace más de 2 horas
- **THEN** la respuesta es 422 y la contraseña no cambia

#### Scenario: Token desconocido
- **WHEN** se envía `POST /api/v1/password/reset` con un token que no existe
- **THEN** la respuesta es 422
