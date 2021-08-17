FROM bash:latest as builder

RUN apk add --no-cache build-base cmake ninja python3 \
      binutils-dev curl-dev elfutils-dev
WORKDIR /root

ENV BATS=https://github.com/bats-core/bats-core/archive/refs/tags/v1.4.1.tar.gz
RUN wget -q ${BATS} -O - | tar xz -C ./ --strip-components 1
RUN ./install.sh /usr/local

ENV BATS_ASSERT=https://github.com/bats-core/bats-assert/archive/refs/tags/v2.0.0.tar.gz
RUN wget -q ${BATS_ASSERT} -O - | tar xz -C ./ --strip-components 1 \
 && mv ./src/assert.bash /opt/bats-assert.bash \
 && chmod 644 /opt/bats-assert.bash

ENV KCOV=https://github.com/SimonKagstrom/kcov/archive/v38.tar.gz
RUN wget -q ${KCOV} -O - | tar xz -C ./ --strip-components 1
RUN mkdir build && cd build \
 && CXXFLAGS="-D__ptrace_request=int" cmake -G Ninja .. \
 && cmake --build . --target install

FROM bash:latest

RUN apk add --no-cache bash python3 binutils-dev curl-dev elfutils-libelf \
    && rm -rf /var/cache/apk/*

COPY --from=builder /usr/local/bin/bats /usr/local/bin/
COPY --from=builder /usr/local/libexec/bats-core/* /usr/local/libexec/bats-core/
COPY --from=builder /usr/local/lib/bats-core/* /usr/local/lib/bats-core/
COPY --from=builder /usr/local/share/man/man*  /usr/local/share/man/

COPY --from=builder /opt/*  /opt/

COPY --from=builder /usr/local/bin/kcov* /usr/local/bin/
COPY --from=builder /usr/local/share/doc/kcov /usr/local/share/doc/kcov

# batsの最新バージョンを使用するためapkではなくソースから行う
#RUN apk add --no-cache bats=1.3.0-r0 --repository=http://dl-cdn.alpinelinux.org/alpine/edge/main \
# && rm -rf /var/cache/apk/*

# 以降からPJ毎に変更要
RUN addgroup -S appgroup \
 && adduser -S appuser -G appgroup

RUN apk add --no-cache postgresql-client=13.4-r0 --repository=http://dl-cdn.alpinelinux.org/alpine/edge/main \
 && rm -rf /var/cache/apk/*
ENV APP_HOME=/srv/local/app
ENV COMMON_BASH_DIR=${APP_HOME}/bash/common
ENV TEST_DIR=/srv/local/app/bash/test
ENV TEST_WORK_DIR=${TEST_DIR}/work
RUN mkdir -p ${COMMON_BASH_DIR} \
 && mkdir -p ${TEST_DIR}/common/ \
 && mkdir -p ${TEST_WORK_DIR} \
 && chown -R appuser:appgroup ${APP_HOME}

USER appuser
