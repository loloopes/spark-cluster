# Official Spark image
FROM apache/spark:3.5.2

# ---- Add S3/MinIO support ----
# Keep these aligned with Hadoop 3.3.x bundled in Spark 3.5.x
ARG HADOOP_AWS_VER=3.3.6
ARG AWS_SDK_BUNDLE_VER=1.12.262
ARG MINIFORGE_VERSION=24.11.3-0

USER root

RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends curl bzip2 ca-certificates; \
    rm -rf /var/lib/apt/lists/*

RUN set -eux; \
    curl -fsSLo /tmp/miniforge.sh \
      https://github.com/conda-forge/miniforge/releases/download/${MINIFORGE_VERSION}/Miniforge3-Linux-x86_64.sh; \
    bash /tmp/miniforge.sh -b -p /opt/conda; \
    rm -f /tmp/miniforge.sh; \
    /opt/conda/bin/conda install -y python=3.12; \
    /opt/conda/bin/conda clean -afy

RUN set -eux; \
    curl -fsSLo /opt/spark/jars/hadoop-aws-${HADOOP_AWS_VER}.jar \
      https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-aws/${HADOOP_AWS_VER}/hadoop-aws-${HADOOP_AWS_VER}.jar; \
    curl -fsSLo /opt/spark/jars/aws-java-sdk-bundle-${AWS_SDK_BUNDLE_VER}.jar \
      https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk-bundle/${AWS_SDK_BUNDLE_VER}/aws-java-sdk-bundle-${AWS_SDK_BUNDLE_VER}.jar

ENV PATH=/opt/conda/bin:${PATH} \
    PYSPARK_PYTHON=/opt/conda/bin/python \
    PYSPARK_DRIVER_PYTHON=/opt/conda/bin/python

# Optional: default S3A/MinIO settings (adjust endpoint/keys as needed)
# You can also mount this file instead of baking it in.
COPY ./conf/spark-defaults.conf /opt/spark/conf/spark-defaults.conf
COPY ./conf/hive-site.xml /opt/spark/conf/hive-site.xml

USER spark

# Run master in foreground for Docker
CMD bash -lc 'SPARK_NO_DAEMONIZE=1 /opt/spark/sbin/start-master.sh'
