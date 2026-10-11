# Datos

`DEPOSITOS_CARGADOS_FEBAN_sep2025_sep2026_anon.xlsx` es el único archivo de entrada del proyecto. Es un export de la transacción FEBAN de SAP (posproceso de extractos bancarios electrónicos) con los movimientos de dos cuentas, una en BBVA y otra en HSBC, del 01-09-2025 al 30-09-2026.

## Contenido

La hoja `Hoja1` tiene 4,804 movimientos en orden cronológico, sin duplicados, con las columnas originales del export:

| Columna | Uso en el proyecto |
|---|---|
| `Value date` | Fecha valor; define el día del flujo |
| `Amount` | Importe en MXN (positivo = entrada, negativo = salida) |
| `House Bk` | Banco (BBVA o HSBC) |
| `MR no` | Número de partida dentro del extracto; forma parte de la llave de deduplicación |
| `Payment Notes` | Concepto del banco; con él se separa la tesorería del flujo operativo |
| `External transaction` | Tipo de transacción (NTRF, NCOL, NCOM, NCMS, etc.) |
| `AC` | Moneda; solo hay MXN |
| `Shrt key` | Clave de carga del extracto |
| `Post. rule` | Regla contable |
| `Customer`, `DocumentNo` | Vacías en este export |

La hoja `Notas_datos` documenta cómo se armó el archivo, los supuestos de generación y un resumen mensual con fórmulas sobre `Hoja1`.

## Origen de los movimientos

Solo tenía seis días hábiles de movimientos reales (del 01 al 08-09-2026, 107 movimientos, filas 4,451 a 4,557 de `Hoja1`). El export original traía cada extracto cargado varias veces, así que conservé la carga más reciente de cada banco y eliminé las copias.

Para tener un año de historia completé el resto con datos ficticios (4,697 movimientos): del 01-09-2025 al 31-08-2026 y del 09 al 30-09-2026. Usé las mismas columnas, bancos, tipos de transacción, reglas contables y formatos de concepto del export real, calibrados con los montos de esa semana. Los únicos efectos estacionales que agregué son mayor ingreso de enero a marzo (1.35 veces el normal) y mayor gasto de septiembre a diciembre (2 veces el normal); el resto de los meses es normal. La tesorería sigue la misma lógica del real: HSBC cierra en cero cada día con barridos y BBVA concentra fondos en HSBC cada dos semanas.

El pipeline trata todo el archivo como historia FEBAN con una sola carga por extracto y no distingue qué filas son reales.

## Anonimización

Los conceptos bancarios del export real traen nombres de clientes y proveedores, cuentas CLABE, números de cuenta y de contrato. Antes de subir el archivo reemplacé esos datos:

- Los nombres de clientes y proveedores pasaron a etiquetas genéricas (`CLIENTE ALFA`, `PAGO CLIENTE BETA`, `CARTERA PROVEEDOR`, etc.).
- Las cuentas CLABE de los conceptos SPEI se cambiaron por números consecutivos ficticios, uno por cada cuenta original.
- La cuenta de concentración, el contrato de terminal, el prefijo de cuenta de los depósitos en efectivo y las cuentas de los pagos de terceros también son ficticios.
- Se quitaron el autor y el último editor de las propiedades del archivo.

No cambié importes, fechas, bancos, tipos de transacción ni las palabras con las que se clasifican los movimientos (`SWEEP`, `GIROFONDO`, `SPEI ENVIADO HSB`, `COMISION`, `CARTERA`, `DEPOSITO`, etc.). Comprobé que la versión anonimizada produce exactamente las mismas series diarias y la misma clasificación que el archivo original.

## Usar otro export

El cuaderno y `src/` leen cualquier export FEBAN con estas columnas. Si las columnas cambian de nombre, se ajustan en `COLUMNAS` (celda 4.1 del cuaderno o `src/flujo_efectivo/config.py`), y las reglas para separar la tesorería en `REGLAS_CATEGORIA`, con la lista de cuentas propias de la empresa. Los exports reales sin anonimizar no deben subirse al repositorio; el `.gitignore` excluye cualquier `*FEBAN*.xlsx` que no termine en `_anon.xlsx`.
