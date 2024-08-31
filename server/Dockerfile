FROM node:22-bookworm

# Install DEB dependencies and others.
RUN \
    set -x \
    && sudo apt-get update \
    && sudo apt-get install -y net-tools build-essential

WORKDIR /app

COPY package.json .
COPY tsconfig.json .
COPY src src
RUN npm install

ENV NETTU_REDIS_HOST="host.docker.internal"
ENV NETTU_REDIS_PORT=6379
ENV MONGODB_CONNECTION_STRING=mongodb://root:rootpassword@host.docker.internal:27017
ENV MONGODB_NAME=nettu-meeting


CMD npm run start
