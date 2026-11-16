
# Delimitación de zonas inundadas con Sentinel-2 mediante K-TAU (Valle de Sula)

Este repositorio contiene el código en R Markdown (`inundacion_ktau_interseccion.Rmd`) que implementa un flujo completo de **delimitación de zonas inundadas** en la imagen Sentinel-2 del 12/11/2020 (tile T16PCB, Valle de Sula, Honduras), comparando el algoritmo de clustering robusto **K-TAU** frente a **K-Means** y a líneas base supervisadas y no supervisadas (NDWI, MNDWI, Otsu, SCL, regresión logística).

El análisis no se limita al preprocesamiento: carga y recorta la imagen en su CRS nativo, construye un espacio de 36 variables (vecindades 3 × 3 de NDVI, NDWI, Verde y NIR), ajusta K-TAU y K-Means, calibra los grupos mediante validación cruzada por bloques espaciales y evalúa contra la referencia de Copernicus (producto EMSR481) sobre la **intersección imagen-referencia**.

**Autor:** Wilmer Adonay Morales Cantarero  
**Fecha:** Noviembre 2026

---

## 📦 Datos de entrada

### 1. Imagen Sentinel-2 (`.SAFE`)

Debido a las restricciones de tamaño de GitHub, la imagen satelital original no se aloja en este repositorio. Debe descargarse desde Google Drive:

👉 https://drive.google.com/drive/folders/1pIeX5zR_eSjZSuZjgPj_CdWPsXsexMc5?usp=sharing

Descomprima el archivo (si aplica) y asegúrese de que la carpeta resultante se llame **exactamente**:

```
S2A_MSIL2A_20201112T161531_N0500_R140_T16PCB_20230303T113427.SAFE
```

Colóquela en el mismo directorio donde está el `.Rmd`. El script busca dentro de la carpeta, de forma recursiva:

| Archivo | Obligatorio | Uso |
|---|---|---|
| `*_B02_10m.jp2`, `*_B03_10m.jp2`, `*_B04_10m.jp2`, `*_B08_10m.jp2` | ✅ | Reflectancia (Azul, Verde, Rojo, NIR) |
| `*_SCL_20m.jp2` | ✅ | Máscara de nubes / calidad |
| `*_B11_20m.jp2` | ⚠️ Opcional | Sólo para la línea base MNDWI |
| `MTD_MSIL2A.xml` | ✅ | Offsets `BOA_ADD_OFFSET` y `BOA_QUANTIFICATION_VALUE` |

### 2. Vectores de referencia (Copernicus EMSR481)

Coloque en la subcarpeta `Copernicus_Vector/` los tres shapefiles del producto EMSR481 (cada uno con sus archivos `.shx`, `.dbf`, `.prj` y, preferiblemente, `.cpg`):

```
Copernicus_Vector/
├── EMSR481_AOI01_DEL_PRODUCT_observedEventA_r1_v1.shp   ✅ obligatorio
├── EMSR481_AOI01_DEL_PRODUCT_areaOfInterestA_r1_v1.shp  ✅ obligatorio
└── EMSR481_AOI01_DEL_PRODUCT_hydrographyA_r1_v1.shp     ⚠️ opcional
```

- **`observedEventA`** define los positivos (extensión inundada).
- **`areaOfInterestA`** define el dominio del estudio (intersección con la imagen).
- **`hydrographyA`** (opcional) excluye de la evaluación los píxeles de agua permanente.

Si los nombres o las rutas cambian, actualice los parámetros `ruta_evento`, `ruta_aoi` y `ruta_agua_ref` en el bloque `params:` del YAML del `.Rmd`.

---

## 🛠️ Requisitos previos

### Paquetes de R

El script verifica la presencia de todos los paquetes al inicio y detiene la ejecución si falta alguno:

```r
paquetes <- c("terra", "sf", "ktaucenters", "dplyr", "ggplot2", "knitr", "cluster")
```

Instálelos una sola vez desde la consola de R:

```r
install.packages(
  c("terra", "sf", "ktaucenters", "dplyr", "ggplot2", "knitr", "cluster"),
  repos = "https://cloud.r-project.org"
)
```

### Configuración del repositorio CRAN (¡importante!)

El `.Rmd` incluye al final un chunk que descarga el código fuente de `ktaucenters` para verificar sus constantes de sintonización (`download.packages(...)`). Si el mirror de CRAN no está configurado, el knit falla con:

```
Error in `contrib.url()`:
! trying to use CRAN without setting a mirror
```

**Solución:** antes de tejer el documento, ejecute en la consola:

```r
options(repos = c(CRAN = "https://cloud.r-project.org"))
```

Alternativamente, puede desactivar ese chunk poniendo `eval=FALSE` en su cabecera `{r}` si no necesita regenerar `parametros_ktaucenters.csv`.

---

## ▶️ Cómo ejecutar

1. Clone el repositorio y sitúese en el directorio del proyecto.
2. Descargue la carpeta `.SAFE` desde Google Drive y colóquela junto al `.Rmd` (ver sección anterior).
3. Asegúrese de tener los shapefiles de Copernicus en `Copernicus_Vector/`.
4. En la consola de R, configure el mirror de CRAN y abra el documento:
   ```r
   options(repos = c(CRAN = "https://cloud.r-project.org"))
   rmarkdown::render("inundacion_ktau_interseccion.Rmd")
   ```
   o simplemente pulse **Knit** en RStudio.

El knit tarda varios minutos: muestrea del orden de 40.000 píxeles etiquetados y ejecuta 500 réplicas de bootstrap por bloques espaciales.

### Parámetros ajustables

Todos los parámetros del análisis viven en el bloque `params:` del YAML (semilla, `n_train`, `n_eval`, `k_folds`, `bloque_m`, `k_min`/`k_max`, `k_fijo`, umbrales físicos de agua, `B_boot`, etc.). No hace falta editar el cuerpo del `.Rmd` para reproducir variantes.

---

## 📤 Salidas generadas

Al finalizar, el knit produce:

**Informe HTML** con las tablas de selección de K, firma de grupos, evaluación por método, IC al 95 %, sensibilidad a K, mapas de predicción y diagnósticos.

**Archivos CSV** en el directorio de trabajo:

| Archivo | Contenido |
|---|---|
| `evaluacion_metodos_cv.csv` | Métricas (Recall, Precision, F1, IoU) por método |
| `evaluacion_ic_bloques.csv` | Intervalos de confianza por bootstrap de bloques |
| `seleccion_k_ktau.csv` | Criterios de selección de K (costo, silueta, ARI) |
| `firma_grupos.csv` | Firma espectral de los grupos de K-TAU y K-Means |
| `evaluacion_predicciones_cv.csv` | Predicciones por píxel y pliegue |
| `parametros_ktaucenters.csv` | Constantes de sintonización (c1, c2, eficiencia) |
| `eficiencia_ktau_p36.csv` | Simulación MSE K-Means / K-TAU para p = 36 |

---

## 📁 Estructura esperada del proyecto

```
proyecto/
├── inundacion_ktau_interseccion.Rmd
├── S2A_MSIL2A_20201112T161531_N0500_R140_T16PCB_20230303T113427.SAFE/
│   ├── MTD_MSIL2A.xml
│   └── GRANULE/.../IMG_DATA/
│       ├── R10m/  *_B02_10m.jp2, *_B03_10m.jp2, *_B04_10m.jp2, *_B08_10m.jp2
│       └── R20m/  *_SCL_20m.jp2, *_B11_20m.jp2
└── Copernicus_Vector/
    ├── EMSR481_AOI01_DEL_PRODUCT_observedEventA_r1_v1.shp (+ .shx .dbf .prj)
    ├── EMSR481_AOI01_DEL_PRODUCT_areaOfInterestA_r1_v1.shp (+ .shx .dbf .prj)
    └── EMSR481_AOI01_DEL_PRODUCT_hydrographyA_r1_v1.shp    (+ .shx .dbf .prj)
```

---

## ⚠️ Limitaciones conocidas

- El modelo no supervisado ve (sin etiquetas) los píxeles de los pliegues retenidos: la validación es **transductiva**, no inductiva. Los bloques de 2 km adyacentes siguen correlacionados, por lo que los resultados son algo optimistas respecto a una zona totalmente nueva.
- La referencia `observedEventA` incluye vegetación y suelo anegado además de agua abierta; un detector puramente de agua (NDWI, MNDWI) puede tener `recall` bajo por diseño. El script reporta la diferencia entre "zona inundada" y "agua abierta".
- La línea base MNDWI sólo se calcula si `*_B11_20m.jp2` está presente en el `.SAFE`.

