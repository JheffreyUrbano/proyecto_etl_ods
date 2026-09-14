# Presuntos suicidios en Colombia (2015-2024)

Proyecto de la asignatura ETL (G51) del programa de Ingeniería de Datos e Inteligencia Artificial.

## 1. Objetivo

El objetivo es construir un proceso ETL para organizar y analizar los registros de presuntos suicidios ocurridos en Colombia entre 2015 y 2024.

Con este proyecto buscamos responder preguntas como:

- ¿Cómo cambia el número de casos por año?
- ¿Qué grupos de edad y sexo concentran más registros?
- ¿Cómo se comportan los casos en algunos municipios del Valle del Cauca?
- ¿Qué información temporal, geográfica, demográfica y circunstancial se puede consultar desde una base de datos?

El proyecto no pretende diagnosticar las causas de los casos ni reemplazar un análisis médico, judicial o de salud pública. Su propósito es organizar la información disponible y facilitar su consulta.

## 2. Relación con los ODS

El proyecto se relaciona con el **ODS 3: Salud y bienestar**, especialmente con la prevención y el análisis de problemas relacionados con la salud mental.

La base permite observar diferencias por edad, sexo, año y ubicación. Estos resultados pueden servir como punto de partida para identificar grupos o territorios que requieren estudios más detallados. La información debe interpretarse con cuidado porque el conjunto contiene presuntos suicidios registrados por el sistema médico-legal, no todos los casos que puedan ocurrir en el país.

## 3. Fuente de datos y evaluación

La fuente utilizada es el portal oficial de Datos Abiertos de Colombia:

- **Dataset:** [Presuntos Suicidios. Colombia, 2015 a 2024. Cifras definitivas](https://www.datos.gov.co/Justicia-y-Derecho/Presuntos-Suicidios-Colombia-2015-a-2024-Cifras-de/f75u-mirk/about_data)
- **Entidad responsable:** Instituto Nacional de Medicina Legal y Ciencias Forenses.
- **Cobertura:** nacional.
- **Periodo:** 2015 a 2024.
- **Número de columnas:** 32.
- **Registros utilizados:** 26.558.
- **Frecuencia de actualización:** anual.
- **Última actualización consultada en el portal:** 18 de mayo de 2026.

La fuente fue seleccionada porque es pública, oficial y cumple los requisitos de la entrega: tiene más de 10.000 filas y más de 10 características. Además, contiene variables de tiempo, ubicación, características de la víctima y circunstancias del hecho.

El portal aclara que los datos corresponden a casos conocidos y registrados por Medicina Legal. Por esta razón, los resultados se presentan como un análisis de registros de presuntos suicidios y no como una medición completa de todos los casos ocurridos en Colombia.

## 4. Requisitos del proyecto

El proyecto busca cumplir los siguientes requisitos de la primera entrega:

1. Obtener un conjunto de datos relacionado con un ODS.
2. Evaluar la fuente, su calidad y sus características.
3. Limpiar y transformar el archivo CSV con Python y Pandas.
4. Diseñar un modelo dimensional para consultas analíticas.
5. Migrar los datos a PostgreSQL.
6. Realizar el EDA usando consultas a la base de datos.
7. Crear visualizaciones que respondan las preguntas del proyecto.
8. Documentar la arquitectura, las herramientas y el proceso realizado.

## 5. Arquitectura del proceso

El flujo implementado es:

```text
CSV de Datos Abiertos
  |
  v
Extracción con Pandas
  |
  v
Limpieza y homologación de categorías
  |
  v
Construcción de dimensiones y tabla de hechos
  |
  v
PostgreSQL
  |
  v
Consultas SQL y visualizaciones del EDA
```

El archivo `01_etl_pipeline.ipynb` realiza la extracción, limpieza, transformación y carga. El archivo `02_etl_eda.ipynb` consulta PostgreSQL y genera las visualizaciones.

## 6. Herramientas utilizadas

- **Python:** lenguaje principal del proyecto.
- **Pandas:** lectura del CSV, limpieza, transformación y preparación de tablas.
- **NumPy:** manejo de valores y tipos de datos.
- **Regex y Unicode:** limpieza de espacios, mayúsculas y tildes.
- **SequenceMatcher:** detección de categorías escritas de forma muy parecida.
- **SQLAlchemy y psycopg2:** conexión entre Python y PostgreSQL.
- **PostgreSQL:** almacenamiento relacional y aplicación de llaves foráneas.
- **Jupyter Notebook:** documentación y ejecución paso a paso.
- **Matplotlib y Seaborn:** elaboración de gráficos.
- **Git y GitHub:** control de versiones y organización del proyecto.

## 7. Transformaciones realizadas

Durante la limpieza se realizaron las siguientes actividades:

- Lectura del CSV con codificación UTF-8.
- Eliminación de espacios al inicio y al final de los nombres de columnas.
- Normalización de los textos: mayúsculas, espacios y tildes.
- Reemplazo de textos vacíos y valores como `N/A`, `NA`, `NULL` y `NONE` por valores nulos.
- Homologación de categorías parecidas mediante `SequenceMatcher`.
- Homologación manual de categorías cuyo significado es equivalente.
- Unificación de categorías educativas. Por ejemplo, `PREESCOLAR` y `EDUCACION INICIAL Y EDUCACION PREESCOLAR` quedan como `PREESCOLAR`; las categorías de especialización, maestría y doctorado quedan como `POSGRADO`.
- Conversión a tipo entero de `ID`, año y códigos DANE.
- Eliminación de registros completamente duplicados.
- Eliminación de duplicados por `ID`, conservando el primer registro.

Después de la limpieza se conservan 26.558 registros, sin duplicados completos y sin valores nulos inesperados en las columnas revisadas.

## 8. Modelo de datos: esquema estrella

Se implementó un esquema estrella. La tabla de hechos contiene un registro por evento y se relaciona con cuatro dimensiones.

![Esquema estrella](assets/primera_entrega_star.png)

### Tabla de hechos

`fact_presuntos_suicidios` contiene:

- `id_hecho_original`
- `id_tiempo`
- `id_geografia`
- `id_victima`
- `id_circunstancia`
- `cantidad`

### Dimensiones

- `dim_tiempo`: año, mes, día y rango de hora.
- `dim_geografia`: códigos DANE, municipio, departamento, zona y localidad.
- `dim_victima`: edad, sexo, ciclo vital, estado civil, escolaridad, pertenencia étnica y otras características.
- `dim_circunstancia`: escenario, circunstancia, manera de muerte, mecanismo, diagnóstico y razón del suicidio.

El script de creación de la base de datos se encuentra en `sql/primera_entrega.sql`. La tabla de hechos tiene llaves foráneas hacia las cuatro dimensiones.

## 9. EDA y visualizaciones

El notebook `02_etl_eda.ipynb` realiza consultas SQL sobre PostgreSQL y usa los resultados para generar:

1. **Tendencia anual:** línea con el total de casos por año entre 2015 y 2024.
2. **Perfil demográfico:** barras por grupo de edad quinquenal y sexo.
3. **Análisis geográfico:** tendencia anual para Cali, Buenaventura, Yumbo, Tuluá, Palmira y Jamundí, en el Valle del Cauca.

De esta forma, el análisis se construye a partir de las tablas almacenadas en PostgreSQL y no directamente desde el CSV original.

## 10. Estructura del repositorio

```text
etl_project/
├── assets/
│   └── primera_entrega_star.png
├── data/
│   └── Presuntos_Suicidios._Colombia,_2015_a_2024.csv
├── docs/
├── notebooks/
│   ├── 01_etl_pipeline.ipynb
│   └── 02_etl_eda.ipynb
├── sql/
│   └── primera_entrega.sql
├── .gitignore
├── README.md
└── requirements.txt
```

## 11. Instalación y ejecución

Se necesita tener instalado Python, PostgreSQL y Jupyter Notebook.

### Instalar las librerías

Desde la carpeta principal del proyecto:

```bash
pip install -r requirements.txt
```

### Crear la base de datos

1. Crear en PostgreSQL una base de datos llamada `suicidios_colombia`.
2. Ejecutar el script `sql/primera_entrega.sql` sobre esa base de datos.
3. Revisar en la celda de carga del notebook el usuario, contraseña, host, puerto y nombre de la base de datos.

### Ejecutar el ETL

Abrir `notebooks/01_etl_pipeline.ipynb` y ejecutar sus celdas en orden. La última celda limpia las tablas con `TRUNCATE ... RESTART IDENTITY CASCADE`, carga nuevamente las dimensiones y después carga la tabla de hechos.

### Ejecutar el EDA

Abrir `notebooks/02_etl_eda.ipynb` después de ejecutar el ETL. El notebook ejecuta las consultas SQL y muestra los gráficos.

## 12. Consideraciones de interpretación

Los datos tienen carácter estadístico y forense. La palabra “presuntos” es importante: los registros no constituyen por sí mismos una conclusión judicial ni explican completamente las causas de cada caso. Las visualizaciones se utilizan para observar patrones generales y no para hacer afirmaciones sobre personas o grupos específicos.

## 13. Referencias

- [Portal de Datos Abiertos de Colombia](https://www.datos.gov.co/)
- [Ficha oficial del dataset](https://www.datos.gov.co/Justicia-y-Derecho/Presuntos-Suicidios-Colombia-2015-a-2024-Cifras-de/f75u-mirk/about_data)
- [Instituto Nacional de Medicina Legal y Ciencias Forenses](https://www.medicinalegal.gov.co/)