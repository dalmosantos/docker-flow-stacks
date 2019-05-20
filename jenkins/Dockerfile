FROM jenkins/jenkins:lts-slim

USER root

# install docker
RUN apt-get update && \
    apt-get install cpio rpm2cpio zip unzip -y && \
    curl -fsSL get.docker.com -o get-docker.sh && bash get-docker.sh && \
    apt-get clean autoclean && \
    apt-get autoremove -y && \
    rm -rf /var/lib/{apt,dpkg,cache,log}
RUN usermod -aG docker jenkins
# install kubectl
RUN curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && \
    chmod +x ./kubectl; mv ./kubectl /usr/local/bin/kubectl

USER jenkins
ENV SECRETS_DIR=/run/secrets
# Whether to skip setup wizard
ENV JAVA_OPTS="-Djenkins.install.runSetupWizard=false"
# Creates username and password specified through environment variables JENKINS_USER_SECRET and JENKINS_PASS_SECRET
COPY security.groovy /usr/share/jenkins/ref/init.groovy.d/security.groovy

# Setting the number of executors
COPY executors.groovy /usr/share/jenkins/ref/init.groovy.d/executors.groovy

# Install groovy global libraries for pipeline plugin
#COPY var/jenkins_home/org.jenkinsci.plugins.workflow.libs.GlobalLibraries.xml /usr/share/jenkins/ref/org.jenkinsci.plugins.workflow.libs.GlobalLibraries.xml

# Install a list of plugins from the file 'plugins.txt' and their dependencies
COPY plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN /usr/local/bin/install-plugins.sh < /usr/share/jenkins/ref/plugins.txt