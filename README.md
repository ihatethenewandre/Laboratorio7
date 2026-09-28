# Laboratorio 7 – Perfiles y salario de la población asalariada de la ENEIC con Spark

Análisis exploratorio, segmentación y predicción del salario mensual de la población asalariada de Guatemala con las bases de Personas de la Encuesta Nacional de Empleo e Ingresos Continua, procesadas con Spark 3.5 y la API de DataFrames de pyspark.ml; se construyen perfiles de trabajadores con KMeans y se comparan pipelines de regresión lineal y Random Forest entrenados con 2025 y evaluados en el primer trimestre de 2026.

## Integrantes

• André Emilio Pivaral López – 23574  

**Universidad del Valle de Guatemala**  
Facultad de Ingeniería  
Departamento de Computación  
Data Science  
  
**Catedrático:** Boris Fernando Becerra Peláez  
**Sección:** 30

## Descripción

El proyecto trabaja con cinco cortes de la base de Personas de la ENEIC del Instituto Nacional de Estadística: los trimestres I, II, III y IV de 2025, con 203,676 registros en total, y el trimestre I de 2026, con 49,843 registros. Cada archivo viene acompañado de su diccionario de datos. Los archivos originales se leen con openpyxl, se convierten a DataFrames de Spark con un esquema explícito y se guardan en Parquet; desde ese punto toda la preparación, las estadísticas y el aprendizaje automático se realizan con Spark y MLlib, sin scikit-learn.

El cuaderno resuelve los ocho ejercicios del laboratorio. Armoniza los cinco archivos, identifica la procedencia de cada registro, documenta los faltantes, aplica los filtros de la población asalariada en un orden fijo y verifica la unicidad de la clave de registro. Describe el salario, la edad, la antigüedad y la jornada, mide sus correlaciones con Correlation.corr y segmenta a los asalariados en perfiles con KMeans. Después construye pipelines de regresión lineal y Random Forest con StringIndexer, OneHotEncoder y VectorAssembler, selecciona la configuración de cada algoritmo con el cuarto trimestre de 2025, reentrena con todo 2025, evalúa en 2026 frente a un modelo de referencia y analiza los errores por grupo y por percentil del salario. El informe en PDF resume la metodología, los resultados y la discusión, sin incluir código.

## Estructura del proyecto

    Laboratorio7/
    ├── data/
    │   ├── processed/
    │   │   ├── eneic_2025T1_seleccion.parquet a eneic_2026T1_seleccion.parquet     columnas seleccionadas de cada corte
    │   │   ├── eneic_2025_analitica.parquet        población analítica de 2025
    │   │   ├── eneic_2026_analitica.parquet        población analítica de 2026 para la prueba final
    │   │   └── metadatos_archivos.json             estructura y tipos detectados en cada archivo original
    │   └── raw/
    │       ├── BaseDatos_Personas_ENEIC_I_2025.xlsx a BaseDatos_Personas_ENEIC_I_2026.xlsx
    │       └── Diccionario_Personas_ENEIC_I_2025.xlsx a Diccionario_Personas_ENEIC_I_2026.xlsx
    ├── figures/
    │   ├── Figura1.png a Figura13.png              figuras exportadas por el notebook
    │   └── indice_figuras.csv                      número y descripción de cada figura
    ├── models/
    │   ├── regresion_lineal_validacion/            mejor regresión lineal entrenada con 2025T1 a 2025T3
    │   ├── random_forest_validacion/               mejor Random Forest entrenado con 2025T1 a 2025T3
    │   ├── regresion_lineal_final/                 regresión lineal reentrenada con todo 2025
    │   └── random_forest_final/                    Random Forest reentrenado con todo 2025
    ├── notebook/
    │   └── Laboratorio7.ipynb
    ├── .gitignore
    ├── Informe.pdf
    └── README.md

## Requisitos

- Python; se utilizó la versión 3.12
- Java; se utilizó la versión 21.0.12
- Paquetes; pyspark, pandas, numpy, openpyxl, matplotlib, seaborn, jupyterlab y setuptools
- En Windows, Spark necesita winutils.exe y hadoop.dll de Hadoop para leer y escribir Parquet

## Contenido del análisis

- Ejercicio 1. Carga, armonización y calidad de los datos, con validación de los códigos contra los cinco diccionarios, comparación de la estructura de columnas de cada corte, unión de 2025 con unionByName, faltantes antes de filtrar, ocho filtros aplicados en orden fijo con conteo de exclusiones, verificación de unicidad de periodo, hogar y persona, y almacenamiento en Parquet de 2025 y 2026 por separado.
- Ejercicio 2. Estadística descriptiva del salario, la edad, la antigüedad y las horas, con media, mediana, desviación estándar, extremos y percentiles 25, 75 y 95, y análisis gráfico de la distribución de los registros, de la asimetría del salario y de su variación por nivel educativo, categoría ocupacional, dominio y trimestre.
- Ejercicio 3. Correlación de Pearson entre salario, edad, antigüedad y horas con VectorAssembler y Correlation.corr sobre todos los registros elegibles, complementada con la correlación de Spearman.
- Ejercicio 4. Segmentación con KMeans sobre variables estandarizadas, comparando espacios con y sin salario para K de 2 a 5 mediante inercia, silueta y tamaño del menor clúster, con un criterio de selección verificable y la descripción de cada perfil.
- Ejercicio 5. Pipeline de regresión lineal con StringIndexer, OneHotEncoder, VectorAssembler y estandarización interna, siete configuraciones de regularización comparadas por RMSE de validación, modelo de referencia, interpretación de coeficientes y almacenamiento del mejor modelo.
- Ejercicio 6. Pipeline de Random Forest con cuatro combinaciones de número de árboles y profundidad, comparación con la regresión lineal y la referencia, importancia de variables y almacenamiento del mejor modelo.
- Ejercicio 7. Reentrenamiento con todo 2025 y evaluación de ambos modelos sobre los mismos 13,258 registros de 2026 con MAE, RMSE y R².
- Ejercicio 8. Salario real frente a predicho, residuos frente a predicho, MAE y error medio por nivel educativo y dominio, y análisis del error por bandas de percentil del salario real.
