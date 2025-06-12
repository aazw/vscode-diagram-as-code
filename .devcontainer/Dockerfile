# debian version
# 10 (Buster)
# 11 (Bullseye)
# 12 (Bookworm) **now**
# 13 (Trixie)
# 14 (Forky)

# for main image
ARG DOCKER_DEBIAN_VERSION=12.11

# for builder images
ARG DOCKER_GOLANG_VERSION=1.24
ARG DOCKER_NODE_VERSION=22.16.0-bookworm
ARG DOCKER_JQ_VERSION=1.8.0
ARG DOCKER_YQ_VERSION=4.45.4

# for go install
ARG GO_DOCKERFMT_VERSION=v0.3.7
ARG GO_SHFMT_VERSION=v3.11.0
ARG GO_GO_TASK_VERSION=v3.43.3

# for npm tools
ARG NPM_PUPPETEER_VERSION=24.10.1
ARG NPM_MERMAID_CLI_VERSION=11.4.3
ARG NPM_PRETTIER_VERSION=3.5.3
ARG NPM_RENOVATE_VERSION=40.51.0

# for java tools
ARG JAVA_PLANTUML_VERSION=1.2025.3

# ##########################################################################################
# builder images
# ##########################################################################################

# golang
# https://hub.docker.com/_/golang
FROM golang:${DOCKER_GOLANG_VERSION} AS golang-builder

# dockerfmt
# https://github.com/reteps/dockerfmt
ARG GO_DOCKERFMT_VERSION
RUN go install github.com/reteps/dockerfmt@${GO_DOCKERFMT_VERSION}

# shfmt
# https://github.com/mvdan/sh
ARG GO_SHFMT_VERSION
RUN go install mvdan.cc/sh/v3/cmd/shfmt@${GO_SHFMT_VERSION}

# go-task
# https://github.com/go-task/task
# https://taskfile.dev/
ARG GO_GO_TASK_VERSION
RUN go install github.com/go-task/task/v3/cmd/task@${GO_GO_TASK_VERSION}

# node
# https://hub.docker.com/_/node
# https://note.milldea.com/posts/two-ways-to-install-nodejs-with-fixed-version-in-dockerfile
FROM node:${DOCKER_NODE_VERSION} AS node-builder

# jq
# https://github.com/mikefarah/yq
# https://github.com/jqlang/jq
FROM ghcr.io/jqlang/jq:${DOCKER_JQ_VERSION} AS jq-builder

# yq
# https://github.com/mikefarah/yq
# https://hub.docker.com/r/mikefarah/yq
FROM mikefarah/yq:${DOCKER_YQ_VERSION} AS yq-builder

# ##########################################################################################
# main image
# ##########################################################################################

FROM debian:${DOCKER_DEBIAN_VERSION}
ENV DEBIAN_FRONTEND=noninteractive
ENV DEBCONF_NOWARNINGS=yes

# Install OS dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    wget \
    git \
    nano \
    less \
    gnupg \
    dirmngr \
    graphviz \
    libdbus-1-3 \
    libasound2 \
    libatk-bridge2.0-0 \
    libatk1.0-0 \
    libcups2 \
    libgbm-dev \
    libgbm1 \
    libgtk-3-0 \
    libnss3 \
    libpango-1.0-0 \
    libpangocairo-1.0-0 \
    libx11-6 \
    libxcomposite-dev \
    libxcomposite1 \
    libxdamage1 \
    libxkbcommon-x11-0 \
    libxrandr2 \
    libxshmfence1 \
    && rm -rf /var/lib/apt/lists/*

# dockerfmt
# refs above
COPY --from=golang-builder /go/bin/dockerfmt /usr/local/bin/

# shfmt
# refs above
COPY --from=golang-builder /go/bin/shfmt /usr/local/bin/

# go-task
# refs above
COPY --from=golang-builder /go/bin/task /usr/local/bin/

# node
# refs above
COPY --from=node-builder /usr/local/bin/node /usr/local/bin/node
COPY --from=node-builder /usr/local/lib/node_modules /usr/local/lib/node_modules
RUN ln -s /usr/local/bin/node /usr/local/bin/nodejs \
    && ln -s /usr/local/lib/node_modules/npm/bin/npm-cli.js /usr/local/bin/npm \
    && ln -s /usr/local/lib/node_modules/npm/bin/npm-cli.js /usr/local/bin/npx
RUN npm install -g yarn

# jq
# refs above
COPY --from=jq-builder /jq /usr/local/bin/

# yq
# refs above
COPY --from=yq-builder /usr/bin/yq /usr/local/bin/

# prettier
# https://prettier.io/docs/install
ARG NPM_PRETTIER_VERSION
RUN yarn global add prettier@${NPM_PRETTIER_VERSION}

# renovate
# https://github.com/renovatebot/renovate
# https://www.npmjs.com/package/renovate
ARG NPM_RENOVATE_VERSION
RUN yarn global add renovate@${NPM_RENOVATE_VERSION}

# Install Java
RUN apt-get update && apt-get install -y --no-install-recommends \
    openjdk-17-jre-headless

# add vscode user
# https://code.visualstudio.com/remote/advancedcontainers/add-nonroot-user#_creating-a-nonroot-user
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME \
    && apt-get update \
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL >/etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME
USER $USERNAME

# Switch to vscode user and install CLI tools via Yarn (so global bin is in /home/vscode/.yarn/bin)
USER vscode
WORKDIR /home/vscode

# Install Mermaid CLI and Puppeteer globally
# https://github.com/puppeteer/puppeteer
# https://www.npmjs.com/package/puppeteer
# https://github.com/mermaid-js/mermaid-cli
ARG NPM_PUPPETEER_VERSION
ARG NPM_MERMAID_CLI_VERSION
RUN yarn global add puppeteer@${NPM_PUPPETEER_VERSION} @mermaid-js/mermaid-cli@${NPM_MERMAID_CLI_VERSION}

# Switch back to root to install plantuml.jar
USER root

# Install PlantUML
# https://github.com/plantuml/plantuml
# https://github.com/plantuml/plantuml/releases/latest/download/plantuml.jar
# https://github.com/plantuml/plantuml/releases/download/v1.2025.2/plantuml-1.2025.2.jar
ARG JAVA_PLANTUML_VERSION
RUN curl -L -o /usr/local/bin/plantuml.jar \
    https://github.com/plantuml/plantuml/releases/download/v${JAVA_PLANTUML_VERSION}/plantuml-${JAVA_PLANTUML_VERSION}.jar \
    && chmod +x /usr/local/bin/plantuml.jar \
    && printf '#!/bin/bash\nexec java -Djava.awt.headless=true -jar /usr/local/bin/plantuml.jar "$@"\n' \
        >/usr/local/bin/plantuml \
    && chmod +x /usr/local/bin/plantuml

# Ensure Yarn global bin is on PATH
ENV PATH="${PATH}:/home/vscode/.yarn/bin"

# sudo権限を付与
RUN echo "vscode ALL=(ALL) NOPASSWD:ALL" >>/etc/sudoers

# Arm環境のDocker内で(も)Puppeteerを動かせるようにする
# https://zenn.dev/tom1111/articles/0dc7cde5c8e9bf
RUN apt-get update -y && apt-get install -y chromium
ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true
ENV PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium

# puppeteer-config.json with alias
COPY ./puppeteer-config.json /home/vscode/puppeteer-config.json
RUN echo "alias mmdc='mmdc --puppeteerConfigFile /home/vscode/puppeteer-config.json'" >>/home/vscode/.bashrc

# デフォルトユーザー
USER vscode
WORKDIR /workspace
