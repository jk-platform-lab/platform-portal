FROM node:22-bookworm-slim

# Python for node-gyp (used when installing some dependencies)
ENV PYTHON=/usr/bin/python3

# Build tools needed by the scaffolder's isolated-vm dependency
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get install -y --no-install-recommends python3 python3-venv g++ build-essential libsqlite3-dev && \
    rm -rf /var/lib/apt/lists/*

# mkdocs for TechDocs, in its own virtual environment, at a pinned version
RUN python3 -m venv /opt/mkdocs && \
    /opt/mkdocs/bin/pip install --no-cache-dir mkdocs-techdocs-core==1.7.1
ENV PATH="/opt/mkdocs/bin:$PATH"

# From here on, run as the unprivileged node user, not root
USER node
WORKDIR /app

# Yarn's own files
COPY --chown=node:node .yarn ./.yarn
COPY --chown=node:node .yarnrc.yml ./
COPY --chown=node:node backstage.json ./

ENV NODE_ENV=production
ENV NODE_OPTIONS="--no-node-snapshot"

# Dependencies first: this layer is reused as long as package.json files don't change
COPY --chown=node:node yarn.lock package.json packages/backend/dist/skeleton.tar.gz ./
RUN tar xzf skeleton.tar.gz && rm skeleton.tar.gz

RUN --mount=type=cache,target=/home/node/.cache/yarn,sharing=locked,uid=1000,gid=1000 \
    yarn workspaces focus --all --production && rm -rf "$(yarn cache clean)"

# org.yaml with your User entity (needed for sign-in)
COPY --chown=node:node examples ./examples

# The app itself, and only the two config files production uses
COPY --chown=node:node packages/backend/dist/bundle.tar.gz app-config.yaml app-config.production.yaml ./
RUN tar xzf bundle.tar.gz && rm bundle.tar.gz

CMD ["node", "packages/backend", "--config", "app-config.yaml", "--config", "app-config.production.yaml"]
