# Demand Forecasting FMCG – MVP Técnico Defendible

Proyecto demostrativo de predicción de demanda semanal en retail utilizando el dataset **Kaggle – Favorita Grocery Sales Dataset**.

Este repositorio demuestra criterio técnico en modelado supervisado para forecasting en retail, con control explícito de leakage, validación temporal realista y métricas apropiadas para presencia de ceros.

No es un sistema productivo.
Es un MVP diseñado para demostrar fundamentos sólidos.

## 1. Problema de Negocio

En retail, errores de predicción generan:
* Sobreinventario → capital inmovilizado.
* Stockouts → pérdida directa de ingresos.
* Mala asignación promocional.

Objetivo del MVP:

Predecir ventas agregadas semanales (t+1) por producto en una tienda específica.

Reformulación técnica del problema:

Dado un histórico semanal por producto, estimar la demanda de la siguiente semana usando únicamente información disponible hasta t.

Restricción crítica:
No se permite uso de información futura (no leakage).

## 2. Definición Formal del Problema

Sea:

yᵢₜ = ventas semanales del producto i en semana t

Se construye un modelo global:

f(Xᵢₜ) → yᵢₜ₊₁

Donde Xᵢₜ contiene:
* Variables calendario
* Indicador promocional agregado
* Lags históricos

Rolling means desplazadas

## 3. Decisiones Técnicas y Justificación
### 3.1 Agregación semanal

Se agrega la demanda diaria a nivel semanal:

* Reduce ruido.
* Reduce sparsity.
* Simplifica modelado.
* Evita complejidad innecesaria para un MVP.

Debilidad reconocida:
Se pierde granularidad intra-semana.

### 3.2 Modelo Global vs Modelo por Producto

Se entrena un solo modelo para todos los productos seleccionados.

Justificación:

* Mayor cantidad de datos efectivos.
* Permite capturar patrones compartidos.
* Reduce riesgo de overfitting en series cortas.
* Simula escenarios reales con cientos de SKUs.

Pregunta incómoda que debe poder responderse:

¿Por qué no entrenar 15 modelos separados?

Respuesta técnica:

Porque cada serie individual tiene pocas observaciones tras agregación semanal. El modelo global reduce varianza al compartir estructura entre productos.

### 3.3 Exclusión de Demanda Intermitente

Se excluyen productos con >40% ceros.

No es una decisión cómoda. Es una decisión metodológica.

Razón:

El objetivo del MVP no es modelar demanda intermitente.
Eso requeriría métodos como:

* John Croston (método Croston)
* Modelos de probabilidad cero-inflados
* Separación clasificación + regresión

No implementarlos evita pseudo-complejidad mal resuelta.

## 4. Data Leakage — Definición y Control

Data leakage ocurre cuando el modelo utiliza información que no estaría disponible al momento de predecir.

En este contexto, el leakage suele ocurrir en:

*  Rolling windows sin shift.
* Agregaciones mal construidas.
* Mezclar productos en cálculos de lag.

Ejemplo incorrecto:

rolling_mean_3 = unit_sales.rolling(3).mean()

Incluye la semana actual → invalida la evaluación.

Ejemplo correcto:

rolling_mean_3 = unit_sales.shift(1).rolling(3).mean()

Además:

Todos los lags y rollings se calculan usando groupby("item_nbr") para evitar contaminación cruzada entre productos.

## 5. Validación Temporal

Split único:

* Train: 2013–2016
* Test: 2017

No se usa shuffle.

Justificación:

En forecasting, mezclar pasado y futuro en entrenamiento invalida el escenario real.

Pregunta difícil:

¿Por qué no usar TimeSeriesSplit?

Respuesta defendible:

Para un MVP demostrativo con horizonte t+1, un holdout temporal único es suficiente para mostrar mejora frente al baseline. Cross-validation avanzada es útil en producción, pero no necesaria para validar fundamentos.

## 6. Modelos

Baseline:

Naive → ŷₜ₊₁ = yₜ

Esto es obligatorio.
Si el modelo no supera esto, el proyecto falla.

Modelos entrenados:
* Regresión Lineal
* RandomForestRegressor

No se usa:
* XGBoost
* LSTM
* Transformers

Pregunta incómoda:

¿Por qué no usar LSTM?

Respuesta sólida:
* El dataset agregado es pequeño.
* Horizonte t+1.
* No hay secuencias largas complejas.
* Random Forest captura no linealidades con menor complejidad.
* Mejor interpretabilidad.

En un MVP demostrativo, complejidad adicional sin ganancia clara es mala ingeniería.

## 7. Métricas y Problema de MAPE

Retail contiene ceros.

MAPE = |y − ŷ| / y

Si y ≈ 0 → métrica explota.

Por eso:

Se reportan:
* MAE
* RMSE
* WMAPE
* MAPE solo donde y > 0

WMAPE = Σ|y − ŷ| / Σy

Es más estable y refleja impacto real en volumen.

Pregunta incómoda:

¿WMAPE penaliza igual productos de bajo volumen?

No.
Pondera por volumen total.
Eso es coherente si el objetivo es impacto financiero agregado.

## 8. Resultados Esperados

El proyecto se considera técnicamente válido si:

* Supera baseline Naive.
* WMAPE ≤ 18%.
* No existe leakage.
* El error por producto es analizado y explicado.

Si Random Forest no supera al Naive, el modelo no se justifica.

## 9. Escalabilidad

Pregunta crítica:

¿Cómo escalar esto a 50 tiendas?

Respuesta técnica:

Agregar dimensión store_nbr como feature categórica y mantener modelo global.

Riesgo:

* Heterogeneidad alta entre tiendas → posible necesidad de modelos jerárquicos o embeddings categóricos.

Pero eso está fuera del alcance del MVP.

## 10. Limitaciones Declaradas

Este proyecto:
* No modela demanda intermitente.
* No implementa Croston.
* No hace forecasting multi-step.
* No optimiza inventario.
* No está desplegado.
* No realiza tuning exhaustivo.
* No implementa validación cruzada temporal avanzada.

Estas limitaciones son intencionales para mantener foco pedagógico.