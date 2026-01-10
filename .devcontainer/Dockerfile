FROM jetpackio/devbox:latest AS builder

WORKDIR /code/vscode-diagram-as-code
USER root:root
RUN mkdir -p /code/vscode-diagram-as-code && chown ${DEVBOX_USER}:${DEVBOX_USER} /code/vscode-diagram-as-code
USER ${DEVBOX_USER}:${DEVBOX_USER}

COPY --chown=${DEVBOX_USER}:${DEVBOX_USER} devbox.json devbox.json
COPY --chown=${DEVBOX_USER}:${DEVBOX_USER} devbox.lock devbox.lock
RUN devbox install


FROM aazw/devcontainers-devbox-java:5 AS runtime

USER root
COPY --from=builder /nix/store /nix/store
COPY --from=builder /code/vscode-diagram-as-code/.devbox /code/vscode-diagram-as-code/.devbox
USER vscode

ENV PATH="/code/vscode-diagram-as-code/.devbox/nix/profile/default/bin:${PATH}"
