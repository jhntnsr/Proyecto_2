
# Random Forest – Modelado y Evaluación
Este documento describe la implementación de **cuatro modelos Random Forest** aplicados a un dataset con objetivo de clasificación binaria. Los modelos se ejecutan usando la librería **randomForest** en R.

---

## 📂 1. Preparación del Entorno

```r
if(!require(randomForest)){
    install.packages("randomForest")
    library(randomForest)
} else {
    library(randomForest)
}
set.seed(123)
```

---

## 📊 2. Exploración Inicial del Dataset

Antes de entrenar un modelo, es importante validar la estructura del dataset.

```r
cat("Estructura del dataset:\n")
print(str(data))

cat("\nPrimeras filas:\n")
print(head(data))
```

Se asume:

- `data` es un **data.frame** cargado previamente.
- `target` es la columna objetivo (0/1).

---

# 🌲 3. Modelo 1 – Random Forest para Clasificación Binaria

```r
modelo_binario <- randomForest(
    target ~ .,
    data = data,
    ntree = 500,
    mtry = floor(sqrt(ncol(data) - 1)),
    importance = TRUE
)

print(modelo_binario)
importance(modelo_binario)
varImpPlot(modelo_binario, main="Importancia de Variables - Modelo Binario")
```

---

# 🎨 4. Modelo 2 – Random Forest Multiclase

```r
if(length(unique(data$target)) > 2){
    modelo_multiclase <- randomForest(
        target ~ .,
        data = data,
        ntree = 400,
        mtry = floor(sqrt(ncol(data) - 1)),
        importance = TRUE
    )

    print(modelo_multiclase)
    varImpPlot(modelo_multiclase, main="Importancia - Modelo Multiclase")
} else {
    cat("\n(Multiclase no ejecutado: target solo tiene 2 clases)\n")
}
```

---

# 📈 5. Modelo 3 – Random Forest para Regresión

```r
data_reg <- data
data_reg$target <- as.numeric(as.character(data$target))

modelo_regresion <- randomForest(
    target ~ .,
    data = data_reg,
    ntree = 300,
    mtry = 4,
    importance = TRUE
)

print(modelo_regresion)
varImpPlot(modelo_regresion, main="Importancia - Modelo Regresión")
```

---

# 🧪 6. Modelo 4 – Random Forest con Train/Test (70/30)

```r
set.seed(123)
idx <- sample(1:nrow(data), 0.7 * nrow(data))

train <- data[idx, ]
test  <- data[-idx, ]

modelo_split <- randomForest(
    target ~ .,
    data = train,
    ntree = 500,
    mtry = floor(sqrt(ncol(data) - 1)),
    importance = TRUE
)

print(modelo_split)

pred <- predict(modelo_split, test)

tabla <- table(Real = test$target, Predicho = pred)
tabla

accuracy <- sum(diag(tabla)) / sum(tabla)
cat("\nAccuracy del modelo con Train/Test:", accuracy, "\n")
```

---

# ✅ 7. Conclusiones

Los cuatro modelos permiten:

- Evaluar la capacidad predictiva del dataset.
- Identificar variables más importantes mediante `importance()`.
- Comparar rendimiento con datos separados.
- Generar un modelo continuo útil para scores probabilísticos.

---

¿Necesitas este archivo también en **PDF, DOCX o IPYNB**?
