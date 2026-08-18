# 基于 Bitnami 的 PostgreSQL 最新镜像
FROM bitnami/postgresql:latest

# 切换到 root 用户以执行安装操作（安装完成后会切回 postgres）
USER root

# 将两个架构的 pg_idkit 压缩包复制到临时目录
COPY pg_idkit-0.4.0-pg18-gnu-x86.tar.gz   /tmp/pg_idkit-x86.tar.gz
COPY pg_idkit-0.4.0-pg18-gnu-arm64.tar.gz       /tmp/pg_idkit-arm.tar.gz

# 定义构建参数 TARGETARCH（由 docker build --platform 自动传入）
ARG TARGETARCH

# 根据架构选择对应的压缩包，解压并安装扩展文件
RUN set -eux; \
    # 确定目标架构（优先使用 TARGETARCH，否则用 uname -m）
    arch="${TARGETARCH:-$(uname -m)}"; \
    case "$arch" in \
        x86_64|amd64)   pkg="/tmp/pg_idkit-x86.tar.gz" ;; \
        aarch64|arm64)  pkg="/tmp/pg_idkit-arm.tar.gz" ;; \
        *) echo "Unsupported architecture: $arch"; exit 1 ;; \
    esac; \
    # 解压到临时目录，并去除顶层目录
    mkdir -p /tmp/pg_idkit; \
    tar -xzf "$pkg" -C /tmp/pg_idkit --strip-components=1; \
    cd /tmp/pg_idkit; \
    # 获取 PostgreSQL 库文件和共享文件目录
    pg_libdir=$(pg_config --pkglibdir); \
    pg_sharedir=$(pg_config --sharedir); \
    # 复制扩展文件（.so 到 lib，.control 和 .sql 到 share/extension）
    cp -v lib/postgresql/*.so "$pg_libdir/"; \
    cp -v share/postgresql/extension/*.control "$pg_sharedir/extension/"; \
    cp -v share/postgresql/extension/*.sql "$pg_sharedir/extension/"; \
    # 清理临时文件
    rm -rf /tmp/pg_idkit /tmp/*.tar.gz;
    # 修正目录权限，确保 postgres 用户可访问
    # chown -R postgres:postgres "$pg_libdir" "$pg_sharedir"

# 切换回 postgres 用户（Bitnami 镜像的默认用户）
# USER postgres

# （可选）在容器首次启动时自动创建 pg_idkit 扩展
# 将包含 "CREATE EXTENSION IF NOT EXISTS pg_idkit;" 的 SQL 文件放入初始化目录
# 注意：需在构建目录中准备 create-extension.sql 文件并取消下一行注释
# COPY create-extension.sql /docker-entrypoint-initdb.d/