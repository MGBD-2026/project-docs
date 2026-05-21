# 07 — Modelo Físico

## Motor de base de datos

- **Motor:** PostgreSQL 16
- **Extensión:** `uuid-ossp` (habilitada para soporte futuro de UUIDs)
- **Schemas:** 7 (`security`, `clientes`, `catalogo`, `promociones`, `ventas`, `logistica`, `inventario`)

---

## Tablas, tipos, claves y restricciones

### security.rol

| Columna        | Tipo           | Restricciones                    |
|----------------|----------------|----------------------------------|
| id_rol         | SERIAL         | PRIMARY KEY                      |
| nombre         | VARCHAR(50)    | NOT NULL, UNIQUE                 |
| descripcion    | TEXT           | —                                |
| fecha_creacion | TIMESTAMP      | NOT NULL, DEFAULT CURRENT_TIMESTAMP |

### security.empleado

| Columna        | Tipo           | Restricciones                              |
|----------------|----------------|--------------------------------------------|
| id_empleado    | SERIAL         | PRIMARY KEY                                |
| nombre         | VARCHAR(100)   | NOT NULL                                   |
| correo         | VARCHAR(120)   | NOT NULL, UNIQUE                           |
| password       | VARCHAR(255)   | NOT NULL                                   |
| fecha_creacion | TIMESTAMP      | NOT NULL, DEFAULT CURRENT_TIMESTAMP        |
| estado         | BOOLEAN        | NOT NULL, DEFAULT TRUE                     |
| id_rol         | INTEGER        | NOT NULL, FK → security.rol(id_rol)        |

### clientes.cliente

| Columna         | Tipo         | Restricciones                       |
|-----------------|--------------|-------------------------------------|
| id_cliente      | SERIAL       | PRIMARY KEY                         |
| nombre          | VARCHAR(100) | NOT NULL                            |
| correo          | VARCHAR(120) | NOT NULL, UNIQUE                    |
| telefono        | VARCHAR(20)  | —                                   |
| fecha_registro  | TIMESTAMP    | NOT NULL, DEFAULT CURRENT_TIMESTAMP |

### catalogo.categoria

| Columna      | Tipo        | Restricciones         |
|--------------|-------------|-----------------------|
| id_categoria | SERIAL      | PRIMARY KEY           |
| nombre       | VARCHAR(80) | NOT NULL, UNIQUE      |
| descripcion  | TEXT        | —                     |
| estado       | BOOLEAN     | NOT NULL, DEFAULT TRUE|

### catalogo.material

| Columna     | Tipo        | Restricciones    |
|-------------|-------------|------------------|
| id_material | SERIAL      | PRIMARY KEY      |
| nombre      | VARCHAR(80) | NOT NULL, UNIQUE |
| descripcion | TEXT        | —                |

### catalogo.producto

| Columna        | Tipo           | Restricciones                              |
|----------------|----------------|--------------------------------------------|
| id_producto    | SERIAL         | PRIMARY KEY                                |
| nombre         | VARCHAR(150)   | NOT NULL                                   |
| descripcion    | TEXT           | —                                          |
| precio         | NUMERIC(10,2)  | NOT NULL                                   |
| stock          | INTEGER        | NOT NULL, DEFAULT 0                        |
| fecha_creacion | TIMESTAMP      | NOT NULL, DEFAULT CURRENT_TIMESTAMP        |
| estado         | BOOLEAN        | NOT NULL, DEFAULT TRUE                     |
| id_categoria   | INTEGER        | NOT NULL, FK → catalogo.categoria          |
| id_material    | INTEGER        | NOT NULL, FK → catalogo.material           |

### catalogo.imagen_producto

| Columna     | Tipo    | Restricciones                    |
|-------------|---------|----------------------------------|
| id_imagen   | SERIAL  | PRIMARY KEY                      |
| url_imagen  | TEXT    | NOT NULL                         |
| orden       | INTEGER | DEFAULT 1                        |
| id_producto | INTEGER | NOT NULL, FK → catalogo.producto |

### promociones.promocion

| Columna              | Tipo          | Restricciones                       |
|----------------------|---------------|-------------------------------------|
| id_promocion         | SERIAL        | PRIMARY KEY                         |
| nombre               | VARCHAR(100)  | NOT NULL                            |
| descripcion          | TEXT          | —                                   |
| porcentaje_descuento | NUMERIC(5,2)  | NOT NULL                            |
| fecha_inicio         | TIMESTAMP     | NOT NULL                            |
| fecha_fin            | TIMESTAMP     | NOT NULL                            |
| activo               | BOOLEAN       | NOT NULL, DEFAULT TRUE              |
| fecha_creacion       | TIMESTAMP     | NOT NULL, DEFAULT CURRENT_TIMESTAMP |

### promociones.promocion_producto

| Columna               | Tipo          | Restricciones                         |
|-----------------------|---------------|---------------------------------------|
| id_promocion_producto | SERIAL        | PRIMARY KEY                           |
| precio_promocional    | NUMERIC(10,2) | NOT NULL                              |
| id_promocion          | INTEGER       | NOT NULL, FK → promociones.promocion  |
| id_producto           | INTEGER       | NOT NULL, FK → catalogo.producto      |

### ventas.carrito

| Columna        | Tipo        | Restricciones                                              |
|----------------|-------------|------------------------------------------------------------|
| id_carrito     | SERIAL      | PRIMARY KEY                                                |
| fecha_creacion | TIMESTAMP   | NOT NULL, DEFAULT CURRENT_TIMESTAMP                        |
| estado         | VARCHAR(20) | NOT NULL, DEFAULT 'activo', CHECK IN ('activo','procesado','abandonado') |
| id_cliente     | INTEGER     | NOT NULL, FK → clientes.cliente                            |

### ventas.item_carrito

| Columna        | Tipo          | Restricciones                        |
|----------------|---------------|--------------------------------------|
| id_item_carrito| SERIAL        | PRIMARY KEY                          |
| cantidad       | INTEGER       | NOT NULL, CHECK (cantidad > 0)       |
| precio_unitario| NUMERIC(10,2) | NOT NULL, CHECK (precio_unitario > 0)|
| id_carrito     | INTEGER       | NOT NULL, FK → ventas.carrito        |
| id_producto    | INTEGER       | NOT NULL, FK → catalogo.producto     |

### logistica.estado_pedido

| Columna     | Tipo        | Restricciones    |
|-------------|-------------|------------------|
| id_estado   | SERIAL      | PRIMARY KEY      |
| nombre      | VARCHAR(50) | NOT NULL, UNIQUE |
| descripcion | TEXT        | —                |

### ventas.pedido

| Columna           | Tipo          | Restricciones                           |
|-------------------|---------------|-----------------------------------------|
| id_pedido         | SERIAL        | PRIMARY KEY                             |
| direccion_envio   | VARCHAR(255)  | NOT NULL                                |
| telefono_contacto | VARCHAR(20)   | NOT NULL                                |
| total             | NUMERIC(10,2) | NOT NULL, CHECK (total > 0)             |
| fecha_pedido      | TIMESTAMP     | NOT NULL, DEFAULT CURRENT_TIMESTAMP     |
| id_cliente        | INTEGER       | NOT NULL, FK → clientes.cliente         |
| id_estado_actual  | INTEGER       | NOT NULL, FK → logistica.estado_pedido  |

### ventas.detalle_pedido

| Columna         | Tipo          | Restricciones                                  |
|-----------------|---------------|------------------------------------------------|
| id_detalle      | SERIAL        | PRIMARY KEY                                    |
| cantidad        | INTEGER       | NOT NULL, CHECK (cantidad > 0)                 |
| precio_unitario | NUMERIC(10,2) | NOT NULL, CHECK (precio_unitario > 0)          |
| subtotal        | NUMERIC(10,2) | GENERATED ALWAYS AS (cantidad * precio_unitario) STORED |
| id_pedido       | INTEGER       | NOT NULL, FK → ventas.pedido (CASCADE DELETE)  |
| id_producto     | INTEGER       | NOT NULL, FK → catalogo.producto               |

### logistica.historial_estado_pedido

| Columna      | Tipo      | Restricciones                                   |
|--------------|-----------|-------------------------------------------------|
| id_historial | SERIAL    | PRIMARY KEY                                     |
| fecha_cambio | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP             |
| observacion  | TEXT      | —                                               |
| id_pedido    | INTEGER   | NOT NULL, FK → ventas.pedido (CASCADE DELETE)   |
| id_estado    | INTEGER   | NOT NULL, FK → logistica.estado_pedido          |

### inventario.tipo_movimiento

| Columna           | Tipo        | Restricciones    |
|-------------------|-------------|------------------|
| id_tipo_movimiento| SERIAL      | PRIMARY KEY      |
| nombre            | VARCHAR(50) | NOT NULL, UNIQUE |
| descripcion       | TEXT        | —                |

### inventario.inventario_movimiento

| Columna           | Tipo         | Restricciones                                   |
|-------------------|--------------|-------------------------------------------------|
| id_movimiento     | SERIAL       | PRIMARY KEY                                     |
| cantidad          | INTEGER      | NOT NULL, CHECK (cantidad != 0)                 |
| fecha_movimiento  | TIMESTAMP    | NOT NULL, DEFAULT CURRENT_TIMESTAMP             |
| referencia        | VARCHAR(100) | —                                               |
| id_producto       | INTEGER      | NOT NULL, FK → catalogo.producto                |
| id_tipo_movimiento| INTEGER      | NOT NULL, FK → inventario.tipo_movimiento       |

---

## Objetos SQL implementados

| Tipo       | Nombre                                    | Descripción                                    |
|------------|-------------------------------------------|------------------------------------------------|
| Function   | inventario.f_update_stock_on_insert       | Suma cantidad al stock en INSERT               |
| Function   | inventario.f_update_stock_on_update       | Ajusta stock en UPDATE de movimiento           |
| Function   | inventario.f_revert_stock_on_delete       | Revierte stock al eliminar movimiento          |
| Procedure  | ventas.sp_actualizar_estado_pedido        | Cambia estado y registra historial             |
| Procedure  | ventas.sp_vaciar_carrito                  | Vacía ítems y marca carrito como abandonado    |
| Trigger    | trg_update_stock_on_insert                | AFTER INSERT en inventario_movimiento          |
| Trigger    | trg_update_stock_on_update                | AFTER UPDATE en inventario_movimiento          |
| Trigger    | trg_revert_stock_on_delete                | BEFORE DELETE en inventario_movimiento         |
| Vista      | catalogo.vw_producto_detalle              | Productos con categoría y material             |
| Vista      | promociones.vw_producto_promocion_activa  | Productos con promociones vigentes             |
| Vista      | ventas.vw_pedido_cliente                  | Pedidos con datos del cliente                  |
| Vista      | ventas.vw_pedido_detalle_producto         | Detalle completo de pedidos con productos      |
| Vista      | logistica.vw_pedido_historial_estados     | Historial de estados por pedido                |
| Vista      | ventas.vw_carrito_activo_cliente          | Carritos activos con resumen                   |
| Vista      | inventario.vw_movimientos_producto        | Movimientos de inventario por producto         |
| Vista      | inventario.vw_producto_bajo_stock         | Productos con stock bajo o crítico             |
| Vista      | ventas.vw_ventas_por_mes                  | Resumen de ventas por mes                      |
| Vista      | ventas.vw_top_productos_vendidos          | Top 10 productos más vendidos                  |

---

## Índices

35 índices de rendimiento definidos en `01_ddl/09_indexes/001_performance_indexes.sql`, cubriendo: búsqueda por correo, nombre, categoría, material, precio, estado, fecha y combinaciones compuestas frecuentes.
