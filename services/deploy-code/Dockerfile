# syntax=docker/dockerfile:1.7
FROM node:20-alpine

WORKDIR /app

# ── Layer 1: system deps (apk) ──────────────────────────────────
RUN apk add --no-cache \
  bash \
  ca-certificates \
  curl \
  docker-cli \
  docker-cli-compose \
  git \
  openssh-client \
  rsync \
  tar \
  unzip

# ── Layer 2: node dependencies ──────────────────────────────────
COPY package*.json ./
RUN --mount=type=cache,target=/root/.npm \
  npm ci --omit=dev --omit=optional --no-audit --no-fund --loglevel=error --no-update-notifier

# ── Layer 3: source code ────────────────────────────────────────
COPY src ./src
COPY public ./public

ENV DOCKER_DEPLOY_CODE_PORT=53999
EXPOSE 53999

CMD ["node", "src/index.js"]