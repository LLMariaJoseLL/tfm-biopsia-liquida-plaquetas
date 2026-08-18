===============================================================================
Optimización de la biopsia líquida mediante Machine Learning: 
Detección y diagnóstico del origen tumoral a partir del transcriptoma plaquetario

Trabajo Fin de Máster - Máster Universitario en Big Data y Ciencia de Datos
Universidad Internacional de Valencia (VIU)

Autora: María José Lorido Lorido
===============================================================================


1. DESCRIPCIÓN DEL PROYECTO
-------------------------------------------------------------------------------
Este repositorio contiene el cuaderno de análisis del Trabajo Fin de Máster,
que estudia la capacidad de los perfiles de ARN de plaquetas educadas por
tumor para detectar la presencia de cáncer y para distinguir el tipo tumoral
de origen.

Se llevan a cabo dos tareas de clasificación:

  - Tarea binaria: distinguir individuos sanos de pacientes con cáncer.
  - Tarea multiclase: distinguir entre seis tipos de tumores (mama,
    colorrectal, glioblastoma, hepatobiliar, pulmón y páncreas).

Para cada tarea se comparan cinco algoritmos (regresión logística, k vecinos
más próximos, máquina de vectores soporte, bosque aleatorio y XGBoost)
combinados con tres estrategias frente al desbalance de clases (ninguna,
ponderación de clases y remuestreo SMOTE).

El trabajo sigue la metodología CRISP-DM y se organiza en cinco fases:
comprensión de los datos, preparación, modelado, evaluación e identificación
de los genes característicos.


2. DATOS
-------------------------------------------------------------------------------
Los datos proceden del repositorio público GEO, conjunto GSE68086, 
publicado por Best et al. (2015).

  Enlace: https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE68086

Se necesitan dos archivos:

  a) Matriz de expresión (conteos por gen y muestra)
     GSE68086_TEP_data_matrix.txt
     Dimensiones: 57736 genes x 285 muestras

  b) Metadatos de las muestras (series matrix)
     GSE68086_series_matrix.txt
     Contiene, para cada muestra, su identificador GSM, título, tipo de
     cáncer, lote de procesamiento, subclase mutacional e identificador de
     paciente.

DESCARGA
La descarga se realiza de forma manual desde el enlace anterior, ya que la
descarga automatizada desde el cuaderno puede bloquearse por las
restricciones del servidor de GEO. Una vez descargados, ambos archivos deben
descomprimirse y situarse en la carpeta de Google Drive indicada en una de las 
primeras celdas de código del cuaderno.

NOTA SOBRE EL TAMAÑO DEL CONJUNTO
El análisis trabaja finalmente con 284 muestras, no con las 285 originales.
La muestra Lung-0020J-1 se elimina por tratarse de una posible réplica
técnica de Lung-0020J: comparte tejido, tipo celular, tipo tumoral, subclase
mutacional y lote, y presenta una profundidad de secuenciación muy inferior
(0,59 millones de lecturas frente a 5,05 millones). Se conserva la de mayor
calidad. Esta decisión evita que dos secuenciaciones del mismo individuo
puedan repartirse entre entrenamiento y validación.


3. ENTORNO DE EJECUCIÓN
-------------------------------------------------------------------------------
El cuaderno está preparado para ejecutarse en Google Colab, con acceso a
Google Drive para la lectura de los archivos de datos.

Requisitos:
  - Cuenta de Google con acceso a Google Colab.
  - Los dos archivos de datos alojados en Google Drive.

También puede ejecutarse en un entorno local con Jupyter, sustituyendo la
celda de montaje de Google Drive por la ruta local de los archivos.

BIBLIOTECAS UTILIZADAS
  numpy
  pandas
  matplotlib
  seaborn
  scipy
  scikit-learn
  imbalanced-learn
  xgboost

Todas ellas están disponibles en el entorno por defecto de Google Colab,
salvo imbalanced-learn y xgboost, que el cuaderno instala si no las
encuentra.

VERSIONES EXACTAS
  python            3.12.13
  numpy             2.0.2
  pandas            2.2.2
  matplotlib        3.10.0
  seaborn           0.13.2
  scipy             1.16.3
  sklearn           1.6.1
  imblearn          0.14.2
  xgboost           3.3.0


4. INSTRUCCIONES DE EJECUCIÓN
-------------------------------------------------------------------------------
  1. Descargar los dos archivos de datos desde GEO y descomprimirlos.
  2. Subirlos a Google Drive, en la carpeta indicada en una de las primeras 
     celdas de código del cuaderno. Si se usa otra ruta, modificar las variables
     archivo_expresion y archivo_metadatos.
  3. Abrir el cuaderno en Google Colab.
  4. Reiniciar el entorno de ejecución (Entorno de ejecución > Reiniciar
     entorno de ejecución) para garantizar que no queden variables de
     ejecuciones anteriores.
  5. Ejecutar todas las celdas en orden, de principio a fin (Entorno de
     ejecución > Ejecutar todas).

El cuaderno debe ejecutarse siempre de forma secuencial. Varias celdas
dependen de objetos creados en celdas anteriores, de modo que una ejecución
parcial o desordenada producirá errores.

TIEMPO APROXIMADO DE EJECUCIÓN
La ejecución completa necesita un tiempo considerable. Los apartados más
costosos son el ajuste de hiperparámetros, especialmente las configuraciones
de XGBoost y del bosque aleatorio, y los análisis de estabilidad, que
repiten el entrenamiento sobre veinte particiones distintas.
El tiempo aproximado de ejecución es: 41 minutos y 2 segundos


5. SALIDAS GENERADAS
-------------------------------------------------------------------------------
  anexo_correspondencia_muestras.csv
     Tabla de correspondencia entre cada columna de la matriz de expresión y
     su registro en GEO. Recoge la posición, el nombre de la columna, el
     identificador GSM, el título del repositorio, el identificador de
     paciente, la clase, el lote, la subclase mutacional y el criterio
     utilizado para realizar el emparejamiento. Se incluye en el repositorio 
     para garantizar la trazabilidad de las etiquetas.

El resto de resultados (tablas, métricas y gráficas) se muestran en la propia
salida del cuaderno.

6. RESULTADOS PRINCIPALES
-------------------------------------------------------------------------------
TAREA BINARIA (detección de cáncer)
  Modelo: regresión logística sin corrección del desbalance, C = 0,1, umbral 0,5
  Exactitud balanceada  0,946    IC 95% [0,898 - 0,988]
  AUC                   0,984    IC 95% [0,951 - 1,000]
  Exactitud simple      0,912
  Sin falsos positivos sobre los 11 controles sanos del conjunto de prueba.

TAREA MULTICLASE (seis tipos tumorales)
  Modelo: regresión logística con SMOTE, C = 0,1
  Exactitud balanceada  0,599    IC 95% [0,441 - 0,764]
  AUC                   0,822    IC 95% [0,736 - 0,909]
  Exactitud simple      0,652

Los intervalos proceden de un remuestreo bootstrap con 2000 repeticiones. La
amplitud de los intervalos de la tarea multiclase refleja el reducido tamaño
del conjunto de prueba y debe tenerse en cuenta al interpretar las cifras.


7. REPRODUCIBILIDAD
-------------------------------------------------------------------------------
SEMILLA
Todas las operaciones con componente aleatorio utilizan la semilla 42, fijada
en la variable "semilla" al comienzo del cuaderno. Afecta a la partición
entre entrenamiento y prueba, a la validación cruzada, al remuestreo SMOTE,
a los modelos que lo admiten y al remuestreo bootstrap de los intervalos de
confianza.

VARIACIONES POSIBLES
Los resultados numéricos pueden cambiar de forma mínima entre ejecuciones
realizadas con versiones distintas de las bibliotecas, especialmente en el
remuestreo SMOTE y en los algoritmos basados en árboles. Las conclusiones
metodológicas no dependen de esas diferencias, que se sitúan muy por debajo
de la variabilidad observada entre particiones.

SEPARACIÓN ENTRE ENTRENAMIENTO Y PRUEBA
El conjunto de prueba se reserva al comienzo y no interviene en la selección 
de modelos ni en el ajuste de hiperparámetros, decisiones que se toman 
exclusivamente con validación cruzada sobre el entrenamiento. Los análisis 
planteados después de observar su comportamiento se identifican de forma 
explícita como exploratorios.

RESULTADOS CONFIRMATORIOS Y EXPLORATORIOS
El cuaderno distingue entre ambos tipos de resultado. Se
considera confirmatorio el obtenido con las decisiones fijadas antes de
conocer el conjunto de prueba, e incluye el modelo binario evaluado con el
umbral por defecto de 0,5. Se presentan como exploratorios o post hoc los
análisis planteados después de observar el comportamiento del modelo, como la
revisión del umbral mediante el índice de Youden o la revisión de la
selección de genes en la tarea multiclase.


8. ESTRUCTURA DEL CUADERNO
-------------------------------------------------------------------------------
FASE 1. COMPRENSIÓN DE LOS DATOS
  Carga de la matriz de expresión y de los metadatos. Reconstrucción de la
  correspondencia entre ambos archivos y asignación de etiquetas. Análisis
  exploratorio inicial. Detección y tratamiento de la posible réplica
  técnica.

FASE 2. PREPARACIÓN DE LOS DATOS
  Diagnóstico del filtrado de genes poco expresados. Transposición y
  separación de variables y objetivo. Partición entre entrenamiento y
  prueba. Normalización y estandarización. Control de calidad mediante
  análisis de componentes principales, con estudio del efecto de lote y de
  las muestras atípicas. Preparación de las estrategias frente al desbalance.

FASE 3. MODELADO
  Selección del número de genes. Comparación de la rejilla de combinaciones
  de algoritmo y estrategia. Ajuste de hiperparámetros mediante búsqueda en
  rejilla con validación cruzada. Selección de los modelos definitivos.

FASE 4. EVALUACIÓN
  Entrenamiento y predicción sobre el conjunto de prueba. Análisis de las
  matrices de confusión. Revisión del umbral de corte en la tarea binaria.
  Revisión de la selección de genes en la tarea multiclase. Estabilidad de
  los resultados frente al reparto, sobre veinte particiones. Intervalos de
  confianza estimados por bootstrap. Contraste con los estudios de
  referencia.

FASE 5. IDENTIFICACIÓN DE LOS GENES CARACTERÍSTICOS
  Extracción de la firma génica de cada tarea. Estabilidad de las firmas a lo
  largo de veinte particiones. Interpretación biológica de los genes
  reproducibles. Contraste entre los genes de la firma y los asociados a la
  variación de origen técnico.


9. PRINCIPALES LIMITACIONES
-------------------------------------------------------------------------------
  - El conjunto de datos es reducido, con 284 muestras repartidas entre siete
    categorías, y el grupo hepatobiliar cuenta con solo catorce muestras. Las
    métricas de las clases minoritarias no son fiables.

  - Existe un efecto de lote asociado al centro de procedencia. Las tandas
    Batch05 y Batch06 corresponden a la cohorte del Massachusetts General
    Hospital, que no incluye controles sanos, glioblastomas ni casos de
    páncreas. El lote está por tanto parcialmente confundido con la clase, lo
    que impide aplicar los métodos habituales de corrección y también una
    evaluación separada por centro.

  - Las firmas génicas resultan inestables entre particiones. Solo unos pocos
    genes se seleccionan de forma reproducible, de modo que las listas
    obtenidas deben tratarse como hipótesis exploratorias y no como conjuntos
    de biomarcadores validados.

  - Los coeficientes de la regresión logística describen asociaciones útiles
    para clasificar, pero no constituyen una prueba de expresión diferencial
    ni permiten atribuir una función biológica determinada.

  - El análisis no estudia el rendimiento por estadio tumoral, y los
    controles son individuos sanos, sin incluir pacientes con patologías
    inflamatorias, cardiovasculares o tumores benignos que puedan alterar el
    ARN plaquetario. Los resultados no son extrapolables a un uso clínico
    real.


10. REFERENCIAS PRINCIPALES
-------------------------------------------------------------------------------
Abeel, T., Helleputte, T., Van de Peer, Y., Dupont, P., & Saeys, Y. (2010). 
Robust biomarker identification for cancer diagnosis with ensemble 
feature selection methods. Bioinformatics, 26(3), 392–398. 
https://doi.org/10.1093/bioinformatics/btp630

Best, M. G., Sol, N., Kooi, I., Tannous, J., Westerman, B. A., Rustenburg, F., 
Schellen, P., Verschueren, H., Post, E., Koster, J., Ylstra, B., Ameziane, N., 
Dorsman, J., Smit, E. F., Verheul, H. M., Noske, D. P., Reijneveld, J. C., Nilsson, 
R. J. A., Tannous, B. A., … Wurdinger, T. (2015). RNA-Seq of tumor-educated 
platelets enables blood-based pan-cancer, multiclass, and molecular 
pathway cancer diagnostics. Cancer Cell, 28(5), 666–676. 
https://doi.org/10.1016/j.ccell.2015.09.018

Huang, Y.-K., Fan, X.-G., & Qiu, F. (2016). TM4SF1 promotes proliferation, 
invasion, and metastasis in human liver cancer cells. International Journal of 
Molecular Sciences, 17(5), 661. https://doi.org/10.3390/ijms17050661

Jopek, M. A., Pastuszak, K., Sieczczyński, M., Cygert, S., Żaczek, A. J., Rondina, 
M. T., & Supernat, A. (2024). Improving platelet-RNA-based diagnostics: A comparative 
analysis of machine learning models for cancer detection and multiclass 	
classification. Molecular Oncology, 18(11), 2743–2754. 	
https://doi.org/10.1002/1878-0261.13689

Stelzer, G., Rosen, N., Plaschkes, I., Zimmerman, S., Twik, M., Fishilevich, S., Stein, 
T. I., Nudel, R., Lieder, I., Mazor, Y., Kaplan, S., Dahary, D., Warshawsky, D., 
Guan-Golan, Y., Kohn, A., Rappaport, N., Safran, M., & Lancet, D. (2016). 
The GeneCards suite: From gene data mining to disease genome sequence analyses. 
Current Protocols in Bioinformatics, 54(1), 1.30.1–1.30.33. https://doi.org/10.1002/cpbi.5

Tang, Q., Chen, J., Di, Z., Yuan, W., Zhou, Z., Liu, Z., Han, S., Liu, Y., Ying, G., 
Shu, X., & Di, M. (2020). TM4SF1 promotes EMT and cancer stemness via the 
Wnt/beta-catenin/SOX2 pathway in colorectal cancer. Journal of Experimental & 
Clinical Cancer Research, 39(1), 232. https://doi.org/10.1186/s13046-020-01690-z


11. LICENCIA Y USO
-------------------------------------------------------------------------------
Los datos originales pertenecen al repositorio público GEO y están sujetos a
las condiciones de uso de dicho repositorio y del estudio original de Best et
al. (2015).

El código de este repositorio se publica con fines académicos, como material
de apoyo al Trabajo Fin de Máster. No constituye una herramienta de
diagnóstico ni debe utilizarse con finalidad clínica.

===============================================================================
