# Proyecto: Minería de Datos en R

## --- Instalar paquetes ---
```r
install.packages("rpart")
install.packages("rpart.plot")
```

## --- Ejecutar librerías ---
```r
library(haven)   # Para .sav
library(readxl)  # Para .xlsx
library(dplyr)
library(rpart)
library(rpart.plot)
library(arules)
```

## --- Función para limpiar nombres de columnas ---
```r
limpiar_nombres <- function(df) {
  names(df) <- tolower(names(df))
  names(df) <- gsub("[^a-z0-9]", "_", names(df))
  return(df)
}
```

## --- Función para convertir variables labelled ---
```r
convertir_labelled <- function(df) {
  df[] <- lapply(df, function(x) {
    if ("labelled" %in% class(x)) {
      if (is.numeric(x)) as.numeric(x)
      else as.character(x)
    } else x
  })
  df
}
```

## --- Crear lista de dataframes ---
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

## --- Identificar columnas comunes ---
```r
col_comunes_div <- Reduce(intersect, lapply(divorcios_list, names))
col_comunes_mat <- Reduce(intersect, lapply(matrimonios_list, names))
```

## --- Filtrar columnas comunes ---
```r
divorcios_list <- lapply(divorcios_list, function(df) df[, col_comunes_div, drop = FALSE])
matrimonios_list <- lapply(matrimonios_list, function(df) df[, col_comunes_mat, drop = FALSE])
```

## --- Combinar todos los años ---
```r
d_2009_2024 <- bind_rows(divorcios_list)
m_2009_2024 <- bind_rows(matrimonios_list)
```

## --- Revisar resultados ---
```r
dim(d_2009_2024)
dim(m_2009_2024)
names(d_2009_2024)
names(m_2009_2024)
```

## --- Verificar Data Frames ---
```r
data.frame(1:ncol(d_2009_2024), colnames(d_2009_2024))
data.frame(1:ncol(m_2009_2024), colnames(m_2009_2024))
```

---

# Árboles de Decisión

## --- Árbol para clase de unión ---
```r
arbolcu <- rpart(clauni ~ nachom + nacmuj + edadmuj + edadhom,
                 data = m_2009_2024, method = "class")

rpart.plot(arbolcu, type=2, extra=0, under=TRUE, fallen.leaves=TRUE, box.palette="BuGn",
           main="Clase de unión", cex=0.5)

claseu <- data.frame(edadmuj=50, edadhom=60, nachom=320, nacmuj=320)
result <- predict(arbolcu, claseu, type="prob")
result
```

## --- Árbol para mes de ocurrencia ---
```r
arbol_mesocu <- rpart(mesocu ~ edadmuj + edadhom + nachom + nacmuj,
                      data = m_2009_2024, method = "class")

rpart.plot(arbol_mesocu, type=2, extra=0, under=TRUE, fallen.leaves=TRUE, box.palette="BuGn",
           main="Árbol para mes de ocurrencia", cex=0.5)

mesdeocu <- data.frame(edadmuj=40, edadhom=45, nachom=320, nacmuj=320)
resultmo <- predict(arbol_mesocu, mesdeocu, type="prob")
resultmo
```

## --- Árbol para departamento de registro ---
```r
arbol_depreg <- rpart(depreg ~ mesreg + clauni + depocu,
                      data = m_2009_2024, method = "class")

rpart.plot(arbol_depreg, type=2, extra=0, under=TRUE, fallen.leaves=TRUE, box.palette="BuGn",
            main = "Árbol para departamento de registro", cex=0.5)

dep_regis <- data.frame(mesreg=1, clauni=1, depocu=1)
resultdr <- predict(arbol_depreg, dep_regis, type="prob")
resultdr
```

## --- Árbol para año de registro ---
```r
arbol_ar <- rpart(a_oreg ~ edadhom + edadmuj + clauni + depreg,
                  data = m_2009_2024, method = "class")

rpart.plot(arbol_ar, type=2, extra=0, under=TRUE, fallen.leaves=TRUE, box.palette="BuGn",
           main = "Árbol para año de registro", cex=0.5)

adereg <- data.frame(edadhom=21, edadmuj=18, clauni=1, depreg=1)
resultar <- predict(arbol_ar, adereg, type="prob")
resultar
```
