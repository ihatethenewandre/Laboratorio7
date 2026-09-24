# Laboratorio 7 – Perfiles y salarios de la población asalariada de la ENEIC con Spark

Análisis de los ingresos laborales de la población asalariada de Guatemala a partir de las bases de Personas de la Encuesta Nacional de Empleo e Ingresos Continua, utilizando Spark 3.5 y la API de DataFrames de pyspark.ml para la exploración, la segmentación de perfiles y la predicción del salario mensual.

## Integrantes

- Estuardo André Castro Bonifaz – 23890
- Juan Marcos Cruz Melara – 23110
- André Emilio Pivaral López – 23574

**Universidad del Valle de Guatemala**  
Facultad de Ingeniería  
Departamento de Computación  
Data Science

**Catedrático:** Boris Fernando Becerra Peláez  
**Sección:** 30

## Descripción

El proyecto construye una base analítica a partir de cinco cortes de la ENEIC, los cuatro trimestres de 2025 para desarrollo y entrenamiento y el primer trimestre de 2026 reservado para la evaluación final. Los archivos originales vienen en hoja de cálculo, por lo que se leen con pandas y openpyxl, se convierten a DataFrames de Spark con tipos explícitos y se almacenan en Parquet. A partir de ese punto toda la preparación analítica, el cálculo de métricas y el aprendizaje automático se realizan con Spark y MLlib, sin utilizar scikit-learn. A pandas solo se transfieren tablas agregadas y muestras de a lo sumo diez mil registros para graficar.

Esta entrega corresponde al avance del laboratorio y cubre los ejercicios 1 al 4. El Ejercicio 1 carga y armoniza los cinco archivos, identifica la procedencia de cada registro, documenta los valores faltantes, aplica una cadena de filtros en orden fijo con el conteo de exclusiones por paso, verifica la unicidad de la clave de registro sin ocultar duplicados y guarda los conjuntos preparados en Parquet. El Ejercicio 2 describe la población analítica mediante estadísticos de posición y dispersión y responde con evidencia gráfica cómo se distribuyen los registros y el salario. El Ejercicio 3 estima la correlación de Pearson entre las variables numéricas con Correlation.corr de pyspark.ml.stat. El Ejercicio 4 segmenta perfiles con KMeans, compara una variante con salario y otra sin salario, evalúa valores de K entre 2 y 5 con el criterio de codo y el coeficiente de silueta, y describe los grupos resultantes.

No existe informe en PDF para esta entrega. El análisis completo, con los resultados y sus interpretaciones, se encuentra dentro del notebook.

## Estructura del proyecto

    lab7/
    ├── .venv/                              entorno virtual de Python
    ├── data/
    │   ├── raw/                            archivos originales de la ENEIC, sin modificar
    │   └── processed/                      parquet por archivo y conjuntos analíticos de 2025 y 2026
    ├── figures/                            figuras exportadas e indice_figuras.csv
    ├── notebook/
    │   └── Laboratorio7_avance.ipynb       cuaderno entregable de los ejercicios 1 al 4
    ├── .gitignore
    ├── README.md
    └── requirements.txt                    dependencias del proyecto

Los archivos originales no se versionan por su tamaño. Deben colocarse en data/raw con estos nombres exactos, o bien ajustarse el diccionario ARCHIVOS_ENEIC del notebook si los nombres difieren:

    ENEIC 2025 I Personas.xlsx
    ENEIC 2025 II Personas.xlsx
    ENEIC 2025 III Personas.xlsx
    ENEIC 2025 IV Personas.xlsx
    ENEIC 2026 I Personas.xlsx

## Requisitos

Python 3.11 y Java 17 o posterior, que es la máquina virtual sobre la que corre Spark. Los paquetes necesarios son pyspark 3.5.1, pandas, numpy, openpyxl, pyarrow, matplotlib, seaborn y jupyterlab. La lectura de las hojas de cálculo requiere openpyxl, y la escritura de Parquet requiere pyarrow.

La conversión con Arrow se mantiene desactivada en la sesión de Spark del cuaderno. A partir de Java 17 Arrow necesita permisos adicionales de la máquina virtual, y en este proyecto solo viajan a pandas tablas agregadas y muestras acotadas, por lo que desactivarla evita un fallo de memoria sin costo apreciable de rendimiento.

Instalación en Windows con PowerShell:

    cd "ruta del proyecto"
    python -m venv .venv
    .\.venv\Scripts\Activate.ps1
    python -m pip install --upgrade pip
    pip install -r requirements.txt

Si PowerShell bloquea la activación del entorno, ejecutar antes:

    Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass

Instalación en Linux o macOS:

    cd "ruta del proyecto"
    python3 -m venv .venv
    source .venv/bin/activate
    python -m pip install --upgrade pip
    pip install -r requirements.txt

## Ejecución

Colocar los cinco archivos originales en data/raw, abrir JupyterLab y ejecutar el cuaderno de principio a fin:

    jupyter lab notebook/Laboratorio7_avance.ipynb

El cuaderno no depende de variables creadas en ejecuciones anteriores. La primera corrida convierte cada archivo original a Parquet y las corridas siguientes reutilizan esos Parquet, por lo que son considerablemente más rápidas. Para forzar la reconversión basta con llamar convertir_a_parquet con el argumento forzar en verdadero, o borrar el contenido de data/processed.

## Contenido del análisis

- Verificación de entorno con la versión de Python, el intérprete en uso, la versión de Java, la disponibilidad de los paquetes requeridos y la existencia de los cinco archivos originales.
- Conversión individual de cada archivo a Parquet con un esquema explícito, normalizando los códigos que pueden llegar como número en un archivo y como texto en otro.
- Identificación del corte publicado mediante periodo_archivo, anio_archivo, trimestre_calendario y archivo_origen, construidos a partir del archivo de procedencia. La columna original TRIMESTRE se conserva intacta y se contrasta con esa identificación.
- Unión de los cuatro archivos de 2025 con unionByName, necesaria porque el corte IV de 2025 se publica con más columnas que los demás y la posición de columna no identifica la misma variable en todos los archivos.
- Tabla de cantidad y porcentaje de valores faltantes por variable seleccionada, medida antes de aplicar cualquier filtro.
- Cadena de ocho filtros aplicada siempre en el mismo orden, con el número de registros excluidos en cada paso: edad, condición de ocupación, categoría ocupacional asalariada, salario positivo, antigüedad en años, componente de meses, consistencia entre antigüedad y edad y horas habituales.
- Verificación de unicidad de la combinación periodo_archivo, NUM_HOGAR y NUM_PERSONA, distinguiendo repeticiones exactas de registros en conflicto, sin utilizar dropDuplicates. Se documenta además cuántas personas aparecen en más de un período del panel longitudinal.
- Estadística descriptiva del salario mensual, la edad, la antigüedad y las horas habituales, con cantidad de observaciones, media, mediana, desviación estándar, mínimo, máximo y percentiles 25, 75 y 95 calculados sobre el conjunto completo.
- Distribución de los registros entre categorías ocupacionales, niveles educativos y dominios, y distribución del salario en escala original y logarítmica con su asimetría y su curtosis.
- Salario mediano por nivel educativo, categoría ocupacional y dominio, acompañado del número de observaciones de cada grupo, y evolución del tamaño de la muestra analítica y del salario mediano entre trimestres.
- Matriz de correlación de Pearson entre salario, edad, antigüedad y horas habituales, obtenida con VectorAssembler y Correlation.corr sobre todos los registros elegibles, presentada con etiquetas y con un mapa de calor.
- Segmentación con KMeans sobre variables estandarizadas con StandardScaler, comparando una variante sin salario y otra con salario para K igual a 2, 3, 4 y 5. La selección se realiza por el mayor coeficiente de silueta, contrastado con el criterio de codo, y cada clúster se describe con sus promedios en unidades originales, su composición categórica dominante y su salario mediano.

## Nota sobre los datos

Los resultados describen a los registros analizados y no constituyen estimaciones oficiales de la población guatemalteca. El clustering, los modelos y las métricas principales son no ponderados, de acuerdo con el alcance definido en el enunciado. La columna FACTOR se conserva para documentar el diseño muestral y se utilizaría para expandir a totales poblacionales y calcular promedios ponderados en un análisis representativo, pero no interviene en ninguna de las métricas reportadas.

La población analítica se restringe a personas de 15 años o más, ocupadas, asalariadas en las categorías 1 a 4 de P05C16 y con un salario mensual positivo registrado, por lo que quedan fuera los trabajadores por cuenta propia, los patronos, los trabajadores familiares no remunerados y quienes no reportaron salario. Las asociaciones encontradas son descriptivas y no demuestran relaciones causales ni constituyen recomendaciones sobre cuánto debería ganar una persona.

## Referencias

Instituto Nacional de Estadística de Guatemala. Encuesta Nacional de Empleo e Ingresos. https://www.ine.gob.gt/encuesta-nacional-de-empleo-e-ingresos/

The Apache Software Foundation. MLlib: Main Guide, Spark 3.5.1 Documentation. https://spark.apache.org/docs/3.5.1/ml-guide.html
