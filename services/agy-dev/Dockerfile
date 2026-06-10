# syntax=docker/dockerfile:1.7
FROM node:20-slim

# ── Layer 1: minimal system deps ────────────────────────────────
RUN apt-get update \
  && apt-get install -y --no-install-recommends curl ca-certificates bash expect \
  && rm -rf /var/lib/apt/lists/*

# ── Layer 2: Antigravity CLI (agy) — layer phụ thuộc network ─────
# NOTE: install.sh sets up `agy` binary at /usr/local/bin/agy (hoặc
#       /root/.local/bin/agy tuỳ phiên bản installer).
#
# ⚠️ TRƯỚC ĐÂY: `curl ... | bash || true` — `|| true` NUỐT mọi lỗi cài đặt.
#    Nếu mạng lỗi nhất thời (hay xảy ra trên Azure agent cold-start), layer
#    này vẫn "thành công" và tạo ra image KHÔNG có binary `agy`. Tệ hơn,
#    layer hỏng đó bị cache (Azure dùng local cache xoay vòng theo tuần)
#    → mọi build sau đều dùng lại image hỏng suốt cả tuần, auth luôn fail.
#
# 🔧 BÂY GIỜ: tách curl và bash thành 2 bước riêng — curl retry nếu mạng lỗi,
#    bash/install.sh dùng || true vì install.sh exit non-zero do warning PATH
#    dù cài thành công. Verify binary sau cùng mới là nguồn sự thật duy nhất.
#    Cài hỏng → build FAIL RÕ RÀNG (không tạo image lỗi ngầm, không đầu độc cache).
#
# 💡 Bump AGY_CACHE_BUST (trong .env) để force rebuild layer này và tải agy mới nhất.
ARG AGY_INSTALL_URL="https://antigravity.google/cli/install.sh"
# Bump giá trị này (qua .env hoặc --build-arg) để force tải agy version mới nhất
ARG AGY_CACHE_BUST="2026-06-10"
RUN set -eux; \
  echo "==> [agy-install] cache bust: ${AGY_CACHE_BUST}"; \
  export PATH="/root/.local/bin:/usr/local/bin:${PATH}"; \
  downloaded=0; \
  for attempt in 1 2 3; do \
  echo "==> [agy-install] attempt ${attempt}/3 from ${AGY_INSTALL_URL}"; \
  if curl -fsSL --retry 3 --retry-delay 2 --connect-timeout 15 "${AGY_INSTALL_URL}" -o /tmp/agy-install.sh; then \
  downloaded=1; \
  break; \
  fi; \
  echo "==> [agy-install] attempt ${attempt} failed, retrying..." >&2; \
  sleep 3; \
  done; \
  if [ "${downloaded}" != "1" ]; then \
  echo "❌ [agy-install] Tải install.sh thất bại sau 3 lần thử." >&2; \
  exit 1; \
  fi; \
  # install.sh có thể exit non-zero do warning PATH dù cài thành công
  # → dùng || true, verify binary bên dưới mới là nguồn sự thật
  bash /tmp/agy-install.sh || true; \
  rm -f /tmp/agy-install.sh; \
  # ── Verify binary thật sự tồn tại & chạy được (fail-loud) ──
  if ! command -v agy >/dev/null 2>&1; then \
  echo "❌ [agy-install] install.sh chạy xong nhưng KHÔNG tìm thấy binary 'agy' trong PATH." >&2; \
  echo "   PATH=${PATH}" >&2; \
  echo "   Liệt kê các vị trí thường gặp:" >&2; \
  ls -la /usr/local/bin /root/.local/bin 2>/dev/null || true; \
  exit 1; \
  fi; \
  echo "✅ [agy-install] agy resolved at: $(command -v agy)"

# ── Layer 3: wrapper script (đổi khi exec-wrapper.sh đổi) ───────
COPY exec-wrapper.sh /exec-wrapper.sh
RUN chmod +x /exec-wrapper.sh

RUN mkdir -p /root/.gemini/antigravity-cli

WORKDIR /app

# Keep container alive (stdin_open + tty are set in docker-compose)
CMD ["tail", "-f", "/dev/null"]