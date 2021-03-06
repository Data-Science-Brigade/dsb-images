FROM debian:stretch
LABEL maintainer="Data Science Brigade <hello@dsbrigade.com>"
LABEL inspired_by="https://github.com/gettyimages"

RUN apt-get update --fix-missing && \
    apt-get install -yq --no-install-recommends \
    locales \
    make \
    wget \
    ssh \
    netcat \
    curl \
    bzip2 \
    python-dev \
    python3-setuptools \
    unzip \
    vim \
    zip \
    openjdk-8-jre \
    && dpkg-reconfigure -f noninteractive locales \
    && locale-gen C.UTF-8 \
    && /usr/sbin/update-locale LANG=C.UTF-8 \
    && echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen \
    && locale-gen \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

ENV JAVA_HOME /usr/lib/jvm/java-8-openjdk-amd64

# Users with other locales should set this in their derivative image
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

# http://blog.stuart.axelbrooke.com/python-3-on-spark-return-of-the-pythonhashseed
ENV PYTHONHASHSEED 0
ENV PYTHONIOENCODING UTF-8
ENV PIP_DISABLE_PIP_VERSION_CHECK 1

# Create SSH keys
RUN \
  rm -f ~/.ssh/id_rsa && \
  ssh-keygen -t rsa -N '' -f ~/.ssh/id_rsa && \
  cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys && \
  chmod 0600 ~/.ssh/authorized_keys

# Configure environment
ENV CONDA_DIR=/opt/conda \
SHELL=/bin/bash \
LC_ALL=en_US.UTF-8 \
LANG=en_US.UTF-8 \
LANGUAGE=en_US.UTF-8

ENV PATH=$CONDA_DIR/bin:$PATH
ARG PYTHON_VERSION=default

ENV MINICONDA_VERSION=4.7.10 \
    MINICONDA_MD5=1c945f2b3335c7b2b15130b1b2dc5cf4 \
    CONDA_VERSION=4.7.12
RUN cd /tmp && \
    wget --quiet https://repo.continuum.io/miniconda/Miniconda3-${MINICONDA_VERSION}-Linux-x86_64.sh && \
    echo "${MINICONDA_MD5} *Miniconda3-${MINICONDA_VERSION}-Linux-x86_64.sh" | md5sum -c - && \
    /bin/bash Miniconda3-${MINICONDA_VERSION}-Linux-x86_64.sh -f -b -p $CONDA_DIR && \
    rm Miniconda3-${MINICONDA_VERSION}-Linux-x86_64.sh && \
    echo "conda ${CONDA_VERSION}" >> $CONDA_DIR/conda-meta/pinned && \
    $CONDA_DIR/bin/conda config --system --prepend channels conda-forge && \
    $CONDA_DIR/bin/conda config --system --set auto_update_conda false && \
    $CONDA_DIR/bin/conda config --system --set show_channel_urls true && \
    if [ ! $PYTHON_VERSION = 'default' ]; then conda install --yes python=$PYTHON_VERSION; fi && \
    conda list python | grep '^python ' | tr -s ' ' | cut -d '.' -f 1,2 | sed 's/$/.*/' >> $CONDA_DIR/conda-meta/pinned && \
    $CONDA_DIR/bin/conda install --quiet --yes conda && \
    $CONDA_DIR/bin/conda update --all --quiet --yes && \
    conda clean --all -f -y && \
    rm -rf /home/$NB_USER/.cache/yarn

# copy ssh config
ADD conf/ssh/ssh_config /root/.ssh/config

# HADOOP
ENV HADOOP_VERSION 3.0.0
ENV HADOOP_HOME /usr/hadoop-$HADOOP_VERSION
ENV HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
ENV PATH $PATH:$HADOOP_HOME/bin
RUN curl -L --retry 3 \
  "http://archive.apache.org/dist/hadoop/common/hadoop-$HADOOP_VERSION/hadoop-$HADOOP_VERSION.tar.gz" \
  | gunzip \
  | tar -x -C /usr/ \
 && rm -rf $HADOOP_HOME/share/doc \
 && chown -R root:root $HADOOP_HOME

# SPARK
ENV SPARK_VERSION 2.4.4
ENV SPARK_PACKAGE spark-${SPARK_VERSION}-bin-without-hadoop
ENV SPARK_HOME /usr/spark-${SPARK_VERSION}
ENV SPARK_DIST_CLASSPATH="$HADOOP_HOME/etc/hadoop/*:$HADOOP_HOME/share/hadoop/common/lib/*:$HADOOP_HOME/share/hadoop/common/*:$HADOOP_HOME/share/hadoop/hdfs/*:$HADOOP_HOME/share/hadoop/hdfs/lib/*:$HADOOP_HOME/share/hadoop/hdfs/*:$HADOOP_HOME/share/hadoop/yarn/lib/*:$HADOOP_HOME/share/hadoop/yarn/*:$HADOOP_HOME/share/hadoop/mapreduce/lib/*:$HADOOP_HOME/share/hadoop/mapreduce/*"
ENV PATH $PATH:${SPARK_HOME}/bin
RUN curl -L --retry 3 \
  "https://archive.apache.org/dist/spark/spark-${SPARK_VERSION}/${SPARK_PACKAGE}.tgz" \
  | gunzip \
  | tar x -C /usr/ \
 && mv /usr/$SPARK_PACKAGE $SPARK_HOME \
 && chown -R root:root $SPARK_HOME

RUN conda install --quiet --yes \
  'conda-forge::blas=*=openblas' \
  'cython=0.29*' \
  'pip=19.3*' \
  'ipython=7.11.1' && \
  conda clean --all -f -y

COPY requirements.txt requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

WORKDIR $SPARK_HOME

CMD ["/usr/spark-2.4.4/bin/spark-class", "org.apache.spark.deploy.master.Master"]
