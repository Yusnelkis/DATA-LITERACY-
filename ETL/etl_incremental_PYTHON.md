
# ETL Incremental de Emisiones de Carbono

Este proyecto simula un flujo ETL (Extracción, Transformación y Carga) incremental para una base de datos de emisiones de carbono. El objetivo es extraer solo los datos nuevos o actualizados desde una fecha específica, transformarlos y luego cargarlos en una tabla de destino para análisis.

## Descripción del Proyecto

Este script ETL está diseñado para ejecutarse periódicamente y actualizar solo los registros nuevos o modificados en la tabla `emisiones_procesadas` de la base de datos `emisiones_carbono`. La lógica incremental utiliza una columna `updated_at` en la tabla `emisiones` para filtrar los registros que han cambiado desde la última ejecución.

## Requisitos

- **Python 3.x**
- **MySQL** (corriendo en XAMPP o cualquier otro servidor)
- **Bibliotecas de Python**:
  - `pandas`
  - `SQLAlchemy`
  - `pymysql`

Para instalar las bibliotecas necesarias, ejecuta:
```bash
pip install pandas sqlalchemy pymysql
```

## Estructura de la Tabla de Emisiones

La tabla `emisiones` en la base de datos `emisiones_carbono` debe tener la siguiente estructura:

| Columna               | Tipo         | Descripción                           |
|-----------------------|--------------|---------------------------------------|
| `id`                  | INT          | Identificador único                   |
| `sector`              | VARCHAR(255) | Sector industrial                     |
| `emisiones_toneladas` | FLOAT        | Emisiones en toneladas                |
| `fecha`               | DATE         | Fecha de las emisiones                |
| `updated_at`          | TIMESTAMP    | Marca de tiempo de última actualización |

## Código ETL Incremental en Python

Guarda el siguiente código en un archivo llamado `etl_incremental.py`.

```python
from sqlalchemy import create_engine
import pandas as pd

# Configuración de la conexión a la base de datos
engine = create_engine("mysql+pymysql://root:2611@127.0.0.1:3306/emisiones_carbono")

# Definir la fecha de la última carga de datos (fecha de referencia para ETL incremental)
ultima_actualizacion = '2023-01-01'  # Fecha de referencia

# Extracción: Consultar los datos nuevos o modificados
query_incremental = f"""
SELECT * FROM emisiones 
WHERE updated_at > '{ultima_actualizacion}'
"""

# Leer los datos en un DataFrame de Pandas
df_incremental = pd.read_sql(query_incremental, engine)
print("Datos extraídos:\n", df_incremental)

# Transformación: Calcular el porcentaje de emisiones por sector respecto al total
total_emisiones = df_incremental['emisiones_toneladas'].sum()
df_incremental['porcentaje_emisiones'] = (df_incremental['emisiones_toneladas'] / total_emisiones) * 100
print("Datos transformados:\n", df_incremental)

# Carga: Guardar los datos transformados en una tabla de destino
df_incremental.to_sql('emisiones_procesadas', engine, if_exists='append', index=False)
print("Datos cargados en la tabla 'emisiones_procesadas'.")
```

## Ejecución Automática con el Programador de Tareas en Windows

Para automatizar la ejecución de este script en intervalos regulares, puedes usar el **Programador de Tareas de Windows**. Sigue estos pasos:

1. Guarda el archivo `etl_incremental.py` en una ubicación accesible, por ejemplo: `C:\Scripts\etl_incremental.py`.
2. Abre el **Programador de Tareas** en Windows.
3. Crea una nueva tarea con las siguientes configuraciones:
   - **Acción**: Ejecutar un programa.
   - **Programa o script**: La ruta de tu ejecutable de Python, por ejemplo `C:\Python39\python.exe` o la ruta de Anaconda si estás usando Anaconda.
   - **Agregar argumentos**: La ruta del archivo `etl_incremental.py`, por ejemplo:
     ```plaintext
     C:\Scripts\etl_incremental.py
     ```
4. Configura el desencadenador para que la tarea se ejecute en el intervalo deseado.

## Verificación

Para verificar que el ETL está funcionando, puedes ejecutar el siguiente código en Python o hacer una consulta SQL directamente a la tabla `emisiones_procesadas` en MySQL:

```python
# Verificación de los datos cargados en la tabla de destino
query_verificacion = "SELECT * FROM emisiones_procesadas LIMIT 10"
df_verificacion = pd.read_sql(query_verificacion, engine)
print("Datos en la tabla de destino:\n", df_verificacion)
```

---

## Notas Adicionales

- **Incrementalidad**: Asegúrate de actualizar la variable `ultima_actualizacion` en cada ejecución automática para capturar correctamente solo los datos nuevos o modificados.
- **Optimización**: Si los datos son grandes, considera optimizar la consulta y la transformación para reducir el tiempo de ejecución.

¡Este flujo ETL incremental está diseñado para ayudarte a gestionar y analizar los datos de emisiones de manera más eficiente!
