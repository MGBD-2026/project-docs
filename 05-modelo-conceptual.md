# 05 — Modelo Conceptual

## Entidades y relaciones de alto nivel

El modelo conceptual del sistema Accesorios DM identifica **17 entidades** agrupadas en 7 dominios funcionales.

---

## Dominio: Seguridad

```
ROL ──(1:N)── EMPLEADO
```

- Un **ROL** agrupa empleados con los mismos permisos del sistema.
- Un **EMPLEADO** tiene exactamente un rol y usa sus credenciales para operar.

---

## Dominio: Clientes

```
CLIENTE ──(1:N)── CARRITO
CLIENTE ──(1:N)── PEDIDO
```

- Un **CLIENTE** puede tener múltiples carritos a lo largo del tiempo (activos, procesados, abandonados).
- Un **CLIENTE** puede realizar múltiples pedidos.

---

## Dominio: Catálogo

```
CATEGORIA ──(1:N)── PRODUCTO
MATERIAL  ──(1:N)── PRODUCTO
PRODUCTO  ──(1:N)── IMAGEN_PRODUCTO
```

- Un **PRODUCTO** pertenece a una **CATEGORIA** y está fabricado con un **MATERIAL**.
- Un producto puede tener varias **IMÁGENES** para mostrar en la tienda.

---

## Dominio: Promociones

```
PROMOCION ──(N:M)── PRODUCTO  →  PROMOCION_PRODUCTO
```

- Una **PROMOCION** puede aplicarse a varios productos.
- Un **PRODUCTO** puede tener varias promociones en distintos períodos.
- La tabla intermedia **PROMOCION_PRODUCTO** guarda el precio promocional específico.

---

## Dominio: Ventas

```
CLIENTE ──(1:N)── CARRITO ──(1:N)── ITEM_CARRITO ──(N:1)── PRODUCTO
CLIENTE ──(1:N)── PEDIDO  ──(1:N)── DETALLE_PEDIDO ──(N:1)── PRODUCTO
PEDIDO  ──(N:1)── ESTADO_PEDIDO
```

- Un **CARRITO** tiene ítems. Cada **ITEM_CARRITO** referencia un producto.
- Un **PEDIDO** tiene detalles. Cada **DETALLE_PEDIDO** registra producto, cantidad y precio al momento de la compra.
- El **ESTADO_PEDIDO** actual del pedido (PENDIENTE, PAGADO, ENVIADO, ENTREGADO, CANCELADO).

---

## Dominio: Logística

```
PEDIDO ──(1:N)── HISTORIAL_ESTADO_PEDIDO ──(N:1)── ESTADO_PEDIDO
```

- Cada cambio de estado de un pedido queda registrado en el **HISTORIAL**, con fecha y observación.

---

## Dominio: Inventario

```
TIPO_MOVIMIENTO ──(1:N)── INVENTARIO_MOVIMIENTO ──(N:1)── PRODUCTO
```

- Cada **MOVIMIENTO** de inventario clasifica la operación (ENTRADA, SALIDA, AJUSTE) y afecta el stock del producto automáticamente vía trigger.
