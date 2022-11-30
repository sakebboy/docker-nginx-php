FROM nginx:latest

ARG NODE_VERSION=16

RUN apt-get update \
	&& apt-get install -y vim
	
RUN apt-get install -y curl

RUN curl -sL https://deb.nodesource.com/setup_$NODE_VERSION.x | bash 
RUN apt-get install -y nodejs 
# RUN apt-get install npm

WORKDIR /var/www


