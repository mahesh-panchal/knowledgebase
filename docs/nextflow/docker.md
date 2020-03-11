---
layout: default
title: Docker
parent: Nextflow
nav_order: 3
---

## Getting started with Docker

- [Getting started](https://docs.docker.com/get-started/part2/)

## Dockerfile basics

- Use multi-stage build if you need to compile.
- Keep things on one line ( each line is a layer ).
- Install tools alphabetically for readability.

Example:
```
FROM ubuntu:18.04 AS builder

# Install Dependencies
RUN apt-get update && apt-get -y install \
    cmake \
    g++ \
    gcc \
    git \
    libarmadillo-dev \
    libblas-dev \
    libboost-dev \
    liblapack-dev

WORKDIR /tmp
RUN git clone --depth 1 https://bitbucket.org/WegmannLab/atlas.git && \
    cd atlas && make

FROM ubuntu:18.04

LABEL description="Atlas ancient DNA toolkit container" \
      author="Mahesh Binzer-Panchal" \
      version="0.9.9" \
      tool_author="Vivian Link, Athanasios Kousathanas, Krishna Veeramah, Christian Sell, Amelie Scheu, Daniel Wegmann" \
      doi = "https://doi.org/10.1101/105346"

RUN apt-get update && apt-get -y install \
    libarmadillo-dev \
    libblas-dev \
    libboost-dev \
    liblapack-dev

COPY --from=builder /tmp/atlas/atlas /usr/local/bin/atlas

CMD [ "atlas" ]
```

## Publishing docker image to github packages.

1. Log into Github packages with docker

```bash
docker login -u <USERNAME> docker.pkg.github.com
```

When prompted, paste the Github access token as the password.


2. A) Make the image locally, tag it, and push it.

```bash
docker image build -t <IMAGE_NAME>:<VERSION> .
docker images
docker tag <IMAGE_ID> docker.pkg.github.com/<OWNER>/<REPOSITORY>/<IMAGE_NAME>:<VERSION>
docker push docker.pkg.github.com/<OWNER>/<REPOSITORY>/<IMAGE_NAME>:<VERSION>
```

2. B) Build directly and push.

```bash
docker build -t docker.pkg.github.com/<OWNER>/<REPOSITORY>/<IMAGE_NAME>:<VERSION> .
docker push docker.pkg.github.com/<OWNER>/<REPOSITORY>/<IMAGE_NAME>:<VERSION>
```
