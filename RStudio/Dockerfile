FROM rocker/rstudio:latest

MAINTAINER Brandon Barker version 0.0.1

RUN apt-get update && apt-get install -y procps openssh-client openssh-server && \
    apt-get clean && apt-get purge && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    mkdir /var/run/sshd

CMD ["/init"]
