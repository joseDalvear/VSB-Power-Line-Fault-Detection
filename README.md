# VSB-Power-Line-Fault-Detection
Detección de fallas en la línea de alimentación VSB - Analítica y Ciencia de Datos - Alfonso Cubillos y Jose Daniel Alvear

# Carpeta de apoyo con archivos del proyecto
Esta carpeta contiene los archivos que se encuentran en este repositorio y algunos adicionales para entender mas el contexto del proyecto; ademas se tienen algunas imagenes con el Pipeline del proyecto y como se deberia monitorear en caso de salir en productivo con este modelo
https://drive.google.com/drive/folders/1ouP_u7iDyexLqhTWF0FHqBQOxZid7GEB?usp=sharing

# Resumen del proyecto

El proyecto consiste en detectar patrones de descarga parcial en señales adquiridas de estas líneas eléctricas con un nuevo medidor diseñado por Centrum  ENET en VSB. El objetivo general es la detección temprana de las descargas para la reducción de los costos de mantenimiento de las mismas; además, eliminar los impactos ambientales nocivos generados por los incendios que se presentan por el mal estado en las líneas. 

Los datos que se tienen disponibles para la solución de este proyecto consisten en 8712 señales, cada una de las cuales cuenta con 800.000 mediciones de voltaje, tomadas durante 20 milisegundos; como la red eléctrica funciona a 50 HZ, esto significa que cada señal cubre un único ciclo de red completo. La red opera con un esquema de energía trifásica y las tres fases son medidas de forma simultánea.

Se realiza un análisis exploratorio de los datos, donde se toman las primeras 2000 mediciones con la idea de conocer el comportamiento de las mismas; Se grafican algunas señales y se observan algunos outliers.
Debido a que cada una de las señales tienen 800.000 mediciones fue necesario realizar técnicas de ingeniería de características para poder construir un entrenamiento adecuado, ya que a las máquinas a las que tenemos acceso, no cuentan con las capacidades para el procesamiento de tal volumen.

# DESCRIPCIÓN DEL PROBLEMA

Las líneas eléctricas aéreas de media tensión recorren cientos de millas para suministrar energía a las ciudades. Estas grandes distancias hacen que sea costoso inspeccionar manualmente las líneas en busca de daños que no provoquen inmediatamente un corte de energía, como una rama de un árbol golpeando la línea o una falla en el aislante. Estos modos de daño conducen a un fenómeno conocido como descarga parcial, una descarga eléctrica que no une completamente los electrodos entre un sistema de aislamiento. Las descargas parciales dañan lentamente la línea eléctrica, por lo que si no se reparan, eventualmente provocarán un corte de energía o provocarán un incendio.

Se pretende dar solución a la anterior problemática presentada por medio de modelos de clasificación, donde el modelo predice la probabilidad de la existencia de descarga parcial en alguna señal; esta será clasificada donde como resultado obtendremos un 0 cuando no existe falla alguna o un 1 en caso de presentar posible falla.

# APROXIMACIÓN DESDE LA ANALÍTICA DE DATOS

Para brindar una solución óptima, es necesario utilizar modelos de clasificación donde cada modelo sea capaz de predecir la probabilidad en la existencia de descarga parcial en alguna señal; luego se analiza automáticamente los resultados y por medio del uso de un valor umbral se obtiene como resultado una probabilidad final igual 0 si es el caso en que no se presentan descargas y de ser 1 cuando se presenten. 
Esta solución Analítica será capaz de clasificar cuando una probabilidad resultante de una señal es propensa a generar una descarga parcial, de allí toma la decisión de cómo será clasificada dicha señal y nos entrega como resultado la probabilidad de que suceda o no.
Los modelos utilizados son: LightGBM, XGBoost, CatBoost, Vecinos más cercanos K de Scikit-learn, Red neuronal de Scikit-learn, Regresión logística de Scikit-learn, Bayes ingenuo gaussiano de Scikit-learn; Luego de promediar resultados se toma la decisión.

# Metricas de desempeño
●	Los modelos utilizados son de clasificación, estos algoritmos predirán el tipo de datos a partir de ciertas matrices de datos. Por ejemplo,  puede responder con sí (1)/ no(0) . Es necesario utilizar la métrica de exhaustividad y productividad; las estimaciones devueltas para todas las clases están ordenadas por la etiqueta de clases. 
●	Las métrica de negocio se define con el coeficiente de “correlación de Matthews”, para cada señal en el conjunto de prueba, debe predecir una predicción binaria para la variable 

# --> Datos
●	El modo de acceso a los datos fue a través de las competencias de Kaggle, donde se invitaba a la comunidad a participar en el reto de detección de fallas en las líneas de alimentación VSB; Los archivos para el entrenamiento y test son bastante grandes, en total suman 11.75GB todos los datos disponibles en el reto. También es necesario aclarar que los archivos de entrenamiento y Test  utilizan un tipo de archivo (.parquet). 

Los datos pueden ser utilizados por cualquier persona que desee descargarlos de Kaggle, en estos archivos no se tienen datos personales ni sensibles; además se realizó un análisis de sesgo a los datos y efectivamente no contamos con datos sesgados

no se evidencian datos que podrian presentar sesgos, ya que no se tienen datos de personas, ni localidades como para evidenciar posibles casos de discriminación o algunos de los que se presentan comúnmente cuando se trabaja en proyectos de analítica. En la monografía se trabajan con las señales de las corrientes eléctricas por lo que no se tienen datos de ubicaciones específicas que puedan crear algún tipo de sesgo; solo se tiene el número del trayecto del cable asignado (Cantidad 8712) a los cuales se les tomó 800.000 mediciones que fueron tomadas por un grupo de sensores.

# Ingeniería de caracteristicas
Este es uno de los hitos superados más importantes y donde se tomó más tiempo al momento de desarrollar y ejecutar el código; esto debido a que cada una de las señales tiene 800.000 lecturas y recordando que son 8712 para el entrenamiento de los modelos, es esencial que extraigamos características de esas lecturas para construir un entrenamiento adecuado y conjuntos de datos de prueba eficientes al momento de procesarlos. 

# Filtrado de las señales
Todas las operaciones que se describen a continuación se realizan después de filtrar las señales, ya que contienen mucho ruido como se describe anteriormente. Para hacerlo, aplicamos la eliminación de ruido de ondas (Scikit-image's wavelet denoising), donde el ruido gaussiano tiende a estar representado por valores pequeños en el dominio y puede eliminarse estableciendo coeficientes por debajo de un umbral determinado en cero (umbral estricto) o reduciendo todos los coeficientes hacia cero en una cantidad determinada (umbral suave).

# Calculo de picos de las señales

Para el cálculo de los picos en las señales se utilizó (Scipy.signal.find_peaks) función de Scipy para extraer características útiles de las señales. Esta función toma una matriz unidimensional (1-D) y encuentra todos los máximos locales mediante una simple comparación de valores vecinos. También permite la selección de un subconjunto de estos picos especificando las condiciones para las propiedades de un pico.

Se utiliza esta función con diferentes combinaciones de parámetros. Por ejemplo, en la primera ocasión se usa para encontrar picos en la señal cuya altura es superior a 5, en la segunda ocasión fue para cuando la altura sea superior a 15, En la tercera para cuando la altura del pico es mayor de 15 y la distancia horizontal entre picos vecinos es de más de 25 muestras, en la cuarta ocasión para cuando la altura del pico es superior a 25 y el ancho del pico es superior a 10 muestras; etc.
También es necesario usar (Scipy's scipy.signal.peak_prominences) donde una matriz que coincida con Xo una secuencia de 2 elementos de la primera; el primer elemento siempre se interpreta como el mínimo y el segundo, si se suministra, como la prominencia máxima requerida. Además, se utiliza también (scipy.signal.peak_widths) para obtener la prominencia y el ancho de máximos y mínimos de los picos. Esto nos genera 146 características.

# Información estadistica de la señal
En la construcción de características se calculan algunas estadísticas de las señales por medio de Numpy como la media, la desviación estándar, el mínimo, el máximo, la asimetría, el percentil 30, el percentil 70, etc.

# Selección de características
Se utiliza la función de LightGBM (feature_importance) para obtener las características más importantes, así que, en lugar de alrededor de 300 características, se seleccionan las 68 características más importantes y luego agregamos características que representan la sustracción de algunas características de otras. Esto se debe a que calculamos la información de los picos positivos y negativos, por lo que puede ser útil restar las características de los picos negativos de las características de los picos positivos.

# Modelos utilizados:

●	CatBoostClassifier
●	xgboost
●	KNeighborsClassifier
●	BaggingClassifier
●	MLPClassifier
●	LogisticRegression
●	GaussianNB

La idea de unificar los resultados de varios modelos surge luego de que al aplicar todo el preprocesamiento de los datos, estos quedan mucho más manejables, lo que nos abrió las posibilidades de poder ensayar varios modelos y finalmente poder mejorar nuestra capacidad de clasificar una señal; utilizando tanto modelos sencillos y clásicos hasta algo más profundo como lo es una Red neuronal.

# Instrucciones para la ejecución del Notebook:

Se recomienda leer el instructivo de ejecución https://drive.google.com/file/d/1LZkf5kQ0h2Fq988a9DaHoToRXPvCYyNO/view?usp=sharing
Es necesario ejecutar el notebook en la plataforma de Kaggle ya que nos proporciona 16 GB de memoria RAM y una buena velocidad de procesamiento. Adicionalmente al final de estas instrucciones se deja el link personal del proyecto en Kaggle por si deseas visitarlo.

•	El notebook cuenta con una fase inicial de Análisis exploratorio de los datos llamada “Iteración 1”. Con esta parte entenderas como están distribuidos los datos, un poco de su comportamiento y también el tamaño de los datos a los que nos vamos a enfrentar.

•	Iteración 2
Primera parte: En esta parte se trabaja con las señales, se calculan los picos y se hace todo un pre procesamiento de la señal; además se generan 2 archivos que son el insumo para la tercera parte de la iteración 2
Segunda parte: Se realiza un proceso muy similar a la anterior; todo un pre procesamiento de las señales y se generan 2 archivos que sirven de insumo para la tercera parte de la iteración 2 
Tercera parte: Se realiza un proceso de reducción de características para luego pasar como insumo los datos a los modelos. Luego de hacer uso de varios modelos, se promedian los resultados para obtener una clasificación más segura y siempre en búsqueda de mejorar la predicción.

# Pipeline
Se dejan una explicación grafica en la carpeta compartida
https://drive.google.com/file/d/1LZkf5kQ0h2Fq988a9DaHoToRXPvCYyNO/view?usp=sharing


# Visitar la publicación del proyecto en Kaggle
Si quieres visitar el proyecto alojado en Kaggle directamente esta es la dirección
https://www.kaggle.com/josedanielalvear/proyecto-analitica



