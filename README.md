# 📊 Análisis y modelado de la producción de pozos no convencionales de YPF en Vaca Muerta

> Proyecto de Data Science orientado al análisis exploratorio, aprendizaje supervisado y clustering de pozos no convencionales operados por YPF en la formación Vaca Muerta.

## 🧭 Resumen del proyecto

El proyecto analiza el comportamiento productivo de pozos no convencionales de YPF asociados a Vaca Muerta durante sus primeros 24 meses calendario desde el inicio de actividad efectiva. El trabajo integra información técnica y registros mensuales de producción, desarrolla un proceso de análisis exploratorio y preparación de datos, construye un modelo supervisado para estimar la producción acumulada de petróleo a 24 meses y aplica aprendizaje no supervisado para identificar perfiles productivos y operativos similares entre pozos.

## 🗂️ Fuentes de datos

Se utilizaron dos datasets públicos de la **Secretaría de Energía de la Nación Argentina**, relacionados mediante el identificador `idpozo`: [Capítulo IV — Listado de pozos](http://datos.energia.gob.ar/dataset/produccion-de-petroleo-y-gas-por-pozo/archivo/cb5c0f04-7835-45cd-b982-3e25ca7d7751), con información técnica y descriptiva de cada pozo, y [Producción de pozos de gas y petróleo no convencional](http://datos.energia.gob.ar/dataset/produccion-de-petroleo-y-gas-por-pozo/archivo/b5b58cdc-9e07-41f9-b392-fb9ec68b0725), con registros mensuales de petróleo, gas, agua y tiempo efectivo de producción (`tef`). El universo de análisis se restringió a pozos operados por YPF, de recurso no convencional y asociados a la formación Vaca Muerta.

## 🎯 Objetivo

El objetivo fue estudiar el comportamiento de los pozos durante sus primeros 24 meses desde el inicio de actividad efectiva desde dos perspectivas complementarias: **aprendizaje supervisado**, estimando la producción acumulada de petróleo a 24 meses (`pet_24m`) a partir de información disponible al comienzo del pozo, y **aprendizaje no supervisado**, identificando grupos de pozos con perfiles productivos y operativos similares sin utilizar una etiqueta objetivo.

## 🔎 Principales hallazgos del análisis exploratorio

El AED mostró distribuciones productivas fuertemente asimétricas, presencia de ceros y numerosos valores extremos, por lo que los outliers no se eliminaron automáticamente mediante IQR y se analizaron con criterios de plausibilidad sobre los meses activos. Para construir una ventana temporal comparable se definió como mes 1 el primer registro con `tef > 0` y se conservaron los pozos con 24 meses calendario completos, permitiendo interrupciones posteriores; así se obtuvo una cohorte final de **1.431 pozos y 34.344 registros pozo-mes**. También se identificaron diferencias productivas entre grupos y cambios temporales en la distribución de la producción, aspecto relevante para evaluar la capacidad de generalización de los modelos.

## 🤖 Modelos de aprendizaje supervisado

Para regresión se compararon **Dummy Regressor, Regresión Lineal, Árbol de Decisión y Random Forest**, utilizando validación temporal y MAE como criterio principal de selección; además se optimizó Random Forest mediante búsqueda de hiperparámetros. La **Regresión Lineal** obtuvo el mejor MAE promedio de validación temporal (≈ 11.996) y fue seleccionada antes de evaluar el test; sobre pozos temporalmente posteriores alcanzó **MAE ≈ 17.135, RMSE ≈ 21.061 y R² ≈ 0,22**, lo que indica capacidad predictiva moderada. La clasificación binaria alta/baja explorada previamente no se mantuvo como modelo final porque el cambio temporal de la distribución hacía que cerca del 96 % del test quedara en una única clase, por lo que modificar el umbral utilizando el test habría introducido sesgo.

## 🧩 Aprendizaje no supervisado

Para clustering se aplicó **K-Means** sobre variables de producción acumulada, continuidad operativa y concentración temprana de producción. Debido a las diferencias de escala y a la asimetría de los acumulados se comparó una representación base estandarizada con una versión mejorada aplicando `log1p` a petróleo, gas y agua antes del escalado; el **Silhouette aumentó de 0,565 a 0,613**, por lo que se seleccionó la representación mejorada con **k = 2**, considerando además el método del codo y un tamaño mínimo de cluster del 5 %. El Cluster 0 agrupó **1.259 pozos (87,98 %)** con un perfil predominantemente petrolero y sostenido, mientras que el Cluster 1 reunió **172 pozos (12,02 %)** con muy baja continuidad petrolera, mayor producción de gas y una mayor presencia de interrupciones operativas.

## ✅ Conclusiones

El trabajo mostró que las características disponibles al inicio del pozo contienen información útil para estimar la producción futura, aunque la capacidad de generalización es limitada y una parte importante de la variabilidad productiva queda sin explicar. El clustering permitió complementar esa mirada predictiva identificando dos perfiles claramente diferenciados dentro de la cohorte. Los resultados se interpretan como **patrones y asociaciones presentes en los datos, no como relaciones causales**, y deben considerarse como apoyo analítico que no reemplaza criterios especializados de ingeniería, geología u operación.

## 🛠️ Tecnologías utilizadas

Python · Pandas · NumPy · Matplotlib · Seaborn · Scikit-learn · Google Colab

---

**Proyecto académico de Data Science — Grupo 3**
