# Hands-on : build images with Buildah

## Prerequisistes: installation of Buildah

Read the [Installation](https://podman.io/getting-started/installation/) documentation (Very similar to Podman).

Check if everything works (no need of `sudo`):

```
buildah version
```

## Build a simple image

```sh
# Create a new container on disk from alpine
newcontainer=$(buildah from alpine)

# Update packages and Install bash
buildah run $newcontainer -- apk update
buildah run $newcontainer -- apk add bash

# Add a sample script as entrypoint
buildah copy $newcontainer ./runecho.sh /usr/bin
buildah config --entrypoint /usr/bin/runecho.sh $newcontainer

# Inspect the container image meta data
buildah inspect $newcontainer

# Commit the container to an OCI image called demo-wescale-training
buildah commit $newcontainer demo-wescale-training

# List the images we have
buildah images
```

## Run the simple image

1. Run the container using Podman
2. Run the container using Docker
    1. Do you see your image when typing `docker images`?
    2. Push the image to the local Docker repository using docker-daemon
    ```sh
    podman push demo-wescale-training docker-daemon:demo-wescale-training:latest
    ```

## Build with a Dockerfile

See the Dockerfile:
```
FROM alpine

RUN apk update && apk add bash &&\
    echo '#!/usr/bin/env bash' > /usr/bin/entrypoint.sh &&\
    echo 'echo "Hi from Bud!"' >> /usr/bin/entrypoint.sh &&\
    chmod +x /usr/bin/entrypoint.sh

ENTRYPOINT /usr/bin/entrypoint.sh
```

How to build it with `buildah`?


---

Inspired from: https://github.com/containers/buildah/blob/master/demos/