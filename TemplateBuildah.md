# Build d’image docker avec Buildah

Category: Kubernetes
Created By: Thibault Lengagne
Created Time: August 19, 2022 10:05 AM
Edited By: Oussama Cherifi-Alaoui
Stack: AWS, GitLab
Type: Operational Documentation

## Description

---

> Describe in a few words what your Operational Documentation is about

Buildah allows us to build docker image more securely, without privileged pod.

This documentation explains how to configure and use Buildah with Gitlab Runners.

## Prerequisite

---

> **Think about everything that is mandatory before following this Operational Documentation**

Kubernetes Cluster

Gitlab Runner

## Details

---

> **Describe precisely each step of your Operational Documentation**

### Create the image with buildah and useful tools

- Dockerfile

```docker
FROM quay.io/buildah/stable:v1.26

RUN dnf install -y jq python3 python3-pip && dnf clean all && \
 pip install awscli
```

- Build the image : `buildah build -t <buildah_image>:<tag> .`

### Edit gitlab jobs to use buildah instead of docker

Changes from docker

- Change the image name: `image: ${DOCKER_REGISTRY}/<buildah_image>:<tag>`
- Remove `dind` service :

    ```yaml
    services:
     - name: docker:20.10.8-dind
      [...]
    ```

- Change login method to the registry :

    ```yaml
    - aws ecr get-login-password --region <aws_region> | buildah login --username AWS --password-stdin ${DOCKER_REGISTRY}
    ```

- Replace all instances of `docker` with `buildah`
- In the `docker build` command, the `.` (Dockerfile folder) must be at the end, after all flags
- The flag `-all-tags` doesn’t work with buildah: a push per tag is required `buildah push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:<tag>`
- Remove docker’s variables:

    ```yaml
    variables:
     DOCKER_HOST: tcp://docker:2375
     DOCKER_TLS_CERTDIR: ""
    ```

- Add variables for buildah

    ```yaml
    variables:
     # Overwrite storage request because of the size of the docker image
     # See https://docs.gitlab.com/runner/executors/kubernetes.html#overwriting-container-resources
     KUBERNETES_EPHEMERAL_STORAGE_REQUEST: 30Gi
     # Use vfs with buildah. Docker offers overlayfs as a default, but buildah
     # cannot stack overlayfs on top of another overlayfs filesystem.
     STORAGE_DRIVER: vfs
     # You may need this workaround for some errors: https://stackoverflow.com/a/70438141/1233435
     BUILDAH_ISOLATION: chroot
     BUILDAH_FORMAT: docker
    ```

- For caching, add these paremeters to the `buildah build` command (from v1.27)

    ```bash
    --layers
    --cache-from ${DOCKER_REGISTRY}/${DOCKER_IMAGE}
    ```

### Configure Gitlab runner

- Add this config to gitlab runner `values.yaml` :

```yaml
runners:
 config: |
    [[runners]]
      [runners.kubernetes]
        privileged = false
        # Limit overwrite from ci Job
        # See https://docs.gitlab.com/runner/executors/kubernetes.html#overwriting-container-resources
        ephemeral_storage_request_overwrite_max_allowed = "50Gi"
        ephemeral_storage_limit_overwrite_max_allowed = "50Gi"
      [runners.kubernetes.pod_annotations]
        "container.apparmor.security.beta.kubernetes.io/build" = "unconfined"
        "container.seccomp.security.alpha.kubernetes.io/build" = "unconfined"
```

## Code Examples

---

> **Give code example, or links to Git**

- Gitlab CI example

```bash
.build:
  image: ${DOCKER_REGISTRY}/<buildah_image>:<tag>
  script:
    - cd $WORKDIR
    - aws ecr get-login-password --region eu-west-1 | buildah login --username AWS --password-stdin ${DOCKER_REGISTRY}
    - >
      buildah build
      ${DOCKER_BUILD_ARGS}
      -t ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${ENV_DOCKER_TAG}
      -t ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}
      -f ${DOCKERFILE} .
    - buildah push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}
  variables:
  WORKDIR: .
    DOCKERFILE: "Dockerfile"
    # Overwrite storage request because of the size of the docker image
    # See https://docs.gitlab.com/runner/executors/kubernetes.html#overwriting-container-resources
    KUBERNETES_EPHEMERAL_STORAGE_REQUEST: 30Gi
    # Use vfs with buildah. Docker offers overlayfs as a default, but buildah
    # cannot stack overlayfs on top of another overlayfs filesystem.
    STORAGE_DRIVER: vfs
    # You may need this workaround for some errors: https://stackoverflow.com/a/70438141/1233435
    BUILDAH_ISOLATION: chroot
    BUILDAH_FORMAT: docker
```

## Checks

---

> **Explain how to check that the steps have been followed successfully**

- Deploy a testing runner with `privileged: false` and the configuration above ([Configure Gitlab runner](https://www.notion.so/Configure-Gitlab-runner-5c96ad06fed0495188d77646d82b4663))
- Edit your CI to use this testing runner (with tags)
- Run a build job

## Links

---

> Useful links

[https://github.com/containers/buildah](https://github.com/containers/buildah)
