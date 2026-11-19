# Brief: ventas en corredor vial con peaje y clima - Restaurante + Discoteca

## Usuario
El dueño / administrador del restaurante El Caserón y de la discoteca en el corredor Medellín - Sopetrán, Antioquia. Usa el resultado para decidir qué reponer, cuándo reforzar personal y cuándo programar eventos.

## Fuente
1. Archivo mensual de INVIAS: `6._Resumen_Mensual_por_estación_de_Peaje_Tráfico_y_Recaudo_2014_a_jun_2026.xlsx` - Tráfico y recaudo por estación de peaje.
2. API pública de clima Open-Meteo: pronóstico diario sin llave, por latitud / longitud de Sopetrán.
3. Archivos POS del restaurante: `ventas_caseron.xlsx` + `productos_vendidos_Caseron.xlsx`, unidos por `producto_id`.
4. Archivo POS de la discoteca: por confirmar archivo, misma estructura que el restaurante, con `negocio = 'discoteca'`.

## Destino
Dos tablas en PostgreSQL:
1. `ventas_por_producto_y_dia` - Una fila por producto, por negocio, por día.
2. `ventas_por_negocio_y_dia` - Una fila por negocio y día, con su clima y su peaje del mes.

## Tres KPI
1. Ventas por día, por negocio - Restaurante vs discoteca.
2. Productos más vendidos, por negocio - Qué plato y qué bebida vende más.
3. Ticket promedio, por negocio y por día - Cuánto gasta en promedio cada venta.

El peaje y el clima no son KPI, son contexto para explicar los KPI: si un día vendiste poco, miras si llovió o si el peaje subió ese mes.

## Frecuencia
- Ventas + clima: cada día a las 6:00 a.m.
- Peajes INVIAS: cada mes, cuando INVIAS publica el resumen mensual.

## Calidad mínima
- Sin ventas repetidas (sin `venta_id` duplicado, ni entre restaurante y discoteca).
- Sin fechas ilegibles.
- Sin productos sin precio en el catálogo.
- Sin ventas con `producto_id` que no existe en `productos`.
- Sin ventas sin `negocio` - Toda venta tiene que decir si es `restaurante` o `discoteca`.
- Sin peaje sin estación ni mes.

## Terminó bien
Corre dos veces seguidas con el mismo resultado, y los tres KPI coinciden con una cuenta manual en Excel, para restaurante y para discoteca por separado.

## Fuera
- Compras a proveedores y devoluciones.
- Tráfico diario de peajes: INVIAS solo da resumen mensual, no diario.
- Nómina y costos de personal.