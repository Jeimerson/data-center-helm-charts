# ![Atlassian Bamboo](https://wac-cdn.atlassian.com/dam/jcr:560a991e-c0e3-4014-bd7d-2e65d4e4c84a/bamboo-icon-gradient-blue.svg?cdnVersion=814){: style="height:35px;width:35px"} Bamboo Agent

## Overview

A Bamboo Agent is a service that can run job builds. Each agent has a defined set of capabilities and can run builds
only for jobs whose requirements match the agent's capabilities. 


If you are looking for **Bamboo Docker Image** it can be found [here](https://hub.docker.com/r/atlassian/bamboo/).
To learn more about Bamboo, see: <https://www.atlassian.com/software/bamboo>

This Docker container makes it easy to get a Bamboo Remote Agent up and running. It is intended to be used as a base to 
build from, and as such contains limited built-in capabilities:

* JDK 11, JDK 17 (from v9.4.0), JDK 21 (from v10.1.0)
* Git & Git LFS
* Maven 3
* Python 3

Using this image as a base, you can create a custom remote agent image with your
desired build tools installed. Note that Bamboo Agent Docker Image does not
include a Bamboo server.

**Use docker version >= 20.10.9.**

## Quick Start

For the `BAMBOO_AGENT_HOME` directory that is used to store the repository data (amongst other things) we recommend mounting a host directory as a [data volume](https://docs.docker.com/engine/tutorials/dockervolumes/#/data-volumes), or via a named volume.

To get started you can use a data volume, or named volumes. In this example we'll use named volumes.

Run an Agent:
```shell
docker volume create --name bambooAgentVolume
docker run -e BAMBOO_SERVER=http://bamboo.mycompany.com/agentServer/ -v bambooAgentVolume:/var/atlassian/application-data/bamboo-agent --name="bambooAgent" --hostname="bambooAgent" -d atlassian/bamboo-agent-base
```
!!! success "The Bamboo remote agent is now available to be approved in your Bamboo administration."

### Verbose container entrypoint logging

During the startup process of the container, various operations and checks are performed to ensure that the application
is configured correctly and ready to run. To help in troubleshooting and to provide transparency into this process, you
can enable verbose logging. The `VERBOSE_LOGS` environment variable enables detailed debug messages to the container's
log, offering insights into the actions performed by the entrypoint script.

* `VERBOSE_LOGS` (default: false)

  Set to `true` to enable detailed debug messages during the container initialization.

## Configuration

* `BAMBOO_SERVER` (required)

   The URL of the Bamboo Server the remote agent should connect to, e.g. `http://bamboo.mycompany.com/agentServer/`

* `SECURITY_TOKEN` (default: NONE)

   If security token verification is enabled, this value specifies the token required to authenticate to the Bamboo server

* `WRAPPER_JAVA_INITMEMORY` (default: 256)

   The minimum heap size of the JVM. This value is in MB and should be specified as an integer

* `WRAPPER_JAVA_MAXMEMORY` (default: 512)

   The maximum heap size of the JVM. This value is in MB and should be specified as an integer

* `IGNORE_SERVER_CERT_NAME` (default: false)

   Ignore SSL certificate hostname if it's issued to a different host than the one under your Bamboo Base URL hostname

* `ALLOW_EMPTY_ARTIFACTS` (default: false)

   Allow empty directories to be published as artifacts

* `BAMBOO_AGENT_PERMISSIVE_READINESS` (default: unset/false)

   If set to 'true', the readiness probe will be more permissive and not expect
   the agent to be fully configured, only that the startup wrapper is
   running. This is primarily intended for use when deploying agents into
   environments where the server may not yet be configured.

* `BAMBOO_AGENT_CLASSPATH_DIR` (default: NONE)

   If set, agent startup process will copy agent classpath from designated location instead of downloading it from the server.
   This can speed up the process and reduce the load on the Bamboo server.

### Dedicated agent specific configuration

* `AGENT_EPHEMERAL_FOR_KEY` (default: NONE)

  The value specifies the purpose for spawning the agent. It needs to be a valid ResultKey.

* `KUBE_NUM_EXTRA_CONTAINERS` (default: 0) 

  The number of extra containers that run in parallel with the Bamboo Agent. We make sure these extra containers are run before the Agent kick in.

* `EXTRA_CONTAINERS_REGISTRATION_DIRECTORY` (default: /pbc/kube)

  The directory where extra containers should register their readiness by creating any file. The image waits for having `KUBE_NUM_EXTRA_CONTAINERS` number of files inside this directory (if the one exists) before processing further and running the actual agent.

### Ephemeral agent specific configuration

* `BAMBOO_EPHEMERAL_AGENT_DATA` (default: NONE)

  The Bamboo Ephemeral Agents specific configuration. It was designed to pass multiple key-value properties separated by the `#`. Example: `BAMBOO_SERVER=http://localhost#SECURITY_TOKEN=123456789#bamboo.agent.ephemeral.for.key=PROJ-PLAN-JOB1-1`

### Additional agent wrapper properties

* `BAMBOO_WRAPPER_JAVA_ADDITIONAL_PROPERTIES` (default: NONE)

  Adds `wrapper.java.additional.X` entries to the Agent's `conf/wrapper.conf`. It was designed to pass multiple key-value properties separated by the `#`. Example: `log4j2.configurationFile=/path/to/log4j2.properties#javax.net.ssl.keyStore=/var/atlassian/application-data/bamboo-agent/ssl/bamboo.secure.client.ks`

## Extending base image

This Docker image contains only minimal setup to run a Bamboo agent which might not be sufficient to run your builds. If you need additional capabilities you can extend the image to suit your needs.

Example of extending the agent base image by Maven and Git:
```Dockerfile
FROM atlassian/bamboo-agent-base:9.6.8
USER root
RUN apt-get update && \
    apt-get install maven -y && \
    apt-get install git -y

USER ${BAMBOO_USER}
RUN /bamboo-update-capability.sh "system.builder.mvn3.Maven 3.3" /usr/share/maven
RUN /bamboo-update-capability.sh "system.git.executable" /usr/bin/git
```

## Building your own image

* Clone the Atlassian repository at <https://bitbucket.org/atlassian-docker/docker-bamboo-agent-base>
* Modify or replace the [Jinja](https://jinja.palletsprojects.com/) templates
  under `config`; _NOTE_: The files must have the `.j2` extensions. However you
  don't have to use template variables if you don't wish.
* Build the new image with e.g: `docker build --tag my-bamboo-image --platform linux/amd64,linux/arm64 --build-arg BAMBOO_VERSION=X.Y.Z .`
* Optionally push to a registry, and deploy.

## Issue tracker

* You can view know issues [here](https://jira.atlassian.com/projects/BAM/issues/filter=allissues).
* Please contact our support if you encounter any problems with this Dockerfile.

## Supported JDK versions and base images

Bamboo Docker images are based on JDK 11, JDK 17 (from Bamboo 9.4), and JDK 21 (from Bamboo 10.1) and generated from the
[official Eclipse Temurin OpenJDK Docker images](https://hub.docker.com/_/eclipse-temurin) and
[Red Hat Universal Base Images](https://catalog.redhat.com/software/containers/ubi9/openjdk-17/61ee7c26ed74b2ffb22b07f6?architecture=amd64).
Starting in Bamboo 9.4, UBI tags are available in 2 formats: `<version>-ubi9`, `<version>-ubi9-jdk17`, or `<version>-ubi9-jdk21>`

The Docker images follow the [Atlassian Support end-of-life
policy](https://confluence.atlassian.com/support/atlassian-support-end-of-life-policy-201851003.html);
images for unsupported versions of the products remain available but will no longer
receive updates or fixes.

However, Bamboo is an exception to this. Due to the need to support JDK 11 and
Kubernetes, we currently only generate new images for Bamboo 9.1 and up. Legacy
builds for JDK 8 are still available in Docker Hub, and building custom images
is available (see above).

Historically, we have also generated other versions of the images, including
JDK 8, Alpine, and 'slim' versions of the JDK. These legacy images still exist in
Docker Hub, however they should be considered deprecated, and do not receive
updates or fixes.

If for some reason you need a different version, see "Building your own image".

## Migration to UBI

If you have been mounting any files to `${JAVA_HOME}` directory in `eclipse-temurin` based container, `JAVA_HOME` in UBI container is set to `/usr/lib/jvm/java-17` (JDK17) and `/usr/lib/jvm/java-21` (JDK21).

Also, if you have been mounting and running any custom scripts in the container, UBI-based images may lack some tools and utilities that are available out of the box in eclipse-temurin tags. If that's the case, see [Building your own image](#building-your-own-image).

## Supported architectures

Currently, the Atlassian Docker images are built for the `linux/amd64` and `linux/arm64` target
platforms; we do not have other architectures on our roadmap at this
point. However, the Dockerfiles and support tooling have now had all
architecture-specific components removed, so if necessary it is possible to
build images for any platform supported by Docker.

### Building on the target architecture

The simplest method of getting a platform image is to build it on a target
machine; and specify either `linux/amd64` or `linux/arm64`. See [Building your own image](#building-your-own-image) above.

Note: This method is known to work on Mac M1 and AWS ARM64 machines, but has not
been extensively tested.

## Support

For product support, go to <https://support.atlassian.com>

You can also visit the [Atlassian Data Center](https://community.atlassian.com/t5/Atlassian-Data-Center-on/gh-p/DC_Kubernetes)
forum for discussion on running Atlassian Data Center products in containers.
