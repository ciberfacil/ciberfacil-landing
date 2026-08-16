# ciberfacil-landing

Landing page estática (`index.html`) para el registro de alumnos en los cursos de CiberFácil. Se sirve con nginx (`Dockerfile`: `nginx:alpine` + `COPY . /usr/share/nginx/html`).

Flujo: el visitante rellena nombre, email y código de acceso, y el formulario llama al webhook de registro del bot. Si la respuesta es correcta, la página genera un enlace de Telegram (`https://t.me/<bot_username>?start=<token>`) para que el alumno vincule su cuenta.

## Integración con telegram-onboarding-bot

### Contrato del webhook

El formulario (`index.html`) hace `POST` a `WEBHOOK_URL` (constante definida en el `<script>` de `index.html`, actualmente `https://ciberfacil.com/registro`) con JSON:

```json
{
  "org": "test-alta",
  "nombre": "Nombre Apellido",
  "email": "alumno@ejemplo.com",
  "access_code": "CODIGO"
}
```

Este endpoint lo implementa `api_registro.ts` (`POST /registro`) en el repo `telegram-onboarding-bot`. Respuesta esperada:

```json
{ "status": "ok", "bot_username": "NombreDelBot", "codigo_unico": "TOKEN123" }
```

o, en caso de error, `{ "status": "error", "message": "..." }`. La landing usa `bot_username` y `codigo_unico` para construir el enlace `https://t.me/<bot_username>?start=<codigo_unico>`.

### Enrutado por organización

El cliente se selecciona por query param en la URL de la landing: `?org=<slug>`. Si no viene, la landing usa `general` por defecto. Ese `slug` viaja tal cual en el `org` del POST, y el bot lo resuelve contra la tabla `clientes` (`SELECT id, bot_username FROM clientes WHERE slug = $1`) para saber a qué cliente y a qué bot pertenece el registro.

### Más detalles

Para el esquema de base de datos, el flujo completo de vinculación por token, y la arquitectura del bot/panel, ver `docs/` en el repo [`telegram-onboarding-bot`](https://github.com/ciberfacil/telegram-onboarding-bot).
