# 06 — Modelo Lógico

## Cardinalidades y normalización

El modelo está en **Tercera Forma Normal (3FN)**:
- Cada tabla tiene clave primaria (`PK`).
- No hay dependencias parciales (todos los atributos dependen de la PK completa).
- No hay dependencias transitivas (los atributos no-clave dependen solo de la PK).

---

## Tablas y relaciones con cardinalidades

### security

| Tabla     | PK          | FK                        | Cardinalidad           |
|-----------|-------------|---------------------------|------------------------|
| rol       | id_rol      | —                         | —                      |
| empleado  | id_empleado | id_rol → rol.id_rol       | N empleados : 1 rol    |

### clientes

| Tabla   | PK         | FK | Cardinalidad |
|---------|------------|----|--------------|
| cliente | id_cliente | —  | —            |

### catalogo

| Tabla           | PK          | FK                                                   | Cardinalidad                   |
|-----------------|-------------|------------------------------------------------------|--------------------------------|
| categoria       | id_categoria| —                                                    | —                              |
| material        | id_material | —                                                    | —                              |
| producto        | id_producto | id_categoria → categoria, id_material → material     | N productos : 1 cat : 1 mat    |
| imagen_producto | id_imagen   | id_producto → producto                               | N imágenes : 1 producto        |

### promociones

| Tabla              | PK                    | FK                                              | Cardinalidad                  |
|--------------------|-----------------------|-------------------------------------------------|-------------------------------|
| promocion          | id_promocion          | —                                               | —                             |
| promocion_producto | id_promocion_producto | id_promocion → promocion, id_producto → producto| N:M (tabla puente)            |

### ventas

| Tabla          | PK          | FK                                                          | Cardinalidad                        |
|----------------|-------------|-------------------------------------------------------------|-------------------------------------|
| carrito        | id_carrito  | id_cliente → cliente                                        | N carritos : 1 cliente              |
| item_carrito   | id_item     | id_carrito → carrito, id_producto → producto                | N ítems : 1 carrito : 1 producto    |
| pedido         | id_pedido   | id_cliente → cliente, id_estado_actual → estado_pedido      | N pedidos : 1 cliente : 1 estado    |
| detalle_pedido | id_detalle  | id_pedido → pedido, id_producto → producto                  | N detalles : 1 pedido : 1 producto  |

### logistica

| Tabla                  | PK           | FK                                             | Cardinalidad                     |
|------------------------|--------------|------------------------------------------------|----------------------------------|
| estado_pedido          | id_estado    | —                                              | —                                |
| historial_estado_pedido| id_historial | id_pedido → pedido, id_estado → estado_pedido  | N historiales : 1 pedido         |

### inventario

| Tabla                 | PK              | FK                                                           | Cardinalidad                       |
|-----------------------|-----------------|--------------------------------------------------------------|------------------------------------|
| tipo_movimiento       | id_tipo_movimiento | —                                                         | —                                  |
| inventario_movimiento | id_movimiento   | id_producto → producto, id_tipo_movimiento → tipo_movimiento | N movimientos : 1 producto         |

---

## Decisiones de normalización destacadas

| Decisión | Justificación |
|---|---|
| `detalle_pedido.precio_unitario` guarda el precio al momento de la compra, separado de `catalogo.producto.precio` | Evita que cambios futuros de precio alteren el historial de ventas |
| `item_carrito.precio_unitario` idem para el carrito | Coherencia con el precio vigente cuando se agregó el ítem |
| `promocion_producto.precio_promocional` almacena el precio calculado | Permite auditoría sin recalcular el descuento futuro |
| `detalle_pedido.subtotal` es columna generada | Garantiza que `subtotal = cantidad * precio_unitario` siempre sea correcto, sin redundancia editable |
| `historial_estado_pedido` separada de `pedido` | Permite auditoría completa sin modificar el registro principal del pedido |
