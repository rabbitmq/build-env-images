# Images for Building and Testing of Open Source RabbitMQ Packages

## Disclaimer

This subproject is very new. Things can break, documentation will be lacking,
design decisions will be revisited, and so on. Please do not rely on this repository
unless you plan to regularly contribute to RabbitMQ and have consulted with the RabbitMQ Core Team.

## What is This?

This repository contains OCI images useful for

1. Building RabbitMQ from a source tarball
2. Building several package types: Debian, RPM, the Windows installer

These images are NOT meant to be used to run RabbitMQ. For that, see the [community Docker image](https://github.com/docker-library/rabbitmq),
Bitnami, and so on.

The images are then [published to Docker Hub](https://hub.docker.com/u/pivotalrabbitmq).


## Repository Layout

This repository is GitHub Actions-driven. Workflow definitions can be found under
their usual place, `.github/workflows`.

The Dockerfiles can be found under `dockerfiles`.


## Build Results

See [Actions](https://github.com/rabbitmq/build-env-images/actions).


## Erlang 26.x (amd64)

This Debian Bookworm-based image provides a very recent version of Erlang 26.x plus a very
recent version of Elixir.

Only `amd64` packages are produced because our team only produces `amd64` Debian packages for Erlang
at the moment.

To pull it:

```bash
docker pull rabbitmqdevenv/build-env-26.2
```

## Chocolatey Package Publishing

[RabbitMQ Chocolatey package](https://github.com/rabbitmq/chocolatey-package) is published
using the official OCI image from the creators of Chocolatey, [`chocolatey/choco`](https://hub.docker.com/r/chocolatey/choco).
Its source can be [inspected on GitHub](https://github.com/chocolatey/choco/blob/develop/docker/).


## Copyright and License

This work is dual-licensed under the Apache License 2.0 and the Mozilla Public License 2.0.
Users can choose any of these licenses according to their needs.

SPDX-License-Identifier: Apache-2.0 OR MPL-2.0