# La pobreza como limitante del acceso a servicios esenciales

## Notas sobre los datos crudos

Ver bitacora_fuentes.csv para detalle de origen, URL y fecha de descarga de cada fuente.

### Contenido
- `marco_geo/`: Marco Geoestadístico 2020 integrado (solo nivel municipal y estatal; se excluyeron AGEB y manzana por peso y porque no se usan en el análisis).
- `iter/`: Censo 2020, Principales resultados por localidad (ITER), cuarta edición (286 columnas). Incluye agregados a nivel localidad, municipal, estatal y nacional en el archivo crudo.
- `coneval/`: Concentrado de indicadores de pobreza municipal 2020 (versión completa, hoja "Concentrado municipal"), con series 2010-2015-2020.
- `denue/`: Directorio Estadístico Nacional de Unidades Económicas, corte noviembre 2020. Las unidades de interés están en las categorías "Comercio al por menor" (farmacias y supermercados) y "Servicios de salud y asistencia social" (guarderías). Sólo se conservaron los 3 archivos sectoriales con códigos SCIAN de interés.

### Notas
- **2,469 municipios** confirmados de forma independiente entre shapefile (`00mun.shp`), catálogo (`municipios.csv`), ITER y CONEVAL.
- `TAMLOC` no aplica a nivel municipal (viene marcado con `*`). El % de población rural debe calcularse desde el desglose por localidad del ITER (categorías `TAMLOC` 01-04, antes de agregar a municipio), no desde el archivo ya filtrado a totales municipales.
- CONEVAL: la población reportada en ese archivo es una **estimación calibrada a nivel estatal**, no un conteo directo — puede diferir de INEGI/CONAPO a nivel municipal. Usar `POBTOT` del ITER como denominador de tasas, no la población de CONEVAL.
- CONEVAL: los códigos `n.d.` (sin estimación, por muestra insuficiente o municipio de nueva creación) y `n.a.` (sin población en el indicador) deben tratarse como NA reales.
- Municipios sin estimación de pobreza 2020 por muestra insuficiente del Censo: Seybaplaya (04012), Honduras de la Sierra (07125), La Magdalena Tlaltelulco (29048). NA reales, documentados por CONEVAL.
- Municipios de nueva creación (pueden faltarles estimaciones 2010 y/o 2015): San Quintín (02006), Seybaplaya (04012), El Parral (07122), Emiliano Zapata (07123), Mezcalapa (07124), Capitán Luis Ángel Vidal (07120), Rincón Chamula San Pedro (07121), Honduras de la Sierra (07125), Coatetelco (17034), Xoxocotla (17035), Hueyapan (Morelos), Bacalar (23010), Puerto Morelos (23011).
- DENUE: la descarga histórica se hizo por **sector de actividad económica** (no por entidad). "Comercio al por menor" viene partido en 4 archivos por rango de código SCIAN; solo 2 de los 4 contienen códigos de interés (ver tabla). Se usó la descarga masiva (`inegi.org.mx/app/descarga`), **no** el visualizador actual del DENUE, que ya corre sobre Censos Económicos 2024 y rompería la comparabilidad temporal con el resto de las fuentes (2020).
- DENUE: `cve_ent`/`cve_mun` deben leerse como texto (`dtype=str`) para no perder ceros a la izquierda al cruzar con el catálogo de municipios.

### Hallazgos
| Servicio	                | Código	| Establecimientos |
| :--- | :---: | ---: |
| Supermercados	            | 462111	| 6,482
| Farmacias sin minisúper	| 464111	| 50,918
| Farmacias con minisúper	| 464112	| 11,707
| Guarderías privadas	    | 624411	| 6,977
| Guarderías públicas	    | 624412	| 3,874
| **Total consolidado**     |           | **79,958** |


## Configuración del entorno

Este proyecto usa [uv](https://docs.astral.sh/uv/) para gestionar dependencias y el entorno virtual.

### 1. Instalar uv

**macOS / Linux**

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

**Windows (PowerShell)**

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

Verifica la instalación:

```bash
uv --version
```

### 2. Sincronizar el entorno

Desde la raíz del proyecto:

```bash
uv sync
```

Esto crea el entorno virtual en `.venv/`, instala la versión de Python declarada en `pyproject.toml` si hace falta, e instala las dependencias exactas de `uv.lock`.


### 3. Ejecutar comandos

No es necesario activar el entorno; basta con anteponer `uv run`:

```bash
uv run python main.py
uv run jupyter lab
```

Si prefieres activarlo manualmente:

```bash
source .venv/bin/activate      # macOS / Linux
.venv\Scripts\activate         # Windows
```

Si alguien requiere una lista `requirements.txt` para usar con pip:

```bash
uv export --no-emit-project --no-hashes -o requirements.txt
```