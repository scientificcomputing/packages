FROM ubuntu:22.04

RUN ln -snf /usr/share/zoneinfo/Etc/UTC /etc/localtime \
    && echo "Etc/UTC" > /etc/timezone \
    && apt-get update \
    && apt-get upgrade -y \
    && apt-get install -y texlive-full \
    xzdec \
    make \
    git \
    python3 \
    bibtool \ 
    xzdec \
    wget && \ 
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
