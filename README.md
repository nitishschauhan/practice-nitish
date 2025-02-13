# Infrastructure Assessment - GCP

![Tw logo](logo.png)
![gcp logo](gcp.png)

This project contains 3 services:

* `quotes` - it serves a random quote from `quotes/resources/quotes.json`
* `newsfeed` - it aggregates several RSS feeds together
* `front-end` - it calls the `quotes & newsfeed` to display the results

These services are provided as **Docker images**. 

This README documents the steps to build the images and provision the infrastructure for the services.

:exclamation: **NB** All infrastructure should be already provisioned and available to you at the time of the interview. It is not your responsibility to deploy the base infrastructure

## Development and operations tools setup

There are 2 options for getting the right tools on your laptop:

* **Quick:** it leverages Docker+Dojo. Install Docker & dojo on your laptop
* **Manual:** all tools are installed manually

:exclamation: **NB** Please refer to [MANUAL_SETUP.md](MANUAL_SETUP.md) for the manual setup.

### Docker+Dojo setup

For sake of simplicity and full automation, the entire code base, docker images, and infrastructure can be built and provisioned from your machine using the following 4 tools:

- Docker
- [Dojo](https://github.com/kudulab/dojo)
- GNU Make
- Google Cloud CLI SDK (`gcloud`), see [install](https://cloud.google.com/sdk/docs/install)

The [Dojo](https://github.com/kudulab/dojo) Docker wrapper tool is used to encapsulate a lot of cross-platform/version dependencies and ensures a uniform environment to run consistently across different platforms using Docker containers.

:exclamation: **NB** To install all the tools required on a Unix based machine run:

```bash
# OSX
brew install kudulab/homebrew-dojo-osx/dojo

# Linux
DOJO_VERSION=0.10.3
wget -O dojo https://github.com/kudulab/dojo/releases/download/${DOJO_VERSION}/dojo_linux_amd64
chmod +x dojo
sudo mv dojo /usr/local/bin
```

## Setup GCP authentication

The copy of the codebase that you have received should contain the *credentials* required, in the file `infra/.interviewee-creds.json`. 

:exclamation: **NB**  If something went wrong with the credentials, your interviewer should be able to email you those.

You can activate your credentials using either of the following commands:

```bash
make login-gcloud

# OR

gcloud auth activate-service-account --key-file=infra/.interviewee-creds.json
```

## Infrastructure setup

:exclamation: **NB** To save time, your interviewer will already have run the following steps before the interview starts. You should still be aware of how they work.

This is a multi-step guide to setup some base infrastructure, and then, on top of it, the test environment for the newsfeed application.

These steps will provision:

* a Terraform backend using a GCS bucket
* a minimal VPC with 1 subnet
* GCR repositories for Docker images

The `make deploy_interview` helper command will run all the steps in sequence.

### Step 1: Base infrastructure setup

With the assumption that we have a new, empty GCP Project, we need to first provision some base infrastructure.

First create *basic project infrastructure*, such as a container registry, by running the following command:

```bash
make base.infra
```

### Step 2: Build the application artifacts

If you have not built the jars and static resources yet, do so by running the following command:

```bash
make apps
```

### Step 3: Build Docker images

Artifacts from the previous step will be packaged into Docker images, then pushed to ECR.

Each application has its own image. Individual image can be built with the following command:

```bash
make <app-name>.docker
# for example:
make front-end.docker
```

You can build all images at once by running the following command:

```bash
make docker
```

### Step 4: Push Docker images

Before applications can be deployed on GCP, the Docker images have to be pushed to GCR (private).

You can push the Docker images to the GCR repository by running the following command:

```bash
make push
```

### Step 5: Provision services

To provision the backend and front-end services run the following command:

```bash
make news.infra
```

Terraform will print the output with the URL of the `front_end` server, e.g.

```bash
# Outputs:

frontend_url = http://34.244.219.156:8080
```
