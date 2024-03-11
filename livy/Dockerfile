FROM maven:3.8.6-openjdk-8-slim

# Arguments
ARG LIVY_TAGGED_RELEASE
ARG SCALA_VERSION
ARG SPARK_VERSION
ENV SPARK_VERSION=$SPARK_VERSION
ARG HADOOP_VERSION_SHORT
ENV HADOOP_VERSION_SHORT=$HADOOP_VERSION_SHORT
ARG ELASTICSEARCH_HADOOP_CONNECTOR_VERSION
ENV ELASTICSEARCH_HADOOP_CONNECTOR_VERSION=$ELASTICSEARCH_HADOOP_CONNECTOR_VERSION
ENV LIVY_TAGGED_RELEASE=$LIVY_TAGGED_RELEASE
ENV SCALA_VERSION=$SCALA_VERSION
ENV ELASTICSEARCH_HADOOP_CONNECTOR_VERSION=$ELASTICSEARCH_HADOOP_CONNECTOR_VERSION
ENV LANG=en_US.UTF-8
ENV LANGUAGE=en_US:en
ENV LC_ALL=en_US.UTF-8
ENV PYTHONHASHSEED=0
ENV PYTHONIOENCODING=UTF-8
ENV PIP_DISABLE_PIP_VERSION_CHECK=1


# Download and install Livy
RUN apt-get update \
 && apt-get install git python3-dev python3-pip unzip lsof -y \
 && update-alternatives --install /usr/bin/python python /usr/bin/python3 10 \
 && pip install py4j pyjxslt \
 && cd /opt \
 && git clone https://github.com/apache/incubator-livy livy \
 && cd livy \
 && git checkout -b tags/${LIVY_TAGGED_RELEASE} ${LIVY_TAGGED_RELEASE} \
 && mvn package -DskipTests \
 && mkdir logs \
 && ln -s /combinelib/ingestion3_${SCALA_VERSION}-0.0.1.jar /opt/livy/rsc/target/jars/ingestion3.jar \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

# Copy configurations
COPY livy/livy.conf /opt/livy/conf/livy.conf
RUN cat /opt/livy/conf/livy.conf
COPY livy/livy-env.sh /opt/livy/conf/livy-env.sh
COPY livy/log4j.properties /opt/livy/conf/log4j.properties


# Create symlink to ElasticSearch/Hadoop connector
RUN ln -s /combinelib/elasticsearch-hadoop-${ELASTICSEARCH_HADOOP_CONNECTOR_VERSION}.jar /opt/livy/rsc/target/jars/elasticsearch-hadoop-${ELASTICSEARCH_HADOOP_CONNECTOR_VERSION}.jar

# Download and install Spark
RUN cd /tmp \
 && curl https://archive.apache.org/dist/spark/spark-${SPARK_VERSION}/spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION_SHORT}.tgz -o /tmp/spark.tgz \
 && tar -xvf spark.tgz \
 && mv /tmp/spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION_SHORT} /opt/spark \
 && rm /tmp/spark.tgz
ENV SPARK_HOME=/opt/spark

# Copy configurations
COPY livy/spark-defaults.conf /opt/spark/conf/spark-defaults.conf

# Remove ivy cache directory
# https://github.com/MI-DPLA/combine-docker/issues/5
RUN rm -r /root/.m2