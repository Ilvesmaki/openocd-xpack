# How to build the Nuclei OpenOCD

![Build Nuclei OpenOCD](https://github.com/riscv-mcu/openocd-xpack/workflows/Build%20Nuclei%20OpenOCD/badge.svg)

## Introduction

This project also includes the scripts and additional files required to
build and publish the
[Nuclei OpenOCD](https://github.com/riscv-mcu/riscv-openocd) binaries.

The build scripts use the
[xPack Build Box (XBB)](https://github.com/xpack/xpack-build-box),
a set of elaborate build environments based on GCC 7.4 (Docker containers
for GNU/Linux and Windows or a custom folder for MacOS).

## Repositories

- `https://github.com/riscv-mcu/riscv-openocd` - the URL of the
  [Nuclei OpenOCD fork](https://github.com/riscv-mcu/riscv-openocd)
- `https://github.com/riscv/riscv-openocd` - the URL of the
  [upstream RISC-V OpenOCD](https://github.com/riscv/riscv-openocd).

The build scripts use the first repo; to merge
changes from upstream it is necessary to add a remote named
`upstream`, and merge the `upstream/riscv` into the local branch.

### Branches

- `nuclei-master` - the updated content, used during builds and releases
- `nuclei-develop` - the updated content, used during development

## Prerequisites

The prerequisites are common to all binary builds. Please follow the
instructions in the separate
[Prerequisites for building binaries](https://xpack.github.io/xbb/prerequisites/)
page and return when ready.

## Download the build scripts

The build scripts are available in the `scripts` folder of the
[`riscv-mcu/openocd-xpack`](https://github.com/riscv-mcu/openocd-xpack)
Git repo.

> Nuclei build script branch is *xpack-nuclei-build*

To download them, the following shortcut is available:

```console
$ curl -L https://github.com/riscv-mcu/openocd-xpack/raw/xpack-nuclei-build/scripts/git-clone.sh | bash
```

This small script issues the following two commands:

```console
$ rm -rf ~/Downloads/openocd-xpack.git
$ git clone --recurse-submodules https://github.com/riscv-mcu/openocd-xpack.git \
~/Downloads/openocd-xpack.git
```

> Note: the repository uses submodules; for a successful build it is
> mandatory to recurse the submodules.


## The `Work` folder

The scripts create a temporary build `Work/openocd-${version}` folder in
the user home. Although not recommended, if for any reasons you need to
change the location of the `Work` folder,
you can redefine `WORK_FOLDER_PATH` variable before invoking the script.

> You can made changes for openocd directly to `Work/openocd-${version}` folder,
> and when you rebuild openocd binaries for the same version, your changes
> will be included, since the folder will not checkout again, unless you
> remove it or change the `script/VERSION` file.
> See function `download_openocd` defined in [scripts/common-apps-functions-source.sh](scripts/common-apps-functions-source.sh)

## How to build local/native binaries

## How to build distributions

### Build

#### Build the Intel GNU/Linux and Windows binaries

The current platform for Intel GNU/Linux and Windows production builds is a
Ubuntu 20.04 LTS.

> By default, it will build release version defined [scripts/VERSION](scripts/VERSION).
> If you want to change to `develop` version, just change it to `develop`.
> The openocd version build and commit relationship is defined in [scripts/common-versions-source.sh](scripts/common-versions-source.sh).

Before starting a build, check if Docker is started:

```console
$ docker info
```

Before running a build for the first time, it is recommended to preload the
docker images.

```console
$ bash ~/Downloads/openocd-xpack.git/scripts/build.sh preload-images
```

The result should look similar to:

```console
$ docker images
REPOSITORY          TAG                              IMAGE ID            CREATED             SIZE
ilegeul/ubuntu      i386-12.04-xbb-v3.2              fadc6405b606        2 days ago          4.55GB
ilegeul/ubuntu      amd64-12.04-xbb-v3.2             3aba264620ea        2 days ago          4.98GB
hello-world         latest                           bf756fb1ae65        5 months ago        13.3kB
```

Since the build takes a while, use `screen` to isolate the build session
from unexpected events, like a broken
network connection or a computer entering sleep.

```console
$ screen -S openocd

$ sudo rm -rf ~/Work/openocd-*
# Build for all platforms, if you only need one platform, check section <Subsequent runs>
$ bash ~/Downloads/openocd-xpack.git/scripts/build.sh --all
```

or, for development builds:

```console
$ bash ~/Downloads/openocd-xpack.git/scripts/build.sh --develop --without-pdf --linux64 --linux32 --win64 --win32
```

To detach from the session, use `Ctrl-a` `Ctrl-d`; to reattach use
`screen -r openocd`; to kill the session use `Ctrl-a` `Ctrl-k` and confirm.

About serval minutes later(it depends on machine), the output of the build script is a set of 4
archives and their SHA signatures, created in the `deploy` folder:

```console
$ ls -l ~/Work/openocd-*/deploy
total 13568
-rw-rw-rw- 1 hqfang nucleisys 3735749 Oct 28 14:35 nuclei-openocd-0.10.0-14-linux-x32.tar.gz
-rw-rw-rw- 1 hqfang nucleisys     108 Oct 28 14:35 nuclei-openocd-0.10.0-14-linux-x32.tar.gz.sha
-rw-rw-rw- 1 hqfang nucleisys 3671257 Oct 28 14:28 nuclei-openocd-0.10.0-14-linux-x64.tar.gz
-rw-rw-rw- 1 hqfang nucleisys     108 Oct 28 14:28 nuclei-openocd-0.10.0-14-linux-x64.tar.gz.sha
-rw-rw-rw- 1 hqfang nucleisys 3232054 Oct 28 14:37 nuclei-openocd-0.10.0-14-win32-x32.zip
-rw-rw-rw- 1 hqfang nucleisys     105 Oct 28 14:37 nuclei-openocd-0.10.0-14-win32-x32.zip.sha
-rw-rw-rw- 1 hqfang nucleisys 3226868 Oct 28 14:32 nuclei-openocd-0.10.0-14-win32-x64.zip
-rw-rw-rw- 1 hqfang nucleisys     105 Oct 28 14:32 nuclei-openocd-0.10.0-14-win32-x64.zip.sha

```

To copy the files from the build machine to the current development
machine, either use NFS to mount the entire folder, or open the `deploy`
folder in a terminal and use `scp`:

```console
$ (cd ~/Work/openocd-*/deploy; scp * your_account@your_server:Downloads/xpack-binaries/openocd)
```

### Subsequent runs

#### Separate platform specific builds

Instead of `--all`, you can use any combination of:

```
--linux32 --linux64
--win32 --win64 
```

#### `clean`

To remove most build temporary files, use:

```console
$ bash ~/Downloads/openocd-xpack.git/scripts/build.sh --all clean
```

To also remove the library build temporary files, use:

```console
$ bash ~/Downloads/openocd-xpack.git/scripts/build.sh --all cleanlibs
```

To remove all temporary files, use:

```console
$ bash ~/Downloads/openocd-xpack.git/scripts/build.sh --all cleanall
```

Instead of `--all`, any combination of `--win32 --win64 --linux32 --linux64`
will remove the more specific folders.

## Files cache

The XBB build scripts use a local cache such that files are downloaded only
during the first run, later runs being able to use the cached files.

However, occasionally some servers may not be available, and the builds
may fail.

The workaround is to manually download the files from an alternate
location (like
https://github.com/xpack-dev-tools/files-cache/tree/master/libs),
place them in the XBB cache (`Work/cache`) and restart the build.

## More build details

The build process is split into several scripts. The build starts on
the host, with `build.sh`, which runs `container-build.sh` several
times, once for each target, in one of the two docker containers.
Both scripts include several other helper scripts. The entire process
is quite complex, and an attempt to explain its functionality in a few
words would not be realistic. Thus, the authoritative source of details
remains the source code.

## Notes

If you want to redistribute your own version, you can change these files:

* [scripts/common-versions-source.sh](scripts/common-versions-source.sh)
* [scripts/defs-source.sh](scripts/defs-source.sh)
* [scripts/VERSION](scripts/VERSION)

For more details, please check the other documentations and commit history.