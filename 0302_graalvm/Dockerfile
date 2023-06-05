FROM ghcr.io/graalvm/graalvm-ce:22.3.1 as build

WORKDIR /build

ARG MAVEN_VERSION=3.9.2
ARG USER_HOME_DIR="/root"
ARG BASE_URL=https://apache.osuosl.org/maven/maven-3/${MAVEN_VERSION}/binaries

COPY . .

RUN mkdir -p /usr/share/maven /usr/share/maven/ref \
 && curl -fsSL -o /tmp/apache-maven.tar.gz ${BASE_URL}/apache-maven-${MAVEN_VERSION}-bin.tar.gz \
 && tar -xzf /tmp/apache-maven.tar.gz -C /usr/share/maven --strip-components=1 \
 && rm -f /tmp/apache-maven.tar.gz \
 && ln -s /usr/share/maven/bin/mvn /usr/bin/mvn

ENV MAVEN_HOME /usr/share/maven
RUN gu install native-image

RUN mvn -Pnative verify

RUN groupadd --gid 20001 java-app \
  && useradd --uid 10001 --gid java-app --shell /sbin/nologin java-app

RUN chmod +x target/native.bin
RUN file target/native.bin
# RUN ldd target/native.bin

FROM scratch as final

WORKDIR /opt/workspace

COPY --from=build /etc/group /etc/group
COPY --from=build /etc/passwd /etc/passwd
COPY --from=build /lib64/libz.so.1 /lib64/libc.so.6 /lib64/ld-linux-x86-64.so.2 /lib64/
COPY --from=build --chown=java-app:java-app /build/target/native.bin ./native.bin

USER java-app:java-app
ENV LD_LIBRARY_PATH=/lib64/
ENV PORT=17
EXPOSE $PORT

ENTRYPOINT ["/opt/workspace/native.bin"]