FROM maven:3.6-jdk-8 AS downloader
SHELL ["/bin/bash", "-c"]
RUN mkdir -p /build/plugins
WORKDIR /build/

RUN set -eux; \
    apt-get update; \
    apt-get install -y curl tar;

COPY ./pom.xml /build/
RUN mvn dependency:resolve \
        dependency:copy-dependencies -DoutputDirectory=plugins/confluent-schema-registry/;

ENV DEBEZIUM_VERSION=2.1.3.Final \
    MAVEN_REPO_CENTRAL=https://repo1.maven.org/maven2
RUN set -eux; \
    DEBEZIUM_CONNECTOR=( \
        mysql postgres \
        # mongodb sqlserver oracle db2 \
        # cassandra-3 cassandra-4 vitess \
        # spanner \
    ); \
    for connector in ${DEBEZIUM_CONNECTOR[@]}; do \
        curl -sSL "${MAVEN_REPO_CENTRAL}/io/debezium/debezium-connector-${connector}/${DEBEZIUM_VERSION}/debezium-connector-${connector}-${DEBEZIUM_VERSION}-plugin.tar.gz" \
            | tar -xzvf - -C plugins --no-same-owner; \
    done

FROM quay.io/strimzi/kafka:0.34.0-kafka-3.4.0
LABEL maintainer="Do Ngoc Son <dongocson.work@gmail.com>"

COPY --from=downloader /build/plugins /opt/kafka/plugins
