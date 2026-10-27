# Análisis y modelado predictivo de la producción de petróleo en pozos no convencionales de YPF en Vaca Muerta

**Grupo 3 — Data Science**  
**Programa Energía Digital — Fundación YPF / Jump Educación**

---

## Descripción del proyecto

Este proyecto analiza el comportamiento productivo de pozos no convencionales operados por **YPF** y asociados a la formación **Vaca Muerta**.

Se integran datos técnicos de los pozos con registros mensuales de producción para construir un flujo de trabajo completo de Data Science que incluye:

- exploración y control de calidad de datos;
- transformación y feature engineering;
- construcción de una cohorte temporal comparable;
- prevención de fuga de información (*data leakage*);
- modelos supervisados de regresión y clasificación;
- validación cruzada temporal;
- análisis de sobreajuste;
- optimización de hiperparámetros;
- evaluación final sobre un conjunto de prueba temporal.

El objetivo principal es estimar la **producción acumulada de petróleo durante los primeros 24 meses calendario desde el inicio de actividad efectiva de cada pozo**.

---

## Objetivo general

Analizar y modelar el comportamiento productivo de los pozos no convencionales de YPF asociados a Vaca Muerta durante una ventana comparable de **24 meses calendario contados desde el primer mes con actividad efectiva (`tef > 0`)**, permitiendo eventuales interrupciones posteriores.

El problema se aborda mediante dos enfoques supervisados:

### Regresión

Predecir la producción acumulada de petróleo durante los primeros 24 meses calendario:

```text
pet_24m
```

### Clasificación

Clasificar cada pozo según si su producción acumulada a 24 meses se encuentra:

```text
0 = producción baja
1 = producción alta
```

El umbral de clasificación se calcula exclusivamente con la mediana de `pet_24m` del conjunto de entrenamiento, evitando utilizar información del conjunto de prueba.

---

## Preguntas de investigación

### Pregunta exploratoria

¿Cómo se comporta la producción de petróleo durante los primeros 24 meses calendario desde el inicio de actividad efectiva y qué variabilidad e interrupciones se observan entre pozos?

### Pregunta predictiva

¿En qué medida las características conocidas al inicio de la actividad permiten predecir la producción acumulada de petróleo durante los siguientes 24 meses calendario?

---

## Fuentes de datos

Se utilizan dos datasets públicos de la **Secretaría de Energía de la Nación Argentina**.

### 1. Capítulo IV — Pozos

Contiene atributos técnicos y descriptivos de cada pozo, entre ellos:

- área;
- cota;
- profundidad;
- clasificación;
- tipo de pozo;
- sistema de extracción;
- estado;
- fechas de perforación y terminación.

**Granularidad:** una fila por pozo.

Fuente:  
http://datos.energia.gob.ar/dataset/produccion-de-petroleo-y-gas-por-pozo/archivo/cb5c0f04-7835-45cd-b982-3e25ca7d7751

### 2. Producción de Pozos de Gas y Petróleo No Convencional

Contiene información mensual de:

- producción de petróleo;
- producción de gas;
- producción de agua;
- tiempo efectivo de producción (`tef`).

**Granularidad:** una fila por pozo y mes.

Fuente:  
http://datos.energia.gob.ar/dataset/produccion-de-petroleo-y-gas-por-pozo/archivo/b5b58cdc-9e07-41f9-b392-fb9ec68b0725

Ambos datasets se vinculan mediante:

```text
idpozo
```

---

## Definición de la cohorte

Uno de los puntos metodológicos centrales del proyecto es construir una ventana temporal comparable entre pozos.

Se define:

```text
Mes 1 = primer mes con tef > 0
```

A partir de ese momento, un pozo se conserva en la cohorte si:

- dispone de los **24 meses calendario completos** entre los meses 1 y 24;
- tiene `tef > 0` en el mes 1;
- puede presentar `tef = 0` entre los meses 2 y 24.

Por lo tanto, **no se exige actividad continua durante los 24 meses**.

Esta decisión permite conservar pozos que experimentaron interrupciones operativas y medir su producción acumulada durante una ventana calendario homogénea.

### Cohorte final

La cohorte utilizada contiene:

```text
1.431 pozos
```

Distribución de meses con producción positiva de petróleo:

| Situación | Pozos |
|---|---:|
| Producción positiva durante los 24 meses | 1.057 |
| Producción positiva entre 1 y 23 meses | 308 |
| Producción de petróleo igual a cero durante toda la ventana | 66 |
| **Total** | **1.431** |

Los pozos con producción de petróleo igual a cero se conservan porque representan resultados posibles dentro del universo analizado.

---

## Variable objetivo

La variable principal de regresión es:

```text
pet_24m
```

Se define como la suma de `prod_pet` durante los primeros **24 meses calendario** desde el inicio activo.

En la cohorte final:

| Estadístico | `pet_24m` |
|---|---:|
| Pozos | 1.431 |
| Media | 31.163,52 |
| Mediana | 23.450,43 |
| Mínimo | 0 |
| Máximo | 144.062,93 |

Los meses con interrupciones permanecen dentro de la ventana de análisis.

---

## Variables predictoras

El modelo utiliza únicamente información que puede considerarse disponible al inicio de la ventana productiva.

La versión base utiliza:

```python
features_base = [
    "area",
    "cota",
    "profundidad_valida",
    "duracion_perforacion_modelo",
    "anio_inicio"
]
```

`idpozo` se mantiene únicamente como identificador y **no se utiliza como predictor**.

No se incorporan como variables explicativas datos que solo pueden conocerse después del inicio del pozo, como producción futura, picos productivos, acumulados futuros de gas/agua o interrupciones observadas dentro de los 24 meses.

Esto permite reducir el riesgo de **data leakage**.

---

## Preprocesamiento

El preprocesamiento se realiza dentro de un `Pipeline` de Scikit-learn.

### Variables numéricas

- imputación de faltantes mediante la mediana;
- estandarización con `StandardScaler`.

### Variable categórica

- imputación con la categoría más frecuente;
- codificación mediante `OneHotEncoder`.

Los transformadores se ajustan exclusivamente con los datos de entrenamiento.

---

## Estrategia temporal de entrenamiento y evaluación

La separación entre entrenamiento y prueba **no es aleatoria**.

Los pozos se ordenan por su `fecha_inicio_activo` y se realiza una división temporal aproximada:

```text
datos históricos → entrenamiento
datos posteriores → prueba
```

Los pozos que comparten la misma fecha de inicio permanecen siempre del mismo lado del corte.

Esta estrategia busca aproximar un escenario real: entrenar con pozos históricos y evaluar la capacidad de generalizar hacia pozos iniciados posteriormente.

El conjunto de prueba se reserva para la evaluación final y no se utiliza para elegir modelos ni ajustar hiperparámetros.

---

## Validación cruzada temporal

La selección de modelos se realiza dentro del conjunto de entrenamiento mediante validación cruzada temporal con ventana expansiva.

Los folds se construyen sobre **fechas únicas de inicio de actividad**, evitando que dos pozos pertenecientes a una misma cohorte mensual queden separados entre entrenamiento y validación.

La lógica es:

```text
pasado → entrenamiento
futuro → validación
```

---

## Modelos de regresión

Se comparan:

- `DummyRegressor` como baseline;
- Regresión Lineal;
- Árbol de Decisión;
- Random Forest Regressor.

### Métricas

Se utilizan:

- **MAE** — Error Absoluto Medio;
- **RMSE** — Raíz del Error Cuadrático Medio;
- **R²** — Coeficiente de determinación.

La métrica principal para seleccionar el modelo es el **MAE de validación temporal**.

---

## Optimización de hiperparámetros

El Random Forest se optimiza mediante:

```text
RandomizedSearchCV
```

La búsqueda se realiza únicamente sobre el conjunto de entrenamiento utilizando los mismos folds temporales.

Se exploran hiperparámetros relacionados con la complejidad del bosque, entre ellos:

- número de árboles;
- profundidad máxima;
- mínimo de observaciones para dividir un nodo;
- mínimo de observaciones por hoja;
- cantidad de variables consideradas en cada división.

El objetivo no es maximizar el ajuste sobre entrenamiento, sino mejorar la **generalización hacia cohortes posteriores** y reducir el sobreajuste.

---

## Modelo de clasificación

El segundo problema supervisado transforma `pet_24m` en una variable binaria.

El umbral corresponde a:

```text
mediana de pet_24m calculada exclusivamente en train
```

Se comparan:

- `DummyClassifier`;
- Regresión Logística;
- Árbol de Decisión;
- Random Forest Classifier.

### Métricas

Debido a que la distribución de clases puede cambiar entre períodos, la evaluación no se basa únicamente en `accuracy`.

Se utilizan:

- Accuracy;
- Balanced Accuracy;
- Precision;
- Recall;
- F1-score;
- ROC-AUC;
- Matriz de confusión.

La métrica principal de selección es **Balanced Accuracy**.

---

## Flujo metodológico

```text
Datos públicos
      ↓
Exploración y calidad de datos
      ↓
Integración por idpozo
      ↓
Feature engineering
      ↓
Definición del inicio activo: tef > 0
      ↓
Cohorte con 24 meses calendario completos
      ↓
Construcción de pet_24m
      ↓
Selección de variables disponibles al inicio
      ↓
Split temporal train / test
      ↓
Pipeline de preprocesamiento
      ↓
Validación cruzada temporal
      ↓
Comparación de modelos
      ↓
Optimización de hiperparámetros
      ↓
Selección del modelo
      ↓
Evaluación final sobre test
```

---

## Tecnologías utilizadas

- Python
- Pandas
- NumPy
- Matplotlib
- Seaborn
- Scikit-learn
- Jupyter Notebook / Google Colab

---

## Estructura principal del proyecto

```text
.
├── Entrega_oct_Grupo_3_Data_Science_CORREGIDO_FINAL.ipynb
├── README.md
├── capitulo-iv-pozos.csv
└── produccin-de-pozos-de-gas-y-petrleo-no-convencional.csv
```

> Los archivos CSV provienen de las fuentes públicas indicadas anteriormente. Si no se incluyen en el repositorio, pueden descargarse nuevamente desde la Secretaría de Energía.

---

## Ejecución

### Google Colab

El notebook está preparado para leer los archivos desde:

```text
/content/drive/MyDrive/GRUPO3-DATA/
```

Archivos esperados:

```text
capitulo-iv-pozos.csv
produccin-de-pozos-de-gas-y-petrleo-no-convencional.csv
```

### Ejecución local

Colocar ambos CSV en el mismo directorio que el notebook y ejecutar las celdas en orden.

Dependencias principales:

```bash
pip install pandas numpy matplotlib seaborn scikit-learn
```

---

## Decisiones metodológicas importantes

- La unidad de análisis del modelo es **un pozo**.
- El inicio se define mediante el primer mes con `tef > 0`.
- Se utilizan 24 meses calendario completos, permitiendo interrupciones posteriores.
- Los valores de producción iguales a cero no se eliminan automáticamente.
- Los valores extremos se analizan de forma exploratoria antes de tomar decisiones de limpieza.
- Las variables predictoras se restringen a información disponible al inicio.
- La imputación y transformación de variables se realizan dentro del pipeline.
- El split entre entrenamiento y prueba respeta el tiempo.
- La validación cruzada también respeta las cohortes temporales.
- El test no participa en la selección de modelos ni hiperparámetros.

---

## Limitaciones

Entre las principales limitaciones del análisis se encuentran:

- cambios temporales en la distribución productiva;
- ausencia, dentro de las variables utilizadas, de una identificación explícita de orientación horizontal/vertical;
- ausencia de profundidad vertical verdadera como predictor explícito;
- no incorporación de una variable de densidad de pozos vecinos;
- disponibilidad limitada de algunas características exactamente al momento del inicio productivo.

Una futura extensión podría incorporar información espacial de pozos vecinos, siempre respetando el tiempo y contando únicamente los pozos que ya existían al momento de inicio del pozo objetivo.

---

## Interpretación de los resultados

Los modelos desarrollados permiten evaluar **capacidad predictiva y asociaciones** entre las características disponibles al inicio y la producción futura.

Los resultados **no deben interpretarse como evidencia causal**.

---

## Notebook principal

El desarrollo completo del proyecto, incluyendo análisis exploratorio, transformaciones, visualizaciones, modelos, validación temporal y optimización, se encuentra en:

```text
Entrega_oct_Grupo_3_Data_Science_CORREGIDO_FINAL.ipynb
```

---

## Equipo

**Grupo 3 — Data Science**

Proyecto desarrollado en el marco del **Programa Energía Digital**.
