FROM ubuntu:22.04

ARG HTTP_PROXY
ARG DEV_USERNAME
ARG USER_UID
ARG USER_GID
ARG POSTGRES_UID
ARG POSTGRES_GID

ENV DEBIAN_FRONTEND=noninteractive
ENV http_proxy=${HTTP_PROXY}
ENV https_proxy=${HTTP_PROXY}
ENV HTTP_PROXY=${HTTP_PROXY}
ENV HTTPS_PROXY=${HTTP_PROXY}
ENV ALL_PROXY=${HTTP_PROXY}
ENV CLANGD_VERSION=21.1.0
ENV CLANGD_HOME=/usr/local/lib/clangd
ENV PG_VERSION=18
ENV PG_HOME=/usr/lib/postgresql/${PG_VERSION}
ENV PGDATA=/opt/transwarp/pg_duckdb/pg_runtime
ENV PATH=${CLANGD_HOME}/bin:${PG_HOME}/bin:${PATH}
ENV TZ=Asia/Shanghai

RUN \
    # prepare apt
    sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list && \
    sed -i 's/security.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list && \
    echo 'Acquire::ForceIPv4 "true";' >> /etc/apt/apt.conf.d/1000-force-ipv4-transport && \
    echo 'Acquire::Retries "100";' >> /etc/apt/apt.conf.d/80-retries && \
    # echo 'Acquire::http::Proxy "${HTTP_PROXY}";' >> /etc/apt/apt.conf.d/99-proxy && \
    # echo 'Acquire::https::Proxy "${HTTPS_PROXY}";' >> /etc/apt/apt.conf.d/99-proxy && \
    apt-get update -y && \
    # basics
    PKGS="software-properties-common" && \
    apt-get install -y ${PKGS} && \
    add-apt-repository -y ppa:ubuntu-toolchain-r/ppa && \
    apt-get update -y && \
    PKGS="vim sudo locales git curl unzip wget zip iputils-ping iproute2 tzdata tree jq bear" && \
    PKGS="${PKGS} gdb net-tools telnet cmake ninja-build ccache pkg-config linux-tools-common linux-tools-generic" && \
    apt-get install -y ${PKGS} && \
    locale-gen en_US.UTF-8 && \
    echo ${TZ} > /etc/timezone && \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    dpkg-reconfigure tzdata && \
    # duckdb deps
    PKGS="build-essential libreadline-dev zlib1g-dev flex bison libxml2-dev \
    libxslt-dev libssl-dev libxml2-utils xsltproc pkg-config libc++-dev \
    libc++abi-dev libglib2.0-dev libtinfo6 cmake libstdc++-12-dev \
    liblz4-dev ninja-build g++ liblz4-dev libcurl4-openssl-dev" \
    apt-get install -y ${PKGS} && \
    # pg runtime
    apt-get install -y curl ca-certificates && \
    install -d /usr/share/postgresql-common/pgdg && \
    curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc && \
    bash -c "echo 'deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt jammy-pgdg main' > /etc/apt/sources.list.d/pgdg.list" && \
    apt-get update -y && \
    apt install -y gosu postgresql-${PG_VERSION} postgresql-server-dev-${PG_VERSION} && \
    apt-get autoclean

RUN \
    # clangd
    wget "https://github.com/clangd/clangd/releases/download/${CLANGD_VERSION}/clangd-linux-${CLANGD_VERSION}.zip" -O /tmp/clangd.zip && \
    unzip /tmp/clangd.zip -d ${CLANGD_HOME} && \
    mv ${CLANGD_HOME}/clangd_${CLANGD_VERSION}/* ${CLANGD_HOME} && \
    rm -rf ${CLANGD_HOME}/clangd_${CLANGD_VERSION} && \
    rm -rf /tmp/clangd.zip

RUN \
    # proxy for wget
    echo "use_proxy = on" >> /etc/wgetrc && \
    echo "http_proxy = ${http_proxy}" >> /etc/wgetrc && \
    echo "https_proxy = ${http_proxy}" >> /etc/wgetrc && \
    # misc
    echo "* soft core unlimited" >>/etc/security/limits.conf && \
    mkdir /coredumps && \
    chmod o+rwx /coredumps

# use postgres as dev username; so we keep uid and group id with our host
RUN \
    # Create the user
    groupmod --gid ${USER_GID} ${DEV_USERNAME} && \
    usermod --uid ${USER_UID} ${DEV_USERNAME} && \
    #
    # [Optional] Add sudo support. Omit if you don't need to install software after connecting.
    echo $DEV_USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$DEV_USERNAME && \
    chmod 0440 /etc/sudoers.d/$DEV_USERNAME

RUN \
    install --verbose --directory --owner postgres --group postgres --mode 1777 ${PGDATA}

ENV LC_ALL="en_US.UTF-8"
ENV LANG="en_US.UTF-8"

USER $DEV_USERNAME

# unset
ENV http_proxy=
ENV https_proxy=
