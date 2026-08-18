# odilab-CI-Image: schlanke Multi-Arch-Runtime (amd64/arm64).
# Bewusst getrennt vom Upstream-Dockerfile (idp-server/src/main/docker/),
# das fuer lokale Builds und den gematik-Stand unveraendert bleibt.
ARG JAVA_IMAGE=eclipse-temurin:21-jre-alpine

FROM --platform=$BUILDPLATFORM ${JAVA_IMAGE} AS extract
WORKDIR /build
COPY idp-server-*.jar idp-server.jar
RUN java -Djarmode=tools -jar idp-server.jar extract --layers --launcher --destination extracted

FROM ${JAVA_IMAGE}
ARG COMMIT_HASH
ARG VERSION
LABEL org.opencontainers.image.source="https://github.com/odilab/ref-idp-server" \
      org.opencontainers.image.revision="${COMMIT_HASH}" \
      org.opencontainers.image.version="${VERSION}"

STOPSIGNAL SIGTERM

HEALTHCHECK --interval=15s \
            --timeout=10s \
            --start-period=30s \
            --retries=3 \
  CMD ["/usr/bin/wget", "-q", "--spider", \
       "http://localhost:8180/actuator/health"]

ARG USERID=10000
ARG GROUPID=10000
USER $USERID:$GROUPID
WORKDIR /app

COPY --from=extract --chown=$USERID:$GROUPID /build/extracted/dependencies/ ./
COPY --from=extract --chown=$USERID:$GROUPID /build/extracted/spring-boot-loader/ ./
COPY --from=extract --chown=$USERID:$GROUPID /build/extracted/snapshot-dependencies/ ./
COPY --from=extract --chown=$USERID:$GROUPID /build/extracted/application/ ./

ENTRYPOINT ["java", "org.springframework.boot.loader.launch.JarLauncher"]
