# Docker Registry UI

## Overview

This project aims to provide a user interface for your private docker registry v2.
There is no default registry on this UI, you should add your own with the UI.
You can manage more than one registry server.
All registries will be stored in the [local storage](https://en.wikipedia.org/wiki/Web_storage#Local_and_session_storage) of your browser.

This web user interface uses [Riot](https://github.com/Riot/riot) the react-like user interface micro-library and [riot-mui](https://github.com/kysonic/riot-mui) components.

## [GitHub Page](https://joxit.github.io/docker-registry-ui) and [Live Demo](https://joxit.github.io/docker-registry-ui/demo/)

![screenshot](https://raw.github.com/Joxit/docker-registry-ui/master/screenshot.png "Screenshot of Docker Registry UI")

## Features

-   List all your repositories/images.
-   List all tags for a repository/image
-   Sort the tag list
-   One interface for many registries
-   Use a secured docker registry
-   Share your docker registry with query parameter `url` (e.g. `https://joxit.github.io/docker-registry-ui/demo?url=https://registry.example.com`)
-   Use `joxit/docker-registry-ui:static` as reverse proxy to your docker registry (This will avoid CORS).
-   Display image size (see #30)
-   Add Title when using REGISTRY_URL (see #28)
-   Alpine and Debian based images

## Getting Started

### Basic

First you need node and npm in order to download dependencies.

```sh
git clone https://github.com/Joxit/docker-registry-ui.git
cd docker-registry-ui
npm install
```

Now you can open index.html with your browser or use a http-server

```sh
npm install -g http-server
http-server
```

### Docker

The docker contains the source code and a node webserver in order to serve the docker-registry-ui.

#### Get the docker image

You can get the image in three ways

From sources with this command:

```sh
git clone https://github.com/Joxit/docker-registry-ui.git
# Alpine
docker build -t joxit/docker-registry-ui:latest docker-registry-ui
docker build -t joxit/docker-registry-ui:static -f docker-registry-ui/static.dockerfile docker-registry-ui
# Debian
docker build -t joxit/docker-registry-ui:debian -f docker-registry-ui/debian.dockerfile docker-registry-ui
docker build -t joxit/docker-registry-ui:static -f docker-registry-ui/debian-static.dockerfile docker-registry-ui
```

Or build with the url:

```sh
# Alpine
docker build -t joxit/docker-registry-ui:latest github.com/Joxit/docker-registry-ui
docker build -t joxit/docker-registry-ui:static -f static.dockerfile github.com/Joxit/docker-registry-ui
# Debian
docker build -t joxit/docker-registry-ui:debian -f debian.dockerfile github.com/Joxit/docker-registry-ui
docker build -t joxit/docker-registry-ui:debian-static -f debian-static.dockerfile github.com/Joxit/docker-registry-ui
```

Or pull the image from [docker hub](https://hub.docker.com/r/joxit/docker-registry-ui/):

```sh
# Alpine
docker pull joxit/docker-registry-ui:latest
docker pull joxit/docker-registry-ui:static
# Debian
docker pull joxit/docker-registry-ui:debian
docker pull joxit/docker-registry-ui:debian-static
```

#### Run the docker

To run the docker and see the website on your 80 port, try this:

```sh
docker run -d -p 80:80 joxit/docker-registry-ui
```

#### Run the static docker

Some env options are available for use this interface for only one server.

-   `URL`: set the static URL to use (You will need CORS configuration). Example: `http://127.0.0.1:5000`. (`Required`)
-   `REGISTRY_URL`: your docker registry URL to contact (CORS configuration is not needed). Example: `http://my-docker-container:5000`. (Can't be used with `URL`, since 0.3.2).
-   `DELETE_IMAGES`: if this variable is empty or `false`, delete feature is deactivated. It is activated otherwise.
-   `REGISTRY_TITLE`: Set a custom title for your user interface when using `REGISTRY_URL` (since 0.3.4)

Example with `URL` option.

```sh
docker run -d -p 80:80 -e URL=http://127.0.0.1:5000 -e DELETE_IMAGES=true joxit/docker-registry-ui:static
```

Example with `REGISTRY_URL`, this will add a proxy to your registry.
Your registry will be accessible here : `http://127.0.0.1/v2`, this will avoid CORS errors (see #25).
Be careful, `joxit/docker-registry-ui` and `registry:2` will communicate, both containers should be in the same network or use your private IP.

```sh
docker network create registry-ui-net
docker run -d --net registry-ui-net --name registry-srv registry:2
docker run -d --net registry-ui-net -p 80:80 -e REGISTRY_URL=http://registry-srv:5000 -e DELETE_IMAGES=true -e REGISTRY_TITLE="My registry" joxit/docker-registry-ui:static
```

## Using CORS

Your server should be configured to accept CORS.

If your docker registry does not need credentials, you will need to send this HEADER:

    Access-Control-Allow-Origin: '*'

If your docker registry need credentials, you will need to send these HEADERS:

```yml
http:
  headers:
    Access-Control-Allow-Origin: '<your docker-registry-ui url>'
    Access-Control-Allow-Credentials: true
    Access-Control-Allow-Methods: ['HEAD', 'GET', 'OPTIONS'] # Optional
```

## Using delete

For deleting images, you need to activate the delete feature in your registry:

```yml
storage:
    delete:
      enabled: true
```

And you need to add these HEADERS:

```yml
http:
  headers:
    Access-Control-Allow-Methods: ['HEAD', 'GET', 'OPTIONS', 'DELETE']
    Access-Control-Expose-Headers: ['Docker-Content-Digest']
```

## Registry example

Example of docker registry configuration file:

```yml
version: 0.1
log:
  fields:
    service: registry
storage:
  delete:
    enabled: true
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
    Access-Control-Allow-Origin: ['http://127.0.0.1:8001']
    Access-Control-Allow-Methods: ['HEAD', 'GET', 'OPTIONS', 'DELETE']
    Access-Control-Allow-Headers: ['Authorization']
    Access-Control-Max-Age: [1728000]
    Access-Control-Allow-Credentials: [true]
    Access-Control-Expose-Headers: ['Docker-Content-Digest']
auth:
  htpasswd:
    realm: basic-realm
    path: /etc/docker/registry/htpasswd
```
