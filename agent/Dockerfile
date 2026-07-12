# 基础 Node.js 镜像
FROM node:26-slim

# 清除旧版源配置，并写入清华源 (DEB822 格式)
RUN rm -f /etc/apt/sources.list && \
    cat <<'EOF' > /etc/apt/sources.list.d/debian.sources
Types: deb
URIs: http://mirrors.tuna.tsinghua.edu.cn/debian
Suites: trixie trixie-updates trixie-backports
Components: main contrib non-free non-free-firmware
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg

# 以下安全更新软件源为官方源配置
Types: deb
URIs: http://security.debian.org/debian-security
Suites: trixie-security
Components: main contrib non-free non-free-firmware
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg
EOF

# 更新系统并安装必要依赖，同时切换 npm 镜像源以加速 CLI 下载
RUN apt-get update && apt-get install -y curl git vim iproute2 && \
    rm -rf /var/lib/apt/lists/* && \
    npm config set registry http://registry.npmmirror.com

# 通过 npm 全局安装所有的 CLI 工具 (合并为一层以优化镜像大小)
RUN npm install -g \
    @anthropic-ai/claude-code@latest \
    openclaw@latest \
    @mimo-ai/cli

# 加入 DeepSeek 的官方 API 模型环境变量 (使用全局 ENV 确保所有进程均可读取) --- 在镜像构建时设置，如果不使用下面的ENV设置，则在容器运行时要根据官方文档进行手动设置设置
ENV ANTHROPIC_BASE_URL="https://api.deepseek.com/anthropic" \
    ANTHROPIC_AUTH_TOKEN="sk-79431e0ca70045218c86308264c93c26" \
    ANTHROPIC_MODEL="deepseek-v4-pro[1m]" \
    ANTHROPIC_DEFAULT_OPUS_MODEL="deepseek-v4-pro[1m]" \
    ANTHROPIC_DEFAULT_SONNET_MODEL="deepseek-v4-pro[1m]" \
    ANTHROPIC_DEFAULT_HAIKU_MODEL="deepseek-v4-flash" \
    CLAUDE_CODE_SUBAGENT_MODEL="deepseek-v4-flash" \
    CLAUDE_CODE_EFFORT_LEVEL="max"

# 设置工作目录
WORKDIR /project

# 保持容器运行的默认指令
CMD ["tail", "-f", "/dev/null"]