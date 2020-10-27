FROM jenkins/jenkins:lts-slim

USER root

ARG DEBIAN_FRONTEND=noninteractive
# Install Essentials
RUN mkdir -p /usr/share/man/man1 && \
    apt-get update ; apt-get upgrade -y && \
# Install Packages
    apt-get install --no-install-recommends -y cpio  groff less rpm2cpio jq zip unzip vim-tiny sshpass git && \
# Install maven
    mkdir -p /usr/share/maven /usr/share/maven/ref && \
    curl https://apache.osuosl.org/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz -o /tmp/apache-maven.tar.gz && \
    tar -xzf /tmp/apache-maven.tar.gz -C /usr/share/maven --strip-components=1 && \
    rm -f /tmp/apache-maven.tar.gz && \
    ln -s /usr/share/maven/bin/mvn /usr/bin/mvn && \
#Install AWS CLI
    cd /tmp && \
    curl https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip -o awscliv2.zip && \
    unzip awscliv2.zip && \
    ./aws/install && \
# Install kubectl
    curl https://storage.googleapis.com/kubernetes-release/release/v1.19.0/bin/linux/amd64/kubectl -o /usr/local/bin/kubectl && \
    chmod +x /usr/local/bin/kubectl && \
# Install terraform
    curl https://releases.hashicorp.com/terraform/0.13.4/terraform_0.13.4_linux_amd64.zip -o terraform_0.13.4_linux_amd64.zip && \
    unzip -j terraform_0.13.4_linux_amd64.zip -d /usr/local/bin/ && \
#Clean
    apt-get clean autoclean && \
    apt-get autoremove -y && \
    rm -rf /tmp/* \
           /var/lib/{apt,dpkg,cache,log} \
           /var/lib/apt/lists/* \
           /var/tmp/* \
           /var/log/*log \
           /usr/share/doc/*
USER jenkins
WORKDIR /var/jenkins
ENV SECRETS_DIR=/run/secrets
# Whether to skip setup wizard
ENV JAVA_OPTS="-Djenkins.install.runSetupWizard=false"
# Creates username and password specified through environment variables JENKINS_USER_SECRET and JENKINS_PASS_SECRET
COPY security.groovy /usr/share/jenkins/ref/init.groovy.d/security.groovy

# Setting the number of executors
#COPY executors.groovy /usr/share/jenkins/ref/init.groovy.d/executors.groovy

# Install groovy global libraries for pipeline plugin
#COPY var/jenkins_home/org.jenkinsci.plugins.workflow.libs.GlobalLibraries.xml /usr/share/jenkins/ref/org.jenkinsci.plugins.workflow.libs.GlobalLibraries.xml

# Install a list of plugins from the file 'plugins.txt' and their dependencies
COPY plugins-1.txt /usr/share/jenkins/ref/plugins-1.txt
RUN /usr/local/bin/install-plugins.sh < /usr/share/jenkins/ref/plugins-1.txt