# Heavily based off https://github.com/codecentric/springboot-maven3-centos
FROM centos/s2i-base-centos7

# Spring Boot Maven 3
# ===
EXPOSE 8080

LABEL io.k8s.description="Subatomic S2I for building and running Java applications" \
      io.k8s.display-name="Subatomic Oracle JDK with Maven 3" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="builder,java,java8,maven,maven3" \
      io.fabric8.s2i.version.jolokia="1.4.0.redhat-1" \
      io.fabric8.s2i.version.prometheus.jmx_exporter="0.10" \      
      io.openshift.s2i.scripts-url="image:///usr/local/s2i" \
      io.openshift.s2i.destination="/tmp" \      
      io.openshift.s2i.version="s2i-core-rhel7:1-16" \      
      subatomic-version="2.0" \
      author="Subatomic <https://github.com/absa-subatomic>"

RUN yum -y update && \
  yum -y install wget java-1.8.0-openjdk-devel && \

  source /etc/environment && \
  yum clean all && \
  rm -rf /var/cache/yum

ENV JAVA_HOME_BASE=/usr/lib/jvm
ENV JAVA_HOME=${JAVA_HOME_BASE}/java
RUN echo $JAVA_HOME
ENV PATH=$JAVA_HOME:$PATH

ENV MAVEN_VERSION 3.5.4
RUN curl -fsSL https://archive.apache.org/dist/maven/maven-3/$MAVEN_VERSION/binaries/apache-maven-$MAVEN_VERSION-bin.tar.gz | tar xzf - -C /usr/share \
  && mv /usr/share/apache-maven-$MAVEN_VERSION /usr/share/maven \
  && ln -s /usr/share/maven/bin/mvn /usr/bin/mvn
ENV MAVEN_HOME /usr/share/maven
ENV GLOBAL_MAVEN_SETTINGS /opt/app-root/src/.m2/
COPY settings.xml ${GLOBAL_MAVEN_SETTINGS}/settings.xml
# Thanks to https://www.lewuathe.com/maven-wrapper-home.html
ENV MAVEN_USER_HOME /tmp
# ===

# Java S2I. See https://github.com/fabric8io-images/s2i/tree/v2.3.1/java/images/rhel
# === 
# Prometheus JMX exporter agent
 RUN mkdir -p /opt/prometheus/etc \
  && curl -sSL http://central.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/0.3.1/jmx_prometheus_javaagent-0.3.1.jar \
          -o /opt/prometheus/jmx_prometheus_javaagent.jar
 RUN curl -sSL https://raw.githubusercontent.com/fabric8io-images/s2i/v2.3.1/java/images/rhel/prometheus-opts -o /opt/prometheus/prometheus-opts && \
  curl -sSL https://raw.githubusercontent.com/fabric8io-images/s2i/v2.3.1/java/images/rhel/prometheus-config.yml -o /opt/prometheus/prometheus-config.yml
 RUN chmod 444 /opt/prometheus/jmx_prometheus_javaagent.jar \
  && chmod 444 /opt/prometheus/prometheus-config.yml \
  && chmod 755 /opt/prometheus/prometheus-opts \
  && chmod 775 /opt/prometheus/etc \
  && chgrp root /opt/prometheus/etc

EXPOSE 9779

# Jolokia agent
RUN mkdir -p /opt/jolokia/etc \
 && curl -sSL https://maven.repository.redhat.com/ga/org/jolokia/jolokia-jvm/1.5.0.redhat-1/jolokia-jvm-1.5.0.redhat-1-agent.jar \
         -o /opt/jolokia/jolokia.jar
RUN curl -sSL https://raw.githubusercontent.com/fabric8io-images/s2i/v2.3.1/java/images/rhel/jolokia-opts -o /opt/jolokia/jolokia-opts
RUN chmod 444 /opt/jolokia/jolokia.jar \
 && chmod 755 /opt/jolokia/jolokia-opts \
 && chmod 775 /opt/jolokia/etc \
 && chgrp root /opt/jolokia/etc

 # S2I scripts + README
COPY s2i /usr/local/s2i
RUN chmod 755 /usr/local/s2i/*
ADD README.md /usr/local/s2i/usage.txt

# Add run script as /opt/run-java/run-java.sh and make it executable
COPY run-java.sh /deployments/
RUN chmod 755 /deployments/run-java.sh

# Necessary to permit running with a randomised UID
RUN mkdir -p /deployments \ 
 && chmod -R "g+rwX" /deployments

# Use the run script as default since we are working as an hybrid image which can be
# used directly to. (If we were a plain s2i image we would print the usage info here)
CMD [ "/usr/local/s2i/run" ]
# ===

# Subatomic
# ===
ADD https://raw.githubusercontent.com/absa-subatomic/local-hadron-collider/master/minishift-addons/subatomic/certs/subatomic.ca.crt /etc/ca.cert.pem
RUN keytool -importcert \
  -file /etc/ca.cert.pem \
  -alias subatomic.local \
  -keystore $JAVA_HOME_BASE/jre/lib/security/cacerts \
  -storepass changeit \
  -noprompt

ADD logback.xml /maven/config/logback.xml

VOLUME /tmp

USER 1001
