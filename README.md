# Análisis de datos de Instacart

**Aprendiz:** Carlos Eduardo Herrera Sibaja  
**Programa:** Análisis y Desarrollo de Software  
**Última actualización:** 11/11/2026

## Propósito

Proyecto de análisis de los cinco archivos de Instacart, desarrollado por actividades:

| Actividad | Contenido | Notebook | Reporte |
|---|---|---|---|
| Entorno y exploración inicial | Entorno virtual, instalación de Pandas y JupyterLab, carga y exploración de las cinco tablas | [`notebooks/00_entorno.ipynb`](notebooks/00_entorno.ipynb), [`notebooks/01_exploracion_instacart.ipynb`](notebooks/01_exploracion_instacart.ipynb) | — |
| Diagnóstico inicial de calidad | Estructura, tipos, nulos, duplicados, rangos e integridad de claves | [`notebooks/data_quality.ipynb`](notebooks/data_quality.ipynb) | [`reports/data_quality.md`](reports/data_quality.md) |
| GA1-220501094-01-AA1-EV06: limpieza, análisis y estructuración con Pandas | Lecciones 2, 3 y 4: laboratorios guiados y 21 ejercicios de reto | [`notebooks/pandas_labs.ipynb`](notebooks/pandas_labs.ipynb) | [`reports/data_processing.md`](reports/data_processing.md) |

## Preguntas del proyecto

- **Pregunta analítica:** ¿Qué patrones descriptivos se observan en los pedidos y artículos de Instacart, en particular en el horario de compra y la recompra de productos?
- **Pregunta orientadora de la actividad EV06:** ¿Cómo preparar y estructurar los datos de Instacart para obtener información confiable sobre los hábitos de compra?

## Datasets

Los archivos CSV usan `;` como separador:

| Archivo | Una fila es | Clave |
|---|---|---|
| `instacart_orders.csv` | Un pedido: cliente, número de pedido, día (`order_dow`, 0 = domingo), hora y días desde el pedido anterior | `order_id` |
| `order_products.csv` | Un producto dentro de un pedido, su posición en el carrito y si es recompra | `order_id` + `product_id` |
| `products.csv` | Un producto del catálogo con su pasillo y departamento | `product_id` |
| `aisles.csv` | Un pasillo | `aisle_id` |
| `departments.csv` | Un departamento | `department_id` |

Los CSV originales se guardan localmente en `data/raw/` y están excluidos del control de versiones por su tamaño. Para obtenerlos, consulte la [carpeta de datasets de Instacart](https://drive.google.com/drive/folders/19vYGYdE0hI1DzvpBcCZskzJ0u2TZjwas) y coloque los cinco archivos en `data/raw/`. Los archivos de `data/raw/` no se modifican; todo resultado se escribe en `data/processed/`.

## Estructura

```text
instacart-analisis/
├── README.md
├── .gitignore
├── requirements.txt
│
├── data/
│   ├── raw/                          # CSV originales (locales, no versionados)
│   │   ├── instacart_orders.csv
│   │   ├── order_products.csv
│   │   ├── products.csv
│   │   ├── aisles.csv
│   │   └── departments.csv
│   └── processed/                    # Archivos generados por pandas_labs.ipynb
│
├── notebooks/
│   ├── 00_entorno.ipynb              # Actividad anterior: verificación del entorno
│   ├── 01_exploracion_instacart.ipynb  # Actividad anterior: exploración inicial
│   ├── data_quality.ipynb            # Actividad anterior: diagnóstico de calidad
│   └── pandas_labs.ipynb             # Lecciones 2, 3 y 4
│
└── reports/
    ├── data_quality.md               # Actividad anterior: informe de diagnóstico
    └── data_processing.md            # Informe de limpieza, análisis y combinación
```

## Preparar el entorno

Desde la carpeta raíz del proyecto, cree un entorno virtual e instale las dependencias.

En Windows PowerShell:

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

Si PowerShell bloquea la activación del entorno, ejecute antes `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`. En Linux o macOS active el entorno con `source .venv/bin/activate`.

## Ejecutar los notebooks

1. Verifique que los cinco CSV estén en `data/raw/`.
2. Abra Jupyter con `jupyter lab` (o abra el notebook en VS Code) y seleccione el kernel del entorno `.venv`.
3. Ejecute los notebooks completos con **Run All**. Para esta actividad basta con `notebooks/pandas_labs.ipynb` (alrededor de un minuto); los notebooks de las actividades anteriores (`00_entorno`, `01_exploracion_instacart`, `data_quality`) se pueden ejecutar de forma independiente. Las lecciones de `pandas_labs.ipynb` comparten variables, así que debe ejecutarse completo y en orden, no por celdas sueltas.

Alternativa sin abrir Jupyter, desde la carpeta `notebooks/`:

```bash
cd notebooks
jupyter nbconvert --to notebook --execute --inplace pandas_labs.ipynb
```

`pandas_labs.ipynb` crea la carpeta `data/processed/` si no existe y vuelve a generar todos sus archivos en cada ejecución.

## Contenido de `pandas_labs.ipynb`

| Lección | Ejemplos | Laboratorio guiado | Reto |
|---|---|---|---|
| 2. Limpieza y estandarización | Duplicados, tipos con `astype`, nulos, texto con `.str`, duplicados implícitos, funciones de limpieza | Pipeline de limpieza con `pipe` e informe de cambios | 7 ejercicios |
| 3. Filtrado, ordenamiento y agrupación | Máscaras booleanas, `query`, `sort_values`, `nlargest`, `groupby`/`agg`, `pivot_table`, `transform` | Perfil de hábitos de compra por segmento de cliente y franja horaria | 7 ejercicios |
| 4. Combinación de datasets | `merge` con `how`, `validate` e `indicator`, tabla maestra, `concat`, `pivot_table`/`melt`, Parquet | Tabla de análisis por producto y líder de recompra por departamento | 7 ejercicios |

Cada ejercicio de reto tiene su código, el resultado ejecutado y una interpretación breve. El notebook termina con conclusiones generales.

## Archivos procesados (`data/processed/`)

| Archivo | Lección | Contenido | ¿Versionado? |
|---|---|---|---|
| `orders_clean.parquet` | 2 | `orders` sin duplicados y con tipos optimizados (478.952 filas) | Sí |
| `products_clean.parquet` | 2 | `products` con nombres estandarizados y `Unknown` en lugar de nulos | Sí |
| `order_products_clean.parquet` | 2 | `order_products` con tipos optimizados (4.545.007 filas) | Sí |
| `cleaning_report.csv` | 2 | Informe de cambios del pipeline: filas, nulos y memoria por paso | Sí |
| `orders_by_weekday.csv` | 3 | Pedidos, clientes y espera media por día de la semana | Sí |
| `customer_segments.csv` | 3 | Clientes, pedidos y espera mediana por segmento | Sí |
| `segment_time_profile.csv` | 3 | Porcentaje de pedidos de cada segmento por franja horaria | Sí |
| `instacart_master.parquet` | 4 | Tabla maestra: 4.545.007 filas × 14 columnas con las cinco tablas unidas | **No** (≈104 MB, supera el límite de GitHub; se regenera con el notebook) |
| `department_summary.csv` | 4 | Artículos y tasa de recompra por departamento | Sí |
| `orders_by_hour_weekday_long.csv` | 4 | Pedidos por hora y día en formato largo | Sí |
| `product_profile.parquet` | 4 | Ficha por producto: veces, clientes, tasa de recompra y posición media | Sí |
| `department_leaders.csv` | 4 | Producto con mayor tasa de recompra de cada departamento (≥ 1.000 pedidos) | Sí |
| `items_by_department_weekday.csv` | 4 | Artículos por departamento y día de la semana | Sí |

Los archivos Parquet conservan los tipos de dato (`Int16`, `int8`, etc.); para leerlos use `pd.read_parquet(...)`, que requiere `pyarrow`.

## Resumen de resultados

**Limpieza (lección 2)**

| Tabla | Antes | Después | Cambio principal |
|---|---:|---:|---|
| `orders` | 478.967 filas, 23,0 MB | 478.952 filas, 7,2 MB | 15 duplicados eliminados; tipos enteros ajustados |
| `products` | 1.258 nombres nulos | 0 nombres nulos | Nulos → `Unknown`; 6.321 nombres estandarizados |
| `order_products` | 145,4 MB | 54,5 MB | Tipos ajustados; 836 nulos de posición conservados (límite de 64 en el carrito) |

**Hábitos de compra (lección 3)**

- Hora pico: 10 a. m. (40.578 pedidos); días con más pedidos: domingo y lunes.
- El 19,4 % de los clientes (frecuentes, 5 pedidos o más) hace casi la mitad de los pedidos y vuelve con una mediana de 6 días.
- Los clientes frecuentes compran más por la mañana (35,2 %) que los ocasionales (30,0 %).

**Combinación (lección 4)**

- La tabla maestra conserva las 4.545.007 filas de `order_products`, validada con `validate='m:1'`.
- 28.906 pedidos (6 %) no tienen productos registrados; un `inner` los habría eliminado sin aviso.
- Los productos más pedidos son Banana (66.050) y Bag Of Organic Bananas (53.297); los pasillos con mayor recompra son `milk`, `water seltzer sparkling water` y `fresh fruits`.

El detalle está en [`reports/data_processing.md`](reports/data_processing.md).

## Dependencias

`requirements.txt` fija las versiones exactas del entorno. Los paquetes principales son:

| Paquete | Versión | Uso |
|---|---|---|
| `pandas` | 3.1.0 | Limpieza, análisis y combinación de los datos |
| `pyarrow` | 26.0.1 | Lectura y escritura de archivos Parquet (nuevo en esta actividad) |
| `numpy` | 2.5.4 | Límite infinito (`np.inf`) en la segmentación con `pd.cut` |
| `jupyterlab` | 4.6.4 | Entorno para abrir y ejecutar los notebooks |
| `ipykernel` | 7.4.0 | Kernel de Python del entorno virtual |

Probado con Python 3.13.

## Control de versiones

| Rama | Contenido |
|---|---|
| `main` | Versión estable entregada |
| `develop` | Rama de integración; recibe las ramas de cada actividad |
| `feature/pandas-data-processing` | Actividad EV06: lecciones 2, 3 y 4 (fusionada en `develop`) |

Commits de la actividad EV06, uno por lección más la documentación:

| Avance | Commit |
|---|---|
| Lección 2 | `feat: complete lesson 2 data cleaning` |
| Lección 3 | `feat: complete lesson 3 data analysis` |
| Lección 4 | `feat: complete lesson 4 data merging` |
| Documentación | `docs: update README and processing report` |
