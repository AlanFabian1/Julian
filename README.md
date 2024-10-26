# Julian
Modelo de predicción de delitos en la cidad de México, basados en datos de la Fiscalía de la Ciudad de México

DashBoard Tableau: https://us-east-1.online.tableau.com/#/site/a01368831-c74c8bc863/redirect_to_view/10912758


Descripción General
Este script utiliza pandas, datetime, y pycaret.regression para realizar un análisis de series de tiempo y predecir el número de delitos en distintas alcaldías y grupos de delitos en la Ciudad de México para el año 2024. Los pasos incluyen la preparación de los datos, la configuración de un entorno de modelado en PyCaret, la selección y entrenamiento de un modelo, y finalmente, la predicción de delitos futuros.

Librerías Importadas
pandas: Para la manipulación de datos.
datetime: Para trabajar con fechas.
pycaret.regression: Módulo de PyCaret para regresión, utilizado para la configuración, selección y entrenamiento de modelos de predicción.
Explicación Paso a Paso
1. Cargar los Datos
python
Copiar código
df = data_exportable.copy()
Se copia el conjunto de datos original data_exportable en df para no modificar el original.

2. Convertir la Fecha a un Objeto de Tiempo
python
Copiar código
df['fecha'] = pd.to_datetime(df['fecha'])
La columna fecha se convierte al formato datetime para realizar manipulaciones y extracciones de información de tiempo.

3. Crear Columnas de Año, Mes y Día de la Semana
python
Copiar código
df['año'] = df['fecha'].dt.year
df['mes'] = df['fecha'].dt.month
df['dia_semana'] = df['fecha'].dt.dayofweek  # 0=Lunes, 6=Domingo
Nuevas columnas para año, mes, y día de la semana se generan para facilitar el análisis temporal y mejorar el rendimiento del modelo de predicción.

4. Filtrar los Datos
python
Copiar código
df = df[df['año'] < 2024]
Los datos se filtran para incluir solo los años anteriores a 2024, asegurando que el modelo se entrene sin incluir datos del año que se quiere predecir.

5. Agrupar Datos por Año, Mes, Grupo de Delito y Alcaldía
python
Copiar código
df_grouped = df.groupby(['año', 'mes', 'Grupo_Delito', 'Alcaldia']).size().reset_index(name='num_delitos')
Los datos se agrupan y cuentan por combinación de año, mes, Grupo_Delito, y Alcaldia, resultando en un DataFrame que contiene el número de delitos (num_delitos) para cada combinación.

6. Configurar el Entorno de PyCaret
python
Copiar código
exp = setup(data=df_grouped, target='num_delitos', ignore_features=['fecha'], fold_strategy='timeseries', fold_shuffle=False, data_split_shuffle=False, session_id=123)
PyCaret se configura para realizar modelado de series de tiempo:

target: La columna objetivo num_delitos que se desea predecir.
ignore_features: Se ignora la columna fecha durante el modelado.
fold_strategy: Se usa la estrategia de series de tiempo para dividir los datos en pliegues (folds).
fold_shuffle y data_split_shuffle: Desactivados para preservar el orden temporal en los datos.
session_id: Un valor para asegurar reproducibilidad en el modelo.
7. Comparar Modelos y Seleccionar el Mejor
python
Copiar código
best_model = compare_models()
PyCaret prueba varios modelos de regresión y selecciona el que ofrece el mejor rendimiento para la predicción de delitos.

8. Entrenar el Modelo Seleccionado
python
Copiar código
final_model = finalize_model(best_model)
El modelo seleccionado se entrena completamente con el conjunto de datos de entrenamiento para prepararlo para la predicción.

9. Crear un DataFrame con Datos de 2024 para Predicciones
python
Copiar código
fechas_2024 = pd.date_range(start='2024-01-01', end='2025-12-31', freq='M')
df_2024 = pd.DataFrame({'fecha': fechas_2024})
df_2024['año'] = df_2024['fecha'].dt.year
df_2024['mes'] = df_2024['fecha'].dt.month
Un rango de fechas mensuales desde enero de 2024 hasta diciembre de 2024 se genera para crear el DataFrame df_2024, que contiene el año y mes de cada mes del año.

10. Generar Combinaciones de Grupo de Delito y Alcaldía
python
Copiar código
grupos_delito = df['Grupo_Delito'].unique()
alcaldias = df['Alcaldia'].unique()
Se obtiene una lista de valores únicos de Grupo_Delito y Alcaldia para asegurar que todas las combinaciones posibles estén representadas en el conjunto de predicciones.

11. Crear un DataFrame con Todas las Combinaciones
python
Copiar código
df_2024 = df_2024.assign(key=1).merge(pd.DataFrame({'Grupo_Delito': grupos_delito, 'key': 1}), on='key').drop('key', axis=1)
df_2024 = df_2024.assign(key=1).merge(pd.DataFrame({'Alcaldia': alcaldias, 'key': 1}), on='key').drop('key', axis=1)
Usando una combinación con una columna clave temporal key, se crea un DataFrame con todas las combinaciones de año, mes, Grupo_Delito, y Alcaldia.

12. Añadir el Día de la Semana
python
Copiar código
df_2024['dia_semana'] = df_2024['fecha'].dt.dayofweek
Se agrega una columna dia_semana que especifica el día de la semana de cada fecha en df_2024.

13. Realizar las Predicciones para 2024
python
Copiar código
predictions = predict_model(final_model, data=df_2024)
Usando el modelo entrenado final_model, se predicen los delitos para cada combinación en df_2024 y se almacena el resultado en predictions.

Notas Adicionales
Este código permite generar predicciones mensuales de delitos a nivel de alcaldía y tipo de delito. La estructura temporal de los datos y el uso de PyCaret simplifican el proceso de comparación de modelos y generan predicciones confiables para la planificación y el análisis de políticas de seguridad pública en la Ciudad de México para el año 2024.
