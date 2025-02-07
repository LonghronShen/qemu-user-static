# qemu-user-static

[![License](https://img.shields.io/github/license/alekitto/qemu-user-static.svg?style=flat-square)](./LICENSE) ![actions](https://github.com/alekitto/qemu-user-static/workflows/actions/badge.svg) [![Releases](https://img.shields.io/github/commits-since/alekitto/qemu-user-static/latest.svg?style=flat-square)](https://github.com/alekitto/qemu-user-static/releases) [![Docker Hub](https://img.shields.io/docker/pulls/alekitto/qemu-user-static.svg?style=flat-square)](https://hub.docker.com/r/alekitto/qemu-user-static/)

![](https://raw.githubusercontent.com/alekitto/dockerfile/master/logo.jpg)

**alekitto/qemu-user-static** is to enable an execution of different multi-architecture containers by QEMU [1] and binfmt_misc [2].
Here are examples with Docker [3].

## Getting started

```
$ uname -m
x86_64

$ docker run --rm -t arm64v8/ubuntu uname -m
standard_init_linux.go:211: exec user process caused "exec format error"

$ docker run --rm --privileged alekitto/qemu-user-static --reset -p yes

$ docker run --rm -t arm64v8/ubuntu uname -m
aarch64
```

It works on many architectures and OS container images.

```
$ docker run --rm -t arm32v6/alpine uname -m
armv7l

$ docker run --rm -t ppc64le/debian uname -m
ppc64le

$ docker run --rm -t s390x/ubuntu uname -m
s390x

$ docker run --rm -t arm64v8/fedora uname -m
aarch64

$ docker run --rm -t arm32v7/centos uname -m
armv7l

$ docker run --rm -t ppc64le/busybox uname -m
ppc64le

$ docker run --rm -t i386/ubuntu uname -m
x86_64
```

Podman [4] also works.

```
$ sudo podman run --rm --privileged alekitto/qemu-user-static --reset -p yes

$ podman run --rm -t arm64v8/fedora uname -m
aarch64
```

Singularity [5] also works.

```
$ sudo singularity run docker://alekitto/qemu-user-static --reset -p yes

$ singularity run --cleanenv docker://arm64v8/fedora uname -m
aarch64
```

## Usage

### alekitto/qemu-user-static images

alekitto/qemu-user-static images are managed on the [Docker Hub](https://hub.docker.com/r/alekitto/qemu-user-static/) container repository.
The images have below tags.

**Images**

1. `alekitto/qemu-user-static` image
2. `alekitto/qemu-user-static:register` image

**Description**

* `alekitto/qemu-user-static` image container includes both a register script to register binfmt_misc entries and all the `/usr/bin/qemu-$arch-static` binary files in the container in it.
* `alekitto/qemu-user-static:register` image has only the register script binfmt_misc entries.

`alekitto/qemu-user-static` and `alekitto/qemu-user-static:register` images execute the register script that registers below kind of `/proc/sys/fs/binfmt_misc/qemu-$arch` files for all supported processors except the current one in it when running the container. See binfmt_misc manual [2] for detail of the files.
As the `/proc/sys/fs/binfmt_misc` are common between host and inside of container, the register script modifies the file on host.

```
$ cat /proc/sys/fs/binfmt_misc/qemu-$arch
enabled
interpreter /usr/bin/qemu-$arch-static
flags: F
offset 0
magic 7f454c460201010000000000000000000200b700
mask ffffffffffffff00fffffffffffffffffeffffff
```

The `--reset` option is implemented at the register script that executes `find /proc/sys/fs/binfmt_misc -type f -name 'qemu-*' -exec sh -c 'echo -1 > {}' \;` to remove binfmt_misc entry files before register the entry.
When same name's file `/proc/sys/fs/binfmt_misc/qemu-$arch` exists, the register command is failed with an error message "sh: write error: File exists".

```
$ docker run --rm --privileged alekitto/qemu-user-static [--reset][--help][-p yes][options]
```

On below image, we can not specify `-p yes` (`--persistent yes`) option. Because an interpreter's existance is checked when registering a binfmt_misc entry. As the interpreter does not exist in the container, the register script finishes with an error.

```
$ docker run --rm --privileged alekitto/qemu-user-static:register [--reset][--help][options]
```

Then the register script executes QEMU's [scripts/qemu-binfmt-conf.sh](https://github.com/qemu/qemu/blob/master/scripts/qemu-binfmt-conf.sh) script with options.
You can check `usage()` in the file about the options.

```
Usage: qemu-binfmt-conf.sh [--qemu-path PATH][--debian][--systemd CPU]
                           [--help][--credential yes|no][--exportdir PATH]
                           [--persistent yes|no][--qemu-suffix SUFFIX]
       Configure binfmt_misc to use qemu interpreter
       --help:        display this usage
       --qemu-path:   set path to qemu interpreter ($QEMU_PATH)
       --qemu-suffix: add a suffix to the default interpreter name
       --debian:      don't write into /proc,
                      instead generate update-binfmts templates
       --systemd:     don't write into /proc,
                      instead generate file for systemd-binfmt.service
                      for the given CPU. If CPU is "ALL", generate a
                      file for all known cpus
       --exportdir:   define where to write configuration files
                      (default: $SYSTEMDDIR or $DEBIANDIR)
       --credential:  if yes, credential and security tokens are
                      calculated according to the binary to interpret
       --persistent:  if yes, the interpreter is loaded when binfmt is
                      configured and remains in memory. All future uses
                      are cloned from the open file.
```

If you have `qemu-$arch-static` binary files on your local environment, you can set it to the container by `docker -v` volume mounted file.

```
$ docker run --rm --privileged alekitto/qemu-user-static:register --reset

$ docker run --rm -t arm64v8/ubuntu uname -m
standard_init_linux.go:211: exec user process caused "no such file or directory"

$ docker run --rm -t -v /usr/bin/qemu-aarch64-static:/usr/bin/qemu-aarch64-static arm64v8/ubuntu uname -m
aarch64
```

### multiplatform images

`alekitto/qemu-user-static` images are built for the following platforms:

- linux/amd64
- linux/arm64
- linux/arm/v7
- linux/ppc64le
- linux/s390x

## Contributing

We encourage you to contribute to **alekitto/qemu-user-static**! Please check out the [Contributing to alekitto/qemu-user-static guide](CONTRIBUTING.md) for guidelines about how to proceed.

See [Developers guide](docs/developers_guide.md) for detail.

## Examples & articles

Please note that some examples using compatible images are deprecated.

See [Examples & articles](docs/examples.md).

## References

* [1] QEMU: https://www.qemu.org/
* [2] binfmt_misc: https://www.kernel.org/doc/html/latest/admin-guide/binfmt-misc.html
* [3] Docker: https://www.docker.com/
* [4] Podman: https://podman.io/
* [5] Singularity: https://sylabs.io/singularity/
