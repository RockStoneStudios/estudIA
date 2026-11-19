# Contrato: fuentes del proyecto corredor vial

## 1. Peajes INVIAS - `peajes_mensual`

| columna | tipo | ¿puede faltar? | ejemplo |
| --- | --- | --- | --- |
| anio | entero | no | 2026 |
| mes | entero | no | 6 |
| estacion_peaje | texto | no | Túnel de Occidente |
| trafico_vehiculos | entero | no | 145230 |
| recaudo_total | número | no | 2850000000 |

Clave: `anio` + `mes` + `estacion_peaje`
Frecuencia: mensual, cuando INVIAS publica el resumen.
Responsable: INVIAS

## 2. Clima - `clima_diario` - API Open-Meteo

| columna | tipo | ¿puede faltar? | ejemplo |
| --- | --- | --- | --- |
| fecha | fecha | no | 2026-11-19 |
| temperatura_max | número | no | 28.5 |
| lluvia_mm | número | no | 6.3 |
| es_dia_lluvioso | sí/no | no | TRUE |

Clave: `fecha`
Frecuencia: diaria, 6:00 a.m.
Responsable: Open-Meteo - `https://api.open-meteo.com/v1/forecast?latitude=6.50&longitude=-75.75&daily=temperature_2m_max,precipitation_sum&timezone=America%2FBogota`

## 3. Productos - `productos` - Catálogo de los 2 negocios

| columna | tipo | ¿puede faltar? | ejemplo |
| --- | --- | --- | --- |
| producto_id | texto | no | P-0045 |
| negocio | texto | no | restaurante / discoteca |
| nombre_producto | texto | no | Bandeja paisa / Cerveza |
| categoria | texto | no | Plato fuerte / Bebida |
| precio_unitario | número | no | 38000 |

Clave: `producto_id` + `negocio` - Porque el mismo código puede existir en los 2 negocios con precio diferente.
Frecuencia: cambia cuando cambia el menú.
Responsable: Restaurante El Caserón + Discoteca

## 4. Ventas - `ventas_pos` - Hecho de los 2 negocios

| columna | tipo | ¿puede faltar? | ejemplo |
| --- | --- | --- | --- |
| venta_id | texto | no | V-10234 |
| fecha_venta | fecha | no | 2026-11-19 |
| negocio | texto | no | restaurante / discoteca |
| producto_id | texto | no | P-0045 |
| cantidad | entero | no | 2 |
| valor_total | número | no | 76000 |

Clave: `venta_id` + `negocio`
Frecuencia: diaria, 6:00 a.m.
Responsable: Restaurante El Caserón + Discoteca

## Cómo se unen - Los 3 JOIN de todo el proyecto

```sql
-- 1. Ventas + Productos: para saber el nombre y la categoría
ventas_pos.producto_id = productos.producto_id
AND ventas_pos.negocio = productos.negocio

-- 2. Ventas + Clima: para saber si ese día llovió
ventas_pos.fecha_venta = clima_diario.fecha

-- 3. Ventas + Peajes: para saber el tráfico y recaudo de ese mes
EXTRACT(YEAR FROM ventas_pos.fecha_venta) = peajes_mensual.anio
AND EXTRACT(MONTH FROM ventas_pos.fecha_venta) = peajes_mensual.mes