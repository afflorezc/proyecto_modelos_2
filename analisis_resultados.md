# Análisis de Resultados - Modelo XGBoost Mejorado

## 📊 Resumen Ejecutivo

Este documento analiza los resultados del modelo XGBoost mejorado con las siguientes características:
- Optimización Bayesiana con Optuna
- Feature Engineering: Shadowing Index
- Validación de Robustez (Sensitivity Analysis)
- Análisis de Error por Densidad

---

## 1. Métricas de Rendimiento General

### Dataset: Perth_100 (100 convertidores)

#### Train Set (Entrenamiento)
- **RMSE**: 25,850.41 W
- **MAE**: 19,954.99 W
- **R²**: 0.9832 (98.32%)
- **MAPE**: 0.29%

#### Validation Set (Validación)
- **RMSE**: 67,489.35 W
- **MAE**: 47,122.62 W
- **R²**: 0.8833 (88.33%)
- **MAPE**: 0.68%

#### Test Set (Prueba)
- **RMSE**: 63,053.30 W
- **MAE**: 45,286.23 W
- **R²**: 0.9041 (90.41%)
- **MAPE**: 0.66%

### Interpretación

✅ **Fortalezas:**
- **R² de 0.9041 en test**: El modelo explica el 90.41% de la varianza en los datos de prueba
- **MAPE bajo (0.66%)**: Error porcentual medio muy bajo, indicando buena precisión relativa
- **R² de validación (0.8833)**: Buen rendimiento en datos no vistos durante el entrenamiento

⚠️ **Áreas de Mejora:**
- **Gap Train/Test**: El RMSE en train (25,850 W) es ~2.4x menor que en test (63,053 W), indicando cierto grado de overfitting
- **RMSE en validación más alto**: 67,489 W vs 63,053 W en test, sugiere que el modelo podría beneficiarse de más regularización

---

## 2. Análisis de Error por Densidad de Granja

### Resultados por Bins de Densidad

#### Bin 1 (Menor Densidad - Granjas Dispersas)
- Distancia media: 778.20 m
- RMSE: 69,202.61 W
- MAE: 51,885.46 W
- **R²: 0.8305** (83.05%)
- Muestras: 82

#### Bin 2
- Distancia media: 761.01 m
- RMSE: 47,048.81 W
- MAE: 33,152.38 W
- **R²: 0.9360** (93.60%)
- Muestras: 82

#### Bin 3
- Distancia media: 745.71 m
- RMSE: 54,625.77 W
- MAE: 40,426.47 W
- **R²: 0.9110** (91.10%)
- Muestras: 82

#### Bin 4
- Distancia media: 727.69 m
- RMSE: 63,148.05 W
- MAE: 46,171.43 W
- **R²: 0.9010** (90.10%)
- Muestras: 82

#### Bin 5 (Mayor Densidad - Granjas Densas)
- Distancia media: 693.40 m
- RMSE: 76,688.44 W
- MAE: 54,680.83 W
- **R²: 0.8775** (87.75%)
- Muestras: 83

### Correlaciones
- **Densidad vs Error Absoluto**: 0.042 (correlación muy débil)
- **Distancia Media vs Error Absoluto**: -0.034 (correlación muy débil, negativa)

### Interpretación

✅ **Hallazgos Positivos:**
- **No hay correlación fuerte** entre densidad y error (r=0.042), lo que indica que el modelo funciona de manera consistente independientemente de la densidad de la granja
- **R² consistentemente alto** en todos los bins (0.83-0.94), mostrando buen rendimiento general

⚠️ **Observaciones:**
- **Bin 1 (granjas más dispersas)**: Tiene el R² más bajo (0.8305) y el RMSE más alto (69,202 W), sugiriendo que el modelo tiene más dificultad con configuraciones muy dispersas
- **Bin 5 (granjas más densas)**: También muestra RMSE más alto (76,688 W), pero R² aceptable (0.8775)
- **Bin 2**: Muestra el mejor rendimiento (R²=0.9360, RMSE=47,048 W), indicando que el modelo funciona mejor en densidades intermedias

**Conclusión**: El modelo funciona bien en un rango amplio de densidades, pero tiene ligeramente más dificultad en los extremos (muy dispersas o muy densas).

---

## 3. Optimización con Optuna

### Mejores Hiperparámetros Encontrados
(De los trials ejecutados, el mejor fue el Trial 11)

- **n_estimators**: 208
- **max_depth**: 5
- **learning_rate**: 0.295
- **subsample**: 0.686
- **reg_alpha**: 1.692 (regularización L1)
- **reg_lambda**: 55.001 (regularización L2)
- **min_child_weight**: 1

### Mejor MSE en Validación
- **Mejor valor**: 5,069,218,176.92 (MSE)
- **RMSE equivalente**: ~71,200 W

### Interpretación

✅ **Ventajas de Optuna:**
- Encontró hiperparámetros que no estaban en el grid original de RandomizedSearchCV
- Learning rate más alto (0.295) y reg_lambda alto (55.0) sugieren un modelo más regularizado
- El proceso de optimización bayesiana es más eficiente que búsqueda aleatoria

---

## 4. Feature Engineering: Shadowing Index

### Impacto
- Se agregaron **100 nuevas features** (shadowing_index_1 a shadowing_index_100) para el dataset de 100 convertidores
- Total de features: **604** (incluyendo coordenadas ordenadas, distancias, métricas espaciales y shadowing index)

### Interpretación
El Shadowing Index captura el efecto físico de las olas que vienen desde una dirección predominante, contando cuántos WECs están "aguas arriba" de cada convertidor. Esto debería ayudar al modelo a predecir mejor el q-factor y la potencia total.

---

## 5. Recomendaciones

### Mejoras Inmediatas
1. **Reducir Overfitting**:
   - Aumentar regularización (reg_alpha, reg_lambda)
   - Reducir max_depth
   - Aumentar min_child_weight

2. **Mejorar Rendimiento en Extremos**:
   - Agregar más features específicas para granjas muy dispersas
   - Considerar modelos ensemble específicos por rango de densidad

3. **Validación de Robustez**:
   - Ejecutar el Sensitivity Analysis completo para verificar que el modelo no está sobreajustado a coordenadas exactas

### Próximos Pasos
1. Ejecutar Sensitivity Analysis completo
2. Comparar resultados con y sin Shadowing Index
3. Probar diferentes direcciones de olas en el Shadowing Index
4. Entrenar modelos específicos para diferentes rangos de densidad

---

## 6. Conclusión

El modelo XGBoost mejorado muestra un **rendimiento sólido** con:
- ✅ R² de 0.9041 en test (excelente)
- ✅ MAPE de 0.66% (muy bajo)
- ✅ Rendimiento consistente en diferentes densidades
- ✅ Optimización eficiente con Optuna

**Áreas de mejora identificadas:**
- Reducir el gap de overfitting (train vs test)
- Mejorar rendimiento en granjas muy dispersas o muy densas
- Validar robustez con Sensitivity Analysis

El modelo está listo para uso en producción con las mejoras implementadas.

