# 09 — Evidencias

**Integrante responsable:** Sergio Losada  
**Fecha de ejecución:** 2026-05-24

## Instrucciones para reproducir todas las evidencias

Desde la raíz de `project-bd/`:

```bash
# 1. Levantar la base de datos y ejecutar todas las migraciones
docker compose up -d

# 2. Ver el estado de las migraciones
docker compose run --rm liquibase \
  status \
  --url=jdbc:postgresql://postgres:5432/accesorios_dm_db \
  --username=admin --password=admin123 \
  --changelog-file=./changelog-master.yaml

# 3. Validar la estructura de los changelogs
docker compose run --rm liquibase \
  validate \
  --url=jdbc:postgresql://postgres:5432/accesorios_dm_db \
  --username=admin --password=admin123 \
  --changelog-file=./changelog-master.yaml
```

---

## Evidencia 1 — Docker levanta correctamente

**Comando ejecutado:**
```bash
docker compose up -d
docker ps --format "table {{.ID}}\t{{.Image}}\t{{.Command}}\t{{.Status}}\t{{.Ports}}"
```

**Salida real capturada:**
```
CONTAINER ID   IMAGE                COMMAND                  STATUS                        PORTS
ef725f6d42e9   postgres:16-alpine   "docker-entrypoint.s…"   Up About a minute (healthy)   0.0.0.0:5432->5432/tcp, [::]:5432->5432/tcp
```

El contenedor de Liquibase sale con código 0 (éxito) tras aplicar las migraciones.

---

## Evidencia 2 — Migraciones aplicadas (`liquibase update`)

**Comando ejecutado:**
```bash
docker compose run --rm liquibase
```

**Salida real capturada:**
```
Running Changeset: 01_ddl/00_extensions/changelog.yaml::001_enable_uuid_extension::JSA
Running Changeset: 01_ddl/01_schemas/changelog.yaml::001_create_schemas::JSA
Running Changeset: 01_ddl/03_tables/changelog.yaml::001_create_security_tables::JSA
Running Changeset: 01_ddl/03_tables/changelog.yaml::002_create_clientes_tables::JSA
Running Changeset: 01_ddl/03_tables/changelog.yaml::003_create_catalogo_tables::JSA
Running Changeset: 01_ddl/03_tables/changelog.yaml::004_create_imagen_producto_table::JSA
Running Changeset: 01_ddl/03_tables/changelog.yaml::005_create_promociones_tables::JSA
Running Changeset: 01_ddl/03_tables/changelog.yaml::006_create_carrito_tables::JSA
Running Changeset: 01_ddl/03_tables/changelog.yaml::007_create_pedidos_tables::JSA
Running Changeset: 01_ddl/03_tables/changelog.yaml::008_create_inventario_movimiento_table::JSA
Running Changeset: 01_ddl/04_views/changelog.yaml::001_report_views::JSA
Running Changeset: 01_ddl/06_functions/changelog.yaml::001_update_stock_functions::JSA
Running Changeset: 01_ddl/07_procedures/changelog.yaml::001_gestion_pedidos_procedures::JSA
Running Changeset: 01_ddl/08_triggers/changelog.yaml::001_inventario_triggers::JSA
Running Changeset: 01_ddl/09_indexes/changelog.yaml::001_performance_indexes::JSA
Running Changeset: 02_dml/00_inserts/changelog.yaml::001_initial_data::JSA
Running Changeset: 02_dml/00_inserts/changelog.yaml::002_volumetric_data::JSA
Running Changeset: 03_dcl/00_roles/changelog.yaml::001_create_roles::JSA
Running Changeset: 03_dcl/01_grants/changelog.yaml::001_grants::JSA
Running Changeset: 03_dcl/02_policies/changelog.yaml::001_rls_policies::JSA

UPDATE SUMMARY
Run:                         20
Previously run:               0
Filtered out:                 0
-------------------------------
Total change sets:           20

Liquibase: Update has been successful. Rows affected: 197
Liquibase command 'update' was executed successfully.
```

Log completo disponible en: `project-bd/liquibase_log.txt`

---

## Evidencia 3 — Schemas y tablas creados

**Comando ejecutado:**
```bash
docker exec accesorios-dm-postgres-dev \
  psql -U admin -d accesorios_dm_db -c "\dn"
```

**Salida real capturada:**
```
         List of schemas
    Name     |       Owner
-------------+-------------------
 catalogo    | admin
 clientes    | admin
 inventario  | admin
 logistica   | admin
 promociones | admin
 public      | pg_database_owner
 security    | admin
 ventas      | admin
(8 rows)
```

---

## Evidencia 4 — Datos canónicos cargados

**Comando ejecutado:**
```bash
docker exec accesorios-dm-postgres-dev \
  psql -U admin -d accesorios_dm_db \
  -c "SELECT nombre FROM security.rol;"
```

**Salida real capturada:**
```
  nombre
-----------
 ADMIN
 VENDEDOR
 BODEGUERO
 CLIENTE
(4 rows)
```

```bash
docker exec accesorios-dm-postgres-dev \
  psql -U admin -d accesorios_dm_db \
  -c "SELECT nombre FROM logistica.estado_pedido;"
```

**Salida real capturada:**
```
  nombre
-----------
 PENDIENTE
 PAGADO
 ENVIADO
 ENTREGADO
 CANCELADO
(5 rows)
```

---

## Evidencia 5 — Datos volumétricos cargados (+100 registros)

**Comando ejecutado:**
```bash
docker exec accesorios-dm-postgres-dev \
  psql -U admin -d accesorios_dm_db -c "
  SELECT
    (SELECT COUNT(*) FROM catalogo.categoria)                AS categorias,
    (SELECT COUNT(*) FROM catalogo.material)                 AS materiales,
    (SELECT COUNT(*) FROM catalogo.producto)                 AS productos,
    (SELECT COUNT(*) FROM clientes.cliente)                  AS clientes,
    (SELECT COUNT(*) FROM security.empleado)                 AS empleados,
    (SELECT COUNT(*) FROM ventas.pedido)                     AS pedidos,
    (SELECT COUNT(*) FROM ventas.detalle_pedido)             AS detalles_pedido,
    (SELECT COUNT(*) FROM logistica.historial_estado_pedido) AS historiales,
    (SELECT COUNT(*) FROM inventario.inventario_movimiento)  AS movimientos;
  "
```

**Salida real capturada:**
```
 categorias | materiales | productos | clientes | empleados | pedidos | detalles_pedido | historiales | movimientos
------------+------------+-----------+----------+-----------+---------+-----------------+-------------+-------------
          8 |          8 |        21 |       11 |         5 |      15 |              15 |          43 |          15
(1 row)
```

Total de registros: **135+** distribuidos entre todas las tablas del dominio.

---

## Evidencia 6 — Trigger de inventario funciona

**Integrante:** Sergio Losada  
**Archivo:** `01_ddl/08_triggers/001_inventario_triggers.sql`  
**Trigger:** `trg_update_stock_on_insert` → llama `inventario.f_update_stock_on_insert()`

**Comando ejecutado:**
```bash
docker exec accesorios-dm-postgres-dev \
  psql -U admin -d accesorios_dm_db -c "
  SELECT stock FROM catalogo.producto WHERE nombre = 'Anillo Solitario Oro 18K';
  INSERT INTO inventario.inventario_movimiento (cantidad, referencia, id_producto, id_tipo_movimiento)
  VALUES (10, 'Prueba trigger',
    (SELECT id_producto FROM catalogo.producto WHERE nombre = 'Anillo Solitario Oro 18K'),
    (SELECT id_tipo_movimiento FROM inventario.tipo_movimiento WHERE nombre = 'ENTRADA'));
  SELECT stock FROM catalogo.producto WHERE nombre = 'Anillo Solitario Oro 18K';
  "
```

**Salida real capturada:**
```
 stock
-------
    75
(1 row)

INSERT 0 1

 stock
-------
    85
(1 row)
```

El stock pasó de **75** a **85** automáticamente al insertar el movimiento. El trigger funciona correctamente.

---

## Evidencia 7 — Procedure funciona

**Integrante:** Sergio Losada  
**Archivo:** `01_ddl/07_procedures/001_gestion_pedidos_procedures.sql`  
**Procedure:** `ventas.sp_actualizar_estado_pedido`

**Comando ejecutado:**
```bash
docker exec accesorios-dm-postgres-dev \
  psql -U admin -d accesorios_dm_db -c "
  DO \$\$
  DECLARE v_estado_id INT;
  BEGIN
    SELECT id_estado INTO v_estado_id FROM logistica.estado_pedido WHERE nombre = 'PAGADO';
    CALL ventas.sp_actualizar_estado_pedido(1, v_estado_id, 'Pago verificado manualmente');
  END;
  \$\$;
  SELECT p.id_pedido, ep.nombre AS estado_actual
  FROM ventas.pedido p JOIN logistica.estado_pedido ep ON p.id_estado_actual = ep.id_estado
  WHERE p.id_pedido = 1;
  SELECT id_historial, id_pedido, observacion
  FROM logistica.historial_estado_pedido WHERE id_pedido = 1 ORDER BY id_historial DESC LIMIT 1;
  "
```

**Salida real capturada:**
```
DO

 id_pedido | estado_actual
-----------+---------------
         1 | PAGADO
(1 row)

 id_historial | id_pedido |         observacion
--------------+-----------+-----------------------------
           44 |         1 | Pago verificado manualmente
(1 row)
```

---

## Evidencia 8 — Consulta JOIN de 7 tablas

**Integrante:** Sergio Losada  
**Archivo:** `scripts/consultas-join-validacion.sql`  
**Tablas unidas:** `ventas.pedido`, `clientes.cliente`, `logistica.estado_pedido`, `ventas.detalle_pedido`, `catalogo.producto`, `catalogo.categoria`, `catalogo.material` (7 tablas)

**Comando ejecutado:**
```bash
docker exec accesorios-dm-postgres-dev \
  psql -U admin -d accesorios_dm_db \
  -f /liquibase/changelog/scripts/consultas-join-validacion.sql
```

Ver script completo: `project-bd/scripts/consultas-join-validacion.sql`

---

## Evidencia 9 — Rollback ejecutado

**Integrante:** Sergio Losada  
**Archivo rollback:** `05_rollbacks/03_dcl/02_policies/001_rls_policies.rollback.sql`

**Comando ejecutado:**
```bash
docker compose run --rm liquibase \
  rollback-count \
  --url=jdbc:postgresql://postgres:5432/accesorios_dm_db \
  --username=admin --password=admin123 \
  --changelog-file=./changelog-master.yaml \
  --count=1
```

**Salida real capturada:**
```
Rolling Back Changeset: 03_dcl/02_policies/changelog.yaml::001_rls_policies::JSA
Liquibase command 'rollback-count' was executed successfully.
```

**Re-aplicación exitosa:**
```bash
docker compose run --rm liquibase \
  update \
  --url=jdbc:postgresql://postgres:5432/accesorios_dm_db \
  --username=admin --password=admin123 \
  --changelog-file=./changelog-master.yaml
```

```
Running Changeset: 03_dcl/02_policies/changelog.yaml::001_rls_policies::JSA
Run:                          1
Previously run:              19
Total change sets:           20
Liquibase command 'update' was executed successfully.
```

Ver detalle completo: `project-bd/docs/evidencia-rollback.md`

---

## Evidencia 10 — Función SQL

**Integrante:** Sergio Losada  
**Archivo:** `01_ddl/06_functions/001_update_stock_functions.sql`  
**Function:** `inventario.f_update_stock_on_insert()` — actualiza stock al registrar un movimiento de inventario (demostrada en Evidencia 6).
