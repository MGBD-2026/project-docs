# 10 — Conclusiones Técnicas

## Decisiones de diseño y su justificación

### 1. Separación por schemas en lugar de prefijos de tabla

Se optó por usar 7 schemas separados (`security`, `clientes`, `catalogo`, `promociones`, `ventas`, `logistica`, `inventario`) en lugar de un único schema con prefijos en los nombres de tabla. Esto permite:

- Aplicar permisos granulares por schema (RLS y GRANTs diferenciados).
- Agrupar lógicamente las tablas por dominio funcional.
- Facilitar la migración futura a microservicios: cada schema puede convertirse en una base de datos independiente.

### 2. Liquibase para migraciones

Liquibase resolvió los tres problemas principales de gestión de base de datos en equipo:
- **Reproducibilidad:** cualquier persona puede levantar la BD desde cero con `docker compose up -d`.
- **Versionado:** cada cambio queda registrado en la tabla `DATABASECHANGELOG` con autor, fecha y hash.
- **Rollback controlado:** cada changeset tiene su script de reversión espejo en `05_rollbacks/`.

**Problema encontrado:** Los cuerpos PL/pgSQL en funciones no son compatibles con `splitStatements: true` porque Liquibase intenta dividir el SQL en los `;` internos del cuerpo. **Solución aplicada:** usar `splitStatements: false` en los changesets de funciones y procedures.

### 3. Triggers para automatizar el stock

Se eligió implementar la actualización de stock mediante triggers en lugar de lógica de aplicación porque:
- Garantiza que el stock se actualice sin importar desde qué capa se inserte el movimiento.
- Centraliza la regla de negocio en la base de datos, evitando inconsistencias.
- Los tres triggers (`on_insert`, `on_update`, `on_delete`) cubren todos los casos posibles de modificación de movimientos.

### 4. Columna generada para `subtotal`

`ventas.detalle_pedido.subtotal` se define como `GENERATED ALWAYS AS (cantidad * precio_unitario) STORED`. Esto garantiza que el subtotal siempre sea matemáticamente correcto y elimina la posibilidad de que la aplicación inserte un subtotal incorrecto.

### 5. Historial de estados separado del pedido

En lugar de sobrescribir el estado del pedido, se mantiene un `historial_estado_pedido` separado. Esto permite auditoría completa de la trayectoria de cada pedido y no impacta el rendimiento de las consultas de pedidos activos.

### 6. Precio almacenado en detalle de pedido y carrito

El `precio_unitario` en `detalle_pedido` e `item_carrito` se guarda al momento de la operación, independiente del precio actual del producto. Esto evita que cambios futuros de precio en el catálogo alteren el historial de ventas.

---

## Problemas encontrados y soluciones aplicadas

| Problema | Solución |
|---|---|
| `$$` en funciones PL/pgSQL genera error `Unterminated dollar quote` en Liquibase | Cambiar cuerpos de funciones y procedures a comillas simples `'...'` y usar `splitStatements: false` |
| Funciones con `'` internas necesitan escapar las comillas | Usar `''` (dos comillas simples) para representar una comilla dentro del cuerpo |
| `docker-compose.yml` faltaba para el ambiente de desarrollo | Creado con configuración de puerto 5432 y credenciales hardcodeadas para desarrollo |
| `.env.example` faltaba para producción | Creado con la variable `DB_PASSWORD` requerida por `docker-compose.prod.yml` |

---

## Resumen de entregables técnicos

| Elemento                    | Cantidad | Archivo(s)                                      |
|-----------------------------|----------|-------------------------------------------------|
| Schemas                     | 7        | `01_ddl/01_schemas/001_create_schemas.sql`      |
| Tablas                      | 17       | `01_ddl/03_tables/*.sql`                        |
| Vistas                      | 10       | `01_ddl/04_views/001_report_views.sql`          |
| Functions                   | 3        | `01_ddl/06_functions/001_update_stock_functions.sql` |
| Procedures                  | 2        | `01_ddl/07_procedures/001_gestion_pedidos_procedures.sql` |
| Triggers                    | 3        | `01_ddl/08_triggers/001_inventario_triggers.sql`|
| Índices                     | 35       | `01_ddl/09_indexes/001_performance_indexes.sql` |
| Datos canónicos             | ~25 reg. | `02_dml/00_inserts/001_initial_data.sql`        |
| Datos volumétricos          | 130+ reg.| `02_dml/00_inserts/002_volumetric_data.sql`     |
| Consultas JOIN (6+ tablas)  | 3        | `scripts/consultas-join-validacion.sql`         |
| Políticas RLS               | 12       | `03_dcl/02_policies/001_rls_policies.sql`       |
| Scripts rollback            | 20       | `05_rollbacks/`                                 |
| Ambientes Docker            | 3        | `docker-compose.yml`, `docker-compose.prod.yml` |
| Changesets Liquibase        | 20       | `changelog-master.yaml` + subcarpetas           |
