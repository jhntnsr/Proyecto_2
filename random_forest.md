# Guía de uso: Integración y Modelado con Random Forest (Matrimonios y Divorcios 2009–2024)

Este documento describe, paso a paso, el flujo para:
1) Cargar y estandarizar datasets de **matrimonios** y **divorcios** (2009–2024) en formatos `.sav` y `.xlsx`.
2) Unificar columnas comunes y combinar en dataframes maestros.
3) Entrenar tres modelos **Random Forest** para:
   - **Clase de Unión** (`clauni`)
   - **Mes de Ocurrencia** (`mesocu`)
   - **Departamento de Registro** (`depreg`)
4) Generar métricas (matriz de confusión y exactitud) y **predicciones** (clase y probabilidades).

---

## 1. Requisitos

- **R** (>= 4.2 recomendado)
- Paquetes:
  - `haven` (lectura de `.sav`)
  - `readxl` (lectura de `.xlsx`)
  - `dplyr`
  - `arules`
  - `randomForest`

### Instalación de paquetes
```r
install.packages("randomForest")
# Si aún no están instalados:
install.packages(c("haven", "readxl", "dplyr", "arules"))
```

### Cargar librerías
```r
library(haven)   # Para .sav
library(readxl)  # Para .xlsx
library(dplyr)
library(arules)
library(randomForest)
```

---

## 2. Estructura esperada de archivos

Coloque los archivos en la ruta:
```
D:/Documentos/Maestria/Introduccion a la Mineria de Datos/Proyecto_1/
```

Con la siguiente convención de nombres:

- **Divorcios**: `Divorcios_<año>.<ext>`
- **Matrimonios**: `matrimonios_<año>.<ext>`

> **Extensión**: Para **2023–2024** se espera `.xlsx`, para **2009–2022** se espera `.sav`.

Ejemplos:
- `Divorcios_2016.sav`, `Divorcios_2023.xlsx`
- `matrimonios_2011.sav`, `matrimonios_2024.xlsx`

---

## 3. Utilidades para limpieza y conversión

### Limpiar nombres de columnas
Convierte a minúsculas y reemplaza cualquier carácter no alfanumérico por `_`.
```r
limpiar_nombres <- function(df) {
  names(df) <- tolower(names(df))
  names(df) <- gsub("[^a-z0-9]", "_", names(df))
  return(df)
}
```

### Convertir variables `labelled` a numérico/carácter
Convierte columnas de clase `labelled` (de `haven`) a tipos base.
```r
convertir_labelled <- function(df) {
  df[] <- lapply(df, function(x) {
    if ("labelled" %in% class(x)) {
      if (is.numeric(x)) as.numeric(x) else as.character(x)
    } else x
  })
  df
}
```

---

## 4. Carga de datos por año (2009–2024)

```r
años <- 2009:2024

# Divorcios
divorcios_list <- lapply(años, function(a) {
  file_ext <- if (a %in% 2023:2024) ".xlsx" else ".sav"
  file_path <- paste0("D:/Documentos/Maestria/Introduccion a la Mineria de Datos/Proyecto_1/Divorcios_", a, file_ext)
  
  df <- if (file_ext == ".sav") read_sav(file_path) else read_excel(file_path)
  df <- limpiar_nombres(df)
  df <- convertir_labelled(df)
  return(df)
})

# Matrimonios
matrimonios_list <- lapply(años, function(a) {
  file_ext <- if (a %in% 2023:2024) ".xlsx" else ".sav"
  file_path <- paste0("D:/Documentos/Maestria/Introduccion a la Mineria de Datos/Proyecto_1/matrimonios_", a, file_ext)
  
  df <- if (file_ext == ".sav") read_sav(file_path) else read_excel(file_path)
  df <- limpiar_nombres(df)
  df <- convertir_labelled(df)
  return(df)
})
```

---

## 5. Alineación de columnas y combinación

```r
# Columnas comunes
col_comunes_div <- Reduce(intersect, lapply(divorcios_list, names))
col_comunes_mat <- Reduce(intersect, lapply(matrimonios_list, names))

# Filtrar sólo columnas comunes
divorcios_list <- lapply(divorcios_list, function(df) df[, col_comunes_div, drop = FALSE])
matrimonios_list <- lapply(matrimonios_list, function(df) df[, col_comunes_mat, drop = FALSE])

# Unificar
d_2009_2024 <- dplyr::bind_rows(divorcios_list)
m_2009_2024 <- dplyr::bind_rows(matrimonios_list)
```

---

## 6. Modelos Random Forest

Incluye ejemplos para `clauni`, `mesocu` y `depreg`.

---

## 7. Buenas prácticas

- Ajustar `ntree` y `mtry`.
- Validación cruzada.
- Manejo de valores especiales (`999`).

---

## 8. Conclusión

Este flujo permite integrar datos y entrenar modelos predictivos robustos para análisis de registros civiles.

---
