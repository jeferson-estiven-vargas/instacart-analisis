# Exploración de datos de Instacart con Python y Pandas

Actividad **GA1-220501094-01-AA1-EV04 – Fundamentos en Pandas**
*Explorando Instacart: descubre patrones de compra.*

Aprendiz: jeferson estiven vargas

## Contenido

| Ruta | Descripción |
|---|---|
| `notebooks/00_entorno.ipynb` | Comprobación del entorno (pandas) y función `franja` |
| `notebooks/01_exploracion_instacart.ipynb` | **Entregable EV04**: los 6 momentos de la actividad |
| `data/` | CSV de Instacart (no se versionan; solo existe `.gitkeep`) |
| `requirements.txt` | Dependencias del entorno (generado con `pip freeze`) |

## Datos

Los CSV usan `;` como separador y **no se suben al repositorio** (`data/*.csv` está en `.gitignore`).
Colócalos en `data/` con estos nombres:

`instacart_orders.csv` · `products.csv` · `aisles.csv` · `departments.csv` · `order_products.csv`

## Reconstruir el entorno

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
jupyter lab
```

En macOS/Linux la activación es `source .venv/bin/activate`.

## Flujo de ramas y versiones

- `main`: solo versiones estables.
- `develop`: integración.
- `feat/*`: una rama por fase de trabajo, integrada con `merge --no-ff`.

| Tag | Contenido |
|---|---|
| `v0.1` | Estructura, `.gitignore` y entorno virtual |
| `v0.2` | pandas instalado y `requirements.txt` |
| `v0.3` | JupyterLab en el mismo entorno |
| `v0.4` | Cuaderno `00_entorno` con pandas y `franja` |



## EV06 – Diagnóstico de calidad de datos

**Actividad:** GA1-220501094-01-AA1-EV06 – Diagnosticar la calidad de datos

| Ruta | Descripción |
|---|---|
| `notebooks/02_diagnostico_calidad.ipynb` | **Entregable EV06**: carga, inspección estructural, diagnóstico de calidad, hallazgos y limitaciones |
| `reports/informe_diagnostico.md` | Informe técnico con resultados, limitaciones y acciones recomendadas |
| `reports/diagnostico_estructural.csv` | Filas, columnas, tipos y memoria de cada dataset |
| `reports/hallazgos_calidad.csv` | Hallazgos de calidad con su impacto y la acción propuesta |

**Resultado principal:** 10 hallazgos de calidad (2 de impacto alto, 2 medio y 6 bajo). No hay problemas de integridad entre tablas ni valores fuera de rango.

### Cómo reproducir el diagnóstico

1. Copiar los 5 CSV de Instacart en la carpeta `data/` (no se suben a GitHub).
2. Crear el entorno e instalar las dependencias desde Git Bash:

   ```bash
   python -m venv .venv
   source .venv/Scripts/activate
   pip install -r requirements.txt
   ```

3. Abrir `notebooks/02_diagnostico_calidad.ipynb` y ejecutar **Run All**. La última celda debe mostrar 6 líneas `OK`, y se regeneran las tablas de `reports/`.
