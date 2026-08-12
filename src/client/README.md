# Client (Frontend)

Interfaz de usuario: componentes, vistas y assets.

## Estructura sugerida

```
src/client/
├── src/
│   ├── components/   # Componentes reutilizables
│   ├── views/        # Vistas / páginas enrutadas
│   ├── services/     # Llamadas a la API del server
│   ├── assets/       # Imágenes, fuentes, estilos globales
│   └── main.*        # Punto de entrada
├── public/
├── Dockerfile
└── package.json
```

El `Dockerfile` de esta carpeta lo espera el servicio `client` de
`docker-compose.yml` y debe exponer el puerto `5173`.
