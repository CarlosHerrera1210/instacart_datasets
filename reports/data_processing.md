# Informe técnico: limpieza, análisis y estructuración de datos de Instacart

**Aprendiz:** Carlos Eduardo Herrera Sibaja  
**Programa:** Análisis y Desarrollo de Software  
**Evidencia:** GA1-220501094-01-AA1-EV06  
**Fecha:** 11/11/2026

## 1. Objetivo

Continuar el proyecto Instacart a partir del diagnóstico de calidad ([`data_quality.md`](data_quality.md)) aplicando limpieza, filtrado, agrupación y combinación de datos con Pandas.

**Pregunta orientadora:** ¿Cómo preparar y estructurar los datos de Instacart para obtener información confiable sobre los hábitos de compra?

Todo el código, sus resultados y las interpretaciones de los 21 ejercicios están en un notebook por lección: [`lesson_2_data_cleaning.ipynb`](../notebooks/lesson_2_data_cleaning.ipynb), [`lesson_3_data_analysis.ipynb`](../notebooks/lesson_3_data_analysis.ipynb) y [`lesson_4_data_merging.ipynb`](../notebooks/lesson_4_data_merging.ipynb). El índice [`pandas_labs.ipynb`](../notebooks/pandas_labs.ipynb) indica el orden de ejecución y reúne las conclusiones generales. Las cifras de este informe provienen de su ejecución completa con pandas 3.1.0.

## 2. Lección 2: limpieza y estandarización

### 2.1 Decisiones de limpieza

Cada problema detectado en el diagnóstico se resolvió según lo que significa el dato:

| Tabla | Problema | Evidencia | Decisión | Justificación |
|---|---|---|---|---|
| `orders` | Filas duplicadas | 15 pedidos repetidos, todos del miércoles a las 2 a. m. | **Eliminar** con `drop_duplicates()` | Fallo de carga, no pedidos reales |
| `orders` | Nulos en `days_since_prior_order` | Coinciden exactamente con los primeros pedidos (`order_number == 1`) | **Conservar** | No hay pedido anterior; un 0 inventaría una recompra el mismo día |
| `products` | 1.258 nombres nulos | Todos en el pasillo 100 y el departamento 21 (`missing`) | **Corregir** con `'Unknown'` | El producto existe y aparece en pedidos |
| `products` | Texto sin estandarizar | 403 nombres con espacios dobles y 16 con espacio invisible (`\xa0`) | **Corregir** con `.str.replace`, `.str.strip` y `.str.title` | Evita contar el mismo producto como dos |
| `products` | Duplicados implícitos | 141 nombres repetidos tras estandarizar; 268 de los 280 productos implicados están en pedidos | **Conservar y documentar** | Tienen `product_id` distintos; se agrupa por `product_id`, no por nombre |
| `order_products` | 836 nulos en `add_to_cart_order` | 70 pedidos, todos con 65 productos o más; posición máxima registrada: 64 | **Conservar** | El sistema solo registraba 64 posiciones; el artículo existe |
| `orders`, `order_products` | Tipos sobredimensionados | Enteros `int64` y `float64` por los nulos | **Corregir** con `astype` (`int8`, `int16`, `int32`, `Int16` nullable) | Menos memoria sin perder información |

### 2.2 Informe de cambios del pipeline

El Laboratorio guiado 2 encadena los pasos con `pipe` y registra el estado de cada tabla después de cada paso (`data/processed/cleaning_report.csv`):

| Tabla | Paso | Filas | Nulos | Memoria (MB) |
|---|---|---:|---:|---:|
| `orders` | 0 crudo | 478.967 | 28.819 | 23,0 |
| `orders` | 1 sin duplicados | 478.952 | 28.817 | 23,0 |
| `orders` | 2 tipos | 478.952 | 28.817 | 7,2 |
| `products` | 0 crudo | 49.694 | 1.258 | 3,1 |
| `products` | 1 nombres | 49.694 | 0 | 3,1 |
| `order_products` | 0 crudo | 4.545.007 | 836 | 145,4 |
| `order_products` | 1 tipos | 4.545.007 | 836 | 54,5 |

El pipeline termina con una validación automática (`assert`): `order_id` único, horas entre 0 y 23, productos sin nulos ni espacios invisibles y `reordered` solo con 0 y 1. Además, el resultado del pipeline es idéntico (`equals`) al de la limpieza manual.

### 2.3 Hallazgos del Reto 2

- `order_products` no tiene duplicados exactos ni pares `order_id` + `product_id` repetidos, y `reordered` solo contiene 0 y 1.
- `aisles` y `departments` no necesitan limpieza de texto: sin duplicados ni espacios sobrantes.
- La estandarización cambió 6.321 de 48.436 nombres (13,1 %), sobre todo por mayúsculas.
- Pasar el día de la semana de texto a categoría ordenada reduce su memoria un 93 % (de 7,14 a 0,48 MB).
- Los productos `Unknown` suman 9.547 artículos (0,21 %) en 8.254 pedidos. No se eliminan: son compras reales y 24 pedidos quedarían vacíos.
- Reemplazar `'missing'` por `pd.NA` en `departments` hace que `groupby` descarte ese grupo y 1.258 productos desaparezcan del resumen; por eso se mantiene como texto.

## 3. Lección 3: filtrado, ordenamiento y agrupación

### 3.1 Cuándo se compra

| Día (`order_dow`) | Pedidos | Clientes | Espera media (días) |
|---|---:|---:|---:|
| 0 (domingo) | 84.090 | 57.661 | 11,8 |
| 1 (lunes) | 82.185 | 56.479 | 11,3 |
| 2 (martes) | 65.833 | 48.587 | 11,2 |
| 3 (miércoles) | 60.897 | 45.331 | 10,7 |
| 4 (jueves) | 59.810 | 44.281 | 10,5 |
| 5 (viernes) | 63.488 | 46.127 | 10,5 |
| 6 (sábado) | 62.649 | 45.604 | 11,4 |

- La hora pico es las 10 a. m. (40.578 pedidos) y la combinación más alta es lunes a las 10 (7.881 pedidos).
- Los domingos el pico se desplaza a la tarde: 15 h (7.585), 14 h (7.533) y 13 h (7.483).
- Las combinaciones día-franja más activas son domingo, lunes y martes en la tarde.

### 3.2 Segmentos de clientes (Laboratorio guiado 3)

Los clientes se segmentaron con `pd.cut` según su número de pedidos:

| Segmento | Clientes | % clientes | Pedidos | Espera mediana (días) |
|---|---:|---:|---:|---:|
| Ocasional (1 pedido) | 55.357 | 35,2 | 55.357 | 16 |
| Regular (2 a 4) | 71.553 | 45,4 | 191.649 | 11 |
| Frecuente (5 o más) | 30.527 | 19,4 | 231.946 | 6 |

Porcentaje de pedidos de cada segmento por franja horaria:

| Segmento | Madrugada (0–5) | Mañana (6–11) | Tarde (12–17) | Noche (18–23) |
|---|---:|---:|---:|---:|
| Ocasional | 2,1 | 30,0 | 49,4 | 18,5 |
| Regular | 1,9 | 31,6 | 48,4 | 18,1 |
| Frecuente | 1,7 | 35,2 | 45,9 | 17,2 |

**Interpretación:** el 19,4 % de los clientes hace casi la mitad de los pedidos y vuelve cada 6 días. Todos los segmentos compran sobre todo en la tarde, pero los frecuentes compran más por la mañana: una campaña de fidelización debería enviarse en esa franja.

### 3.3 Hallazgos del Reto 3

- 6.327 clientes (4 %) tienen 10 pedidos o más.
- 44.577 pedidos (9,31 %) se hicieron exactamente una semana después del anterior: muchos clientes hacen una compra semanal.
- La espera media es mayor a las 0 h (12,52 días) y menor a las 22 h (10,62 días).
- 49.420 pedidos tienen 20 productos o más; el más grande (pedido 61355) tiene 127.
- La tasa de recompra baja según la posición en el carrito, de 0,679 en la posición 1 a 0,617 en la 5: primero se agregan los productos habituales.
- La espera mediana es de 7 días entre semana y de 8 días en fin de semana.

## 4. Lección 4: combinación de datasets

### 4.1 Validación de la tabla maestra

La tabla maestra parte de `order_products` (tabla de hechos) y le une `products`, `aisles`, `departments` y `orders` con `how='left'` y `validate='m:1'`:

| Paso | Filas | Columnas |
|---|---:|---:|
| `order_products` + `products` | 4.545.007 | 7 |
| + `aisles` | 4.545.007 | 8 |
| + `departments` | 4.545.007 | 9 |
| + `orders` | 4.545.007 | 14 |

Controles aplicados:

- `assert len(maestra) == len(order_products)`: ninguna unión creó ni perdió filas.
- Sin nulos en `product_name`, `aisle`, `department` ni `user_id`: todas las claves encontraron pareja.
- Con `validate='m:1'` se comprobó que una clave repetida en el catálogo se detecta como `MergeError`.
- Con `indicator=True` se detectaron 28.906 pedidos (6 %) sin productos registrados; con `inner` se habrían perdido sin aviso (450.046 filas frente a 478.952 con `left`).

### 4.2 Tabla de análisis por producto (Laboratorio guiado 4)

La tabla maestra se agregó por producto (45.573 productos pedidos al menos una vez), se recuperaron nombre y departamento con `validate='1:1'` y se filtraron los 739 productos con al menos 1.000 pedidos. Los líderes de recompra de cada departamento (18) incluyen:

| Departamento | Producto | Pedidos | Tasa de recompra |
|---|---|---:|---:|
| dairy eggs | Whole Organic Omega 3 Milk | 1.286 | 0,863 |
| produce | Banana | 66.050 | 0,844 |
| snacks | Extra Fancy Unsalted Mixed Nuts | 1.436 | 0,822 |
| beverages | Zero Calorie Cola | 1.256 | 0,818 |
| bakery | Organic Honey Sweet Whole Wheat Bread | 1.607 | 0,799 |

**Interpretación:** el producto más pedido no siempre es el más fiel; en lácteos, una leche con pocos pedidos se recompra el 86 % de las veces.

### 4.3 Hallazgos del Reto 4

- Entre los 77 pasillos con al menos 10.000 artículos, los de mayor recompra son `milk` (0,781), `water seltzer sparkling water` (0,731) y `fresh fruits` (0,718). El de menor recompra es `spices seasonings` (0,152).
- En la madrugada (0 a 5 h), `beverages` entra en el top 3 de departamentos en lugar de `snacks`.
- Los snacks más pedidos son Lightly Salted Baked Snap Pea Crisps, Original Veggie Straws y Sea Salt Pita Chips.
- 149.626 clientes tienen artículos registrados, con 30,4 artículos en promedio; el cliente 103634 compró 580 artículos en 18 pedidos.
- La tabla dinámica de artículos por departamento y día tiene forma (21, 7). El alcohol es el único departamento con su pico el viernes; los otros 20 lo tienen el domingo o el lunes.

## 5. Archivos generados

Todos los archivos se escriben en `data/processed/` al ejecutar el notebook. El detalle de cada uno está en el [`README.md`](../README.md#archivos-procesados-dataprocessed).

| Formato | Uso |
|---|---|
| Parquet | Tablas limpias y fichas (`orders_clean`, `products_clean`, `order_products_clean`, `product_profile`): conservan los tipos y ocupan menos |
| CSV | Resúmenes pequeños para abrir en Excel (informe de limpieza, segmentos, departamentos, líderes, tablas por día) |

La tabla maestra `instacart_master.parquet` (≈104 MB) no se versiona porque supera el límite de 100 MB por archivo de GitHub; se regenera ejecutando `lesson_4_data_merging.ipynb`.

## 6. Limitaciones

- `days_since_prior_order` tiene un tope de 30 días, por lo que el valor 30 agrupa "30 días o más" y las medias de espera están subestimadas.
- Los datos son una muestra: un cliente "ocasional" puede tener pedidos anteriores que no están en el archivo.
- Las tasas de recompra de productos con pocos pedidos no son confiables; por eso el análisis de líderes usa un mínimo de 1.000 pedidos.
- El 6 % de los pedidos no tiene productos registrados, así que los análisis por artículo se basan en 450.046 pedidos.

## 7. Conclusiones

Los datos de Instacart quedan listos para el análisis cuando cada problema de calidad se trata según su causa (eliminar, corregir o conservar), cada paso queda registrado en un informe de cambios y se valida con reglas automáticas, y cada unión entre tablas se comprueba con `validate` e `indicator`. Sobre esa base, los hábitos de compra se describen con cifras reproducibles: la demanda se concentra los domingos y lunes entre las 10 y las 15 h, un grupo pequeño de clientes frecuentes genera casi la mitad de los pedidos, y los productos básicos (leche, agua, fruta) son los que más se recompran.
