# ============================================================
# DevContainer – Ubuntu + Node.js 22 + Claude Code
# ============================================================
# ビルド:
#   docker build -t devcontainer-claude-code .
# 実行 (ホストの Claude 認証情報をマウント):
#   docker run -it -v ~/.claude:/home/vscode/.claude devcontainer-claude-code
# ============================================================

FROM ubuntu:24.04

# ── 基本パッケージ ──────────────────────────────────────────
ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    ca-certificates \
    git \
    git-lfs \
    gnupg \
    unzip \
    zip \
    jq \
    sudo \
    locales \
    tzdata \
    build-essential \
    python3 \
    python3-pip \
    && rm -rf /var/lib/apt/lists/*

# ── ロケール ────────────────────────────────────────────────
RUN locale-gen en_US.UTF-8
ENV LANG=en_US.UTF-8 \
    LANGUAGE=en_US:en \
    LC_ALL=en_US.UTF-8

# ── Node.js 22 (LTS) ────────────────────────────────────────
RUN curl -fsSL https://deb.nodesource.com/setup_22.x | bash - \
    && apt-get install -y --no-install-recommends nodejs \
    && rm -rf /var/lib/apt/lists/*

# ── 開発ユーザー作成 ────────────────────────────────────────
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=1000

RUN if getent group ${USER_GID} > /dev/null 2>&1; then \
        groupmod -n ${USERNAME} "$(getent group ${USER_GID} | cut -d: -f1)"; \
    else \
        groupadd --gid ${USER_GID} ${USERNAME}; \
    fi \
    && if getent passwd ${USER_UID} > /dev/null 2>&1; then \
        usermod -l ${USERNAME} -d /home/${USERNAME} -m \
            "$(getent passwd ${USER_UID} | cut -d: -f1)"; \
    else \
        useradd --uid ${USER_UID} --gid ${USER_GID} -m -s /bin/bash ${USERNAME}; \
    fi \
    && usermod -g ${USER_GID} -s /bin/bash ${USERNAME} \
    && echo "${USERNAME} ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/${USERNAME} \
    && chmod 0440 /etc/sudoers.d/${USERNAME}

# ── Claude Code インストール ─────────────────────────────────
# WORKDIR を /tmp に設定してからインストールすることで
# ルートからのファイルシステムスキャンによる OOM / ハングを防止
WORKDIR /tmp
RUN npm install -g @anthropic-ai/claude-code \
    && claude --version

# ── 作業ディレクトリ ────────────────────────────────────────
WORKDIR /workspace

# ── ユーザー切り替え ────────────────────────────────────────
USER ${USERNAME}

# .claude ディレクトリを事前作成（ボリュームマウント用）
RUN mkdir -p /home/${USERNAME}/.claude

VOLUME ["/home/vscode/.claude"]

# ── デフォルトシェル ────────────────────────────────────────
CMD ["/bin/bash"]

