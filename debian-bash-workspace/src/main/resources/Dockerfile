FROM debian:latest as builder

RUN apt-get update \
 && apt-get install -y --no-install-recommends \
     binutils-dev ninja-build libiberty-dev python3 libcurl4-openssl-dev libelf-dev libdw-dev cmake gcc g++ zlib1g-dev wget \
 && apt-get -y clean \
 && rm -rf /var/lib/apt/lists/*

WORKDIR /root

ENV BATS=https://github.com/bats-core/bats-core/archive/refs/tags/v1.4.1.tar.gz
RUN wget -q ${BATS} --no-check-certificate -O - | tar xz -C ./ --strip-components 1
RUN ./install.sh /usr/local

ENV BATS_ASSERT=https://github.com/bats-core/bats-assert/archive/refs/tags/v2.0.0.tar.gz
RUN wget -q ${BATS_ASSERT} --no-check-certificate -O - | tar xz -C ./ --strip-components 1 \
 && mv ./src/assert.bash /opt/bats-assert.bash \
 && chmod 644 /opt/bats-assert.bash

ENV KCOV=https://github.com/SimonKagstrom/kcov/archive/v38.tar.gz
RUN wget -q ${KCOV} --no-check-certificate -O - | tar xz -C ./ --strip-components 1
RUN mkdir build \
 && cd build \
 && cmake -G Ninja .. \
 && cmake --build . --target install

FROM debian:latest

RUN apt-get update \
 && apt-get install -y --no-install-recommends \
    python3 binutils-dev libelf-dev libdw-dev libcurl4-openssl-dev \
 && apt-get -y clean \
 && rm -rf /var/lib/apt/lists/*

RUN mkdir -p /usr/local/libexec/bats-core/ \
 && mkdir -p /usr/local/lib/bats-core/

COPY --from=builder /usr/local/bin/bats /usr/local/bin/
COPY --from=builder /usr/local/libexec/bats-core/* /usr/local/libexec/bats-core/
COPY --from=builder /usr/local/lib/bats-core/* /usr/local/lib/bats-core/
COPY --from=builder /usr/local/share/man/man*  /usr/local/share/man/

COPY --from=builder /opt/*  /opt/

COPY --from=builder /usr/local/bin/kcov* /usr/local/bin/
COPY --from=builder /usr/local/share/doc/kcov /usr/local/share/doc/kcov

# 以降からPJ毎に変更要
RUN addgroup --gid 1000 appgroup \
 && useradd -m --uid 1000 --groups appgroup appuser

RUN apt-get update \
 && apt-get install -y --no-install-recommends postgresql-client-13 \
 && apt-get -y clean \
 && rm -rf /var/lib/apt/lists/*

ENV APP_HOME=/srv/local/app
ENV COMMON_BASH_DIR=${APP_HOME}/bash/common
ENV TEST_DIR=/srv/local/app/bash/test
ENV TEST_WORK_DIR=${TEST_DIR}/work
RUN mkdir -p ${COMMON_BASH_DIR} \
 && mkdir -p ${TEST_DIR}/common/ \
 && mkdir -p ${TEST_WORK_DIR} \
 && chown -R appuser:appgroup ${APP_HOME}

USER appuser
