# Contrato: fuentes del proyecto corredor vial

## 1. Peajes INVIAS - `peajes_invias`
Filtro del proyecto: `Peaje ILIKE '%SAN CRISTOBAL%'` y `Departamento = 'Antioquia'`.
Solo se usa San Cristóbal, porque es el peaje del corredor Medellín - Sopetrán. 

Archivo: `6._Resumen_Mensual_por_estación_de_Peaje_Tráfico_y_Recaudo_2014_a_jun_2026.xlsx`
Hoja: `TABLA DE SALIDA DETALLE REPORTE`

| columna real en Excel | tipo | ¿puede faltar? | ejemplo de tu foto |
| --- | --- | --- | --- |
| IdReporte | entero | no | 17800 |
| IdPeaje | entero | no | 26 |
| Peaje | texto | no | San Cristobal |
| Departamento | texto | no | Antioquia |
| Municipio | texto | no | San Cristobal |
| Divipola | texto | no | 05190 |
| Proyecto | texto | no | IP - Vías del Nus |
| FechaDesde | fecha | no | 1/01/2026 |
| FechaHasta | fecha | no | 15/01/2026 |
| IdCategoriaTarifa | entero | no | 1 |
| Tarifa | número | no | 24600 |
| Trafico | entero | no | 98093 |
| TraficoEvasores | entero | no | 0 |
| TraficoExentos787 | entero | no | 560 |
| TraficoTotal | entero | no | 98653 |
| Recaudo | número | no | 2413087800 |
| RecaudoEvasores | número | no | 0 |
| RecaudoTotal | número | no | 2413087800 |
| TPDM | entero | no | 3182 |
| FechaCorte | fecha | no | Ver nota |

Clave: `IdReporte` - Cada fila es un peaje, en un rango de fechas, para una categoría.
Frecuencia: mensual / quincenal, cuando INVIAS publica.
Responsable: INVIAS
Nota: `FechaCorte` en tu foto sale como `####` porque la columna está angosta en Excel. Al leer el archivo hay que ensancharla, no está vacía.

## 2. Clima - `clima_diario` - API Open-Meteo

| columna | tipo | ¿puede faltar? | ejemplo |
| --- | --- | --- | --- |
| fecha | fecha | no | 2026-10-03 |
| temperatura_max | número | no | 28.5 |
| lluvia_mm | número | no | 6.3 |
| es_dia_lluvioso | sí/no | no | TRUE |

Clave: `fecha`
Frecuencia: diaria, 6:00 a.m.
Responsable: Open-Meteo - `https://api.open-meteo.com/v1/forecast?latitude=6.50&longitude=-75.75&daily=temperature_2m_max,precipitation_sum&timezone=America%2FBogota`

## 3. Ventas Caserón - `ventas_caseron` - TU ARCHIVO REAL

Archivo: `ventas_caseron.xlsx`
Título fila 1: `Reporte de Ventas Datos de Ventas Desde 2026-10-01 a 2026-10-31, Periodo de Tiempo por Dia 12:00 AM - 11:59 PM`

| columna real en Excel | tipo | ¿puede faltar? | ejemplo de tu foto 2026-10-01 |
| --- | --- | --- | --- |
| Fecha | fecha | no | 2026-10-01 |
| Total de Pedidos | entero | no | 18 |
| Monto Total de Impuestos | número | no | 0.00 |
| Efectivo | número | no | 743,000.00 |
| UPI | número | no | 0.00 |
| Tarjeta | número | no | 0.00 |
| Razorpay | número | no | 0.00 |
| Stripe | número | no | 0.00 |
| Flutterwave | número | no | 0.00 |
| Costo de Envío | número | no | 0.00 |
| Descuento | número | no | 0.00 |
| Propina | número | no | 0.00 |
| Total | número | no | 831,000.00 |
| Total (Sin Propina) | número | no | 831,000.00 |

Clave: `Fecha`
Frecuencia: diaria
Responsable: Restaurante El Caserón
Nota: Fila 1 es título, se ignora. Encabezado real en fila 2.
Nota 2: Las plataformas de pago NO se quitan. Son 6 columnas y se guardan todas: `Efectivo`, `UPI`, `Tarjeta`, `Razorpay`, `Stripe`, `Flutterwave`.

## 4. Productos - `productos_vendidos_caseron` - TU ARCHIVO REAL

Archivo: `productos_vendidos_Caseron.xlsx`
Título fila 1: `Reporte por Producto Datos de Ventas Desde 2026-01-01 a 2026-11-19, Periodo de Tiempo por Dia 12:00 AM - 11:59 PM`

| columna real en Excel | tipo | ¿puede faltar? | ejemplo |
| --- | --- | --- | --- |
| Nombre del Producto | texto | no | Por verificar en tu foto, primera columna |
| Nombre de la Categoría | texto | no | Por verificar, segunda columna |
| Cantidad Vendida | entero | no | Ej: 120 |
| Precio de Venta | número | no | Ej: 38000 |
| Ingresos Totales | número | no | Ej: 4560000 |

Clave: `Nombre del Producto` + `Nombre de la Categoría`
Frecuencia: diaria / por periodo, viene agregado.
Responsable: Restaurante El Caserón
Nota: Fila 1 es título, se ignora. Encabezado real en fila 2.

## 5. Discoteca - `ventas_discoteca` - POR CONFIRMAR

Misma estructura que `ventas_caseron.xlsx`, con una columna extra `negocio = 'discoteca'` para separar de restaurante. Archivo por confirmar.

## Cómo se unen - CORREGIDO, sin inventar JOIN

```sql
-- 1. Ventas + Clima: SÍ se puede, por fecha
ventas_caseron.Fecha = clima_diario.fecha

-- 2. Ventas + Peajes: SÍ se puede, por rango de fechas
-- Si la fecha de venta está entre FechaDesde y FechaHasta del peaje
ventas_caseron.Fecha BETWEEN peajes_invias.FechaDesde AND peajes_invias.FechaHasta

-- 3. Ventas + Productos: NO se puede a nivel de venta individual
-- Porque ventas viene agregado por día y productos viene agregado por producto.
-- No hay venta_id ni producto_id que los una.
-- Se analizan por separado: ventas por día, productos por producto.