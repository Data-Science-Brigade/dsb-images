FROM ubuntu:18.04

WORKDIR /mnt/code

RUN apt-get update --fix-missing && \
    apt-get install -yq --no-install-recommends \
    wget \
    bzip2 \
    ca-certificates \
    build-essential \
    libatlas-base-dev \
    byobu \
    curl \
    git-core \
    htop \
    pkg-config \
    locales \
    python-dev \
    gfortran \
    unzip \
    unrar \
    openjdk-8-jdk* openjdk-8-jre-headless ca-certificates-java \
    vim \
    python-pydot \
    python-pydot-ng \
    graphviz \
    ssh \
    zip \
    && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Configure environment
ENV CONDA_DIR=/opt/conda \
    SHELL=/bin/bash \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    LANGUAGE=en_US.UTF-8

RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen

ENV PATH=$CONDA_DIR/bin:$PATH

ENV MINICONDA_VERSION 4.7.10
RUN cd /tmp && \
    wget https://repo.continuum.io/miniconda/Miniconda3-${MINICONDA_VERSION}-Linux-x86_64.sh && \
    echo "1c945f2b3335c7b2b15130b1b2dc5cf4 *Miniconda3-${MINICONDA_VERSION}-Linux-x86_64.sh" | md5sum -c - && \
    /bin/bash Miniconda3-${MINICONDA_VERSION}-Linux-x86_64.sh -f -b -p $CONDA_DIR && \
    rm Miniconda3-${MINICONDA_VERSION}-Linux-x86_64.sh && \
    $CONDA_DIR/bin/conda config --system --prepend channels conda-forge && \
    $CONDA_DIR/bin/conda config --system --set auto_update_conda false && \
    $CONDA_DIR/bin/conda config --system --set show_channel_urls true && \
    $CONDA_DIR/bin/conda install --quiet --yes conda="${MINICONDA_VERSION%.*}.*" && \
    $CONDA_DIR/bin/conda update --all --quiet --yes && \
    conda clean -tips

RUN conda install --quiet --yes \
    'conda-forge::blas=*=openblas' \
    'pandas=0.23.4' \
    'numexpr=2.6*' \
    'matplotlib=2.2*' \
    'scipy=1.1*' \
    'seaborn=0.9*' \
    'scikit-learn=0.20*' \
    'cython=0.28*' \
    'statsmodels=0.9*' \
    'numpy==1.15.1' \
    'bokeh=0.13*' \
    'xlrd=1.2.0' \
    'pyarrow=0.11.1' && \
    conda remove --quiet --yes --force qt pyqt && \
    conda clean -tipsy

RUN conda install pip
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

ENV  APACHE_SPARK_VERSION=2.4.1 \
     HADOOP_VERSION=2.7
RUN cd /tmp && \
        wget -q https://archive.apache.org/dist/spark/spark-${APACHE_SPARK_VERSION}/spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz && \
        echo "360a7b57290537c5eb3570c70d0d0b9580c4f9db8d0fa9746c3bbb6544bbb8f629901582968955aceb5649cb9d66c2d524971e4e3ef34c35d96f02ff6dba4d72 *spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz" | sha512sum -c - && \
        tar xzf spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz -C /usr/local --owner root --group root --no-same-owner && \
        rm spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz
RUN cd /usr/local && ln -s spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION} spark

RUN wget https://storage.googleapis.com/hadoop-lib/gcs/gcs-connector-hadoop2-1.9.17.jar -P /usr/local/spark/jars
RUN wget https://storage.googleapis.com/hadoop-lib/bigquery/bigquery-connector-hadoop2-latest.jar -P /usr/local/spark/jars

ENV HADOOP_HOME /
ENV SPARK_HOME /usr/local/spark
ENV SPARK_OPTS --driver-java-options=-Xms1024M --driver-java-options=-Xmx4096M --driver-java-options=-Dlog4j.logLevel=info
ENV PYTHONPATH $SPARK_HOME/python:$SPARK_HOME/python/lib/py4j-0.10.7-src.zip
ENV PATH /opt/conda/bin:$PATH

COPY requirements.txt requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

RUN /opt/conda/bin/ipython3 profile create
RUN echo 'import pandas as pd; pd.set_option("display.width", 160)' >> /root/.ipython/profile_default/ipython_config.py

EXPOSE 4040
EXPOSE 8080
EXPOSE 7077
