FROM node:8

ENV DEBIAN_FRONTEND noninteractive

RUN wget -qO - https://raw.githubusercontent.com/yarnpkg/releases/gh-pages/debian/pubkey.gpg | apt-key add -
RUN apt-get update && apt-get install -y autoconf libtool pkg-config nasm
