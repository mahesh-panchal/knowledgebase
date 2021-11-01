# Internal NBIS Training: Introduction to Containers

Date and Time: Friday October 22nd, 9.00-12.00. 
## Instructors:

- Mahesh Binzer-Panchal
- Erik Fasterius
- Verena Kutschera
- Jonas Hagberg
- Martin Dahlö
- Björn Viklund

## Aims:

- Understand what a container is and why we use them.
- Understand the difference between a container image and a container instance.
- Understand what a container Registry is, and what a repository is.
- Understand differences between the Docker and Singularity container technologies.
- Be able to retrieve Docker or Singularity container Images.
- Be able to run commands in a Docker or Singularity container instance.
- Be able to pass data to and from a container.
- Be able to manage container images and their instances.
- Be able to create new Docker images for your needs.
- Understand how to use a container from a script or workflow manager.

:::info
As this is an introduction, best-practices for creating images are not covered in-depth.
For more in-depth questions, there will be time at the end for discussion.
:::

## Lesson

#### What is a container?

> “A Docker container image is a lightweight, standalone,
executable package of software that includes everything
needed to run an application: code, runtime, system tools,
system libraries and settings.”
[https://www.docker.com/resources/what-container]

#### Why use a container?

> “Available for both Linux and Windows-based applications,
containerized software will always run the same, regardless
of the infrastructure. Containers isolate software from its
environment and ensure that it works uniformly…”
[https://www.docker.com/resources/what-container]

#### Images and Container instances

A container has two aspects; the container image, and the container instance.

| Container image                 | Container instance           |
| ------------------------------- | ---------------------------- |
| The starting state (blueprint). | The working instance.        |
| It doesn’t change (immutable).  | Can modify contents.         |
| Shareable                       | Starts from the image state. |

#### Container Technologies

There are several container technologies available.

- Docker
- Singularity
- Podman
- Railcar
- Shifter
- more...

#### Registries vs Repositories

Registries serve, store, and distribute container images.

- Docker Hub (Docker): https://hub.docker.com/
- Redhat Quay (Docker, Podman, Rkt): https://quay.io/
- Singularity Hub (Singularity): https://singularity-hub.org/

Registries manage repositories; collections of images with the same name.

- quay.io/biocontainers/fastqc:0.11.9--0
- quay.io/biocontainers/fastqc:0.11.8--2
- quay.io/biocontainers/fastqc:0.11.7--6

and follow the pattern:
`<registry>/<name_space>/<repository>/<image>:<tag>--<build>`

##### Useful container links:

- [Biocontainers](https://biocontainers.pro/#/registry): Provides links to latest Conda package, Docker container, and Singularity container builds. One can also build mulled containers, which contain multiple conda packages. These are not easy to find however. Use the file https://github.com/BioContainers/multi-package-containers/blob/master/combinations/hash.tsv to see if a container exists for your needs, and then use the instructions in the README https://github.com/BioContainers/multi-package-containers to find the container path.
- [Rocker](https://www.rocker-project.org/images/): R in Docker; provides multipackage images from base R, to R + TidyVerse, and some others.
- [Rocker-ML](https://hub.docker.com/r/asachet/rocker-ml): Statistical Modelling libraries on top of Rocker, including plotting packages. 
- [Jupyter Notebooks](https://jupyter-docker-stacks.readthedocs.io/en/latest/using/selecting.html): Various packages (e.g., deep learning, scientific python) bundled together with jupyter notebook tools.
- [Nvidia CUDA container](https://hub.docker.com/r/nvidia/cuda): The CUDA container images provide an easy-to-use distribution for CUDA supported platforms and architectures.

### Docker

#### Run commands in a Docker container

The first step is to pull (retrieve or update) an image.
```bash
# Retrieve a copy of the fastqc image locally from quay.io
docker pull quay.io/biocontainers/fastqc:0.11.9--0
```

We can see the image we've pulled using the command
```bash
docker image ls

REPOSITORY                    TAG         IMAGE ID       CREATED        SIZE
quay.io/biocontainers/fastqc  0.11.9--0   9d444341a7b2   3 months ago   531MB 
```

Then we can use it to run commands. If the image is not locally 
available (from pull) then docker performs a pull first.
```bash
# Start an interactive (-it) container with name fqc
docker run --name fqc -it \
  quay.io/biocontainers/fastqc:0.11.9--0 bash

> fastqc --version
FastQC v0.11.9
> exit
```

Docker does not automatically clean up container instances. 
Once a container has completed it's task, it goes from the 
running state to the stopped state.
```bash
# list container images
docker container ls -a
# remove a container
docker rm fqc
```

You can also run a command directly in the container.
```bash
# Start a container based on the image `docker run <image>`
# Run a command `fastqc --version`
# Remove the container on completion `--rm`
docker run --rm \
  quay.io/biocontainers/fastqc:0.11.9--0 \
  fastqc --version

FastQC v0.11.9
```

###### Exercise 1: 15 mins

Pick a tool you commonly use in your work. Search in your browser for a container with the included tool. If you have docker installed, use `docker pull` to retrieve the container. Then test you can obtain the help text by running `<command> --help` or a similar instruction. If you cannot think of a tool, look for a container of the tool `jq`. Discuss with your group what indicators you can use to check if the tool is safe and suitable for your needs.

    - Was it difficult to find a container for your tool?
    - Was the container suitable for your needs (does it have the necessary dependancies installed e.g., do you also need bash)?
    - Could you run the help command directly and also from within an interactive shell?
    - What is the latest version? Does the tag `latest` correspond to a specific version tag?

Discussion notes:
- Room 1:
- Room 2:
- Room 3:
- Room 4:
- Room 5:
- Room 6:

#### Run commands in a Docker container

Docker can also be used to run services like an RStudio server, web server, database server, etc.
Services stay alive and keep the container in running state until the container is shutdown.
```bash
# Run RStudio server (default command)
docker run --rm \
  rocker/verse:4.1.1
  
# In a new terminal
docker ps

CONTAINER ID  IMAGE               COMMAND  CREATED         STATUS                     PORTS  NAMES
4c76b0f0fff1  rocker/verse:4.1.1  "/init"  2 minutes ago   Exited (0) 39 seconds ago         focused_swartz

# Execute R --version in the container named focused_swartz
docker exec focused_swartz R --version

R version 4.1.1 (2021-08-10) -- "Kick Things"
...
```

#### Connect outside to inside

Containers are isolated environments. To use your files inside, and save files outside, you need to map the outside to the inside.

```bash
# Connect ports with -p <outside_port>:<inside_port>
# Connect folders with -v <outside_folder>:<inside_folder>
# Set environment variables with -e <VAR>=<value>
docker run --rm -p 127.0.0.1:8787:8787 \
  -v $HOME/Documents/Rstudio:/home/rstudio \
  -e PASSWORD=<choose-a-password> \
  rocker/verse:4.1.1
```

:::warning
By default, Docker exposes container ports to the IP address 0.0.0.0 
(this matches any IP on the system). A good practice is to 
bind the exposed port to the localhost IP address (127.0.0.1).
:::

Processes run as the user “root”, by default, unless changed by the container image.
Supply your user and group id to read files written by container.

```bash
# Provide current user (UID) and group id (GROUPS array)
docker run --rm \
  -v $HOME/proj_2020:/home/timmy/proj_2020 \
  -u "$UID:$GROUPS" \
  quay.io/biocontainers/fastqc:0.11.9--0 \
  bash -c "fastqc /home/timmy/proj_2020/*.fastq.gz"
```

##### Docker compose

These options can be difficult to remember. Use a `docker-compose.yml` 
with `docker compose` to reproduce docker environments.

`docker-compose.yml`:
```yaml
# Run using: docker compose up
version: "3.9"
services:
  rstudio:
    image: 'rocker/verse:4.1.1'
    ports:
      - 127.0.0.1:8787:8787
    environment:
      - 'DISABLE_AUTH=true'
    volumes:
      - "$PWD:/home/rstudio"
    working_dir: /home/rstudio
    # user: "$UID:$GROUPS"
```

```bash
docker compose -f docker-compose.yml up -d # Start docker env
docker compose -f docker-compose.yml down  # Stop  docker env
```

#### Manage images and containers

Docker lets you manage both the images and container instances through the Docker API.
```bash
# List locally stored images
docker image ls

REPOSITORY                    TAG         IMAGE ID       CREATED        SIZE
quay.io/biocontainers/fastqc  0.11.9--0   9d444341a7b2   3 months ago   531MB 

# Remove a specific image (use REPOSITORY:TAG, or IMAGE ID)
docker image rm <image>

# Remove all unused images not connected to a container (be careful with images not retrieved from a repository)
docker image prune -a
```

```bash
# List locally running containers.
docker container ls
# List all containers (running, stopped, etc)
docker container ls -a

# Remove specific container
docker rm <container>

# Remove all stopped containers
docker container prune
```

#### Modify an existing image

1. Run a container:
    ```bash
    # Run an RStudio instance
    docker run --rm -p 127.0.0.1:8787:8787 \
      -v "$HOME/Documents/Rstudio:/home/rstudio" \
      -e PASSWORD=<choose-a-password> \
      rocker/verse:4.1.1
    ```
2. Make changes to the container instance (e.g., install package "distill").
    ```r
    installed.packages()
    install.packages('distill')
    ```
3. Open a new terminal and list the running container.
    ```bash
    docker container ls
    CONTAINER ID   IMAGE                COMMAND ...
    6e48f7cc7696   rocker/verse:4.1.1   "/init" ...
    ```
4. Check changes 
    ```bash
    docker diff 6e48f7cc7696
    ```
6. Commit changes to an image.
    ```bash
    # docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
    docker commit -m "Add distill" 6e48f7cc7696 verse_distill:1.0
    ```

#### Build your own container image

Docker container image definitions are written in a Dockerfile.

- [Dockerfile best practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

Example of a Dockerfile I needed to write for a support project (Private to NBIS: https://github.com/NBISweden/SMS-5084-20-Predicting-heteroresistance-in-bacterial-genomes).

Software: https://github.com/yachielab/SPADE

Dockerfile (annotated original)
```dockerfile=
# Select a base image, e.g., an OS or another image with tools included.
# https://hub.docker.com/r/continuumio/miniconda3/dockerfile
FROM continuumio/miniconda3:4.8.2

# Select the shell to use.
SHELL ["/bin/bash", "-c"]

# Add metadata to the container using labels.
LABEL description="Search for Patterned DNA Elements" \
      author="Mahesh Binzer-Panchal" \
      version="1.0"

# Update and install software dependencies
# with APT (Advanced Packaging  Tool)
RUN apt-get update && \
    apt-get install -y procps ghostscript

# Install tools using the conda package manager
RUN conda update -n base conda && \
    conda install -c conda-forge -c bioconda \
        python=3.6 \
        mafft=7.455 \
        blast=2.9.0 \
        openssl=1.1.1e && \
    conda clean --all -f -y

# Manual installation in another directory
WORKDIR /opt
RUN git clone --depth 1 \
        https://github.com/yachielab/SPADE && \
    cd SPADE && \
    chmod u+x *.py && \
    pip install matplotlib==2.2.3 && \
    pip install seaborn==0.8.1 && \
    pip install weblogo==3.6.0 && \
    pip install biopython==1.76

# Set environment variables
ENV PATH="/opt/SPADE:${PATH}"

# Provide a default command (entry point)
CMD [ "SPADE.py" ]
```

:::warning
Commands such as `apt-get update` and `conda update -n base conda` make the 
docker build unreproducible after some time has passed. Not including versions,
installation channels, and such, also reduce reproducibility. This makes it 
important to publically publish a Docker image you used in your analysis to 
a stable registry.
:::

Dockerfile (updated, after much trial and error)
```dockerfile=
# Select a base image, e.g., an OS or another image with tools included.
# https://hub.docker.com/r/mambaorg/micromamba
FROM mambaorg/micromamba:0.15.2

# Select the shell to use.
SHELL ["/bin/bash", "-c"]

# Add metadata to the container using labels.
LABEL description="Spade (Search for Patterned DNA Elements) container" \
      author="Mahesh Binzer-Panchal" \
      version="1.0.0"

# Change user to root to install things (set to micromamba in base image)
USER root

# Update and install software dependencies
# with APT (Advanced Packaging  Tool)
RUN apt-get update && \
    apt-get install -y procps ghostscript git

# Install tools using the mamba package manager
RUN micromamba install -y -n base -c conda-forge -c bioconda\
        python=3.7.10 \
        mafft=7.487 \
        blast=2.12.0 \
        openssl=1.1.1l \
        perl-bioperl=1.7.2 && \
    micromamba clean --all --yes

# Manual installation in another directory
WORKDIR /opt
RUN git clone --depth 1 https://github.com/yachielab/SPADE && \
    cd SPADE && chmod u+x *.py && \
    pip install matplotlib==2.2.3 && \
    pip install seaborn==0.8.1 && \
    pip install weblogo==3.6.0 && \
    pip install biopython

ENV PATH="/opt/SPADE:${PATH}"

CMD [ "SPADE.py" ]
```

```bash
# Build an image from the Dockerfile in the current directory
docker build -t spade:1.0 .       # Note: Last argument is  "."

# Or build and push to a repository
docker build \
  -t ghcr.io/<user-name>/<repository>/spade:1.0 .
docker push \
  ghcr.io/<user-name>/<repository>/spade:1.0
```

After pushing to Github, you need to connect the package to the repository.
Select "Your profile" on Github from your user-icon, and then select "Packages".
Click on the package and select the repository you want to connect it to. 

:::info
To push to the GitHub Container Registry, you need to first provide Docker with credentials from GitHub.

We'll use a Personal Access Token (PAT) here.
In your web-browser, click on your user-icon in GitHub in the top-right of the screen. Then go to Settings > Developer Settings > Personal Access Tokens, click on "Generate new token", and enable "write:packages". Provide a note, duration, and check other settings, followed by "Generate token". Copy the token.
Then in your terminal:
```bash
export GHCR_TOKEN=<your_token>
echo "$GHCR_TOKEN" | docker login ghcr.io -u <GitHub username> --password-stdin
```
:::

### Singularity

#### Differences to Docker

These are a few differences between Singularity and Docker.

| Singularity                                     | Docker                                     |
| ------------------------------------------------| ------------------------------------------ |
| Runs as current user                            | Runs as root user                          |
| $HOME, $PWD, and /tmp are mounted automatically | Isolated                                   |
| Linux, Windows 10 only (Mac beta)               | Linux, Mac, and Windows integration.       |
| Supports Singularity and Docker images          | Supports Docker images                     |
| Images are normal files                         | Image interaction through docker interface |
| Internal folders read-only                      | Internal folders writable                  |

#### Run commands in a Singularity container

Working with Singularity has similarities to working with Docker.
```bash
# Pull an image
singularity pull \
  docker://quay.io/biocontainers/fastqc:0.11.9--0

ls
fastqc_0.11.9--0.sif
```

```bash
# Run an interactive terminal
singularity shell \
  docker://quay.io/biocontainers/fastqc:0.11.9--0

> fastqc --version
FastQC v0.11.9
```

```bash
# Execute a command
singularity exec \
  docker://quay.io/biocontainers/fastqc:0.11.9--0 \
  fastqc --version
FastQC v0.11.9
```

:::info
Retrieve the Rocker verse singularity image on Rackham from:
```
/proj/snic2018-8-34/mahesh/intro-to-containers/rocker-verse_v4.1.1.sif
```
Alternatively, use singularity to convert the Docker image to Singularity 
format (it will take some time).
```bash
singularity pull --name rocker-verse_v4.1.1.sif docker://rocker/verse:4.1.1
```
:::

Services need to be run in Singularity in a different manner.
```bash
# Start a singularity instance
singularity instance start \
  rocker-verse_4.1.1.sif \
  my-rstudio

# Execute a command on the instance
singularity exec instance://my-rstudio R --version

# Stop the instance
singularity instance stop my-rstudio
```

#### Building a Singularity image

Building a Singularity image requires "root" permissions.

Cloud Builder:
```bash
# Build an image using cloud builder (https://cloud.sylabs.io/builder)
# See 'singularity build --help' for writing definition files
singularity build --remote my_image.sif Singularityfile
```

Singularity in Docker:
```bash
# Image from https://hub.docker.com/r/kaczmarj/singularity
docker run --rm --privileged \
  -v $(pwd):/work \
  kaczmarj/singularity:3.8.0 \
  build myimage.sif Singularity_test
```

###### Exercise 2: 15 mins

Discuss in your groups pros and cons of the differences between singularity and docker and how you might use them for your needs. Some specific discussion points are listed:

    - What effect could the auto mounting of $HOME have? Do any of the tools you use load data from hidden folders in $HOME?
    - How can you transfer an image to/from a computer cluster with limited internet access (e.g., Bianca)?
    - How can you work reproducibly with docker and singularity?

Discussion notes:
- Room 1:
- Room 2:
- Room 3:
- Room 4:
- Room 5:
- Room 6:

### Scripting with a container.

#### Bash:

```bash
#! /usr/bin/env bash

# <slurm settings>

CPUS="${SLURM_NPROCS:-6}"
JOB="${SLURM_ARRAY_TASK_ID:-0}"

DATA_DIR='/path/to/reads'
FILES=( "$DATA_DIR"/*_R1.fastq.gz )

FASTQ="${FILES[$JOB]}"

singularity exec \
  docker://quay.io/biocontainers/fastqc:0.11.9--0 \
  fastqc -t "$CPUS" "$FASTQ" "${FASTQ/_R1./_R2.}"
```

#### Snakemake:

```snakemake
rule NAME:
    input:
        "table.txt"
    output:
        "plots/myplot.pdf"
    container:
        "docker://rocker/rstudio:4.1.0"
    script:
        "scripts/plot-stuff.R"
```

and:

```bash
snakemake --use-singularity
```

#### Nextflow:

```nextflow
process FASTQC {

    container "https://depot.galaxyproject.org/singularity/fastqc:0.11.9--0"

    input:
    tuple val(sample), path(reads)

    output:
    path ("fastqc_${sample}_logs")

    script:
    """
    mkdir fastqc_${sample}_logs
    fastqc -t ${task.cpus} -o fastqc_${sample}_logs -f fastq -q ${reads}
    """

}
```

and in the nextflow.config:
```nextflow
singularity.enabled = true
```

###### Exercise 3: 10 mins

Download/copy the following data:
```
/proj/snic2018-8-34/mahesh/intro-to-containers/SRR2589044_{1,2}.fastq.gz
# Also publically available from:
# ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR258/004/SRR2589044/SRR2589044_{1,2}.fastq.gz
```
Write a script to use existing containers for fastqc and fastp to process the data.
```bash
#! /usr/bin/env bash

## QC reads with FastQC
fastqc *.fastq.gz

## Trim adapters from reads with FastP
for READ1 in *_1.fastq.gz; do
    PREFIX=$( basename "$READ1" _1.fastq.gz )
    READ2="${READ1/_1./_2.}"
    fastp -i "$READ1" -I "$READ2" \
        -o "${PREFIX}_1.trimmed.fastq.gz" \
        -O "${PREFIX}_2.trimmed.fastq.gz" \
        --json "$PREFIX.json"
done
```

#### Container use in practice

This is an example of how I use containers in practice (and attempt to work reproducibly).

https://github.com/NBISweden/SMS-5084-20-Predicting-heteroresistance-in-bacterial-genomes

https://github.com/mahesh-panchal/NBIS_project_template

###### Exercise 4: 20 mins

Build an image containing the tools `fastqc`, `fastp` `bwa`, and `samtools` using either Singularity or Docker. Use the miniconda image as the base image. 

### General comments

#### Tips

- Don't reinvent the wheel. Use official sources when possible.
- Save changes to a container by committing changes to a new image.
- Limit container images to one service.
- Separate workflows and data from the service. Don’t use containers as an analysis archive.
- Use package managers to handle the bulk of installation.
- Build containers following best practices. 
    - Don’t keep unnecessary files.
    - Use version numbers.
    - Use explicit installation channels.
    - Be aware of unreproducible commands e.g., `apt-get update`
- Beware of including sensitive data.

#### Summary

- Docker and Singularity are widely supported.
- Containers provide reusable services.
- Reproducible - Return in 5 years and have the same environment.
- Simplifies tool installation for user.
- This barely scratches the surface.

## Discussion Rooms

1. Reflection room:   Discussion of todays workshop and exercises. (Mahesh)
2. Supermax room:     Container security and encryption. (Jonas)
3. J.A.R.V.I.S. room: Containers on Uppmax (Rackham and Bianca) (Martin and Björn)
4. DuctTape room:     Scripting (Bash, Nextflow, SnakeMake) with containers. (Verena and Erik)

## Further Reading

- [NBIS Workshop: Tools for reproducible research](https://uppsala.instructure.com/courses/51980)
- [Carpentries: Introduction to Docker](https://carpentries-incubator.github.io/docker-introduction/)
- [Carpentries: Introduction to Singularity](https://carpentries-incubator.github.io/singularity-introduction/)

## TODO
- docker save - save as tarball
- add whoami to user instruction above
- add live demo to process data inside
- add warnings on container security, only expose local host, and folder you are using (not My Documents)
- why work modularly?
  - update 1 tool, don't need to rebuild everything 
  - allows your analysis to scale
  - don't waste time building containers, usually trustworthy and tested.
- docker commit: No paper trail of what happened to build image.
- Singularity build tips on Uppmax: https://pmitev.github.io/UPPMAX-Singularity-workshop/goods_bads/
- https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1008316