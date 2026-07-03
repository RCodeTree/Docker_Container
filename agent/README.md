# 容器为基安装Agent CLI

## 介绍

- **这是一个基于Docker的容器，用于安装Claude-CLI。Claude-CLI是一个用于在命令行中与Claude进行交互的工具，使用docker命令运行Dockerfile构建的镜像即可**
- **现在的沙盒机制，如无特殊要求，本地物理机环境运行某些命令，比较安全并且不会影响到物理机环境，但是使用docker容器的隔离机制，可以更好地保护物理机环境的安全，再加上数据卷挂载优势，可以在容器中运行Claude-CLI和OpenClaw-CLI，而不会影响到物理机环境，只局限于该项目目录运行**

    ``` bash
    # 前提是将 Dockerfile 放到当前目录
    docker build -t claude-cli-env .
    docker run -itd --name container-name -v $(pwd):your-workdir claude-cli-env
    ```

### 基础镜像配置
- 镜像：`node:26-slim` 
    - 镜像配置方式(任选其一)
        - [Node官方网站](https://nodejs.org/zh-cn/download/current/)
        - docker pull node:26-slim

---

## 安装 Claude-CLI 和 OpenClaw

### 编写 Dockerfile 构建镜像
1. 引入基础镜像
2. 更换镜像软件源 - 加速镜像构建
    - [清华源](https://mirrors.tuna.tsinghua.edu.cn/)(推荐)
3. 安装 Claude-CLI 和 OpenClaw-CLI 必要依赖
4. npm/pnpm/bun 拉取 @anthropic-ai/claude-code@latest
5. npm/pnpm/bun 拉取 openclaw@latest
6. 设置工作目录
7. 保持容器运行的默认指令
    - ["tail", "-f", "/dev/null"]
8. docker build -t claude-cli-env .
9. 相关命令参考文档
   - [openclaw官网参考文档](https://docs.openclaw.ai/)
   - [claude官网参考文档](https://code.claude.com/docs/)

### 运行容器(关键步骤)

#### bridge 模式 (默认网络模式) 
**在 bridge 模式下，容器会与宿主机网络隔离，通过宿主机的IP映射容器的虚拟IP和端口访问外部网络(虚拟网桥)，容器无法直接访问宿主机的网络。**
``` bash
# 显性指定网络模式为 bridge 模式
docker run -itd --name container-name -v $(pwd):your-workdir --network bridge claude-cli-env

# 隐性指定网络模式为 bridge 模式
docker run -itd --name container-name -v $(pwd):your-workdir claude-cli-env

# 显性指定网络模式为 bridge 模式，并直接接入Claude CLI，从现有安装中迁移到 DeepSeek(临时变量)
docker run -itd --name container-name \
-v $(pwd):your-workdir \
--network bridge \
-e ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic \
-e ANTHROPIC_AUTH_TOKEN=<your DeepSeek API Key> \
-e ANTHROPIC_MODEL=deepseek-v4-pro[1m] \
-e ANTHROPIC_DEFAULT_OPUS_MODEL=deepseek-v4-pro[1m] \
-e ANTHROPIC_DEFAULT_SONNET_MODEL=deepseek-v4-pro[1m] \
-e ANTHROPIC_DEFAULT_HAIKU_MODEL=deepseek-v4-flash \
-e CLAUDE_CODE_SUBAGENT_MODEL=deepseek-v4-flash \
-e CLAUDE_CODE_EFFORT_LEVEL=max \
claude-cli-env;

# 永久配置，指定使用模型和提供商(在 image 构建阶段，即docker build -t tagname .)
# 下方配置写在 Dockerfile 中
ENV ANTHROPIC_BASE_URL="https://api.deepseek.com/anthropic" \
    ANTHROPIC_AUTH_TOKEN="<your DeepSeek API Key>" \
    ANTHROPIC_MODEL="deepseek-v4-pro[1m]" \
    ANTHROPIC_DEFAULT_OPUS_MODEL="deepseek-v4-pro[1m]" \
    ANTHROPIC_DEFAULT_SONNET_MODEL="deepseek-v4-pro[1m]" \
    ANTHROPIC_DEFAULT_HAIKU_MODEL="deepseek-v4-flash" \
    CLAUDE_CODE_SUBAGENT_MODEL="deepseek-v4-flash" \
    CLAUDE_CODE_EFFORT_LEVEL="max"

```
1. `Claude-CLI`
    - 国内不能直接访问Claude
    - **使用魔法上网(Clash 或者 Clash Verge 等工具)使用工具 TUN模式 并开启系统代理**
2. `OpenClaw-CLI` 
    - 国内能直接访问OpenClaw
    - 直接通过OpenClaw-CLI即可(宿主机会直接将容器虚拟的IP映射到的宿主机IP，通过网关转发出去)
3. **备注**：由于使用 `Claude-CLI` 不能直接访问Claude，所以大多数都是用 `deepseek` 等model服务商，这里以 `deepseek`(Linux) 为例说明：
    - 所需工具：[CC Switch](https://ccswitch.io/zh/)
    - [deepseek API 开放平台](https://platform.deepseek.com/) - 充值相关费用，生成 API Key，导入到 CC Switch 中，启动 CC Switch即可食用

    ``` bash
    # Linux / Mac 用户 接入Claude CLI，从现有安装中迁移到 DeepSeek
    export ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic
    export ANTHROPIC_AUTH_TOKEN=<your DeepSeek API Key>
    export ANTHROPIC_MODEL=deepseek-v4-pro[1m]
    export ANTHROPIC_DEFAULT_OPUS_MODEL=deepseek-v4-pro[1m]
    export ANTHROPIC_DEFAULT_SONNET_MODEL=deepseek-v4-pro[1m]
    export ANTHROPIC_DEFAULT_HAIKU_MODEL=deepseek-v4-flash
    export CLAUDE_CODE_SUBAGENT_MODEL=deepseek-v4-flash
    export CLAUDE_CODE_EFFORT_LEVEL=max
    ```

### host 模式
**在 host 模式下，容器会直接访问宿主机的网络，容器的网络配置会与宿主机的网络配置保持一致。**
``` bash
# 显性指定网络模式为 host 模式
docker run -itd --name container-name -v $(pwd):your-workdir --network host claude-cli-env

# 隐性指定网络模式为 host 模式
docker run -itd --name container-name -v $(pwd):your-workdir claude-cli-env

# 显性指定网络模式为 host 模式，并直接接入Claude CLI，从现有安装中迁移到 DeepSeek
docker run -itd --name container-name \
-v $(pwd):your-workdir \
--network host \
-e ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic \
-e ANTHROPIC_AUTH_TOKEN=<your DeepSeek API Key> \
-e ANTHROPIC_MODEL=deepseek-v4-pro[1m] \
-e ANTHROPIC_DEFAULT_OPUS_MODEL=deepseek-v4-pro[1m] \
-e ANTHROPIC_DEFAULT_SONNET_MODEL=deepseek-v4-pro[1m] \
-e ANTHROPIC_DEFAULT_HAIKU_MODEL=deepseek-v4-flash \
-e CLAUDE_CODE_SUBAGENT_MODEL=deepseek-v4-flash \
-e CLAUDE_CODE_EFFORT_LEVEL=max \
claude-cli-env;
```
1. `Claude-CLI`
   - 国内不能直接访问Claude
   - **使用魔法上网(Clash 或者 Clash Verge 等工具)使用工具 TUN模式 并开启系统代理**
   - 配置容器内的魔法上网代理配置：
        - 虽然宿主机已经开启了魔法上网代理并且容器内的网络配置会与宿主机的网络配置保持一致，但是在系统代理配置上，宿主机的代理配置不等于容器内的代理配置，所以在容器内开启魔法上网代理，需要在容器内执行以下命令：
        ``` bash
        export HTTP_PROXY=http://127.0.0.1:your-proxy-port
        export HTTPS_PROXY=http://127.0.0.1:your-proxy-port
        ```
        - 或者在启动和运行容器时，通过环境变量配置魔法上网代理：
        ``` bash
        docker run -it --network host \
        -e http_proxy=http://127.0.0.1:your-proxy-port \
        -e https_proxy=http://127.0.0.1:your-proxy-port \
        -v $(pwd):/workspace \
        claude-cli-env bash
        ```
2. `OpenClaw-CLI` - 国内能直接访问OpenClaw(容器与宿主机共用网络ip和端口配置)
3. **备注**：由于使用 `Claude-CLI` 不能直接访问Claude，所以大多数都是用 `deepseek` 等model服务商，这里以 `deepseek`(Linux) 为例说明：
    - 所需工具：[CC Switch](https://ccswitch.io/zh/)
    - [deepseek API 开放平台](https://platform.deepseek.com/) - 充值相关费用，生成 API Key，导入到 CC Switch 中，启动 CC Switch即可食用

    ``` bash
    # Linux / Mac 用户 接入Claude CLI，从现有安装中迁移到 DeepSeek
    export ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic
    export ANTHROPIC_AUTH_TOKEN=<your DeepSeek API Key>
    export ANTHROPIC_MODEL=deepseek-v4-pro[1m]
    export ANTHROPIC_DEFAULT_OPUS_MODEL=deepseek-v4-pro[1m]
    export ANTHROPIC_DEFAULT_SONNET_MODEL=deepseek-v4-pro[1m]
    export ANTHROPIC_DEFAULT_HAIKU_MODEL=deepseek-v4-flash
    export CLAUDE_CODE_SUBAGENT_MODEL=deepseek-v4-flash
    export CLAUDE_CODE_EFFORT_LEVEL=max
    ```