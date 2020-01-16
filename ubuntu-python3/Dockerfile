# Ubuntu 18.04 (bionic) from 2019-06-12
# https://github.com/tianon/docker-brew-ubuntu-core/commit/3c462555392cb188830b7c91e29311b5fad90cfe
FROM ubuntu:bionic-20190612@sha256:9b1702dcfe32c873a770a32cfd306dd7fc1c4fd134adfb783db68defc8894b3c

RUN apt-get update --fix-missing && \
    apt-get install -yq --no-install-recommends \
    ca-certificates \
    sudo \
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
    && dpkg-reconfigure -f noninteractive locales \
    && echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen \
    && locale-gen &&\
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Users with other locales should set this in their derivative image
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

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

RUN conda install --quiet --yes \
  'conda-forge::blas=*=openblas' \
  'cython=0.29*' \
  'pip=19.3*' \
  'ipython=7.11.1' && \
  conda clean --all -f -y

COPY requirements.txt requirements.txt
RUN pip install --no-cache-dir -r requirements.txt
