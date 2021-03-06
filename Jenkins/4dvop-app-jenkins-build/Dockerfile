FROM bitnami/minideb-extras:stretch-r493
LABEL maintainer "Bitnami <containers@bitnami.com>"

# Install required system packages and dependencies
RUN install_packages fontconfig fonts-dejavu-extra libc6 libcomerr2 libcurl3 libffi6 libgcc1 libgcrypt20 libgmp10 libgnutls30 libgpg-error0 libgssapi-krb5-2 libhogweed4 libidn11 libidn2-0 libk5crypto3 libkeyutils1 libkrb5-3 libkrb5support0 libldap-2.4-2 libnettle6 libnghttp2-14 libp11-kit0 libpsl5 librtmp1 libsasl2-2 libssh2-1 libssl1.0.2 libssl1.1 libtasn1-6 libunistring0 openssh-client unzip zlib1g
RUN bitnami-pkg install java-1.8.232-0 --checksum f4e5ff1d4fb13baf9bb2a4f2e5ffb328cf9eeb26d0b91214196e4473c2874adc
RUN bitnami-pkg install git-2.23.0-1 --checksum c473c33b37175fe4d1b4f6ec63ccfd0aea79669aab082e4c45b73e90a032568b
RUN bitnami-pkg unpack jenkins-2.190.2-0 --checksum 9d6eca153b2fc7d084d0153c4207850c2765ce059cb6b13026159e18dd2d7810
RUN mkdir -p /usr/share/jenkins/ref

# Install docker and ansible
RUN apt-get update -qq \
    && apt-get install -qqy apt-transport-https ca-certificates curl gnupg2 software-properties-common
RUN curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
RUN add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
RUN apt-get update  -qq \
    && apt-get install docker-ce=17.12.1~ce-0~debian python-pip sshpass -y
RUN pip install ansible==2.5 && pip install --force-reinstall cryptography==2.7

COPY rootfs /
RUN ln -sf /install-plugins.sh /usr/local/bin/install-plugins.sh
ENV BITNAMI_APP_NAME="jenkins" \
    BITNAMI_IMAGE_VERSION="2.190.2-debian-9-r1" \
    DISABLE_JENKINS_INITIALIZATION="no" \
    JAVA_OPTS="" \
    JENKINS_HOME="/opt/bitnami/jenkins/jenkins_home" \
    JENKINS_PASSWORD="bitnami" \
    JENKINS_USERNAME="user" \
    PATH="/opt/bitnami/java/bin:/opt/bitnami/git/bin:$PATH"

# install go
RUN apt-get install wget \
  && wget https://dl.google.com/go/go1.13.3.linux-amd64.tar.gz \
  && tar -xvf go1.13.3.linux-amd64.tar.gz \
  && mv go /usr/local \
  && rm go1.13.3.linux-amd64.tar.gz

ENV GOROOT /usr/local/go
ENV GOPATH $HOME/go 
ENV PATH $GOPATH/bin:$GOROOT/bin:$PATH

# install dep
RUN go get github.com/golang/dep \
  && cd $GOPATH/src/github.com/golang/dep \
  && go install ./...

# install clair
RUN cd $GOPATH/src/ \
  && git clone https://github.com/arminc/clair-scanner.git \
  && cd clair-scanner/ \
  && make ensure && make build

ENV CLAIR_SCANNER $GOPATH/src/clair-scanner
ENV PATH $CLAIR_SCANNER:$PATH

# Install git secret
RUN git clone https://github.com/awslabs/git-secrets.git \
    && cd git-secrets \
    && make install \
    && cp git-secrets /usr/local/bin/ \
    && cd .. \
    && rm -Rf git-secrets

EXPOSE 8080 8443 50000

ENTRYPOINT [ "/app-entrypoint.sh" ]
CMD [ "/run.sh" ]

