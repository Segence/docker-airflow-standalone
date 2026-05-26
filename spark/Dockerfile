FROM apache/airflow:3.2.1-python3.14

COPY requirements.txt /requirements.txt

RUN pip install --no-cache-dir -r /requirements.txt

USER root

RUN apt-get update && apt-get install -y \
    wget

COPY script/entrypoint.sh /entrypoint.sh
USER airflow
ENTRYPOINT ["/bin/bash","/entrypoint.sh"]