# 01 — Planteamiento del Problema

## Título del proyecto

**Accesorios DM** — Sistema de base de datos para comercio electrónico de accesorios.

## Descripción del dominio

Accesorios DM es una empresa que vende accesorios (anillos, collares, pulseras, aretes, cadenas, broches y tobilleras) a través de una tienda en línea. Los productos se fabrican en distintos materiales (oro, plata, acero, titanio, cobre) y se clasifican por categorías.

## Problema a representar

La empresa no cuenta con un sistema centralizado de base de datos que permita:

1. Registrar y consultar el catálogo de productos con su categoría y material.
2. Gestionar clientes, carritos de compra y pedidos.
3. Controlar el inventario y registrar movimientos de entrada, salida y ajuste de stock.
4. Aplicar promociones con vigencia temporal a productos específicos.
5. Hacer seguimiento del ciclo de vida de un pedido (estados: pendiente → pagado → enviado → entregado).
6. Separar accesos según el rol del usuario (administrador, vendedor, bodeguero, cliente).

## Procesos del dominio

| Proceso                   | Actor            | Descripción                                                    |
|---------------------------|------------------|----------------------------------------------------------------|
| Registrar producto        | Administrador    | Crear un producto con categoría, material, precio y stock.     |
| Consultar catálogo        | Cliente          | Ver productos disponibles con precio y estado.                 |
| Agregar al carrito        | Cliente          | Seleccionar productos y cantidades antes de confirmar pedido.  |
| Generar pedido            | Cliente          | Confirmar el carrito, registrar dirección y teléfono.          |
| Registrar pago            | Vendedor         | Marcar pedido como pagado y registrar en historial.            |
| Despachar pedido          | Vendedor         | Marcar pedido como enviado.                                    |
| Registrar movimiento      | Bodeguero        | Registrar entradas, salidas y ajustes de inventario.           |
| Aplicar promoción         | Administrador    | Vincular una promoción con descuento a productos específicos.  |
| Consultar historial       | Administrador    | Ver todos los cambios de estado de un pedido.                  |

## Objetivo del proyecto

Diseñar, implementar y versionar una base de datos relacional en PostgreSQL que soporte todos los procesos del dominio de Accesorios DM, usando Liquibase para migraciones, Docker para reproducibilidad, y políticas RLS para seguridad.

## Alcance

- Base de datos PostgreSQL con 7 schemas y 17 tablas.
- Migraciones versionadas con Liquibase (DDL, DML, DCL).
- Tres ambientes: desarrollo (puerto 5432), QA (5433), producción (5434).
- Datos canónicos y volumétricos (+100 registros).
- Funciones, procedures, triggers, vistas e índices.
- Políticas Row Level Security (RLS) por rol.
