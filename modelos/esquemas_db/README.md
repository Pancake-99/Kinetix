# Esquemas de base de datos

Modelado relacional y scripts de migración.

## Estructura

- `modelo-relacional.md` — Diccionario de datos y descripción de entidades.
- `migrations/` — Scripts numerados e **inmutables** una vez aplicados:
  `001_init.sql`, `002_add_usuarios.sql`, …
- `seeds/` — Datos de prueba para entornos de desarrollo.

## Nota sobre Docker

`docker-compose.yml` monta esta carpeta en `/docker-entrypoint-initdb.d` del
contenedor de PostgreSQL. Los `.sql` en la raíz se ejecutan en orden alfabético
**solo la primera vez** que se crea el volumen `pgdata`. Para reejecutarlos:
`docker compose down -v`.
