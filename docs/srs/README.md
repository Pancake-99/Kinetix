# Especificación de Requisitos de Software (SRS)
## SIGFaR — Sistema Integrado de Gestión de Fauna Rescatada
### Bioparque Municipal Vesty Pakos · Mallasa, La Paz

| Campo | Valor |
|---|---|
| Código | SRS-SIGFAR-001 |
| Versión | 1.0 |
| Fecha | 22/09/2026 |
| Norma de referencia | ISO/IEC/IEEE 29148:2018 |
| Estado | Borrador para validación del cliente (COM-06, 05/10/2026) |
| Documento fuente | ACT-SIS213-BIO-001 — Minuta de entrevista del 17/09/2026 |
| Ruta en repositorio | `docs/srs/SRS-SIGFaR-v1.0.md` |

**Historial de versiones**

| Versión | Fecha | Autor | Cambio |
|---|---|---|---|
| 0.1 | 17/09/2026 | Equipo consultor | Requisitos preliminares incluidos en la minuta (6 RF, 5 RNF) |
| 1.0 | 22/09/2026 | Equipo consultor | SRS completo: 10 RF, 5 RNF, matriz de atributos y matriz de trazabilidad |

**Equipo consultor (UCB)**

| Nombre | Rol |
|---|---|
| Gemina Ponce | Director de Proyecto y Analista de Sistemas |
| Emilia Crespo | Especialista QA y Desarrolladora Frontend |
| Alejandro Bobarin | Arquitecto de Software y Desarrollador Backend |

> Nota: nombres de funcionarios y cifras operativas son simulados con fines académicos.

---

## 1. Introducción

### 1.1 Propósito
Este documento especifica los requisitos funcionales y no funcionales del SIGFaR. Está dirigido a la Dirección del Bioparque (validación del alcance), al equipo de desarrollo (diseño e implementación) y al equipo de QA (verificación).

### 1.2 Alcance
El SIGFaR registrará cada animal rescatado desde su ingreso hasta su destino final (liberación, traslado, permanencia o deceso), reemplazando el uso actual de papel, cuadernos, pizarra y al menos cuatro archivos Excel desconectados.

- **Nivel TPS (núcleo):** ingresos, código único por individuo, historia clínica, alertas sanitarias, movimientos entre recintos, dietas, inventario de insumos, egresos y control de acceso.
- **Nivel MIS (v1.0):** reporte mensual consolidado y costeo por especie e individuo.

**Fuera de alcance de v1.0:** contabilidad general, planillas, boletería/visitantes e integración directa con sistemas de la autoridad ambiental. Las necesidades NEC-10 (sin conexión), NEC-14, NEC-15 y NEC-16 se difieren a v1.1 (ver sección 6).

**Objetivos medibles del negocio**

| Indicador | Situación AS-IS | Meta con SIGFaR |
|---|---|---|
| Tiempo para consultar antecedentes clínicos | 20–40 min | ≤ 2 s |
| Tiempo para armar el reporte mensual | ~2 semanas | < 60 s |
| Individuos registrados en duplicado | Casos recurrentes | 0 |
| Pérdida de historial por daño físico | Ocurrió en 2022 | Respaldo diario externo |

### 1.3 Definiciones y acrónimos

| Término | Definición |
|---|---|
| SIGFaR | Sistema Integrado de Gestión de Fauna Rescatada |
| Código SIGFaR | Identificador único generado por el sistema para cada individuo (formato `SGF-AAAA-NNNNN`) |
| POFOMA | Policía Forestal y de Preservación del Medio Ambiente |
| TPS / MIS | Sistema de procesamiento de transacciones / Sistema de información gerencial |
| MoSCoW | Priorización Must, Should, Could, Won't |
| RBAC | Control de acceso basado en roles |
| JWT | JSON Web Token, usado para autenticación en la API |
| FEFO | First Expired, First Out: se consume primero el lote que vence antes |
| RPO / RTO | Pérdida máxima de datos tolerada / tiempo máximo de recuperación |
| p95 | Percentil 95 del tiempo de respuesta |

### 1.4 Referencias
1. ISO/IEC/IEEE 29148:2018 — Systems and software engineering — Life cycle processes — Requirements engineering.
2. ACT-SIS213-BIO-001 — Minuta de entrevista y relevamiento de requerimientos (`docs/entrevistas/`).
3. Documento de Arquitectura, Modelo C4 niveles 1–3 (`docs/arquitectura/`, `modelos/c4/`).

### 1.5 Organización del documento
La sección 2 describe el producto y su entorno; la 3, los requisitos específicos; la 4, la matriz de atributos; la 5, la trazabilidad bidireccional; la 6, los requisitos diferidos. Los apéndices listan historias de usuario y casos de prueba.

---

## 2. Descripción general

### 2.1 Perspectiva del producto
SIGFaR es un sistema nuevo compuesto por:

- **Aplicación web (React):** Dirección, veterinarios, bióloga y administración.
- **Aplicación móvil (Flutter):** cuidadores en los recintos.
- **API REST (ASP.NET Core 8):** lógica de negocio, seguridad y tareas programadas.
- **Base de datos (SQL Server):** almacenamiento transaccional.
- **Caché (Redis):** caché de consultas frecuentes y reportes.
- **Sistemas externos:** almacenamiento de respaldos fuera del Bioparque y servicio de notificaciones (correo / push).

### 2.2 Funciones del producto
Registro de ingresos con código único · historia clínica · alertas sanitarias · movimientos entre recintos · dietas y raciones · inventario por lote · egresos · control de acceso por roles · reporte mensual · costeo por especie e individuo.

### 2.3 Clases de usuario

| Rol | Stakeholder | Uso principal | Frecuencia |
|---|---|---|---|
| Directora | Lic. Carla Mamani | Consulta general, reportes | Semanal |
| Veterinario | Dr. Rodrigo Choque | Historia clínica, tratamientos, alertas | Diaria |
| Bióloga | Lic. Andrea Salazar | Ingresos, movimientos, dietas, egresos | Diaria |
| Cuidador | Personal de recintos | Consulta de fichas, observaciones, raciones (móvil) | Diaria |
| Administrador-Contador | Lic. Jorge Huanca | Inventario, costos, reportes | Semanal |
| Administrador del sistema | Soporte TI | Usuarios y roles | Ocasional |

### 2.4 Entorno operativo
- Web: navegadores Chrome, Edge y Firefox en sus dos últimas versiones.
- Móvil: Android 10 o superior.
- Servidor: .NET 8, SQL Server 2022, Redis 7.
- Conectividad irregular en algunas zonas de Mallasa (Bióloga, P7).

### 2.5 Restricciones de diseño e implementación
- Stack obligatorio: React, Flutter, ASP.NET Core 8, SQL Server, Redis.
- Comunicación HTTPS (TLS 1.2 o superior) con API REST/JSON.
- Autenticación con JWT.
- Interfaz en español; zona horaria BOT (UTC-4); fechas en formato dd/mm/aaaa.

### 2.6 Supuestos y dependencias
- El Bioparque cuenta con conexión a internet estable en oficinas y clínica.
- Existe un proveedor de almacenamiento externo para respaldos.
- La Bióloga y el Veterinario entregan muestras de actas, fichas clínicas y el Excel de inventario (COM-05) para migración y validación.

---

## 3. Requisitos específicos

### 3.1 Interfaces externas
- **Usuario:** aplicación web responsiva y aplicación móvil.
- **Software:** servicio de notificaciones (correo/push) y almacenamiento externo de respaldos.
- **Comunicaciones:** HTTPS entre clientes y API; conexión cifrada entre API y base de datos.

### 3.2 Requisitos funcionales

Estado: **Validado verbalmente** = confirmado por el stakeholder en ACT-SIS213-BIO-001, pendiente de aprobación formal (COM-06).

---

#### RF-01 — Registrar ingreso de animal

| Campo | Valor |
|---|---|
| Descripción | El sistema deberá registrar el ingreso de un animal con procedencia (POFOMA, ciudadano, otro centro u otro), fecha, especie, estado inicial y acta de recepción digitalizada, generando automáticamente un código SIGFaR único. |
| Fuente | NEC-01, NEC-02 · Directora (P2), Bióloga (P5, P9) |
| Prioridad | Must |
| Estado | Validado verbalmente |

Criterios de aceptación:
- El código SIGFaR se genera al guardar con formato `SGF-AAAA-NNNNN` y no es editable.
- Microchip y anilla son opcionales, pero únicos: si ya existen, el sistema muestra el individuo registrado y ofrece registrar un reingreso en vez de crear otro individuo.
- El acta admite archivos PDF, JPG o PNG de hasta 10 MB.

#### RF-02 — Gestionar historia clínica

| Campo | Valor |
|---|---|
| Descripción | El sistema deberá registrar y consultar la historia clínica de un individuo (consultas, diagnósticos, tratamientos y exámenes) en orden cronológico, incluyendo ingresos anteriores del mismo individuo. |
| Fuente | NEC-03 · Veterinario (P3) |
| Prioridad | Must |
| Estado | Validado verbalmente |

Criterios de aceptación:
- Búsqueda por código SIGFaR, microchip, anilla o nombre.
- Se muestran los registros de todos los ingresos del individuo.
- Se pueden adjuntar resultados de exámenes.

#### RF-03 — Generar alertas sanitarias

| Campo | Valor |
|---|---|
| Descripción | El sistema deberá generar alertas de vacunas, desparasitaciones y fin de cuarentena que venzan en los próximos 7 días y notificarlas a los veterinarios. |
| Fuente | NEC-04 · Veterinario (P4) |
| Prioridad | Must |
| Estado | Validado verbalmente |

Criterios de aceptación:
- Una tarea programada evalúa las fechas de próxima aplicación una vez al día.
- La cuarentena es de 30 días por defecto y configurable por especie.
- Una alerta vencida permanece destacada hasta que se registra la aplicación.

#### RF-04 — Registrar movimientos entre recintos

| Campo | Valor |
|---|---|
| Descripción | El sistema deberá registrar cada traslado de un individuo entre recintos con fecha-hora de entrada y salida, sin modificar ni eliminar movimientos anteriores. |
| Fuente | NEC-05 · Bióloga (P6) |
| Prioridad | Must |
| Estado | Validado verbalmente |

Criterios de aceptación:
- Al registrar un movimiento nuevo, el anterior se cierra automáticamente con su fecha de salida.
- Un individuo no puede estar en dos recintos al mismo tiempo.
- Se puede consultar en qué recinto estaba un individuo en una fecha dada.

#### RF-05 — Controlar inventario de insumos

| Campo | Valor |
|---|---|
| Descripción | El sistema deberá registrar el ingreso de medicamentos y alimento por lote (vencimiento y costo unitario), descontar stock al registrar un consumo asociado a un individuo y alertar cuando el stock alcance el punto de reorden. |
| Fuente | NEC-08 · Contador (P10) |
| Prioridad | Must |
| Estado | Validado verbalmente |

Criterios de aceptación:
- El descuento sigue el criterio FEFO.
- No se permite stock negativo.
- Cuando el stock es menor o igual al punto de reorden, se notifica al Administrador-Contador.

#### RF-06 — Generar reporte mensual consolidado

| Campo | Valor |
|---|---|
| Descripción | El sistema deberá generar el reporte mensual de ingresos, egresos, decesos, ocupación de recintos y gastos, exportable a PDF y Excel. |
| Fuente | NEC-12 · Directora (P12, P13) |
| Prioridad | Must |
| Estado | Validado verbalmente |

Criterios de aceptación:
- El usuario selecciona mes y año.
- Los totales coinciden con los registros transaccionales del período.
- El reporte se genera en menos de 60 segundos (Directora, P13).

#### RF-07 — Registrar egreso

| Campo | Valor |
|---|---|
| Descripción | El sistema deberá registrar el destino final de un individuo: liberación (coordenadas del sitio), traslado (centro de destino y documento de respaldo), permanencia definitiva o deceso (informe de necropsia). |
| Fuente | NEC-07 · Bióloga (P9) |
| Prioridad | Must |
| Estado | Validado verbalmente |

Criterios de aceptación:
- Los campos obligatorios dependen del tipo de egreso.
- Al registrar liberación, traslado o deceso se cierra el movimiento de recinto activo.
- Un individuo egresado no admite nuevos consumos ni tratamientos.

#### RF-08 — Controlar acceso por roles

| Campo | Valor |
|---|---|
| Descripción | El sistema deberá autenticar a los usuarios y restringir las operaciones según su rol: solo el Veterinario crea o modifica diagnósticos y tratamientos; el Cuidador consulta fichas y registra observaciones. |
| Fuente | NEC-09 · Veterinario (P4.1) |
| Prioridad | Must |
| Estado | Validado verbalmente |

Criterios de aceptación:
- Roles: Directora, Veterinario, Bióloga, Cuidador, Administrador-Contador, Administrador del sistema.
- Una operación no permitida responde HTTP 403 y la acción no aparece en la interfaz.

#### RF-09 — Gestionar dietas y raciones

| Campo | Valor |
|---|---|
| Descripción | El sistema deberá permitir definir dietas por especie o por individuo (la dieta individual prevalece) y registrar diariamente la ración servida y el sobrante. |
| Fuente | NEC-06 · Bióloga (P8) |
| Prioridad | Should |
| Estado | Validado verbalmente |

Criterios de aceptación:
- La cocina consulta la lista de raciones del día agrupada por recinto.
- El sistema calcula el porcentaje de sobrante por individuo y día.

#### RF-10 — Calcular costo por especie e individuo

| Campo | Valor |
|---|---|
| Descripción | El sistema deberá calcular el costo mensual por especie e individuo a partir de los consumos de alimento y medicamento registrados con su costo unitario. |
| Fuente | NEC-13 · Contador (P11) |
| Prioridad | Should |
| Estado | Validado verbalmente |

Criterios de aceptación:
- Costo = Σ (cantidad consumida × costo unitario del lote).
- Filtrable por mes y especie; exportable a Excel.

---

### 3.3 Requisitos no funcionales

| ID | Categoría | Descripción | Métrica cuantitativa | Fuente | Prioridad | Estado |
|---|---|---|---|---|---|---|
| RNF-01 | Rendimiento | La consulta de la historia clínica completa de un individuo debe responder rápidamente. | p95 ≤ 2 s con 20 usuarios concurrentes y una base de 2 000 individuos con 5 años de historial | Veterinario (P3) | Must | Validado verbalmente |
| RNF-02 | Seguridad | Toda modificación en tablas clínicas, de inventario y de egresos queda auditada y la bitácora no es editable. | 100 % de altas, cambios y bajas registrados con usuario, fecha-hora, valor anterior y nuevo; 0 roles con permiso de modificar la bitácora; contraseñas con hash y JWT con expiración ≤ 60 min | Directora (P14), Veterinario (P4.1) | Must | Validado verbalmente |
| RNF-03 | Disponibilidad | Respaldo automático fuera de las instalaciones del Bioparque. | Respaldo diario (RPO ≤ 24 h); restauración ≤ 4 h (RTO); disponibilidad ≥ 99 % de 07:00 a 19:00 | Directora (P14) | Must | Validado verbalmente (el 99 % es propuesta del equipo) |
| RNF-04 | Usabilidad | Un cuidador sin capacitación previa registra una observación desde el celular. | ≤ 60 s y ≤ 3 pantallas, logrado por al menos 8 de 10 cuidadores de prueba | Bióloga (P7) | Should | Validado verbalmente |
| RNF-05 | Mantenibilidad | El backend se organiza en capas (Controllers, Services, Repositories) y se verifica en cada Pull Request. | Cobertura de pruebas unitarias ≥ 70 % en Services; complejidad ciclomática ≤ 10 por método; 0 issues críticos en el análisis estático antes de integrar a `main` | Propuesta del equipo | Should | Propuesto |

**Cambios respecto a la minuta (v0.1 → v1.0)**

| ID en minuta | ID en SRS v1.0 | Motivo |
|---|---|---|
| RNF-01 | RNF-01 | Se agregaron condiciones de carga para hacerlo medible |
| RNF-02 (reporte < 60 s) | Criterio de aceptación de RF-06 | Afecta a una sola función; se libera espacio para mantenibilidad |
| RNF-03 | RNF-02 | Reordenado por categoría |
| RNF-04 | RNF-03 | Se agregaron RPO y disponibilidad |
| RNF-05 | RNF-04 | Se agregaron número de pantallas y criterio de éxito |
| — | RNF-05 | Nuevo: la práctica exige un RNF de mantenibilidad |

### 3.4 Requisitos lógicos de datos
Entidades núcleo: ESPECIE, INDIVIDUO, INGRESO, EGRESO, RECINTO, MOVIMIENTO_RECINTO, HISTORIA_CLINICA, TRATAMIENTO, DIETA, INSUMO, LOTE, CONSUMO, USUARIO, ROL y BITACORA. El modelo E-R detallado se entrega en COM-02.

---

## 4. Matriz de atributos

**Escalas**
- **Riesgo** (probabilidad × impacto de fallar o cambiar): Alto, Medio, Bajo.
- **Estabilidad:** Firme (no se espera cambio) · Estable (cambios menores) · Cambiante (reglas aún en definición).
- **Método de verificación (ISO 29148):** Prueba, Inspección, Análisis, Demostración.

| ID | Prioridad | Riesgo | Justificación del riesgo | Estabilidad | Verificación |
|---|---|---|---|---|---|
| RF-01 | Must | Alto | Todo el sistema depende del código único; un error rompe la integridad | Firme | Prueba funcional |
| RF-02 | Must | Medio | Volumen de historial y adjuntos | Firme | Prueba funcional |
| RF-03 | Must | Alto | Depende de tarea programada y servicio externo de notificaciones; reglas de cuarentena por especie | Cambiante | Prueba de integración |
| RF-04 | Must | Medio | Regla de no solapamiento de fechas | Firme | Prueba funcional |
| RF-05 | Must | Alto | Concurrencia en el descuento de stock y lógica FEFO | Firme | Prueba de integración |
| RF-06 | Must | Medio | Depende de la calidad de los datos TPS; el formato puede ajustarse | Estable | Inspección + prueba de rendimiento |
| RF-07 | Must | Bajo | Reglas claras por tipo de egreso | Firme | Prueba funcional |
| RF-08 | Must | Alto | Falla de seguridad expone datos clínicos | Firme | Prueba de seguridad |
| RF-09 | Should | Medio | Proceso de cocina aún manual; requiere cambio de hábitos | Cambiante | Demostración |
| RF-10 | Should | Medio | Exige que todos los consumos se registren | Estable | Análisis (comparación con cálculo manual) |
| RNF-01 | Must | Medio | Crecimiento del historial | Firme | Prueba de carga |
| RNF-02 | Must | Alto | Requisito legal y de confianza del cliente | Firme | Inspección de BD |
| RNF-03 | Must | Alto | Depende de proveedor externo | Firme | Prueba de recuperación |
| RNF-04 | Should | Medio | Señal irregular en Mallasa | Estable | Prueba de usabilidad |
| RNF-05 | Should | Bajo | Controlado por el propio equipo | Estable | Análisis estático |

---

## 5. Matriz de trazabilidad bidireccional

### 5.1 Hacia adelante: Requisito → Fuente → Historia Jira → Componente C4 → Caso de prueba

| Requisito | Fuente | HU Jira | Contenedor C4 (C2) | Componente C4 (C3) | Caso de prueba |
|---|---|---|---|---|---|
| RF-01 | NEC-01, NEC-02 · P2, P5, P9 | SIGFAR-1, SIGFAR-2 | Web React, API | IngresoController → IngresoService → IndividuoRepository; CodigoSigfarGenerator | CP-01, CP-02 |
| RF-02 | NEC-03 · P3 | SIGFAR-3 | Web React, API | HistoriaClinicaController → HistoriaClinicaService → HistoriaClinicaRepository | CP-03 |
| RF-03 | NEC-04 · P4 | SIGFAR-4 | API | AlertaSchedulerService → NotificacionAdapter | CP-04 |
| RF-04 | NEC-05 · P6 | SIGFAR-5 | Web React, App Flutter, API | MovimientoController → MovimientoService → MovimientoRepository | CP-05 |
| RF-05 | NEC-08 · P10 | SIGFAR-6, SIGFAR-7 | Web React, API | InventarioController → InventarioService → LoteRepository | CP-06, CP-07 |
| RF-06 | NEC-12 · P12, P13 | SIGFAR-8 | Web React, API, Redis | ReporteController → ReporteService → ExportAdapter | CP-08 |
| RF-07 | NEC-07 · P9 | SIGFAR-9 | Web React, API | EgresoController → EgresoService → IndividuoRepository | CP-09 |
| RF-08 | NEC-09 · P4.1 | SIGFAR-10 | API | AuthController, JwtMiddleware | CP-10 |
| RF-09 | NEC-06 · P8 | SIGFAR-11 | App Flutter, API | DietaController → DietaService → DietaRepository | CP-11 |
| RF-10 | NEC-13 · P11 | SIGFAR-12 | Web React, API | ReporteController → CosteoService → ConsumoRepository | CP-12 |
| RNF-01 | P3 | SIGFAR-3 | API, SQL Server, Redis | HistoriaClinicaRepository, RedisCacheAdapter | CP-13 |
| RNF-02 | NEC-09, NEC-11 · P4.1, P14 | SIGFAR-13 | API, SQL Server | AuditInterceptor, JwtMiddleware | CP-14 |
| RNF-03 | NEC-11 · P14 | SIGFAR-14 | SQL Server, Respaldo externo | BackupAdapter | CP-15 |
| RNF-04 | NEC-10 · P7 | SIGFAR-15 | App Flutter, API | ObservacionController | CP-16 |
| RNF-05 | Propuesta del equipo | SIGFAR-16 | API | Todas las capas del backend | CP-17 |

### 5.2 Hacia atrás: Necesidad → Requisito

| Necesidad | Requisito(s) | Cobertura |
|---|---|---|
| NEC-01 | RF-01 | Completa |
| NEC-02 | RF-01 | Completa |
| NEC-03 | RF-02, RNF-01 | Completa |
| NEC-04 | RF-03 | Completa |
| NEC-05 | RF-04 | Completa |
| NEC-06 | RF-09 | Completa |
| NEC-07 | RF-07 | Completa |
| NEC-08 | RF-05 | Completa |
| NEC-09 | RF-08, RNF-02 | Completa |
| NEC-10 | RNF-04 | Parcial: el modo sin conexión se difiere a v1.1 |
| NEC-11 | RNF-02, RNF-03 | Completa |
| NEC-12 | RF-06 | Completa |
| NEC-13 | RF-10 | Completa |
| NEC-14 | — | Diferida a v1.1 |
| NEC-15 | — | Diferida a v1.1 |
| NEC-16 | — | Diferida a v1.1 (los datos los genera RF-04) |

### 5.3 Hacia atrás: Problema AS-IS → Requisito

| Problema | Requisito(s) |
|---|---|
| PRB-01 Sin identificador único | RF-01 |
| PRB-02 Historia clínica en cuadernos | RF-02, RNF-01, RNF-03 |
| PRB-03 Movimientos solo en pizarra | RF-04 |
| PRB-04 Tratamientos sin alertas | RF-03 |
| PRB-05 Inventario desactualizado | RF-05 |
| PRB-06 Informes armados a mano | RF-06 |
| PRB-07 Costos no asignados | RF-10 |
| PRB-08 Un solo editor del Excel | RF-08, RNF-04 |

La trazabilidad Caso de prueba → Requisito se encuentra en el Apéndice B.

---

## 6. Requisitos diferidos (backlog v1.1)

| Necesidad | MoSCoW | Motivo del diferimiento |
|---|---|---|
| NEC-10 Operación móvil sin conexión | Could | Requiere estrategia de sincronización y resolución de conflictos; en v1.0 la app funciona en línea |
| NEC-14 Informe anual para la autoridad ambiental | Should | Pendiente recibir el formato oficial; se construirá sobre RF-06 |
| NEC-15 Tablero de indicadores | Could | Depende de acumular datos del TPS |
| NEC-16 Rastreo de contactos | Could | Se implementa como consulta sobre los datos de RF-04 |

---

## Apéndice A — Historias de usuario (Jira)

| Clave | Historia | Requisito |
|---|---|---|
| SIGFAR-1 | Como bióloga, quiero registrar el ingreso de un animal para que tenga un código único desde el primer día | RF-01 |
| SIGFAR-2 | Como bióloga, quiero que el sistema detecte un microchip o anilla ya registrados para evitar duplicados | RF-01 |
| SIGFAR-3 | Como veterinario, quiero consultar la historia clínica completa de un individuo para decidir su tratamiento | RF-02, RNF-01 |
| SIGFAR-4 | Como veterinario, quiero recibir alertas de vacunas, desparasitaciones y cuarentenas para no atrasarlas | RF-03 |
| SIGFAR-5 | Como bióloga, quiero registrar el cambio de recinto para conservar el historial de ubicación | RF-04 |
| SIGFAR-6 | Como contador, quiero registrar lotes de insumos para conocer el stock real | RF-05 |
| SIGFAR-7 | Como contador, quiero ser alertado al llegar al punto de reorden para evitar compras de emergencia | RF-05 |
| SIGFAR-8 | Como directora, quiero generar el reporte mensual para rendir cuentas al municipio | RF-06 |
| SIGFAR-9 | Como bióloga, quiero registrar el destino final de un animal con su respaldo | RF-07 |
| SIGFAR-10 | Como veterinario, quiero que solo mi rol pueda cambiar diagnósticos para proteger la información clínica | RF-08 |
| SIGFAR-11 | Como bióloga, quiero definir dietas y registrar sobrantes para usarlos como indicador de salud | RF-09 |
| SIGFAR-12 | Como contador, quiero conocer el costo mensual por especie para justificar el presupuesto | RF-10 |
| SIGFAR-13 | Como directora, quiero saber quién modificó cada dato para garantizar la responsabilidad | RNF-02 |
| SIGFAR-14 | Como directora, quiero respaldos diarios externos para no perder información | RNF-03 |
| SIGFAR-15 | Como cuidador, quiero registrar una observación desde el celular en menos de un minuto | RNF-04 |
| SIGFAR-16 | Como equipo de desarrollo, queremos un pipeline de calidad para mantener el código | RNF-05 |

## Apéndice B — Casos de prueba

| ID | Requisito | Tipo | Escenario | Resultado esperado |
|---|---|---|---|---|
| CP-01 | RF-01 | Funcional | Registrar un ingreso con datos válidos | Se genera un código `SGF-2026-NNNNN` no editable |
| CP-02 | RF-01 | Funcional | Registrar un ingreso con un microchip existente | Se impide el duplicado y se muestra el individuo existente |
| CP-03 | RF-02 | Funcional | Consultar un individuo con dos ingresos | Se muestra el historial de ambos en orden cronológico |
| CP-04 | RF-03 | Integración | Tratamientos con próxima aplicación a 5 y a 8 días; ejecutar la tarea programada | Solo se genera alerta para el de 5 días |
| CP-05 | RF-04 | Funcional | Mover un individuo del recinto A al B | El movimiento en A se cierra; existen dos registros en el historial |
| CP-06 | RF-05 | Integración | Consumir 2 unidades con dos lotes disponibles | Se descuenta del lote que vence primero |
| CP-07 | RF-05 | Integración | Consumo que deja el stock en el punto de reorden; consumo mayor al stock | Se alerta al contador; el segundo consumo se rechaza |
| CP-08 | RF-06 | Funcional / rendimiento | Generar el reporte de un mes con datos de prueba | Totales iguales al conteo en BD; PDF y Excel descargables; < 60 s |
| CP-09 | RF-07 | Funcional | Registrar una liberación sin coordenadas y luego con coordenadas | La primera se rechaza; la segunda cambia el estado y cierra el recinto activo |
| CP-10 | RF-08 | Seguridad | Un cuidador intenta modificar un diagnóstico | HTTP 403 |
| CP-11 | RF-09 | Funcional | Registrar ración de 500 g con sobrante de 100 g | Sobrante calculado: 20 % |
| CP-12 | RF-10 | Análisis | Calcular el costo mensual con consumos de prueba | Coincide con el cálculo manual |
| CP-13 | RNF-01 | Carga | 20 usuarios virtuales durante 10 min consultando historias clínicas | p95 ≤ 2 s |
| CP-14 | RNF-02 | Inspección de BD | Modificar un diagnóstico e intentar editar la bitácora | Registro con usuario, fecha-hora y valores; edición de bitácora denegada |
| CP-15 | RNF-03 | Recuperación | Restaurar el respaldo del día anterior en un servidor limpio | Sistema operativo en ≤ 4 h |
| CP-16 | RNF-04 | Usabilidad | 10 cuidadores sin capacitación registran una observación | Al menos 8 lo logran en ≤ 60 s |
| CP-17 | RNF-05 | Análisis estático | Ejecutar el pipeline de CI sobre un Pull Request | Cobertura ≥ 70 %, complejidad ≤ 10, 0 issues críticos |

---

## Aprobación

| Nombre | Cargo | Firma | Fecha |
|---|---|---|---|
| Lic. Carla Mamani Quispe | Directora, Bioparque Municipal Vesty Pakos | | |
| Dr. Rodrigo Choque Villca | Jefe de Clínica Veterinaria | | |
| Gemina Ponce | Director de Proyecto, Consultora UCB | | |
