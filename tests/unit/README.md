# Pruebas unitarias

Pruebas aisladas por componente o módulo. Sin red, sin base de datos, sin
sistema de archivos: toda dependencia externa va mockeada.

## Convención

- Un archivo de prueba por módulo, espejando la ruta en `src/`.
  Ejemplo: `src/server/src/services/auth.js` → `tests/unit/server/services/auth.test.js`
- Nombre: `<modulo>.test.*`
- Cada prueba sigue Arrange / Act / Assert y valida un solo comportamiento.
