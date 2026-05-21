# 08 — Diccionario de Datos

## Convenciones

| Símbolo | Significado            |
|---------|------------------------|
| PK      | Llave primaria         |
| FK      | Llave foránea          |
| NN      | NOT NULL               |
| UQ      | UNIQUE                 |
| CK      | CHECK                  |
| GEN     | GENERATED (columna calculada) |

---

## security.rol

Almacena los roles del sistema que agrupan empleados con los mismos permisos.

| Campo          | Tipo        | Restricción       | Descripción                                   |
|----------------|-------------|-------------------|-----------------------------------------------|
| id_rol         | SERIAL      | PK                | Identificador autoincremental del rol         |
| nombre         | VARCHAR(50) | NN, UQ            | Nombre del rol (ADMIN, VENDEDOR, BODEGUERO, CLIENTE) |
| descripcion    | TEXT        | —                 | Descripción del rol                           |
| fecha_creacion | TIMESTAMP   | NN, DEFAULT NOW() | Fecha de creación del registro                |

---

## security.empleado

Almacena los empleados que operan el sistema.

| Campo          | Tipo         | Restricción              | Descripción                          |
|----------------|--------------|--------------------------|--------------------------------------|
| id_empleado    | SERIAL       | PK                       | Identificador autoincremental        |
| nombre         | VARCHAR(100) | NN                       | Nombre completo del empleado         |
| correo         | VARCHAR(120) | NN, UQ                   | Correo para autenticación            |
| password       | VARCHAR(255) | NN                       | Contraseña (encriptada en producción)|
| fecha_creacion | TIMESTAMP    | NN, DEFAULT NOW()        | Fecha de registro                    |
| estado         | BOOLEAN      | NN, DEFAULT TRUE         | TRUE = activo, FALSE = inactivo      |
| id_rol         | INTEGER      | NN, FK → security.rol    | Rol asignado al empleado             |

---

## clientes.cliente

Almacena los clientes que compran en la tienda.

| Campo          | Tipo         | Restricción | Descripción                          |
|----------------|--------------|-------------|--------------------------------------|
| id_cliente     | SERIAL       | PK          | Identificador autoincremental        |
| nombre         | VARCHAR(100) | NN          | Nombre completo del cliente          |
| correo         | VARCHAR(120) | NN, UQ      | Correo electrónico del cliente       |
| telefono       | VARCHAR(20)  | —           | Teléfono de contacto                 |
| fecha_registro | TIMESTAMP    | NN, DEFAULT NOW() | Fecha de registro en la tienda  |

---

## catalogo.categoria

Clasificación de los productos de la tienda.

| Campo        | Tipo        | Restricción     | Descripción                              |
|--------------|-------------|-----------------|------------------------------------------|
| id_categoria | SERIAL      | PK              | Identificador autoincremental            |
| nombre       | VARCHAR(80) | NN, UQ          | Nombre de la categoría (Anillos, Collares, etc.) |
| descripcion  | TEXT        | —               | Descripción de la categoría              |
| estado       | BOOLEAN     | NN, DEFAULT TRUE| TRUE = activa en catálogo                |

---

## catalogo.material

Materiales con que se fabrican los productos.

| Campo       | Tipo        | Restricción | Descripción                          |
|-------------|-------------|-------------|--------------------------------------|
| id_material | SERIAL      | PK          | Identificador autoincremental        |
| nombre      | VARCHAR(80) | NN, UQ      | Nombre del material (Oro 18K, Plata 925, etc.) |
| descripcion | TEXT        | —           | Descripción del material             |

---

## catalogo.producto

Productos disponibles en el catálogo de la tienda.

| Campo          | Tipo          | Restricción                     | Descripción                                    |
|----------------|---------------|---------------------------------|------------------------------------------------|
| id_producto    | SERIAL        | PK                              | Identificador autoincremental                  |
| nombre         | VARCHAR(150)  | NN                              | Nombre del producto                            |
| descripcion    | TEXT          | —                               | Descripción detallada                          |
| precio         | NUMERIC(10,2) | NN                              | Precio de venta en COP                         |
| stock          | INTEGER       | NN, DEFAULT 0                   | Unidades disponibles. Actualizado por trigger  |
| fecha_creacion | TIMESTAMP     | NN, DEFAULT NOW()               | Fecha de alta en el catálogo                   |
| estado         | BOOLEAN       | NN, DEFAULT TRUE                | TRUE = visible para clientes                   |
| id_categoria   | INTEGER       | NN, FK → catalogo.categoria     | Categoría del producto                         |
| id_material    | INTEGER       | NN, FK → catalogo.material      | Material del producto                          |

---

## catalogo.imagen_producto

Imágenes asociadas a un producto para la tienda en línea.

| Campo       | Tipo    | Restricción                    | Descripción                                   |
|-------------|---------|--------------------------------|-----------------------------------------------|
| id_imagen   | SERIAL  | PK                             | Identificador autoincremental                 |
| url_imagen  | TEXT    | NN                             | URL de la imagen (CDN o almacenamiento)       |
| orden       | INTEGER | DEFAULT 1                      | Orden de visualización (1 = imagen principal) |
| id_producto | INTEGER | NN, FK → catalogo.producto     | Producto al que pertenece la imagen           |

---

## promociones.promocion

Promociones con descuento aplicables a productos durante un período.

| Campo                | Tipo          | Restricción     | Descripción                            |
|----------------------|---------------|-----------------|----------------------------------------|
| id_promocion         | SERIAL        | PK              | Identificador autoincremental          |
| nombre               | VARCHAR(100)  | NN              | Nombre de la promoción                 |
| descripcion          | TEXT          | —               | Descripción de la promoción            |
| porcentaje_descuento | NUMERIC(5,2)  | NN              | Porcentaje de descuento (ej: 15.00)    |
| fecha_inicio         | TIMESTAMP     | NN              | Inicio de la vigencia                  |
| fecha_fin            | TIMESTAMP     | NN              | Fin de la vigencia                     |
| activo               | BOOLEAN       | NN, DEFAULT TRUE| FALSE = desactivada manualmente        |
| fecha_creacion       | TIMESTAMP     | NN, DEFAULT NOW()| Fecha de creación del registro        |

---

## promociones.promocion_producto

Tabla puente que relaciona promociones con los productos específicos que aplican.

| Campo                 | Tipo          | Restricción                        | Descripción                              |
|-----------------------|---------------|------------------------------------|------------------------------------------|
| id_promocion_producto | SERIAL        | PK                                 | Identificador autoincremental            |
| precio_promocional    | NUMERIC(10,2) | NN                                 | Precio con descuento aplicado            |
| id_promocion          | INTEGER       | NN, FK → promociones.promocion     | Promoción asociada                       |
| id_producto           | INTEGER       | NN, FK → catalogo.producto         | Producto en promoción                    |

---

## ventas.carrito

Carrito de compras de un cliente en la tienda.

| Campo          | Tipo        | Restricción                              | Descripción                                       |
|----------------|-------------|------------------------------------------|---------------------------------------------------|
| id_carrito     | SERIAL      | PK                                       | Identificador autoincremental                     |
| fecha_creacion | TIMESTAMP   | NN, DEFAULT NOW()                        | Fecha de creación del carrito                     |
| estado         | VARCHAR(20) | NN, DEFAULT 'activo', CK IN (...)        | activo / procesado / abandonado                   |
| id_cliente     | INTEGER     | NN, FK → clientes.cliente                | Cliente propietario del carrito                   |

---

## ventas.item_carrito

Productos agregados al carrito antes de confirmar el pedido.

| Campo           | Tipo          | Restricción                   | Descripción                                |
|-----------------|---------------|-------------------------------|--------------------------------------------|
| id_item_carrito | SERIAL        | PK                            | Identificador autoincremental              |
| cantidad        | INTEGER       | NN, CK (cantidad > 0)         | Unidades del producto en el carrito        |
| precio_unitario | NUMERIC(10,2) | NN, CK (precio_unitario > 0)  | Precio del producto al momento de agregar  |
| id_carrito      | INTEGER       | NN, FK → ventas.carrito       | Carrito al que pertenece el ítem           |
| id_producto     | INTEGER       | NN, FK → catalogo.producto    | Producto seleccionado                      |

---

## logistica.estado_pedido

Catálogo de los estados posibles de un pedido.

| Campo       | Tipo        | Restricción | Descripción                                     |
|-------------|-------------|-------------|-------------------------------------------------|
| id_estado   | SERIAL      | PK          | Identificador autoincremental                   |
| nombre      | VARCHAR(50) | NN, UQ      | PENDIENTE, PAGADO, ENVIADO, ENTREGADO, CANCELADO|
| descripcion | TEXT        | —           | Descripción del estado                          |

---

## ventas.pedido

Pedido confirmado generado por un cliente.

| Campo             | Tipo          | Restricción                        | Descripción                                   |
|-------------------|---------------|------------------------------------|-----------------------------------------------|
| id_pedido         | SERIAL        | PK                                 | Identificador autoincremental                 |
| direccion_envio   | VARCHAR(255)  | NN                                 | Dirección de entrega del pedido               |
| telefono_contacto | VARCHAR(20)   | NN                                 | Teléfono de contacto para el envío            |
| total             | NUMERIC(10,2) | NN, CK (total > 0)                 | Valor total del pedido en COP                 |
| fecha_pedido      | TIMESTAMP     | NN, DEFAULT NOW()                  | Fecha y hora en que se generó el pedido       |
| id_cliente        | INTEGER       | NN, FK → clientes.cliente          | Cliente que realizó el pedido                 |
| id_estado_actual  | INTEGER       | NN, FK → logistica.estado_pedido   | Estado actual del pedido                      |

---

## ventas.detalle_pedido

Línea de producto dentro de un pedido confirmado.

| Campo           | Tipo          | Restricción                             | Descripción                                        |
|-----------------|---------------|-----------------------------------------|----------------------------------------------------|
| id_detalle      | SERIAL        | PK                                      | Identificador autoincremental                      |
| cantidad        | INTEGER       | NN, CK (cantidad > 0)                   | Unidades del producto en el pedido                 |
| precio_unitario | NUMERIC(10,2) | NN, CK (precio_unitario > 0)            | Precio del producto al momento de la venta         |
| subtotal        | NUMERIC(10,2) | GEN (cantidad * precio_unitario) STORED | Subtotal calculado automáticamente                 |
| id_pedido       | INTEGER       | NN, FK → ventas.pedido (CASCADE DELETE) | Pedido al que pertenece este detalle               |
| id_producto     | INTEGER       | NN, FK → catalogo.producto              | Producto vendido                                   |

---

## logistica.historial_estado_pedido

Registro cronológico de cada cambio de estado de un pedido.

| Campo        | Tipo      | Restricción                                     | Descripción                              |
|--------------|-----------|-------------------------------------------------|------------------------------------------|
| id_historial | SERIAL    | PK                                              | Identificador autoincremental            |
| fecha_cambio | TIMESTAMP | NN, DEFAULT NOW()                               | Fecha y hora del cambio de estado        |
| observacion  | TEXT      | —                                               | Nota del operador sobre el cambio        |
| id_pedido    | INTEGER   | NN, FK → ventas.pedido (CASCADE DELETE)         | Pedido al que pertenece el historial     |
| id_estado    | INTEGER   | NN, FK → logistica.estado_pedido                | Estado registrado en este momento        |

---

## inventario.tipo_movimiento

Catálogo de tipos de movimiento de inventario.

| Campo              | Tipo        | Restricción | Descripción                        |
|--------------------|-------------|-------------|------------------------------------|
| id_tipo_movimiento | SERIAL      | PK          | Identificador autoincremental      |
| nombre             | VARCHAR(50) | NN, UQ      | ENTRADA, SALIDA, AJUSTE            |
| descripcion        | TEXT        | —           | Descripción del tipo de movimiento |

---

## inventario.inventario_movimiento

Registro de cada movimiento de stock de un producto.

| Campo              | Tipo         | Restricción                                    | Descripción                                         |
|--------------------|--------------|------------------------------------------------|-----------------------------------------------------|
| id_movimiento      | SERIAL       | PK                                             | Identificador autoincremental                       |
| cantidad           | INTEGER      | NN, CK (cantidad != 0)                         | Positivo = entrada, negativo = salida               |
| fecha_movimiento   | TIMESTAMP    | NN, DEFAULT NOW()                              | Fecha y hora del movimiento                         |
| referencia         | VARCHAR(100) | —                                              | Referencia del movimiento (Pedido #X, Compra #Y)    |
| id_producto        | INTEGER      | NN, FK → catalogo.producto                     | Producto afectado por el movimiento                 |
| id_tipo_movimiento | INTEGER      | NN, FK → inventario.tipo_movimiento            | Tipo de movimiento aplicado                         |
