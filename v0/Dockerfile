# Description: This dockerfile is used to build nezha dashboard v0 with the new geoip database to fix the correct national flag display.
#
# Copyright (c) 2025 honeok <i@honeok.com>
#
# References:
# https://github.com/naiba/nezha
#
# SPDX-License-Identifier: Apache-2.0

ARG NEZHA_VERSION=""

FROM golang:1.21-alpine AS builder
LABEL maintainer="honeok <i@honeok.com>"
ARG NEZHA_VERSION=""
WORKDIR /go/src/github.com/naiba/nezha
ENV CGO_ENABLED=1
ENV CGO_CFLAGS="-D_LARGEFILE64_SOURCE=1"
# 1. 安装依赖 & 抓取源码
RUN set -ex \
    && apk add --update --no-cache git gcc musl-dev \
    && git clone --depth=1 --branch ${NEZHA_VERSION} https://github.com/nezhahq/nezha.git .
# 2. 复制 geoip
COPY geoip.db /go/src/github.com/naiba/nezha/pkg/geoip/geoip.db

# 3. 统一替换资源为今日头条源
RUN set -ex && \
    # 3.1 预处理：定义目标前缀，确保包含 /cdn/
    # 注意：这里直接匹配到 expire 这一层，将其整体替换为 pstatp 的 cdn 路径
    TARGET_CDN="https://s3.pstatp.com/cdn" && \
    ORIGIN_CDN="https://lf6-cdn-tos\.bytecdntp\.com/cdn/expire-1-[ym]" && \
    \
    # 批量替换所有模板文件中的前缀
    find resource/template/theme-angel-kanade/ -type f -name "*.html" | xargs sed -i "s|${ORIGIN_CDN}|${TARGET_CDN}|g"

# 4. 编译二进制
RUN set -ex \
    && go mod tidy -v \
    && go build -v -trimpath -ldflags="\
    -s -w -buildid= \
    -X 'github.com/naiba/nezha/service/singleton.Version=${NEZHA_VERSION#v}' \
    -extldflags '-static -fpic' \
    " -o /go/bin/dashboard ./cmd/dashboard
# 5. 最终镜像
FROM ghcr.io/naiba/nezha-dashboard:${NEZHA_VERSION} AS dist
LABEL maintainer="honeok <i@honeok.com>"
COPY --from=builder /go/bin/dashboard /dashboard/app
