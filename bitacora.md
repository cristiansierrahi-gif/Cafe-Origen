# Bitácora del proyecto — Café Origen Analítica

## Diccionario de datos

_______________________________________________________________________________________________________________________________
| Propiedad | Tipo de dato                        | Significado                                                                 |
|___________|_____________________________________|_____________________________________________________________________________|
| `id`      | `Number` (entero)                   | Identificador único secuencial de cada registro de venta (1 a 12).          |    
| `fecha`   | `String` (formato ISO `YYYY-MM-DD`) | Fecha en la que se registró la transacción en septiembre de 2026.           |
| `cliente` | `String`                            | Código identificador del cliente que realizó la compra (ej. `C01`, `C02`).  |
| `producto`| `String`                            | Nombre o referencia del café de origen o accesorio vendido.                 |
| `ciudad`  | `String`                            | Ciudad de destino o donde se realizó la venta.                              |
| `cantidad`| `Number` (entero)                   | Número de unidades del producto asociadas al registro.                      |
| `precio`  | `Number` (entero)                   | Precio unitario en pesos colombianos (COP) del producto.                    |
|_______________________________________________________________________________________________________________________________|

## Casos borde detectados

1. **Inconsistencia de capitalización en `ciudad` (`"medellín"` vs `"Medellín"`)**
   * **Hallazgo:** En el registro con `id: 5` (`C04`, `Origen Sierra 500 g`), la ciudad figura en minúscula (`"medellín"`), mientras que en los demás registros de esa ciudad aparece capitalizada (`"Medellín"`). Si se agrupan ventas por ciudad sin normalizar, se contarían como categorías distintas.
   * **Decisión:** Conservar el valor original en `src/datos.js` por trazabilidad, pero normalizar el texto (por ejemplo, aplicando formato de título / `toLowerCase()` + capitalización) al momento de limpiar o agrupar datos en `src/analitica.js`.

2. **Cantidad igual a `0` (`id: 8`, cliente `C06`, `Origen Huila 500 g`, `Cali`)**
   * **Hallazgo:** El registro `8` tiene `cantidad: 0` con un precio unitario de `42000`. Esto puede representar una venta anulada, un error de captura, una muestra sin cobro o un registro incompleto.
   * **Decisión:** Mantener el registro intacto en `src/datos.js` y definir explícitamente en `src/analitica.js` si las filas con `cantidad <= 0` se excluyen del cálculo de unidades/ingresos efectivos o se reportan aparte como anomalías/registros no facturados.

3. **Clientes recurrentes vs. compras únicas y variación de ticket (`C01`, `C02`, `C03` vs. `C04`–`C08`)**
   * **Hallazgo:** Clientes como `C01` (3 compras: `id` 1, 4 y 11), `C02` (2 compras: `id` 2 y 7, incluyendo un `Molino manual` de `120000`) y `C03` (2 compras: `id` 3 y 9) aparecen múltiples veces, mientras que `C04` a `C08` aparecen una sola vez. Además, hay mezcla de productos consumibles de café y equipamiento/accesorios (`Prensa francesa`, `Molino manual`).
   * **Decisión:** No eliminar registros duplicados de cliente porque corresponden a compras distintas en distintas fechas; en el análisis posterior se agrupará por `cliente` para calcular recurrencia y gasto acumulado, y se segmentará por tipo de producto si se evalúa ticket promedio.