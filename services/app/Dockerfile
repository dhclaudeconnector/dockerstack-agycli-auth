# syntax=docker/dockerfile:1.7
FROM node:20-slim

WORKDIR /app

# ── Layer 1: system deps (Docker CLI + Compose plugin) ──────────
# Chỉ rebuild khi chính dòng RUN này đổi; độc lập với source code.
RUN apt-get update \
    && apt-get install -y --no-install-recommends ca-certificates curl gnupg \
    && install -m 0755 -d /etc/apt/keyrings \
    && curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg \
    && chmod a+r /etc/apt/keyrings/docker.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian $(. /etc/os-release && echo \"$VERSION_CODENAME\") stable" > /etc/apt/sources.list.d/docker.list \
    && apt-get update \
    && apt-get install -y --no-install-recommends docker-ce-cli docker-compose-plugin \
    && rm -rf /var/lib/apt/lists/*

# ── Layer 2: node dependencies (chỉ đổi khi package*.json đổi) ──
COPY package*.json ./
RUN --mount=type=cache,target=/root/.npm \
    npm install --production --no-audit --no-fund

# ── Layer 3: source code (đổi thường xuyên nhất → để cuối) ──────
COPY src ./src
COPY public ./public

EXPOSE 3000

CMD ["node", "src/index.js"]