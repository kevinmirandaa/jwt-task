# tasks-api

Demo del curso **Diseño de Software**: una Web API en Node.js (Express 5) organizada por capas que usa **Supabase** como base de datos, documentada con Swagger (OpenAPI 3.1) y protegida con **JWT**.

> El guion paso a paso está en [TUTORIAL.md](./TUTORIAL.md).

## Requisitos

- Node.js 20.6 o superior (usa `--env-file` y `--watch`, así que no hacen falta `dotenv` ni `nodemon`)
- Un proyecto en [Supabase](https://supabase.com)

## Puesta en marcha

1. En Supabase → **SQL Editor**, ejecutar en orden `supabase/01_tasks.sql` y `supabase/02_auth.sql`.
2. Copiar `.env.example` a `.env` y completar:
   - `SUPABASE_URL` y `SUPABASE_SECRET_KEY` (Project Settings → API Keys; usar la **secret key**, no la publishable).
   - `JWT_SECRET` (mínimo 32 caracteres). Para generarlo:
     ```bash
     node -e "console.log(require('crypto').randomBytes(48).toString('base64url'))"
     ```
   - `JWT_EXPIRES_IN` (por defecto `1h`).
3. Instalar y levantar:

```bash
npm install
npm run dev
```

4. Abrir la documentación interactiva en **http://localhost:3000/api/docs**.

## Cómo probar

- **Swagger:** `POST /auth/register` → `POST /auth/login` → botón **Authorize** (pegar el `accessToken` sin escribir "Bearer") → probar `/tasks`.
- **Postman:** importar `postman/tasks-api.postman_collection.json` y ejecutarla con **Run collection**. El login guarda el token automáticamente. Son 16 peticiones con sus pruebas.
- **REST Client (VS Code):** `requests.http`.

## Estructura

```
src/
├── server.js                 # Arranca el servidor HTTP (solo eso)
├── app.js                    # Construye la app: middlewares, rutas y manejo de errores
├── routes.js                 # Monta los módulos bajo /api
├── config/env.js             # Lee y valida variables de entorno (zod)
├── lib/
│   ├── supabase.js           # Cliente de Supabase
│   └── jwt.js                # Firma y verifica tokens (único archivo que conoce jsonwebtoken)
├── docs/                     # Swagger: helpers, openapi.js (junta los módulos), docs.routes.js
├── middlewares/              # validate, auth (requireAuth), not-found, error-handler
├── utils/http-error.js       # Error con código HTTP
└── modules/
    ├── health/               # GET /api/health, GET /api/health/db
    ├── auth/                 # register, login, me
    └── tasks/                # CRUD de tareas (protegido con requireAuth)
supabase/
├── 01_tasks.sql              # Tabla tasks + datos de ejemplo
└── 02_auth.sql               # Tabla users (email + hash bcrypt)
postman/
└── tasks-api.postman_collection.json
```

Cada módulo sigue las capas `routes → controller → service → repository`, con sus `*.schema.js` (zod) y `*.docs.js` (Swagger).

## Endpoints

| Método | Ruta                     | Auth | Descripción                              |
|--------|--------------------------|------|------------------------------------------|
| GET    | `/api/docs`              | No   | Swagger UI                               |
| GET    | `/api/docs/openapi.json` | No   | Especificación OpenAPI (JSON)            |
| GET    | `/api/health`            | No   | La API está viva                         |
| GET    | `/api/health/db`         | No   | La API llega a Supabase                  |
| POST   | `/api/auth/register`     | No   | Registrar usuario (`email`, `password`)  |
| POST   | `/api/auth/login`        | No   | Devuelve `{ accessToken }`               |
| GET    | `/api/auth/me`           | Sí   | Usuario dueño del token                  |
| GET    | `/api/tasks`             | Sí   | Listar tareas                            |
| GET    | `/api/tasks/:id`         | Sí   | Obtener una tarea                        |
| POST   | `/api/tasks`             | Sí   | Crear (`title`, `description?`, `done?`) |
| PATCH  | `/api/tasks/:id`         | Sí   | Actualizar parcialmente                  |
| DELETE | `/api/tasks/:id`         | Sí   | Eliminar                                 |

Las rutas protegidas exigen el header `Authorization: Bearer <token>`; sin él (o con un token inválido o expirado) responden `401`.

Errores siempre con la forma `{ "error": { "message": "...", "details": [...] } }`.
