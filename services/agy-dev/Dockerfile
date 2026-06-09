# syntax=docker/dockerfile:1.7
FROM node:20-slim

# ── Layer 1: minimal system deps ────────────────────────────────
RUN apt-get update \
    && apt-get install -y --no-install-recommends curl ca-certificates bash expect \
    && rm -rf /var/lib/apt/lists/*

# ── Layer 2: Antigravity CLI (agy) — layer phụ thuộc network ─────
# NOTE: install.sh sets up `agy` binary at /usr/local/bin/agy
# Nên pin version nếu install.sh hỗ trợ để layer này ổn định cache.
RUN curl -fsSL https://antigravity.google/cli/install.sh | bash || true

# ── Layer 3: wrapper script (đổi khi exec-wrapper.sh đổi) ───────
COPY exec-wrapper.sh /exec-wrapper.sh
RUN chmod +x /exec-wrapper.sh

RUN mkdir -p /root/.gemini/antigravity-cli

WORKDIR /app

# Keep container alive (stdin_open + tty are set in docker-compose)
CMD ["tail", "-f", "/dev/null"]