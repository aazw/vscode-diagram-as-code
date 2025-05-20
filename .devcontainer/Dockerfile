# for main image
ARG DOCKER_IMAGE_UBUNTU_VERSION=24.04

# for builder images
ARG DOCKER_GOLANG_VERSION=1.24

# for go install
ARG GO_DOCKERFMT_VERSION=v0.3.7
ARG GO_SHFMT_VERSION=v3.11.0

# for npm tools
ARG NPM_PUPPETEER_VERSION=24.9.0
ARG NPM_MERMAID_CLI_VERSION=11.4.2
ARG NPM_PRETTIER_VERSION=3.5.3
ARG NPM_RENOVATE_VERSION=40.14.4

# for java tools
ARG JAVA_PLANTUML_VERSION=1.2025.2

# ##############################
# builder images
# ##############################

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

# ##############################
# main image
# ##############################

FROM ubuntu:${DOCKER_IMAGE_UBUNTU_VERSION}

# Install OS dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    git \
    nano \
    ca-certificates \
    gnupg \
    dirmngr \
    openjdk-17-jre-headless \
    graphviz \
    libdbus-1-3 \
    libasound2t64 \
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

# Create vscode user
RUN useradd -m -s /bin/bash vscode

# Install Node.js (LTS) via NodeSource
RUN curl -fsSL https://deb.nodesource.com/setup_lts.x | bash - \
    && apt-get update && apt-get install -y --no-install-recommends nodejs \
    && npm install -g yarn \
    && rm -rf /var/lib/apt/lists/*

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

# puppeteer-config.json with alias
COPY ./puppeteer-config.json /home/vscode/puppeteer-config.json
RUN echo "alias mmdc='mmdc --puppeteerConfigFile /home/vscode/puppeteer-config.json'" >>/home/vscode/.bashrc

# dockerfmt
# refs above
COPY --from=golang-builder /go/bin/dockerfmt /usr/local/bin/

# shfmt
# refs above
COPY --from=golang-builder /go/bin/shfmt /usr/local/bin/

# prettier
# https://prettier.io/docs/install
ARG NPM_PRETTIER_VERSION
RUN yarn global add prettier@${NPM_PRETTIER_VERSION}

# renovate
# https://github.com/renovatebot/renovate
# https://www.npmjs.com/package/renovate
ARG NPM_RENOVATE_VERSION
RUN yarn global add renovate@${NPM_RENOVATE_VERSION}

# デフォルトユーザー
USER vscode
WORKDIR /workspace