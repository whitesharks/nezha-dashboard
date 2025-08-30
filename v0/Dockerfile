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
# 3. 统一替换所有失效的 lf6-cdn-tos 资源（国内可访问）
RUN set -ex && \
    # 3.1 home.html 国旗 CDN
    sed -i 's|https://lf6-cdn-tos\.bytecdntp\.com/cdn/expire-1-y/flag-icon-css/4\.1\.5/flags/1x1/|https://cdn.staticfile.org/flag-icon-css/4.1.5/flags/1x1/|g' \
        resource/template/theme-angel-kanade/home.html && \
    # 3.2 header/footer 通用前缀替换
    sed -i 's|https://lf6-cdn-tos\.bytecdntp\.com/cdn/expire-1-y|https://cdn.bootcdn.net/ajax/libs|g' \
        resource/template/theme-angel-kanade/header.html \
        resource/template/theme-angel-kanade/footer.html && \
    # 3.3 版本号纠正（font-logos 0.17）
    sed -i 's|cdn\.bootcdn\.net/ajax/libs/font-logos/0\.19|cdn.bootcdn.net/ajax/libs/font-logos/0.17|g' \
        resource/template/theme-angel-kanade/header.html
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
