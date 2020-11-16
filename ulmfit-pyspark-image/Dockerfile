# Source Dockerfile for image https://hub.docker.com/r/datasciencebrigade/ulmfit-pyspark-image:1.0
FROM python:3.7-stretch

WORKDIR /mnt/code

RUN apt-get update \
  && apt-get -y install sudo ssh vim gcc netcat poppler-utils wait-for-it default-jre zip

ENV TZ 'America/Sao_Paulo'
RUN echo $TZ > /etc/timezone && \
    apt-get update && apt-get install -y tzdata && \
    rm /etc/localtime && \
    ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && \
    dpkg-reconfigure -f noninteractive tzdata && \
    apt-get clean

# HADOOP
ENV HADOOP_VERSION 2.7.7
ENV HADOOP_HOME /usr/hadoop-$HADOOP_VERSION
ENV HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
ENV PATH $PATH:$HADOOP_HOME/bin
RUN curl -L --retry 3 \
  "https://archive.apache.org/dist/hadoop/core/hadoop-$HADOOP_VERSION/hadoop-$HADOOP_VERSION.tar.gz" \
  | gunzip \
  | tar -x -C /usr/ \
 && rm -rf $HADOOP_HOME/share/doc \
 && chown -R root:root $HADOOP_HOME
ENV JAVA_HOME /usr/lib/jvm/java-8-openjdk-amd64

RUN groupadd -g 10000 dsb \
  && adduser --system -u ${LOCAL_USER_ID:-1000} hdfs \
  && usermod -a -G dsb hdfs \
  && usermod -aG sudo hdfs \
  && echo "%sudo ALL=NOPASSWD: ALL" >> /etc/sudoers

RUN mkdir -p /run/sshd && ssh-keygen -A
RUN ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa \
 && cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys \
 && chmod 0600 ~/.ssh/authorized_keys \
 && echo "Host *\n   UserKnownHostsFile /dev/null\n   StrictHostKeyChecking no" >> /root/.ssh/config \
 && mkdir /home/hdfs/.ssh \
 && chmod 0700 /home/hdfs/.ssh/ \
 && cp -Rfv /root/.ssh /home/hdfs/ \
 && chown -Rfv hdfs.dsb /home/hdfs/.ssh \
 && chown -R hdfs:dsb /home/hdfs/

COPY requirements.txt requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

COPY entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/entrypoint.sh

# Textract dependencies
RUN apt update && apt install -y libxml2-dev libxslt1-dev \
  antiword unrtf poppler-utils pstotext tesseract-ocr \
  flac ffmpeg lame libmad0 libsox-fmt-mp3 sox libjpeg-dev swig

# LIBREOFFICE
RUN apt install -y libreoffice libreoffice-writer \
  ure libreoffice-java-common libreoffice-core libreoffice-common

USER hdfs

RUN python3 -c 'import stanza;\
                stanza.download("pt", processors="tokenize,pos,mwt,lemma");'
RUN python3 -c 'from fastai.datasets import URLs, untar_data;\
                untar_data(URLs.WT103_FWD, data=False);'

EXPOSE 5000
EXPOSE 4040

ENTRYPOINT ["entrypoint.sh"]
CMD ["uvicorn", "--reload", "--host", "0.0.0.0", "--port", "5000", "api:app"]
