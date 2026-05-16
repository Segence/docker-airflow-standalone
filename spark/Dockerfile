FROM python:3.12-bookworm

# Never prompts the user for choices on installation/configuration of packages
ENV DEBIAN_FRONTEND noninteractive
ENV TERM linux

ARG AIRFLOW_VERSION=2.10.4
ARG AIRFLOW_HOME=/usr/local/airflow
ARG SPARK_VERSION=3.5.4
ARG HADOOP_VERSION=3
ENV AIRFLOW_GPL_UNIDECODE yes

ENV LANGUAGE en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LC_ALL en_US.UTF-8
ENV LC_CTYPE en_US.UTF-8
ENV LC_MESSAGES en_US.UTF-8

COPY requirements.txt /requirements.txt

RUN set -ex \
    && buildDeps=' \
        freetds-dev=1.3.17+ds-2 \
        libkrb5-dev=1.20.1-2+deb12u2 \
        libsasl2-dev=2.1.28+dfsg-10 \
        libssl-dev=3.0.15-1~deb12u1 \
        libffi-dev=3.4.4-1 \
        libpq-dev=15.10-0+deb12u1 \
        git=1:2.39.5-0+deb12u1 \
    ' \
    && apt-get update -yqq \
    && apt-get upgrade -yqq \
    && apt-get install -yqq --no-install-recommends  --allow-downgrades \
        $buildDeps \
        freetds-bin=1.3.17+ds-2 \
        build-essential=12.9 \
        default-libmysqlclient-dev=1.1.0 \
        apt-utils=2.6.1 \
        curl=7.88.1-10+deb12u8 \
        rsync=3.2.7-1+deb12u2 \
        netcat-openbsd=1.219-1 \
        locales=2.36-9+deb12u9 \
        iputils-ping=3:20221126-1+deb12u1 \
        telnet=0.17+2.4-2+deb12u1 \
    && sed -i 's/^# en_US.UTF-8 UTF-8$/en_US.UTF-8 UTF-8/g' /etc/locale.gen \
    && locale-gen \
    && update-locale LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8 \
    && useradd -ms /bin/bash -d ${AIRFLOW_HOME} airflow \
    && pip install -U --no-cache-dir pip==24.3.1 setuptools==75.8.0 wheel==0.45.1 \
    && pip install --no-cache-dir -r requirements.txt \
    && apt-get purge --auto-remove -yqq $buildDeps \
    && apt-get autoremove -yqq --purge \
    && apt-get clean \
    && rm -rf \
        /var/lib/apt/lists/* \
        /tmp/* \
        /var/tmp/* \
        /usr/share/man \
        /usr/share/doc \
        /usr/share/doc-base \
    && python --version \
    && pip freeze

# JAVA installation

RUN apt-get update && \
    apt-get install -y --no-install-recommends software-properties-common=0.99.30-4.1~deb12u1 gnupg2=2.2.40-1.1 && \
    apt-key adv --keyserver keyserver.ubuntu.com --recv-keys EB9B1D8886F44E2A && \
    add-apt-repository "deb http://security.debian.org/debian-security bookworm/updates main" && \ 
    apt-get update && \
    apt-get install -y --no-install-recommends openjdk-17-jdk=17.0.13+11-2~deb12u1 && \
    apt-get autoremove -yqq --purge && \
    apt-get clean && \
    rm -rf \
        /var/lib/apt/lists/* \
        /tmp/* \
        /var/tmp/* \
        /usr/share/man \
        /usr/share/doc \
        /usr/share/doc-base && \
    pip freeze && \
    java -version $$ \
    javac -version

# Setup JAVA_HOME
RUN ln -s "/usr/lib/jvm/java-17-openjdk-$(dpkg-architecture -qDEB_HOST_ARCH)" /usr/lib/jvm/java-17-openjdk
ENV JAVA_HOME /usr/lib/jvm/java-17-openjdk
RUN export JAVA_HOME

## Finished JAVA installation

# SPARK files and variables

ENV SPARK_HOME /usr/local/spark

WORKDIR /tmp

# Spark submit binaries and jars (Spark binaries must be the same version of spark cluster)
RUN wget --no-verbose "https://archive.apache.org/dist/spark/spark-${SPARK_VERSION}/spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz" && \
        tar -xvzf "spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz" && \
        mkdir -p "${SPARK_HOME}/bin" && \
        mkdir -p "${SPARK_HOME}/assembly/target/scala-2.12/jars" && \
        cp -a "spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}/bin/." "${SPARK_HOME}/bin/" && \
        cp -a "spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}/jars/." "${SPARK_HOME}/assembly/target/scala-2.12/jars/" && \
        rm "spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz"

# Create SPARK_HOME env var
RUN export SPARK_HOME
ENV PATH $PATH:/usr/local/spark/bin

# Finished SPARK files and variables

COPY script/entrypoint.sh /entrypoint.sh
COPY config/airflow.cfg ${AIRFLOW_HOME}/airflow.cfg

RUN chown -R airflow: ${AIRFLOW_HOME} && \
    chmod +x /entrypoint.sh

EXPOSE 8080 5555 8793

USER airflow
WORKDIR ${AIRFLOW_HOME}
ENTRYPOINT ["/entrypoint.sh"]
CMD ["webserver"]