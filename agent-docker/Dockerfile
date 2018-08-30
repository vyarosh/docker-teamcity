FROM openjdk:8-jdk

ARG MAVEN_VERSION=3.5.4
ARG USER_HOME_DIR="/root"
ARG SHA=ce50b1c91364cb77efe3776f756a6d92b76d9038b0a0782f7d53acf1e997a14d
ARG BASE_URL=https://apache.osuosl.org/maven/maven-3/${MAVEN_VERSION}/binaries

ENV TEAMCITY_PACKAGE TeamCity-9.0.2.tar.gz
ENV TEAMCITY_DOWNLOAD http://download.jetbrains.com/teamcity

VOLUME /data/teamcity_agent/conf
ENV CONFIG_FILE /data/teamcity_agent/conf/buildAgent.properties

RUN apt-get update
RUN apt-get -y install tar wget
RUN apt-get -y install git
RUN curl -SL https://raw.githubusercontent.com/JetBrains/teamcity-docker-minimal-agent/master/run-agent.sh --output /run-agent.sh
RUN curl -SL https://raw.githubusercontent.com/JetBrains/teamcity-docker-minimal-agent/master/run-services.sh --output /run-services.sh

RUN mkdir -p /dist/ \
    && cd /dist/ \
    && wget $TEAMCITY_DOWNLOAD/$TEAMCITY_PACKAGE \
    && tar xvzf $TEAMCITY_PACKAGE TeamCity/buildAgent \
    && rm -f $TEAMCITY_PACKAGE \
    && cd ..

RUN cp -r /dist/TeamCity/buildAgent /opt/buildagent
RUN rm -rf /dist/
    
RUN mkdir -p /usr/share/maven /usr/share/maven/ref \
  && curl -fsSL -o /tmp/apache-maven.tar.gz ${BASE_URL}/apache-maven-${MAVEN_VERSION}-bin.tar.gz \
  && echo "${SHA}  /tmp/apache-maven.tar.gz" | sha256sum -c - \
  && tar -xzf /tmp/apache-maven.tar.gz -C /usr/share/maven --strip-components=1 \
  && rm -f /tmp/apache-maven.tar.gz \
  && ln -s /usr/share/maven/bin/mvn /usr/bin/mvn

ENV MAVEN_HOME /usr/share/maven
ENV MAVEN_CONFIG "$USER_HOME_DIR/.m2"

RUN curl -SL https://raw.githubusercontent.com/carlossg/docker-maven/f581ea002e5d067deb6213c00a4d217297cad469/jdk-8/mvn-entrypoint.sh --output /usr/local/bin/mvn-entrypoint.sh
RUN curl -SL https://raw.githubusercontent.com/carlossg/docker-maven/f581ea002e5d067deb6213c00a4d217297cad469/jdk-8/settings-docker.xml --output /usr/share/maven/ref/settings-docker.xml

RUN useradd -m buildagent \ 
    && chmod +x /run-agent.sh /run-services.sh && sync

CMD ["/run-services.sh"]

EXPOSE 9090