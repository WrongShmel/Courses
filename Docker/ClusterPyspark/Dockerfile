# ================================
# Stage 1: base
# ================================
FROM eclipse-temurin:17-jre-jammy AS base

ARG shared_workspace=/opt/workspace

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        curl \
        python3 \
        python3-pip \
        ca-certificates && \
    ln -s /usr/bin/python3 /usr/bin/python && \
    mkdir -p ${shared_workspace} && \
    rm -rf /var/lib/apt/lists/*

ENV SHARED_WORKSPACE=${shared_workspace}
VOLUME ${shared_workspace}
WORKDIR ${shared_workspace}

# ================================
# Stage 2: spark-base
# ================================
FROM base AS spark-base

ARG spark_version=4.0.0
ARG hadoop_version=3

RUN curl -L https://archive.apache.org/dist/spark/spark-${spark_version}/spark-${spark_version}-bin-hadoop${hadoop_version}.tgz \
    | tar -xzC /opt && \
    mv /opt/spark-${spark_version}-bin-hadoop${hadoop_version} /opt/spark

ENV SPARK_HOME=/opt/spark
ENV PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin
ENV PYSPARK_PYTHON=python3

# ================================
# Stage 3: spark-master
# ================================
FROM spark-base AS spark-master
EXPOSE 7077 8080
CMD ["/opt/spark/sbin/start-master.sh"]

# ================================
# Stage 4: spark-worker
# ================================
FROM spark-base AS spark-worker
EXPOSE 8081
CMD ["/opt/spark/sbin/start-worker.sh", "spark://spark-master:7077"]

# ================================
# Stage 5: jupyterlab
# ================================
FROM spark-base AS jupyterlab

RUN pip install --no-cache-dir \
    jupyterlab==4.2.5 \
    pyspark==4.0.0 \
    pandas pyarrow matplotlib seaborn

EXPOSE 8888
WORKDIR /opt/workspace
CMD ["jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--allow-root", "--NotebookApp.token=", "--NotebookApp.password="]