# Sistema de Control de Producción — AppSheet + Google Sheets + Looker Studio

Sistema integral de captura, almacenamiento y visualización de datos de producción para una planta de confección textil. Diseñado y desarrollado de punta a punta: desde el formulario de campo hasta los dashboards gerenciales de costos.

## Problema que resuelve

La planta registraba órdenes de corte y avances de fabricación en papel o Excel disperso, sin trazabilidad por operario, máquina u orden de corte, y sin visibilidad de costos de mano de obra e insumos en tiempo real. Esto generaba:

- Demoras para saber cuánto se había producido de cada orden.
- Falta de control de costos de mano de obra por operación/operario.
- Nula trazabilidad del consumo de tela (insumo) por orden de corte.
- Reportes gerenciales armados manualmente y con horas de atraso.

## Solución

Se construyó un flujo de datos de tres capas:

```
┌─────────────────┐      ┌──────────────────┐      ┌───────────────────┐
│   APPSHEET      │      │  GOOGLE SHEETS   │      │  LOOKER STUDIO    │
│  (captura móvil │ ───► │  (base de datos  │ ───► │  (dashboards      │
│   en planta)    │      │   centralizada)  │      │   gerenciales)    │
└─────────────────┘      └──────────────────┘      └───────────────────┘
```

1. **AppSheet** — App móvil/web usada por operarios y digitadores para registrar órdenes de corte, avances de fabricación, operarios, máquinas y productos, directamente desde el piso de planta.
2. **Google Sheets** — Toda la información capturada se almacena en una hoja de cálculo centralizada en Drive, que actúa como base de datos relacional simplificada (tablas de producción, fabricación, operarios, máquinas, costos).
3. **Looker Studio** — Se conecta directamente a Google Sheets y transforma esos datos crudos en 4 reportes interactivos: producción general, órdenes de corte, registros de fabricación y dashboard gerencial de costos.

Ver documentación técnica completa en [`FUNCIONAMIENTO.md`](./FUNCIONAMIENTO.md).

## Vista previa

| Reporte | Descripción |
|---|---|
| **Producción — General Día** | Comparativo entre lo ordenado (orden de corte) y lo realmente fabricado (registro de fabricación), por producto. |
| **Producción — Órdenes de Corte** | Detalle de metros cuadrados (M2) producidos por orden y producto, con tabla dinámica y filtros. |
| **Producción — Registros de Fabricación** | Trazabilidad por operación, operario y máquina, con tiempos de inicio/fin. |
| **Dashboard Gerencial** | Costo de mano de obra por operación/operario y costo de insumo (tela) por orden de corte, en COP. |

## Stack técnico

- **Captura de datos:** Google AppSheet (vistas deck, formularios, slices, vistas de referencia)
- **Base de datos:** Google Sheets (Drive)
- **Visualización:** Looker Studio (gráficos, tablas dinámicas, filtros cruzados, exportación a PDF)
- **Automatizaciones adicionales:** Make (para flujos complementarios del negocio)

## Impacto

- Trazabilidad completa de +200 mil unidades registradas por orden de corte, producto y operario.
- Visibilidad de costos de mano de obra y de insumos (tela) actualizada en tiempo real, sin trabajo manual de consolidación.
- Reducción del tiempo de generación de reportes gerenciales, antes armados manualmente.

## Sobre el autor

Ernesto De la Parra — Ingeniero Civil especializado en gestión de proyectos y presupuestos de construcción, con habilidades complementarias en analítica de datos y automatización (SQL, Power BI, Looker Studio, AppSheet, Make).

- LinkedIn: [linkedin.com/in/ernesto-de-la-parra-67a4001a2](https://linkedin.com/in/ernesto-de-la-parra-67a4001a2)
