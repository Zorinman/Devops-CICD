FROM jenkins/jenkins:2.504-jdk17 
USER root
RUN apt-get update && \
    apt-get install -y unzip && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
COPY apache-maven-3.9.9-bin.tar.gz /usr/local/
COPY sonar-scanner-cli-4.8.0.2856-linux.zip /usr/local/
WORKDIR /usr/local/
RUN tar -xzf apache-maven-3.9.9-bin.tar.gz && \
    rm apache-maven-3.9.9-bin.tar.gz
RUN unzip sonar-scanner-cli-4.8.0.2856-linux.zip && \
    rm sonar-scanner-cli-4.8.0.2856-linux.zip
ENV MAVEN_HOME=/usr/local/apache-maven-3.9.9
ENV SONAR_SCANNER_HOME=/usr/local/sonar-scanner-cli-4.8.0.2856
ENV PATH=$JAVA_HOME/bin:$MAVEN_HOME/bin:$SONAR_SCANNER_HOME/bin:$PATH
RUN echo "jenkins ALL=NOPASSWD: ALL" >> /etc/sudoers
USER jenkins


