# Keycloak Bitnami Image

[![build](https://github.com/eminaktas/keycloak-bitnami-image/actions/workflows/release.yaml/badge.svg?branch=main)](https://github.com/eminaktas/keycloak-bitnami-image/actions/workflows/release.yaml)

This project aims to build a secure and up-to-date container image for Bitnami's Keycloak Chart, leveraging the Apko and Melange projects to create an image free from Common Vulnerabilities and Exposures (CVEs).

| Image Tag  | Keycloak Version  | Description                               |
|------------|-------------------|-------------------------------------------|
| **latest** | **25.0.8**        | Latest stable release of Keycloak         |
| latest-24  | 24.0.8            | Latest release within Keycloak version 24 |
| latest-23  | 23.0.7            | Latest release within Keycloak version 23 |
| latest-22  | 22.0.13           | Latest release within Keycloak version 22 |

## Table of Contents

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Building the Container Image](#building-the-container-image)
- [License](#license)

## Introduction

Keycloak Bitnami Container Image is a project that automates the process of building a container image for Bitnami's Keycloak Helm Chart while ensuring it is free from known security vulnerabilities. This is achieved through the use of the Apko and Melange projects, which provide tools for scanning and remediating CVEs in container images.

## Prerequisites

Before you begin, ensure you have the following prerequisites installed:

- [Docker](https://www.docker.com/get-started/)
- [Apko](https://github.com/chainguard-dev/apko)
- [Melange](https://github.com/melange-re/melange)

## Getting Started

To quickly get started, run the following command to launch Keycloak Bitnami using a specific image:

```bash
docker run --rm -it -p 8080:8080 \
  -e KEYCLOAK_DATABASE_VENDOR=dev-file \
  -e KEYCLOAK_ENABLE_HEALTH_ENDPOINTS=true \
  ghcr.io/eminaktas/keycloak-bitnami-image:latest
```

Also, you can use Docker Compose to run the Keycloak with PostgreSQL:

```bash
docker-compose up
```

This command pulls and runs the specified Keycloak image, exposing it on port 8080. Adjust the environment variables as needed.

## Building the Container Image

To build the container image using Melange and Apko, follow these steps:

- Generate temporary keys with Melange:

```bash
docker run --rm -v "$(pwd)":/work cgr.dev/chainguard/melange keygen
```

- Build the package for Keycloak metrics SPI:

```bash
KEYCLOAK_VERSION=24 docker run --privileged --rm -v "$(pwd)":/work -w /work \
  cgr.dev/chainguard/melange build $KEYCLOAK_VERSION/keycloak-metrics-spi-melange.yaml \
  --signing-key melange.rsa
```

- Build the package for Keycloak:

```bash
KEYCLOAK_VERSION=24 docker run --privileged --rm -v "$(pwd)":/work -w /work \
  cgr.dev/chainguard/melange build $KEYCLOAK_VERSION/keycloak-melange.yaml \
  --signing-key melange.rsa --pipeline-dir pipelines
```

- Build the container image:

```bash
KEYCLOAK_VERSION=24 docker run --rm -v "$(pwd)":/work -w /work cgr.dev/chainguard/apko build $KEYCLOAK_VERSION/apko.yaml \
  keycloak-bitnami-image:latest keycloak-bitnami-image-latest.tar \
  --keyring-append melange.rsa.pub
```

These commands generate the necessary keys, apk packages for Keycloak, and build the final container image for Keycloak Bitnami.

## License

This project is licensed under the [Apache-2.0 License](LICENSE).
