## The contents of this file are subject to the Mozilla Public License
## Version 1.1 (the "License"); you may not use this file except in
## compliance with the License. You may obtain a copy of the License
## at http://www.mozilla.org/MPL/
#
## Software distributed under the License is distributed on an "AS IS"
## basis, WITHOUT WARRANTY OF ANY KIND, either express or implied. See
## the License for the specific language governing rights and
## limitations under the License.
#
## The Original Code is RabbitMQ.
#
## The Initial Developer of the Original Code is GoPivotal, Inc.
## Copyright (c) 2007-2025 Broadcom, Inc. or its subsidiaries.

FROM debian:bookworm

ENV LANG='C.UTF-8'

# Enable backports.
RUN echo 'deb http://httpredir.debian.org/debian bookworm-backports main' \
    >> /etc/apt/sources.list.d/backports.list

# Prerequisites to mess with packages.
RUN apt-get clean && \
    apt-get update && \
    apt-get install -y -V --no-install-recommends \
    apt-transport-https \
    ca-certificates \
    wget \
    debconf-utils \
    gnupg

# Our own rabbitmq-erlang repository to provision Erlang.
RUN echo 'deb https://ppa2.rabbitmq.com/rabbitmq/rabbitmq-erlang/deb/debian bookworm main' > /etc/apt/sources.list.d/rabbitmq-erlang.list && \
    wget -O- https://dl.cloudsmith.io/public/rabbitmq/rabbitmq-erlang/gpg.E495BB49CC4BBE5B.key | apt-key add -

# We need to set an APT preference to make sure $ERLANG_VERSION is
# used for all erlang* packages. Without this, apt-get(1) would try to
# install dependencies using the latest version. This would conflict
# with the strict pinning in all packages, and thus fail.
RUN ERLANG_VERSION=1:26.2.5.9-1 && \
    echo 'Package: erlang*' > /etc/apt/preferences.d/erlang && \
    echo "Pin: version $ERLANG_VERSION" >> /etc/apt/preferences.d/erlang && \
    echo 'Pin-Priority: 1000' >> /etc/apt/preferences.d/erlang

# --------------------------------------------------------------------
# Packages to build RabbitMQ.
# --------------------------------------------------------------------

RUN apt-get clean && \
    apt-get update && \
    apt-get install -y -V --fix-missing --no-install-recommends \
    build-essential \
    clang \
    curl \
    debhelper \
    # for Debian package builds
    elixir \
    erlang-base \
    erlang-crypto \
    erlang-edoc \
    erlang-eldap \
    erlang-erl-docgen \
    erlang-eunit \
    erlang-inets \
    erlang-mnesia \
    erlang-os-mon \
    erlang-parsetools \
    erlang-public-key \
    erlang-runtime-tools \
    erlang-ssh \
    erlang-ssl \
    erlang-syntax-tools \
    erlang-tools \
    erlang-xmerl \
    erlang-nox \
    erlang-dev \
    erlang-src \
    erlang-common-test \
    erlang-dialyzer \
    libcurl3-gnutls \
    man \
    mandoc \
    osslsigncode \
    7zip \
    rsync \
    xmlto \
    xsltproc \
    zip \
    unzip && \
    ln -s $(which 7zz) /usr/bin/7z

RUN apt-get install -y -V --fix-missing --no-install-recommends \
    -t bookworm-backports \
    git openssh-client

# Verify the version of the installed Erlang packages.
# The version was pinned above, but if that specific version is unavailable,
# the latest version will be installed, which we don't want.
RUN set -xe \
    && installed_version=$(dpkg -s erlang-nox | grep -E '^Version:' | awk '{ print $2; }') \
    && wanted_version=$(awk '/^Pin:/ { print $3; }' < /etc/apt/preferences.d/erlang) \
    && test "$installed_version" = "$wanted_version"

RUN set -xe \
    && REBAR3_VERSION="3.24.0" \
    && REBAR3_DOWNLOAD_URL="https://github.com/erlang/rebar3/releases/download/${REBAR3_VERSION}/rebar3" \
    && REBAR3_DOWNLOAD_SHA256="d2d31cfb98904b8e4917300a75f870de12cb5167cd6214d1043e973a56668a54" \
    && curl -fSL -o "rebar3-${REBAR3_VERSION}" "$REBAR3_DOWNLOAD_URL" \
    && echo "$REBAR3_DOWNLOAD_SHA256 rebar3-${REBAR3_VERSION}" | sha256sum -c - \
    && mv "rebar3-${REBAR3_VERSION}" /usr/local/bin/rebar3 \
    && chmod +x /usr/local/bin/rebar3

# Compile Elixir from source so that we know for sure what version was used to built it.
# Elixir's bin and lib are installed under /usr/local/bin and /usr/local/lib.
RUN set -xe \
    && ELIXIR_VERSION="1.17.3" \
    && ELIXIR_DOWNLOAD_URL="https://github.com/elixir-lang/elixir/archive/refs/tags/v${ELIXIR_VERSION}.zip" \
    && ELIXIR_DOWNLOAD_SHA256="94544b7200265ee45ac7fa6d4681824815c888a009face7f2d1d6fe38624814c" \
    && curl -fSL -o elixir-source.zip $ELIXIR_DOWNLOAD_URL \
    && echo "$ELIXIR_DOWNLOAD_SHA256 elixir-source.zip" | sha256sum -c - \
    && unzip -d /tmp/elixir-source elixir-source.zip \
    && make -C "/tmp/elixir-source/elixir-${ELIXIR_VERSION}" \
    && mv /tmp/elixir-source/elixir-${ELIXIR_VERSION}/bin/* /usr/local/bin/ \
    && mv /tmp/elixir-source/elixir-${ELIXIR_VERSION}/lib/* /usr/local/lib/ \
    && mv /tmp/elixir-source/elixir-${ELIXIR_VERSION}/man/* /usr/local/man/ \
    && rm -rf elixir-source.zip /tmp/elixir-source

# Put erl_call(1) in the $PATH.
RUN ln -fs /usr/lib/erlang/lib/erl_interface-*/bin/erl_call /usr/bin/erl_call

RUN apt-get install -y -V --fix-missing --no-install-recommends \
    ldap-utils \
    netcat-traditional \
    python3-dev \
    python3-simplejson \
    python3 \
    python3-dev \
    python3-pip \
    python3-setuptools \
    daemonize

# --------------------------------------------------------------------
# Tools for building RabbitMQ packages (Debian, RPM, Windows installer)
# --------------------------------------------------------------------

RUN apt-get install -y -V --fix-missing --no-install-recommends \
    nsis \
    rpm \
    tofrodos

# --------------------------------------------------------------------
# Tools for testing RabbitMQ packages (Debian, RPM, Windows installer)
# --------------------------------------------------------------------

#
# Debian
#

# Unlike apt, dpkg -i requires package dependencies to be pre-installed
RUN apt-get install -y -V --fix-missing --no-install-recommends \
    logrotate

# --------------------------------------------------------------------
# For running bats tests
# --------------------------------------------------------------------

RUN git clone --depth 1 https://github.com/bats-core/bats-core.git && \
    cd bats-core && \
    ./install.sh /usr && \
    cd .. && \
    rm -rf bats-core

# --------------------------------------------------------------------
# For debugging from the container directly
# --------------------------------------------------------------------

RUN apt-get install -y -V --fix-missing --no-install-recommends \
    bash-completion \
    elinks \
    vim

RUN apt-get autoclean -y
