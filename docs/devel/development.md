# Development Guide

**Table of Contents**

- [Development Guide](#development-guide)
  - [Requirements](#requirements)
    - [Go](#go)
    - [Node.js](#nodejs)
  - [Clone source code](#clone-source-code)
  - [Building binary](#building-binary)
  - [Building docker image](#building-docker-image)
  - [Releasing docker image](#releasing-docker-image)

This document is the canonical source of truth for things like supported toolchain versions for building TKEStack.

Please submit an [[issue]](https://github.com/tkestack/tke/issues/new/choose) on Github if you
* Notice a requirement that this doc does not capture.
* Find a different doc that specifies requirements (the doc should instead link
  here).

## Requirements

TKEStack development helper scripts assume an up-to-date GNU tools environment.

* Recent Linux distros should work out-of-the-box.

* MacOS ships with outdated BSD-based tools. We recommend installing [[macOS GNU tools]](https://apple.stackexchange.com/questions/69223/how-to-replace-mac-os-x-utilities-with-gnu-core-utilities).

* Note that Mingw64/Cygwin on Windows is not supported yet.

> If you don't have Git-LFS installed, see [Git-LFS](https://github.com/git-lfs/git-lfs) for instructions on how to install on different operating systems.

### Go

TKEStack's backend is written in [Go](http://golang.org). If you don't have a Go development environment, please [set one up](http://golang.org/doc/code.html).

| TKEStack | requires Go |
| -------- | ----------- |
| 0.8-0.12 | 1.12.5      |
| 1.0+     | 1.13.3      |

> TKEStack uses [go modules](https://github.com/golang/go/wiki/Modules) to manage dependencies.
> If you have problem of accessing some go package, please check [goproxy](https://goproxy.io/).

### Node.js

TKEStack's frontend is written in [Typescript](https://www.typescriptlang.org/).
To bundle TKEStack's frontend code, you need a Node.js and NPM execution environment, you can use [nvm](https://github.com/nvm-sh/nvm/) to install them.

| TKEStack | requires Node.js | requires NPM |
| -------- | ---------------- | ------------ |
| 0.8-0.12 | 9.4+             | 5.6+         |
| 1.0-1.3  | 10.3+            | 6.1+         |
| 1.4+     | 12.5+            | 6.14+        |

## Clone source code

```sh
# Clone the repository on your machine
git clone git@github.com:tkestack/tke.git

# If you don't have a SSH key, feel free to clone using HTTPS instead
# git clone https://github.com/tkestack/tke.git
```

## Building binary

The following section is a quick start on how to build TKE on a local OS/shell
environment.

```sh
make build
```

The best way to validate your current setup is to build a small part of TKEStack.
This way you can address issues without waiting for the full build to complete.
To build a specific part of TKE use the `BINS` environment variable to let the build scripts know you want to build only a certain package/executable.

```sh
make build BINS=${package_you_want}
make build BINS="${package_you_want_1} ${package_you_want_2}"
```

*Note:* This applies to all top level folders under [tke/cmd](../../cmd).

So for the tke-gateway, you can run:

```sh
make build BINS=tke-gateway
```

If everything checks out you will have an executable in the `_output/{platform}` directory to play around with.

*Note:* If you are using `CDPATH`, you must either start it with a leading colon, or unset the variable. The make rules and scripts to build require the current directory to come first on the CD search path in order to properly navigate between directories.

```sh
cd ${working_dir}/tke
make
```

To build binaries for multiple CPU architecture:

```sh
make build.multiarch
```

To build a specific OS/arch of TKEStack use the `PLATFORMS` environment variable to let the build scripts know you want to build only for OS/arch.

```sh
make build.multiarch PLATFORMS="linux_amd64 windows_amd64 darwin_amd64"
```

## Building docker image

In a production environment, it is recommended to run the TKEStack components in the [Kubernetes](https://kubernetes.io/) cluster. TKEStack will build and push the image to the [Docker Hub](https://cloud.docker.com/u/tkestack/repository/list) after each release.

If you don't have docker installed, see [here](running-locally.md#docker) for instructions on how to install on different operating systems.

If you need to build a container image locally, you can simply execute the instructions:

```sh
make image
```

If you want to build a container image of just one or more components, you can use `IMAGES` variables to control:

```sh
make image IMAGES=${package_you_want}
make image IMAGES="${package_you_want_1} ${package_you_want_2}"
```

*Note:* This applies to all top level folders under [build/docker](../../build/docker) (except [build/docker/tools](../../build/docker/tools)).

So for the tke-platform-api, you can run:

```sh
make image IMAGES=tke-platform-api
```

To build container images for multiple CPU architecture (i.e., linux_amd64 and linux_arm64), type:

```sh
make image.multiarch
```

Above all `make image` commands will use experimental features of Docker daemon (i.e. docker build --platform). 
Please refer to [docker build docs](https://docs.docker.com/engine/reference/commandline/build/#--platform) to enable experimental features.

To build a specific OS/arch for TKEStack container images, please use the `PLATFORMS` environment variable to
let the build scripts know which OS/arch you want to build.

```sh
make image.multiarch PLATFORMS="linux_amd64 linux_arm64"
```

## Releasing docker image

Below is a quick start on how to push TKEStack container images to docker hub.

```sh
make push
```

TKEStack manages docker images via manifests and manifest lists.
Please make sure you enable experimental features in the Docker client.
You can find more details in [docker manifest docs](https://docs.docker.com/engine/reference/commandline/manifest/).

> Generally, `make push` is used by CI, not developer. Make sure you know what you are doing when using it.

For more functions of other components, please see [here](components.md).

## Development example

Before new feature development or bug fix, you should prepare a TKEStack environment. If you don't have one, please check [installation guide](https://github.com/tkestack/tke/tree/master/docs/guide/zh-CN/installation).

After modifying the code, you might want to make sure the code will work as you wish, so you should replace the component(s) with yours. What you need to do is as follows:

> Take tke-platform-controller as an example. 

1. Build a container image. Run `make image IMAGES=tke-platform-controller` in tke directory, you wil get a image named `tkestack/tke-platform-controller-amd64:v1.4.0.xx.xxxx`.
2. Retag the image for TKEStack registry. By default, you should run `docker tag tkestack/tke-platform-controller-amd64:v1.4.0.xx.xxxx registry.tke.com/library/tke-platform-controller:{unique version}`. If you changed the registry domain, please change `registry.tke.com` to the domain you set.
![registry domain](../../docs/images/registry-domain.png)
3. Save image as a archive file. Run `docker save -o tke-platform-controller.tar registry.tke.com/library/tke-platform-controller:{unique version}`
4. Send archive file to the TKEStack host. Run `scp tke-platform-controller.tar root@{your TKEStack host}:/root/`.
5. Load image on your TKEStack host. Login your TKEStack host through ssh, `ssh root@{your TKEStack host}`, and run `docker load -i tke-platform-controller.tar`.
6. Login TKEStack registry. Run `docker login registry.tke.com` in your TKEStack host, if the registry domain is changed, please use your custom domain. The registry's username and password are your TKEStack username and password.
![TKEStack username and password](../../docs/images/tkestack-user-pwd.png)
7. Push image to registry. Run `docker push registry.tke.com/library/tke-platform-controller:{unique version}` in your TKEStack host.
8. Replace component. Access TKEStack in web browser, and change tke-platform-controller's image.
![TKEStack username and password](../../docs/images/replace-component.png)

For tke-installer development, verification processes are as follows:

1. Build tke-installer binary. Run `VERSION={your tkestack version} make build BINS=tke-installer` in tke directory.
2. Send binary to your installer-node host. Run `scp _output/linux/amd64/tke-installer root@{your installer-node host}:/root/`.
3. Replace binary in container. Run `docker cp tke-installer tke-installer:/app/bin/` in installer-node host.
4. Restart container. Run `docker restart tke-installer` in installer-node host.

For web console development, please check [here](https://github.com/tkestack/tke/blob/master/web/console/README.md).