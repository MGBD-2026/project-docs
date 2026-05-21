# 09 — Evidencias

## Instrucciones para reproducir todas las evidencias

Desde la raíz de `project-bd/`:

```bash
# 1. Levantar la base de datos y ejecutar todas las migraciones
docker compose up -d

# 2. Ver el estado de las migraciones
docker compose run --rm liquibase status

# 3. Validar la estructura de los changelogs
docker compose run --rm liquibase validate
```

---

## Evidencia 1 — Docker levanta correctamente

```bash
docker compose up -d
docker ps
```

Resultado esperado:
```
CONTAINER ID   IMAGE                  COMMAND                  STATUS
xxxxxxxxxxxx   postgres:16-alpine     "docker-entrypoint.s…"   Up (healthy)
xxxxxxxxxxxx   liquibase/liquibase:…  "liquibase update ..."   Exited (0)
```

El contenedor de Liquibase sale con código 0 (éxito) tras aplicar las migraciones.

---

## Evidencia 2 — Migraciones aplicadas (`liquibase update`)

```bash
docker compose run --rm liquibase update
```

Resultado esperado:
```
UPDATE SUMMARY
Run:            17
Previously run:  0
Filtered out:    0
-------------------------------
Total change sets: 17

Liquibase command 'update' was executed successfully.
```

Log completo disponible en: `project-bd/liquibase_log.txt`

---

## Evidencia 3 — Schemas y tablas creados

```bash
docker exec -it accesorios-dm-postgres-dev \
  psql -U admin -d accesorios_dm_db -c "\dn"
```

Resultado esperado:
```
      List of schemas
     Name      |  Owner
---------------+--------
 catalogo      | admin
 clientes      | admin
 inventario    | admin
 logistica     | admin
 promociones   | admin
 public        | pg_database_owner
 security      | admin
 ventas        | admin
```

---

## Evidencia 4 — Datos canónicos cargados

```bash
docker exec -it accesorios-dm-postgres-dev \
  psql -U admin -d accesorios_dm_db \
  -c "SELECT nombre FROM security.rol;"
```

Resultado esperado: `ADMIN`, `VENDEDOR`, `BODEGUERO`, `CLIENTE`

```bash
docker exec -it accesorios-dm-postgres-dev \
  psql -U admin -d accesorios_dm_db \
  -c "SELECT nombre FROM logistica.estado_pedido;"
```

Resultado esperado: `PENDIENTE`, `PAGADO`, `ENVIADO`, `ENTREGADO`, `CANCELADO`

---

## Evidencia 5 — Datos volumétricos cargados (+100 registros)

```bash
docker exec -it accesorios-dm-postgres-dev \
  psql -U admin -d accesorios_dm_db -c "
  SELECT
    (SELECT COUNT(*) FROM catalogo.categoria)            AS categorias,
    (SELECT COUNT(*) FROM catalogo.material)             AS materiales,
    (SELECT COUNT(*) FROM catalogo.producto)             AS productos,
    (SELECT COUNT(*) FROM clientes.cliente)              AS clientes,
    (SELECT COUNT(*) FROM security.empleado)             AS empleados,
    (SELECT COUNT(*) FROM ventas.pedido)                 AS pedidos,
    (SELECT COUNT(*) FROM ventas.detalle_pedido)         AS detalles_pedido,
    (SELECT COUNT(*) FROM logistica.historial_estado_pedido) AS historiales,
    (SELECT COUNT(*) FROM inventario.inventario_movimiento)  AS movimientos
  ;"
```

Resultado esperado (mínimos):

| categorias | materiales | productos | clientes | empleados | pedidos | detalles_pedido | historiales | movimientos |
|---|---|---|---|---|---|---|---|---|
| 8 | 8 | 21 | 11 | 5 | 15 | 15 | 45+ | 15 |

---

## Evidencia 6 — Trigger de inventario funciona

```bash
docker exec -it accesorios-dm-postgres-dev \
  psql -U admin -d accesorios_dm_db -c "
  SELECT stock FROM catalogo.producto WHERE nombre = 'Anillo Solitario Oro 18K';
  INSERT INTO inventario.inventario_movimiento (cantidad, referencia, id_producto, id_tipo_movimiento)
  VALUES (10, 'Prueba trigger', 
    (SELECT id_producto FROM catalogo.producto WHERE nombre = 'Anillo Solitario Oro 18K'),
    (SELECT id_tipo_movimiento FROM inventario.tipo_movimiento WHERE nombre = 'ENTRADA'));
  SELECT stock FROM catalogo.producto WHERE nombre = 'Anillo Solitario Oro 18K';
  "
```

Resultado esperado: el stock aumenta en 10 unidades automáticamente.

---

## Evidencia 7 — Procedure funciona

```bash
docker exec -it accesorios-dm-postgres-dev \
  psql -U admin -d accesorios_dm_db -c "
  CALL ventas.sp_actualizar_estado_pedido(1, 
    (SELECT id_estado FROM logistica.estado_pedido WHERE nombre = 'PAGADO'),
    'Pago verificado manualmente');
  SELECT id_estado_actual FROM ventas.pedido WHERE id_pedido = 1;
  SELECT * FROM logistica.historial_estado_pedido WHERE id_pedido = 1 ORDER BY fecha_cambio DESC LIMIT 1;
  "
```

---

## Evidencia 8 — Consulta JOIN de 7 tablas

```bash
docker exec -it accesorios-dm-postgres-dev \
  psql -U admin -d accesorios_dm_db \
  -f /liquibase/changelog/scripts/consultas-join-validacion.sql
```

Ver script completo: `project-bd/scripts/consultas-join-validacion.sql`

---

## Evidencia 9 — Rollback ejecutado

```bash
# Revertir el último changeset (datos volumétricos)
docker compose run --rm liquibase rollback-count --count=1

# Verificar que los datos fueron eliminados
docker exec -it accesorios-dm-postgres-dev \
  psql -U admin -d accesorios_dm_db \
  -c "SELECT COUNT(*) FROM clientes.cliente;"
```

Resultado esperado después del rollback: `1` (solo el cliente demo)

```bash
# Re-aplicar
docker compose run --rm liquibase update
```

Ver detalle completo: `project-bd/docs/evidencia-rollback.md`

---

## Evidencia 10 — Verificación completa por ambiente

Scripts de verificación disponibles en `project-bd/scripts/`:

```bash
# Windows
.\scripts\verify-develop.ps1
.\scripts\verify-qa.ps1
.\scripts\verify-main.ps1
```
