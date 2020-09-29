FROM openjdk:11-jdk

ARG MAVEN_VERSION=3.6.3
ARG USER_HOME_DIR="/root"
ARG SHA=c35a1803a6e70a126e80b2b3ae33eed961f83ed74d18fcd16909b2d44d7dada3203f1ffe726c17ef8dcca2dcaa9fca676987befeadc9b9f759967a8cb77181c0
ARG BASE_URL=https://apache.osuosl.org/maven/maven-3/${MAVEN_VERSION}/binaries

ENV TEAMCITY_PACKAGE TeamCity-2020.1.tar.gz
ENV TEAMCITY_DOWNLOAD http://download.jetbrains.com/teamcity
VOLUME /data/teamcity_agent/conf
ENV CONFIG_FILE /data/teamcity_agent/conf/buildAgent.properties

RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y  --allow-unauthenticated \
	curl \
    wget \
	git \
	sudo \
    tar
RUN sudo apt-get -y dist-upgrade --allow-unauthenticated

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
  && echo "${SHA}  /tmp/apache-maven.tar.gz" | sha512sum -c - \
  && tar -xzf /tmp/apache-maven.tar.gz -C /usr/share/maven --strip-components=1 \
  && rm -f /tmp/apache-maven.tar.gz \
  && ln -s /usr/share/maven/bin/mvn /usr/bin/mvn
ENV MAVEN_HOME /usr/share/maven
ENV M2_HOME /usr/share/maven
ENV M2=$M2_HOME/bin
ENV MAVEN_CONFIG "$USER_HOME_DIR/.m2"
RUN export PATH=$M2:$PATH
RUN curl -SL https://raw.githubusercontent.com/carlossg/docker-maven/master/openjdk-11/mvn-entrypoint.sh --output /usr/local/bin/mvn-entrypoint.sh
RUN curl -SL https://raw.githubusercontent.com/carlossg/docker-maven/master/openjdk-11/settings-docker.xml --output /usr/share/maven/ref/settings-docker.xml
RUN useradd -m buildagent \
    && chmod +x /run-agent.sh /run-services.sh && sync

CMD ["/run-services.sh"]
EXPOSE 9090