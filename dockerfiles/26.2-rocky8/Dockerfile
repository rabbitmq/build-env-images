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

FROM rockylinux:8

ENV LANG='C.UTF-8'

# Enable PowerTools (codeready-builder) repository for additional packages
RUN dnf install -y 'dnf-command(config-manager)' && \
    dnf config-manager --set-enabled powertools

# Prerequisites to mess with packages.
RUN dnf clean all && \
    dnf install -y \
    ca-certificates \
    curl \
    gnupg2 \
    wget

# Install Erlang 26.x from GitHub releases
# Following instructions from:
# https://github.com/rabbitmq/erlang-rpm/blob/erlang-26/README.md
# Since the YUM repository doesn't have aarch64 packages, we download
# directly from GitHub releases
RUN ERLANG_VERSION="26.2.5.14" && \
    ERLANG_RPM_VERSION="26.2.5.14-1.el8" && \
    ARCH=$(uname -m) && \
    cd /tmp && \
    curl -fSL -o erlang-${ERLANG_VERSION}-1.el8.${ARCH}.rpm \
    "https://github.com/rabbitmq/erlang-rpm/releases/download/v${ERLANG_VERSION}/erlang-${ERLANG_RPM_VERSION}.${ARCH}.rpm" && \
    dnf install -y erlang-${ERLANG_VERSION}-1.el8.${ARCH}.rpm && \
    rm -f erlang-${ERLANG_VERSION}-1.el8.${ARCH}.rpm

# --------------------------------------------------------------------
# Packages to build RabbitMQ.
# --------------------------------------------------------------------

# Install build dependencies
RUN dnf install -y \
    gcc \
    gcc-c++ \
    make \
    clang \
    curl \
    rsync \
    zip \
    unzip \
    xmlto \
    libxslt \
    man-db

# Install p7zip from EPEL
RUN dnf install -y epel-release && \
    dnf install -y p7zip p7zip-plugins && \
    ln -sf /usr/bin/7za /usr/bin/7z

# Install git from a more recent source if needed
RUN dnf install -y git openssh-clients

# Verify the version of the installed Erlang packages.
RUN set -xe \
    && installed_version=$(rpm -q --queryformat '%{VERSION}-%{RELEASE}' erlang) \
    && echo "Installed Erlang version: $installed_version" \
    && rpm -q erlang | grep -q "26.2.5.14"

RUN set -xe \
    && REBAR3_VERSION="3.24.0" \
    && REBAR3_DOWNLOAD_URL="https://github.com/erlang/rebar3/releases/download/${REBAR3_VERSION}/rebar3" \
    && REBAR3_DOWNLOAD_SHA256="d2d31cfb98904b8e4917300a75f870de12cb5167cd6214d1043e973a56668a54" \
    && curl -fSL -o "rebar3-${REBAR3_VERSION}" "$REBAR3_DOWNLOAD_URL" \
    && echo "$REBAR3_DOWNLOAD_SHA256 rebar3-${REBAR3_VERSION}" | sha256sum -c - \
    && mv "rebar3-${REBAR3_VERSION}" /usr/local/bin/rebar3 \
    && chmod +x /usr/local/bin/rebar3

# Compile Elixir from source so that we know for sure what version was
# used to built it. Elixir's bin and lib are installed under
# /usr/local/bin and /usr/local/lib.
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
    && mv /tmp/elixir-source/elixir-${ELIXIR_VERSION}/man/* /usr/local/share/man/ \
    && rm -rf elixir-source.zip /tmp/elixir-source

# Put erl_call(1) in the $PATH.
RUN ln -fs /usr/lib64/erlang/lib/erl_interface-*/bin/erl_call /usr/bin/erl_call

RUN dnf install -y \
    openldap-clients \
    nmap-ncat \
    python3-devel \
    python3 \
    python3-pip \
    python3-setuptools

# --------------------------------------------------------------------
# Tools for building RabbitMQ packages (Debian, RPM, Windows installer)
# --------------------------------------------------------------------

# Install NSIS for Windows installer builds
# Note: NSIS is not available in standard Rocky repos, using EPEL
RUN dnf install -y epel-release && \
    dnf install -y \
    rpm-build \
    rpm-sign \
    createrepo \
    osslsigncode \
    dos2unix

# --------------------------------------------------------------------
# Tools for testing RabbitMQ packages (Debian, RPM, Windows installer)
# --------------------------------------------------------------------

# Install logrotate for package testing
RUN dnf install -y logrotate

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

RUN dnf install -y \
    bash-completion \
    elinks \
    vim

RUN dnf clean all
