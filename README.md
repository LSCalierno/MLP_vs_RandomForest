# Random Forest vs MLP para la detección de intrusiones en redes

## 1. Título y descripción breve

Este proyecto compara dos enfoques de aprendizaje supervisado para la detección de intrusiones en redes: **Random Forest**, como modelo clásico de aprendizaje automático, y **Multilayer Perceptron (MLP)**, como modelo de aprendizaje profundo.

Los modelos se aplican sobre el dataset **CICIDS-2017**, que contiene tráfico de red normal y diferentes tipos de ataques. El objetivo es analizar su desempeño mediante distintas métricas de clasificación y contrastar los resultados obtenidos con los reportados en el trabajo de Ali et al. (2025).

---

## 2. Motivación

La detección de intrusiones en redes es un problema de gran relevancia dentro de la seguridad informática, ya que permite identificar  distintos tipos de ataques a partir de características observadas en el tráfico de red.

El trabajo toma como referencia el artículo *Deep Learning vs. Machine Learning for Intrusion Detection in Computer Networks: A Comparative Study* (Ali et al., 2025), que compara modelos clásicos de aprendizaje automático con modelos de aprendizaje profundo sobre CICIDS-2017.

La comparación entre Random Forest y MLP resulta especialmente interesante porque permite analizar si un modelo de aprendizaje profundo presenta una ventaja frente a un modelo clásico en un problema de datos tabulares y, a partir de los resultados, discutir bajo qué condiciones podría ser conveniente cada enfoque.

---

## 3. Datos

### Dataset

Se utiliza **CICIDS-2017 (Canadian Institute for Cybersecurity Intrusion Detection Evaluation Dataset 2017)**.

El dataset contiene tráfico de red capturado durante cinco días, incluyendo tanto tráfico normal como diferentes tipos de ataques. En su versión original cuenta con aproximadamente **2.830.743 registros y 79 columnas**, incluyendo la variable objetivo `Label`.

Las variables predictoras corresponden principalmente a características estadísticas del tráfico de red, mientras que `Label` identifica el tipo de tráfico.

### Etiquetas

Para simplificar el problema de clasificación, las etiquetas originales se consolidaron en categorías comunes:

| Etiquetas originales | Categoría |
|---|---|
| BENIGN | BENIGN |
| Etiquetas que contienen `DoS` | DoS |
| Etiquetas que contienen `DDoS` | DDoS |
| Etiquetas que contienen `PortScan` | PortScan |
| Etiquetas que contienen `Bot` | Bot |
| Etiquetas que contienen `Web Attack` | Web Attack |
| Resto de etiquetas | Other |

De esta manera, el problema queda planteado como una clasificación multiclase con seis categorías principales.

El dataset original no se modifica. Los datos procesados utilizados para el modelado se almacenan en `data/processed/`.

---

## 4. Metodología

El análisis sigue un pipeline de preprocesamiento basado en el procedimiento indicado en el paper de referencia.

### Preprocesamiento

1. **Eliminación de duplicados**

   Se identifican y eliminan registros duplicados para evitar que observaciones repetidas tengan una influencia excesiva sobre el entrenamiento.

2. **Tratamiento de valores faltantes**

   Los valores faltantes se imputan utilizando la **media de cada variable numérica**, siguiendo el procedimiento indicado en el trabajo de referencia.

3. **Eliminación de valores infinitos**

   Se eliminan los registros que contienen valores `+∞` o `-∞`, que pueden aparecer en determinadas características derivadas del tráfico de red.

4. **Consolidación de etiquetas**

   Las diferentes variantes de las etiquetas originales se agrupan en categorías comunes para reducir la complejidad del problema y obtener clases con un significado más general.

5. **Separación entrenamiento/test**

   Los datos se dividen en conjuntos de entrenamiento y prueba utilizando una partición estratificada según la variable objetivo. De esta forma se conserva la proporción de las clases en ambos conjuntos.

6. **Estandarización**

   Se aplica `StandardScaler` a las variables numéricas. El escalador se ajusta únicamente sobre el conjunto de entrenamiento y posteriormente se aplica tanto a entrenamiento como a prueba.

   La estandarización resulta particularmente importante para el MLP, ya que las redes neuronales pueden verse afectadas por variables con escalas muy diferentes.

7. **Selección de variables**

   Se calcula la matriz de correlación entre las variables y se eliminan aquellas que presentan una correlación absoluta superior a `0.85` con otra variable, conservando una variable representativa de cada conjunto de features altamente correlacionadas.

8. **Balanceo mediante SMOTE**

   El dataset presenta un fuerte desbalance entre clases, principalmente debido al predominio de la clase `BENIGN`. Para reducir este problema se utiliza **SMOTE (Synthetic Minority Over-sampling Technique)** únicamente sobre el conjunto de entrenamiento.

   El conjunto de prueba no se modifica, ya que debe representar datos no vistos por los modelos.

### Modelos

Se entrenan y comparan dos modelos:

- **Random Forest:** conjunto de árboles de decisión entrenados sobre muestras bootstrap y con selección aleatoria de variables en cada división. El uso de múltiples árboles permite reducir la varianza y obtener un modelo robusto para datos tabulares.

- **MLP (Multilayer Perceptron):** red neuronal formada por una capa de entrada, capas ocultas completamente conectadas y una capa de salida. Las capas ocultas utilizan funciones de activación no lineales y los parámetros se ajustan mediante optimización basada en gradiente y backpropagation.

Los principales hiperparámetros utilizados se documentan en el notebook de modelado.

### Evaluación

El desempeño de ambos modelos se evalúa sobre el conjunto de prueba mediante:

- Accuracy
- Precision
- Recall
- F1-score macro
- F1-score ponderado
- Matriz de confusión

También se analiza la evolución de la pérdida durante el entrenamiento del MLP.

---

## 5. Resultados principales

Los resultados propios se comparan con los valores reportados por Ali et al. (2025).

Los resultados se analizan teniendo en cuenta tanto las métricas globales como la matriz de confusión, prestando especial atención al comportamiento de las distintas clases.

Las diferencias respecto del paper pueden estar relacionadas con factores como la versión concreta del dataset, las decisiones de preprocesamiento, los hiperparámetros utilizados y la semilla aleatoria. 

---

## 6. Cómo ejecutar

El proyecto está organizado como un repositorio reproducible:

tpfinal_mad1_randomforest_mlp/
│
├── README.md
├── data/
│   ├── raw/
│   └── processed/
│
├── notebooks/
│   ├── Exploracion_y_preprocesamiento.ipynb
│   ├── Modelado.ipynb
│   └── Resultados.ipynb
│
├── src/
│   └── utils.py
│
├── reports/
    |── reporte_final.pdf
    └── reporte_final.txt