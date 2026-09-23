# Kinetix

Repositorio del proyecto Kinetix. Este documento define la **estructura**, las
**reglas de trabajo** y el **onboarding** del equipo.

---

## Estructura del repositorio

```
kinetix-workspace/
├── docs/                      # Documentación institucional y gestión
│   ├── actas/                 # Acta de constitución, minutas de reunión
│   ├── normativas/            # Checklists ISO (25010, 9001, 27001)
│   ├── manuales/              # Guías de despliegue y estilo de código
│   ├── entrevistas/           # Entrevistas a stakeholders
│   ├── srs/                   # Especificación de requisitos de software
│   ├── trazabilidad/          # Matrices de trazabilidad de requisitos
│   └── arquitectura/          # Decisiones y vistas de arquitectura
├── modelos/                   # Diseño arquitectónico y datos
│   ├── diagramas_uml/         # Casos de uso, clases, actividades
│   ├── esquemas_db/           # Modelado relacional y scripts de migración
│   └── c4/                    # Diagramas de arquitectura (modelo C4)
├── src/                       # Código fuente
│   ├── client/                # Frontend (componentes, vistas, assets)
│   └── server/                # Backend (lógica de negocio, controladores, rutas)
├── tests/                     # Aseguramiento de calidad (QA)
│   ├── unit/                  # Pruebas unitarias por componente/módulo
│   └── integration/           # Pruebas de integración de API y BD
├── .gitignore
├── .env.example               # Plantilla de variables de entorno
├── docker-compose.yml         # Despliegue en contenedores
└── README.md
```

Cada carpeta tiene su propio `README.md` con la convención de nombres y el
contenido esperado. **Léelo antes de agregar archivos ahí.**

---

## Onboarding

### Requisitos

- Git
- Docker Desktop (incluye `docker compose`)
- Node.js LTS (solo si trabajas fuera de contenedores)

### Puesta en marcha

```bash
git clone <url-del-repositorio>
cd Kinetix

cp .env.example .env        # Windows PowerShell: Copy-Item .env.example .env
# Editar .env con las credenciales locales

docker compose up -d
```

| Servicio | URL |
|---|---|
| Cliente  | http://localhost:5173 |
| Servidor | http://localhost:3000 |
| Base de datos | `localhost:5432` |

Comandos útiles:

```bash
docker compose ps                # Estado de los servicios
docker compose logs -f server    # Logs en vivo
docker compose down              # Detener
docker compose down -v           # Detener y borrar el volumen de la BD
```

> El stack asume Node.js + PostgreSQL. `src/client` y `src/server` necesitan su
> propio `Dockerfile` para que `docker compose up` funcione; hasta entonces,
> levanta solo la base de datos con `docker compose up -d db`.

---

## Reglas del repositorio

### 1. Nunca subir secretos

`.env`, claves, certificados y volcados de base de datos están en `.gitignore`.
Si necesitas agregar una variable nueva, decláralas en `.env.example` **sin
valores reales**.

### 2. Ramas

| Rama | Propósito |
|---|---|
| `main` | Estable. Solo recibe merges vía Pull Request. |
| `develop` | Integración del trabajo en curso. |
| `feature/<nombre>` | Nueva funcionalidad. |
| `fix/<nombre>` | Corrección de errores. |
| `docs/<nombre>` | Cambios solo de documentación. |

No se hace push directo a `main`.

### 3. Commits

Formato [Conventional Commits](https://www.conventionalcommits.org/):

```
<tipo>(<alcance>): <descripción en imperativo>
```

Tipos: `feat`, `fix`, `docs`, `test`, `refactor`, `chore`, `style`.

```
feat(server): agregar endpoint de autenticación
docs(actas): registrar minuta del 2026-08-11
test(unit): cubrir validación de usuarios
```

### 4. Pull Requests

- Describe **qué** cambia y **por qué**.
- Enlaza el issue o acuerdo de minuta correspondiente.
- Las pruebas deben pasar antes de solicitar revisión.
- Requiere al menos una aprobación de otro integrante.

### 5. Calidad

- Todo módulo nuevo en `src/` incorpora su prueba en `tests/unit/`.
- Todo endpoint nuevo incorpora su prueba en `tests/integration/`.
- Los cambios de esquema van como migración numerada en
  `modelos/esquemas_db/migrations/`; las migraciones ya aplicadas no se editan.

### 6. Documentación

Un cambio no está terminado hasta que la documentación afectada se actualiza:
decisiones de arquitectura en `modelos/`, acuerdos en `docs/actas/`, y
procedimientos en `docs/manuales/`.
