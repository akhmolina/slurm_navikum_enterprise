FROM bellsoft/alpaquita-linux-gcc:12.2-glibc@sha256:f1500fc38a5af3619330c7251e72a478fb676a72cfebecbbe12b2fe5ce7c893c as verify

ARG GPG_KEY_URL=https://raw.githubusercontent.com/slurmorg/build-containers-trusted/main/key.gpg
ARG GPG_KEY_FPR=70092656FB28DBB76C3BB42E89619023B6601234

ARG ROOTFS_URL=https://github.com/slurmorg/build-containers-trusted/raw/main/rootfs.tar.gz
ARG ROOTFS_SHA=https://raw.githubusercontent.com/slurmorg/build-containers-trusted/main/rootfs.tar.gz.sha512
ARG ROOTFS_SHA_ASC=https://raw.githubusercontent.com/slurmorg/build-containers-trusted/main/rootfs.tar.gz.sha512.asc

ARG MAVEN_URL=https://github.com/slurmorg/build-containers-trusted/raw/main/apache-maven-3.9.1-bin.tar.gz
ARG MAVEN_SHA=https://raw.githubusercontent.com/slurmorg/build-containers-trusted/main/apache-maven-3.9.1-bin.tar.gz.sha512
ARG MAVEN_SHA_ASC=https://raw.githubusercontent.com/slurmorg/build-containers-trusted/main/apache-maven-3.9.1-bin.tar.gz.sha512.asc

ARG TOMCAT_URL=https://github.com/slurmorg/build-containers-trusted/raw/main/apache-tomcat-10.1.7.tar.gz
ARG TOMCAT_SHA=https://raw.githubusercontent.com/slurmorg/build-containers-trusted/main/apache-tomcat-10.1.7.tar.gz.sha512
ARG TOMCAT_SHA_ASC=https://raw.githubusercontent.com/slurmorg/build-containers-trusted/main/apache-tomcat-10.1.7.tar.gz.sha512.asc

RUN apk update && \
    apk add --no-cache gnupg tar

ADD ${GPG_KEY_URL} ./key.gpg
RUN gpg --dry-run --import-options show-only --import key.gpg | grep -F ${GPG_KEY_FPR} && \
    gpg --import key.gpg

ADD ${ROOTFS_URL} rootfs.tar.gz
ADD ${ROOTFS_SHA} rootfs.tar.gz.sha512
ADD ${ROOTFS_SHA_ASC} rootfs.tar.gz.sha512.asc
ADD ${MAVEN_URL} apache-maven-3.9.1-bin.tar.gz
ADD ${MAVEN_SHA} apache-maven-3.9.1-bin.tar.gz.sha512
ADD ${MAVEN_SHA_ASC} apache-maven-3.9.1-bin.tar.gz.sha512.asc
ADD ${TOMCAT_URL} apache-tomcat-10.1.7.tar.gz
ADD ${TOMCAT_SHA} apache-tomcat-10.1.7.tar.gz.sha512
ADD ${TOMCAT_SHA_ASC} apache-tomcat-10.1.7.tar.gz.sha512.asc

RUN gpg --verify rootfs.tar.gz.sha512.asc rootfs.tar.gz.sha512 && \
    gpg --verify apache-maven-3.9.1-bin.tar.gz.sha512.asc apache-maven-3.9.1-bin.tar.gz.sha512 && \
    gpg --verify apache-tomcat-10.1.7.tar.gz.sha512.asc apache-tomcat-10.1.7.tar.gz.sha512
RUN sha512sum -cs ./rootfs.tar.gz.sha512 ./apache-maven-3.9.1-bin.tar.gz.sha512 ./apache-tomcat-10.1.7.tar.gz.sha512

RUN mkdir -p /opt/bin/rootfs/ && tar -xvzf rootfs.tar.gz -C /opt/bin/rootfs/
RUN tar -xvzf apache-maven-3.9.1-bin.tar.gz -C /opt/bin/
RUN tar -xvzf apache-tomcat-10.1.7.tar.gz -C /opt/bin/ && \
    rm -rf /opt/bin/apache-tomcat-10.1.7/webapps/*

RUN addgroup --gid 20001 tomcat \
  && adduser -u 10001 -G tomcat --system --disabled-password -s /bin/sh tomcat

FROM scratch as build

ENV PATH=/usr/lib/jvm/jdk-17.0.6-bellsoft-x86_64/bin:/opt/bin/maven/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
ENV LANG=en_US.UTF-8
ENV LANGUAGE=en_US.UTF-8:en
ENV JAVA_HOME=/usr/lib/jvm/jdk-17.0.6-bellsoft-x86_64
ENV MAVEN_HOME=/opt/bin/maven
ENV MAVEN_CONFIG "/opt/home/.m2"

COPY --from=verify /opt/bin/rootfs/ /
COPY --from=verify /opt/bin/apache-maven-3.9.1 /opt/bin/maven

WORKDIR /opt

COPY pom.xml ./
COPY src/ src/

RUN ["mvn", "verify"]

FROM scratch as final

ENV PATH=/usr/lib/jvm/jdk-17.0.6-bellsoft-x86_64/bin:/opt/bin/tomcat/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
ENV LANG=en_US.UTF-8
ENV LANGUAGE=en_US.UTF-8:en
ENV JAVA_HOME=/usr/lib/jvm/jdk-17.0.6-bellsoft-x86_64
ENV CATALINA_HOME=/opt/bin/tomcat

COPY --from=verify /opt/bin/rootfs/ /
COPY --from=verify /opt/bin/apache-tomcat-10.1.7 /opt/bin/tomcat

COPY --from=verify /etc/group /etc/group
COPY --from=verify /etc/passwd /etc/passwd
COPY --from=build /opt/target/api.war $CATALINA_HOME/webapps/ROOT.war

RUN chown -R tomcat:tomcat $CATALINA_HOME

USER tomcat:tomcat
EXPOSE 8080

CMD catalina.sh run