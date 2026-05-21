# 04 — Diagramas Iniciales

## Diagrama de contexto (actores y procesos)

```
                        ┌──────────────────────────────────┐
                        │       ACCESORIOS DM              │
                        │   Sistema de E-commerce          │
                        └──────────────┬───────────────────┘
                                       │
          ┌────────────────────────────┼───────────────────────────┐
          │                            │                           │
    ┌─────▼──────┐             ┌───────▼───────┐          ┌───────▼────────┐
    │  CLIENTE   │             │   VENDEDOR    │          │  BODEGUERO     │
    │            │             │               │          │                │
    │ - Ver cat. │             │ - Ver pedidos │          │ - Entradas     │
    │ - Carrito  │             │ - Reg. pago   │          │ - Salidas      │
    │ - Pedido   │             │ - Despachar   │          │ - Ajustes      │
    └────────────┘             └───────────────┘          └────────────────┘
          │                            │                           │
          └────────────────────────────┼───────────────────────────┘
                                       │
                               ┌───────▼───────┐
                               │ ADMINISTRADOR │
                               │               │
                               │ - Productos   │
                               │ - Promociones │
                               │ - Empleados   │
                               │ - Reportes    │
                               └───────────────┘
```

## Diagrama de flujo principal — Ciclo de vida de un pedido

```
  CLIENTE                    SISTEMA                      VENDEDOR
     │                          │                             │
     │── ver catálogo ─────────>│                             │
     │<─ lista productos ───────│                             │
     │                          │                             │
     │── agregar al carrito ───>│                             │
     │── confirmar pedido ─────>│                             │
     │                          │── crear pedido (PENDIENTE) ─│
     │                          │                             │
     │                          │<── registrar pago ──────────│
     │                          │── actualizar (PAGADO) ──────│
     │                          │── registrar historial ──────│
     │                          │                             │
     │                          │<── despachar ───────────────│
     │                          │── actualizar (ENVIADO) ─────│
     │                          │── registrar historial ──────│
     │                          │                             │
     │<─ pedido ENTREGADO ──────│                             │
```

## Entidades candidatas identificadas en el análisis

```
┌──────────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│     ROL      │   │   EMPLEADO   │   │   CLIENTE    │   │   CARRITO    │
└──────────────┘   └──────────────┘   └──────────────┘   └──────────────┘

┌──────────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│  CATEGORIA   │   │   MATERIAL   │   │   PRODUCTO   │   │ ITEM_CARRITO │
└──────────────┘   └──────────────┘   └──────────────┘   └──────────────┘

┌──────────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│   PEDIDO     │   │DET._PEDIDO   │   │ESTADO_PEDIDO │   │   HISTORIAL  │
└──────────────┘   └──────────────┘   └──────────────┘   └──────────────┘

┌──────────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│  PROMOCION   │   │PROM_PRODUCTO │   │TIPO_MOVIMIEN.│   │  INVENTARIO  │
└──────────────┘   └──────────────┘   └──────────────┘   └──────────────┘

┌──────────────┐
│IMAGEN_PRODUCT│
└──────────────┘
```

## Diagrama MER completo

Ver imagen: `project-bd/docs/DIAGRAMA-MER.png`

Ver descripción textual: `project-bd/docs/mer-diagram.md`
