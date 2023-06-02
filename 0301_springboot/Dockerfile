ARG JDBC_URL="jdbc:postgresql://host.docker.internal:5432/db?user=user&password=pass"

FROM bitnami/java:17 as build

WORKDIR /build
COPY mvnw mvnw.cmd ./
COPY .mvn/ .mvn/
COPY pom.xml ./
COPY src/ src/

ARG JDBC_URL
ENV JDBC_URL=$JDBC_URL

# RUN java -Djarmode=layertools -jar application.jar extract
# RUN $JAVA_HOME/bin/jlink \
#          --add-modules `jdeps --ignore-missing-deps -q -recursive --multi-release ${RELEASE} --print-module-deps -cp 'dependencies/BOOT-INF/lib/*':'snapshot-dependencies/BOOT-INF/lib/*' application.jar` \
#          --strip-debug \
#          --no-man-pages \
#          --no-header-files \
#          --compress=2 \
#          --output jdk
RUN chmod +x mvnw
RUN ./mvnw verify


FROM bitnami/java:17 as final

ARG JDBC_URL
ENV JDBC_URL=$JDBC_URL
ENV PATH "${JAVA_HOME}/bin:${PATH}"

WORKDIR /opt/workspace

RUN groupadd --gid 20001 spring-app \
  && useradd --uid 10001 --gid spring-app --shell /sbin/nologin spring-app

USER spring-app:spring-app
EXPOSE 8080

COPY --from=build --chown=spring-app:spring-app /build/target/app.jar ./app.jar

HEALTHCHECK --interval=1m --timeout=3s \
  CMD curl -f http://localhost:8080/actuator/health || exit 1

ENTRYPOINT ["java", "-jar", "app.jar"]