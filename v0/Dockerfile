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
# 3. 统一替换所有失效的 lf6-cdn-tos 资源（改为国内可访问的 Staticfile/BootCDN 组合）
# 3.1 替换 home.html 的国旗 CDN
RUN sed -i 's|https://lf6-cdn-tos\.bytecdntp\.com/cdn/expire-1-y/flag-icon-css/4\.1\.5/flags/1x1/|https://cdn.staticfile.org/flag-icon-css/4.1.5/flags/1x1/|g' \
        resource/template/theme-angel-kanade/home.html && \
# 3.2 替换 header/footer 中的 CSS/JS CDN
    sed -i 's|https://lf6-cdn-tos\.bytecdntp\.com/cdn/expire-1-y|https://cdn.staticfile.org|g' \
        resource/template/theme-angel-kanade/header.html \
        resource/template/theme-angel-kanade/footer.html && \
# 3.3 Semantic-UI / font-logos 在 Staticfile 没有 2.4.1/0.17，切到 BootCDN
    sed -i 's|cdn\.staticfile\.org/semantic-ui/2\.4\.1|cdn.bootcdn.net/ajax/libs/semantic-ui/2.4.1|g' \
        resource/template/theme-angel-kanade/header.html \
        resource/template/theme-angel-kanade/footer.html && \
    sed -i 's|cdn\.staticfile\.org/font-logos/0\.17|cdn.bootcdn.net/ajax/libs/font-logos/0.19|g' \
        resource/template/theme-angel-kanade/header.html && \
# 3.4 Bootstrap-Icons 切到 BootCDN
    sed -i 's|cdn\.staticfile\.org/bootstrap-icons/1\.8\.1|cdn.bootcdn.net/ajax/libs/bootstrap-icons/1.8.1|g' \
        resource/template/theme-angel-kanade/header.html && \
# 3.5 jQuery / Vue / Semantic-JS 切到 BootCDN
    sed -i 's|cdn\.staticfile\.org/jquery/3\.6\.0|cdn.bootcdn.net/ajax/libs/jquery/3.6.0|g' \
        resource/template/theme-angel-kanade/footer.html && \
    sed -i 's|cdn\.staticfile\.org/semantic-ui/2\.4\.1/semantic\.min\.js|cdn.bootcdn.net/ajax/libs/semantic-ui/2.4.1/semantic.min.js|g' \
        resource/template/theme-angel-kanade/footer.html && \
    sed -i 's|cdn\.staticfile\.org/vue/2\.6\.14|cdn.bootcdn.net/ajax/libs/vue/2.6.14|g' \
        resource/template/theme-angel-kanade/footer.html
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
