# Funcionamiento del sistema

Este documento explica en detalle cómo fluye la información desde que se captura en planta hasta que se visualiza en los dashboards gerenciales.

## 1. Origen: captura de datos en AppSheet

AppSheet es la interfaz con la que interactúan los usuarios de planta (digitadores y operarios). No requiere código: se configura sobre una base de datos en Google Sheets y genera automáticamente una app funcional para web y celular.

**Vistas principales configuradas:**

- **Ingreso Orden de Corte** (vista tipo *deck*): registra cada orden de corte con producto, cantidad, ancho de rollo, número de capas, medida por capa y total de metros cuadrados (M2), calculado automáticamente.
- **Registro de Fabricación** (vista tipo *deck*): registra cada avance de producción — operación realizada, producto, operario, máquina utilizada, hora de inicio y hora final.
- **Vistas de seguimiento**: agrupan y filtran los registros por orden de corte, permitiendo ver el avance acumulado de cada una.
- **Menú de navegación**: formularios de ingreso independientes para digitador, operario, máquina, operación y producto — esto mantiene las tablas maestras (catálogos) separadas de las tablas transaccionales.

**Cómo está organizada la data dentro de AppSheet:**

Cada vista apunta a una tabla o *slice* específico (por ejemplo, la vista "Ingreso Orden de Corte" usa la tabla `O PRODUCCION`, mientras que "Registro de Fabricación" usa `R FABRICACION`). Los *slices* permiten filtrar subconjuntos de datos sin duplicar información.

Cada registro capturado en AppSheet escribe una fila nueva directamente en la hoja de Google Sheets correspondiente — no hay una base de datos intermedia; AppSheet lee y escribe en tiempo real sobre el archivo de Drive.

## 2. Almacenamiento: base de datos en Google Sheets

El archivo central (`BD MAIN PRODUCCION 1` en el proyecto original) vive en Google Drive y cumple el rol de base de datos relacional simplificada, con una hoja por tabla:

| Hoja / Tabla | Contenido | Alimentada por |
|---|---|---|
| `O PRODUCCION` | Órdenes de corte: fecha, producto, cantidad, ancho de rollo, capas, metros totales (M2) | Vista "Ingreso Orden de Corte" |
| `R FABRICACION` | Registros de fabricación: operación, producto, operario, máquina, horas de inicio/fin | Vista "Registro de Fabricación" |
| Tablas maestras | Catálogos de digitador, operario, máquina, operación y producto | Formularios del menú de navegación |

**Columnas clave de `O PRODUCCION`:** No, Fecha, Orden de corte, Producto, Cantidad, Ancho de Rollo, Prenda x Capa, Número de Capas, Medida x Capa, Total Metros (M2), Observación, Realizado, Archivo adjunto.

Cada fila incluye además la ruta del archivo adjunto que AppSheet sube automáticamente a una carpeta de Drive (`/appsheet/data/...`), lo que permite trazabilidad fotográfica del proceso sin salir de la app.

Este archivo de Sheets es el **único punto de verdad**: tanto AppSheet como Looker Studio leen y escriben (o solo leen, en el caso de Looker) sobre él, evitando duplicidad de datos.

## 3. Visualización: dashboards en Looker Studio

Looker Studio se conecta como fuente de datos directamente a las hojas de `BD MAIN PRODUCCION 1` (conector nativo de Google Sheets) y arma cuatro reportes:

### 3.1 Producción — General Día
Comparativo lado a lado entre lo que se ordenó cortar (`O PRODUCCION`) y lo que efectivamente se fabricó (`R FABRICACION`), agrupado por producto y orden de corte. Incluye filtros por orden de corte, por registro de fabricación y por periodo, y botón de exportación a PDF.

### 3.2 Producción — Órdenes de Corte
Detalle de metros cuadrados (M2) producidos a lo largo del tiempo y por producto, con:
- Serie temporal de M2 producidos.
- Tabla dinámica cruzando orden de corte × producto × fecha.
- Tabla de detalle con las variables de cálculo (ancho de rollo, número de capas, medida por capa) que originan el M2 total — esto permite auditar cómo se calculó cada cifra.

### 3.3 Producción — Registros de Fabricación
Trazabilidad operativa: qué operación se hizo, quién la hizo (operario), en qué máquina y en qué horario. Incluye:
- Conteo de registros por producto.
- Tabla dinámica de cantidad realizada por operario y por máquina.
- Gráfico de cantidad realizada por máquina (fileteadora, plana, etc.), útil para detectar cuellos de botella.

### 3.4 Dashboard Gerencial
Vista de costos, pensada para dirección/gerencia:
- **Costo de mano de obra**: se calcula multiplicando la cantidad realizada por operación × un costo unitario por operación (tabla de tarifas), agrupado por operario y por operación.
- **Costo de insumo (tela)**: cruza la cantidad de metros cuadrados producidos por orden de corte con el costo unitario del insumo, mostrando el costo total de tela consumida por producto y por orden.
- KPIs totales: costo total de mano de obra (COP) y costo total de insumo (COP) del periodo seleccionado.

Todos los reportes comparten el mismo patrón de interacción: selector de orden de corte / registro de fabricación, selector de periodo, botón "Restablecer" filtros y botón "Reporte PDF" para exportar.

## 4. Flujo completo resumido

```
Operario/digitador registra en planta (AppSheet, celular o tablet)
        │
        ▼
Fila nueva se escribe en Google Sheets (BD MAIN PRODUCCION)
        │
        ▼
Looker Studio lee la hoja como fuente de datos (refresco automático)
        │
        ▼
Dashboards de producción y costos se actualizan sin intervención manual
```

## 5. Por qué esta arquitectura

- **Sin código de servidor**: todo corre sobre herramientas no-code/low-code (AppSheet + Sheets + Looker), lo que reduce el costo de mantenimiento para una PYME.
- **Un solo punto de verdad**: al no duplicar datos entre sistemas, se elimina el riesgo de inconsistencias entre lo que ve el operario y lo que ve gerencia.
- **Trazabilidad end-to-end**: desde la orden de corte hasta el costo final, cada cifra en el dashboard gerencial se puede rastrear hasta el registro original capturado en planta.

## 6. Limitaciones conocidas y mejoras futuras

- Al depender de Google Sheets como base de datos, el sistema tiene límites de volumen y velocidad frente a una base de datos relacional tradicional (ej. PostgreSQL) para operaciones muy grandes.
- Se evalúa migrar la capa de almacenamiento a una base de datos SQL con un conector directo a Looker Studio, manteniendo AppSheet como interfaz de captura.
- Automatización de alertas (ej. vía Make) cuando el costo de una orden supera un umbral definido.
