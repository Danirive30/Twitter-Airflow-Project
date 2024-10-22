# Proyecto de Ingeniería de Datos: Pipeline de Datos de Twitter usando Airflow

## Introducción

- Este es un proyecto de ingeniería de datos de extremo a extremo, diseñado para principiantes, en el cual se implementa un pipeline de datos utilizando Apache Airflow y Python. El objetivo del proyecto es extraer datos de Twitter mediante la API de Twitter, transformarlos utilizando Python y almacenar el resultado final en Amazon S3.

## Descripción del Proyecto

- En este proyecto, aprenderás cómo construir un pipeline de datos completo utilizando las siguientes tecnologías:

  - **Twitter API**: Para la extracción de datos (tweets).
  - **Python**: Para la transformación de los datos.
  - **Apache Airflow**: Para la orquestación del pipeline.
  - **Amazon S3**: Para el almacenamiento de los datos transformados.

## Funcionalidades Principales

1. **Extracción de datos desde la API de Twitter**: Se extraen los tweets más recientes de una cuenta específica de Twitter (por ejemplo, @elonmusk).
2. **Transformación de los datos**: Se procesan los datos extraídos y se convierten en un formato más útil y manejable.
3. **Almacenamiento de los datos**: Los datos transformados se almacenan en un archivo CSV y se pueden subir a Amazon S3 para un almacenamiento escalable.
4. **Orquestación del pipeline con Airflow**: Utilizamos Apache Airflow para automatizar y gestionar el proceso ETL (Extracción, Transformación y Carga).

## Requisitos

Para ejecutar este proyecto, necesitarás los siguientes componentes:

- Python 3.x
- Apache Airflow
- Pandas
- S3FS
- Tweepy (para acceder a la API de Twitter)
- Una cuenta de Amazon AWS con acceso a S3
- Credenciales de la API de Twitter

## Instalación y configuración

### **Instalación de dependencias**

Primero, asegúrate de tener instalado Python 3.x. Luego, puedes instalar las dependencias necesarias utilizando los siguientes comandos:

```bash
sudo apt-get update
sudo apt install python3-pip
sudo pip install apache-airflow
sudo pip install pandas 
sudo pip install s3fs
sudo pip install tweepy
```

### **Conexión a la API de Twitter y Proceso ETL**

El siguiente código se utiliza para conectarse a la API de Twitter, extraer los tweets, transformarlos y almacenarlos en un archivo CSV:

```python
import tweepy
import pandas as pd
import json
from datetime import datetime
import s3fs

def run_twitter_etl():

    access_key = "TU_ACCESS_KEY"
    access_secret = "TU_ACCESS_SECRET"
    consumer_key = "TU_CONSUMER_KEY"
    consumer_secret = "TU_CONSUMER_SECRET"

    # Autenticación en Twitter
    auth = tweepy.OAuthHandler(access_key, access_secret)
    auth.set_access_token(consumer_key, consumer_secret)

    # Creación del objeto API
    api = tweepy.API(auth)
    tweets = api.user_timeline(screen_name='@elonmusk',
                               count=200,
                               include_rts=False,
                               tweet_mode='extended')

    list = []
    for tweet in tweets:
        text = tweet._json["full_text"]

        refined_tweet = {"user": tweet.user.screen_name,
                         'text': text,
                         'favorite_count': tweet.favorite_count,
                         'retweet_count': tweet.retweet_count,
                         'created_at': tweet.created_at}

        list.append(refined_tweet)

    df = pd.DataFrame(list)
    df.to_csv('refined_tweets.csv')
```

### **DAG en Airflow**

El siguiente código define un DAG en Airflow para ejecutar el proceso ETL automáticamente cada día:

```python
from datetime import timedelta
from airflow import DAG
from airflow.operators.python_operator import PythonOperator
from airflow.utils.dates import days_ago
from datetime import datetime
from twitter_etl import run_twitter_etl

default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': datetime(2020, 11, 8),
    'email': ['airflow@example.com'],
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=1),
}

dag = DAG(
    'twitter_dag',
    default_args=default_args,
    description='Our first DAG with ETL process!',
    schedule_interval=timedelta(days=1),
)

run_etl = PythonOperator(
    task_id='complete_twitter_etl',
    python_callable=run_twitter_etl,
    dag=dag,
)

run_etl
```

## Ejecución del proyecto

1. **Iniciar Apache Airflow**: Una vez que Airflow esté instalado, puedes iniciar el servidor web de Airflow con el siguiente comando:

```bash
airflow webserver --port 8080
```

2. **Verificar y ejecutar el DAG**:

- Desde la interfaz web de Airflow, puedes verificar y ejecutar el DAG manualmente o esperar a que se ejecute de acuerdo a su programación.

3. **Revisar los datos**:

- Los datos transformados se almacenarán en un archivo CSV (`refined_tweets.csv`). Opcionalmente, puedes cargar este archivo a Amazon S3 si configuras un bucket en tu cuenta de AWS.

## Licencia

Este proyecto se distribuye bajo la licencia MIT.

# Twitter-Airflow-Project
