# Informe técnico: diagnóstico de calidad de datos de Instacart

**Actividad:** GA1-220501094-01-AA1-EV06 – Diagnosticar la calidad de datos  
**Rol:** analista de datos de Instacart  
**Aprendiz:** jeferson estiven vargas  
**Notebook de soporte:** `notebooks/02_diagnostico_calidad.ipynb`  
**Tablas de soporte:** `reports/diagnostico_estructural.csv`, `reports/hallazgos_calidad.csv`

---

## 1. Contexto y pregunta analítica

Instacart es una plataforma de compras de supermercado en línea. En la EV04 se exploraron sus datos para identificar patrones de compra. Antes de profundizar en ese análisis es necesario saber si los datos son confiables.

**Pregunta analítica:**

> ¿Tienen los cinco conjuntos de datos de Instacart la calidad suficiente (completitud, tipos correctos y consistencia entre tablas) para analizar los patrones de compra de los clientes, y qué acciones de limpieza se necesitan antes de hacerlo?

## 2. Datos analizados

| Dataset | Filas | Columnas | Numéricas | Texto | Memoria (MB) |
|---|---:|---:|---:|---:|---:|
| `aisles` | 134 | 2 | 1 | 1 | 0,01 |
| `departments` | 21 | 2 | 1 | 1 | 0,00 |
| `instacart_orders` | 478.967 | 6 | 6 | 0 | 21,93 |
| `order_products` | 4.545.007 | 4 | 4 | 0 | 138,70 |
| `products` | 49.694 | 4 | 3 | 1 | 4,87 |
| **Total** | **5.073.823** | | | | **165,51** |

`order_products` concentra cerca del 90 % de las filas y el 84 % de la memoria, por lo que cualquier operación de limpieza sobre esa tabla debe planearse pensando en el rendimiento.

## 3. Metodología

El diagnóstico se hizo con Python y pandas en cinco revisiones:

1. **Inspección estructural:** `head`, `tail`, `shape`, `columns`, `dtypes`, `info` y `describe`.
2. **Completitud:** conteo y porcentaje de valores faltantes (`NaN`) por columna.
3. **Validez de tipos:** los identificadores y contadores deben ser enteros; se marcaron los que quedaron como decimales.
4. **Inconsistencias:**
   - filas idénticas y claves repetidas;
   - integridad entre tablas (`order_products` → `instacart_orders` y `products`; `products` → `aisles` y `departments`);
   - rangos válidos (día 0–6, hora 0–23, días entre pedidos 0–30, recompra 0/1);
   - textos con espacios extra, diferencias de mayúsculas y valores de relleno (`missing`, `unknown`).
5. **Clasificación del impacto** según el porcentaje de filas afectadas en su tabla: alto (≥ 5 %), medio (≥ 1 %) y bajo (< 1 %).

Todas las cifras se calculan con código en el notebook; ninguna se escribió a mano.

## 4. Resultados

Se registraron **10 hallazgos**: 2 de impacto alto, 2 de impacto medio y 6 de impacto bajo.

| # | Dimensión | Dataset.columna | Problema | Filas | % | Impacto |
|---:|---|---|---|---:|---:|---|
| 1 | Completitud | `instacart_orders.days_since_prior_order` | Valores faltantes | 28.819 | 6,02 | Alto |
| 2 | Validez (tipo) | `instacart_orders.days_since_prior_order` | Decimal en vez de entero | 28.819 | 6,02 | Alto |
| 3 | Consistencia | `departments.department` | Valor de relleno | 1 | 4,76 | Medio |
| 4 | Completitud | `products.product_name` | Valores faltantes | 1.258 | 2,53 | Medio |
| 5 | Consistencia | `aisles.aisle` | Valor de relleno | 1 | 0,75 | Bajo |
| 6 | Consistencia | `products.product_name` | Diferencias solo de mayúsculas | 104 | 0,21 | Bajo |
| 7 | Completitud | `order_products.add_to_cart_order` | Valores faltantes | 836 | 0,02 | Bajo |
| 8 | Validez (tipo) | `order_products.add_to_cart_order` | Decimal en vez de entero | 836 | 0,02 | Bajo |
| 9 | Consistencia | `products.product_name` | Espacios al inicio o al final | 4 | 0,01 | Bajo |
| 10 | Unicidad | `instacart_orders` | Filas idénticas repetidas | 15 | 0,00 | Bajo |

**Lo que no presentó problemas:**

- **Integridad entre tablas:** todos los `order_id`, `product_id`, `aisle_id` y `department_id` de las tablas hijas existen en su tabla principal.
- **Rangos:** ningún valor de día, hora, días entre pedidos, recompra, número de pedido o posición en el carrito está fuera de su rango válido.
- **Claves repetidas:** los únicos `order_id` repetidos corresponden a las 15 filas idénticas; no hay pedidos distintos con el mismo identificador.

**Lectura de los hallazgos principales:**

- **Hallazgos 1 y 2:** son el mismo problema visto desde dos dimensiones. La columna quedó como decimal porque pandas no puede guardar `NaN` en una columna de enteros. La hipótesis es que el faltante corresponde al **primer pedido** de cada cliente, que no tiene un pedido anterior. *Pendiente de verificar:* comparar estas filas con `order_number == 1`.
- **Hallazgos 3 y 5:** en las tablas pequeñas (21 departamentos y 134 pasillos), un solo valor de relleno da un porcentaje alto. El impacto real depende de cuántos productos están asociados a esa categoría.
- **Hallazgo 4:** 1.258 productos sin nombre (2,53 %). Afecta los análisis por producto, pero no la integridad, porque los productos siguen teniendo su `product_id`.
- **Hallazgos 7 y 8:** 836 filas sin posición en el carrito; su peso es mínimo (0,02 %).

## 5. Limitaciones

**De los datos:**

- No hay fechas reales de los pedidos, solo el día de la semana, la hora y los días desde el pedido anterior. No se pueden estudiar tendencias mensuales ni temporadas.
- No hay precios, cantidades ni información de los clientes.
- `days_since_prior_order` parece tener un tope de 30 días, así que los clientes que tardan más en volver quedan agrupados en ese valor.

**Del diagnóstico:**

- Los rangos válidos son reglas asumidas según el significado de cada columna; no vienen de un diccionario de datos oficial.
- La revisión de textos no detecta errores de ortografía ni productos con nombres parecidos.
- Las causas de los faltantes son hipótesis; deben verificarse antes de limpiar.
- En esta actividad solo se diagnostica: no se modificaron los datos.

## 6. Acciones recomendadas (en orden de prioridad)

1. **`days_since_prior_order`:** no eliminar las filas. Verificar la relación con `order_number == 1`; luego convertir a `Int64` (entero que admite faltantes) o crear una columna `es_primer_pedido`.
2. **`products.product_name`:** reemplazar los faltantes por `"Unknown"`; aplicar `str.strip()` y normalizar mayúsculas antes de agrupar por nombre.
3. **Valores de relleno en `departments` y `aisles`:** tratarlos como una categoría explícita `"Unknown"` y revisar cuántos productos pertenecen a ella.
4. **`instacart_orders`:** eliminar las 15 filas duplicadas con `drop_duplicates()`.
5. **`order_products.add_to_cart_order`:** revisar la causa de los 836 faltantes y convertir a `Int64`.

## 7. Conclusión

Los datos de Instacart tienen **buena calidad general**:

- no hay problemas de integridad entre tablas ni valores fuera de rango;
- 8 de los 10 hallazgos afectan menos del 5 % de las filas de su tabla.

El único problema relevante, los faltantes de `days_since_prior_order`, probablemente tiene una explicación lógica: el primer pedido de cada cliente. Por eso **no debe resolverse eliminando filas**.

**Respuesta a la pregunta analítica:** sí, los datos son adecuados para analizar patrones de compra, siempre que antes se apliquen las acciones del punto 6, en especial el tratamiento de los faltantes y la normalización de los nombres de productos.
