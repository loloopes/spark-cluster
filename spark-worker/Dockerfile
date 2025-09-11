# Official Spark image
FROM apache/spark:3.5.2

# ---- Add S3/MinIO support ----
# Keep these aligned with Hadoop 3.3.x bundled in Spark 3.5.x
ARG HADOOP_AWS_VER=3.3.6
ARG AWS_SDK_BUNDLE_VER=1.12.262

RUN set -eux; \
    curl -fsSLo /opt/spark/jars/hadoop-aws-${HADOOP_AWS_VER}.jar \
      https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-aws/${HADOOP_AWS_VER}/hadoop-aws-${HADOOP_AWS_VER}.jar; \
    curl -fsSLo /opt/spark/jars/aws-java-sdk-bundle-${AWS_SDK_BUNDLE_VER}.jar \
      https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk-bundle/${AWS_SDK_BUNDLE_VER}/aws-java-sdk-bundle-${AWS_SDK_BUNDLE_VER}.jar

# Optional: default S3A/MinIO settings (adjust endpoint/keys as needed)
# You can also mount this file instead of baking it in.
COPY ./conf/spark-defaults.conf /opt/spark/conf/spark-defaults.conf

# Run master in foreground for Docker
CMD bash -lc 'SPARK_NO_DAEMONIZE=1 /opt/spark/sbin/start-master.sh'
