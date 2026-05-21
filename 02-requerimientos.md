# 02 — Requerimientos

## Requerimientos funcionales

| ID   | Requerimiento                                                                 |
|------|-------------------------------------------------------------------------------|
| RF01 | El sistema debe permitir registrar productos con nombre, descripción, precio, stock, categoría y material. |
| RF02 | El sistema debe clasificar productos por categoría y material.                |
| RF03 | El sistema debe registrar clientes con nombre, correo y teléfono.             |
| RF04 | El sistema debe gestionar carritos de compra activos por cliente.             |
| RF05 | El sistema debe permitir agregar ítems a un carrito con cantidad y precio.    |
| RF06 | El sistema debe registrar pedidos con dirección de envío, teléfono y total.   |
| RF07 | El sistema debe registrar el detalle de cada pedido (producto, cantidad, precio unitario, subtotal). |
| RF08 | El sistema debe llevar un historial de cambios de estado por pedido.          |
| RF09 | El sistema debe registrar movimientos de inventario (entrada, salida, ajuste) por producto. |
| RF10 | El sistema debe actualizar automáticamente el stock al registrar un movimiento de inventario. |
| RF11 | El sistema debe permitir definir promociones con porcentaje de descuento y vigencia temporal. |
| RF12 | El sistema debe vincular promociones a productos con precio promocional específico. |
| RF13 | El sistema debe controlar el acceso por rol (ADMIN, VENDEDOR, BODEGUERO, CLIENTE). |
| RF14 | El sistema debe proporcionar vistas para reportes de ventas, inventario y pedidos. |

## Requerimientos no funcionales relacionados con base de datos

| ID    | Requerimiento                                                                              |
|-------|--------------------------------------------------------------------------------------------|
| RNF01 | La base de datos debe ejecutarse en PostgreSQL 16 dentro de un contenedor Docker.          |
| RNF02 | Todos los cambios de estructura deben estar versionados con Liquibase.                     |
| RNF03 | Cada changeset debe tener su script de rollback correspondiente.                           |
| RNF04 | El stock de un producto no puede ser negativo (regla de negocio en trigger).               |
| RNF05 | Las cantidades en pedidos y carritos deben ser positivas (restricción CHECK).              |
| RNF06 | Los precios y totales deben ser positivos (restricción CHECK).                             |
| RNF07 | El correo de clientes y empleados debe ser único (restricción UNIQUE).                     |
| RNF08 | Los índices deben cubrir las consultas frecuentes (búsqueda por cliente, producto, fecha). |
| RNF09 | Las políticas RLS deben impedir que un cliente vea datos de otro cliente.                  |
| RNF10 | La base de datos debe ser reproducible: cualquier persona con Docker puede levantarla desde cero con un solo comando. |
