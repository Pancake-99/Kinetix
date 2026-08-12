# Server (Backend)

Lógica de negocio, controladores y rutas.

## Estructura sugerida

```
src/server/
├── src/
│   ├── routes/       # Definición de endpoints
│   ├── controllers/  # Orquestación de la petición
│   ├── services/     # Lógica de negocio
│   ├── models/       # Acceso a datos / entidades
│   ├── middlewares/  # Autenticación, validación, errores
│   └── index.*       # Punto de entrada
├── Dockerfile
└── package.json
```

El `Dockerfile` de esta carpeta lo espera el servicio `server` de
`docker-compose.yml` y debe exponer el puerto `3000`.

La conexión a la base de datos se toma de `DATABASE_URL`; nunca se escriben
credenciales en el código.
