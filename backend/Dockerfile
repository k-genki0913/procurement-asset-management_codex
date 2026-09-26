# Eclipse Temurin 21.0.12+8 / Ubuntu 24.04; multi-architecture index includes ARM64.
FROM eclipse-temurin:21-jdk-noble@sha256:4d271cd5e0624598cf563342f47281b09cb364bc13acbbd7251f49f83470018d

RUN groupadd --gid 1001 developer \
    && useradd --uid 1001 --gid 1001 --create-home developer \
    && mkdir -p /workspace/backend /home/developer/.m2 \
    && chown -R developer:developer /workspace /home/developer/.m2

USER developer
WORKDIR /workspace/backend
EXPOSE 8080
CMD ["./mvnw", "--batch-mode", "spring-boot:run"]
