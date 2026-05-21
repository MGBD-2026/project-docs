# 03 — Reglas de Negocio

## Catálogo

| ID   | Regla                                                                                      | Implementación                          |
|------|--------------------------------------------------------------------------------------------|-----------------------------------------|
| RN01 | Un producto debe pertenecer a exactamente una categoría y un material.                     | FK `NOT NULL` en `catalogo.producto`    |
| RN02 | El precio de un producto debe ser mayor a cero.                                            | No hay CHECK directo en producto, se valida en DML |
| RN03 | El nombre de una categoría y de un material deben ser únicos en el sistema.                | `UNIQUE` en `nombre`                    |
| RN04 | Un producto inactivo (`estado = FALSE`) no debe aparecer en catálogos de venta.            | Filtro en vistas y consultas            |

## Inventario

| ID   | Regla                                                                                      | Implementación                          |
|------|--------------------------------------------------------------------------------------------|-----------------------------------------|
| RN05 | Todo movimiento de inventario debe afectar el stock del producto automáticamente.          | Triggers `trg_update_stock_on_insert/update/delete` |
| RN06 | La cantidad de un movimiento no puede ser cero.                                            | `CHECK (cantidad != 0)` en `inventario_movimiento` |
| RN07 | Un movimiento debe clasificarse como ENTRADA, SALIDA o AJUSTE.                             | FK a `inventario.tipo_movimiento`       |

## Carrito y Pedidos

| ID   | Regla                                                                                      | Implementación                          |
|------|--------------------------------------------------------------------------------------------|-----------------------------------------|
| RN08 | Un carrito solo puede estar en estado `activo`, `procesado` o `abandonado`.               | `CHECK` en `ventas.carrito`             |
| RN09 | La cantidad de ítems en un carrito y en un pedido debe ser mayor a cero.                  | `CHECK (cantidad > 0)` en ambas tablas  |
| RN10 | El precio unitario en carrito y pedido debe ser mayor a cero.                             | `CHECK (precio_unitario > 0)`           |
| RN11 | El total de un pedido debe ser mayor a cero.                                              | `CHECK (total > 0)` en `ventas.pedido`  |
| RN12 | El subtotal de un detalle de pedido se calcula siempre como `cantidad * precio_unitario`. | Columna `GENERATED ALWAYS AS ... STORED`|
| RN13 | Un pedido debe registrar el estado inicial al crearse.                                    | Procedure `sp_actualizar_estado_pedido` |
| RN14 | Cada cambio de estado de un pedido debe quedar registrado en el historial.                | Procedure `sp_actualizar_estado_pedido` |
| RN15 | Eliminar un pedido elimina en cascada su detalle e historial de estados.                  | `ON DELETE CASCADE` en FK              |

## Promociones

| ID   | Regla                                                                                      | Implementación                          |
|------|--------------------------------------------------------------------------------------------|-----------------------------------------|
| RN16 | Una promoción tiene fecha de inicio y fecha de fin obligatorias.                          | `NOT NULL` en `promociones.promocion`   |
| RN17 | Solo las promociones con `activo = TRUE` y dentro de fechas vigentes se muestran al cliente.| Filtro en vista `vw_producto_promocion_activa` |

## Seguridad

| ID   | Regla                                                                                      | Implementación                          |
|------|--------------------------------------------------------------------------------------------|-----------------------------------------|
| RN18 | Un cliente solo puede ver sus propios pedidos, carritos y datos personales.               | Políticas RLS en tablas de `ventas` y `clientes` |
| RN19 | Un vendedor puede ver todos los pedidos y clientes, pero no puede modificar la estructura. | `GRANT SELECT` + políticas RLS         |
| RN20 | Un bodeguero solo puede acceder al schema de inventario.                                  | `GRANT USAGE` solo en schema `inventario` |
