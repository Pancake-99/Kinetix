# Pruebas de integración

Verifican la API y la base de datos trabajando en conjunto.

## Requisitos

Necesitan la infraestructura levantada:

```bash
docker compose up -d db server
```

## Convención

- Nombre: `<recurso>.integration.test.*` — por ejemplo `usuarios.integration.test.js`
- Cada prueba deja la base de datos en el estado en que la encontró
  (transacción con rollback o limpieza en el teardown).
- Se ejecutan contra una base de datos de prueba, **nunca** contra producción.
