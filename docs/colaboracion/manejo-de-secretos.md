# Manejo de secretos para contribuidores

Esta app maneja datos personales de personas en una emergencia. Las credenciales
permiten leer, escribir o desplegar partes sensibles del sistema, así que se tratan
como material privado aunque sean de staging.

## Reglas rápidas

- Nunca subas `.env*`, llaves, tokens, URLs de base de datos ni capturas que los muestren.
- Usa `.env.local` para desarrollo local; se crea desde `.env.example` y está ignorado por Git.
- Pide valores reales por el canal privado del equipo. No los pegues en issues, PRs, logs ni chat público.
- Si una credencial se filtra, avísalo de inmediato por canal privado para rotarla. No intentes ocultarlo con force-push.
- Ante la duda, trata cualquier variable sin `NEXT_PUBLIC_` como secreta.

## Archivos locales

Para levantar el proyecto:

```bash
cp .env.example .env.local
```

Luego completa solo lo que necesites para tu tarea. No hace falta tener todas las
integraciones activas para trabajar en UI, docs o lógica pura.

| Archivo | Uso | Se versiona |
|---|---|---|
| `.env.example` | Plantilla documentada sin secretos reales | Sí |
| `.env.local` | Valores locales reales o de staging | No |
| `.env`, `.env.production`, backups de env | No deben aparecer en el repo | No |

## Variables públicas vs privadas

En Next.js, cualquier variable con prefijo `NEXT_PUBLIC_` puede terminar en el
bundle del navegador. Eso no significa que sea irrelevante, pero sí debe ser
segura de exponer.

Públicas aceptables:

- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY`
- `NEXT_PUBLIC_SITE_URL`
- URLs públicas de mapas o endpoints pensados para navegador

Privadas, solo servidor:

- `SUPABASE_SECRET_KEY` o `SUPABASE_SERVICE_ROLE_KEY`
- `SUPABASE_DB_URL`
- `OPENAI_API_KEY`
- `FR_API_KEY`
- API keys de socios, tokens de admin, webhooks y credenciales de deploy

No agregues `NEXT_PUBLIC_` a una variable para "hacerla funcionar" en el cliente.
Si el navegador necesita una operación sensible, crea o usa un endpoint/server
action que valide permisos y use la credencial del lado servidor.

## Supabase

La publishable key (`sb_publishable_...`) está pensada para el navegador y queda
protegida por RLS y por las vistas `public_*`. No permite asumir que cualquier
lectura/escritura es segura: la seguridad real depende de las policies, grants y
endpoints.

La secret key (`sb_secret_...`) y la legacy `service_role` son de servidor. Omiten
RLS y nunca deben llegar al cliente, logs, screenshots ni herramientas de terceros.
Úsalas solo desde server actions, route handlers, scripts locales controlados o
GitHub/Vercel secrets.

## OpenAI, FR y servicios externos

`OPENAI_API_KEY` y `FR_API_KEY` son server-only. Si faltan, la app debe degradar de
forma segura: el clasificador usa heurística local y el reconocimiento facial se
desactiva o responde sin bloquear el flujo. No uses keys personales en PRs ni las
compartas para reproducir bugs públicamente.

## GitHub Actions y despliegue

Los secretos de CI/CD viven en GitHub Secrets o Vercel, no en el repositorio.
Ejemplos:

- `SUPABASE_DB_URL` / `SUPABASE_DB_URL_STAGING`
- `NEXT_PUBLIC_SUPABASE_URL` / `NEXT_PUBLIC_SUPABASE_URL_STAGING`
- `SUPABASE_SECRET_KEY` / `SUPABASE_SECRET_KEY_STAGING`

Los workflows deben referenciar `secrets.*`; no hardcodees valores ni los imprimas
con `echo`.

## Si se filtra un secreto

1. No abras un issue público con el valor.
2. Avisa por el canal privado de seguridad o a los maintainers.
3. Indica qué secreto fue, en qué commit/log/captura aparece y desde cuándo.
4. Espera rotación antes de volver a usarlo.
5. Si ya está en Git, asume que está comprometido aunque borres el commit.

La prioridad es rotar rápido y reducir impacto, no buscar culpables.
