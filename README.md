# Atlassian JIRA Service Desk

[![Docker Pulls](https://img.shields.io/docker/pulls/draca/atlassian-jira-servicedesk.svg)](https://hub.docker.com/r/draca/atlassian-jira-servicedesk/)
[![Build Status](https://img.shields.io/docker/build/draca/atlassian-jira-servicedesk.svg)](https://hub.docker.com/r/draca/atlassian-jira-servicedesk/builds/)
[![Docker Stars](https://img.shields.io/docker/stars/draca/atlassian-jira-servicedesk.svg)](https://hub.docker.com/r/draca/atlassian-jira-servicedesk/)

This image enables you to run [Atlassian JIRA Service Desk](https://www.atlassian.com/software/jira/service-desk).

It is based on Alpine Linux to provide an as small as possible image.
* Versions prior to 7.12 (or service desk 3.15) use [alpine-java](https://hub.docker.com/r/anapsix/alpine-java/) to provide an Oracle JDK.
* Starting from 7.13 (or service desk 3.16) [adoptopenjdk/openjdk8](https://hub.docker.com/r/adoptopenjdk/openjdk8/) is used since Atlassian supports OpenJDK from this version.
* An -oracle tag is also available for versions starting from 7.13

# Notice

In order to set up the autogeneration of images the repositories had to be restructured meaning that if you were using the old (application)-X.Y.Z style tags they are now obsolete, you should switch to the X.Y.Z style tags.

# Versions

There are tags available for latest, latest major, latest minor and individual versions. If for example you want to run the latest 3.15 version, you can use draca/atlassian-jira-servicedesk:3.15

# Autogeneration

The images are autogenerated as soon as they appear for download on the Atlassian website. This means that sometimes things might break, be aware of this and as always test in staging environments first.

You can find the script that generates these in the [atlassian-generator](https://github.com/draca-be/atlassian-generator) repository. Feel free to create pull requests to that repository if you want to make improvements either to the script or the Dockerfile templates. Please do not make pull requests against this repository as they will be ignored.

# Environment variables

A number of environment variables are supported.

## Run as non-root

By default the application runs as a non-root user. You can influence which user by setting these variables. Note that the names need to be known inside the container so results might not be what you expect.

* RUN_USER
* RUN_GROUP

Note that if you change the username, Java requires that the HOME directory for that user exists, this image automatically assigns the data folder as home directory.

## Run behind a proxy

If you are running the application behind a reverse proxy you need to set these variables so that it knows where to redirect requests to.

* JIRA_PROXY_NAME : the hostname (for example jira.mycompany.com)
* JIRA_PROXY_PORT : the port (for example 80 or 443)
* JIRA_PROXY_SCHEME : either http or https
* JIRA_CONTEXT_PATH : if the application isn't running on the root path (for example mycompany.com/jira: set this to jira)

## Disable mail

If you want to disable the incoming and outgoing mail on for example a staging server set DISABLE_NOTIFICATIONS to TRUE

## Memory

Change the default JVM memory size

* JVM_MINIMUM_MEMORY
* JVM_MAXIMUM_MEMORY

## Additional JVM args

If you need to pass additional args you can set the JIRA_ARGS variable.

## Timezone

You can set the CONTAINER_TZ variable to set the default timezone in your container. Jira inherits this if it is configured to use the system default.

## Access logs

This image disables the Tomcat access logs by default as they can grow quite large for popular instances and quickly fill up the container. Should you have need for them you can enable them again by setting KEEP_ACCESS_LOGS to TRUE. You probably also want to mount a volume to /opt/atlassian/jira/install/logs.

# Self-signed certificates

The bane of every Atlassian Expert their existence! But fear no longer as this image can automatically import the certificates into the key database. It searches for files ending with .crt in /opt/atlassian/jira/certs so just mount a volume and Bob's your uncle.

If you don't know how to get the certificates here's a simple one-liner fetching the certificate from Google, replace the domain with the one you want to import from.

```
openssl s_client -connect google.com:443 < /dev/null | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > google-public.crt
```

# Volumes

If you want to mount a volume or a directory to store your data outside of the container you should mount it over /opt/atlassian/jira/data

# Usage

Example:

    docker run -it --rm -p 8080:8080 draca/atlassian-jira-servicedesk

A very quick docker-compose file could be:

```
version: '3'
services:
  jira:
    image: draca/atlassian-jira-servicedesk
    environment:
      - DISABLE_NOTIFICATIONS=TRUE
      - JIRA_ARGS=-Datlassian.plugins.enable.wait=300
    volumes:
      - ./data:/opt/atlassian/jira/data
    ports:
      - 8080:8080
    restart: always

  jiradb:
    image: postgres:9.6
    environment:
      - POSTGRES_PASSWORD=secret
      - POSTGRES_USER=jira
      - POSTGRES_DB=jira
    volumes:
      - ./db:/var/lib/postgresql/data
    restart: always
```

# Disclaimer

A lot of care was taken in creating these images however running them is at your own risk and no claims can be made should data loss occur. By using these images you confirm that you are complying by any and all of the licenses of the 3rd party software included in this build.