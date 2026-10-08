# 🚦 Movilidad Urbana y Economía | De datos a decisiones de infraestructura

**Análisis de Datos | Python · Pandas · NumPy · Seaborn · Matplotlib · Jupyter Notebook**

## 1. 📊 Contexto y problema de negocio

La movilidad urbana es un factor importante para el funcionamiento de las grandes ciudades. Los congestionamientos vehiculares pueden afectar los tiempos de traslado, la accesibilidad y el desarrollo de las actividades económicas.

Sin embargo, una ciudad con mayor congestión no necesariamente presenta menor productividad económica. Para comprender esta relación, es necesario comparar indicadores de movilidad con variables económicas y demográficas.

El desafío de este proyecto consistió en analizar información procedente de dos fuentes:

- **TomTom Traffic Index:** indicadores de congestión, atascos y tiempos de viaje.
- **OECD Cities:** indicadores de PIB per cápita, desempleo y población.

La información se encontraba en archivos independientes, con diferentes estructuras, formatos de datos y niveles de detalle, lo que dificultaba realizar comparaciones directas.

### Preguntas de negocio

- ¿Qué relación existe entre la congestión vehicular y el PIB per cápita?
- ¿Las ciudades económicamente más productivas presentan mayores niveles de tráfico?
- ¿Qué ciudades combinan problemas de movilidad con indicadores económicos desfavorables?
- ¿Qué ciudades podrían considerarse prioritarias para investigar inversiones en infraestructura de transporte?

**Problema central:** integrar indicadores de movilidad y economía para identificar patrones urbanos que permitan orientar estudios y decisiones sobre infraestructura de transporte.

---

## 2. 🎯 Objetivo y estrategia de análisis

El objetivo fue analizar la relación entre las condiciones de movilidad urbana y la productividad económica durante 2024, mediante la preparación, integración y visualización de información de ambas fuentes.

Para abordar el problema, organicé el análisis en cinco etapas:

1. **Exploración:** comprender la estructura y las características de los datasets.
2. **Preparación:** corregir tipos de datos, estandarizar columnas y delimitar el período de análisis.
3. **Agregación:** calcular promedios anuales de indicadores de tráfico por ciudad.
4. **Integración y visualización:** combinar información económica y de movilidad para identificar patrones.
5. **Interpretación:** formular conclusiones y recomendaciones a partir de los resultados exploratorios.

La estrategia consistió en preparar las fuentes de información, comparar indicadores urbanos y utilizar los hallazgos para identificar ciudades que requieren mayor investigación.

---

## 3. 🧰 Tecnologías utilizadas

| Herramienta | Aplicación |
|---|---|
| Python | Desarrollo del análisis |
| Pandas | Limpieza, transformación, agregaciones y unión de datasets |
| NumPy | Apoyo para operaciones numéricas |
| Seaborn | Visualización de distribuciones y valores atípicos |
| Matplotlib | Gráficos comparativos e histogramas |
| Jupyter Notebook | Ejecución, documentación e interpretación de resultados |

### Fuentes de datos

| Dataset | Contenido |
|---|---|
| `tomtom_traffic.csv` | Congestión, duración de viajes y atascos |
| `oecd_city_economy.csv` | PIB per cápita, desempleo, población e indicadores urbanos |

**Período seleccionado:** 2024.

**Cobertura económica reportada:** 15 ciudades de 7 países, según el resumen ejecutivo del notebook.

---

## 4. 🔎 Desarrollo del proyecto

### Etapa 1. Exploración de las fuentes de información

**Problema:** los datasets contenían información complementaria, pero utilizaban estructuras y formatos diferentes.

Comencé importando las librerías necesarias y cargando ambos archivos CSV en DataFrames independientes.

**Acciones realizadas:**

- Exploración de las primeras filas con `.head()`.
- Revisión de nombres de columnas y tipos de datos.
- Identificación de variables relevantes para el análisis.
- Detección de campos almacenados como texto que requerían conversión.

Se identificó que las fechas del dataset de tráfico estaban almacenadas como `object`, mientras que algunos indicadores económicos, como el PIB per cápita y el porcentaje de desempleo, también necesitaban transformación.

**Resultado:** identifiqué las principales incompatibilidades que debían corregirse para preparar los datos.

### Etapa 2. Limpieza y transformación de datos

**Problema:** los formatos originales dificultaban realizar operaciones numéricas y filtrar correctamente los registros.

Para solucionarlo, apliqué transformaciones utilizando Pandas.

**Trabajo realizado:**

- Renombré columnas para facilitar su identificación.
- Convertí fechas mediante `pd.to_datetime()`.
- Utilicé `errors='coerce'` y `utc=True` para procesar los campos temporales.
- Eliminé símbolos y ajusté separadores decimales.
- Convertí el PIB per cápita y la tasa de desempleo a `float64`.
- Transformé la población expresada en millones a valores absolutos.
- Verifiqué los tipos de datos después de las conversiones.

Una de las transformaciones fue:

```python
eco['unemployment_pct'] = (
    eco['Unemployment %']
    .astype(str)
    .str.replace('%', '')
    .str.replace(',', '.')
    .astype(float)
)
```

También calculé la población absoluta:

```python
eco['Population'] = eco['Population_m'] * 1000000
```

**Resultado:** se obtuvieron columnas numéricas adecuadas para cálculos y comparaciones, conservando las variables necesarias para integrar información económica y de movilidad.

### Etapa 3. Filtrado temporal y agregación

**Problema:** las fuentes contenían información de diferentes períodos y el dataset de tráfico registraba múltiples observaciones por ciudad.

Para realizar un análisis temporal consistente, extraje el año de las fechas de tráfico:

```python
traffic['year'] = traffic['update_time_utc'].dt.year
```

Posteriormente filtré ambas fuentes para trabajar con registros correspondientes a 2024.

Para obtener una visión resumida del tráfico, agrupé la información por ciudad, país y año utilizando `groupby()` y calculé los promedios de los indicadores numéricos.

**Indicadores analizados:**

| Indicador | Descripción |
|---|---|
| `JamsDelay` | Indicador de demora asociada a los atascos |
| `TrafficIndexLive` | Índice de tráfico registrado |
| `JamsLengthInKms` | Longitud de los congestionamientos |
| `JamsCount` | Número de atascos |
| `MinsDelay` | Diferencia en minutos respecto al tiempo histórico |
| `TravelTimeLivePer10KmsMins` | Tiempo de viaje actual por cada 10 km |
| `TravelTimeHistoricPer10KmsMins` | Tiempo histórico de viaje por cada 10 km |

**Resultado:** construí `traffic_city_year_2024`, una tabla de promedios anuales por ciudad, país y año.

La tabla agrupada incluyó 387 combinaciones de ciudad, país y año correspondientes a distintas regiones del mundo. El análisis económico posterior se enfocó en las ciudades disponibles en ambas fuentes.

### Etapa 4. Integración de movilidad y economía

**Problema:** los indicadores económicos y de tráfico se encontraban en archivos diferentes, lo que impedía analizarlos conjuntamente.

Seleccioné las columnas relevantes y utilicé `pd.merge()` con una unión `INNER` sobre ciudad y año.

```python
merged = pd.merge(
    traffic_2024_small,
    eco_2024_small,
    on=['City', 'year'],
    how='inner'
)
```

Este procedimiento permitió conservar los registros cuyas ciudades y años tenían correspondencia en ambos datasets.

El dataset integrado incluyó información sobre:

- Congestión y atascos.
- Duración de los viajes.
- PIB per cápita.
- Desempleo.
- Población.
- Contaminación por partículas PM2.5.

**Resultado:** generé un DataFrame con variables de movilidad y economía que podía utilizarse para desarrollar las comparaciones visuales.

**Consideración metodológica:** la unión implementada utiliza los registros de tráfico filtrados de 2024 y no directamente la tabla previamente agregada por ciudad y año. Por tanto, el resultado conserva múltiples observaciones de tráfico para una misma ciudad. Una versión posterior del análisis podría realizar el `merge` sobre los promedios anuales para disponer de una única observación por ciudad.

### Etapa 5. Visualización y análisis exploratorio

**Problema:** una vez integrados los datos, era necesario examinar su distribución e identificar posibles relaciones entre indicadores de movilidad y actividad económica.

Desarrollé tres visualizaciones principales.

**1. Boxplot de congestión**

Utilicé Seaborn para analizar la distribución de `JamsDelay`, incluyendo la media.

El objetivo fue explorar la dispersión de la congestión e identificar valores extremos que pudieran influir en los resultados.

**2. Histograma del PIB per cápita**

Construí un histograma de `city_gdp_capita` con Matplotlib, para observar cómo se distribuían los valores económicos entre las ciudades.

**3. Gráfico de barras comparativo**

Desarrollé una comparación visual entre `JamsDelay` y `city_gdp_capita`, agrupando los datos por ciudad y calculando promedios.

Esta visualización permitió explorar si las ciudades con mayores indicadores de congestión también presentaban mayores niveles de PIB per cápita.

**Resultado:** las gráficas facilitaron identificar diferencias entre ciudades y formular conclusiones exploratorias sobre la relación entre tráfico y economía.

La comparación visual no constituye, por sí misma, una prueba de correlación estadística o de causalidad.

---

## 5. 📈 Resultados y hallazgos principales

### Hallazgo 1. Una mayor congestión no implica necesariamente mayor productividad

La comparación exploratoria no mostró un patrón visual suficientemente consistente para afirmar que las ciudades con mayor PIB per cápita presentan necesariamente mayor congestión vehicular.

Ciudades como Buenos Aires y Montevideo sirvieron como referencias para cuestionar una relación directa entre ambas variables.

**Interpretación:** la congestión urbana y el desempeño económico deben analizarse conjuntamente con otros factores, en lugar de asumir una relación automática.

### Hallazgo 2. Ciudad de México presentó un comportamiento relevante

En la exploración se identificó a Ciudad de México como un caso destacado por su elevada congestión y sus indicadores económicos.

En la tabla anual de tráfico, Ciudad de México registró aproximadamente:

| Indicador de tráfico | Promedio 2024 |
|---|---:|
| `JamsDelay` | 2,133.4 |
| `TrafficIndexLive` | 28.21 |
| `JamsCount` | 594.97 |
| Tiempo de viaje por 10 km | 21.81 minutos |

Estos valores corresponden a los promedios calculados en el notebook y no deben interpretarse como horas o días de retraso sin validar previamente la unidad específica del indicador.

**Interpretación:** una ciudad con actividad económica importante también puede presentar problemas considerables de movilidad, por lo que el desempeño económico no debe utilizarse como sustituto de una medición de congestión.

### Hallazgo 3. Bogotá destacó como candidata para estudiar inversiones de transporte

A partir de la comparación de indicadores de congestión, PIB per cápita y desempleo, identifiqué a Bogotá como una ciudad que merecía especial atención.

En mi interpretación, Bogotá presentaba una combinación desfavorable de movilidad y condiciones económicas, lo que la convirtió en una candidata para investigar oportunidades de mejora en infraestructura.

**Interpretación:** esta combinación de indicadores puede utilizarse como criterio inicial de priorización. Sin embargo, no demuestra que la congestión sea la causa del desempeño económico ni que una inversión concreta vaya a producir una mejora cuantificable.

### Hallazgo 4. La relación entre movilidad y economía requiere más variables

Los resultados exploratorios sugieren que estudiar únicamente el PIB per cápita y la congestión resulta insuficiente para explicar las diferencias entre ciudades.

Variables como desempleo, población y características del sistema de transporte podrían aportar contexto adicional.

**Interpretación:** para construir un modelo explicativo más sólido sería necesario ampliar el análisis, incorporar controles relevantes y utilizar métodos estadísticos apropiados.

---

## 6. ✅ Validación y confiabilidad del análisis

Para documentar la preparación y consistencia de los datos, el notebook contiene diferentes verificaciones realizadas durante el proceso.

| Control | Evidencia del notebook |
|---|---|
| Exploración inicial | Inspección de registros y columnas mediante `.head()` y `.dtypes` |
| Conversión de fechas | Uso de `pd.to_datetime()` |
| Conversión numérica | Verificación mediante `.info()` |
| Consistencia temporal | Filtro de registros correspondientes a 2024 |
| Agregación de tráfico | Tabla de promedios por ciudad, país y año |
| Integración | Visualización de las primeras filas del `merge` |
| Exploración visual | Boxplot, histograma y gráfico de barras |
| Exportación | Código para generar el CSV integrado |

### Alcance de la validación

Estas comprobaciones documentan que se realizaron las principales transformaciones y que el notebook produjo salidas observables.

No obstante, el análisis presenta aspectos que podrían fortalecerse:

- Incorporar verificaciones explícitas de duplicados y valores faltantes después de la unión.
- Utilizar la tabla de promedios anuales en el dataset comparativo final.
- Verificar que las claves de unión identifiquen correctamente ciudad y país.
- Calcular una correlación estadística entre los indicadores y evaluar sus limitaciones.
- Revisar la escala de los indicadores utilizados conjuntamente en las visualizaciones.
- Documentar una comparación reproducible de todas las ciudades antes de establecer un ranking de inversión.

**Criterio de confiabilidad:** los resultados se consideran exploratorios y útiles para orientar investigaciones posteriores, pero no representan evidencia causal ni una evaluación financiera de proyectos de infraestructura.

---

## 7. 💡 Conclusiones y recomendaciones

El análisis permitió integrar información económica y de movilidad para explorar las condiciones urbanas de distintas ciudades durante 2024.

La principal conclusión fue que **no se identificó una relación visual directa y uniforme entre el PIB per cápita y la congestión vehicular**.

Esto indica que ambos indicadores deben evaluarse en conjunto con otras variables y que una ciudad económicamente productiva también puede presentar problemas importantes de tránsito.

### Recomendaciones

**1. Priorizar la investigación de ciudades con múltiples indicadores desfavorables**

Bogotá fue identificada como candidata para un estudio más profundo, considerando su combinación de congestión y condiciones económicas.

Antes de proponer inversiones específicas, sería necesario comprobar los datos e incorporar información sobre infraestructura existente, accesibilidad y demanda de transporte.

**2. Analizar la movilidad más allá del PIB per cápita**

Recomiendo ampliar las comparaciones para incluir desempleo, población, tiempos de traslado y otros indicadores urbanos.

**3. Fortalecer el análisis estadístico**

Una siguiente etapa podría incorporar correlaciones, análisis multivariable y comparaciones entre distintos años para evaluar con mayor rigor las relaciones observadas.

**4. Evaluar proyectos de infraestructura con indicadores medibles**

Para decidir sobre inversiones, sería conveniente estimar costos, ahorro potencial en tiempos de traslado y beneficios esperados para la población.

### Valor aportado

El proyecto permitió transformar dos fuentes de información independientes en un análisis exploratorio que facilita comparar ciudades e identificar posibles prioridades de investigación urbana.

Desde el punto de vista técnico, apliqué habilidades de preparación de datos, transformación de variables, agregación, integración de datasets y visualización con Python.

Desde el punto de vista del negocio, traduje los resultados en interpretaciones y recomendaciones orientadas a la evaluación de infraestructura de transporte.

---

## 8. 📁 Recursos del proyecto

**Archivos de entrada**

- `tomtom_traffic.csv`
- `oecd_city_economy.csv`

**Notebook de análisis**

- `S5 ladb_mobility_economy_project_student (1) (1).ipynb`

**Archivo de salida previsto**

- `ladb_mobility_economy_2024_clean.csv`

El notebook contiene el código de exportación:

```python
merged.to_csv(
    'ladb_mobility_economy_2024_clean.csv',
    index=False,
    encoding='utf-8-sig'
)
```

Para reproducir el análisis se requieren los datasets originales, las dependencias de Python y las rutas correspondientes a los archivos.

---

## 👨‍💻 Autor

**Manuel Eduardo Solís Vega**

Data Analyst Jr. | Python · SQL · Power BI · Business Intelligence

Proyecto de análisis exploratorio desarrollado como parte de mi portafolio profesional, enfocado en transformar información de movilidad y economía en hallazgos útiles para la toma de decisiones.
