Análisis exploratorio de pozos no convencionales de YPF en Vaca Muerta
Grupo 3 --- 
Conformado por: Hugo Espinosa, Martina Sarmiento y Paula Perez Gianolini. 
Primera entrega: exploración, transformación y
visualización de datos con Pandas
Descripción del proyecto
Este proyecto estudia pozos no convencionales asociados a la formación
Vaca Muerta operados por YPF, integrando información técnica de los
pozos con sus registros históricos mensuales de producción.
La primera entrega se concentra en el Análisis Exploratorio de Datos
(AED): comprensión y calidad de los datos, transformaciones,
visualizaciones y análisis descriptivos. El trabajo deja planteada una
base para una etapa posterior de Machine Learning, pero no desarrolla
modelos predictivos en esta entrega.
Objetivo general
Analizar el comportamiento productivo de los pozos no convencionales de
YPF asociados a la formación Vaca Muerta durante sus primeros dos años
de operación, integrando la información técnica de los pozos con sus
registros históricos mensuales de producción.
A través del AED se busca comprender la estructura y calidad de los
datasets, realizar las transformaciones necesarias y caracterizar cuánto
produce un pozo típico en ese período y qué tan distinto es ese volumen
entre pozos.
Pregunta de investigación
¿Cuánto petróleo produce un pozo no convencional de YPF en Vaca Muerta
durante sus primeros 24 meses de operación, cómo se distribuye esa
producción a lo largo de esos meses y qué tan distinta es entre pozos?
Como preguntas complementarias, el notebook analiza:
en qué parte de esos 24 meses se concentra la producción;
diferencias de producción según área y tipo de pozo;
posibles caídas de producción coincidentes con la perforación de un
pozo vecino cercano, como análisis exploratorio de potencial
interferencia entre pozos.
El análisis de interferencia es asociativo y exploratorio y no
pretende establecer causalidad.
Fuentes de datos
Se utilizan dos fuentes publicadas por la Secretaría de Energía de la
Nación:
Capítulo IV --- Listado de pozos: contiene información
descriptiva y técnica de los pozos, como área, yacimiento,
profundidad, tipo de pozo, sistema de extracción, estado y fechas de
perforación y terminación. Su granularidad es una fila por pozo.
Producción de pozos de gas y petróleo no convencional: contiene
registros mensuales de producción de petróleo, gas y agua, junto con
variables operativas. Su granularidad es una fila por pozo y mes.
Ambos datasets comparten el identificador `idpozo`, utilizado para
relacionar la información técnica con la productiva.
Definición del universo de análisis
En el dataset técnico se seleccionan los pozos que cumplen
simultáneamente los siguientes criterios:
empresa cuya razón social contiene `YPF`;
recurso `NO CONVENCIONAL`;
formación `vaca muerta`.
YSUR queda excluida deliberadamente porque aparece identificada por
separado y su denominación no contiene la cadena `YPF`.
Luego del filtrado, el dataset técnico contiene 1.818 pozos únicos.
En el dataset productivo, el filtro inicial de YPF y Vaca Muerta
contiene 142.979 registros mensuales correspondientes a 1.890 pozos.
Al comparar ambas fuentes se obtienen:
1.818 pozos presentes en ambos datasets.
Para mantener consistencia entre las fuentes, el análisis productivo se
restringe a los pozos presentes en ambos datasets. El universo
resultante contiene 141.920 registros mensuales y 1.818 pozos
únicos.
No se detectaron filas duplicadas ni registros duplicados para una misma
combinación de `idpozo`, año y mes.
Preparación y transformación de los datos
Entre las tareas realizadas se encuentran:
revisión de valores faltantes y columnas sin capacidad informativa;
conversión de fechas de perforación y terminación a `datetime`;
cálculo de duraciones de perforación y terminación;
tratamiento auxiliar de profundidades iguales a cero sin eliminar
los registros originales;
construcción de variables temporales para ordenar la historia
mensual de cada pozo;
separación de meses con actividad mediante el criterio `tef > 0`;
cálculo de caudales normalizados a partir de la producción y el
tiempo efectivo informado;
transformación `log1p` para explorar variables productivas con
fuerte asimetría; integración de información técnica y productiva mediante `idpozo`.
El período presente en el dataset productivo analizado se extiende desde

Análisis exploratorio
El notebook analiza características técnicas y operativas como:
profundidad;
duración de perforación y terminación;
tipo de pozo;
sistema de extracción;
clasificación y subclasificación;
área.
También se estudian las distribuciones de producción de petróleo, gas y
agua. Las variables productivas presentan una marcada asimetría positiva
y una proporción relevante de registros iguales a cero.
Los valores potencialmente atípicos se estudian mediante boxplots, rango
intercuartílico y revisión de caudales extremos. No se eliminan
observaciones automáticamente por ser identificadas como atípicas, ya
que pueden corresponder a comportamientos reales del proceso productivo.
Además, se comprobó que `tipopozo` y `tipoextraccion` pueden cambiar a
lo largo de la historia de un mismo pozo. Por ese motivo, las
comparaciones que utilizan estas variables dentro del dataset productivo
se interpretan a nivel de registro mensual y no como características
necesariamente fijas durante toda la vida del pozo.
Comparaciones entre grupos
Como contexto para la pregunta principal se comparan los niveles de
producción según:
tipo de pozo;
sistema de extracción;
área.
Para el análisis por área se incorpora la información técnica mediante
un `merge` por `idpozo`. La comparación gráfica se concentra en siete
áreas seleccionadas por su cantidad de pozos, que reúnen 1.655 pozos
y representan 94,92 % del universo activo utilizado en esa
comparación.
Las diferencias encontradas son descriptivas y no se interpretan como
relaciones causales.
Análisis de los primeros 24 meses
Según el criterio temporal implementado actualmente en el notebook:
1.805 pozos cuentan con registros de producción en el universo
activo;
1.404 pozos tienen 24 meses o más de historia según la variable
temporal construida;
401 pozos quedan fuera por historia insuficiente;
el cálculo final de producción acumulada de petróleo de los primeros
24 meses incluye 1.313 pozos.
Para esos 1.313 pozos, la producción acumulada de petróleo calculada en
el notebook presenta:
media: 33.311;
mediana: 29.639;
primer cuartil: 4.810;
tercer cuartil: 56.772;
máximo: 144.063.
La dispersión observada muestra que el volumen acumulado durante la
ventana analizada varía considerablemente entre pozos.
El notebook también calcula qué proporción del acumulado de 24 meses
corresponde a los primeros seis meses. La mediana obtenida es 38,7%.
