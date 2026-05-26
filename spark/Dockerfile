# Debian Bookworm
FROM apache/airflow:3.2.1-python3.14

COPY requirements.txt /requirements.txt

RUN pip install --no-cache-dir -r /requirements.txt

USER root

RUN apt-get update && apt-get install --no-install-recommends -y \
    wget=1.21.3-1+deb12u1 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

COPY script/entrypoint.sh /entrypoint.sh
USER airflow
ENTRYPOINT ["/bin/bash","/entrypoint.sh"]