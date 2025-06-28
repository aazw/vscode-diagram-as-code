# debian version
# 10 (Buster)
# 11 (Bullseye)
# 12 (Bookworm) **now**
# 13 (Trixie)
# 14 (Forky)

# for main image
ARG DOCKER_DEBIAN_VERSION=12.11

# for builder images
ARG DOCKER_GOLANG_VERSION=1.24.4-bookworm
ARG DOCKER_NODE_VERSION=22.17.0-bookworm
ARG DOCKER_PYTHON_VERSION=3.13.5-bookworm
ARG DOCKER_JAVA_VERSION=21.0.7_6-jdk-jammy

# for go install
ARG GO_DOCKERFMT_VERSION=v0.3.7
ARG GO_SHFMT_VERSION=v3.11.0
ARG GO_GO_TASK_VERSION=v3.44.0

# for npm tools
ARG NPM_NPM_VERSION=11.4.2
ARG NPM_PRETTIER_VERSION=3.6.2
ARG NPM_RENOVATE_VERSION=41.15.0
ARG NPM_CLAUDE_CODE_VERSION=1.0.35
ARG NPM_CCUSAGE_VERSION=15.2.0
ARG NPM_PUPPETEER_VERSION=24.11.1
ARG NPM_MERMAID_CLI_VERSION=11.6.0

# for java tools
ARG JAVA_PLANTUML_VERSION=1.2025.4

# for other tools (github-tags)
ARG OTHER_JQ_VERSION=1.8.0
ARG OTHER_YQ_VERSION=4.45.4
ARG OTHER_XQ_VERSION=1.3.0
ARG OTHER_TAPLO_VERSION=0.10.0
ARG OTHER_SHELLCHECK_VERSION=v0.10.0
ARG OTHER_RIPGREP_VERSION=14.1.1

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

# java
# https://hub.docker.com/_/eclipse-temurin
# https://adoptium.net/temurin/
FROM eclipse-temurin:${DOCKER_JAVA_VERSION} AS java-builder
ENV JAVA_HOME=/opt/java/openjdk
RUN apt-get update && apt-get install -y --no-install-recommends binutils \
    && rm -rf /var/lib/apt/lists/*
RUN $JAVA_HOME/bin/jlink \
    --add-modules java.base,java.logging,java.desktop,java.xml,java.datatransfer \
    --strip-debug --no-man-pages --no-header-files --compress=2 \
    --output /javaruntime

# ##########################################################################################
# main image
# ##########################################################################################

# debian
# https://hub.docker.com/_/debian/
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
    gzip \
    tar \
    zip \
    unzip \
    xz-utils \
    locales-all \
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

# java
# refs above
ENV JAVA_HOME=/opt/java/openjdk
ENV PATH="$JAVA_HOME/bin:$PATH"
COPY --from=java-builder /javaruntime $JAVA_HOME

# npm
# https://github.com/npm/cli
# https://www.npmjs.com/package/npm
ARG NPM_NPM_VERSION
RUN npm install -g npm@${NPM_NPM_VERSION}

# prettier
# https://prettier.io/docs/install
ARG NPM_PRETTIER_VERSION
RUN npm install -g prettier@${NPM_PRETTIER_VERSION}

# renovate
# https://github.com/renovatebot/renovate
# https://www.npmjs.com/package/renovate
ARG NPM_RENOVATE_VERSION
RUN npm install -g renovate@${NPM_RENOVATE_VERSION}

# claude-code
# https://github.com/anthropics/claude-code
# https://github.com/anthropics/claude-code/blob/main/.devcontainer/devcontainer.json
# https://github.com/anthropics/claude-code/blob/main/.devcontainer/Dockerfile
# https://github.com/anthropics/claude-code/blob/main/.devcontainer/init-firewall.sh
RUN apt-get update -y && apt-get install -y \
    less \
    git \
    procps \
    sudo \
    fzf \
    zsh \
    man-db \
    unzip \
    gnupg2 \
    gh \
    iptables \
    ipset \
    iproute2 \
    dnsutils \
    aggregate
ARG NPM_CLAUDE_CODE_VERSION
RUN npm install -g @anthropic-ai/claude-code@${NPM_CLAUDE_CODE_VERSION}
ENV DEVCONTAINER=true

## ccusage
# https://github.com/ryoppippi/ccusage
# https://www.npmjs.com/package/ccusage
ARG NPM_CCUSAGE_VERSION
RUN npm install -g ccusage@${NPM_CCUSAGE_VERSION}

# puppeteer
# https://github.com/puppeteer/puppeteer
# https://www.npmjs.com/package/puppeteer
ARG NPM_PUPPETEER_VERSION
RUN npm install -g puppeteer@${NPM_PUPPETEER_VERSION}

# mermaid
# https://github.com/mermaid-js/mermaid-cli
ARG NPM_MERMAID_CLI_VERSION
RUN npm install -g @mermaid-js/mermaid-cli@${NPM_MERMAID_CLI_VERSION}

# jq
# https://github.com/jqlang/jq
ARG OTHER_JQ_VERSION
RUN curl -fsSL -o /jq https://github.com/jqlang/jq/releases/download/jq-${OTHER_JQ_VERSION}/jq-linux-$(uname -m | sed 's/aarch64/arm64/' | sed 's/x86_64/amd64/') \
    && chmod +x /jq \
    && mv /jq /usr/local/bin/ \
    && jq --version

# yq
# https://github.com/mikefarah/yq
# https://hub.docker.com/r/mikefarah/yq
ARG OTHER_YQ_VERSION
RUN curl -fsSL -o /yq https://github.com/mikefarah/yq/releases/download/v${OTHER_YQ_VERSION}/yq_linux_$(uname -m | sed 's/aarch64/arm64/' | sed 's/x86_64/amd64/') \
    && chmod +x /yq \
    && mv /yq /usr/local/bin/ \
    && yq --version

# xq
# https://github.com/sibprogrammer/xq
ARG OTHER_XQ_VERSION
RUN curl -fsSL -o /xq.tar.gz https://github.com/sibprogrammer/xq/releases/download/v${OTHER_XQ_VERSION}/xq_${OTHER_XQ_VERSION}_linux_$(uname -m | sed 's/aarch64/arm64/' | sed 's/x86_64/amd64/').tar.gz \
    && mkdir -p /xq \
    && tar -xvzf /xq.tar.gz -C /xq/ \
    && chmod +x /xq/xq \
    && mv /xq/xq /usr/local/bin/ \
    && rm /xq.tar.gz \
    && rm -rf /xq \
    && xq --version

# taplo
# https://github.com/tamasfe/taplo
# https://taplo.tamasfe.dev/
# https://hub.docker.com/r/tamasfe/taplo
ARG OTHER_TAPLO_VERSION
RUN curl -fsSL -o /taplo.gz https://github.com/tamasfe/taplo/releases/download/${OTHER_TAPLO_VERSION}/taplo-linux-$(uname -m | sed 's/arm64/aarch64/' | sed 's/amd64/x86_64/').gz \
    && gzip -d /taplo.gz \
    && chmod +x /taplo \
    && mv /taplo /usr/local/bin/ \
    && taplo --version

# shellcheck
# https://github.com/koalaman/shellcheck
# https://hub.docker.com/r/koalaman/shellcheck
ARG OTHER_SHELLCHECK_VERSION
RUN curl -fsSL -o /shellcheck.tar.xz https://github.com/koalaman/shellcheck/releases/download/${OTHER_SHELLCHECK_VERSION}/shellcheck-${OTHER_SHELLCHECK_VERSION}.linux.$(uname -m | sed 's/arm64/aarch64/' | sed 's/amd64/x86_64/').tar.xz \
    && tar -xvf /shellcheck.tar.xz -C / \
    && chmod +x /shellcheck-${OTHER_SHELLCHECK_VERSION}/shellcheck \
    && mv /shellcheck-${OTHER_SHELLCHECK_VERSION}/shellcheck /usr/local/bin/ \
    && rm /shellcheck.tar.xz \
    && rm -rf /shellcheck-${OTHER_SHELLCHECK_VERSION} \
    && shellcheck --version

# ripgrep
# https://github.com/BurntSushi/ripgrep
ARG OTHER_RIPGREP_VERSION
RUN mkdir -p /ripgrep \
    && curl -fsSL -o /ripgrep/ripgrep.tar.gz -L https://github.com/BurntSushi/ripgrep/releases/download/${OTHER_RIPGREP_VERSION}/ripgrep-${OTHER_RIPGREP_VERSION}-$(uname -m)-unknown-linux-gnu.tar.gz \
    && cd /ripgrep \
    && tar -xvzf ripgrep.tar.gz \
    && mv ripgrep-${OTHER_RIPGREP_VERSION}-$(uname -m)-unknown-linux-gnu/rg /usr/local/bin/ \
    && chmod +x /usr/local/bin/rg \
    && rm -rf /ripgrep

# plantuml
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

# Arm環境のDocker内で(も)Puppeteerを動かせるようにする
# https://zenn.dev/tom1111/articles/0dc7cde5c8e9bf
RUN apt-get update -y && apt-get install -y chromium
ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true
ENV PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium

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

RUN echo "vscode ALL=(ALL) NOPASSWD:ALL" >>/etc/sudoers
USER $USERNAME

# puppeteer-config.json with alias
COPY ./puppeteer-config.json /home/vscode/puppeteer-config.json
RUN echo "alias mmdc='mmdc --puppeteerConfigFile /home/vscode/puppeteer-config.json'" >>/home/vscode/.bashrc
