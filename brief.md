# Brief: ventas en corredor vial con peaje y clima - Restaurante El Caserón + Discoteca

## Usuario
El dueño / administrador del restaurante El Caserón y de la discoteca en el corredor Medellín - Sopetrán, Antioquia. Usa el resultado para decidir qué reponer, cuándo reforzar personal y cuándo programar eventos.

## Fuente
1. Archivo de INVIAS: `6._Resumen_Mensual_por_estación_de_Peaje_Tráfico_y_Recaudo_2014_a_jun_2026.xlsx` - Tabla de salida detalle reporte, por peaje y por rango de fechas.
2. API pública de clima Open-Meteo: pronóstico diario sin llave, por latitud / longitud de Sopetrán.
3. Archivo real del restaurante: `ventas_caseron.xlsx` - Reporte de Ventas 2026-10-01 a 2026-10-31, agregado por día.
4. Archivo real de productos: `productos_vendidos_Caseron.xlsx` - Reporte por Producto 2026-01-01 a 2026-11-19, agregado por producto.
5. Archivo de discoteca: por confirmar, con misma estructura que restaurante.

## Destino
Dos tablas en PostgreSQL:
1. `ventas_por_dia` - Una fila por día y por negocio.
2. `ventas_por_producto` - Una fila por producto y categoría.

## Tres KPI
1. Ventas por día = `Total (Sin Propina)` de `ventas_caseron.xlsx`
2. Pedidos por día = `Total de Pedidos` de `ventas_caseron.xlsx`
3. Ticket promedio = `Total (Sin Propina)` / `Total de Pedidos`

Desglose extra con columnas reales, no es KPI nuevo:
- Ventas por método de pago = `Efectivo` + `UPI` + `Tarjeta` + `Razorpay` + `Stripe` + `Flutterwave`
- Producto más vendido = Mayor `Cantidad Vendida` en `productos_vendidos_Caseron.xlsx`
- Mayor ingreso por producto = Mayor `Ingresos Totales` en `productos_vendidos_Caseron.xlsx`

Ejemplo real de ventas: 2026-10-03, 43 pedidos, $3,143,000.00 total, ticket = 3143000 / 43 = $73,093.
Ejemplo real de INVIAS: Peaje 26 CISNEROS, 1/01/2026 a 15/01/2026, Tarifa $24,600, TráficoTotal 98,653, RecaudoTotal $2,413,087,800.

## Frecuencia
- Ventas + productos + clima: cada día a las 6:00 a.m.
- Peajes INVIAS: cada mes / quincena, cuando INVIAS publica el resumen. En tu archivo viene por rango `FechaDesde` - `FechaHasta`.

## Calidad mínima
- La fila 1 de los Excel es título, no es dato. Se salta.
- La fila 2 es el encabezado real.
- Sin fechas duplicadas en ventas.
- Sin `Total de Pedidos` en 0 con `Total` mayor a 0.
- Sin `Total (Sin Propina)` vacío.
- Sin producto sin `Nombre de la Categoría`.
- Sin peaje sin `IdPeaje`, sin `FechaDesde` ni `FechaHasta`.
- Sin `FechaCorte` con `####` - En tu foto de INVIAS sale como `####` porque la columna está muy angosta en Excel, no porque esté vacía. Hay que ensancharla al leerla.

## Terminó bien
Corre dos veces con el mismo resultado, y los KPI coinciden con el Excel:
- 2026-10-10 tiene que dar 78 pedidos y $1,887,000.00 en ventas.
- Peaje 26 CISNEROS, 1/01/2026 a 15/01/2026, tiene que dar TráficoTotal 98,653 y RecaudoTotal $2,413,087,800.

## Fuera
- Compras a proveedores, devoluciones y nómina.
- Tráfico diario de peajes: INVIAS en tu archivo viene por rango `FechaDesde` - `FechaHasta`, no día a día.
- Detalle venta por venta con producto: tu archivo de ventas es resumen por día, tu archivo de productos es resumen por producto. No hay un archivo que diga "el 2026-10-03 a las 12pm se vendió 1 bandeja". Por eso no se hace JOIN de ventas por día con productos por producto a nivel de venta individual.